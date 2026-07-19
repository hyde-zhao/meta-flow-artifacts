---
status: "cp4-design-ready"
version: "1.2"
feature_id: "FEAT-CR172-I03"
---

# FEAT-CR172-I03 PATH-I Authorization and Claim Governance — TEST-PLAN

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始六动作、DAG、empirical/path/signal/claim 负向与静态测试计划。 |
| 1.1 | 2026-07-18 | meta-se-critical | 增加 decision-origin/target-kind typed binding 与 fixture-to-real deny。 |
| 1.2 | 2026-07-18 | meta-se-critical | 增加 approved-ledger current-v1 双 false、无 evidence_kind 第二真相和 REQ-013 runtime-deferred claim guard。 |

## 1. Contract / permission cases

| Case | 输入 | 期望 |
|---|---|---|
| I03-P01 | 每个 action 独立有效 record + valid predecessor fixture | decision `6/6` 可独立产生；无权限并集 |
| I03-N01 | missing/expired/revoked/scope/hash/path mismatch | eligible=`false` for all mutation classes |
| I03-N02 | runtime record approved but read missing/deny/expired | runner/workspace/pointer=`0/0/0` |
| I03-N03 | generation approved but runtime predecessor/context mismatch | candidate byte=`0` |
| I03-N04 | sync/pull approved but artifact predecessor missing | NAS/execution staging/pointer=`0` |
| I03-N05 | `repository_fixture` decision + `real_operation` target | accepted/side-effect=`0/0` |
| I03-N06 | caller 自报 `approved_ledger` + valid-looking record/real target | authorized/eligible=`false/false`；reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE` |
| I03-R01 | revoke before pointer commit | current pointer unchanged；partial non-runtime |

## 2. Claim / boundary cases

| Case | 输入 | 期望 |
|---|---|---|
| I03-C01 | declared fixture matrix | declared_exact；empirical claim=0 |
| I03-C02 | source/v2/auth absent without integrity conflict | typed_unavailable；C1=false |
| I03-C03 | valid-looking empirical refs pre-FU-v2 | positive count=0；downgrade design allowed |
| I03-C04 | hash/window/alignment/tamper/unapproved repair | BLOCKED；no relabel |
| I03-PATH | new/legacy path fixtures | new decision=1；legacy mutation=0 |
| I03-PATH-RUNTIME | current runner/default enforcement inventory | contract ready=1；runtime enforcement/default switch=0/0；CP8 runtime-delivered claim=0 |
| I03-SIG | exact/missing/extra/credential/order/intraday fixtures | exact slots 8/8；malformed/forbidden reject；exchange implementation=0 |
| I03-CLM | CP8 flag fixture | five high-order flags=false |

## 3. Static and zero-operation guard

- authorization backend/network/env/credential reads=`0`。
- six real actions authorized/executed=`0/6`,`0/6`。
- Signal mailbox/state/ack/idempotency/replay/consumer/intraday production symbols=`0`。
- public C1/effective estimator production call/write=`0`。
- PATH-C/A auto-resume、E1、OI-003/004/005 closure=`0`。
- `evidence_kind` test helper/schema/assertion=`0`；provenance 只读 decision origin + target kind + fixture URI/port。

## 4. Coverage / failure route

覆盖 SC-A02/Q02/C02/G01/G02/S01～S06 以及 runtime-without-read design simulation。合同歧义→NEEDS_DESIGN_CLARIFICATION；实现错误→NEEDS_REWORK；真实操作/权限合并/positive overclaim/signal scope escape→BLOCKED。CP4 实际测试执行=`0`。

## 5. Gotchas

- 必须分别断言 `authorized` 和 `eligible_to_execute`，不能只测一个布尔值。
- `typed_unavailable` 与 `BLOCKED` 的 mutation fixtures 不可共享预期。
