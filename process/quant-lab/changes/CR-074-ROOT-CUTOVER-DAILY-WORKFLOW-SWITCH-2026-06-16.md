---
cr_id: "CR-074"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "涉及 authoritative root / 日常工作根语义切换、旧根保留边界和后续会话入口，必须保留 CP2/CP3/CP5 人工门禁。"
rollback_to: "CR073 cutover readiness baseline"
approval_result: "approved"
created_at: "2026-06-16T22:14:44+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-16T22:41:34+08:00"
closed_by: "user"
closed_at: "2026-06-17T15:46:35+08:00"
release_decision: "READY_WITH_RISK"
execution_result: "closed-current-delivery-ready-with-risk"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-073"
source_checkpoint: "process/checks/CR073-TARGET-CUTOVER-READINESS-VERIFICATION-2026-06-16.md"
source_decision_id: "CR074-candidate"
follow_up_type: "root-cutover-daily-workflow-switch"
risk_class: "local-root-authority-switch"
owner: "host-orchestrator"
revisit_condition: "用户要求真正删除/重命名/移动旧根、复制 reports/data/.venv/node_modules/.env、读取凭据、重建 target env、远端 Git 写入、data lake 切换、QMT/MiniQMT runtime、CR046 recovery、系统级 symlink / shell / IDE / cron / service 修改时重访。"
acceptance_criteria: "CP2/CP3/CP5 approved 后，才允许把 /home/hyde/workspace/quant-lab 标记为后续 authoritative daily workflow root；旧根 /home/hyde/workspace/local_backtest 保留为 legacy root；不删除/移动旧根，不复制 forbidden paths，不读取凭据，不远端 Git 写入，不触发 runtime；验证 target CR tracking PASS、CR073 evidence 完整和 forbidden paths 为空。"
close_condition: "logical root cutover evidence 完成并通过静态验证；不执行 old root retirement、env rebuild、data/reports access 或 remote Git governance。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-074 Root Cutover / Daily Workflow Switch

## 变更描述

用户回复“继续”，承接 CR073 的后续候选 `CR074-candidate`。本 CR 目标是定义并审查“逻辑 root cutover”：在 CR073 已让 `/home/hyde/workspace/quant-lab` 具备 cutover readiness 后，确认是否把 target 标记为后续日常工作根和 authoritative root。

本轮不直接切换当前会话工作目录，不修改 shell / IDE / cron / service，不删除、重命名或移动旧根，不恢复数据/报告/env，不做远端 Git 写入，不触发 data lake、QMT/MiniQMT runtime 或 CR046 recovery。

## CR 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 与 CR073 影响面重叠 | continuation | CR073 已 `active-cutover-readiness-achieved`，CR074 承接其 follow-up candidate；不重新同步 forbidden paths。 |
| 与 CR046 影响面重叠 | false | CR046 仍是 user-paused runtime 线；CR074 不触发 QMT/MiniQMT、账户、订单、simulation/live。 |
| 与 CR075 / CR076 重叠 | false | CR074 不重建 target env、不恢复 data/reports。 |
| 与 CR077 重叠 | false | CR074 保留旧根，不做 old root retirement。 |
| 与 CR078 重叠 | false | CR074 不做 remote Git governance、push、branch/default branch。 |
| 是否允许启动 | true | 用户“继续”视为启动下一个候选 CR 的明确指令；实际 root cutover 仍需 CP2/CP3/CP5 approve。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md` | 新增 | N/A | 本文件 | approved |
| `process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/context/CP5-CR074-CUTOVER-READINESS-CONTEXT.yaml` | 新增 | N/A | capsule | approved |
| `process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | 新增 | N/A | 自动预检 | approved |
| `process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md` | 新增 | N/A | 自动预检 | approved |
| `process/checks/CP5-CR074-CUTOVER-READINESS.md` | 新增 | N/A | 自动预检 | approved |
| `process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checkpoints/CP5-CR074-CUTOVER-READINESS.md` | 新增 | N/A | 人工审查稿 | approved |
| `process/checks/CP2-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP2 人工门禁发起消息 | approved |
| `process/checks/CP3-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP3 人工门禁发起消息 | approved |
| `process/checks/CP5-CR074-HUMAN-GATE-LAUNCH-MESSAGE.md` | 新增 | N/A | CP5 人工门禁发起消息 | approved |
| `process/checks/CR074-ROOT-CUTOVER-EXECUTION-2026-06-16.md` | 新增 | N/A | execution evidence | approved |
| `process/checks/CR074-ROOT-CUTOVER-VERIFICATION-2026-06-16.md` | 新增 | N/A | verification evidence | approved |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused、CR073 readiness 基线 | frontmatter / history | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR073 / CR075-CR078 follow-up 关系 | `active_crs` / `follow_up_candidates` | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| `/home/hyde/workspace/local_backtest` | legacy root | 原目录保留 | CR074 不删除、不移动、不重命名旧根。 |
| `/home/hyde/workspace/quant-lab` | authoritative root candidate | CR073 evidence 保留 | CR074 只在批准后把 target 标记为后续日常工作根。 |
| CR073 cutover readiness | CR074 root cutover gate | 原证据保留 | CR074 消费 CR073 验证结果，不重新扩大同步范围。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | 日常工作根语义 | true | 新增 CR074 范围基线；不改产品需求正文。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | root cutover 静态验证 | true | 新增 target authoritative / legacy root retained / forbidden paths 检查场景。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CP2/CP3/CP5 -> logical cutover -> validation | true | 标准模式；CP2/CP3/CP5 approve 前不得执行 root cutover。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | 工作根、旧根、forbidden paths、外部写入 | true | 不授权 destructive / credential / runtime / remote Git；真实切换前需人工确认。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | STATE、CR-INDEX、checkpoint/context evidence | true | 生成 CR074 门禁；后续批准后只写逻辑 cutover evidence。 |

## 回退决策

- 影响范围：高风险逻辑根切换；不含物理删除或移动。
- 回退到阶段：`CR073 cutover readiness baseline`。
- 需要重新确认的对象：root authority、legacy root policy、forbidden paths、target readiness、rollback wording。
- 回退行为：批准前 reject 即不切换；批准后如需回退，旧根仍保留，可通过后续 CR 把 authoritative marker 切回旧根。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 日常工作根语义切换会影响后续全部操作入口。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 涉及 authoritative root 和旧根边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不改代码接口，不进入 Story 实现。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要冻结 cutover 方案、失败路径和回退策略。 |
| 是否保持 fast-lane | false | 使用 standard CP2/CP3/CP5。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- 替代门禁：`CP5-CR074-CUTOVER-READINESS`
- 理由：CR074 是本机治理和状态切换门禁，不修改代码、接口、Schema 或 Story 实现；使用 CP5 readiness 替代 Story LLD 批次。
- 启动条件：
  - [x] `process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md` 结论 `approved`
  - [x] `process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md` 结论 `approved`
  - [x] `process/checkpoints/CP5-CR074-CUTOVER-READINESS.md` 结论 `approved`
  - [x] target CR tracking PASS
  - [x] target forbidden paths checks 为空

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR074 与门禁 | 用户“继续”、CR073 readiness | 本 CR、contexts、checks、checkpoints | 不切 root | 发起 CP2/CP3/CP5 |
| 2 | `host-orchestrator` | 发起 CP2/CP3/CP5 人工确认 | 自动预检、Decision Brief | 三份 checkpoint、launch message | 等待用户 `approve` | 通过后才执行逻辑 cutover |
| 3 | `host-orchestrator` | logical root cutover | CP2/CP3/CP5 approved、preflight PASS | root authority evidence / STATE / CR-INDEX | 不改 shell/IDE/service，不删旧根 | 验证 target 和旧根边界 |
| 4 | `host-orchestrator` | cutover readiness verification | target / legacy root / Git 状态 | CR074 verification evidence | forbidden paths 为空 | CR074 可进入收敛 / CP8 |

## 设计方案

### 推荐方案：逻辑 cutover marker

批准后仅把 `/home/hyde/workspace/quant-lab` 标记为后续 authoritative daily workflow root，并保留 `/home/hyde/workspace/local_backtest` 为 legacy root。当前会话不自动 `cd`，不改 shell / IDE / cron / service；后续会话按新 root 启动。

### 备选方案

| Option | Pros | Cons | When to switch |
|---|---|---|---|
| 只发出人工操作说明 | 零自动写入 | 容易遗漏状态证据 | 用户不希望任何 root authority 文件变更 |
| 系统级 symlink / shell 配置切换 | 对用户透明 | 破坏性和平台副作用高 | 另起高风险 runtime/ops CR |
| 继续停留旧根 | 风险最低 | target readiness 无法转化为日常入口 | 用户 reject CR074 |

## 不授权项

| ID | 不授权内容 |
|---|---|
| NA-CR074-001 | 当前会话自动切换 cwd、修改 shell / IDE / cron / service / systemd 配置 |
| NA-CR074-002 | 删除、重命名或移动 `/home/hyde/workspace/local_backtest` |
| NA-CR074-003 | 复制、恢复、读取或比对 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env` |
| NA-CR074-004 | 读取、打印、记录或迁移 token、密码、cookie、session、private key、账户或交易事实 |
| NA-CR074-005 | target env rebuild、Python dependency sync、smoke runtime |
| NA-CR074-006 | 远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url |
| NA-CR074-007 | NAS 新写入 / 删除 / promote，data lake root switch、provider fetch、lake write、catalog publish |
| NA-CR074-008 | QMT / MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live |
| NA-CR074-009 | 自动恢复或推进 CR046 |

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：本 CR §后续事项台账
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`python3 -B scripts/check_cr_tracking_consistency.py --project-root .`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR075-candidate | Target env rebuild and smoke | active | CR | 1 | `process/changes/CR-075-TARGET-ENV-REBUILD-AND-SMOKE-2026-06-16.md` | source=CR073/CR074 | CP2/CP3/CP5 human gate | env rebuild / smoke 仍需人工门禁 approve | 已启动正式 CR075，等待 CP2/CP3/CP5 审查 |
| CR076-candidate | Data/reports access policy | candidate | CR | 2 |  | source=CR073/CR074 | not-started | reports/data 不复制 | 后续决定 NAS / local retained / rebuild 策略 |
| CR077-candidate | Old root retirement policy | candidate | CR | 3 |  | source=CR073/CR074 | not-started | 旧根删除/移动是破坏性动作 | target authoritative 后再评审 |
| CR078-candidate | Remote Git governance after cutover | candidate | CR | 4 |  | source=CR073/CR074 | not-started | 远端写入不在 CR074 范围 | 后续独立 gate |

## 处理结论

- 审批结论：closed-current-delivery / READY_WITH_RISK
- 当前结论：CR074 logical root authority marker 已达成当前交付目标。用户于 2026-06-17T15:46:35+08:00 回复“同意”，批准 CR086 ledger-only 收敛，将 CR074 从 achieved 型 active 状态收敛为当前交付关闭；后续旧根退役、data/reports、runtime、remote archive / governance 和 CR046 recovery 均需独立授权。
- [ ] 自动批准（低风险）
- [ ] 待人工确认（中风险）
- [x] 人工审批已通过（高风险）

## CR086 收敛结果

CR086 仅对过程台账做状态收敛，不执行当前会话 cwd 切换、shell / IDE / cron / service 修改、旧根删除 / 移动、forbidden paths 复制 / 读取、凭据、target env rebuild、远端 Git、NAS、data lake、QMT / MiniQMT runtime 或 CR046 recovery。CR074 当前收敛为 `closed-current-delivery / READY_WITH_RISK`；未来若需要 old root retirement、data/runtime 或 remote archive / governance，必须另起独立 CR。

## 执行与验证结果

| 项目 | 结果 | 证据 | 说明 |
|---|---|---|---|
| 用户审批 | PASS | `process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md` / `process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md` / `process/checkpoints/CP5-CR074-CUTOVER-READINESS.md` | 用户回复“同意”，按 `approve` 处理。 |
| 执行前 preflight | PASS | `process/checks/CR074-ROOT-CUTOVER-EXECUTION-2026-06-16.md` | 本地和 target CR tracking PASS；target CR073 evidence 存在；target forbidden path checks 为空。 |
| logical root authority marker | PASS | `process/STATE.md.root_authority` / `process/changes/CR-INDEX.yaml.root_authority` | `/home/hyde/workspace/quant-lab` 标记为后续 authoritative daily workflow root；旧根保留。 |
| target evidence sync | PASS | `process/checks/CR074-ROOT-CUTOVER-EXECUTION-2026-06-16.md` | scoped rsync，不使用 `--delete`，不包含 `.git`、`reports/`、`data/`、`.venv/`、`node_modules/`、`.env`。 |
| 执行后验证 | PASS | `process/checks/CR074-ROOT-CUTOVER-VERIFICATION-2026-06-16.md` | target marker、CR tracking 和 forbidden path checks 通过。 |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | `CR-073` | target 已达到 cutover readiness。 |
| Target root | `/home/hyde/workspace/quant-lab` | CR074 候选 authoritative root。 |
| Legacy root | `/home/hyde/workspace/local_backtest` | CR074 内继续保留。 |
