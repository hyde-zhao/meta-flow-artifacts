---
artifact: "CP5-CR172-LLD-REVIEW-R1-FINDINGS"
reviewer: "meta-qa-critical"
lane: "lane-quality"
input_type: "review_findings"
round: 1
status: complete
governance_mode: review-gated
---

# Review Findings

## 1. 审查范围

- 目标对象：`process/stories/STORY-CR172-S01-*-LLD.md` ～ `STORY-CR172-S05-*-LLD.md` 五份 full LLD，以及 `engine/mature_multifactor_research.py`、`engine/experiment_family_lineage_store.py` 的 scoped source facts。
- 审查目标：跨 Story public contract、三项用户整改、现有 runner 可实施性、artifact/授权/claim 边界、失败恢复、27-scenario 验证设计与 CP5 准入条件。
- 审查依据：`AGENTS.md` Design Review 1/3/5/8/10/12、CP5 context、CP4 两份 PASS result、`review-artifact-protocol`、`quality-review` 与本轮 handoff 强制审查项。
- 静态验证：五份 LLD 均通过 `meta-flow story lld-check --evidence-type full-lld`；本轮未执行实现测试、真实数据或外部操作。

## 2. Findings

### Advisor Table（CP3 方案形成输入适用）

> 不适用。本轮为 CP5 full-LLD 后评审，不是 CP3 方案形成输入。

<!-- findings-table -->

| ID | Severity | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|----------|----------|--------|------------|--------|
| F-CR172-CP5-001 | blocking | `DR-1/3/5/12`、`S02 F-10` | S02 要求 lineage-finish 异常后 `finish success/new selection=0/0`，并计划 rollback 后调用 `lineage_trial.fail`；但现有 `_ProducerLineageTrial.finish()` 依次追加 `FinishAttempt(SUCCEEDED)`、`FinalizeTrial(SUCCEEDED)`、`RecordSelection`，不是原子操作。`fail()` 对前两个事件复用相同 event_id 但改写为 FAILED；`LocalFamilyLineageRecorder.submit()` 对同 event_id 不同 payload 返回 `EVENT_IDENTITY_CONFLICT`，且 recorder 明示无 resume/repair API。 | 若 `RecordSelection` 或中间 append 失败，已落盘的 SUCCEEDED 事件无法由 pointer rollback 或 `fail()` 撤销；LLD 的 fail-closed、成功 lineage 痕迹为 0 和测试 oracle 当前不可实现。继续 CP5 会把一个无法满足的 DoD 交给实现阶段。 | 在 CP5 前返回设计澄清：首选拆出/前置 lineage 原子批次、outbox 或正式 correction/supersession 合同，并由 lineage owner 授权；若不扩 lineage 能力，则必须收窄 HLD/Feature/S02 对 finish-failure 的声明，明确 partial lineage 进入 BLOCKED audit lane，不能声称 rollback 成功。两种路线都需重写失败顺序与 failure-injection tests；不得仅增加 `try/except`。 | `S02 LLD §2 F-10、§7 steps 3/6、§10 I01-F03、§14`；`engine/mature_multifactor_research.py::_ProducerLineageTrial.finish/fail`；`engine/experiment_family_lineage_store.py::LocalFamilyLineageRecorder.submit` |
| F-CR172-CP5-002 | blocking | `DR-1/3/5/12`、`ReturnDefinitionV1` 业务语义 | S02 把 `turnover.next_rebalance_date/net_forward_return` 固定映射为 `timestamp/simple_return`，并声明为 trial portfolio return。现有 runner 的 `net_forward_return` 实际由当前 `trade_date` 上选中股票的 `label_return` 均值减 turnover cost 得到；`label_return` 覆盖 `trade_date+1` 到 `trade_date+(label_horizon+1)`，而 `next_rebalance_date` 由独立的 `rebalance_step` 决定。默认值为 `label_horizon=20`、`rebalance_step=5`，因此 timestamp 不是该 return 的结束时点，连续观测还会重叠。 | 产物会把 forward-label proxy 误标为可比较的 trial portfolio period return。schema/hash 可完全正确，但业务语义错误；后续 empirical R、effective-count 和跨 trial 对齐会把 horizon/overlap 误差当成 trial 依赖，违背“真实 trial 回报”目标。现有测试只校验两列/排序/数值域，不能发现该错误。 | 在 CP5 前由 architecture owner 明确二选一：A. native producer 计算定义清楚的实际组合持有期回报，并让 timestamp 对应区间端点；B. 把当前值改为 versioned forward-label proxy object，显式记录 `label_horizon/rebalance_step/window semantics`，并禁止其进入 empirical-R/effective-count。若坚持 trial-return，ReturnDefinition/manifest、source mapping、测试与 HLD/ADR 必须同步修订。 | `S02 LLD §1、§2 F-05/F-06、§5 ReturnDefinitionV1、§8 native mapping`；`engine/mature_multifactor_research.py::build_research_frame` 与 `build_portfolio_path` |
| F-CR172-CP5-003 | required | `DR-3/5`、授权边界守卫 | S03/S04/S05 都要求 fixture decision 具有 `fixture_only`/`evidence_kind=fixture`，且不得提升为真实授权；但 S01 的 `ActionAuthorizationRecordV1`/`ActionDecisionV1` 没有 origin/evidence-kind 字段，S02 的 `PathITrialReturnProducerConfigV1` 也没有 `target_kind=repository_fixture` 绑定。S02 仅接收可由纯 evaluator 构造的 eligible decision 和显式绝对 `storage_root`。 | 合同无法机械区分 fixture allow 与真实 approval。测试可构造的 allow decision 在类型层面同样可传给 runner/artifact publisher；“六动作真实授权 0/6”目前只靠流程声明和测试环境约束，不能由 public API fail-closed 保证。 | 冻结一个可编码的 decision-origin/target binding：例如单独的 fixture decision 类型或 decision provenance 枚举，并要求 S02 config 将 `repository_fixture` 与 fixture decision、受限 root 一致绑定；非真实 approval adapter 产出的 decision 对 real target 必须在 first side effect 前拒绝。不得静默扩充 HLD 的 12-field approval record；若需改变 approval schema，回设计门。S05 增加“fixture decision + non-fixture target accepted=0”跨合同测试。 | `S01 LLD §5/§6/§12`；`S02 LLD §5 PathITrialReturnProducerConfigV1、§6 publish API、§12 fixture risk`；`S03 §8`、`S04 §8`、`S05 §5/§9` |
| F-CR172-CP5-004 | required | `DR-1/3/12`、cross-LLD contract correlation | S03 的输入仍写 generic `S02 sealed bundle/source selection`，数据模型还引用不存在的 S01 名称 `AuthorizationContextV1`；实际 S01 公共名是 `ActionScopeContextV1`。S03 receipt 要保存 `original-seal hash`，但 S02 只冻结 canonical seal bytes 和 `ArtifactSealV1`，没有 `seal_sha256` 的格式/domain、没有把它暴露在 `SealedTrialReturnArtifactRefV1`/`ResearchCanonicalSelectionV1`，`verify_sealed_trial_return_bundle()` 的返回值也不含 seal hash。 | S03 实现者必须自行选择 public type、打开哪些 path、如何计算 seal digest，形成第二套 seal 真相；S04 再消费该 receipt 时无法证明其 original-seal hash 与 S02 verifier 的同一 canonical bytes 绑定。并行起草虽合法，但跨 LLD correlation 尚未收敛。 | S02 显式导出 canonical seal bytes/hash 契约（命名、`sha256:` 格式、hash domain）并由 verifier 返回 verified seal digest；S03 API 精确声明 `SealedTrialReturnBundleV1`、`ResearchCanonicalSelectionV1`、`ActionDecisionV1`、`verify_sealed_trial_return_bundle`，receipt 的 `original_seal_sha256` 必须来自该 verified result。把 `AuthorizationContextV1` 更正为 `ActionScopeContextV1`，同步 S04/S05 inventory 与契约测试。 | `S02 LLD §5.1～5.3、§6`；`S03 LLD §3、§5、§6、§8 original seal`；`S04 LLD §5/§6` |
| F-CR172-CP5-005 | optional | `DR-2/11`、审查可读性 | S04 测试表重复列出 `T-S04-N03 unselected/stale receipt`；DoD 又写 `测试 trace=16/16`，当前表为 17 行/16 个唯一 ID。 | 不改变设计语义，但会让 CP6 test inventory、覆盖计数和审计报告出现 17/16 两种口径。 | 删除重复行或给第二行分配真实独立 ID/场景，并让 DoD 明确“16 个唯一测试 ID”。 | `S04 LLD §10、§14` |

## 3. 汇总结论

- blocking_count: 2
- required_count: 2
- optional_count: 1
- recommended_next_action: `revise-and-resubmit`
- decision_impact: `当前不能建议 CP5 approve。至少需修订 S02 的 lineage failure 与真实 return 语义，闭合 S01 fixture/real decision 边界和 S02→S03 seal/input public contract，再重新执行五 LLD correlation 与独立审查。`
- trade_off_note: `保留当前五 Story 切片可减少 CR 数，但会把不可原子 lineage 与非真实 period-return 的风险转移到 CP6；更稳妥的是先补齐方法/lineage 前置，或把本轮收窄为纯 schema/fixture contract，不实现 native runner hook。`

## 4. 待确认项

- `DQ-CR172-CP5-R1-01`：lineage 原子性走独立前置 CR/owner，还是收窄并诚实声明 partial-lineage BLOCKED audit lane？
- `DQ-CR172-CP5-R1-02`：native canonical source 改为真实组合持有期回报，还是把现有 `net_forward_return` 定义为不可进入 empirical-R 的 forward-label proxy？

