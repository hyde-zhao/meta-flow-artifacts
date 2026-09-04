---
source_cr: "CR-037"
status: "candidate-tracking"
created_at: "2026-08-05T20:10:00+08:00"
release_decision: "READY_WITH_RISK"
---

# CR-037 后续跟踪台账

> 本文件只记录候选后续工作，**不创建新的 active CR**，不授权任何运行时操作。任一条目仅在用户确认范围、授权和交付目标后转为独立 CR；在此之前不得执行网络请求、写入或生产操作。

## 汇总

| ID | 类型 | 候选事项 | 触发条件 | 状态 |
|---|---|---|---|---|
| CR-037-FU-R2R4 | runtime_authorization | `--execute` 真机下发端到端验证（S9 真机 deployer 依赖 requests 等运行时组件，mock 覆盖未真实端到端） | 用户单独授权 DQ-037-04 + 真实设备可达 + 凭据环境变量 | candidate |
| CR-037-FU-R3 | follow_up_tracking | MVP 边界扩展：多节点 / SW / Mock 拓扑映射导出（当前显式 TOPO_NODE_COUNT_UNSUPPORTED） | 用户决定扩展 MVP 范围 | candidate |
| CR-037-FU-R5 | follow_up_tracking | platform_alias.yaml 与 device-reference.md 同步维护（上游漂移） | 上游 device-reference.md 更新时 | candidate |
| CR-037-FU-M1 | follow_up_tracking | 参考用例 `devices_direct.yaml` TG 设备组与池 TG 节点关联失败（cosmetic 告警，产物正确） | 需要参考用例演示完整 TG 关联路径 | candidate |
| CR-037-FU-M2 | follow_up_tracking | `script/ptm_team/install.py`（包级入口）补齐 `ensure_target_project_templates`（模板副本不对称） | 需要两安装入口一致的模板副本生成 | candidate |
| CR-037-FU-I1 | follow_up_tracking | STATE.current.json active_change 同步 CR-037 + AGENT-DISPATCH-LEDGER 回补 meta-dev CP6 dispatch 记录 | CR-037 关闭时由 host-orchestrator 执行 | candidate |

## 转换规则

任一候选仅在用户确认范围、授权和交付目标后转为独立 CR；在此之前不得执行网络请求、写入或生产操作。状态取值：`candidate` / `active` / `blocked` / `spike_candidate` / `converted-to-spike` / `closed` / `cancelled` / `superseded`。

## 详细条目

### CR-037-FU-R2R4：`--execute` 真机下发端到端验证
- **背景**：CR-037 `validation_mode=mixed`，层A fixture dry-run 全通过；`--execute` 真机下发 N/A（DQ-037-04 独立 runtime_authorization，不可 waive）。S9 `DeployBridge` 真实 EnvironmentDeployer 依赖 requests 等运行时组件，未做真实 deployer 端到端（mock 覆盖）。
- **范围**：`topo-plan --execute --authorized` 真机下发（NGFW Web API 写 + 台账占用写回 + 真机链路），真实设备（海光3250 + TE 端口）+ 真实接线 + 凭据环境变量。
- **前置**：DQ-037-04 用户独立授权 + 设备可达 + 凭据环境变量 + 层B P-2 真实池数据就绪。
- **关闭条件**：真机端到端 PASS / PASS_WITH_RISK（含 deployer 运行时依赖验证 + ledger 占用/释放闭环 + env-file 产物校验）。
- **风险关联**：R2（--execute N/A）+ R4（S9 运行时依赖）。

### CR-037-FU-R3：MVP 边界扩展（多节点/SW/Mock）
- **背景**：HLD v0.4 §1.3 MVP 边界 = 单 TG + 单 DUT 直连；多节点（≥2 DUT）/ SW / Mock / PC 拓扑在 exporter 显式 `TOPO_NODE_COUNT_UNSUPPORTED`。
- **范围**：扩展 exporter `_validate_mvp_topology` + 对应映射/导出支持多节点拓扑；新增用例覆盖。
- **触发条件**：12 个标准拓扑中需支持 node2 之外（多 DUT / SW / PC）拓扑。
- **关闭条件**：多节点拓扑映射导出 PASS + 对应单测/集成用例通过。

### CR-037-FU-R5：platform_alias.yaml 同步维护
- **背景**：S2 `platform_alias.yaml` 派生自 manaul `device-reference.md`（13 硬件系列）；上游 device-reference.md 更新时需同步维护，否则别名漂移。
- **范围**：当 device-reference.md 变更型号/别名/消歧规则时同步 platform_alias.yaml 并更新测试。
- **触发条件**：上游 device-reference.md 更新。
- **关闭条件**：platform_alias.yaml 与 device-reference.md 一致 + S2 测试全绿。

### CR-037-FU-M1：参考用例 devices_direct.yaml TG 关联 fixture 修正
- **背景**：CP7 独立复核发现参考用例 CLI dry-run 输出「未匹配 devices.yaml 节点: tg-dut1」——`devices_direct.yaml` TG 组仅 `tg.host`（无 `tg.api_server`）；池 TG 节点仅 `management.api_server`（无 `management.host`）。S3-C 关联主键 api_server 归一化精确匹配，host 兜底需池侧 `management.host`，两者均不满足 → 关联失败。
- **影响**：TG 仍从池数据直接可用，映射产物正确；告警仅 cosmetic；参考用例未覆盖 TG api_server 主键关联成功路径（S3 单测已覆盖）。
- **范围**：`skills/topo-planning/tests/fixtures/devices_direct.yaml` 的 tg-dut1 组增加 `api_server: "http://10.113.52.253:8450"`（或池节点补 `management.host`）。
- **关闭条件**：重跑 `topo_plan_cli.py plan --case-file case_ipv4_policy_route.md --dry-run` 输出无「未匹配 devices.yaml 节点: tg-dut1」告警。
- **来源**：docs/quality/REVIEW.md Minor-1 / FIXES.md M1。

### CR-037-FU-M2：ptm_team/install.py 补齐 ensure_target_project_templates
- **背景**：主安装器 `script/install.py` 具备模板副本生成；包级兼容入口 `script/ptm_team/install.py` 只扩展 PTM_TE_SKILLS 5→7 + copy_skill_tree 裁剪，无 `ensure_target_project_templates`。
- **影响**：经包级入口安装时目标项目不自动生成 `topology/physical_pool.yaml` + `physical_ledger.yaml` 模板副本（用户需手动复制或走主安装器）。
- **范围**：将 `ensure_target_project_templates` 提取为共享函数并在两安装入口复用；补测试。
- **关闭条件**：两入口均跑 install dry-run，确认模板副本 DryRun 行均出现。
- **来源**：docs/quality/REVIEW.md Minor-2 / FIXES.md M2。

### CR-037-FU-I1：STATE 同步 + dispatch ledger 回补
- **背景**：`STATE.current.json.active_change` 在 CP7 时为 "CR-036"，与 CR-037 不一致；AGENT-DISPATCH-LEDGER 缺 meta-dev CP6 dispatch 记录。
- **处理**：CR-037 关闭时由 host-orchestrator 运行 `meta-flow cr status-sync --id CR-037 --status closed --project-root .` 同步 STATE.current.json / CR-INDEX / CR-LEDGER；回补 CP6 dispatch 记录。
- **关闭条件**：`STATE.current.json.active_change` 指向 CR-037（或空）且与 CP result 一致；AGENT-DISPATCH-LEDGER 含 meta-dev CP6 dispatch 事件。
- **来源**：docs/quality/FIXES.md I1 / REVIEW.md Info-1。
