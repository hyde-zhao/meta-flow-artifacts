---
checkpoint_id: "CP5"
checkpoint_name: "CR058 Relayout Gate Batch A LLD Batch"
type: "manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-14T13:59:00+08:00"
auto_check: "process/checks/CP5-CR058-S01-intake-conflict-and-scope-freeze-TECHNICAL-NOTE-IMPLEMENTABILITY.md"
launch_message: "process/checks/CP5-CR058-HUMAN-GATE-LAUNCH-MESSAGE.md"
auto_final_authorization: false
---

# CP5 CR058 Relayout Gate Batch A LLD Batch 人工检查点

## 自动预检摘要

| 项目 | 结论 |
|---|---|
| 自动预检 | PASS |
| 批次 | `CR058-RELAYOUT-GATE-BATCH-A` |
| 当前可确认内容 | S01/S05 technical-note、S02/S03/S04 full-lld、candidate / preserve-audit / rollback_ref gate。 |
| 当前阻断 | 无 LLD 缺口；仍不授权真实执行，进入实现需 CP5 人工确认且后续仍受 no-op 边界约束。 |
| 推荐决策 | approve CR058 Batch-A design evidence；不授权真实执行。 |
| 自动终验授权 | false |

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Feature matrix 已生成 | PASS | `docs/design/FEATURE-DESIGN-MATRIX-CR058.md` | S02-S04 标为 full-lld。 |
| Story 卡已生成 | PASS | `process/stories/CR058-S01..S05` | 全部 Story 卡存在。 |
| S01/S05 technical-note 可审 | PASS | CP5 S01/S05 auto checks | no-op 边界明确。 |
| S02/S03/S04 full-lld | PASS | CP5 S02/S03/S04 auto checks | 完整 LLD 已补齐。 |

## Checklist

| # | 检查项 | 推荐结果 | 用户审查 |
|---|---|---|---|
| 1 | 是否确认 CP5 当前只批准设计证据，不批准真实迁移执行 | accept | 通过 |
| 2 | 是否接受 S02/S03/S04 full-lld 设计证据 | accept | 通过 |
| 3 | 是否接受 candidate list schema 和 allowlist-first 策略 | accept | 通过 |
| 4 | 是否接受 preserve-audit allowlist 默认保护历史证据 | accept | 通过 |
| 5 | 是否接受 rollback_ref gate 为后续真实执行硬前置 | accept | 通过 |
| 6 | 是否确认 CP5 不授权 NAS/lake/runtime/git remote/凭据/CR046 恢复 | accept | 通过 |

## Exit Criteria

| 条目 | 通过条件 | 状态 |
|---|---|---|
| 用户回复 `approve` | 接受 Decision Brief 内全部推荐方案，确认 CR058 Batch-A 设计证据通过；仍不授权真实迁移、真实 rewrite 或外部操作。 | PASS |
| 用户回复 `修改: <具体修改点>` | host-orchestrator 按修改点更新相关 DQ / 文档并重新发起门禁。 | pending |
| 用户回复 `reject` | CR058 CP5 设计方向不通过，回到 Story / HLD 修订。 | pending |
| 人工审查结果回填 | 本文件“人工审查结果”填入用户选择、时间和处理结论。 | PASS |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| Feature Matrix | `docs/design/FEATURE-DESIGN-MATRIX-CR058.md` | ready |
| Development Plan | `process/DEVELOPMENT-PLAN-CR058.yaml` | ready |
| S01 Story | `process/stories/CR058-S01-intake-conflict-and-scope-freeze.md` | ready |
| S02 Story | `process/stories/CR058-S02-candidate-list-and-owner-classification-gate.md` / `process/stories/CR058-S02-candidate-list-and-owner-classification-gate-LLD.md` | ready |
| S03 Story | `process/stories/CR058-S03-preserve-audit-allowlist-and-sensitive-filter-gate.md` / `process/stories/CR058-S03-preserve-audit-allowlist-and-sensitive-filter-gate-LLD.md` | ready |
| S04 Story | `process/stories/CR058-S04-rollback-ref-and-preflight-evidence-gate.md` / `process/stories/CR058-S04-rollback-ref-and-preflight-evidence-gate-LLD.md` | ready |
| S05 Story | `process/stories/CR058-S05-mechanical-migration-dry-run-and-authorization-boundary.md` | ready |
| Human Gate Launch Message | `process/checks/CP5-CR058-HUMAN-GATE-LAUNCH-MESSAGE.md` | ready |

## Decision Brief

### 推荐决策

推荐用户回复 `approve` 以接受 CR058 Batch-A 的设计证据：S01/S05 technical-note 与 S02/S03/S04 full-lld 已具备，candidate list / preserve-audit / rollback_ref 是后续真实执行前硬门。此确认只批准 CR058 设计证据，不授权任何真实 move / rewrite、NAS、lake、git remote、runtime、provider、凭据读取或 CR046 恢复。

### 备选方案

| 方案 | 内容 | 优点 | 代价 / 风险 | 切换条件 |
|---|---|---|---|---|
| 推荐方案 A | 接受 CR058 Batch-A 设计证据，保持 no-op / no-real-execution 边界。 | 设计证据完整且不扩大授权。 | 后续真实执行仍需单独授权。 | 默认推荐。 |
| 备选方案 B | 要求继续修改 S02-S04 LLD。 | 可进一步收紧门禁。 | 延迟后续 CP6 文档实现。 | 用户发现 LLD 内容不足。 |
| 备选方案 C | reject，回到 HLD / Story 重拆。 | 可重设边界。 | 延迟 CR058。 | 用户不同意当前 Story 或门禁。 |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---|
| `docs/design/FEATURE-DESIGN-MATRIX-CR058.md` | scanned | 5 | 2 | lld_policy 和 dev_gate 纳入。 |
| `process/stories/CR058-S01..S05` | scanned | 5 | 3 | Story readiness 和 full-lld 设计证据纳入。 |
| CP5 auto checks | scanned | 5 | 3 | S02-S04 full-lld PASS 纳入。 |
| meta-se 建议 | scanned | 5 | 4 | full-lld 建议和 CP5 不授权项纳入。 |
| 合计 | scanned | 20 | 6 | 收敛为 CP5 六项 DQ。 |

## 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR058-01 | implementation | 是否确认 CP5 当前只批准设计证据，不批准真实迁移执行？ | 确认，CP5 后仍保持 no-real-execution。 | 允许 CP6 实迁；不推荐且当前禁止。 | 推荐方案避免误执行；备选越权。 | CP5 approve 不等于真实执行授权。 | 用户另行明确启动执行阶段并补 rollback_ref。 |
| DQ-CP5-CR058-02 | implementation | 是否接受 S02/S03/S04 full-lld 设计证据？ | 接受，候选清单、preserve-audit、rollback_ref 的 full-lld 已补齐。 | 要求继续修改 LLD。 | 推荐方案设计强度匹配风险；修改方案更保守但延迟。 | 接受后仍不得真实执行。 | 用户指出具体 LLD 缺口时回到修改。 |
| DQ-CP5-CR058-03 | architecture | 是否接受 candidate list schema 和 allowlist-first 策略？ | 接受字段覆盖率 100%，不允许 `execute_move`。 | repo-wide rewrite。 | 推荐方案可审查；备选风险高。 | 决定后续 candidate manifest。 | 候选范围扩大需重审。 |
| DQ-CP5-CR058-04 | risk_acceptance | 是否接受 preserve-audit allowlist 默认保护历史证据？ | 接受，历史证据默认不可机械替换。 | 单项移出 allowlist。 | 推荐方案保持审计链；单项移出需强理由。 | 历史 `local_backtest` 会保留。 | 单项移出必须列 path、owner、rollback_ref。 |
| DQ-CP5-CR058-05 | implementation | 是否接受 rollback_ref gate 为后续真实执行硬前置？ | 接受 commit + bundle + manifest + failure manifest。 | commit-only；不推荐。 | 推荐方案可回退；备选证据弱。 | 缺 ref 时 execute_allowed=false。 | 用户显式豁免 bundle 时记录风险接受。 |
| DQ-CP5-CR058-06 | runtime_authorization | 是否确认 CP5 不授权 NAS/lake/runtime/git remote/凭据/CR046 恢复？ | 确认。 | 单项授权；必须独立 CR。 | 推荐方案保持边界；备选需新门禁。 | 防止真实运行或外部写入风险。 | 用户另起 CR060+ / data lake / trading / git remote gate。 |

## 不授权项

如果你回复 approve，表示接受上述 6 项推荐方案；不表示授权以下操作：

| Item ID | 不授权操作 |
|---|---|
| NA-CP5-CR058-01 | 真实 repo-local file move / rename / delete / bulk rewrite |
| NA-CP5-CR058-02 | 改写历史 `process/**`、checks、checkpoints、handoffs、历史 Story / LLD / DEV-LOG |
| NA-CP5-CR058-03 | git history rewrite、remote rename、git push、tag、force push |
| NA-CP5-CR058-04 | `git reset --hard`、`git checkout --` 回退用户未明确要求的工作树 |
| NA-CP5-CR058-05 | NAS mount / scan / mkdir / copy / delete / migration |
| NA-CP5-CR058-06 | `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish |
| NA-CP5-CR058-07 | provider fetch、真实数据拉取、untracked data bulk scan |
| NA-CP5-CR058-08 | `.env`、token、password、cookie、session、private key、账户凭据读取 |
| NA-CP5-CR058-09 | QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live |
| NA-CP5-CR058-10 | 恢复 CR046 CP7 或推进 CR046 runtime verification |
| NA-CP5-CR058-11 | 从 CR053 继承任何运行授权 |

## 人工审查结果

| 字段 | 值 |
|---|---|
| 用户回复 | 同意 |
| 审查人 | user |
| 审查时间 | 2026-06-14T15:13:23+08:00 |
| 决策结果 | approved；接受 `DQ-CP5-CR058-01..06` 推荐方案，确认 CR058 Batch-A 设计证据通过，可进入 CP6 static-only / no-op 文档实现准备。 |
| 不授权项确认 | CP5 approve 不授权真实 repo-local file move / rename / delete / bulk rewrite，不授权 NAS / data lake / provider / git remote / 凭据 / QMT / MiniQMT runtime，不恢复 CR046。 |
| 后续处理 | host-orchestrator 将 CR058 状态推进到 `active-cp5-approved-ready-for-cp6-static`；后续 CP6 仅可产出静态文档 / no-op gate 证据。 |
