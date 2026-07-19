---
tracking_id: "CR-172-FOLLOW-UP-TRACKING-2026-07-19"
source_cr: "CR-172"
source_checkpoint: "CP8"
status: "candidate"
owner: "meta-flow maintainer"
created_at: "2026-07-19T02:10:00+08:00"
---

# CR-172 CP8 Follow-up Tracking

## FU-CR172-LEDGER-001 — Checkpoint retry event identity

| 字段 | 内容 |
|---|---|
| 状态 | `candidate / process-tooling` |
| 问题 | `meta-flow cp ledger-append` 的默认 `event_id` 由 `checkpoint + story_id` 生成；同一 Story 的 CP6/CP7 重试会复用 ID。当前 CR-172 的 append-only `CHECKPOINT-LEDGER.ndjson` 因此有 10 个受影响的重复 ID、12 条额外冲突行，完整 `event check` 失败。 |
| 影响 | 机器唯一性校验不可全绿；各事件仍可由 `checked_at + result_ref + context_ref` 无歧义区分，最终 CP7/CP8 Result 本身均通过。 |
| 根因位置 | `/home/hyde/workspace/meta-flow/meta_flow/checks/cp_result.py::build_checkpoint_event`：默认值未纳入 `checkpoint_id`、result identity 或 attempt。 |
| 推荐修复 | 生成器优先消费 result 的显式 `event_id`；缺省值至少纳入 `checkpoint_id` 或稳定 result identity，并增加 CP6R1/CP7R2 fixture 回归。 |
| 历史迁移 | 另行批准 ledger compaction/migration：保留原 ledger 与 hash、生成唯一 ID 映射、验证引用和 rollback 后才切换；不得在 CR-172 CP8 内原地改写 append-only 历史。 |
| 验收 | 同一 Story 的初验、两轮回修/复验事件 ID `6/6` 唯一；fixture ledger `event check=OK`；旧 ledger migration 有 before/after hash、映射、审计与回滚。 |
| 非授权 | 本 follow-up 不授权修改 meta-flow 源码、重写当前 ledger、compact/archive、提交、推送或远端写入。 |

## 当前 CP8 处置

该问题作为 `REV-CR172-004` 合并进入 `DQ-CP8-CR172-001` 风险接受：允许 PATH-I repository-local 合同进入人工终验，但不得宣称 checkpoint ledger 全局校验为 PASS。用户未接受前，CR-172 保持 `active/cp8_pending`。
