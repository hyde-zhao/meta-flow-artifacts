---
checkpoint_id: "CP3"
checkpoint_name: "CR058 HLD Review Human Gate"
type: "manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-14T13:59:00+08:00"
auto_check: "process/checks/CP3-CR058-HLD-CONSISTENCY.md"
launch_message: "process/checks/CP3-CR058-HUMAN-GATE-LAUNCH-MESSAGE.md"
auto_final_authorization: false
---

# CP3 CR058 HLD Review 人工检查点

## 自动预检摘要

| 项目 | 结论 |
|---|---|
| 自动预检 | PASS |
| HLD | `docs/design/HLD-CR058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE.md` |
| ADR | `docs/design/ARCHITECTURE-DECISION-CR058.md` |
| 推荐决策 | approve HLD / ADR；不授权真实执行。 |
| 自动终验授权 | false |

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP2 baseline 草稿已生成 | PASS | `process/checkpoints/CP2-CR058-REQUIREMENTS-BASELINE.md` | 等待统一人工确认。 |
| HLD / ADR 草案完整 | PASS | docs/design | 设计以 no-op-by-default 为核心。 |
| CR053 输入已引用 | PASS | CR058 HLD | 消费 inventory / path refs / migration plan。 |

## Checklist

| # | 检查项 | 推荐结果 | 用户审查 |
|---|---|---|---|
| 1 | 是否接受 allowlist-first design gate 架构 | accept | 通过 |
| 2 | 是否接受 CR053 Evidence Adapter 只消费静态证据、不继承授权 | accept | 通过 |
| 3 | 是否接受 preserve-audit-by-default 架构 | accept | 通过 |
| 4 | 是否接受 rollback_ref 三件套作为后续真实执行前置 | accept | 通过 |
| 5 | 是否接受当前批次不新增 mover/rewrite 脚本 | accept | 通过 |

## Exit Criteria

| 条目 | 通过条件 | 状态 |
|---|---|---|
| 用户回复 `approve` | 接受 Decision Brief 内全部推荐方案，且不授权列明禁止操作。 | PASS |
| 用户回复 `修改: <具体修改点>` | host-orchestrator 按修改点更新相关 DQ / 文档并重新发起门禁。 | pending |
| 用户回复 `reject` | CR058 CP3 不通过，回到 HLD 修订。 | pending |
| 人工审查结果回填 | 本文件“人工审查结果”填入用户选择、时间和处理结论。 | PASS |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| HLD | `docs/design/HLD-CR058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE.md` | ready |
| ADR | `docs/design/ARCHITECTURE-DECISION-CR058.md` | ready |
| CP3 Auto Check | `process/checks/CP3-CR058-HLD-CONSISTENCY.md` | ready |
| Human Gate Launch Message | `process/checks/CP3-CR058-HUMAN-GATE-LAUNCH-MESSAGE.md` | ready |

## Decision Brief

### 推荐决策

推荐用户回复 `approve`，接受 CR058 HLD / ADR：采用 manifest-first、allowlist-first、preserve-audit-by-default、rollback_ref-before-execution、no-op-by-default 的 repo-local relayout gate。CP3 approve 只允许进入 Story planning / CP5 设计证据确认，不授权真实 move / rewrite。

### 备选方案

| 方案 | 内容 | 优点 | 代价 / 风险 | 切换条件 |
|---|---|---|---|---|
| 推荐方案 A | allowlist-first design gate。 | 安全、可审计、符合 CR053 边界。 | 真实执行后置。 | 默认推荐。 |
| 备选方案 B | repo-wide mechanical rewrite plan。 | 覆盖全面。 | 破坏历史审计风险高。 | 用户明确接受历史证据可改写并新增门禁。 |
| 备选方案 C | defer CR058。 | 零执行风险。 | 无法推进 relayout gate。 | 用户不需要当前设计。 |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---|
| CR058 formal CR | scanned | 5 | 5 | 范围、架构、rollback、不授权均纳入。 |
| CR053 migration inventory | scanned | 3 | 2 | candidate list / owner classification 纳入；真实扫描 N/A。 |
| CR053 path references | scanned | 4 | 2 | preserve-audit / mechanical-candidate 纳入。 |
| meta-se 建议 | scanned | 4 | 3 | HLD 模块和 ADR 建议纳入。 |
| 合计 | scanned | 16 | 5 | 收敛为 CP3 五项 DQ。 |

## 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR058-01 | architecture | 是否接受 allowlist-first design gate？ | 接受，只对明确 future-facing 候选进入后续清单。 | repo-wide rewrite。 | 推荐方案保护审计；备选覆盖大但风险高。 | 决定后续候选范围。 | 用户要求全仓替换时回 CP3 重设风险。 |
| DQ-CP3-CR058-02 | architecture | 是否确认 CR053 Evidence Adapter 只消费静态证据？ | 确认，不继承运行授权。 | 从 CR053 继承授权；不推荐。 | 推荐方案符合 CR053 CP8；备选越权。 | 防止 READY_WITH_RISK 被误读。 | 若重开 CR053 CP8 可重新讨论。 |
| DQ-CP3-CR058-03 | risk_acceptance | 是否接受 preserve-audit-by-default？ | 接受，历史 process/checkpoint/handoff/Story 默认 preserve-audit。 | 单项豁免或批量改写。 | 推荐方案保留历史事实；批量改写风险高。 | 保留 legacy alias 历史引用。 | 单项豁免需 path、owner、rollback_ref。 |
| DQ-CP3-CR058-04 | implementation | 是否接受 rollback_ref 三件套作为真实执行前置？ | 接受 commit + bundle + manifest。 | commit-only；不推荐。 | 推荐方案可回退；备选少证据。 | 后续真实执行缺证据即 blocked。 | 用户显式豁免 bundle 时记录风险接受。 |
| DQ-CP3-CR058-05 | implementation | 当前批次是否不新增 mover/rewrite 脚本？ | 接受，只做门禁设计和 Story 设计证据。 | 立即新增脚本；不推荐。 | 推荐方案避免误执行；脚本可能被误用。 | 后续自动化需另行 CP5/CP6。 | 用户明确要求 no-op 脚本时新增 Story。 |

## 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

| Item ID | 不授权操作 |
|---|---|
| NA-CP3-CR058-01 | 真实 repo-local move / rename / delete / bulk rewrite |
| NA-CP3-CR058-02 | 新增并运行 mover / rewrite 脚本 |
| NA-CP3-CR058-03 | NAS、data lake、provider、QMT / MiniQMT runtime、git remote 操作 |
| NA-CP3-CR058-04 | 凭据或敏感正文读取 |
| NA-CP3-CR058-05 | 恢复 CR046 CP7 |

## 人工审查结果

| 字段 | 值 |
|---|---|
| 用户回复 | 接受，继续推进 |
| 审查人 | user |
| 审查时间 | 2026-06-14T14:59:00+08:00 |
| 决策结果 | approved；接受 `DQ-CP3-CR058-01..05` 推荐方案；确认 HLD / ADR 设计方向通过，不授权真实 move / rewrite、脚本执行、外部系统或 CR046 恢复。 |
