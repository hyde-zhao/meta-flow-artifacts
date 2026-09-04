---
doc_type: LLD
story_id: "CR037-S7"
story_slug: "topo-planning-skill"
cr_id: "CR-037"
tier: "A"
status: "draft"
created_at: "2026-08-05"
owner: "meta-dev"
source_hld: "process/changes/CR-037-HLD.md §4.7"
source_story: "process/stories/STORY-CR037-S7.md"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
scope: "skills/topo-planning/SKILL.md + skills/topo-planning/scripts/topo_plan_cli.py + skills/topo-planning/templates/{physical_pool,physical_ledger}.yaml.example 新建"
revision: "1.1"
revision_at: "2026-08-05"
revision_by: "meta-dev"
---

# STORY-CR037-S7-LLD：topo-planning skill + CLI 详细设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-08-05 | meta-dev | 初稿（CP5 预检通过） |
| v1.1 | 2026-08-05 | meta-dev | CP5 NEEDS_REWORK 修订：S7-A FR8 与向导 5 步流程一致性修正（CLI `--execute` 边界 = 写台账+落盘 env-file，真机部署由 S9 承接）；S7-B `--execute` 必须配 `--authorized` 显式门（DQ-037-04 独立 runtime_authorization，未授权抛 RuntimeAuthorizationError）；S8-B 联动：dry-run 可落预览 env-file 到 run 目录（`executable:false`，不碰真实 topology/ 路径），保证 dry-run 全链路下游可解析；S7-C 保持 dry-run 默认 / NLP→CLI agent 转换 / PYTHONPATH 集成不变 |

> **Tier**: A（新 skill + Prompt-Skill 边界 + CLI 契约需完整设计）  
> **Shared Fragments**: `skills/topo-config/src/*`（TopoMapper/PhysicalPool/EnvironmentDeployer，只读消费）、`skills/topo-planning/src/*`（topology_collection/limit_parser/pool_merge/exporter，S1/S2/S3/S5/S6）、`skills/case-execution/templates/topology.yaml.example`（env-file 契约）、`process/stories/STORY-CR037-S2-LLD.md`（limit-syntax.md 交付物，S7 引用）  
> **Open Items**: 无

---

## 0. 工程依据

本模块是 CR-037 topo-planning skill 的编排层与 CLI 入口。工程依据源自 HLD §4.7（skill 向导 5 步 + CLI `topo-plan`），CR-037 评审意见 G5（NLP→CLI 转换由 agent 实现，limit 语法规范为显式交付物）、G6（dry-run 默认门），以及 DQ-037-06（agent 转换 NLP→CLI，CLI 只收结构化参数）。顶层目标是把 S1-S6 数据/映射/导出链路固化为**一个可被 ptm-te 编排 [1]→[2]（S8）直接调用的 skill 向导 + CLI**：展示池 → 选/建逻辑 topo → 映射 → 导出 → 部署/验证。S7 消费 S1 `find_topology_file`、S2 `parse_limit` + `docs/limit-syntax.md`、S3 `merge_pool`、S5 `export_env_file`/`build_failure_report`、S6 `compute_ip_plan` 作为强输入，向 S8 提供 `topo-plan` CLI 与 `topo-planning` SKILL.md 产物。核心价值：让「在海光3250上执行这个用例，其中需要包含 TE|10GE 链路」这类用户诉求经 agent 转译为结构化 CLI 参数后，一条命令完成逻辑 topo → 物理 topo 映射 → env-file 产物，默认 dry-run 不产生副作用。

---

## 1. 目标

新建 `skills/topo-planning/SKILL.md`（向导 5 步编排说明）、`skills/topo-planning/scripts/topo_plan_cli.py`（CLI `topo-plan`）、`skills/topo-planning/templates/physical_pool.yaml.example` 与 `physical_ledger.yaml.example`（目标项目模板副本参考，S8 install 消费）。实现：

1. **skill 向导 5 步**：展示池 → 选/建逻辑 topo → 映射 → 导出 → 部署/验证，每步完成提示下一步（对齐 topo-config SKILL 既有 5 步模式）。
2. **CLI `topo-plan`**：`--topo-id/--case-file/--limit/--limit-file/--devices-yaml/--pool-path/--ledger-path/--output/--report-out/--dry-run`，默认 dry-run 门。
3. **NLP→CLI 转换由 agent 实现**（G5/DQ-037-06）：CLI 只收结构化参数（`--limit` inline JSON 或 `--limit-file` 文件）；agent 依据 S2 交付的 `docs/limit-syntax.md` 做 NLP→结构化转换。
4. **与 topo-config 引擎 import 集成**：统一 PYTHONPATH 策略（对齐 S1 P2-4），CLI 内 `from topo_config 模块` 与 `from topo_planning 模块` 并存，不引入 importlib 临时代码。
5. **默认 dry-run 门**：dry-run 完整映射 + 导出校验，不写台账占用、不落盘 env-file、不部署。

核心价值：把 S1-S6 的模块链封装为 ptm-te（S8）与人工均可消费的单一入口，产物直接满足 case-execution `${ENV.*}` 9 类占位符硬契约。

---

## 2. Requirements

### Functional

| ID | 需求 | 优先级 |
|----|------|:------:|
| FR1 | `topo-plan --topo-id <id>` 直接指定逻辑拓扑；`--case-file <path>` 从 PC frontmatter 双源提取 topo_id（组网约束优先/组网描述回退，E2） | P0 |
| FR2 | `--limit <json>` / `--limit-file <path>` 收结构化 topo-limit；两者互斥且至少一个为空时按空 limit 处理（S2 `parse_limit` 空 limit 语义） | P0 |
| FR3 | `--devices-yaml` / `--pool-path` / `--ledger-path` 三路径收物理数据；缺省回落 `topology/{devices.yaml,physical_pool.yaml,physical_ledger.yaml}` | P0 |
| FR4 | 向导 5 步每步输出人读摘要并提示下一步（展示池 → 选/建逻辑 topo → 映射 → 导出 → 部署/验证） | P0 |
| FR5 | 映射成功：导出 env-file（S5 `export_env_file`）；dry-run 默认 stdout 展示（`--output` 指向 run 目录时落预览副本 `executable:false`）；`--execute --authorized` 时 `--output <path>` 显式落正式 env-file（幂等原子写） | P0 |
| FR6 | 映射失败：输出结构化报告（S5 `build_failure_report` + `write_report`），`--report-out <path>` 落盘 JSON + 人读，缺省 stdout，退出码 1 | P0 |
| FR7 | 默认 `--dry-run`：不写台账占用（S4 `match` 传 `env_name=None`）、不落真实 `topology/` 路径 env-file、不部署；打印 would-allocate 清单与 env-file 文本；`--output` 指向 `runs/<run-id>/` 时落**预览副本**（`metadata.executable: false`），供下游 load_env_file 解析验证，0 次真机写 | P0 |
| FR8 | `--execute`（显式）：写台账占用（`--env-name` 或自动生成 env_name）、落盘 env-file 到 `--output`（缺省 `topology/topology.yaml`）；**必须同时配 `--authorized` 显式门**（对齐 DQ-037-04 独立 runtime_authorization），未授权抛 `RuntimeAuthorizationError`；真机部署**不**由 CLI 直接触发，由 S9 `deploy_bridge` 独立入口承接（S9 侧同样需 DQ-037-04） | P1 |
| FR9 | `--env-name <name>` 指定本次映射环境名（ledger 占用与 S8 [7] release 的关联键）；缺省自动生成（`<topo_id>-<短时间戳>`） | P1 |
| FR10 | `--list-topologies` 列出可用逻辑拓扑（S1 `list_topologies`），供向导 Step 2 展示 | P1 |
| FR11 | `--report-out` 与 `--output` 缺省路径对齐 case-execution：默认 env-file 输出到 stdout（dry-run）/ `topology/topology.yaml`（execute） | P1 |
| FR12 | `--authorized`（flag）：`--execute` 的显式授权门。`--execute` 无 `--authorized` → 抛 `RuntimeAuthorizationError` 并退出（code 3）；`--authorized` 无 `--execute` → 参数错误（code 2）。授权记录（who/scope/authorized_at/reason）由调用方（S8 编排或人工）按 DQ-037-04 双层审计维护，CLI 只做门检查 | P0 |

### Non-Functional

| ID | 需求 |
|----|------|
| NFR1 | 默认 dry-run，杜绝意外写真实设备/台账（对齐 ptm-te dry-run 默认门，ADR-02/ADR-04） |
| NFR2 | 产物/日志不输出任何凭据（ADR-02）：CLI 不读/不传 `management.password`，日志不打印密码字段 |
| NFR3 | import 集成走统一 PYTHONPATH（S8 install 注入），CLI 内不硬编码绝对路径、不 importlib 临时代码（S1 P2-4） |
| NFR4 | 向导 5 步每步可中断：任一步失败输出结构化错误 + 已做步骤摘要，不产生半成品副作用 |
| NFR5 | CLI 可独立测试：不依赖真实目标项目数据（可用 S10 fixture 造数），纯内存映射 + stdout 输出 |
| NFR6 | 确定性：同输入同池同台账 → 同 env-file（复用 S5 `sort_keys=False`），CLI 无随机成分 |
| NFR7 | `--execute` 是显式授权门：未配 `--authorized` 不得进入写台账/落盘分支，抛 `RuntimeAuthorizationError`（对齐 DQ-037-04，设计通过不等于运行授权） |

---

## 3. 模块拆分与职责

### 3.1 本 Story 新增文件

| 文件 | 职责 |
|------|------|
| `skills/topo-planning/SKILL.md` | skill 编排说明：目标 / 前置条件 / 命令格式 / 向导 5 步 / 安全约束 / 相邻对象边界 / Gotchas |
| `skills/topo-planning/scripts/topo_plan_cli.py` | CLI `topo-plan` 入口：参数解析 → 编排 S1-S6 模块 → 结果输出（env-file / 报告） |
| `skills/topo-planning/templates/physical_pool.yaml.example` | 目标项目物理池模板副本参考（P-1 模板化语义，含 `${PLACEHOLDER}` 占位与注释指引） |
| `skills/topo-planning/templates/physical_ledger.yaml.example` | 目标项目台账模板副本参考（`list + port_status` schema，P0-3） |

### 3.2 CLI 内部函数划分（`topo_plan_cli.py`）

| 函数 | 职责 |
|------|------|
| `main()` | argparse 入口：参数归一化 → 路由到 `run_plan` |
| `run_plan(args) -> int` | 编排主流程：topo 解析 → limit 解析 → 池归并 → 映射 → 导出/报告 → 返回退出码 |
| `_resolve_logical_topo(topo_id, case_file)` | 确定逻辑 Topology（S1 `find_topology_file` + parser；`--case-file` 时先 `_extract_topo_id_from_case`） |
| `_extract_topo_id_from_case(case_path)` | PC frontmatter 双源提取 topo_id（组网约束优先 / 组网描述括号回退，E2）；失败返回 None + 原因 |
| `_resolve_limit(limit, limit_file)` | 归一化 limit 输入（inline JSON / 文件 JSON / None→空），调 S2 `parse_limit` |
| `_build_physical_view(pool_path, devices_path, ledger_path)` | 加载物理数据并调 S3 `merge_pool` → `UnifiedPool`，注入 `PhysicalPool`（台账占用查询） |
| `_run_mapping(topo, topo_limit, unified_pool, env_name)` | 调 topo-config `TopoMapper.match`（含 per-link 约束 + ledger 写回，S4） |
| `_emit_success(mapping_result, topo, pool, ip_plan, output, dry_run)` | 调 S5 `export_env_file` + `compute_ip_plan`（S6）；dry-run stdout / execute 落盘 |
| `_emit_failure(mapping_result, topo, reason, report_out)` | 调 S5 `build_failure_report` + `write_report`；退出码 1 |
| `_show_pool_summary(unified_pool)` | 向导 Step 1 人读池摘要（设备/端口/占用） |
| `_show_mapping_summary(mapping_result)` | 向导 Step 3 人读映射摘要 |
| `_show_next_step(step_name)` | 向导每步结束的「下一步提示」 |

---

## 4. 代码结构与文件影响范围

### 新增文件（S7 写入范围）

```
skills/topo-planning/SKILL.md                              # skill 编排说明（~200 行）
skills/topo-planning/scripts/topo_plan_cli.py              # CLI 主入口（~450 行）
skills/topo-planning/templates/physical_pool.yaml.example   # 物理池模板副本参考（P-1 模板化语义）
skills/topo-planning/templates/physical_ledger.yaml.example # 台账模板副本参考（list + port_status，P0-3）
```

### 只读消费（不修改）

```
skills/topo-planning/src/topology_collection.py   # S1 find_topology_file / list_topologies
skills/topo-planning/src/limit_parser.py          # S2 parse_limit / TopoLimit
skills/topo-planning/docs/limit-syntax.md         # S2 显式交付物（agent NLP→CLI 转换依据）
skills/topo-planning/src/pool_merge.py            # S3 merge_pool / UnifiedPool
skills/topo-planning/src/exporter.py              # S5 export_env_file / build_failure_report / compute_ip_plan（S6）
skills/topo-config/src/topo_mapper.py             # S4 TopoMapper.match（增强后）
skills/topo-config/src/physical_pool.py           # PhysicalPool（台账查询/release，S4）
skills/topo-config/src/parser.py                  # 逻辑 topo 解析（parse_topology）
skills/case-execution/templates/topology.yaml.example  # env-file 契约参考
```

### 不修改

```
skills/topo-config/**         # topo-config 引擎只读消费（S4 已在其写入范围完成增强）
resource/network-topology/**  # 共享只读资源
```

### 与 S8 的接口关系（文件所有权边界）

- S7 primary：`skills/topo-planning/SKILL.md` + `scripts/topo_plan_cli.py` + `templates/*.example`。
- S8 消费 S7 产物：ptm-te 编排 [1]→[2] 调用 `topo-plan` CLI；install.py 安装 `topo-planning` skill（含 S7 文件）并生成目标项目模板副本。
- S7 不修改 `agents/ptm-te.md` / `script/install.py`（S8 写入范围）。

---

## 5. 数据模型与持久化设计

### 5.1 CLI 参数归一化结果（内部 dict）

```python
# run_plan 内部统一结构
{
    "topo_id": "node2_dut1_tg1_link3",       # 来源：--topo-id 或 --case-file 提取
    "topo_limit": <TopoLimit | None>,        # 来源：--limit/--limit-file → S2 parse_limit
    "pool_path": "topology/physical_pool.yaml",
    "devices_path": "topology/devices.yaml",
    "ledger_path": "topology/physical_ledger.yaml",
    "env_name": "node2_dut1_tg1_link3-20260805T1200",  # --env-name 或自动生成
    "output": None,                          # --output 显式路径（execute 落盘）
    "report_out": None,                      # --report-out 显式路径
    "dry_run": True,                         # 默认 true
}
```

### 5.2 env-file 产物（S5 契约，S7 不重新定义）

S7 CLI 直接透传 S5 `export_env_file` 的输出：`metadata + port_mapping + nodes + links` 四段，`${ENV.*}` 9 类占位符解析路径对齐 HLD §7 与 `topology.yaml.example`。S7 只负责调用与展示，不复制契约定义。

### 5.3 结构化失败报告（S5 契约，S7 不重新定义）

S7 CLI 失败路径直接调 S5 `build_failure_report`，问题类型枚举（`missing_device` / `missing_te_port` / `link_constraint_unsatisfied`）由 S5 维护。

### 5.4 模板副本参考文件（S7 产出，S8 消费）

`templates/physical_pool.yaml.example` 与 `physical_ledger.yaml.example` 是**目标项目模板副本的源参考**，语义对齐 P-1 模板化后的 `skills/topo-config/config/physical_pool.yaml`（占位符 + 注释指引）与 `physical_ledger.yaml`（`list + port_status`，P0-3）。二者**不含任何测试造数**（`hg3250-51` 等仅存在于 `tests/fixtures/`）。S8 install 生成模板副本时以 `skills/topo-config/config/` 为源（P-1 改造后），S7 templates 供人工参考与 S10 校验对比。

### 5.5 持久化

- dry-run：**无持久化**（env-file 到 stdout，台账不写，env-file 不落盘）。
- execute：env-file 落 `--output`（缺省 `topology/topology.yaml`，原子写）；台账占用由 S4 `_commit_allocation` 写回（S7 传 `env_name`）。
- CLI 自身不维护运行态文件；`runs/<run-id>/` 由 S8 ptm-te 编排层管理。

---

## 6. API / Interface 设计

### 6.1 CLI 命令格式

```bash
python skills/topo-planning/scripts/topo_plan_cli.py plan \
  [--topo-id <topo_id> | --case-file <case.md>] \
  [--limit <json> | --limit-file <limit.json>] \
  [--devices-yaml <path>] [--pool-path <path>] [--ledger-path <path>] \
  [--output <env-file-path>] [--report-out <report-path>] \
  [--env-name <name>] [--list-topologies] [--dry-run | --execute] [--authorized]
```

（`plan` 子命令；`--list-topologies` 独立列出可用拓扑。缺省模式 = `--dry-run`。`--execute` 必须配 `--authorized` 显式门，未授权抛 `RuntimeAuthorizationError`。）

### 6.2 参数明细

| 参数 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `--topo-id` | str | None | 逻辑拓扑 ID（如 `node2_dut1_tg1_link3`）；与 `--case-file` 二选一 |
| `--case-file` | str | None | PC 文件路径；从 frontmatter 双源提取 topo_id（E2） |
| `--limit` | str | None | inline JSON（结构化 topo-limit，agent 转换后）；与 `--limit-file` 互斥 |
| `--limit-file` | str | None | limit JSON 文件路径；与 `--limit` 互斥 |
| `--devices-yaml` | str | `topology/devices.yaml` | 设备清单路径 |
| `--pool-path` | str | `topology/physical_pool.yaml` | 物理池路径 |
| `--ledger-path` | str | `topology/physical_ledger.yaml` | 台账路径 |
| `--output` | str | None（execute 缺省 `topology/topology.yaml`） | env-file 输出路径 |
| `--report-out` | str | None（stdout） | 失败报告落盘路径（JSON + 人读） |
| `--env-name` | str | 自动生成 | 本次映射环境名（ledger 占用 / S8 release 关联键） |
| `--list-topologies` | flag | False | 列出可用逻辑拓扑后退出（不执行映射） |
| `--dry-run` / `--execute` | flag | `--dry-run` | dry-run：不写台账/不落真实路径/不部署；execute：写台账/落盘（workspace 写，执行前自动步骤） |
| `--authorized` | flag | False | `--execute` 的显式授权门（DQ-037-04 独立 runtime_authorization）；无 `--execute` 时忽略并报参数错误（code 2） |

> **授权分层（S7-B，对齐 DQ-037-04）**：`--execute` 的「写台账 + 落盘 env-file」是 **workspace 写**（执行前自动步骤，映射/台账占用不涉及真机下发），因此只需 `--authorized` 显式门确认，不等同于真机下发授权；若继续真机部署（S9 `deploy_bridge`），S9 侧还需独立 DQ-037-04 runtime_authorization（见 S9 技术说明 §权限变化）。CLI 不把「写台账」与「真机下发」混为一档授权。

### 6.3 `_extract_topo_id_from_case(case_path) -> Optional[tuple[str, str]]`

双源提取（E2），返回 `(topo_id, source)` 或 `(None, 原因)`：

```python
def _extract_topo_id_from_case(case_path: str):
    """从 PC frontmatter 提取 topo_id。
    优先级（E2）：
      1. 组网约束字段：值为 topo_id 形态（node\d+_.+）→ 直接采用（source='组网约束'）
      2. 组网描述字段：括号内提取 topo_id（如 'TOPO-01 基础三链路 (node2_dut1_tg1_link3)'）
         → 正则匹配 node\d+_.+（source='组网描述'）
      3. 均失败 → (None, '组网约束/组网描述均未提取到 topo_id')
    """
    fields = _parse_case_frontmatter(case_path)   # 复用 case-execution frontmatter 解析语义
    constraint = fields.get('组网约束', '') or ''
    m = re.match(r'^(node\d+_.+)$', constraint.strip())
    if m:
        return m.group(1), '组网约束'
    description = fields.get('组网描述', '') or ''
    m2 = re.search(r'(node\d+_[A-Za-z0-9_]+)', description)
    if m2:
        return m2.group(1), '组网描述'
    return None, '组网约束/组网描述均未提取到 topo_id'
```

**失败行为**：提取失败且未显式 `--topo-id` → 报 `TOPO_ID_NOT_FOUND` + 提示用户填「组网约束」或传 `--topo-id`；退出码 2。

### 6.4 与 S4 `TopoMapper.match` 的接线

```python
mapper = TopoMapper(pool)                       # pool = PhysicalPool（台账查询）
result = mapper.match(
    logical_topo=topo,
    topo_limit=topo_limit,                      # S2 TopoLimit
    port_requirement=per_link_requirement,      # S2 链路约束 → per-link（S4 归一化）
    env_name=(None if dry_run else env_name),   # dry-run 不写台账（FR7）
)
```

### 6.5 退出码约定

| 场景 | 退出码 |
|------|:---:|
| 映射成功 + 导出成功（dry-run 或 execute） | 0 |
| 映射失败（结构化报告已输出） | 1 |
| 参数错误 / topo_id 提取失败 / 输入文件缺失 / `--authorized` 无 `--execute` | 2 |
| 内部异常 / `--execute` 未配 `--authorized`（`RuntimeAuthorizationError`） | 3 |

---

## 7. 核心处理流程

### 7.1 主流程

```
topo-plan plan [args]
 │
 ├─ 1. 参数归一化
 │       ├─ --list-topologies → S1 list_topologies → 打印清单 → exit 0
 │       ├─ --topo-id 与 --case-file 均无 → exit 2（参数错误）
 │       ├─ --authorized 无 --execute → exit 2（参数错误）
 │       ├─ --execute 无 --authorized → 抛 RuntimeAuthorizationError → exit 3（S7-B 显式授权门）
 │       └─ dry_run = not args.execute（默认 True）
 │
 ├─ 2. 确定逻辑 Topology（S1 + parser）
 │       ├─ --case-file → _extract_topo_id_from_case → (topo_id, source)
 │       │      失败 → TOPO_ID_NOT_FOUND → exit 2
 │       ├─ topo_file = find_topology_file(topo_id)   # 返回 YAML 路径
 │       ├─ topo = parser.parse_topology(filepath=topo_file)  # P0-1 消费契约
 │       └─ 向导 Step 2 摘要：'选用逻辑拓扑 {topo_id}（来源: {source}）'
 │
 ├─ 3. 解析 limit（S2）
 │       ├─ limit_raw = _resolve_limit(args.limit, args.limit_file)
 │       ├─ topo_limit = limit_parser.parse_limit(limit_raw)  # 空 limit → is_empty
 │       └─ 若 topo_limit 非空 → 打印约束摘要（设备约束 N 条 / 链路约束 M 条）
 │
 ├─ 4. 物理视图（S3）
 │       ├─ unified_pool = pool_merge.merge_pool(pool_path, devices_path, ledger_path)
 │       ├─ pool = PhysicalPool.load(pool_path, ledger_path)  # 台账占用查询源
 │       └─ 向导 Step 1 摘要：_show_pool_summary(unified_pool)
 │
 ├─ 5. 映射（S4）
 │       ├─ result = TopoMapper(pool).match(topo, topo_limit, per_link, env_name or None)
 │       ├─ ip_plan = exporter.compute_ip_plan(result, topo, pool)  # S6
 │       └─ 向导 Step 3 摘要：_show_mapping_summary(result)
 │
 ├─ 6. 结果分支
 │       ├─ result.status in (matched, partial):
 │       │     if dry_run:
 │       │         env_text = exporter.export_env_file(result, topo, pool, ip_plan)
 │       │         print(env_text)                       # stdout，不落真实路径
 │       │         print(would-allocate 清单)            # 打印将写回的端口，不真实写
 │       │         if args.output 指向 runs/<run-id>/ 下:
 │       │             exporter.export_env_file(..., output=args.output,
 │       │                                      preview=True)   # 落预览副本 metadata.executable:false
 │       │     else:
 │       │         # 已通过 --authorized 门（步骤 1）
 │       │         path = exporter.export_env_file(result, topo, pool, ip_plan,
 │       │                                         output=args.output or 'topology/topology.yaml')
 │       │         print(f'env-file 已写入: {path}')
 │       │         # ledger 占用已由 S4 _commit_allocation 写回（env_name 非 None）
 │       │     向导 Step 4 完成 → Step 5 提示（部署由 S9 承接，真机下发需 DQ-037-04 独立授权）
 │       │     exit 0
 │       └─ result.status in (failed, pending):
 │             report = exporter.build_failure_report(result, topo)
 │             exporter.write_report(report, args.report_out)   # --report-out 落盘或 stdout
 │             exit 1
```

### 7.2 向导 5 步输出

| 步 | 触发 | 输出 | 下一步提示 |
|----|------|------|-----------|
| Step 1 展示池 | 流程第 4 步 | 设备/端口/占用摘要 | 「当前 N 台设备空闲。请确认逻辑拓扑或 limit。」 |
| Step 2 选/建逻辑 topo | 流程第 2 步 | 逻辑拓扑 + 来源 | 「已选用 {topo_id}。附加约束？(--limit/--limit-file)」 |
| Step 3 映射 | 流程第 5 步 | 映射摘要（matched/partial/failed + per-link 需求） | 「映射成功/失败。是否导出？(自动)」 |
| Step 4 导出 | 流程第 6 步 | env-file 文本（dry-run）或落盘路径（execute） | 「env-file 已产出。是否部署/验证？(S9 deploy_bridge)」 |
| Step 5 部署/验证 | S9 独立入口承接 | （S7 CLI 到此结束，只提示下一步；真机部署由 S9 `deploy_bridge` 处理，S7 不直接调 deployer） | 「真机部署由 S9 承接；S9 `--execute` 需 DQ-037-04 独立 runtime_authorization（S7 `--authorized` 仅覆盖台账/落盘 workspace 写）」 |

> **一致性说明（S7-A）**：FR8 与 Step 5 对齐——CLI `--execute` 的边界是「写台账占用 + 落盘 env-file」（workspace 写，需 `--authorized` 显式门）；真机下发不属 CLI 边界，由 S9 `deploy_bridge` 独立入口承接（S9 侧再走 DQ-037-04）。向导 Step 5 在 CLI 内只输出提示，不触发部署。

### 7.3 错误路径

| 步骤 | 失败条件 | 行为 | 退出码 |
|------|---------|------|:---:|
| 1 | `--topo-id` 与 `--case-file` 均缺 | 报参数错误 + 提示二选一 | 2 |
| 1 | `--authorized` 无 `--execute` | 报参数错误 + 提示 `--authorized` 仅配合 `--execute` | 2 |
| 1 | `--execute` 无 `--authorized` | 抛 `RuntimeAuthorizationError` + 提示 DQ-037-04 独立授权 | 3 |
| 2 | case frontmatter 无法解析 | `PARSE_FAILED` + 提示格式问题 | 2 |
| 2 | topo_id 双源提取失败 | `TOPO_ID_NOT_FOUND` + 提示填「组网约束」 | 2 |
| 2 | `find_topology_file` 抛 FileNotFoundError（topo 不存在） | 透传 S1 错误消息 + 候选建议 | 1 |
| 3 | `parse_limit` 抛 TopoLimitError | 透传 code + details（如 PARAM_INVALID） | 2 |
| 4 | pool/devices/ledger 文件缺失 | 透传 S3 FileNotFoundError + 提示运行 S8 install 生成模板 | 2 |
| 5 | 映射异常（未捕获） | exit 3 + 已做步骤摘要 | 3 |
| 6 | env-file 落盘失败 | `OUTPUT_WRITE_FAILED` + 结构化错误 | 1 |
| 6 | `--report-out` 落盘失败 | warning + stdout 兜底 | 1 |

---

## 8. 技术细节

### 8.1 import 集成（统一 PYTHONPATH，S1 P2-4）

CLI 运行前提：S8 install 将以下目录加入 `PYTHONPATH`（对齐 `parser.py:15-17` sys.path 策略）：

```
~/.ptm-team/skills/topo-planning/src/     # S1/S2/S3/S5/S6
~/.ptm-team/skills/topo-config/src/       # PhysicalPool / TopoMapper / parser / ipam / topology_exporter
```

CLI 内 import：

```python
from topology_collection import find_topology_file, list_topologies   # S1
from limit_parser import parse_limit, TopoLimitError                  # S2
from pool_merge import merge_pool                                      # S3
from exporter import export_env_file, build_failure_report, write_report, compute_ip_plan  # S5/S6
from physical_pool import PhysicalPool                                 # topo-config
from topo_mapper import TopoMapper                                     # topo-config（S4 增强后）
from parser import parse_topology                                      # topo-config
```

**过渡降级**：仅当 S8 统一 PYTHONPATH 未交付时，CLI 顶部 `sys.path.insert(0, <topo-planning/src>)` + `sys.path.insert(0, <topo-config/src>)` 补路径（对齐 S1 §8.3 过渡策略），不引入 `importlib.util`。

### 8.2 env-name 生成规则（FR9）

```python
def _default_env_name(topo_id: str) -> str:
    """env_name = <topo_id>-<HHMMSS 时间戳>，保证同 topo 多次映射不冲突"""
    return f"{topo_id}-{datetime.now().strftime('%H%M%S')}"
```

### 8.3 dry-run 的 would-allocate 清单与预览副本

dry-run 时 S4 `match` 传 `env_name=None`（不写回台账）；CLI 从 `MappingResult` 收集成功映射端口，打印「[DryRun] 将写回占用: {device}.{port}"」清单，供用户预览真实执行时的台账变化，不产生副作用。

**预览 env-file（S8-B 联动）**：dry-run 默认 env-file 只到 stdout。当 `--output` 指向 `runs/<run-id>/` 下的路径（如 `runs/<run-id>/topology.preview.yaml`）时，CLI 落**预览副本**到该路径：`export_env_file(..., output=..., preview=True)` 在 `metadata` 增加 `executable: false`（供下游 load_env_file 解析验证链路，但任何执行消费必须拒绝 `executable:false` 文件）；不写真实 `topology/topology.yaml`、不写台账占用、0 次真机写。预览副本属于 run 运行态（`runs/` 不入库），不触发 `.gitignore` 新增规则。

### 8.4 SKILL.md 结构（Prompt-Skill 契约）

`skills/topo-planning/SKILL.md` 正文至少包含：

| 章节 | 内容 |
|------|------|
| frontmatter | `name: topo-planning` / `description`（触发条件 + 能力边界）/ `argument-hint` / `user-invokable: true` / `status: active` |
| 目标 | 一句话 + 三价值点（向导 5 步 / CLI 一条命令 / 默认 dry-run） |
| 前置条件 | collection.md 资源（S1）、物理池/台账/devices（S8 install 生成）、PYTHONPATH（S8 注入） |
| 命令格式 | `topo-plan plan ...` 示例（dry-run / execute） |
| 向导 5 步 | 每步做什么 + 输出示例 + 下一步提示 |
| NLP→CLI 转换 | 引用 S2 `docs/limit-syntax.md`：agent 依据语法规范将自然语言转结构化 limit（DQ-037-06） |
| 产物契约 | 引用 S5 env-file（9 类 `${ENV.*}`）+ 失败报告 |
| 默认 dry-run 门 | dry-run / execute 对比表；`--execute` 必须配 `--authorized`（S7-B，workspace 写显式门）；S9 真机部署另需 DQ-037-04 独立授权 |
| 安全约束 | 凭据禁止（ADR-02）、不写真实设备（dry-run 默认） |
| 相邻对象边界 | 与 topo-config / case-execution / ptm-te / deploy_bridge(S9) 的职责差异 |
| Gotchas | import PYTHONPATH、env-name 唯一、dry-run 不写台账等 |
| 修订记录 | 版本 / 日期 / 修订人 / 变更要点 |

### 8.5 模板副本参考文件内容约束

- `physical_pool.yaml.example`：对齐 P-1 模板化 schema（`metadata.nodes.links`），`management.password` 用 `${PLACEHOLDER}`，无真实 IP/密码；**不含测试造数**。
- `physical_ledger.yaml.example`：`devices` 为 `list`，每条含 `device_id/brand/model/version/status/port_status`（`port_status[].allocated_to/allocated_vlan`，P0-3）；**不含测试造数**。

---

## 9. 安全与性能设计

### 安全

- **凭据禁止（ADR-02）**：CLI 全程不读/不传 `management.password` / `web_password` / `api_key`；日志打印设备摘要时只显示 host/型号/端口，不显示密码字段。
- **默认 dry-run 门**：dry-run 零副作用（不写台账、不落真实路径、不连设备、0 次真机写）；`--execute` 才写台账 / 落盘，且必须配 `--authorized` 显式门（S7-B，DQ-037-04 分层：台账/落盘 = workspace 写执行前自动步骤；S9 真机下发 = 独立 DQ-037-04 runtime_authorization）。
- **YAML 安全**：所有 YAML 加载/导出用 `yaml.safe_load` / `yaml.safe_dump`，不透传任意对象。
- **路径安全**：`--output` / `--report-out` / 各输入路径为显式路径参数，不拼接注入。

### 性能

- 单次 `topo-plan` 调用 = S1 解析（<100ms）+ S2 limit（<10ms）+ S3 归并（<200ms）+ S4 回溯 + S5 导出（<50ms）+ S6 IP 规划（内存），总计 < 1s（典型 7 节点池）。
- 向导输出为纯 stdout 文本，无文件 I/O（dry-run）。
- CLI 无守护进程/常驻状态，进程级生命周期。

---

## 10. 测试设计

### 单元测试（`skills/topo-planning/tests/test_topo_plan_cli.py`）

| 测试用例 | 覆盖要点 | 预期 |
|---------|---------|------|
| `test_extract_topo_id_from_constraint` | frontmatter `组网约束: node2_dut1_tg1_link3` | `('node2_dut1_tg1_link3', '组网约束')` |
| `test_extract_topo_id_from_description` | `组网描述: TOPO-01 基础三链路 (node2_dut1_tg1_link3)` | `('node2_dut1_tg1_link3', '组网描述')` |
| `test_extract_topo_id_missing` | 两字段均无 topo_id | `(None, 原因)` |
| `test_plan_success_dry_run` | fixture 池 + `--topo-id`；映射成功 | exit 0，stdout 含 env-file 文本，台账未写（mock save_ledger 断言） |
| `test_plan_success_execute` | `--execute --authorized --env-name case-001` | exit 0，env-file 落盘，台账 `allocated_to=case-001` |
| `test_execute_requires_authorized` | `--execute` 无 `--authorized`（S7-B） | exit 3 + `RuntimeAuthorizationError`，台账未写、env-file 未落盘 |
| `test_authorized_without_execute` | `--authorized` 无 `--execute` | exit 2 + 参数错误 |
| `test_dry_run_zero_real_write` | dry-run 成功（S7-B/S7-C） | 0 次真机写（mock deployer 断言无 deploy/verify/teardown 调用） |
| `test_dry_run_preview_copy` | dry-run + `--output runs/<run-id>/topology.preview.yaml`（S8-B 联动） | 落预览副本含 `metadata.executable:false`，真实 `topology/topology.yaml` 不存在，台账未写 |
| `test_plan_failure_report` | fixture 池无 TE；TE 约束 | exit 1，报告含 `missing_te_port`；`--report-out` 落盘 `<path>.json` + `.txt` |
| `test_plan_case_file` | `--case-file` 提取 topo_id 并映射 | 使用组网约束 topo_id，exit 0 |
| `test_plan_no_topo_args` | 无 `--topo-id` 无 `--case-file` | exit 2 + 参数错误 |
| `test_plan_limit_inline` | `--limit '{"device_constraints":[...]}'` | 约束生效（等价类过滤） |
| `test_plan_limit_file` | `--limit-file limit.json` | 同 inline |
| `test_plan_limit_invalid` | `--limit '{"speed_class":"TE","media_type":"copper"}'` 链路约束非法 | exit 2 + PARAM_INVALID |
| `test_list_topologies` | `--list-topologies` | 列出 12 个拓扑，exit 0 |
| `test_dry_run_no_ledger_write` | dry-run 成功后重载台账 | 台账 `allocated_to` 全空 |
| `test_auto_env_name_unique` | 同 topo 两次 dry-run | env_name 不同（时间戳） |
| `test_no_credential_in_output` | 产物全文扫描 | 无 `password:`/`token:`/`api_key:`/`secret:`（ADR-02） |
| `test_import_without_pythonpath_hack` | import 检查 | CLI 顶部无 `importlib.util`（S1 P2-4） |

### 集成测试

- S10：`topo-plan plan --topo-id node2_dut1_tg1_link3 --limit ... --dry-run` 全链路（fixture 池）→ env-file 文本 → `case_runner load_env_file` 校验。
- S10：`--execute --authorized` 落盘 `topology/topology.yaml` → S8 编排 [2] 消费链路；`--execute` 无 `--authorized` → exit 3。
- S10：dry-run + `--output runs/<run-id>/topology.preview.yaml` → 下游 `load_env_file` 解析预览副本（S8-B 全链路）。
- S10：参考用例 `IPv4策略路由`（TOPO-01）`--case-file` 提取 → 映射 → env-file → dry-run 执行链路打通（HLD §11 层 A 第 8 条）。

### Fixture 需求

- 复用 P-1 fixture 池（`hg3250-51/52` + TE 端口）+ S4 fixture（ledger 占用）。
- 新增 `tests/fixtures/case_topology_constraint.md`（组网约束提取）+ `case_topology_description.md`（组网描述回退）+ `case_no_topo_id.md`（提取失败）。
- 新增 `tests/fixtures/limit_te.json` / `limit_invalid_media.json`（limit 输入样例）。

---

## 11. 实施步骤

| TASK-ID | 描述 | 预估 | 产物 |
|---------|------|:---:|------|
| TASK-S7-01 | `topo_plan_cli.py` 骨架 + argparse 参数表（含 `--authorized`）+ 退出码约定 + `RuntimeAuthorizationError` 门 | 1.5h | CLI 骨架 |
| TASK-S7-02 | `_extract_topo_id_from_case` 双源提取（E2）+ frontmatter 解析 | 1.5h | 提取函数 |
| TASK-S7-03 | `_resolve_logical_topo`（S1 find_topology_file + parser 接线） | 1h | topo 解析 |
| TASK-S7-04 | `_resolve_limit`（--limit/--limit-file 归一化 + S2 parse_limit 接线） | 1h | limit 解析 |
| TASK-S7-05 | `_build_physical_view`（S3 merge_pool + PhysicalPool 接线） | 1h | 物理视图 |
| TASK-S7-06 | `_run_mapping`（S4 TopoMapper.match 接线 + env_name/dry_run 分支） | 1.5h | 映射编排 |
| TASK-S7-07 | `_emit_success` / `_emit_failure`（S5 export_env_file / build_failure_report / write_report + S6 compute_ip_plan 接线） | 1.5h | 结果输出 |
| TASK-S7-08 | `_show_pool_summary` / `_show_mapping_summary` / `_show_next_step` 向导摘要 | 1h | 向导 5 步 |
| TASK-S7-09 | `run_plan` 主流程串联（7.1 流程图） | 1.5h | 主入口 |
| TASK-S7-10 | `SKILL.md` 编写（Prompt-Skill 契约结构） | 2h | skill 编排文档 |
| TASK-S7-11 | `templates/physical_pool.yaml.example` + `physical_ledger.yaml.example`（不含造数） | 1h | 模板参考 |
| TASK-S7-12 | 单元测试（19 用例 + 3 fixture 文件） | 2.5h | test_topo_plan_cli.py |
| **合计** | | **~19.5h** | |

---

## 12. 风险、难点与预研建议

| 风险 | 级别 | 缓解 |
|------|:---:|------|
| PYTHONPATH 未注入时 CLI import 失败 | 中 | S8 install 统一注入；CLI 顶部 sys.path 过渡降级（S1 P2-4）；测试在模拟环境验证 |
| `--case-file` frontmatter 解析与 case-execution 语义不一致 | 中 | 复用 case-execution `parse_frontmatter` 的字段语义（16 列）；S10 用 fixture case 校验双源提取 |
| agent 生成的 limit JSON 与 S2 parse_limit 预期不符 | 中 | S2 已做 schema 前置校验（PARAM_INVALID）；S7 CLI 透传错误 + 提示参考 limit-syntax.md |
| dry-run 与 execute 行为差异导致用户误解 | 低 | SKILL.md dry-run/execute 对比表 + CLI `--help` 显式说明默认 dry-run |
| 向导 5 步输出冗长影响可读性 | 低 | 每步输出压缩为人读摘要（设备数/端口数/约束数），详细日志走 `--verbose`（可选） |
| S5/S6 接口变更（export_env_file 签名） | 低 | S7 只调 S5/S6 公开函数，接口契约已固定（CP5 确认） |

**预研建议**：
1. 用 fixture 池跑一次 `topo-plan plan --topo-id node2_dut1_tg1_link3 --dry-run`，确认向导 5 步输出可读性与 env-file 文本正确。
2. 与 S8 对齐 CLI 调用参数（ptm-te 编排层传 `--case-file` + `--env-name` + `--output topology/topology.yaml` + execute/dry-run 模式映射）。

---

## 13. 实现灰区与取舍记录

### 灰区 1：`--topo-id` vs `--case-file` 优先级

**问题**：两个参数同时提供时以谁为准？

**选项**：
- A. `--topo-id` 显式优先；`--case-file` 仅当 `--topo-id` 缺省时用于提取。
- B. 同时提供时报参数冲突错误（强制二选一）。
- C. `--case-file` 优先（用例是真相源）。

**决策**：选 A。`--topo-id` 是显式确定性输入，`--case-file` 是便捷提取入口；显式优先避免用户明确指定被用例文件意外覆盖。`--case-file` 提取失败但 `--topo-id` 已给时仍可执行。

**证据**：S8 ptm-te 编排总是传 `--case-file`（从用例解析）；S7 设计允许 `--topo-id` 覆盖以便调试单拓扑。

### 灰区 2：dry-run 是否写台账占用

**问题**：ledger 占用闭环（H1）要求映射成功写回，但 dry-run 默认门要求无副作用。dry-run 时是否写台账？

**选项**：
- A. dry-run 不写台账（`match` 传 `env_name=None`），打印 would-allocate 清单供预览。
- B. dry-run 也写台账（保证映射占用真实可见），execute 才释放。
- C. dry-run 只读台账不写，execute 写回。

**决策**：选 A + C 结合（A 即 C 的 dry-run 面）。dry-run 是纯校验模式，写台账会污染共享资源（目标项目物理池台账是用户真实数据）；execute 才写回。would-allocate 清单满足用户预览需求，零副作用（FR7/NFR1）。

### 灰区 3：NLP→CLI 转换归属（G5 / DQ-037-06）

**问题**：用户输入是自然语言（"海光3250上执行，需要 TE|10GE 链路"），CLI 只收结构化参数。谁做 NLP→结构化转换？

**选项**：
- A. agent（ptm-te 或人工）依据 S2 `limit-syntax.md` 转译 → 结构化 limit → `--limit`/`--limit-file`。
- B. S7 CLI 内置 NLP 解析器。
- C. ptm-te 编排层内置转换函数。

**决策**：选 A。HLD §4.7 / DQ-037-06 明确「NLP→CLI 转换由 agent 实现，CLI 只收结构化参数」；S2 已交付 `limit-syntax.md` 作为 agent 转译依据（G5 显式交付物）。S7 CLI 不引入 NLP 解析，保持接口稳定可测。

### 灰区 4：env-file 落盘时机（dry-run stdout vs execute 落盘）

**问题**：S5 的 `--output` 默认 `topology/topology.yaml`。S7 CLI 何时落盘？

**选项**：
- A. dry-run：env-file 到 stdout 不落盘；execute：落 `--output`（缺省 `topology/topology.yaml`）。
- B. dry-run 也落盘到 `--output`（便于人工检查文件）。
- C. 始终落盘，dry-run 落临时目录。

**决策**：选 A。dry-run 零副作用原则（不落盘避免污染目标项目目录）；execute 落固定消费路径 `topology/topology.yaml`（对齐 case-execution 统一环境文件路径，CR-034 缺口 2）。用户需在 execute 前人工审查 stdout 的 env-file 文本。

### 灰区 5：模板副本源（S7 templates vs topo-config config）

**问题**：S7 产出 `templates/*.example`，P-1 已模板化 `skills/topo-config/config/`。S8 install 生成目标项目模板副本的源用哪个？

**选项**：
- A. S8 以 `skills/topo-config/config/physical_pool.yaml` + `physical_ledger.yaml` 为源（P-1 模板化后，单一权威模板）。
- B. S8 以 S7 `templates/*.example` 为源。
- C. 双源并存，install 时校验一致性。

**决策**：选 A + 说明。P-1 已把 `skills/topo-config/config/` 模板化（占位符 + 指引），是引擎直接消费的模板真相源；S7 `templates/*.example` 作为人工参考副本与 S10 校验基准。S8 生成模板副本读 `skills/topo-config/config/`，避免两套模板漂移。S7 templates 在 SKILL.md 中说明其参考定位。

### 灰区 6：`--execute` 授权门与 DQ-037-04 分层（S7-B，CP5 修订新增）

**问题**：`--execute` 的「写台账 + 落盘 env-file」与 S9 真机下发都是"写"，是否共享同一授权档？

**选项**：
- A. `--execute` 必须配 `--authorized` 显式门（workspace 写 = 执行前自动步骤）；S9 真机下发另走 DQ-037-04 独立 runtime_authorization。两级分层。
- B. `--execute` 与真机下发共用同一 DQ-037-04 授权档（一档覆盖全部写）。
- C. `--execute` 不需要显式门（默认允许 workspace 写）。

**决策**：选 A。映射/台账占用与 env-file 落盘是**目标项目 workspace 写**（执行前自动步骤，不触碰真实设备），需要显式 `--authorized` 确认但不与真机下发混档；S9 `deploy_bridge` 真机下发（交换机/防火墙配置）才是 DQ-037-04 独立 runtime_authorization（S8-E 对齐）。CLI 在 `--execute` 无 `--authorized` 时抛 `RuntimeAuthorizationError`（exit 3），杜绝把 workspace 写静默当作运行授权。

**证据**：CR-037 DQ-037-04（`runtime_authorization` 独立授权决策项，不在 CP 批准隐含）；S8-E 授权分层（映射/台账占用不需真机下发授权）；S9 §权限变化（S9 侧 DQ-037-04）。

### 灰区 7：dry-run 是否允许落预览 env-file（S8-B 联动，CP5 修订新增）

**问题**：dry-run 默认门要求"不落盘"，但 S8 编排 [2]/[4] 必须解析 `${ENV.*}`（case-execution `load_env_file`），连 dry-run 链路都走不通。dry-run 是否落盘？

**选项**：
- A. dry-run 默认 env-file 只到 stdout；当 `--output` 指向 `runs/<run-id>/` 时落**预览副本**（`metadata.executable:false`），不写真实 `topology/` 路径。
- B. dry-run 一律不落盘，S8 dry-run 链路由 S5 直接内存传递（绕过文件）。
- C. dry-run 落盘到真实 `topology/topology.yaml`（与 execute 同路径）。

**决策**：选 A。dry-run 落预览副本到 run 目录（运行态，不入库、不碰真实路径），标注 `executable:false` 使下游 load_env_file 可解析验证但任何执行消费必须拒绝；既保持 dry-run 零副作用承诺，又保证 dry-run 全链路可走（S8-B BLOCKER 修复）。B 方案破坏下游消费（case_runner 按文件路径加载）；C 方案污染真实环境文件路径。

---

## 14. 回滚与发布策略

### 回滚

- S7 全部为新增文件（SKILL.md / topo_plan_cli.py / templates/*.example），回滚即删除。
- 无对既有模块的修改；topo-config / case-execution / S1-S6 产物不受影响。
- 若 S8 已安装 `topo-planning` skill，回滚需同时卸载该 skill（`install.py uninstall`）。

### 发布

- S7 随 `topo-planning` skill 由 S8 install 统一安装（`PTM_TE_SKILLS` 增加后随 `install --agent ptm-te` 复制）。
- S7 CLI 依赖 S1-S6 模块已实现；S10 全链路验证后随 CR-037 交付。
- 依赖顺序：S7 = [S1, S2, S3, S5, S6] precedence（DEVELOPMENT-PLAN）；S8 依赖 S7（CLI 是 ptm-te 编排的调用入口）。

---

## 15. Definition of Done（DoD）

- [ ] `topo-plan plan` 支持 `--topo-id` / `--case-file` 双入口，`--case-file` 双源提取 topo_id（组网约束优先/组网描述回退，E2，FR1）
- [ ] `--limit` / `--limit-file` 归一化 + S2 `parse_limit` 接线，空 limit 按默认行为（FR2）
- [ ] `--devices-yaml` / `--pool-path` / `--ledger-path` 三路径 + 缺省回落 `topology/`（FR3）
- [ ] 向导 5 步输出 + 每步「下一步提示」（FR4）
- [ ] 映射成功：dry-run stdout 展示 env-file / execute 落盘 `--output`（FR5，幂等原子写）
- [ ] 映射失败：结构化报告 + `--report-out` JSON + 人读 / stdout，退出码 1（FR6）
- [ ] 默认 `--dry-run`：不写台账、不落真实 `topology/` 路径、不部署，打印 would-allocate 清单（FR7/NFR1）
- [ ] `--execute` + `--authorized`：写台账占用（env_name）、落盘 env-file（FR8，workspace 写执行前自动步骤）；真机部署由 S9 承接（DQ-037-04）
- [ ] `--execute` 无 `--authorized` → 抛 `RuntimeAuthorizationError`（exit 3，FR12/NFR7）；`--authorized` 无 `--execute` → exit 2
- [ ] dry-run + `--output` 指向 run 目录 → 落预览副本 `metadata.executable:false`（S8-B 联动，0 次真机写）
- [ ] `--env-name` 显式 / 自动生成唯一值（FR9）
- [ ] `--list-topologies` 列出 12 个逻辑拓扑（FR10，S1）
- [ ] SKILL.md 含 Prompt-Skill 契约结构（目标/前置/命令/向导 5 步/NLP→CLI/产物契约/dry-run 门/安全/边界/Gotchas/修订记录）
- [ ] templates/*.example 含占位符、无测试造数、台账 `list + port_status`（P0-3）
- [ ] import 走统一 PYTHONPATH（S1 P2-4），无 importlib 临时代码
- [ ] 产物无凭据（ADR-02 正则断言）
- [ ] 19 个单元测试全部通过
- [ ] lld-check: LLD Structure Check OK
