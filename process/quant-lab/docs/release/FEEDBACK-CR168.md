---
document_id: "FEEDBACK-CR168"
cr_id: "CR-168"
status: "OPEN_FOR_FEEDBACK"
created_at: "2026-07-14T15:06:00+08:00"
---

# CR-168 反馈与后续分流

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline | 定义 C3 缺陷、C4/aggregate、真实方法和授权扩展的分流边界。 |

| 反馈类型 | 处理路径 | 不允许的默认动作 |
|---|---|---|
| C3 validation/calculation/hash/fixture defect | ISSUE → 最小回归集 → CR-168 回修或修复 CR | 不删除 failed test，不降低 fail-closed。 |
| Gate4 local adapter defect | CR-168 修复或独立 remediation | 不修改 canonical 全局逻辑来规避局部测试。 |
| canonical direct caller/global hardening、C1-C4 aggregate | `FU-CR161-007` 或独立 CR | 不在 CR-168 追加 aggregate/StrategyAdmissionPackage integration。 |
| C4 capacity/liquidity/ADV/alpha-decay | `FU-CR161-005` | 不把 typed unavailable 改写为 C4 PASS。 |
| real TCA/calibration、真实数据或 event producer | 新独立 CR + 数据/授权/CP2–CP8 | 不在 fixture/static CR 中隐式连接外部。 |
| CR155 promotion 或 paper candidate | `FU-CR161-007` / 独立 CR | 不更改 CR155 BLOCKED 与 paper_candidate=false。 |
| Git push/tag/publish/deploy | 用户单独授权 | CP8 approve 不授权远端写入。 |

反馈应包含最小 synthetic fixture、受影响 component/schema、预期/实际 reason 或 hash，以及是否涉及外部授权；不得提供凭据或生产数据样本。
