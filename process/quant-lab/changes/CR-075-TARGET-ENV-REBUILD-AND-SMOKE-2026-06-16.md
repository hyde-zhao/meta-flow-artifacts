---
cr_id: "CR-075"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "涉及 target .venv 重建、Python dependency sync、网络依赖解析和本地 runtime smoke；必须保留 CP2/CP3/CP5 人工门禁。"
rollback_to: "CR074 logical root authority marker"
approval_result: "approved"
created_at: "2026-06-16T23:31:26+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-16T23:50:58+08:00"
closed_by: "user"
closed_at: "2026-06-17T00:18:01+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-073, CR-074"
source_checkpoint: "process/checks/CR074-ROOT-CUTOVER-VERIFICATION-2026-06-16.md"
source_decision_id: "CR075-candidate"
follow_up_type: "target-env-rebuild-and-smoke"
risk_class: "local-runtime-env-rebuild"
owner: "host-orchestrator"
revisit_condition: "用户要求访问 data/reports、读取 .env 或凭据、provider fetch、lake write、catalog publish、远端 Git 写入、QMT/MiniQMT runtime、CR046 recovery、系统级 shell/IDE/service 修改、或扩大到 full test suite 时重访。"
acceptance_criteria: "CP2/CP3/CP5 approved 后，才允许在 /home/hyde/workspace/quant-lab 中执行 target 基础依赖同步和最小 smoke；验证范围仅限 uv sync、包导入和 package/import layout 小回归；不读取 .env，不触碰 reports/data，不执行 provider fetch/lake write/catalog publish，不启动交易 runtime。"
close_condition: "target env rebuild 和最小 smoke 均有 PASS/BLOCKED 证据；若失败，记录 remediation，不自动清理 .venv 或扩大执行范围。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-075 Target Env Rebuild and Smoke

## 变更描述

用户要求启动 `CR075`。该候选项来自 CR073 / CR074 后续事项台账，目标是在已经完成 logical root cutover marker 后，对 authoritative target root `/home/hyde/workspace/quant-lab` 做最小运行环境重建和 smoke 验证。

本 CR 只启动审批与执行门禁，不在创建时执行 `.venv` 重建、`uv sync`、测试、凭据读取、data/reports 访问、provider fetch、lake write、catalog publish、远端 Git 写入、QMT/MiniQMT runtime 或 CR046 recovery。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-073, CR-074` |
| 来源检查点 | `process/checks/CR074-ROOT-CUTOVER-VERIFICATION-2026-06-16.md` |
| 来源决策 ID | `CR075-candidate` |
| follow-up 类型 | target env rebuild and smoke |
| 风险等级 | high |
| owner | host-orchestrator |
| 重访条件 | 访问 data/reports、凭据、外部接口、远端 Git、交易 runtime、CR046 recovery 或扩大 smoke 范围时重访 |
| 验收标准 | 三门 approved 后执行基础依赖同步、包导入和 package/import layout 小回归 |
| 关闭条件 | smoke PASS 或 BLOCKED 证据完整，失败不自动清理或扩大范围 |

## CR 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 与 CR046 影响面重叠 | false | CR046 是 user-paused QMT/MiniQMT 双目标策略交付框架；CR075 不连接 QMT/MiniQMT、不查询账户、不下单、不恢复 CR046 CP7。 |
| 与 CR073 影响面重叠 | continuation | CR073 只做到 target evidence convergence 和 cutover readiness；CR075 承接其 follow-up，不重新同步 evidence。 |
| 与 CR074 影响面重叠 | continuation | CR074 完成 logical root marker，并明确 env rebuild 属于 CR075；本 CR 正式接管该候选。 |
| 与 CR076 影响面重叠 | false | CR076 处理 data/reports access policy；CR075 不读取、复制、恢复或比对 reports/data。 |
| 与 CR077 影响面重叠 | false | CR075 不删除、重命名、移动旧根。 |
| 与 CR078 影响面重叠 | false | CR075 不做远端 Git、branch/default branch、push/tag/set-url。 |
| 是否允许启动 | true | 用户明确发送“启动CR075”；当前 CR tracking 预检 PASS，候选可转为正式 CR。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-075-TARGET-ENV-REBUILD-AND-SMOKE-2026-06-16.md` | 新增 | N/A | 本文件 | pending |
| `process/context/CP2-CR075-ENV-REBUILD-CONTEXT.yaml` | 新增 | N/A | capsule | pending |
| `process/context/CP3-CR075-ENV-SMOKE-DESIGN-CONTEXT.yaml` | 新增 | N/A | capsule | pending |
| `process/context/CP5-CR075-ENV-SMOKE-READINESS-CONTEXT.yaml` | 新增 | N/A | capsule | pending |
| `process/checks/CP2-CR075-ENV-REBUILD-BASELINE.md` | 新增 | N/A | 自动预检 | pending |
| `process/checks/CP3-CR075-ENV-SMOKE-DESIGN-CONSISTENCY.md` | 新增 | N/A | 自动预检 | pending |
| `process/checks/CP5-CR075-ENV-SMOKE-READINESS.md` | 新增 | N/A | 自动预检 | pending |
| `process/checkpoints/CP2-CR075-ENV-REBUILD-BASELINE.md` | 新增 | N/A | 人工审查稿 | pending |
| `process/checkpoints/CP3-CR075-ENV-SMOKE-DESIGN-REVIEW.md` | 新增 | N/A | 人工审查稿 | pending |
| `process/checkpoints/CP5-CR075-ENV-SMOKE-READINESS.md` | 新增 | N/A | 人工审查稿 | pending |
| `process/checks/CP2-CR075-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP2 人工门禁发起消息 | pending |
| `process/checks/CP3-CR075-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP3 人工门禁发起消息 | pending |
| `process/checks/CP5-CR075-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP5 人工门禁发起消息 | pending |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR073 / CR074 / CR076-CR078 follow-up 关系 | `active_crs` / `follow_up_candidates` | pending |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused、CR073 readiness、CR074 root marker | frontmatter / `cr_tracking` / `human_gate_decisions` | pending |
| `process/changes/CR-073-TARGET-ROOT-CONVERGENCE-CUTOVER-READINESS-GATE-2026-06-16.md` | 原文档更新 | 保留原后续台账行并转 active | `## 后续事项台账` | pending |
| `process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md` | 原文档更新 | 保留原后续台账行并转 active | `## 后续事项台账` | pending |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR073 cutover readiness | CR075 env rebuild gate | 原证据保留 | CR075 不重跑 scoped sync，只消费 target readiness。 |
| CR074 logical root marker | CR075 target runtime readiness | 原 marker 保留 | CR075 在 authoritative root 上做运行环境验证，不回退 root authority。 |
| CR075-candidate | CR-075 formal CR | 台账状态转 active | 详细范围进入正式 CR，台账只保留索引字段。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | target runtime readiness | true | 新增 CR075 范围基线；不修改产品需求正文。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | env rebuild / import smoke / package layout smoke | true | 新增最小 smoke 场景；full tests 和 data/report smoke 不纳入。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CP2/CP3/CP5 -> env rebuild -> smoke verification | true | 标准模式；approved 前不得执行 `uv sync` 或 smoke。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | dependency sync、runtime smoke、forbidden path boundary | true | 禁止 `.env`、reports/data、provider/lake/catalog、remote Git、QMT/MiniQMT。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | STATE、CR-INDEX、checkpoint/context evidence、smoke evidence | true | 生成 CR075 门禁；批准后仅写 CR075 execution / verification evidence。 |

## 回退决策

- 影响范围：authoritative target root 的本地 Python runtime 环境。
- 回退到阶段：`CR074 logical root authority marker`。
- 需要重新确认的对象：dependency sync 范围、smoke 命令、失败处理、是否允许删除或重建 `.venv`。
- 回退行为：批准前 reject 即不执行；批准后失败只记录 BLOCKED，不自动删除 `.venv`，不自动扩大测试或访问 data/reports。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 会写入 target `.venv` 并运行本地 Python smoke。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 触碰此前 CR074 明确不授权的 env rebuild / runtime smoke。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不改代码接口或 Story owner。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要冻结 smoke 方案、失败路径、网络/凭据/数据边界。 |
| 是否保持 fast-lane | false | 使用 standard CP2/CP3/CP5。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- 替代门禁：`CP5-CR075-ENV-SMOKE-READINESS`
- 理由：CR075 是本机运行环境重建和 smoke 执行门禁，不修改代码、接口、Schema、Story 实现或文件 owner；使用 CP5 readiness 替代 Story LLD 批次。
- 执行启动条件：
  - [x] `process/checkpoints/CP2-CR075-ENV-REBUILD-BASELINE.md` 结论 `approved`
  - [x] `process/checkpoints/CP3-CR075-ENV-SMOKE-DESIGN-REVIEW.md` 结论 `approved`
  - [x] `process/checkpoints/CP5-CR075-ENV-SMOKE-READINESS.md` 结论 `approved`
  - [ ] CR tracking PASS
  - [x] 不授权项仍未被执行

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR075 与门禁 | 用户“启动CR075”、CR073/CR074 follow-up | 本 CR、contexts、checks、checkpoints | 不执行 env/smoke | 发起 CP2/CP3/CP5 |
| 2 | `host-orchestrator` | 发起 CP2/CP3/CP5 人工确认 | 自动预检、Decision Brief | 三份 checkpoint、launch message | 等待用户 `approve` | 通过后才执行 env rebuild |
| 3 | `host-orchestrator` | target env rebuild | CP2/CP3/CP5 approved、preflight PASS | CR075 execution evidence | 不读取 `.env` / reports / data | 进入 smoke |
| 4 | `host-orchestrator` | 最小 smoke | target `.venv`、基础依赖 | CR075 verification evidence | 不做 provider/lake/catalog/runtime | CP8 / 收敛 |

## 设计方案

### 推荐方案：基础依赖同步 + 最小 import smoke

批准后在 `/home/hyde/workspace/quant-lab` 执行基础依赖同步，并运行最小 smoke：

1. `uv sync --python 3.11`
2. `uv run --python 3.11 python -c "import quant_lab; print(quant_lab.__version__)"`
3. `uv run --python 3.11 --group dev pytest -q tests/test_cr061_package_import_layout.py`

推荐范围刻意不包含 `--group tushare`、`--group exploration`、`--group backtrader`、`--group jqdata`、`--group ml`、full test suite、data/reports access、provider fetch、lake write、catalog publish 或交易 runtime。

### 备选方案

| Option | Pros | Cons | When to switch |
|---|---|---|---|
| 只做 dry-run / 不执行 env rebuild | 零 runtime 写入 | target 仍非 runtime-ready | 用户不希望创建 `.venv` |
| 基础依赖 + dev smoke（推荐） | 可证明 target 最小可运行 | 会写入 `.venv` 并下载依赖 | 用户批准 CP2/CP3/CP5 |
| 扩大到 optional groups 或 full tests | 覆盖更广 | 风险和耗时更高，可能触碰 provider/data 边界 | 另起或修改 CR075 决策 |

## 不授权项

| ID | 不授权内容 |
|---|---|
| NA-CR075-001 | 读取、打印、记录、复制或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实 |
| NA-CR075-002 | 读取、复制、恢复、比对或删除 `reports/`、`data/` |
| NA-CR075-003 | provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement |
| NA-CR075-004 | 远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url |
| NA-CR075-005 | QMT / MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live |
| NA-CR075-006 | 删除、重命名或移动旧根 `/home/hyde/workspace/local_backtest` |
| NA-CR075-007 | 自动恢复或推进 CR046 |
| NA-CR075-008 | 未经额外确认删除 `.venv`、执行 destructive cleanup、扩大到 optional dependency groups 或 full test suite |

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：本 CR §后续事项台账
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`python3 -B scripts/check_cr_tracking_consistency.py --project-root .`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR076-candidate | Data/reports access policy | candidate | CR | 2 |  | source=CR073/CR074 | not-started | reports/data 不在 CR075 范围 | 后续决定 NAS / local retained / rebuild 策略 |
| CR077-candidate | Old root retirement policy | candidate | CR | 3 |  | source=CR073/CR074 | not-started | 旧根删除/移动是破坏性动作 | target runtime readiness 后再评审 |
| CR078-candidate | Remote Git governance after cutover | candidate | CR | 4 |  | source=CR073/CR074 | not-started | 远端写入不在 CR075 范围 | 后续独立 gate |

## 执行与验证结果

| 项 | 命令 / 范围 | 结果 | 证据 |
|---|---|---|---|
| 执行前 CR tracking preflight | `python3 -B scripts/check_cr_tracking_consistency.py --project-root .` | PASS | 当前执行日志 |
| target env rebuild | `uv sync --python 3.11` | PASS | `process/checks/CR075-ENV-REBUILD-EXECUTION-2026-06-16.md` |
| import smoke | `uv run --python 3.11 python -c "import quant_lab; print(quant_lab.__version__)"` | PASS，输出 `0.1.0` | `process/checks/CR075-ENV-SMOKE-VERIFICATION-2026-06-16.md` |
| CR061 package layout 小回归 | `uv run --python 3.11 --group dev pytest -q tests/test_cr061_package_import_layout.py` | PASS，`4 passed in 0.03s` | `process/checks/CR075-ENV-SMOKE-VERIFICATION-2026-06-16.md` |
| 未授权项 | `.env`、reports/data、provider/lake/catalog、远端 Git、QMT/MiniQMT runtime、CR046 recovery、optional groups、full tests、destructive cleanup | 未执行 | `process/checks/CR075-ENV-SMOKE-VERIFICATION-2026-06-16.md` |

结论：CR075 target env rebuild 和最小 smoke 已 PASS；关闭本 CR 前仍需 CP8 / closure review。该结果不等同于 data/reports、provider/lake/catalog、交易 runtime 或 CR046 recovery 就绪。

## CP8 收尾门禁

| 项 | 结果 | 证据 |
|---|---|---|
| CP8 自动预检 | PASS_WITH_RISK | `process/checks/CP8-CR075-DELIVERY-READINESS.md` |
| CP8 人工审查稿 | approved | `process/checkpoints/CP8-CR075-DELIVERY-READINESS.md` |
| Release Context | READY_WITH_RISK / minimal | `process/release/RELEASE-CONTEXT-CR075.yaml` |
| Human Gate Launch | PASS | `process/checks/CP8-CR075-HUMAN-GATE-LAUNCH-MESSAGE.md` |

用户于 2026-06-17T00:18:01+08:00 回复“同意。处理完成CR075后继续处理076”，按 CP8 `approve` 处理；CR075 关闭为 `closed-current-delivery / READY_WITH_RISK`。

## 处理结论

- 审批结论：approved（用户回复“同意”，2026-06-16T23:50:58+08:00）
- 执行结论：PASS（target env rebuild、import smoke、CR061 package layout 小回归均通过）
- [ ] 自动批准（低风险）
- [x] CP8 人工确认已通过（READY_WITH_RISK）
- [x] 人工审批已通过（仅限 CR075 推荐方案）
- 关闭结论：closed-current-delivery / READY_WITH_RISK（用户回复“同意。处理完成CR075后继续处理076”，2026-06-17T00:18:01+08:00）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | `CR-073` | target evidence convergence 和 cutover readiness 来源。 |
| CR | `CR-074` | logical root authority marker 来源。 |
| Target root | `/home/hyde/workspace/quant-lab` | CR075 执行目标。 |
| Legacy root | `/home/hyde/workspace/local_backtest` | 继续保留；CR075 不删除、不移动、不重命名。 |
