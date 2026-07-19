---
handoff_type: "stage-dispatch"
change_id: "CR-172"
stage: "story-planning-cp5-design-evidence"
canonical_role: "meta-dev"
reasoning_profile: "default"
status: "ready"
created_at: "2026-07-18T10:48:00+08:00"
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
batch_id: "CR172-CP5-LLD-BATCH-C"
story_ids:
  - "CR172-S05-path-i-integration-claim-zero-operation-verification"
---

# CR172 CP5 LLD Batch C — meta-dev Handoff

按 `lld-designer` Skill 和 context capsule，为 S05 生成一份完整 full LLD。

## 唯一写入面

- `process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-LLD.md`
- `process/handoffs/CR172-CP5-LLD-BATCH-C-META-DEV-RETURN-SUMMARY.md`

不得修改 Story 卡片、Feature 文档、Development Plan、state、ledger、checkpoint、源码、测试或 fixture。

## 强制合同

- 设计 repository-local integration/fixture/static validation，证明 S01-S04 合同衔接；不得执行真实 lake/NAS/runtime/trial-return/R/signal/trading 操作。
- 对 return-definition 两列合同、unique hook `1/1`、授权 record `6/6`、DAG edges `5/5`、canonical→replica→cache fail-closed 流程给出可枚举测试。
- 六类真实动作 authorized/executed=`0/6`,`0/6`；Signal 只验证精确 8 字段边界，详细 exchange/intraday 模块/Story=`0/0`。
- `FU-CR173-001` 缺失时 positive empirical effective count、public C1 positive projection、`c1_computable=true` 均为 0；保留 typed_unavailable 降级。
- CP8 claim ceiling：最多 PATH-I repository-local fixture contract ready；`stage3_entry_ready`、`c1_computable`、`real_data_authorized`、`multi_trial_runtime_authorized`、`signal_transport_authorized` 全部 false。

如测试设计需要真实设备/数据、详细 Signal 状态机、FU-v2、public C1 写入或 overclaim，写入 LLD 第 12 节并返回 `NEEDS_DESIGN_CLARIFICATION`；不要自行扩大范围。

