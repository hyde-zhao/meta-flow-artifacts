# CP8-CR061 Batch B Delivery Readiness 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。CR061 Batch B 已完成授权范围内的 package/import/layout 窄实现，CP6 PASS，CP7 PASS_WITH_RISK。推荐发布结论为 `READY_WITH_RISK`：可以接受本批交付，但 full pytest 未运行、bulk relayout 未完成、Git/NAS/lake/runtime 仍需后续 CR。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/release/RELEASE-CONTEXT-CR061.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 release context；只按需读 CP6/CP7 摘要和 release docs。 |
| 全文档读取扩展 | 已读取 feature-scoped implementation、verification、test report、review 摘要。 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP7 verification | `process/checks/CP7-CR061-BATCH-B-PACKAGE-IMPORT-LAYOUT-VERIFICATION-DONE.md` | scanned | 3 | 2 | PASS_WITH_RISK 风险转 DQ。 |
| Release context | `process/release/RELEASE-CONTEXT-CR061.yaml` | scanned | 7 | 3 | release_decision、follow-up、non-authorized 转 DQ。 |
| Release docs | `docs/release/*-CR061.md` | scanned | 5 | 2 | rollback / migration / feedback 风险合并。 |
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前无额外 pending DQ。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP8-CR061-01 | risk_acceptance | 是否接受 CR061 Batch B `READY_WITH_RISK` 并关闭本批交付？ | 接受 READY_WITH_RISK，关闭 CR061 Batch B 当前交付。 | 备选 A：要求补 full pytest 后再 CP8；备选 B：reject 并回滚 Batch B。 | 推荐方案完成 package identity 第一切片；full pytest 更稳但可能触发外部依赖审查；回滚会退回旧 metadata。 | 接受后仍有后续 CR。 | 若 full pytest 是终验硬要求，则回到 CP7 扩大离线回归。 |
| DQ-CP8-CR061-02 | risk_acceptance | 是否接受 full pytest 未运行？ | 接受；本 CR 已完成授权内 smoke/subset，full regression 作为后续风险。 | 备选 A：现在要求 full pytest；备选 B：扩大若干离线 subset。 | 推荐方案遵守授权边界；full pytest 覆盖更强但需确认无外部副作用；扩大 subset 介于两者。 | 可能存在非 import 行为回归未覆盖。 | 用户要求时先做 command risk scan，再执行离线 subset。 |
| DQ-CP8-CR061-03 | follow_up_tracking | 后续迁移 CR 顺序如何处理？ | 下一步推荐 CR062 Git remote cutover / repository publication gate；bulk relayout、NAS/lake、runtime 继续分 CR。 | 备选 A：先做 bulk relayout；备选 B：先做 NAS/data lake。 | 推荐方案先完成 GitHub 可追踪基线；bulk relayout 技术风险高；NAS/lake 涉外部数据风险。 | 远端仍未初始化。 | 用户选择其他顺序时重新做对应 CR intake。 |
| DQ-CP8-CR061-04 | security | 是否确认 CP8 approve 不授权 Git remote、NAS/lake/provider/runtime、凭据或 CR046 recovery？ | 确认不授权；CP8 只接受 readiness。 | 备选 A：合并 Git remote；备选 B：合并 NAS/lake/runtime。 | 推荐方案保持安全边界；合并操作需要独立执行证据和回滚。 | 完整迁移仍未完成。 | 用户明确要求合并时另起 CR / runtime gate。 |
| DQ-CP8-CR061-05 | implementation | 是否保留 legacy roots 并推迟 physical relayout？ | 保留 legacy roots；`quant_lab` 作为兼容命名空间，physical relayout 后续独立 CR。 | 备选 A：现在 bulk move；备选 B：永久不做 physical relayout。 | 推荐方案兼顾兼容和迁移；bulk move 风险高；永久不做会留下目录不一致。 | 源码目录仍不是最终框架。 | 后续 clean worktree + fresh rollback + full dry-run 后再进入 relayout CR。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP8-CR061-001 | 不执行真实 release / push / publish。 |
| NA-CP8-CR061-002 | 不执行 Git remote add / push / tag / branch rename / history rewrite。 |
| NA-CP8-CR061-003 | 不执行 NAS 数据 copy / move / delete / archive promote。 |
| NA-CP8-CR061-004 | 不执行 data lake root replacement、provider fetch、lake write、catalog publish。 |
| NA-CP8-CR061-005 | 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。 |
| NA-CP8-CR061-006 | 不启动 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。 |
| NA-CP8-CR061-007 | 不恢复或推进 CR046 CP7。 |
| NA-CP8-CR061-008 | 不执行 physical move / rename / delete 或 bulk import rewrite。 |

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CP6 PASS | PASS |
| CP7 PASS_WITH_RISK | PASS_WITH_RISK |
| Release context 已生成 | PASS |
| Release docs 已生成 | PASS |
| 自动预检已完成 | PASS_WITH_RISK |

## Checklist

| 检查项 | 结果 |
|---|---|
| 交付范围清晰 | PASS |
| release_decision 合法 | PASS |
| 风险接受项完整 | PASS |
| 不授权项完整 | PASS |
| 后续 CR 跟踪明确 | PASS |
| 用户终验 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户回复 `approve` 后，接受 READY_WITH_RISK 并关闭 CR061 Batch B 当前交付 | PASS |
| 用户回复 `修改: <具体修改点>` 后返工 CP8 | N/A |
| 用户回复 `reject` 后停止关闭并按意见回退 | N/A |

## Deliverables

- `process/release/RELEASE-CONTEXT-CR061.yaml`
- `docs/release/RELEASE-NOTES-CR061.md`
- `docs/release/DEPLOY-CHECKLIST-CR061.md`
- `docs/release/ROLLBACK-CR061.md`
- `docs/release/MIGRATION-CR061.md`
- `docs/release/FEEDBACK-CR061.md`
- `process/checks/CP8-CR061-BATCH-B-DELIVERY-READINESS.md`
- `process/checks/CP8-CR061-HUMAN-GATE-LAUNCH-MESSAGE.md`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T09:23:26+08:00
- 修改意见：用户回复“同意，下一步是做什么？”，按 `approve` 解析，接受 DQ-CP8-CR061-01..05 推荐方案。
- 风险接受项：接受 CR061 Batch B `READY_WITH_RISK` 并关闭本批交付；接受 full pytest 未运行作为后续风险；接受下一步推荐 CR062 Git remote cutover / repository publication gate；确认 CP8 approve 不授权真实 release、Git remote、NAS/data lake/provider/runtime、凭据或 CR046 recovery；接受保留 legacy roots 并将 physical relayout 推迟到后续独立 CR。
