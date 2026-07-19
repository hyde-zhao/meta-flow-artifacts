---
handoff_type: "stage-dispatch"
change_id: "CR-172"
stage: "story-planning-cp5-design-evidence"
canonical_role: "meta-dev"
reasoning_profile: "default"
status: "ready"
created_at: "2026-07-18T10:48:00+08:00"
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
batch_id: "CR172-CP5-LLD-BATCH-B"
story_ids:
  - "CR172-S03-nas-replica-verification"
  - "CR172-S04-execution-cache-materialization"
---

# CR172 CP5 LLD Batch B — meta-dev Handoff

按 `lld-designer` Skill 和 context capsule，为 S03、S04 分别生成一份完整 full LLD。

## 唯一写入面

- `process/stories/STORY-CR172-S03-nas-replica-verification-LLD.md`
- `process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md`
- `process/handoffs/CR172-CP5-LLD-BATCH-B-META-DEV-RETURN-SUMMARY.md`

不得修改 Story 卡片、Feature 文档、Development Plan、state、ledger、checkpoint、源码、测试或 fixture。

## 强制合同

- 研究机本地 sealed artifact 是唯一 active canonical；NAS 只能是 immutable verified replica/backup/distribution；执行机 runtime 只消费本地 immutable cache。
- S03 必须先校验 sealed source、授权、logical URI/hash/schema/manifest，再 staging copy、重验、atomic publish；partial/stale/hash mismatch/authorization failure 不产生 verified pointer。
- S04 必须从 NAS verified replica 拉到执行机 staging，重验后原子物化为本地 cache；执行机 direct-NAS runtime read、direct research-machine read 和 path-based identity 均为 0。
- 执行资格必须消费 S01 独立 record/DAG：`trial_return_generation -> nas_replica_sync -> execution_pull_verify_materialize`，不得把授权合并。
- 稳定身份是 logical URI + content hash；绝对挂载路径不进入 canonical identity；legacy 不迁移、不重写、不成为双默认。
- 只做设计证据，不实现代码；真实 NAS/执行机操作及六类动作 authorized/executed=`0/6`,`0/6`。

如设计要求 NAS 成为工作盘、执行机直接读 NAS、弱化 hash/seal、真实设备路径或新增迁移授权，写入 LLD 第 12 节并返回 `NEEDS_DESIGN_CLARIFICATION`；不要自行扩大范围。

