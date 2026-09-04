---
doc_type: LLD
story_id: "CR037-S8"
story_slug: "ptm-te-integration"
cr_id: "CR-037"
tier: "A"
status: "draft"
created_at: "2026-08-05"
owner: "meta-dev"
depends_on: ["CR037-S7", "CR037-P1"]
dependency_type: "precedence"
source_hld: "process/changes/CR-037-HLD.md §4.8"
source_story: "process/stories/STORY-CR037-S8.md"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
scope: "agents/ptm-te.md 编排流程 [1]→[2] 集成 + script/install.py PTM_TE_SKILLS + 首次生成目标项目模板副本 + resource/component-resource-links.yaml（ptm-te→network-topology 关联）"
revision: "1.2"
revision_at: "2026-08-05"
revision_by: "meta-dev"
---

# STORY-CR037-S8-LLD：ptm-te 编排集成 + install.py 详细设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-08-05 | meta-dev | 初稿（CP5 预检通过） |
| v1.1 | 2026-08-05 | meta-dev | CP5 NEEDS_REWORK 修订：S8-A 造数隔离击穿（copy_skill_tree 裁剪 tests/、__pycache__/、*.pyc，保证「不含测试造数」成立）；S8-B dry-run 落预览 env-file 到 runs/<run-id>/（executable:false）打通下游 load_env_file 链路；S8-C component-resource-links.yaml 增 ptm-te→network-topology 关联（S1 运行时 collection.md 可及）；S8-D 模板单一真相源 = skills/topo-config/config/（P-1 后 device_type 全占位，S7 templates 仅参考）；S8-E 授权门分层重写（映射/台账占用=执行前自动步骤不需 DQ-037-04；真机下发/EnvironmentDeployer=DQ-037-04 独立授权）；S8-F .gitignore 沿用 topology/*.yaml，runs/ 运行态不需新增忽略项 |
| v1.2 | 2026-08-05 | meta-dev（CP5 第二轮复审 Minor-2 修订） | 模板安全隐性依赖显式化：frontmatter 新增 `depends_on=["CR037-S7","CR037-P1"]`（precedence），保证 P-1 模板化先于 S8 模板副本生成；§6.2 `ensure_target_project_templates` 增加 `_assert_p1_template_placeholderized` 前置校验（源含 A1600-HU/A1500-HU/A1300-HU/hg3250/HG3250 特征串即 fail-fast，防止 P-1 未落地时模板副本泄漏真实型号+密码）；Story 卡片与 DEVELOPMENT-PLAN-CR-037.yaml 依赖同步；§10 新增 `test_generate_templates_requires_p1_placeholder` 用例 |

> **Tier**: A（跨 skill 编排集成 + 安装流程修改，涉及 ptm-te agent 编排契约与安装器）  
> **Shared Fragments**: `agents/ptm-te.md`（编排流程 [1]-[8]）、`script/install.py`（PTM_TE_SKILLS / resource 安装 / copy_skill_tree / 安装目录映射）、`skills/case-execution/SKILL.md`（用例 frontmatter 组网约束/组网描述字段，E2）、`skills/topo-planning/SKILL.md`（S7，调用入口）、`skills/topo-planning/scripts/topo_plan_cli.py`（S7 CLI）、`skills/topo-config/config/physical_pool.yaml` + `physical_ledger.yaml`（P-1 模板化后，模板副本源）、`skills/topo-config/src/physical_pool.py`（S4 release 接口，P1-7）  
> **Open Items**: 无

---

## 0. 工程依据

本模块是 CR-037 的集成层，把 S1-S7 构建的 topo-planning 能力挂接进 ptm-te 现有编排流程与安装器。工程依据源自 HLD §4.8（ptm-te 编排 [1]→[2] 之间插入拓扑映射步骤，E1；topo_id 提取优先「组网约束」、回退「组网描述」括号，E2；`PTM_TE_SKILLS` 增加 `topo-config` + `topo-planning`，方案 A；install.py 首次生成目标项目模板副本，AG-7 推荐 A、幂等跳过已有），以及设计审查已关闭项 **P1-7**（`PhysicalPool.release(env_name)` 释放接口由 S4 定义，**调用挂接由 S8 在 ptm-te 编排 [7] 清理时执行**）与 **P1-8**（首次生成目标项目模板副本：模板源路径为层1 skill 内 config/，`topology/` 目录创建，幂等跳过已有，`.gitignore` 忽略沿用 `topology/*.yaml`，不含测试造数，台账模板 schema 与 P0-3 决策一致 `list+port_status`）。核心价值：让 ptm-te 在「用例解析 → 设备准备」之间自动完成逻辑 topo → 物理 topo 映射并产出 env-file，同时保证安装后目标项目立即可用的模板骨架，并在用例清理时释放台账占用，形成「映射 → 占用 → 释放」闭环。

---

## 1. 目标

修改 `agents/ptm-te.md` 与 `script/install.py`，实现：

1. **topo_id 双源提取（E2）**：从用例 frontmatter「组网约束」字段优先提取 topo_id，回退「组网描述」括号内提取；提取失败时按 `TOPO_ID_NOT_FOUND` 终止（结构化错误）。
2. **编排流程 [1]→[2] 之间插入拓扑映射步骤**（E1）：新步骤「[1.5] 拓扑映射」，调用 topo-planning skill（S7 CLI `topo-plan`）将逻辑 topo + limit → 物理 topo 映射 → env-file 产物；映射失败输出结构化报告，不静默继续。
3. **`PTM_TE_SKILLS` 增加 `topo-config` + `topo-planning`**（方案 A）：ptm-te 安装时自动携带两个 skill，保证 CLI/引擎 import 路径可及。
4. **首次生成目标项目模板副本（P1-8）**：install.py 在安装 ptm-te 时检查并生成目标项目 `topology/physical_pool.yaml` + `physical_ledger.yaml` 模板副本（幂等跳过已有，不含测试造数，`.gitignore` 忽略沿用 `topology/*.yaml`）。
5. **P1-7 释放挂接**：ptm-te 编排流程 [7] 用例清理（teardown/环境删除）时调用 `PhysicalPool.release(env_name)` + `save_ledger`，释放台账占用，避免只写不释放。

核心价值：ptm-te 从「人工维护 `topology/topology.yaml`」升级为「按用例自动映射 + 模板化初始化 + 占用闭环释放」，与 case-execution `${ENV.*}` 消费契约无缝衔接。

---

## 2. Requirements

### Functional

| ID | 需求 | 优先级 |
|----|------|:------:|
| FR1 | topo_id 双源提取：`组网约束` 字段原样采用；回退 `组网描述` 括号内提取（`node\d+_.+` 模式，E2）；两字段均失败 → `TOPO_ID_NOT_FOUND` | P0 |
| FR2 | 编排流程 [1]→[2] 之间插入「[1.5] 拓扑映射」步骤：调用 S7 `topo-plan` CLI（`--case-file` + limit + 物理路径 + `--env-name`） | P0 |
| FR3 | 拓扑映射成功：execute 模式 env-file 落盘到 `topology/topology.yaml`（S7 `--execute --authorized`），供编排 [2] 设备准备与后续 `${ENV.*}` 消费；**dry-run 模式 env-file 落预览副本到 `runs/<run-id>/topology.preview.yaml`（`metadata.executable:false`）**，供 [2]/[4] 解析 `${ENV.*}` 验证链路，不写真实 `topology/` 路径（S8-B） | P0 |
| FR4 | 拓扑映射失败：终止该用例执行，`error_type=TOPO_MAPPING_FAILED`，输出结构化报告（缺设备/缺 TE 端口/链路约束不满足），提示用户补物理资源或改 limit | P0 |
| FR5 | 编排 [7] 用例清理阶段挂接 `PhysicalPool.release(env_name)` + `save_ledger`，释放本次 run 的台账占用（P1-7） | P0 |
| FR6 | `PTM_TE_SKILLS` 增加 `topo-config` + `topo-planning`（方案 A），`install --agent ptm-te` 时一并安装 | P0 |
| FR7 | install 首次生成目标项目模板副本：`topology/physical_pool.yaml` + `physical_ledger.yaml`，源 = `skills/topo-config/config/` 模板（P-1 模板化后，`device_type` 全占位，不含测试造数）；**copy_skill_tree 复制 skill 时裁剪 `tests/`、`__pycache__/`、`*.pyc`，保证「不含测试造数」承诺真实成立（S8-A）** | P0 |
| FR8 | 模板副本幂等：目标文件已存在 → 跳过不覆盖（保护用户已填数据，AG-7）；`topology/` 目录不存在时创建 | P0 |
| FR9 | `.gitignore` 忽略：install 时检查目标项目 `.gitignore`，无 `topology/*.yaml`（或 `topology/`）规则则追加（沿用既有模式，不覆盖用户已有规则） | P1 |
| FR10 | 台账模板 schema 与 P0-3 决策一致：`devices` 为 `list`，每条含 `port_status[].{allocated_to, allocated_vlan}`；不含测试造数（hg3250 等仅存于 ptm-team tests/fixtures） | P0 |
| FR11 | dry-run 模式：install `--dry-run` 时打印模板副本与 `.gitignore` 操作，不写文件 | P1 |
| FR12 | layer2 共享资源安装（S8-C）：`component-resource-links.yaml` 为 `ptm-te` 增加 `network-topology` 关联（`tgfw-topo-collection`，install_policy=required），`install --agent ptm-te` 时自动安装 `topology-collection.md` + `index.yaml` 到 `~/.ptm-team/resource/network-topology/`，保证 S1 `find_topology_file` 运行时 collection.md 可及 | P0 |

### Non-Functional

| ID | 需求 |
|----|------|
| NFR1 | 安装幂等：重复 `install --agent ptm-te` 不覆盖模板副本、不重复追加 `.gitignore` 规则 |
| NFR2 | 凭据安全：模板副本用 `${PLACEHOLDER}` 占位，不含真实密码/IP；install 不写任何凭据 |
| NFR3 | 向后兼容：既有 PTM_TE_SKILLS 5 个 skill 安装行为不变；模板副本仅在首次（目标缺失时）生成 |
| NFR4 | 编排解耦：ptm-te agent md 只声明契约（何时调用 topo-planning、何时 release），不内嵌 CLI 实现；CLI 细节由 S7 承载 |
| NFR5 | 释放幂等：`release(env_name)` 对不存在/已释放的 env 返回 0 释放端口，不报错（S4 语义） |
| NFR6 | 模板副本内容与 `skills/topo-config/config/` 模板一致（单一真相源），S10 校验无漂移 |
| NFR7 | copy_skill_tree 裁剪 `tests/`、`__pycache__/`、`*.pyc` 不破坏 skill 运行时（src 与 scripts 不受影响）；裁剪对 ptm-tde / ptm-tse 既有 skill 同样生效且无副作用 |

---

## 3. 模块拆分与职责

### 3.1 本 Story 修改文件

| 文件 | 操作 | 职责 |
|------|------|------|
| `agents/ptm-te.md` | **修改** | 编排流程 [1]→[2] 插入「[1.5] 拓扑映射」步骤；新增 topo_id 双源提取说明；[7] 清理增加 release 挂接；关联 Skill 表 + 修订记录 |
| `script/install.py` | **修改** | `PTM_TE_SKILLS` 增加 `topo-config` + `topo-planning`；`copy_skill_tree` 裁剪 `tests/`、`__pycache__/`、`*.pyc`（S8-A）；新增 `ensure_target_project_templates` 函数（P1-8）；`.gitignore` 检查/追加 |
| `resource/component-resource-links.yaml` | **修改** | 为 `ptm-te` 增加 `network-topology` 关联（`tgfw-topo-collection`，required，S8-C/FR12） |

### 3.2 函数/章节划分

| 对象 | 类型 | 职责 |
|------|------|------|
| `PTM_TE_SKILLS` | 常量（改） | 5 个 → 7 个：增加 `topo-config` + `topo-planning`（方案 A，FR6） |
| `copy_skill_tree(src_dir, dest_dir, dry_run)` | 函数（修改） | 复制 skill 时**裁剪 `tests/`、`__pycache__/`、`*.pyc`**（S8-A，保证安装产物不含测试造数/字节码缓存） |
| `ensure_target_project_templates(workspace_root, source_dir, dry_run)` | 函数（新增） | 生成 `topology/` 模板副本 + `.gitignore` 检查/追加（FR7-FR11，P1-8） |
| `resource/component-resource-links.yaml` | 资源元数据（改） | ptm-te 组件 → network-topology `tgfw-topo-collection`（required，FR12） |
| ptm-te.md「[1.5] 拓扑映射」步骤 | 章节（新增） | 编排契约：topo_id 双源提取 + 调 S7 CLI + 成功/失败分支（FR1-FR4，E1/E2） |
| ptm-te.md「[7] 用例清理」release 挂接 | 章节（修改） | 清理阶段调 `release(env_name)` + `save_ledger`（FR5，P1-7） |

---

## 4. 代码结构与文件影响范围

### 修改文件（S8 写入范围）

```
agents/ptm-te.md                       # 编排流程 [1.5] 拓扑映射 + [7] release 挂接 + 关联 Skill + 修订记录
script/install.py                      # PTM_TE_SKILLS + copy_skill_tree 裁剪（S8-A）+ ensure_target_project_templates + .gitignore
resource/component-resource-links.yaml # ptm-te → network-topology 关联（S8-C/FR12）
```

### 只读消费（不修改）

```
skills/topo-planning/SKILL.md                   # S7 调用入口（CLI 契约）
skills/topo-planning/scripts/topo_plan_cli.py   # S7 CLI（ptm-te 编排调用）
skills/topo-config/config/physical_pool.yaml    # P-1 模板化后，模板副本源
skills/topo-config/config/physical_ledger.yaml  # P-1 模板化后，台账模板（list + port_status）
skills/topo-config/src/physical_pool.py         # S4 release(env_name) + save_ledger
skills/case-execution/SKILL.md                  # frontmatter 组网约束/组网描述字段语义（E2）
skills/case-execution/templates/topology.yaml.example  # env-file 目标路径契约（topology/topology.yaml）
```

### 不修改

```
skills/topo-planning/**   # S7 产物（ptm-te 只调用，不修改）
skills/topo-config/**     # 引擎（S4 已在其写入范围完成 release 接口）
```

### 文件所有权与并行安全

- `agents/ptm-te.md`：S8 primary（`ptm-te.md#编排流程 [1]→[2]`），无其他 Story 占用。
- `script/install.py`：S8 primary（`install.py#PTM_TE_SKILLS` + `copy_skill_tree` 裁剪 + 模板副本函数），无其他 Story 占用。
- `resource/component-resource-links.yaml`：S8 primary（`ptm-te` 组件关联新增，S8-C）；该文件既有 `ptm-tde` 组件条目不动，`ptm-te` 条目为新增（无其他 Story 占用）。
- Wave 3 串行（S7 → S8 → S9），CP4 已确认无文件冲突。

---

## 5. 数据模型与持久化设计

### 5.1 topo_id 提取（E2）

```python
# ptm-te 编排 [1.5] 内联逻辑（agent md 契约层声明，不实现代码）
# 输入：case frontmatter（case-execution 16 列解析）
# 提取优先级：
#   1. 组网约束: 值匹配 node\d+_.+ → 原样采用（source='组网约束'）
#   2. 组网描述: 括号内提取 node\d+_.+（如 'TOPO-01 基础三链路 (node2_dut1_tg1_link3)'）
#   3. 两字段均失败 → TOPO_ID_NOT_FOUND，终止
# 产物：topo_id 传入 S7 CLI --case-file 提取逻辑（S7 复用同一语义，S8 契约引用）
```

### 5.2 编排 [1.5] 拓扑映射数据流

```
用例 frontmatter（组网约束/组网描述）
   │  E2 双源提取
   ▼
topo_id ──► S7 topo-plan CLI: plan --case-file <case> --limit <limit> \
             --devices-yaml topology/devices.yaml --pool-path topology/physical_pool.yaml \
             --ledger-path topology/physical_ledger.yaml --env-name <env_name>
   │
   ├─ execute 模式（--execute --authorized）
   │      └─ --output topology/topology.yaml
   │           ├─ env-file 落盘 topology/topology.yaml → 编排 [2] 设备准备消费 ${ENV.*}
   │           └─ S4 已写台账占用（allocated_to=env_name；workspace 写，执行前自动步骤，不需 DQ-037-04）
   ├─ dry-run 模式（--dry-run，默认）
   │      └─ --output runs/<run-id>/topology.preview.yaml
   │           ├─ 落预览副本（metadata.executable:false）→ [2]/[4] 解析 ${ENV.*} 验证链路（S8-B）
   │           ├─ 不写真实 topology/ 路径、不写台账占用
   │           └─ 0 次真机写
   └─ 失败 → 结构化报告（--report-out）→ error_type=TOPO_MAPPING_FAILED → 终止该用例
```

### 5.3 模板副本（P1-8，S8-D 单一真相源）

```yaml
# 源（P-1 模板化后，单一真相源）：
#   skills/topo-config/config/physical_pool.yaml     # 占位符 + 注释指引，P-1 修订后 device_type 全占位（${DEVICE_TYPE}），无测试造数
#   skills/topo-config/config/physical_ledger.yaml   # devices list + port_status（P0-3）
# 目标：
#   <workspace>/topology/physical_pool.yaml
#   <workspace>/topology/physical_ledger.yaml
# 幂等：目标存在 → 跳过（打印 "已存在，跳过"）
# .gitignore：追加 topology/*.yaml（若缺失）
# 模板副本源唯一 = skills/topo-config/config/；S7 skills/topo-planning/templates/*.example 仅人工参考，
# install 不读取、不复制（S8-D 防双套模板漂移）。
```

### 5.4 持久化

- **模板副本**：install 首次生成（目标缺失时）；用户后续维护真实数据。
- **台账占用**：S4 `_commit_allocation` 写回（execute 模式）→ [7] `release(env_name)` + `save_ledger` 释放。
- **env-file**：S7 CLI execute 模式落盘 `topology/topology.yaml`；dry-run 模式落预览副本 `runs/<run-id>/topology.preview.yaml`（`metadata.executable:false`，S8-B，运行态不入库）。
- **layer2 共享资源**：`install --agent ptm-te` 经 `get_component_resources` 将 `network-topology` 的 `index.yaml` + `topology-collection.md` 安装到 `~/.ptm-team/resource/network-topology/`（FR12，S8-C）。
- 无其他持久化。

---

## 6. API / Interface 设计

### 6.1 `PTM_TE_SKILLS`（修改）

```python
# 修改前（5 个）
PTM_TE_SKILLS = [
    "device-management", "device-connection", "policy-route-execution",
    "trex-traffic", "case-execution",
]

# 修改后（7 个，方案 A，FR6）
PTM_TE_SKILLS = [
    "device-management", "device-connection", "policy-route-execution",
    "trex-traffic", "case-execution",
    "topo-config", "topo-planning",          # CR-037 S8 增加
]
```

安装行为：`install_agent` 遍历 `get_agent_skills("ptm-te")` → `copy_skill_tree` 复制两个 skill 到平台 skill 目录；`uninstall_agent` 逆序移除。既有 5 个 skill 行为不变（NFR3）。

### 6.1b `copy_skill_tree` 裁剪语义（S8-A，BLOCKER 修复）

`copy_skill_tree(src_dir, dest_dir, dry_run)` 从**整树 `shutil.copytree`** 改为**过滤复制**：排除 `tests/`、`__pycache__/`、`*.pyc`、`*.pyo`、`.gitignore` 运行态产物，保证目标项目 `.claude/skills/` 或 `.agents/skills/` 内不携带任何测试造数/字节码缓存。

```python
def copy_skill_tree(src_dir: Path, dest_dir: Path, dry_run: bool) -> None:
    """复制 skill 目录树（过滤运行态/测试资产，S8-A）。
    排除：tests/ 目录、__pycache__/ 目录、*.pyc、*.pyo、*.pyc 缓存。
    保留：SKILL.md、src/**、scripts/**、config/**、templates/**、docs/**、*.md、*.yaml、*.json。
    依赖：安装后 skill 运行时（src/scripts）不依赖 tests/，裁剪无副作用（NFR7）。
    """
    if dry_run:
        print(f"  [DryRun] 复制 skill: {src_dir.name} -> {dest_dir}（排除 tests/__pycache__/*.pyc）")
        return
    if dest_dir.exists():
        shutil.rmtree(dest_dir)
    ignore = shutil.ignore_patterns("tests", "__pycache__", "*.pyc", "*.pyo")
    shutil.copytree(src_dir, dest_dir, ignore=ignore)
    print(f"  ✓ 安装 skill: {src_dir.name}（已排除测试/缓存资产）")
```

> **隔离承诺成立依据**：即使源 skill 目录含 `tests/fixtures/*.yaml`（P-1 造数），install 复制时被 `ignore_patterns("tests", ...)` 排除，目标项目安装产物**不含测试造数**；模板副本源 `skills/topo-config/config/` 本身是 P-1 模板（占位符），双保险保证「不含测试造数」承诺真实成立（S8-A + HLD §8 隔离）。

### 6.2 `ensure_target_project_templates(workspace_root, source_dir, dry_run) -> list[str]`

新增函数（install.py）。**前置校验（Minor-2）**：模板副本源必须是 P-1 占位化后的模板；若源仍含具体型号/真实密码，说明 CR037-P1 未实现，立即 fail-fast 阻断，不得复制真实数据。

```python
# 具体型号特征串黑名单（case-sensitive，Minor-2/Minor-4 对齐 S10 精确断言）
_MODEL_FEATURE_TOKENS = ("A1600-HU", "A1500-HU", "A1300-HU", "hg3250", "HG3250")

def _assert_p1_template_placeholderized(source_dir: Path) -> None:
    """P-1 模板占位化前置校验（Minor-2）。

    依赖声明：S8 `depends_on = [CR037-S7, CR037-P1]`（precedence）——CR037-P1 必须先于
    S8 实现，把 skills/topo-config/config/{physical_pool,physical_ledger}.yaml 占位化
    （device_type=${DEVICE_TYPE}、hardware_platform=${HARDWARE_PLATFORM}、
    password=${PASSWORD_PLACEHOLDER}）。若源模板仍含具体型号特征串或真实密码值，
    说明 P-1 未落地；此时生成模板副本会泄漏真实数据（A1600-HU + 密码），必须 fail-fast。
    dry-run 同样执行（占位化是硬前提，与是否写文件无关）。
    """
    for filename in ("physical_pool.yaml", "physical_ledger.yaml"):
        src = source_dir / "skills/topo-config/config" / filename
        if not src.is_file():
            raise FileNotFoundError(f"P-1 模板源缺失: {src}（请先完成 CR037-P1 physical-pool-template）")
        text = src.read_text(encoding="utf-8")
        for token in _MODEL_FEATURE_TOKENS:
            if token in text:
                raise RuntimeError(
                    f"P-1 模板未占位化: {src} 仍含具体型号特征串 '{token}'。"
                    f"CR037-P1 必须先实现，将 config/*.yaml 占位化后 S8 才能生成模板副本。"
                )

def ensure_target_project_templates(workspace_root: Path, source_dir: Path, dry_run: bool) -> list[str]:
    """首次生成目标项目 topology/ 模板副本（P1-8）。
    源 = skills/topo-config/config/{physical_pool,physical_ledger}.yaml（P-1 模板化后，不含测试造数）。
    前置校验：_assert_p1_template_placeholderized（Minor-2，P-1 占位化硬前提）。
    幂等：目标文件已存在 → 跳过。.gitignore 追加 topology/*.yaml（缺失时）。
    返回已写入/已跳过文件名清单。
    """
    _assert_p1_template_placeholderized(source_dir)   # Minor-2：P-1 占位化前置校验（fail-fast）
    actions: list[str] = []
    topology_dir = workspace_root / "topology"
    source_files = [
        ("physical_pool.yaml", source_dir / "skills/topo-config/config/physical_pool.yaml"),
        ("physical_ledger.yaml", source_dir / "skills/topo-config/config/physical_ledger.yaml"),
    ]

    for filename, src in source_files:
        dest = topology_dir / filename
        if dest.exists():
            actions.append(f"skip (exists): topology/{filename}")
            continue
        if dry_run:
            print(f"  [DryRun] 生成模板副本: {dest}")
            actions.append(f"dry-run: topology/{filename}")
            continue
        ensure_directory(topology_dir)
        shutil.copy2(src, dest)
        print(f"  ✓ 生成模板副本: {dest}")
        actions.append(f"created: topology/{filename}")

    _ensure_gitignore_rule(workspace_root, dry_run, actions)
    return actions
```

### 6.3 `_ensure_gitignore_rule(workspace_root, dry_run, actions)`

```python
def _ensure_gitignore_rule(workspace_root: Path, dry_run: bool, actions: list[str]) -> None:
    """检查目标项目 .gitignore 是否含 topology/*.yaml 规则，缺失则追加（FR9/NFR1）。"""
    gitignore = workspace_root / ".gitignore"
    rule = "topology/*.yaml"
    existing = gitignore.read_text(encoding="utf-8") if gitignore.is_file() else ""
    if rule in existing:
        actions.append("gitignore: already has topology/*.yaml")
        return
    if dry_run:
        print(f"  [DryRun] 追加 .gitignore 规则: {rule}")
        actions.append("gitignore: dry-run append")
        return
    prefix = existing.rstrip()
    new_content = f"{prefix}\n{rule}\n" if prefix else f"{rule}\n"
    gitignore.write_text(new_content, encoding="utf-8")
    print(f"  ✓ 追加 .gitignore 规则: {rule}")
    actions.append("gitignore: appended")
```

### 6.4 ptm-te 编排「[1.5] 拓扑映射」步骤契约（agent md 声明）

| 维度 | 契约 |
|------|------|
| 触发时机 | 编排流程 [1] 用例解析后、[2] 设备准备前（E1） |
| 输入 | case frontmatter（组网约束/组网描述）+ 用户 limit（NLP → agent → 结构化）+ 物理路径（`topology/{physical_pool,physical_ledger,devices}.yaml`） |
| 动作 | execute 模式调用 `topo_plan_cli.py plan --case-file <case> --limit <json> --devices-yaml ... --pool-path ... --ledger-path ... --env-name <env_name> --output topology/topology.yaml --execute --authorized`；dry-run 模式调用 `... --output runs/<run-id>/topology.preview.yaml --dry-run` |
| 输出 | execute：env-file `topology/topology.yaml` + 台账占用写回（allocated_to=env_name）；dry-run：预览副本 `runs/<run-id>/topology.preview.yaml`（`metadata.executable:false`），供 [2]/[4] 解析 `${ENV.*}` 验证链路（S8-B） |
| 失败行为 | `TOPO_MAPPING_FAILED`，输出结构化报告，终止该用例 |
| 与 dry-run 的关系 | ptm-te 默认 dry-run 门：dry-run 时 `--dry-run`（不写台账/不落真实路径，落预览副本）；execute 时 `--execute --authorized`（写台账/落正式 env-file） |
| **授权分层（S8-E）** | 映射 + 台账占用 + env-file 落盘 = **workspace 写，执行前自动步骤**（需 `--authorized` 显式门确认，但**不需 DQ-037-04 真机下发授权**）；真机下发/EnvironmentDeployer（S9）才需 DQ-037-04 独立 runtime_authorization |

### 6.5 ptm-te 编排 [7] release 挂接契约（P1-7）

| 维度 | 契约 |
|------|------|
| 触发时机 | 编排流程 [7] 用例清理（teardown/环境删除） |
| 前置 | 本次 run 使用过 topo-planning（有 `env_name`，execute 模式） |
| 动作 | `pool.release(env_name)` + `pool.save_ledger(ledger_path)` |
| 语义 | 释放该 env_name 全部端口占用（`allocated_to`/`allocated_vlan` 置空），设备态派生 free（S4 §6.4） |
| 幂等 | env 不存在/已释放 → 返回 0 释放端口，不报错（NFR5） |
| 失败行为 | release 失败记 `cleanup_summary`，不阻断 [8] report（与既有清理错误处理一致） |

### 6.6 `component-resource-links.yaml` ptm-te → network-topology 关联（S8-C/FR12）

`resource/component-resource-links.yaml` 新增 `ptm-te` 组件条目，指向层2共享资源 `network-topology`：

```yaml
  - component_id: ptm-te
    component_type: agent
    resources:
      - resource_type: network-topology
        topo_id: tgfw-topo-collection
        install_policy: required
        reason: "CR-037 S1 find_topology_file 运行时消费 topology-collection.md（12 个标准拓扑）；ptm-te 安装时必须随带，否则 S1 运行时缺 collection.md。"
```

效果：`install_agent("ptm-te")` 已有 `get_component_resources(source_dir, "ptm-te", ...)` 逻辑读取该文件，安装时将 `resource/network-topology/{index.yaml,topology-collection.md}` 复制到 `~/.ptm-team/resource/network-topology/`（既有 network-topology 安装分支，`field in ("source","spec","collection")` 覆盖 `collection` 字段）。`ptm-tde` 组件既有条目不动（向后兼容，NFR3）。S10 集成测试校验 `~/.ptm-team/resource/network-topology/topology-collection.md` 存在且 `index.yaml` 含 `tgfw-topo-collection`。

---

## 7. 核心处理流程

### 7.1 安装流程（install.py）

```
install --agent ptm-te
 │
 ├─ 1. install_agent（既有）：安装 ptm-te agent + rule block
 │
 ├─ 2. 安装 7 个 skills（PTM_TE_SKILLS 含新增 topo-config/topo-planning）：
 │       device-management / device-connection / policy-route-execution /
 │       trex-traffic / case-execution / topo-config / topo-planning
 │       └─ copy_skill_tree 过滤复制：排除 tests/、__pycache__/、*.pyc（S8-A）
 │
 ├─ 3. ensure_target_project_templates(workspace_root, source_dir, dry_run)
 │       ├─ topology/physical_pool.yaml（目标存在 → 跳过；否则复制 skills/topo-config/config/，P-1 后 device_type 全占位）
 │       ├─ topology/physical_ledger.yaml（同上，list + port_status）
 │       └─ .gitignore 追加 topology/*.yaml（缺失时）
 │
 └─ 4. 关联 resources：get_component_resources("ptm-te") 经 component-resource-links.yaml
       安装 network-topology（tgfw-topo-collection → topology-collection.md + index.yaml 到 ~/.ptm-team/resource/network-topology/，S8-C）
```

### 7.2 ptm-te 编排流程（修改后）

```
用户将 PC 放入 cases/<协议族>/.../<用例编号>.md
  ↓
[1] 用例解析：--cases-dir 递归 glob，提取 case_steps + expected_result
     └─ 写入 runs/<run-id>/parse-result.json
  ↓
[1.5] 拓扑映射（CR-037 新增，E1）
     ├─ E2 双源提取 topo_id（组网约束优先 / 组网描述括号回退）
     ├─ 用户 limit（NLP → agent → 结构化 JSON）
     ├─ 调 topo-plan CLI（S7）：--case-file + --limit + 物理路径 + --env-name
     │    ├─ execute 模式（--execute --authorized，workspace 写执行前自动步骤，不需 DQ-037-04）：
     │    │     --output topology/topology.yaml → 写台账占用 + env-file 落盘
     │    └─ dry-run 模式（--dry-run，默认）：
     │          --output runs/<run-id>/topology.preview.yaml → 落预览副本（executable:false），
     │          不写真实 topology/ 路径、不写台账、0 次真机写（S8-B 下游 load_env_file 可解析）
     ├─ 成功 → 继续 [2]（env-file 或预览副本已就绪）
     └─ 失败 → error_type=TOPO_MAPPING_FAILED + 结构化报告 → 终止该用例
  ↓
[2] 设备准备：device-management 加载 devices.yaml + 型号映射查表
     └─ device-connection SSH 连接探测 + 系统快照 before（env-file 的 ${ENV.*} 已可解析）
  ↓
[3] login 一次 / [4] 逐条 op / [5] 结果判定 / [6] 执行日志
  ↓
[7] 用例清理：inverse_op 回滚
     ├─ 既有：config inverse_op / restore_snapshot / irreversible
     └─ CR-037 新增（P1-7）：release 台账占用
          pool.release(env_name) + pool.save_ledger(ledger_path)
          （仅 execute 且使用过 topo-planning 时；dry-run 跳过）
  ↓
[8] 快照 after + 结果回写（cleanup_summary 记录 release 结果）
```

### 7.3 错误路径

| 步骤 | 失败条件 | 行为 | error_type |
|------|---------|------|-----------|
| 1.5 | topo_id 双源提取失败（组网约束/组网描述均无） | 终止该用例，提示填「组网约束」或传 `--topo-id` | `TOPO_ID_NOT_FOUND` |
| 1.5 | limit 结构化解析失败（agent 转换错误） | 透传 S2 TopoLimitError（PARAM_INVALID/PLATFORM_UNKNOWN） | `PARAM_INVALID` |
| 1.5 | physical_pool/devices/ledger 缺失 | 提示运行 `install --agent ptm-te` 生成模板副本 + 填写真实数据 | `ENV_NOT_READY` |
| 1.5 | layer2 collection 资源缺失（S1 find_topology_file 找不到 collection.md） | 提示运行 `install --agent ptm-te`（S8-C 已随组件关联安装 network-topology） | `ENV_NOT_READY` |
| 1.5 | 映射失败（缺设备/缺 TE 端口/链路约束不满足） | 输出结构化报告，终止该用例 | `TOPO_MAPPING_FAILED` |
| 1.5 | topo-plan CLI 未安装（skill 缺失） | 提示安装 topo-planning skill | `OP_NOT_FOUND` |
| 7 | release 失败 | 记 `cleanup_summary` 错误，不阻断 [8] | 记录，不终止 |

---

## 8. 技术细节

### 8.1 E2 双源提取正则

```python
# ptm-te 编排 [1.5] 的提取规则（与 S7 CLI _extract_topo_id_from_case 语义一致，避免双实现漂移）
# 1. 组网约束：constraint.strip() 精确匹配 ^(node\d+_.+)$
# 2. 组网描述：re.search(r'(node\d+_[A-Za-z0-9_]+)', description)
# 优先级固定：组网约束 > 组网描述；两字段均失败 → TOPO_ID_NOT_FOUND
```

> S7 CLI 提供 `_extract_topo_id_from_case` 实现；ptm-te 编排直接传 `--case-file` 由 CLI 提取，agent md 只声明提取规则契约，不重复实现（NFR4 解耦）。

### 8.2 PTM_TE_SKILLS 与 install 路径映射

```python
# install_agent 对每个 skill 调 copy_skill_tree（既有逻辑）：
#   src  = source_dir / "skills" / skill_name
#   dest = workspace_root / PLATFORM_DIRS[platform]["skills"] / skill_name
# 新增 skill 自动遵循同一映射：topo-config、topo-planning 与既有 5 个 skill 同级安装。
# uninstall 逆序：uninstall_agent 按 get_agent_skills 移除两个新 skill 目录。
```

### 8.3 release 挂接与既有清理顺序

ptm-te [7] 清理顺序（HLD §12.3）：先 ST-EX-06 用例 mutation ops 逆序清理，再 CR-037 release 台账占用。release 置于既有清理之后、[8] 快照 before 报告之前；仅当 `acquire`/映射使用过 topo-planning 且 execute 模式时执行（dry-run 跳过，对齐既有 dry-run 零副作用原则）。

### 8.4 模板副本与测试造数隔离（S8-A 双保险）

- **第一道防线（copy_skill_tree 裁剪，S8-A）**：`copy_skill_tree` 复制 skill 时排除 `tests/`、`__pycache__/`、`*.pyc`，即使源 skill 目录含 `tests/fixtures/*.yaml`（P-1 造数），也不会被复制进目标项目 `.claude/skills/` / `.agents/skills/`。
- **第二道防线（模板源占位，S8-D）**：模板副本源 `skills/topo-config/config/*.yaml`（P-1 模板化后）只含 `${PLACEHOLDER}` 占位符与注释指引，**不含** `hg3250-51` 等测试设备（那些仅存于 `skills/topo-config/tests/fixtures/` 与 `skills/topo-planning/tests/fixtures/`）；P-1 修订后 `device_type` 字段也全占位（`${DEVICE_TYPE}`）。
- S10 校验：安装后目标项目 `topology/*.yaml` 不含 `hg3250` / `DAS-TGFW-A1600-HU`（造数特征）；`.claude/skills/topo-config/` 与 `.claude/skills/topo-planning/` 下无 `tests/` 目录、无 `*.pyc`；与 `skills/topo-config/config/` 模板一致（NFR6）。

### 8.5 dry-run 预览 env-file 消费（S8-B）

- dry-run 时 [1.5] 落 `runs/<run-id>/topology.preview.yaml`（S7 CLI `--dry-run --output ...`，`metadata.executable:false`）。
- 编排 [2] 设备准备 / [4] 逐条 op 的 `${ENV.*}` 解析（case-execution `load_env_file`）在 dry-run 模式读取该预览副本，保证 **dry-run 全链路可走**（不因「无 env-file」中断）。
- **消费纪律**：任何执行消费（[4] `--execute` 真机下发 / S9 deploy_bridge）必须拒绝 `executable:false` 文件；dry-run 只做解析与命令构造展示，不真实下发（0 次真机写）。预览副本属 run 运行态（`runs/` 不入库），不触碰真实 `topology/topology.yaml` 路径。

---

## 9. 安全与性能设计

### 安全

- **凭据禁止**：模板副本源含 `${PLACEHOLDER}` 占位；install 不写任何真实密码/IP；`release` 只操作 `allocated_to`/`allocated_vlan`，不透传凭据（S4 语义）。
- **幂等保护**：模板副本目标存在即跳过，不覆盖用户已填数据（AG-7）；`.gitignore` 追加不删除用户已有规则。
- **不越权（S8-E 授权分层）**：ptm-te 编排 [1.5] 默认 dry-run（不写台账/不落真实路径，落预览副本）；execute 模式「写台账 + env-file 落盘」是 **workspace 写，执行前自动步骤**（需 `--authorized` 显式门，但不需 DQ-037-04）；**真机下发（S9 EnvironmentDeployer）才需 DQ-037-04 独立 runtime_authorization**（不在 CP 批准隐含）。编排 [1.5] 不因台账/落盘要求用户单独的真机下发授权，也不把 workspace 写误报为真机授权。
- **release 时机**：仅在 execute 且使用过 topo-planning 时释放，避免误释放他人占用。

### 性能

- install 模板副本：2 次 `copy2` + 1 次 `.gitignore` 检查/追加，< 10ms。
- [1.5] 拓扑映射：S7 CLI 全链路 < 1s（典型池）。
- [7] release：O(台账条目 × 端口数)，典型 < 5ms。
- 安装/编排无常驻进程。

---

## 10. 测试设计

### 单元测试（`script/tests/test_install_templates.py` 或 `script/test_install.py` 扩展）

| 测试用例 | 覆盖要点 | 预期 |
|---------|---------|------|
| `test_ptm_te_skills_includes_topo` | `PTM_TE_SKILLS` | 含 `topo-config` + `topo-planning`（方案 A，FR6） |
| `test_generate_templates_requires_p1_placeholder` | 前置校验（Minor-2）：源模板仍含 `A1600-HU`/`hg3250`/真实密码（模拟 P-1 未占位化） | `_assert_p1_template_placeholderized` 抛 RuntimeError，不生成模板副本（fail-fast）；占位化后通过 |
| `test_generate_templates_first_run` | 目标 `topology/` 不存在 | 生成 `physical_pool.yaml` + `physical_ledger.yaml` |
| `test_generate_templates_idempotent` | 目标已存在 | 跳过不覆盖（打印 skip，文件内容不变） |
| `test_generate_templates_no_fixtures` | 生成的模板副本内容扫描 | 无 `hg3250`/`DAS-TGFW-A1600-HU` 测试造数（FR10） |
| `test_ledger_template_schema` | `physical_ledger.yaml` | `devices` 为 list，含 `port_status[].allocated_to/allocated_vlan`（P0-3） |
| `test_pool_template_placeholder` | `physical_pool.yaml` | `management.password` 为 `${PLACEHOLDER}`，无真实 IP |
| `test_gitignore_append` | 目标 `.gitignore` 无规则 | 追加 `topology/*.yaml` |
| `test_gitignore_idempotent` | 已含规则 | 不重复追加（NFR1） |
| `test_gitignore_preserve_existing` | `.gitignore` 已有其他规则 | 保留原规则 + 追加新规则 |
| `test_dry_run_no_write` | install `--dry-run` | 打印 DryRun，不创建文件/不写 .gitignore（FR11） |
| `test_copy_skill_tree_strips_tests` | copy_skill_tree 复制含 `tests/fixtures/` + `__pycache__/` 的 skill（S8-A） | 目标目录无 `tests/`、无 `__pycache__/`、无 `*.pyc`；SKILL.md/src/scripts 保留 |
| `test_dry_run_preview_env_file` | [1.5] dry-run 落预览 env-file（S8-B） | `runs/<run-id>/topology.preview.yaml` 存在且 `metadata.executable:false`；真实 `topology/topology.yaml` 不存在；`load_env_file` 可解析 |
| `test_ptm_te_resource_link` | `component-resource-links.yaml` ptm-te 组件（S8-C/FR12） | 含 network-topology `tgfw-topo-collection` required；`get_component_resources("ptm-te")` 返回该 resource |
| `test_ptm_te_resource_installed` | `install --agent ptm-te` 安装资源（S8-C） | `~/.ptm-team/resource/network-topology/topology-collection.md` + `index.yaml` 存在，`index.yaml` 含 `tgfw-topo-collection` |

### 集成测试（S10）

- 层 A：`install --agent ptm-te --dry-run` 安装 topo-config + topo-planning + 生成模板副本（HLD §11 层 A 第 7 条）；断言目标项目 `topology/` 生成且不含造数；`.claude/skills/topo-config/` 无 `tests/`（S8-A）。
- 层 A：`install --agent ptm-te` 后 `~/.ptm-team/resource/network-topology/` 含 `topology-collection.md` + `index.yaml`（S8-C/FR12），S1 `find_topology_file` 可命中。
- 层 A：`topo-plan plan --case-file <fixture case> --dry-run --output runs/<run-id>/topology.preview.yaml` → 预览副本 → case_runner `load_env_file` 消费（S8 [1.5]→[2] dry-run 全链路，S8-B）。
- 层 A：`topo-plan plan --case-file <fixture case> --execute --authorized` → env-file 落盘 `topology/topology.yaml` → case_runner `load_env_file` 消费（S8 [1.5]→[2] execute 链路）。
- 层 A：映射 → ledger 写回 → `release(env_name)` → 台账恢复 free（P1-7 挂接，HLD §11 层 A 第 4 条）。
- 参考用例 `IPv4策略路由`（TOPO-01）：组网约束双源提取 → 映射 → env-file → dry-run 执行链路打通（HLD §11 层 A 第 8 条）。

### Fixture 需求

- 复用 P-1 fixture 池 + S4 ledger fixture + S7 case fixture。
- 新增 `script/tests/fixtures/gitignore_empty.txt` / `gitignore_existing.txt`（.gitignore 幂等测试）。
- 新增 `script/tests/fixtures/minimal_workspace/`（空 workspace 首装测试，含 `topology/` 缺失场景）。
- 新增 `script/tests/fixtures/skill_with_tests/`（含 `tests/fixtures/*.yaml` + `__pycache__/` + `src/` 的模拟 skill，S8-A 裁剪测试）。

---

## 11. 实施步骤

| TASK-ID | 描述 | 预估 | 产物 |
|---------|------|:---:|------|
| TASK-S8-01 | `PTM_TE_SKILLS` 增加 `topo-config` + `topo-planning` | 10min | 常量修改 |
| TASK-S8-02 | `ensure_target_project_templates` 函数（模板副本 + 幂等 + dry-run） | 1.5h | 模板生成 |
| TASK-S8-03 | `_ensure_gitignore_rule` 函数（检查/追加/幂等/保留既有） | 1h | .gitignore 维护 |
| TASK-S8-04 | `install_agent` 接线：ptm-te 安装流程调用模板副本函数 | 30min | 安装接线 |
| TASK-S8-05 | ptm-te.md 编排 [1.5] 拓扑映射步骤（E1/E2 双源提取 + S7 CLI 调用 + 失败分支） | 2h | 编排步骤 |
| TASK-S8-06 | ptm-te.md [7] release 挂接（P1-7：release + save_ledger + cleanup_summary） | 1.5h | release 挂接 |
| TASK-S8-07 | ptm-te.md 关联 Skill 表 + 异常路径 + dry-run 门（含预览副本）+ 修订记录 | 1h | agent md 文档 |
| TASK-S8-08 | `copy_skill_tree` 过滤复制（ignore_patterns 排除 tests/__pycache__/*.pyc，S8-A） | 1h | 裁剪复制 |
| TASK-S8-09 | `component-resource-links.yaml` ptm-te → network-topology 关联（S8-C/FR12） | 30min | 资源关联 |
| TASK-S8-10 | [1.5] dry-run 预览 env-file 消费链路接线（S8-B：--output runs/<run-id>/topology.preview.yaml） | 1h | dry-run 链路 |
| TASK-S8-11 | 单元测试（14 用例 + 5 fixture） | 2.5h | 测试 |
| TASK-S8-12 | 集成测试：install dry-run → 模板副本 → topo-plan → env-file 消费链路（含 layer2 collection 安装） | 2h | 集成验证 |
| **合计** | | **~14h** | |

---

## 12. 风险、难点与预研建议

| 风险 | 级别 | 缓解 |
|------|:---:|------|
| ptm-te 编排调用 S7 CLI 参数契约漂移 | 中 | S7/S8 设计证据同时确认 CLI 参数表；S10 集成测试锁定调用方式 |
| 模板副本覆盖用户已填数据 | 高 | 幂等跳过已有（AG-7）；S10 用「已存在文件」用例断言内容不变 |
| `topo-config` 引擎在 ptm-te 目标项目不可 import（PYTHONPATH） | 中 | S7 CLI 统一 PYTHONPATH 策略（S1 P2-4）；install 后两个 skill 的 src 目录均注入 |
| release 时机与既有 [7] 清理顺序冲突 | 中 | release 置于既有清理后；仅 execute + 使用过 topo-planning 时触发；dry-run 跳过 |
| topo_id 双源提取与 S7 CLI 实现双份漂移 | 中 | ptm-te 编排直接调 S7 CLI `--case-file` 提取；agent md 只声明规则契约（NFR4） |
| 目标项目 `.gitignore` 已含自定义 topology 规则 | 低 | `_ensure_gitignore_rule` 精确匹配 `topology/*.yaml`，已有则跳过，不覆盖其他规则 |

**预研建议**：
1. 用空 workspace 跑一次 `install --agent ptm-te --dry-run`，确认模板副本 + `.gitignore` 操作符合预期。
2. 与 S7 对齐 CLI 调用参数（`--env-name` 生成规则、execute/dry-run 映射），确保 ptm-te 编排调用无误。

---

## 13. 实现灰区与取舍记录

### 灰区 1：topo_id 提取双源优先级（E2）

**问题**：「组网约束」与「组网描述」两字段的 topo_id 提取优先级与失败语义。

**选项**：
- A. 组网约束优先（值匹配 node\d+_.+ 直接采用）；组网描述括号内提取回退；两字段均失败 → `TOPO_ID_NOT_FOUND` 终止。
- B. 组网描述优先（描述更接近用户意图）。
- C. 两字段不一致时报冲突错误。

**决策**：选 A。HLD §4.8 / 评审核实 E2 明确「组网约束优先、组网描述回退」；组网约束是结构化字段（直接 topo_id），组网描述是自然语言摘要（需括号提取）。失败终止避免静默映射错误拓扑。

### 灰区 2：S7 CLI 调用方式（ptm-te 直接调 CLI vs 内嵌引擎 import）

**问题**：ptm-te 编排 [1.5] 如何执行拓扑映射？

**选项**：
- A. 调 S7 CLI `topo_plan_cli.py plan ...`（subprocess 或直接函数调用），契约清晰、可复用。
- B. ptm-te agent md 内嵌 S1-S6 引擎调用代码（破坏 agent md 契约声明定位）。
- C. 新建独立编排脚本（重复 S7 CLI 逻辑）。

**决策**：选 A。ptm-te 是编排器，只声明「何时调用 topo-planning」契约；CLI 细节由 S7 承载（NFR4）。编排调用 `topo_plan_cli.py`（函数级 `run_plan` 或 subprocess），传结构化参数。

### 灰区 3：模板副本源（P1-8）

**问题**：install 生成目标项目模板副本的源路径。

**选项**：
- A. 源 = `skills/topo-config/config/physical_pool.yaml` + `physical_ledger.yaml`（P-1 模板化后，引擎消费的单一真相源）。
- B. 源 = S7 `templates/*.example`。
- C. 双源并存，install 校验一致性。

**决策**：选 A。P-1 已把 `skills/topo-config/config/` 模板化，是引擎直接消费的模板真相源；S7 templates 供人工参考。避免双套模板漂移（S10 校验一致性，NFR6）。

### 灰区 4：release 挂接时机与条件（P1-7）

**问题**：`PhysicalPool.release(env_name)` 在 ptm-te 编排哪个阶段调用？dry-run 是否释放？

**选项**：
- A. [7] 用例清理（teardown/环境删除）时调用；仅 execute 且使用过 topo-planning 时触发；dry-run 跳过。
- B. [8] 报告后调用（清理末期）。
- C. 每次用例结束后立即释放（run 中途释放）。

**决策**：选 A。S4 §13 灰区 3（P1-7）明确「S8 在 ptm-te 编排 [7] 清理流程（环境删除/teardown）挂接调用」，与既有 inverse_op 清理同一阶段；dry-run 零副作用原则跳过。run 中途释放会导致后续用例端口占用丢失（违背 H1 跨用例防冲突）。

### 灰区 5：`.gitignore` 规则粒度

**问题**：模板副本与真实数据文件需要忽略，规则粒度。

**选项**：
- A. `topology/*.yaml`（忽略物理池/台账/env-file，但保留目录中其他非敏感文件）。
- B. `topology/`（整目录忽略）。
- C. `topology/` + `.env`（连同凭据环境文件）。

**决策**：选 A。任务约束明确「沿用 `topology/*.yaml`」；`topology/` 整目录忽略可能误伤用户希望入库的说明文件（如 `topology/README.md`）。`.env` 已有既有规则（`.gitignore:42`），不重复。

**S8-F 确认**：`.gitignore` 粒度维持 `topology/*.yaml` 单条，无需新增忽略项。dry-run 预览 env-file 写入 `runs/<run-id>/`（ptm-te 运行态，`runs/` 目录不入库，现有 ptm-te 运行时工作目录约定已覆盖），故不触发 `.gitignore` 新增规则。

### 灰区 6：copy_skill_tree 整树复制 vs 造数隔离（S8-A，CP5 BLOCKER 修订新增）

**问题**：`copy_skill_tree` 用 `shutil.copytree` 整树复制，会把 topo-config / topo-planning 的 `tests/fixtures` 造数（hg3250 等）带入目标项目 `.claude/skills/`，击穿「不含测试造数」承诺。

**选项**：
- A. copy_skill_tree 过滤复制：`ignore_patterns("tests", "__pycache__", "*.pyc", "*.pyo")`，统一裁剪（对 ptm-tde / ptm-tse 既有 skill 同样安全）。
- B. 只对 topo-config / topo-planning 特判裁剪（其他 skill 整树复制）。
- C. 不裁剪，仅在 SKILL.md 声明 fixture 不入库（承诺不真实成立）。

**决策**：选 A。统一在 `copy_skill_tree` 层裁剪运行态/测试资产（tests/、__pycache__/、*.pyc、*.pyo），目标安装产物不含造数，承诺真实成立；配合模板源 `skills/topo-config/config/` 占位化（S8-D），双保险。B 方案维护成本高且其他 skill 也可能含 tests；C 方案承诺不成立（BLOCKER 未修）。

### 灰区 7：dry-run 不落盘 vs 下游 `${ENV.*}` 消费（S8-B，CP5 BLOCKER 修订新增）

**问题**：dry-run 无 env-file，但编排 [2] 设备准备 / [4] 逐条 op 必须解析 `${ENV.*}`（case-execution `load_env_file` 按文件路径加载），连 dry-run 链路都走不通。

**选项**：
- A. dry-run 也落预览 env-file 到 `runs/<run-id>/topology.preview.yaml`（`metadata.executable:false`），不落真实 `topology/` 路径；下游解析预览副本验证链路。
- B. dry-run 不落文件，链路内内存传递 env dict（绕过 load_env_file）。
- C. dry-run 落正式 `topology/topology.yaml`（与 execute 同路径）。

**决策**：选 A。dry-run 落预览副本到 run 目录（运行态不入库），标注 `executable:false` 使下游可解析但执行消费拒绝；保持 dry-run 零副作用（不写台账/不碰真实路径/0 次真机写），同时全链路可走。B 破坏 case_runner 文件路径加载模型；C 污染真实环境文件路径且无法区分预览/正式。

### 灰区 8：layer2 collection 资源安装归属（S8-C，CP5 修订新增）

**问题**：S1 `find_topology_file` 运行时消费层2共享资源 `resource/network-topology/topology-collection.md`（`~/.ptm-team/resource/`）。ptm-te 安装若不安装该资源，S1 运行时缺 collection.md。归属在哪？

**选项**：
- A. `component-resource-links.yaml` 为 `ptm-te` 组件增加 network-topology `tgfw-topo-collection`（required），复用 `get_component_resources` 既有安装分支。
- B. install.py 在 ptm-te 分支内联硬编码安装 network-topology。
- C. 依赖 ptm-tde 已安装（ptm-tde 组件已关联 tgfw-topo-collection），ptm-te 不重复装。

**决策**：选 A。`component-resource-links.yaml` 是组件-资源关联的单一声明源，ptm-te 增加声明后 `install_agent` 既有逻辑自动安装（collection.md + index.yaml 到 `~/.ptm-team/resource/network-topology/`）；不硬编码（B 方案破坏声明式管理）；不依赖 ptm-tde（C 方案在 ptm-te 独立安装场景失效，且 tgfw-topo-collection 的 `compatible_consumers` 已含 ptm-te，ptm-te 有独立消费权）。

---

## 14. 回滚与发布策略

### 回滚

- `PTM_TE_SKILLS` 移除 `topo-config`/`topo-planning` 两行 → 恢复 5 个 skill（不卸载已安装目录，install 幂等）。
- `copy_skill_tree` 恢复整树复制（`ignore_patterns` 参数移除）→ 裁剪逻辑回退；已安装 skill 不含 tests（裁剪后安装的），无需额外清理。
- `ensure_target_project_templates` / `_ensure_gitignore_rule` 函数删除 → 模板副本逻辑移除；已生成的模板副本保留（用户数据），回滚代码不影响用户已填数据。
- `component-resource-links.yaml` 移除 `ptm-te` 组件条目 → ptm-te 不再随装 layer2 network-topology；已安装的 `~/.ptm-team/resource/network-topology/` 共享资源保留（`uninstall_agent` 对共享 resource 只移除 installed_for 归属，不删文件）。
- ptm-te.md [1.5] 步骤与 [7] release 挂接移除 → 编排恢复原 [1]-[8]（无拓扑映射）。
- 已写台账占用：回滚后由 `release(env_name)`（若仍安装）或人工置空清理。

### 发布

- S8 随 CR-037 交付；ptm-te 安装需重新执行 `install --agent ptm-te`（install.py 变更生效 + 新 skill 安装 + 模板副本生成）。
- 依赖顺序：S8 = [S7, P-1] precedence（Minor-2）——S7 CLI 是编排调用入口；**P-1 模板化必须先于 S8 模板副本生成**（否则源仍为真实数据，前置校验 `_assert_p1_template_placeholderized` 会 fail-fast）；S4 release 接口已就绪（P1-7 前置）。
- 既有 ptm-te 用户升级：重复 `install --agent ptm-te` 幂等；模板副本只在目标缺失时生成，不覆盖用户数据。

---

## 15. Definition of Done（DoD）

- [ ] `PTM_TE_SKILLS` 含 `topo-config` + `topo-planning`（方案 A，FR6）
- [ ] `copy_skill_tree` 过滤复制：排除 `tests/`、`__pycache__/`、`*.pyc`（S8-A，安装产物不含造数）
- [ ] `ensure_target_project_templates` 生成 `topology/physical_pool.yaml` + `physical_ledger.yaml`（源 = skills/topo-config/config/，P-1 后 device_type 全占位，FR7）
- [ ] **模板副本前置校验（Minor-2）**：`_assert_p1_template_placeholderized` 在生成模板副本前校验源模板已占位化（不含 A1600-HU/hg3250 等特征串），未占位化则 fail-fast；`depends_on` 显式含 CR037-P1（LLD frontmatter + Story 卡片 + DEVELOPMENT-PLAN 三处一致）
- [ ] 模板副本幂等：目标存在 → 跳过不覆盖（FR8/NFR1）
- [ ] `.gitignore` 追加 `topology/*.yaml`（缺失时，保留既有规则，FR9/S8-F）
- [ ] 台账模板 schema = `list + port_status`（P0-3），不含测试造数（FR10）
- [ ] install `--dry-run` 打印模板副本与 .gitignore 操作，不写文件（FR11）
- [ ] `component-resource-links.yaml` ptm-te → network-topology（`tgfw-topo-collection`，required），安装后 `~/.ptm-team/resource/network-topology/` 含 collection.md + index.yaml（FR12/S8-C）
- [ ] ptm-te 编排 [1.5] 拓扑映射步骤：topo_id 双源提取（E2）+ 调 S7 CLI + 成功/失败分支（FR1-FR4，E1）
- [ ] [1.5] dry-run 落预览 env-file 到 `runs/<run-id>/`（`executable:false`），下游 load_env_file 可解析（S8-B/FR3）
- [ ] topo_id 提取失败 → `TOPO_ID_NOT_FOUND` 终止（FR1）
- [ ] 映射失败 → `TOPO_MAPPING_FAILED` + 结构化报告（FR4）
- [ ] 授权分层（S8-E）：[1.5] 映射+台账占用=执行前自动步骤（不需 DQ-037-04）；真机下发（S9）才需 DQ-037-04
- [ ] 编排 [7] 清理挂接 `PhysicalPool.release(env_name)` + `save_ledger`（FR5，P1-7，幂等 NFR5）
- [ ] ptm-te.md 关联 Skill 表 + 异常路径 + dry-run 门 + 修订记录更新
- [ ] 14 个单元测试全部通过
- [ ] lld-check: LLD Structure Check OK
