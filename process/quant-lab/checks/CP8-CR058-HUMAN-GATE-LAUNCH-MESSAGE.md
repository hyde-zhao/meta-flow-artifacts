# CP8 CR058 Human Gate Launch Message

以下消息可由 host-orchestrator 直接发送给用户。

---

请审查：process/checkpoints/CP8-CR058-DELIVERY-READINESS.md

自动预检结论：`READY_WITH_RISK`

请注意：这是“CR058 static-only / no-op relayout gate 交付就绪”的 close gate，不是实迁移授权，也不是 `RELEASED`。

Context Capsule 摘要：

| 项目 | 内容 |
|---|---|
| Release Context | `process/release/RELEASE-CONTEXT-CR058.yaml` |
| Delivery Context | `process/context/CP8-CR058-DELIVERY-CONTEXT.yaml` |
| release_artifact_profile | `full`，原因是 CR058 涉及 migration gate、preserve-audit、rollback_ref、安全权限和真实执行不授权边界。 |
| release_decision | `READY_WITH_RISK` |
| 范围 | CR058-S01..S05：candidate list、preserve-audit allowlist、rollback gate、no-op guardrails。 |
| CP7 质量结论 | PASS_WITH_RISK；REVIEW approve-with-risk；BLOCKER/HIGH/MEDIUM findings 均为 0。 |
| 风险接受候选 | `R-CR058-01`、`R-CR058-02`、`R-CR058-03`、`R-CR058-04` |

决策收集覆盖摘要：

| 项目 | 数量 / 说明 |
|---|---|
| 已扫描来源 | CP7 check、VERIFICATION-REPORT、TEST-REPORT、REVIEW、Release Context、No-op guardrails、Rollback gate、当前对话 |
| 候选问题数 | 32 |
| 纳入待决策数 | 5 |
| N/A / 合并原因 | 重复的授权误读、rollback 缺口、历史引用和后续执行候选已合并到风险接受、运行不授权和 follow-up tracking 决策项。 |

本轮待人工决策项：5

待人工决策清单：

| 决策 ID | 类型 | 推荐方案 | 备选方案 |
|---|---|---|---|
| DQ-CP8-CR058-01 | risk_acceptance | 接受 CR058 static-only/no-op gate 交付 `READY_WITH_RISK`，并关闭当前 CR058 gate delivery。 | `NOT_READY`，退回补 rollback_ref / candidate hash / execution manifest。 |
| DQ-CP8-CR058-02 | runtime_authorization | 确认 CP8 approve 不授权 `NA-CR058-001..012` 中任何真实操作。 | 授予单项运行授权；不推荐，且必须新 CR / 新门禁。 |
| DQ-CP8-CR058-03 | risk_acceptance | 接受 rollback_ref / candidate hash 缺失作为 static-only close 的剩余风险；真实执行继续 `execute_allowed=false`。 | 不接受，退回补 `pre_cr058_commit` / bundle / manifest / hash。 |
| DQ-CP8-CR058-04 | follow_up_tracking | 接受 CR059 repo-local mechanical migration execution gate 作为后续候选，不自动启动。 | 立即启动 CR059；不推荐，需新门禁和执行前置。 |
| DQ-CP8-CR058-05 | risk_acceptance | 接受 preserve-audit 策略下历史 `local_backtest` 引用继续保留。 | 单项豁免改写；需 path、owner、reason、risk、rollback_ref、approval_decision_id。 |

如果你回复 approve，表示你接受以上 5 项推荐方案；不表示授权以下 12 项禁止操作。

不授权项：

| Item ID | 不授权项 |
|---|---|
| NA-CR058-001 | 真实 repo-local file move / rename / delete |
| NA-CR058-002 | 批量路径替换 / reference rewrite |
| NA-CR058-003 | 历史 process/checks/checkpoints/handoffs/Story/LLD/DEV-LOG 机械改写 |
| NA-CR058-004 | NAS mount / scan / mkdir / copy / delete / migration |
| NA-CR058-005 | `MARKET_DATA_LAKE_ROOT` replacement 或 lake write |
| NA-CR058-006 | provider fetch / catalog publish |
| NA-CR058-007 | 凭据、`.env`、token、password、cookie、session、private key、账户凭据读取 |
| NA-CR058-008 | QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live |
| NA-CR058-009 | git push / tag / remote rename / history rewrite |
| NA-CR058-010 | `git reset --hard` 或 `git checkout --` 回退用户未明确要求的工作树 |
| NA-CR058-011 | 从 CR053 继承真实迁移授权 |
| NA-CR058-012 | 恢复 CR046 CP7 或推进 CR046 runtime verification |

请回复以下三种之一：

approve

修改: <具体修改点>

reject
