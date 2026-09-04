---
story_id: "CR037-S9"
story_slug: "deploy-bridge"
cr_id: "CR-037"
title: "可选真机部署接线（EnvironmentDeployer，--execute 独立 runtime_authorization）"
priority: "P1"
wave: 3
status: "lld-ready"
depends_on:
  - "CR037-S5"
  - "CR037-S7"
dependency_type: "precedence"
assignee: null
dev_context: "新建 skills/topo-planning/src/deploy_bridge.py：复用 topo-config EnvironmentDeployer（deployer.py）做真机部署桥接。deploy_bridge 读取 S7 CLI 产出的 env-file 与物理池/台账文件，从 env-file（port_mapping/nodes/links）+ physical_pool 推导 deployer 所需 env/devices/broadcast_domains 三段（S9-A/S9-B，构造参数不可注入），构造 EnvironmentDeployer 并执行 deploy/verify/teardown。--execute 为独立 runtime_authorization 决策项（O3 / DQ-037-04），不在 CP 批准隐含；默认 dry-run。"
validation_context: "dry-run 桥接（构造 EnvironmentDeployer + plan + 打印将下发配置，0 次真机写操作）；deploy/verify/teardown 的 dry_run=True 调用可被单测断言（mock 子进程/API）；--execute 路径仅在 runtime_authorization 批准后可达（gate 检查）；env/devices/broadcast_domains 从 env-file+池推导成功（不崩溃、不静默空转）；凭据走 ${ENV_VAR} 占位，产物不含密码。"
acceptance_criteria:
  - "deploy_bridge.deploy(env_file, pool_path, ledger_path, dry_run=True) 构造 EnvironmentDeployer 并返回 plan 摘要（不写真机）"
  - "dry_run=True 时 0 次真机写操作（deploy/verify/teardown 均走 EnvironmentDeployer dry_run 分支）"
  - "--execute 需 DQ-037-04 独立 runtime_authorization；未授权时阻断并提示"
  - "env/devices/broadcast_domains 三段由 DeployBridge 从 env-file（port_mapping/nodes/links）+ physical_pool 推导（S9-A/S9-B），推导失败抛结构化错误，不静默空转"
  - "teardown 桥接复用 EnvironmentDeployer.teardown(dry_run=...) 并联动 S4 release（P1-7 挂接由 S8 编排承载；S9 独立运行时台账释放归属注明，S9-C）"
  - "凭据禁止：deploy_bridge 不接收/不输出密码，密码仅经环境变量名传递"
output_files:
  - "skills/topo-planning/src/deploy_bridge.py"
file_ownership:
  primary:
    - "skills/topo-planning/src/deploy_bridge.py"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "technical-note"
  trigger_reasons:
    - "复用 EnvironmentDeployer，模式既有"
  rationale: "桥接层接入点明确，Story 技术说明足够"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S9.md"
---

# STORY-CR037-S9：可选真机部署接线

## 目标

新建 `skills/topo-planning/src/deploy_bridge.py`：将 S7 `topo-plan` 产出的 env-file 与映射上下文桥接到 topo-config `EnvironmentDeployer`（`deployer.py`），提供 `deploy` / `verify` / `teardown` 三个桥接入口。默认 dry-run；`--execute` 为独立 `runtime_authorization` 决策项（O3 / DQ-037-04），不在 CP 批准隐含。

## 依赖

- CR037-S5（exporter.py 产出 env-file，桥接输入契约）
- CR037-S7（topo-plan CLI 产出 env-file 与映射上下文）

## 输出文件

| 文件 | 说明 |
|------|------|
| `skills/topo-planning/src/deploy_bridge.py` | 真机部署桥接层（复用 EnvironmentDeployer） |

## AI 任务清单

- [ ] TASK-S9-01：`deploy_bridge.py` 骨架 + `DeployBridge` 类（env_file / pool_path / ledger_path / dry_run / fw_password_env / authorized）
- [ ] TASK-S9-02：`_build_deployment_view()` 转换层：从 env-file（port_mapping/nodes/links）+ physical_pool 推导 env/devices/broadcast_domains（S9-A/S9-B），推导失败抛 `DEPLOY_VIEW_INCOMPLETE`
- [ ] TASK-S9-03：`_build_deployer()`：读取产物 env-file + 池/台账文件构造 EnvironmentDeployer（构造参数仅路径，S9-A）
- [ ] TASK-S9-04：`deploy(env_file, dry_run=True)` 桥接：构造 EnvironmentDeployer + plan + 打印摘要
- [ ] TASK-S9-05：`verify(dry_run=True)` 桥接：EnvironmentDeployer.verify（TRex 打流验证，可选）
- [ ] TASK-S9-06：`teardown(dry_run=True)` 桥接：EnvironmentDeployer.teardown + 联动 S4 release（P1-7 挂接由 S8 承载，S9 不重复实现）
- [ ] TASK-S9-07：`--execute` gate 检查（DQ-037-04 runtime_authorization），未授权阻断；`executable:false` 预览副本拒绝真机部署
- [ ] TASK-S9-08：单测：dry-run 构造 + 部署视图推导 + 0 次真机写 + gate 阻断 + teardown 联动

## 技术说明

### 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-08-05 | meta-dev | 初稿（复用 EnvironmentDeployer + --execute gate） |
| v1.1 | 2026-08-05 | meta-dev | CP5 NEEDS_REWORK 修订：S9-A 构造注入不可行（EnvironmentDeployer 构造仅路径，新增 `_build_deployment_view()` 转换层从 env-file + physical_pool 推导 env/devices/broadcast_domains）；S9-B env-file vs deployer 消费结构不匹配适配（推导失败抛 DEPLOY_VIEW_INCOMPLETE，不静默空转）；S9-C 独立运行台账释放归属显式声明（由 S4 `PhysicalPool.release` + S8 编排 [7] 承载，S9 不重复实现） |

### 设计依据

- HLD §4.9：可选真机部署（S9）复用 `EnvironmentDeployer`；`--execute` 为独立 `runtime_authorization` 决策项（O3 / DQ-037-04），不在 CP 批准隐含。
- 既有能力：`skills/topo-config/src/deployer.py` 提供 `EnvironmentDeployer`，构造签名 `(expected_topo_path, pool_path, ledger_path, interactive, trex_api, fw_password_env)`；方法 `plan(preset_subnets)` / `deploy(dry_run=True)` / `verify(dry_run=True)` / `teardown(dry_run=True)` / `print_report(report)`。S9 只做桥接，不重复实现。
- **构造注入不可行（S9-A，CP5 修订）**：`EnvironmentDeployer` 构造参数只有路径（expected_topo_path/pool_path/ledger_path/interactive/trex_api/fw_password_env），**不存在注入 `env` / `devices` / `broadcast_domains` 三段内存对象的参数**；构造时直接从 `expected_topo_path` 读文件并取 `topo['env']['name']`、`topo['devices']`、`topo['broadcast_domains']`。因此 DeployBridge **不能**「通过构造参数注入」部署上下文，必须先把 env-file + 物理池推导为可被 EnvironmentDeployer 直接消费的**部署视图**（见「数据变化」），再喂给 deployer。
- **env-file 消费结构适配（S9-B，CP5 修订）**：S5 env-file 是 `metadata + port_mapping + nodes + links` 四段（case-execution `${ENV.*}` 消费契约），**不含** `env` / `devices` / `broadcast_domains` 三段（deployer 消费契约）。DeployBridge 在部署路径前做**转换层**：从 env-file 的 `port_mapping/nodes/links` + `physical_pool.yaml` 推导 deployer 所需三段（缺省从池推导），不允许崩溃或静默空转（推导失败抛结构化错误）。
- 与 S7 边界：S7 `topo-plan` CLI 产出 env-file（`topology/topology.yaml` 或 dry-run 预览副本 `runs/<run-id>/topology.preview.yaml`）；S9 `deploy_bridge` 消费 env-file（`executable:true` 才能进入真机部署）与物理池/台账文件，经转换层构造 `EnvironmentDeployer`。
- 与 S4/S8 边界（S9-C）：真机 teardown 后台账释放由 S4 `PhysicalPool.release(env_name)` 承载，调用挂接由 S8 编排 [7]（P1-7）执行；S9 teardown 桥接只负责 `EnvironmentDeployer.teardown` 侧（交换机/防火墙配置清理），台账释放**不在此重复实现**。DeployBridge 独立运行（不经 S8 编排）时，台账释放归属明确为调用方经 S4 `release` 或 S8 编排 [7] 承担，S9 只负责部署侧。

### 文件影响

| 文件 | 操作 | 说明 |
|------|------|------|
| `skills/topo-planning/src/deploy_bridge.py` | **新建** | 桥接层（~200 行）：`DeployBridge` 类 + `deploy/verify/teardown` 三入口 |
| `skills/topo-config/src/deployer.py` | **不修改** | 只读复用 `EnvironmentDeployer` |
| `skills/topo-planning/scripts/topo_plan_cli.py` | **不修改** | S7 CLI 保持「映射 + 导出」边界；部署由 S9 独立入口承接 |
| `agents/ptm-te.md` / `script/install.py` | **不修改** | S8 写入范围；S9 仅提供模块 |

### 接口变化

- 新增 `DeployBridge` 类：

```python
class DeployBridge:
    def __init__(self, env_file: str, pool_path: str, ledger_path: str,
                 dry_run: bool = True, fw_password_env: str = "NGFW_PASSWORD",
                 authorized: bool = False):
        # authorized = DQ-037-04 runtime_authorization 已批准标记（真机下发级）
        ...

    def _build_deployment_view(self) -> dict:
        """转换层（S9-A/S9-B）：从 env-file（port_mapping/nodes/links）+ physical_pool 推导
        deployer 所需 env/devices/broadcast_domains 三段；缺省从池推导，推导失败抛
        DeployBridgeError(DEPLOY_VIEW_INCOMPLETE)，不静默空转。
        - env: name 从 metadata.env_name 推导（缺省 <topo_id>-<run-id>）
        - devices: 从 nodes（tg1/dut1）+ pool（management/api_server/型号/端口）推导
        - broadcast_domains: 从 links 配对端口 + pool 推导广播域（DUT-DUT 直连 L3 域 + SW 域）
        返回部署视图 dict，供写入临时 env-file 或直接喂 EnvironmentDeployer 消费结构。
        """
        ...

    def _build_deployer(self) -> EnvironmentDeployer:
        """读取产物 env-file + 池/台账文件构造 EnvironmentDeployer（S9-A，构造参数仅路径）；
        先用 _build_deployment_view() 补齐 env/devices/broadcast_domains 到 env-file 的部署视图
        （写入 runs/<run-id>/deploy-view.yaml 或内存临时文件），再 EnvironmentDeployer(deploy_view, pool_path, ledger_path, ...)。"""
        ...

    def plan(self, preset_subnets: Optional[dict] = None) -> list:
        """构造 EnvironmentDeployer + ipam.plan，返回 IP 规划（0 次真机写）"""
        ...

    def deploy(self, dry_run: bool = True) -> dict:
        """EnvironmentDeployer.deploy(dry_run)，返回 DeployReport 摘要"""
        ...

    def verify(self, dry_run: bool = True) -> list:
        """EnvironmentDeployer.verify(dry_run)，返回 TrafficResult 列表（可选打流验证）"""
        ...

    def teardown(self, dry_run: bool = True) -> dict:
        """EnvironmentDeployer.teardown(dry_run)，返回清理摘要；台账释放由 S8 [7] release 承载（S9-C，不重复实现）"""
        ...
```

- `--execute` gate：`deploy(dry_run=False)` / `teardown(dry_run=False)` 前检查 `authorized=True`；未授权时抛 `RuntimeAuthorizationError("--execute 需 DQ-037-04 独立 runtime_authorization")`，阻断真机写。
- 无独立 CLI 入口；由 S7 `topo-plan --execute` 或 S8 ptm-te 编排可选调用（S7 `--authorized` 仅覆盖 workspace 写；真机下发还需 S9 侧 `authorized=True`，两级分层，S8-E/S7-B 对齐）。

### 数据变化

- 无持久化 schema 变化。`DeployBridge` 消费 env-file（S5 契约：metadata/port_mapping/nodes/links）+ 物理池/台账（S3 契约）；输出为 `DeployReport` 摘要 / `TrafficResult` 列表（内存对象）。
- **部署视图推导（S9-A/S9-B）**：`EnvironmentDeployer` 构造读 env-file 需 `env` 段（`topo['env']['name']`，TrafficVerifier 消费）、`devices` 段（`_find_trex_api` / `_get_terminal_devices` 消费）、`broadcast_domains` 段（`plan()` 消费）。S5 env-file 不含这三段，DeployBridge 在 `_build_deployment_view()` 中从现有数据推导：
  - `env.name` ← `metadata.env_name`（缺省 `<topo_id>-<run-id>`）；
  - `devices` ← `nodes.tg1/dut1`（node_type / physical_name / management）+ `physical_pool.yaml` 同节点（management.host / api_server / device_type / hardware_platform / 端口物理名），TG 节点含 `node_type: TG` 与 `management.api_server`；
  - `broadcast_domains` ← `links` 配对端口 + `physical_pool.yaml` links（media_type / 端口速率），DUT-DUT 直连链路推导 L3 直连广播域，SW 接入链路推导 L2 广播域。
- 推导产物写入 `runs/<run-id>/deploy-view.yaml`（部署视图，运行态不入库），供 `_build_deployer()` 以该文件作为 `expected_topo_path` 构造 `EnvironmentDeployer`。推导任一段失败（如物理池无对应节点）→ `DeployBridgeError(code="DEPLOY_VIEW_INCOMPLETE", detail=...)`，不静默空转。

### 权限变化

- **`--execute` 为独立 runtime_authorization（O3 / DQ-037-04）**：设计证据（CP5 批准）不等于运行授权；真机写操作需单独授权记录（who/scope/authorized_at/reason），对齐 ptm-te dry-run 默认门（ADR-04）与 case-execution `runtime_authorization` 双层审计。
- **授权分层（S8-E/S7-B 对齐）**：S7 CLI `--authorized` 只覆盖「写台账 + env-file 落盘」（workspace 写，执行前自动步骤）；S9 真机下发（`deploy/teardown` 的 `dry_run=False`）是**另一档 DQ-037-04 runtime_authorization**，`DeployBridge.authorized=True` 单独确认，两者不可互相替代。dry-run 桥接（`dry_run=True`）不需要授权，0 次真机写。
- 凭据禁止：`DeployBridge` 接收 `fw_password_env`（环境变量名）而非密码值；不读取/不输出 `management.password`；密码由 `EnvironmentDeployer` 内部按既有契约消费。
- 不新增网络权限；部署/验证走既有 `deployer.py` 的 REST/Telnet 通道。

### 异常、失败与回退

**失败路径**（结构化暴露，不静默吞错）：

- `RuntimeAuthorizationError`：`dry_run=False` 但 `authorized=False` → 阻断真机写，提示 DQ-037-04 独立授权。
- 部署视图推导失败（env/devices/broadcast_domains 无法从 env-file + 物理池推导，如物理池无对应节点 / links 无法配对广播域）→ `DeployBridgeError(code="DEPLOY_VIEW_INCOMPLETE", detail=...)` + 缺失段与补全建议（补 physical_pool 对应节点 / 检查 env-file links）；**不构造注入、不静默空转**（S9-A/S9-B）。
- env-file 为 dry-run 预览副本（`metadata.executable:false`）却被传入 `deploy(dry_run=False)` → 拒绝，提示先经 execute 生成正式 env-file。
- `EnvironmentDeployer.deploy/teardown` 内部失败（登录失败/配置回滚失败）→ 透传 `DeployReport` 错误摘要 + `deploy_state.json` 残留提示（既有 deployer 行为）。
- 物理池/台账缺失（目标项目未初始化）→ 透传 `PhysicalPool.load` 的 `FileNotFoundError`，提示运行 S8 install 生成模板副本。

**回退**：

- `dry_run=True` 时全部方法 0 次真机写（仅 plan + 打印将下发配置），对齐 dry-run 默认门。
- 真机部署失败 → 既有 `deployer.py` 尽力回滚（`_save_deploy_state` 记录已下发配置）；回滚失败 → 提示查看 `deploy_state.json` 人工清理。
- S9 未就绪不影响 S7 映射/导出（部署是可选步骤）；ptm-te 编排可先走 env-file 消费链路，部署留 follow-up。

### 测试入口

- 单测：`skills/topo-planning/tests/test_deploy_bridge.py`：
  - dry-run 构造：`DeployBridge(env_file, pool, ledger, dry_run=True)` → `plan()` 返回 IP 规划，0 次真机写（mock deployer 方法断言调用参数）。
  - 部署视图推导（S9-A/S9-B）：`_build_deployment_view()` 从 fixture env-file（port_mapping/nodes/links）+ fixture 池推导 `env/devices/broadcast_domains` 三段，断言 `env.name`、`devices[].node_type`、`broadcast_domains` 非空且与池一致。
  - 推导失败：物理池缺对应节点 → `DeployBridgeError(DEPLOY_VIEW_INCOMPLETE)`，不崩溃、不静默空转。
  - gate 阻断：`deploy(dry_run=False)` 未授权 → `RuntimeAuthorizationError`。
  - gate 放行：`authorized=True` → `deploy(dry_run=False)` 调用 mock `EnvironmentDeployer.deploy`。
  - 预览副本拒绝：`metadata.executable:false` 的 env-file 传 `deploy(dry_run=False)` → 拒绝（提示先 execute 生成正式 env-file）。
  - teardown 联动：`teardown()` 调用 mock `EnvironmentDeployer.teardown`；台账释放由 S8 集成测试覆盖（S9-C，不在此重复）。
  - 凭据断言：DeployBridge 构造/输出不含密码明文。
- 集成（S10）：`topo-plan --execute --authorized`（授权后）→ env-file → `DeployBridge.deploy(dry_run=True)` 打印将下发配置；真机 `--execute` 需 DQ-037-04 独立授权（层 B 可选，HLD §11 第 10 条，S10-B 标注 N/A 不默认执行）。
- 复用 P-1 fixture 池（hg3250 + TE 端口）+ S7 env-file fixture。

### 风险与重访条件

| 风险 | 级别 | 重访条件 |
|------|------|---------|
| S5 env-file 不含 `env`/`devices`/`broadcast_domains` 段，`EnvironmentDeployer` 构造失败（S9-A/S9-B） | 中 | S9 已实现部署视图转换层（`_build_deployment_view` 从 env-file + 池推导）；若推导逻辑复杂（多广播域/多 SW）转 S5 设计修订（env-file 增加部署段） |
| 部署视图推导依赖 physical_pool 节点与 env-file nodes 可关联 | 中 | 推导按 host/device_type 关联；关联失败 → `DEPLOY_VIEW_INCOMPLETE` 结构化错误，提示补 physical_pool |
| `EnvironmentDeployer` 依赖交互式输入（`interactive=True`） | 中 | 桥接固定 `interactive=False` + `preset_subnets` 自动模式；若 IPAM 需用户确认则提示 |
| 真机部署失败残留配置 | 中 | 复用 `deployer.py` 尽力回滚 + `deploy_state.json` 人工清理指引 |
| `--execute` 授权记录缺失 | 低 | S8 ptm-te 编排按既有 runtime_authorization 双层审计记录；S9 只做 gate 检查 |
| deployer.py 签名变化（后续 CR） | 低 | 桥接固定调用 `plan/deploy/verify/teardown` 公开方法，签名稳定 |
| 台账释放边界误判（S9-C） | 低 | 文档显式声明台账释放由 S4 `PhysicalPool.release` + S8 [7] 承载；S9 独立运行时由调用方承担，S9 不重复实现 |

### 偏离记录

- **CP5 修订（S9-A/S9-B/S9-C）**：v1.0 曾表述「构造 EnvironmentDeployer 并执行」未说明 env/devices/broadcast_domains 三段来源；修订后明确构造参数不可注入，新增 `_build_deployment_view()` 转换层从 env-file + physical_pool 推导三段（不崩溃、不静默空转）；台账释放归属显式声明由 S4 `PhysicalPool.release` + S8 编排 [7] 承载，S9 不重复实现。其余对齐 HLD §4.9 与 DEVELOPMENT-PLAN S9（technical-note，`deploy_bridge.py` 新建）。

## 设计证据

本 Story 为 technical-note，设计证据内嵌本文件 `## 技术说明` 章节（含设计依据/文件影响/接口/数据/权限/失败/测试/风险）。
