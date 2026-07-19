---
status: "cp4-design-ready"
version: "1.2"
feature_id: "FEAT-CR172-I03"
feature_name: "PATH-I Authorization and Claim Governance"
source_blueprint: "docs/design/BLUEPRINT.md"
source_hld: "docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR172-S01-action-authorization-eligibility-governance"
  - "CR172-S05-path-i-integration-claim-zero-operation-verification"
lld_policy_summary: "2/2 full-lld; CP4 不创建 LLD"
confirmed_by: ""
confirmed_at: ""
---

# FEAT-CR172-I03 PATH-I Authorization and Claim Governance — DESIGN

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初始 CP4 Feature 设计；冻结六 action 独立 record、执行资格 DAG、四态 empirical disposition、run-path 与 SignalBatch 8-slot boundary。 |
| 1.1 | 2026-07-18 | meta-se-critical | CP5 R1：保持 12-field record 不变，在派生 decision/context 冻结 `decision_origin`/`target_kind`，fixture decision 与 real target 接受数为 0。 |
| 1.2 | 2026-07-18 | meta-se-critical | CP5 R2：current-v1 `approved_ledger` 固定双 false + 稳定 reason；删除 `evidence_kind` 第二真相；REQ-013 标为 contract-ready/runtime-enforcement-deferred 并绑定 future path-enforcement。 |

## 1. 摘要与成功标准

| 项目 | 内容 |
|---|---|
| Feature 目标 | 提供所有 PATH-I Story 共用的 deny-default action decision、prerequisite eligibility、claim disposition 和 scope-containment contract。 |
| 推荐方案 | 单一纯 contract/governance 模块由 S01 独占；S02-S04 只消费 typed decision；S05 独立做静态/fixture 验证。 |
| 非目标 | 不实现 authorization ledger/backend，不批准 action，不执行 action，不生成 R/signal，不实现 signal exchange。 |

成功标准：action kinds/records/enforcement points=`6/6/6`；approval union=`0`；eligibility DAG edges=`5/5`；runtime-own-auth/no-read launch=`0`；empirical disposition exactly one of `4/4`；pre-v2 positive count/C1=`0/0`；SignalBatch slots=`8/8`、extra mandatory/exchange Story/module=`0/0/0`；claim ceiling 5 flags all false。

## 2. 模块与对象

| Module / Object | Owner | 职责 | 禁止 |
|---|---|---|---|
| `engine/path_i_governance.py` | S01 | `ActionAuthorizationRecordV1`、带 `decision_origin` 的 `ActionDecisionV1`、带 `target_kind` 的 `ActionScopeContextV1`、eligibility DAG/context equality、claim/path/signal value contract | IO、ledger、env/credential、real approval、R/signal generation |
| `tests/research/test_cr172_path_i_governance.py` | S01 | contract/negative/unit fixtures | real action |
| S05 QAC tests | S05 | 跨 Story coverage、claim/static/no-real-op | 修改 governance production module |

## 3. 六类授权与执行资格

六份 record 独立存在、撤销和到期，绝不合并：

```text
data_lake_read -> multi_trial_runtime_and_workspace_write -> trial_return_generation
trial_return_generation -> empirical_R_computation
trial_return_generation -> nas_replica_sync -> execution_pull_verify_materialize
```

| Action | 自身 record | 直接前置 | deny 时必须为 0 |
|---|---|---|---|
| data_lake_read | required | none | dereference |
| multi_trial_runtime_and_workspace_write | required | same-context eligible read | runner launch/workspace/pointer |
| trial_return_generation | required | eligible runtime | first candidate byte |
| empirical_R_computation | required | sealed generation provenance | estimator entry/positive claim |
| nas_replica_sync | required | sealed generation artifact | NAS staging/pointer |
| execution_pull_verify_materialize | required | verified replica receipt | NAS read/execution staging/cache pointer |

context equality 至少覆盖 `scope_revision/scope_sha256/release_id/run_id/family_id`；缺失、deny、expired、revoked、path mismatch、context mismatch 均 `eligible_to_execute=false`。历史 generation 已 sealed 后，R 计算只要求历史 eligibility evidence 可验证，不要求已不再 dereference 的 read record 当前仍活跃。

12-field approval record 不扩列。`ActionDecisionV1.decision_origin=repository_fixture|approved_ledger`；`ActionScopeContextV1.target_kind=repository_fixture|real_operation`。fixture origin 只允许 fixture target、`fixture://` URI 与 repository-owned fixture/in-memory port；fixture+real accepted=`0`。current-v1 所有 `approved_ledger` 输入固定返回 `authorized=false`、`eligible_to_execute=false`、reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；caller 自报 enum/record 不能解锁。可信 issuer/envelope/adapter 仅由 future runtime-high-risk CR 引入。`evidence_kind` 字段/别名/第二真相=`0`。

## 4. Claim、path 与 signal 边界

| Contract | 冻结内容 | Claim ceiling / deferred |
|---|---|---|
| Empirical R | `declared_exact/empirical/typed_unavailable/BLOCKED` 四态；exactly one | FU-CR173-001 缺失时 positive effective count/C1=0；DQ-003 downgrade 可设计 |
| Run path | `RunPathDecisionV1` contract ready；legacy read-only | runtime enforcement/default switch deferred；future native producer 在 launch/workspace first side effect 前消费 decision，证明 new default=1、legacy write=0 |
| SignalBatch | exact slots：schema_version、batch_id、strategy_id、strategy_package_hash、content_sha256、signature/key_id、valid_from/valid_until、sequence_no | extra mandatory=0；path/state/ack/idempotency/replay/exchange/consumer/intraday module/Story/implementation=0 |
| CP8 claim | `path_i_design_ready` 上限 | stage3_entry_ready/c1_computable/real_data_authorized/multi_trial_runtime_authorized/signal_transport_authorized=false |

Signal 最小 boundary 是 S01 value-object facet，不创建 signal Story；详细交换和 intraday 仅保留 deferred refs。

## 5. 调用合同、失败与撤销

| Caller | 时机 | 输入 | 输出 | 失败行为 |
|---|---|---|---|---|
| S02 runner/hook | lake dereference、workspace、candidate first byte 前 | 单 action request + record + predecessor evidence + context | authorization decision + eligible bool + evidence ref | 不调用对应 action；pointer=0 |
| S03 replica | NAS staging 前 | sync record + original sealed provenance | eligible decision | staging/pointer=0 |
| S04 materializer | NAS read/execution staging 前 | pull record + verified receipt | eligible decision | read/staging/pointer=0 |
| S05 verifier | fixture/static | all six decisions/dispositions | verification evidence | any union/overclaim→BLOCKED |

mid-operation revoke/expiry 后禁止下一步和 pointer commit；partial staging non-distributable/non-runtime；清理需新授权。contract 模块不负责监听真实 backend，未来 action adapter 必须在每个 enforcement point 重新判定。

## 6. Story / Wave

| Wave | Story | 主要 owner | 依赖 |
|---|---|---|---|
| W1 | S01 governance contract | `engine/path_i_governance.py` | none |
| W5 | S05 integration/claim/zero-op | CR172 fixtures/QAC tests | S01-S04 runtime evidence |

S01 是 S02-S04 的 contract dependency；其 LLD 必须先冻结。S05 只读所有 production module，禁止借 verification 修改源文件。

## 7. 风险、回退与 Gotchas

| Risk | 缓解 | 回退 |
|---|---|---|
| approval 被误作 execution eligibility | decision 分开记录 `authorized` 与 `eligible_to_execute` | predecessor 缺失即 deny |
| empirical 前置不足吞掉 tamper | completeness 缺失→typed_unavailable；integrity conflict→BLOCKED | 不自动 repair/relabel |
| Signal boundary 膨胀 | exact slots + forbidden capability inventory | detailed request 路由 deferred |

Gotchas：S01 代码可表达授权 schema，但当前不存在获批 authorization record；fixture allow 不是项目真实授权；Signal 8 slots 不是 mailbox schema；PATH-I PASS 不自动恢复 PATH-C/A。

REQ-013 在本 Feature 仅为 `contract_ready/runtime_enforcement_deferred`；现有 runner diff与 runtime path enforcement 均为 0，CP8 不得声明 runtime delivered。
