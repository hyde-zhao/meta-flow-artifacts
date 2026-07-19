---
status: "cp4-design-ready"
version: "1.1"
feature_id: "FEAT-CR172-I03"
---

# FEAT-CR172-I03 PATH-I Authorization and Claim Governance — TASKS

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始 S01 4 tasks、S05 4 tasks。 |
| 1.1 | 2026-07-18 | meta-se-critical | R3 LLD 输入：approved-ledger 双 false、删除 evidence_kind、REQ-013 runtime enforcement deferred。 |

## CR172-W1 — S01 Governance Contract

| TASK-ID | 顺序 | 任务 | 输出 | Owner | 状态 |
|---|---:|---|---|---|---|
| CR172-S01-T01 | 1 | 编写 R3 full LLD，冻结 record/decision/context/DAG、approved-ledger 双 false、path runtime-deferred、signal | S01 LLD | S01 | pending-r3 |
| CR172-S01-T02 | 2 | 创建纯 governance contracts 与 validators | `engine/path_i_governance.py` | S01 | blocked-before-CP5 |
| CR172-S01-T03 | 3 | 创建六 action、runtime-without-read、revoke/context fixtures | governance tests | S01 | blocked-before-CP5 |
| CR172-S01-T04 | 4 | 创建 empirical/path/signal/claim static guard | governance tests | S01 | blocked-before-CP5 |

## CR172-W5 — S05 Integrated Verification

| TASK-ID | 顺序 | 任务 | 输出 | Owner | 状态 |
|---|---:|---|---|---|---|
| CR172-S05-T01 | 1 | 编写 R3 full LLD，删除 evidence_kind 第二真相并冻结 REQ-013 runtime-delivered=0 | S05 LLD | S05 | pending-r3 |
| CR172-S05-T02 | 2 | 创建跨 Feature deterministic fixtures | `tests/fixtures/cr172_path_i/` | S05 | blocked-before-CP5 |
| CR172-S05-T03 | 3 | 创建 integrated/authorization/claim QAC tests | 3 个 S05 test files | S05 | blocked-before-CP5 |
| CR172-S05-T04 | 4 | 执行 scoped coverage/DAG/source-boundary/zero-op 回归并汇总 | implementation/verification evidence | S05 | blocked-before-CP5 |

## 依赖、禁止与自检

S01 无上游，是 S02-S04 的 contract 前置；S05 依赖 S01-S04。禁止实现真实 authorization backend、Signal exchange、FU-v2、R producer、PATH-C/A auto-resume。tasks=`8/8`；CP4 LLD/code/test=`0/0/0`。

## Gotchas

- S05 只能写自己的测试/fixture，不能修 production module；发现缺陷必须回对应 Story。
- S01 的 fixture allow record 不能被写成真实授权证据。
