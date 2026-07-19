---
status: "cp5-r2-design-ready"
version: "1.1"
feature_id: "FEAT-CR172-I01"
---

# FEAT-CR172-I01 Trial Return Artifact Pipeline — TASKS

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始任务分解。 |
| 1.1 | 2026-07-18 | meta-se-critical | S02 收窄为 pure contract + fixture producer port；删除 mature runner 与 lineage test 修改任务。 |

## Wave CR172-W2 — S02 Artifact Contract and Fixture Producer Port

| TASK-ID | 顺序 | 任务 | 输出 | 状态 |
|---|---:|---|---|---|
| CR172-S02-T01 | 1 | R2 full LLD：fixture/real binding、forward proxy deny、partial lineage audit、seal digest public contract | S02 LLD | pending-r2 |
| CR172-S02-T02 | 2 | 创建 payload/manifest/seal/verifier/fixture-port pure module | `engine/trial_return_artifact.py` | blocked-before-CP5 |
| CR172-S02-T03 | 3 | 创建 schema/hash/binding/failure/runner-zero-diff tests | `tests/research/test_cr172_trial_return_artifact.py` | blocked-before-CP5 |
| CR172-S02-T04 | 4 | 输出 S02→S03 exact public contract 与 zero-real-operation evidence | implementation/evidence refs | blocked-before-CP5 |

## 依赖与禁止

S02 依赖 S01 decision/context contract。primary paths=`2`、duplicate owner=`0`。禁止修改 `engine/mature_multifactor_research.py`、lineage modules/stores 与既有 lineage tests；禁止接受 `forward_label_proxy@v1`，禁止真实 canonical、runtime、NAS、R 或 signal 操作。

## Gotchas

历史 Story ID 仍含 `native-hook`，但 task scope 已明确为 hook=`0`；不得据 ID 恢复已删除任务。
