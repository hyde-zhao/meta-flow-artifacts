---
artifact: "process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-VERIFICATION.md"
reviewer: "meta-qa-critical"
lane: "lane-quality"
input_type: "review_findings"
round: 2
status: "complete"
governance_mode: "review-gated"
story_id: "CR172-S05-path-i-integration-claim-zero-operation-verification"
story_slug: "path-i-integration-claim-zero-operation-verification"
cr_id: "CR-172"
checkpoint: "CP7R1"
validation_mode: "mixed-repository-fixture-and-static"
verification_result: "PASS"
decision: "PASS"
source_story: "process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification.md"
created_by: "meta-qa-critical"
created_at: "2026-07-19T00:49:21+08:00"
updated_at: "2026-07-19T01:22:36+08:00"
runtime_authorization: 0
---

# Review Findings

> 审计说明：§1–§13 完整保留 2026-07-19 CP7 Round 1 的历史审查记录，证明 F-001 的发现、影响和回修路由；§14 是 CP6R1 回修后的 CP7R1 权威复验结论。两者冲突时以 §14 和本文件 frontmatter 的 Round 2 状态为准。

## 1. 审查范围

- 目标对象：S05 repository-local/test-only fixture、三套 QAC 测试、S01-S04 public contracts，以及 CP6 Return/Evidence/Result 机器证据。
- 唯一执行上下文：`process/context/stories/STORY-CR172-S05.CP7.verify-packet.json`，SHA-256 `d57bfa85cb4541656b51f5b2a9efa4b02152bfd63498e35852e4cd023036c7a9`。
- 审查目标：独立验证 15/15 requirements、27/27 scenarios、11/11 outcomes 的数量与语义追踪；三段 artifact 链；六 action/DAG；失败恢复；REQ-013 contract/runtime split；deferred Signal/FU/public C1/legacy 边界；五项 claim ceiling；零真实操作。
- 审查依据：packet `acceptance[]`、Story `量化验收标准`、CP6 Return/Evidence/Result，以及 `verification-execution`、`quality-review`、`review-artifact-protocol`。
- 验证模式：`mixed-repository-fixture-and-static`。只运行仓库内 deterministic fixture、pytest、AST/语义 probe、hash、语法、whitespace 和机器合同检查。
- 非范围：不修改源码、测试、fixture、设计、状态、CURRENT、计划、change、context 或 ledger；不执行真实 lake/NAS/runtime/network/credential/signal/trading/deploy/Git remote 操作；不把 fixture PASS 提升为真实数据、C1、Stage 3 或 runtime readiness。
- 读取边界：完整读取 packet 与全部 required inputs；仅在 cross-contract trigger 下定向审查四个 production modules；未展开 deny-default LLD、IMPLEMENTATION 或完整产品/设计文档。

### 1.1 阶段结论

| 项目 | 结论 |
|---|---|
| CP7 decision | **NEEDS_REWORK** |
| 是否可进入下一阶段 | no |
| 路由 | `meta-dev`，只回修 S05 trace fixture/test |
| Blocking findings | 1 |
| Production defects | 0 |
| Waivers | 0 |
| 真实运行授权 | 0 |

自动化结果为 S05 `27/27`、S01-S05 `154/154`，但独立语义 trace probe 证明 REQ-013 的 contract/runtime split 被错误绑定到 `REQ-CR172-006`。总量相等不能替代 requirement→scenario→test 的语义正确性，因此不得给出 PASS。

### 1.2 前置条件

| 条件 | 结果 | 证据 |
|---|---|---|
| process route health | PASS | `process/current/CURRENT.json`：`health=ok` |
| active Story/context | PASS | STATE/CURRENT 均指向 S05 CP7 packet |
| CP6 gate | PASS | `process/checks/CP6-CR172-S05-IMPLEMENTATION-DONE.result.json`：`decision=PASS` |
| 独立 QA 调度 | PASS | dispatch `AD-CR172-S05-CP7-META-QA-CRITICAL-20260719T004055+0800`；agent/thread `/root/cr172_s04_cp7_meta_qa_critical`；tool `followup_task` |
| 验证环境 | PASS | Python 3.11 + uv；repository fixture/static-only；无外部连接 |
| 五项写边界 | PASS | 仅本报告、CP7 Return、Evidence Index、Result、Summary |

## 2. Findings

### Advisor Table（CP3 方案形成输入适用）

> N/A：本轮 `input_type=review_findings`，不是 CP3 advisor formation。

<!-- findings-table -->

| ID | Severity | Status | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|--------|----------|----------|--------|------------|--------|
| F-001 | BLOCKER | OPEN / BLOCKING | packet acceptance 1、8；Story `量化验收标准`：REQ-013 contract-ready/runtime-deferred；verification trace 必须语义可追溯 | packet 第 10 行与 Story 第 90 行声明 REQ-013 的 `contract_ready/runtime_path_enforcement/default_switch/runtime_delivered/future_prerequisite=1/0/0/0/1`；但 `scenario_catalog.json` 的 `SC-CR172-021` / `test_req013_is_contract_ready_only` 绑定 `REQ-CR172-006`。独立 probe：数量 15/27/11、unknown/duplicate/uncovered=0，但 `req013_contract_binding_exact=false`。现有 `test_traceability_is_exact_and_total` 只比较集合总量，无法识别语义错绑 | CP7 不能证明“正确的 15/15 requirements 覆盖”；CP6/pytest 的全绿结果对错误映射自洽，可能让下游把 REQ-013 contract/runtime split 的证据归到错误 requirement | 将 `SC-CR172-021` 的 requirement 绑定修正为权威 `REQ-CR172-013`（若需保留多重绑定，必须由权威基线明确）；给 trace 测试新增显式语义断言，至少验证 `test_req013_is_contract_ready_only → REQ-CR172-013`，再重跑 27/154 和最小 CP7 复验。生产模块无需修改 | `tests/fixtures/cr172_path_i/scenario_catalog.json:34`；`tests/research/test_cr172_path_i_claim_regression.py:72`、`:39`；`process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification.md:90`；packet `acceptance[7]` |

## 3. 汇总结论

- blocking_count: 1
- required_count: 1
- optional_count: 0
- recommended_next_action: `revise-and-resubmit`
- decision_impact: `NEEDS_REWORK`。实现主链、授权、失败恢复、claim ceiling 与零操作均通过；唯一阻断项是 S05 trace artifact/test 对 REQ-013 的语义错绑。
- trade_off_note: 最小回修只需要更正 S05 scenario catalog 的 requirement 绑定并增加显式语义断言；无需改变 S01-S04 production contracts 或已批准验收目标。

## 4. 验证对象清单

| 对象 | 类型/角色 | 验证方式 | 结果 |
|---|---|---|---|
| `tests/fixtures/cr172_path_i/scenario_catalog.json` | 15/27/11 trace truth | 全文审查、独立集合与语义 probe、hash | **FAIL：F-001** |
| `sealed_chain_v1.json` | S02-S04 deterministic chain input | 全文审查、artifact probe、hash | PASS |
| `failure_mutations_v1.json` | tamper/partial/staging/revoke oracle | 全文审查、独立 failure probe | PASS |
| `zero_operation_oracle_v1.json` | REQ-013/claim/deferred/zero-op oracle | 全文审查、独立 contract probe | PASS |
| `path_i_fixture.py` | test-only public-contract adapter | 全文/AST 审查、compile、hash | PASS：未重定义 production rule、canonical/digest/verifier/claim/authorization 逻辑 |
| 三套 S05 test modules | 27 个 QAC tests | 全文/语义审查、pytest、AST、compile、whitespace、hash | NEEDS_REWORK：执行全绿，但 trace 测试漏检 F-001 |
| 四个 production modules | S01-S04 public contracts | 定向 contract 审查、154 回归、compile、8/8 hash | PASS；无 production diff |
| 四个上游 test modules | S01-S04 regression baseline | 154 回归、8/8 hash | PASS；无上游 test diff |
| CP6 Return/Evidence/Result | 实现证据 | return-check、result-check、hash correlation | PASS_WITH_WARNINGS；无 ERROR |

## 5. 验证追踪矩阵

| Scenario | Requirement(s) | Outcome | Test / Check | 状态 | 风险 |
|---|---|---|---|---|---|
| SC-CR172-001 | REQ-CR172-001 | OUT-001 | `test_canonical_payload_and_seal_contract` | PASS | 无 |
| SC-CR172-002 | REQ-CR172-002 | OUT-002 | `test_three_stage_artifact_chain_is_exact` | PASS | 无 |
| SC-CR172-003 | REQ-CR172-003 | OUT-003 | `test_materialization_calls_s02_verifier_once` | PASS | 无 |
| SC-CR172-004 | REQ-CR172-004 | OUT-004 | `test_forward_label_proxy_is_rejected` | PASS | 无 |
| SC-CR172-005 | REQ-CR172-005 | OUT-005 | `test_tampered_source_fails_closed_before_replica_write` | PASS | 无 |
| SC-CR172-006 | REQ-CR172-006 | OUT-006 | `test_partial_lineage_never_advances_selection` | PASS | 无 |
| SC-CR172-007 | REQ-CR172-007 | OUT-007 | `test_replica_failure_preserves_previous_selection` | PASS | 无 |
| SC-CR172-008 | REQ-CR172-008 | OUT-008 | `test_materialization_failure_preserves_previous_selection` | PASS | 无 |
| SC-CR172-009 | REQ-CR172-009 | OUT-009 | `test_s03_s04_keep_only_original_seal_digest` | PASS | 无 |
| SC-CR172-010 | REQ-CR172-010 | OUT-010 | `test_fixture_provenance_surface_is_minimal` | PASS | 无 |
| SC-CR172-011 | REQ-CR172-011 | OUT-011 | `test_six_actions_records_enforcement_and_dag` | PASS | 无 |
| SC-CR172-012 | REQ-CR172-012 | OUT-001 | `test_all_repository_fixture_actions_are_eligible` | PASS | 无 |
| SC-CR172-013 | REQ-CR172-013 | OUT-002 | `test_runtime_without_read_is_denied` | PASS | 只覆盖 runtime-without-read deny，不替代 contract-ready 显式绑定 |
| SC-CR172-014 | REQ-CR172-014 | OUT-003 | `test_missing_predecessor_fails_closed` | PASS | 无 |
| SC-CR172-015 | REQ-CR172-015 | OUT-004 | `test_revoked_record_fails_closed` | PASS | 无 |
| SC-CR172-016 | REQ-CR172-001 | OUT-005 | `test_predecessor_context_mismatch_fails_closed` | PASS | 无 |
| SC-CR172-017 | REQ-CR172-002 | OUT-006 | `test_approved_ledger_origin_is_unconditionally_denied` | PASS | 无 |
| SC-CR172-018 | REQ-CR172-003 | OUT-007 | `test_valid_looking_real_target_cannot_self_authorize` | PASS | 无 |
| SC-CR172-019 | REQ-CR172-004 | OUT-008 | `test_fixture_origin_cannot_target_real_operation` | PASS | 无 |
| SC-CR172-020 | REQ-CR172-005 | OUT-009 | `test_traceability_is_exact_and_total` | PASS_WITH_GAP | 只检验集合，不检验语义绑定 |
| **SC-CR172-021** | **catalog: REQ-CR172-006；权威要求: REQ-CR172-013** | OUT-010 | `test_req013_is_contract_ready_only` | **GAP / FAIL** | **F-001** |
| SC-CR172-022 | REQ-CR172-007 | OUT-011 | `test_five_high_order_claims_remain_false` | PASS | 无 |
| SC-CR172-023 | REQ-CR172-008 | OUT-001 | `test_signal_is_only_an_eight_slot_boundary` | PASS | 无 |
| SC-CR172-024 | REQ-CR172-009 | OUT-002 | `test_deferred_capabilities_have_no_runtime_surface` | PASS | 无 |
| SC-CR172-025 | REQ-CR172-010 | OUT-003 | `test_zero_real_operation_ledger_is_exact` | PASS | 无 |
| SC-CR172-026 | REQ-CR172-011 | OUT-004 | `test_empirical_r_stays_typed_unavailable_without_real_source` | PASS | 无 |
| SC-CR172-027 | REQ-CR172-012/013/014/015 | OUT-005 | `test_s05_imports_no_runner_lineage_or_real_adapter` | PASS | 无 |

量化 probe：requirements=`15/15`、scenarios=`27/27`、outcomes=`11/11`、collected tests=`27/27`；unknown requirement/outcome/test=`0/0/0`；duplicate scenario/test=`0/0`；uncovered requirement/outcome/test=`0/0/0`。这些集合指标通过，但 `req013_contract_binding_exact=false`，所以整体 trace 状态为 FAIL。

## 6. 设计契约与边界验证

| 合同 | 独立结果 | 状态 |
|---|---|---|
| 三段 artifact 链 S02→S03→S04 | trial port/stage/pull=`1/1/1`；replica/materialized 成功；receipt/selection/hash 绑定一致 | PASS |
| canonical payload 与 S02 seal truth | payload columns=`2/2`；canonical seal bytes/digest/verifier=`1/1/1` | PASS |
| 单 seal/digest truth | S03/S04 secondary digest fields=`0/0`；S04 imported S02 verifier calls=`1` | PASS |
| forward-label 三入口 | trial-return/empirical-R/effective-count accepted=`0/0/0` | PASS |
| approved-ledger caller self-report | accepted/authorized/eligible=`0/0/0`；reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE` | PASS |
| 六 action/record/enforcement | `6/6/6`；record action kinds=`6` | PASS |
| DAG | nodes/edges=`6/5`；cycle/unknown/permission-union=`0/0/0` | PASS |
| runtime-without-read | authorized=1、eligible=0、reason=`PREDECESSOR_MISSING`，guard deny | PASS |
| failure recovery | tamper、replica staging failure、materialization pull failure、revoke accepted/authorized=`0`；partial selection advance=`0`；previous preservation=`2/2`；失败 persist/CAS delta=`0/0` | PASS |
| REQ-013 runtime split 行为 | contract-ready/runtime-enforcement/default-switch/runtime-delivered/future-prerequisite=`1/0/0/0/1`；new/legacy writable=`0/0` | PASS（行为）；FAIL（trace 绑定，F-001） |
| deferred Signal/FU/public C1/legacy/runtime | 12 个 deferred counters 非零=`0/12`；Signal 仅八语义槽；无 transport/mailbox/FU-v2/PublicC1/legacy migrator/real adapter import | PASS |
| 五项 claim ceiling | true=`0/5`，与 oracle 精确相等 | PASS |
| 六类真实动作 | authorized/executed=`0/6`,`0/6` | PASS |
| empirical-R 无真实 source | `typed_unavailable`；c1/effective-count=`0/0` | PASS |
| fixture 单真相 | 本地重定义 production rule=`0`；复制 rule constants=`0`；hash/crypto/subprocess/network imports=`0` | PASS |

`path_i_fixture.py:118` 的 `.replace` 是 `action_kind.value` 字符串规格化，不是 `Path.replace` 或文件迁移；全文 AST/人工审查确认 filesystem mutation、network、credential、真实 adapter 操作为 `0`。

## 7. 分层验证计划与执行结果

| 层 | 方法 | 必跑 | 结果 | 未覆盖/说明 |
|---|---|---:|---|---|
| Static/semantic | 全文 fixture/test 审查、production targeted contract 审查、AST imports/calls | 是 | **FAIL：F-001** | 其余静态边界 PASS |
| Trace contract | 独立 15/27/11 集合 + requirement→test 语义 probe | 是 | **FAIL** | 总量 PASS，REQ-013 语义绑定 FAIL |
| S05 fixture tests | 三套 pytest | 是 | PASS：27/27 | pytest 未捕获语义错绑 |
| Combined regression | S01-S05 七文件 pytest | 是 | PASS：154/154 | failed/skipped=0/0 |
| Syntax | 4 production + fixture helper + 3 S05 tests `py_compile` | 是 | PASS | pycache 路由 `/tmp` |
| Whitespace | README、4 JSON、fixture helper、3 tests | 是 | PASS | 9/9 diagnostics=0；exit=1 为 expected diff |
| Machine contracts | CP6 Return/Result checker | 是 | PASS_WITH_WARNINGS | symlink/path escape/final-attempt/route-plan warning，无 ERROR |
| Hash correlation | 4 production + 4 upstream tests；6 fixture + 3 S05 tests | 是 | PASS | production/upstream 8/8 与最终 CP7 基线一致；S05 9/9 与 CP6 基线一致 |
| Real integration/runtime | 禁止 | 否 | NOT_AUTHORIZED | 执行=0，不得作为 PASS 推导 |

## 8. 自动化与独立 probe 证据

| ID | 检查 | 结果 |
|---|---|---|
| V-001 | S05 三套 pytest | PASS：`27 passed in 0.53s` |
| V-002 | S01-S05 组合 pytest | PASS：`154 passed in 0.94s` |
| V-003 | packet 指定 `py_compile` | PASS：exit=0 |
| V-004 | 9 个 S05 owner assets whitespace guard | PASS：diagnostics=`0/9` |
| V-005 | CP6 Story Return Check | PASS：OK；仅 process symlink expected path warning |
| V-006 | CP6 CP Result Check audit | PASS：OK；仅 INPUT_HASH_PATH_ESCAPE、FINAL_ATTEMPT_UNAVAILABLE、route-plan 既有 warning |
| V-007 | 独立 trace/DAG/single-truth probe | **FAIL：REQ-013 binding false**；其余 15/27/11、DAG 6/5/0、single-truth 0 均通过 |
| V-008 | 独立 artifact/authz/claim probe | PASS：2/2、1/1/1、0/0、verifier=1、approved-ledger 0/0/0、claims 0/5、real actions 0/6 |
| V-009 | 独立 failure probe | PASS：tamper/partial/staging/revoke 全 fail closed；selection advance=0；previous preserved=2/2 |

## 9. Hash 证据

| 文件 | SHA-256 | 与 CP6/上游最终基线 |
|---|---|---|
| `engine/path_i_governance.py` | `bc6abc95a0e660d71fce5567195ef6bff037e0919403b5d400841c579cc82abf` | MATCH |
| `engine/trial_return_artifact.py` | `923fc5e5baaafea88b6cb98d26de5d95b5d43d848fcf0cd51fc41a1190b645f0` | MATCH |
| `engine/research_artifact_replica.py` | `533d34f79e6481dd4bb2bcdb45776c186cb6e3f1a1f6994640f3382f6e466026` | MATCH |
| `engine/research_artifact_materialization.py` | `72541d73cff02bfbd0e20deb19141df63a9ac68389707f97ce4e252d829d008a` | MATCH |
| `tests/research/test_cr172_path_i_governance.py` | `ef4284860453c55a0b9a59357b13a5343ad88c488d5a7ac3fec63ab732de12f9` | MATCH |
| `tests/research/test_cr172_trial_return_artifact.py` | `4497e7b03920e239df752031b1564c09ef16c73687aa455485c6571d5b346969` | MATCH |
| `tests/research/test_cr172_nas_replica_verification.py` | `37c25a7a01bf46ba4fe121f25b57f9f95961e7eb155cb7e0eb49452c2621fd9b` | MATCH |
| `tests/research/test_cr172_execution_materialization.py` | `f58d71fd9f3cd7045d1cf645cea38e62370c0a705efbc3f627ed0f0a32be7049` | MATCH |

S05 fixture/test 9 个 hash 也与 CP6 Result 完全一致，其中阻断文件 `scenario_catalog.json` 保持 CP6 基线 `7a8b4b43881ea9a6b590f2b2349229f96188534b92b6b37df3bafc2e7a39111d`；说明 F-001 是 CP6 已存在但未被原 trace 测试发现的语义缺陷，不是 CP7 执行过程中引入。

## 10. 人工 / 语义质量审查

| 检查项 | 结果 | 阻塞 | 说明 |
|---|---|---:|---|
| 需求一致性 | FAIL | yes | REQ-013 权威语义与 catalog 绑定冲突 |
| 场景覆盖 | FAIL | yes | 数量覆盖完整，但 SC-021 requirement 语义错误 |
| production contract 边界 | PASS | no | S05 只消费 public contracts，8/8 hash 不变 |
| fixture 单真相 | PASS | no | 无 production rule/canonical/digest/verifier/claim logic 复制 |
| failure/happy-path 偏差 | PASS | no | tamper、partial、staging、revoke、context、caller self-report 均覆盖 |
| 错误信息可行动 | PASS | no | F-001 精确到文件/行、影响和最小复验集 |
| 文档/审计可用性 | PASS_WITH_GAP | yes | 机器证据可消费，但 trace truth 必须回修后重新生成 CP6/CP7 evidence |

## 11. 问题、剩余风险与阶段路由

### 11.1 问题清单

| ID | 等级 | 状态 | Owner | 下一动作 |
|---|---|---|---|---|
| F-001 | BLOCKER | OPEN / BLOCKING | meta-dev | 修正 `SC-CR172-021` 的 REQ-013 绑定并增加显式语义断言；重跑 27/154、compile、whitespace、CP6 Return/Result，再进入 CP7R1 |

### 11.2 剩余风险

| Risk | Severity | Status | Owner / 条件 |
|---|---|---|---|
| R-CP7-S05-TRACE-SEMANTIC-MISBINDING | BLOCKER | OPEN | F-001 回修并经独立 CP7R1 关闭前不得推进 |
| R-CR172-RUNTIME-AUTHORIZATION-GAP | HIGH | OPEN / unchanged / not authorized | Host；repository fixture 不授权六类真实动作 |
| R-CR172-REAL-R-DOMAIN-MISMATCH | HIGH | DEFERRED / unchanged | 真实 empirical-R/C1 仍在 S05 范围外 |
| R-CR172-SYNC-REPLICA-STALE | HIGH | CONTRACT-CONTROLLED / runtime-unverified | 真实 adapter/runtime 未授权，不能由 fixture 推导 |

### 11.3 阶段决策

- CP7 decision：`NEEDS_REWORK`。
- next route：`rework_same_story`，只回修 S05 fixture/test trace 语义；不修改 S01-S04 production modules。
- waivers：无。
- design clarification：当前不需要；packet、Story 与 CP6 对 REQ-013 的权威语义一致，属于实现证据错绑。
- TEST-REPORT / REVIEW / FIXES：packet 只允许五项 CP7 证据，因此本文件合并承载完整 verification、test report、quality findings 与最小修复输入，不另写 `docs/**`。
- real operation/runtime authorization：`0`。

## 12. 最小复验集

1. 静态断言 `SC-CR172-021` / `test_req013_is_contract_ready_only` 精确包含权威 `REQ-CR172-013`，且不再误绑定 `REQ-CR172-006`，除非有新的权威基线明确多重绑定。
2. trace test 增加显式 requirement→test 语义断言；重新证明 requirements/scenarios/outcomes=`15/15`,`27/27`,`11/11`，unknown/duplicate/uncovered=`0/0/0`，并新增 `semantic_mismatch=0`。
3. 重跑 S05 `27/27`、S01-S05 `154/154`、py_compile、9 项 whitespace、CP6R1 Return/Result checker。
4. 重算 production/upstream 8/8 hash，必须继续与各自最终 CP7 基线一致；production diff 保持 0。
5. 继续保持六类真实动作 authorized/executed=`0/6`,`0/6`，五项高阶 claim true=`0/5`，所有真实外部操作=0。

## 13. 待确认项

- None。F-001 的权威目标无歧义，不需要用户或设计澄清。

## 14. CP7R1 权威复验结论（Round 2）

### 14.1 阶段结论

| 项目 | 结论 |
|---|---|
| CP7R1 decision | **PASS** |
| verification result | **PASS** |
| 是否可由 Host 准备 CP8 | yes |
| Blocking findings | 0 |
| 新 findings | 0 |
| F-001 | **CLOSED / NON-BLOCKING** |
| Waivers | 0 |
| 真实运行授权 | 0 |

本轮以 `process/context/stories/STORY-CR172-S05.CP7.reverify-1.packet.json` 为唯一执行包（SHA-256 `5ccef55aa9152d2c85c7e3f3bbee4e5aab853881a24341db8143e44cb7515ef3`），先执行独立语义探针，再运行回归和边界检查。F-001 的两条权威绑定均精确成立，semantic mismatch=`0`；因此 Round 1 的唯一阻断项已关闭。

### 14.2 当前 Findings

<!-- cp7r1-findings-table -->

| ID | Severity | Status | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|---|---|---|---|---|---|---|---|
| F-001 | BLOCKER | **CLOSED / NON-BLOCKING** | packet `acceptance[0]`；Story REQ-013 contract-ready/runtime-deferred；语义追踪必须精确 | `SC-CR172-006 → test_partial_lineage_never_advances_selection → REQ-CR172-006`；`SC-CR172-021 → test_req013_is_contract_ready_only → REQ-CR172-013`；独立 probe `semantic_mismatches=[]`；回归测试含显式两条映射断言 | Round 1 的错误归属已消除；15/15 requirement coverage 现在同时满足集合完整性和语义正确性 | 无后续修复；保留显式语义断言作为回归门 | `tests/fixtures/cr172_path_i/scenario_catalog.json:19`、`:34`；`tests/research/test_cr172_path_i_claim_regression.py:72` |

当前没有 OPEN、BLOCKING、REQUIRED 或 OPTIONAL finding。

### 14.3 独立语义与追踪探针

| 检查 | 期望 | 实际 | 结果 |
|---|---:|---:|---|
| SC-CR172-006 test | `test_partial_lineage_never_advances_selection` | exact | PASS |
| SC-CR172-006 requirement | `REQ-CR172-006` only | exact | PASS |
| SC-CR172-021 test | `test_req013_is_contract_ready_only` | exact | PASS |
| SC-CR172-021 requirement | `REQ-CR172-013` only | exact | PASS |
| semantic mismatch | 0 | 0 | PASS |
| requirements | 15/15 | 15/15 | PASS |
| scenarios / collected tests | 27/27 | 27/27 | PASS |
| outcomes | 11/11 | 11/11 | PASS |
| unknown requirement/outcome/test | 0/0/0 | 0/0/0 | PASS |
| duplicate scenario/test | 0/0 | 0/0 | PASS |
| uncovered requirement/outcome/test | 0/0/0 | 0/0/0 | PASS |

探针直接解析 catalog 与三个 S05 测试模块的 AST test definitions，不依赖 pytest 通过状态推断语义；两条目标绑定逐字段比较，`semantic_mismatches=[]`。

### 14.4 自动化、机器合同与哈希复验

| 检查 | 结果 | 证据摘要 |
|---|---|---|
| S05 scoped pytest | PASS | `27 passed in 0.50s`；failed/skipped=`0/0` |
| S01–S05 combined pytest | PASS | `154 passed in 0.95s`；failed/skipped=`0/0` |
| claim regression `py_compile` | PASS | exit=`0`，pycache 定向 `/tmp/cr172-s05-cp7r1-pycompile` |
| catalog whitespace | PASS | expected no-index diff exit=`1`，whitespace diagnostics=`0` |
| claim test whitespace | PASS | expected no-index diff exit=`1`，whitespace diagnostics=`0` |
| CP6R1 Story Return Check | PASS_WITH_WARNINGS | `Story Return Packet Check: OK`；仅既有 expected-path symlink warning |
| CP6R1 CP Result Check audit | PASS_WITH_WARNINGS | `CP Result Check: OK`；仅 INPUT_HASH_PATH_ESCAPE、FINAL_ATTEMPT_UNAVAILABLE、route-plan warning，无 ERROR |
| production/upstream hash | PASS | 8/8 与最终 CP7 基线一致 |
| 其他未改 S05 assets hash | PASS | sealed/failure/zero-op/fixture/integration/authz/README 7/7 与 CP6 基线一致 |
| CP6R1 delta hash | PASS | catalog=`242fbb127626...bafc66`；claim test=`c2fffc200a32...e5fce`，与 CP6R1 证据一致 |

production/upstream 8/8 当前 SHA-256：

| 文件 | SHA-256 | 结果 |
|---|---|---|
| `engine/path_i_governance.py` | `bc6abc95a0e660d71fce5567195ef6bff037e0919403b5d400841c579cc82abf` | MATCH |
| `engine/trial_return_artifact.py` | `923fc5e5baaafea88b6cb98d26de5d95b5d43d848fcf0cd51fc41a1190b645f0` | MATCH |
| `engine/research_artifact_replica.py` | `533d34f79e6481dd4bb2bcdb45776c186cb6e3f1a1f6994640f3382f6e466026` | MATCH |
| `engine/research_artifact_materialization.py` | `72541d73cff02bfbd0e20deb19141df63a9ac68389707f97ce4e252d829d008a` | MATCH |
| `tests/research/test_cr172_path_i_governance.py` | `ef4284860453c55a0b9a59357b13a5343ad88c488d5a7ac3fec63ab732de12f9` | MATCH |
| `tests/research/test_cr172_trial_return_artifact.py` | `4497e7b03920e239df752031b1564c09ef16c73687aa455485c6571d5b346969` | MATCH |
| `tests/research/test_cr172_nas_replica_verification.py` | `37c25a7a01bf46ba4fe121f25b57f9f95961e7eb155cb7e0eb49452c2621fd9b` | MATCH |
| `tests/research/test_cr172_execution_materialization.py` | `f58d71fd9f3cd7045d1cf645cea38e62370c0a705efbc3f627ed0f0a32be7049` | MATCH |

### 14.5 合同、失败恢复与零操作边界

CP7 Round 1 已独立逐项验证 artifact/authz/DAG/failure/REQ-013/deferred/claim/zero-op；CP7R1 在 8/8 production/upstream、7/7 其他 S05 资产哈希不变的前提下，重新执行覆盖这些路径的 27 个 scoped tests 和 154 个 combined tests，结果全部保持 PASS：

| 边界 | 复验结果 | 状态 |
|---|---|---|
| artifact 链 | payload=`2/2`；S02 seal bytes/digest/verifier=`1/1/1`；S03/S04 secondary digest=`0/0`；S04 verifier=`1`；trial/stage/pull=`1/1/1` | PASS |
| caller / authz | forward-label accepted=`0/0/0`；approved-ledger accepted/authorized/eligible=`0/0/0` | PASS |
| 六动作与 DAG | actions/records/enforcement=`6/6/6`；nodes/edges=`6/5`；cycle/unknown/permission-union=`0/0/0` | PASS |
| failure recovery | tamper/partial/staging/revoke 全 fail closed；selection advance=`0`；previous preserved=`2/2`；persist/CAS delta=`0/0` | PASS |
| REQ-013 split | contract-ready/runtime-enforcement/default-switch/runtime-delivered/future-prerequisite=`1/0/0/0/1`；new/legacy writable=`0/0` | PASS |
| deferred boundaries | 12 个 deferred counters 非零=`0/12`；无 runtime surface import | PASS |
| 五项 claim ceiling | true=`0/5` | PASS |
| 六类真实动作 | authorized/executed=`0/6`,`0/6` | PASS |
| 外部/高风险操作 | lake/NAS/runtime/network/credential/signal/trading/deploy/Git remote=`0` | PASS |

本轮源码、测试、fixture、设计、状态、CURRENT、change、context、ledger 写入均为 `0`；只写 packet 授权的五项 CP7R1 证据。未读取凭据，未连接外部环境，未授予或执行任何真实动作。

### 14.6 风险、决策与路由

| Risk | Severity | 当前状态 | 说明 |
|---|---|---|---|
| R-CP7-S05-TRACE-SEMANTIC-MISBINDING | BLOCKER | **CLOSED** | F-001 已由显式语义断言与独立 probe 关闭 |
| R-CR172-RUNTIME-AUTHORIZATION-GAP | HIGH | OPEN / unchanged / not authorized | fixture PASS 不授权真实运行 |
| R-CR172-REAL-R-DOMAIN-MISMATCH | HIGH | DEFERRED / unchanged | 正向 empirical-R/C1 仍不在 S05 范围内 |
| R-CR172-SYNC-REPLICA-STALE | HIGH | CONTRACT-CONTROLLED / runtime-unverified | 真实 adapter/runtime 未授权，不能由 fixture 推导 |

- CP7R1 decision：`PASS`。
- blocking findings：`0`；waivers：`0`；design delta：`0`。
- recommended_next_action：`proceed`。仅由 Host Orchestrator 按 route plan 准备 CP8；本结论不自行推进状态或写 ledger。
- runtime authorization：`0`。仓库内 fixture/static PASS 不等于真实数据、C1、Stage 3 或 runtime readiness。
- 待确认项：None。
