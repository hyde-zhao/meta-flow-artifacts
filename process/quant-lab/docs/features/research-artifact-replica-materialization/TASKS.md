---
status: "cp4-design-ready"
version: "1.1"
feature_id: "FEAT-CR172-I02"
---

# FEAT-CR172-I02 Research Artifact Replica and Materialization — TASKS

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始 S03/S04 各 4 个任务与串行依赖。 |
| 1.1 | 2026-07-18 | meta-se-critical | R3 LLD 输入：S04 从 S03 selection 取 sealed bundle+selection，并复用 S02 唯一 verifier library；禁止旁路/第二 digest。 |

## CR172-W3 — S03 NAS Replica Verification

| TASK-ID | 顺序 | 任务 | 输出 | Owner | 状态 |
|---|---:|---|---|---|---|
| CR172-S03-T01 | 1 | 编写 full LLD，冻结 receipt/canonicalization/pointer/revoke | S03 LLD | S03 | pending-after-CP4 |
| CR172-S03-T02 | 2 | 创建 replica contract/verifier | `engine/research_artifact_replica.py` | S03 | blocked-before-CP5 |
| CR172-S03-T03 | 3 | 创建 positive/negative/recovery fixtures | S03 tests | S03 | blocked-before-CP5 |
| CR172-S03-T04 | 4 | 验证 original seal、research pointer 不变与 no-real-op guard | S03 evidence | S03 | blocked-before-CP5 |

## CR172-W4 — S04 Execution Cache Materialization

| TASK-ID | 顺序 | 任务 | 输出 | Owner | 状态 |
|---|---:|---|---|---|---|
| CR172-S04-T01 | 1 | 编写 R3 full LLD，冻结 S03-selected staging bundle、S02 verifier-library、`4/4` bytes verify 与 direct-NAS deny | S04 LLD | S04 | pending-r3 |
| CR172-S04-T02 | 2 | 创建 materialization contract | `engine/research_artifact_materialization.py` | S04 | blocked-before-CP5 |
| CR172-S04-T03 | 3 | 创建 pull/interruption/mismatch/cache rollback fixtures | S04 tests | S04 | blocked-before-CP5 |
| CR172-S04-T04 | 4 | 验证 previous cache 保留与 no-runtime edge | S04 evidence | S04 | blocked-before-CP5 |

## 依赖与禁止

S03 depends on S01+S02；S04 depends on S01+S03 data selection，并对 S02 增加 verifier-library contract dependency；两个模块无共享写文件。禁止绕过 S03 selection、真实 NAS mount/sync/pull、执行机写入、runtime consumer、迁移和 signal。

## 量化自检

tasks=`8/8`；Waves=`2/2`；duplicate primary owner=`0`；CP4 LLD/code/test created=`0/0/0`。

## Gotchas

- S04 不得跳过 S03 selected replica 取数；可以且必须复用 S02 唯一 verifier library 做 staging bytes 校验。
- S03/S04 LLD 可以引用同一 identity contract，但不得共享 production file owner。
