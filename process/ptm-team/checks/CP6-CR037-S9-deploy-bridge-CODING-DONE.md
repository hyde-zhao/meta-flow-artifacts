---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S9"
story_slug: "deploy-bridge"
cr_id: "CR-037"
wave: 3
design_evidence_type: "technical-note"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S9 deploy_bridge

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-planning/src/deploy_bridge.py` | 新建 | `DeployBridge` 类：`_build_deployment_view()` 从 env-file（port_mapping/nodes/links）+ physical_pool 推导 env/devices/broadcast_domains 三段（S9-A/S9-B，推导失败 `DeployBridgeError(DEPLOY_VIEW_INCOMPLETE)`）；`_build_deployer()` 写 deploy-view.yaml + 构造 `EnvironmentDeployer`（构造参数仅路径，S9-A）；`plan/deploy/verify/teardown` 四入口；`--execute` gate（DQ-037-04，未授权 `RuntimeAuthorizationError`）；executable:false 预览副本拒绝真机部署（EXECUTABLE_FALSE）；EnvironmentDeployer 懒加载（避免模块级 import 拉入 requests） |
| `skills/topo-planning/tests/test_deploy_bridge.py` | 新建 | 8 个 pytest 单元测试（对齐 S9 技术说明 §测试入口） |
| `skills/topo-planning/tests/fixtures/env_file_direct.yaml` | 新建 | S5 风格 env-file（executable:true，匹配 pool_direct_tg_dut） |
| `skills/topo-planning/tests/fixtures/env_file_preview.yaml` | 新建 | 预览副本 env-file（executable:false） |
| `skills/topo-planning/tests/fixtures/env_file_incomplete.yaml` | 新建 | 推导失败 env-file（TG api_server 不存在于池） |

## 2. 设计契约映射

| S9 技术说明契约 | 实现 |
|------|------|
| `DeployBridge(env_file, pool_path, ledger_path, dry_run=True, fw_password_env='NGFW_PASSWORD', authorized=False)` | ✅ 构造签名对齐；`fw_password_env` 只接收环境变量名（凭据禁止，ADR-02） |
| `_build_deployment_view()` 推导 env/devices/broadcast_domains（S9-A/S9-B） | ✅ `env.name` ← `metadata.env_name`（缺省 `<topo_id>-deploy`）；`devices` ← pool TG/DUT 节点（node_type/management/型号）；`broadcast_domains` ← links+port_mapping+pool（每链路 p2p 域，gateway=DUT 侧、endpoint=TG 侧，mode='l3'） |
| 推导失败抛 `DeployBridgeError(DEPLOY_VIEW_INCOMPLETE)`，不崩溃/不静默空转 | ✅ TG/DUT 定位失败、port_mapping/links 缺失、端口反查失败均抛结构化错误（含补全建议） |
| 构造参数仅路径，不注入内存对象（S9-A） | ✅ `_build_deployer()` 先 `_write_deploy_view()` 写 `runs/<run-id>/deploy-view.yaml`，再 `EnvironmentDeployer(deploy_view_path, pool_path, ledger_path, interactive=False, ...)` |
| `plan(preset_subnets=None)`（0 次真机写） | ✅ `deployer.plan()` 返回 IP 规划摘要；不触发 deploy/teardown |
| `deploy(dry_run=True)` / `verify(dry_run=True)` / `teardown(dry_run=True)` | ✅ 均透传 dry_run 到 `EnvironmentDeployer` 对应方法，返回摘要 |
| `--execute` gate（DQ-037-04）：dry_run=False 未授权 → RuntimeAuthorizationError | ✅ `_check_execute_authorized`：dry_run=False 且 authorized=False → 抛（提示 who/scope/authorized_at/reason 记录）；`test_execute_requires_authorization` 验证 |
| authorized=True → `deploy(dry_run=False)` 调 EnvironmentDeployer.deploy | ✅ `test_execute_authorized_deploys` 验证 mock deployer 被调 `deploy(dry_run=False)` |
| executable:false 预览副本拒绝真机部署 | ✅ `_build_deployment_view` 检查 `metadata.executable is False` 且本次 dry_run=False → `DeployBridgeError(EXECUTABLE_FALSE)`；`test_preview_copy_rejected` 验证 |
| teardown 台账释放归属 S4 release + S8 [7]（S9-C） | ✅ `teardown()` 只调 `EnvironmentDeployer.teardown`；台账释放由 S8 编排 [7] `PhysicalPool.release` + `save_ledger` 承载，docstring 显式声明 |
| 凭据禁止（ADR-02） | ✅ DeployBridge 不接收/不输出密码值；`test_no_credential_in_output` 断言部署视图无 password/token/api_key/secret |

## 3. 单元测试结果

命令：`uv run --python 3.11 python -m pytest skills/topo-planning/tests/test_deploy_bridge.py`

结果：**8 passed**

覆盖要点：
- `test_dry_run_plan_zero_real_write`（dry-run 构造 + 0 次真机写：mock deployer 无 deploy/teardown 调用）
- `test_deployment_view_derivation`（S9-A/S9-B：env/devices/broadcast_domains 推导正确、与池一致）
- `test_deployment_view_incomplete`（推导失败 → DEPLOY_VIEW_INCOMPLETE，不静默空转）
- `test_execute_requires_authorization`（DQ-037-04 gate 阻断）
- `test_execute_authorized_deploys`（authorized=True 放行）
- `test_preview_copy_rejected`（executable:false 拒绝真机部署）
- `test_teardown_calls_deployer`（teardown 调 EnvironmentDeployer.teardown）
- `test_no_credential_in_output`（凭据禁止）

## 4. install dry-run 结果

S9 为纯模块（无独立安装器改动）；随 `topo-planning` skill 由 S8 `install --agent ptm-te` 安装（copy_skill_tree 保留 `src/deploy_bridge.py`，裁剪 tests/）。install dry-run 见 S8 CP6。

## 5. 偏离 S9 技术说明设计点

1. **EnvironmentDeployer 懒加载**：技术说明未显式说明 import 方式；实现为避免 `deployer.py` 模块级 `import requests` 导致「未安装 requests 环境（纯测试/构造场景）」import 失败，改为 `_load_deployer()` 懒加载。测试可 patch `deploy_bridge.EnvironmentDeployer`；真实执行 plan/deploy/verify/teardown 时才加载。真实真机部署仍需目标环境安装 requests（运行时依赖，非 install 强制）。
2. **dry_run 语义按「本次调用」生效**：`deploy(dry_run=False)` 会设置 `self.dry_run = dry_run`，使 executable:false 检查基于本次操作模式而非构造时模式（修正初版 bug）。

## 6. 已知限制

- 真实 `EnvironmentDeployer` 运行依赖 requests/telnetlib 等运行时组件（fw_configurator/switch_configurator）；当前环境未安装 requests，未做真实 deployer dry-run 端到端（以 mock 覆盖 gate/推导/调用契约，S10 层 B 标注 N/A 不默认执行真机）。
- MVP 广播域推导：单 TG + 单 DUT 直连 p2p 域；含 SW（L2）拓扑的广播域推导需额外处理（不在本 technical-note 范围）。

## 7. 后续交接

- 交付给 meta-qa：`deploy_bridge.py` + 8 单测。
- S8 消费：ptm-te 编排真机部署可选调 S9 `DeployBridge`（S9 `--execute` 需 DQ-037-04 独立授权）。
- S10 集成：`topo-plan --execute --authorized` → env-file → `DeployBridge.deploy(dry_run=True)` 打印将下发配置（层 B 真机 `--execute` 标注 N/A 不默认执行）。

## 8. 结论

编码完成，CP6 PASS。等待 meta-qa CP7 验证。
