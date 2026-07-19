---
artifact: "CP5-CR172-LLD-REVIEW-R2-FINDINGS"
reviewer: "meta-qa-critical"
lane: "lane-quality"
input_type: "review_findings"
round: 2
status: complete
governance_mode: review-gated
---

# Review Findings

## 1. 审查范围

- 目标对象：`process/stories/STORY-CR172-S01-*-LLD.md` ～ `STORY-CR172-S05-*-LLD.md` 五份 full LLD v1.1，以及 CR172-specific HLD v1.2、ADR v1.2、三个 Feature v1.1、R2 cross-contract correlation 与 R1 findings。
- 审查目标：逐项复核 R1 F-001～F-005 的关闭证据；独立检查授权来源信任边界、S02→S03→S04 seal 验证链、REQ-CR172-013 新路径交付口径、S05 零操作证据与公共合同一致性。
- 审查依据：`AGENTS.md` Design Review 1/2/3/5/8/10/12、R2 handoff 六项强制审查点、`review-artifact-protocol`、`quality-review`、CP5 context 和 R2 correlation result。
- 静态验证：五份 LLD 均重新通过 `meta-flow story lld-check --evidence-type full-lld`；本轮未执行实现测试，未读取/执行 mature runner，未进行真实 lake/NAS/runtime/交易或外部操作。

### 1.1 R1 Findings 关闭判定

| R1 Finding | R2 判定 | 证据与结论 |
|---|---|---|
| F-CR172-CP5-001 lineage atomicity | `CLOSED` | S02 不再调用 lineage；partial success 固定为 `partial_lineage_blocked_audit`，erase/fake rollback/canonical selection advance=`0/0/0`；真实原子 lineage 保留为独立 owner 前置。 |
| F-CR172-CP5-002 return semantics | `CLOSED` | S02 已收窄为显式 period-return repository fixture pure contract；现有 runner/lineage planned diff=`0/0`；`forward_label_proxy@v1` 进入 trial-return/R/effective-count=`0/0/0`。 |
| F-CR172-CP5-003 fixture/real decision boundary | `PARTIALLY_CLOSED` | 12-field record 保持 `12/12`，fixture origin + real target 在 consumer first side effect 前拒绝；但 `approved_ledger` 仍是 caller 可直接传入的枚举，缺少可信 adapter 时 evaluator 仍可产生真实 action allow，详见 F-R2-001。 |
| F-CR172-CP5-004 S02→S03 seal/public contract | `CLOSED_FOR_S02_TO_S03` | S02 导出三个精确 public type、唯一 canonical seal bytes/digest/verifier；S03 调用同一 verifier并透传 `VerifiedTrialReturnBundleV1.original_seal_sha256`，secondary canonicalizer/digest/re-seal=`0/0/0`。S04 的下一跳出现独立新缺陷，详见 F-R2-002。 |
| F-CR172-CP5-005 S04 test ID | `CLOSED` | S04 §10 测试 ID total/unique/duplicate=`16/16/0`。 |

## 2. Findings

### Advisor Table（CP3 方案形成输入适用）

> 不适用。本轮为 CP5 full-LLD R2 独立复审，不是 CP3 方案形成输入。

<!-- findings-table -->

| ID | Severity | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|----------|----------|--------|------------|--------|
| F-CR172-CP5-R2-001 | required | `DR-1/3/5/12`、R1 F-003、HLD real-action ceiling | S01 声明“当前没有 approved-ledger adapter，六类真实动作 authorized/executed=`0/6`,`0/6`”，但唯一 evaluator 把 `decision_origin` 作为 caller 参数，并把 `(approved_ledger, real_operation, approved logical URI)` 直接列为有效 typed binding；固定 reason code 中没有 `APPROVED_LEDGER_ADAPTER_UNAVAILABLE`，处理步骤在 record 有效后即设 `authorized=true`。因此调用方仅构造 12-field record 并传枚举值，就可让 root `data_lake_read` 得到 `eligible_to_execute=true`；“无 adapter”只是文字约束，不是可编码守卫。 | 真实 approval provenance 可被自报枚举伪造，直接与 HLD/Feature 的 real action `0/6` 和“当前不提供真实授权”矛盾。未来任何 real consumer 若只调用 `require_action_eligible(...expected_origin=approved_ledger)`，会接受伪造 decision；安全边界未在 first side effect 前闭合。 | 最小不扩权整改：当前 v1 evaluator 对所有 `approved_ledger` 输入固定返回双 false，并增加稳定 reason `APPROVED_LEDGER_ADAPTER_UNAVAILABLE` 及 `approved_ledger+real_operation accepted/eligible=0/0` 测试；未来真实 adapter 必须由独立 runtime-high-risk CR 引入可信 issuer/verified envelope 或独立 entrypoint，不能靠 caller 传枚举解锁。12-field record 不变。 | `S01 LLD §6 lines 125-132、§7 lines 159-171、§10`；`HLD §10.4 line 364`；`Feature I03 DESIGN §3 line 66` |
| F-CR172-CP5-R2-002 | blocking | `DR-1/3/5/12`、ADR-004、S04 F-S04-05/NFR-S04-01 | S04 承诺从 execution staging 对 release/manifest/original seal/content 做 bytes-level `4/4` 复验，且 receipt 不得替代 bytes；但 S04 的精确依赖只有 S01 decision/context 与 S03 receipt/selection，`MaterializationStoragePort.pull_to_staging(...)` 只返回未定义的 pull outcome，未返回 `SealedTrialReturnBundleV1` 或 seal bytes，S04 也不消费 S02 `verify_sealed_trial_return_bundle`/canonical seal contract。与此同时 S04 禁止生成新的 original seal digest。仅把 receipt 中的 digest 与另一个 metadata digest比较，无法证明 staging seal bytes 及其 logical URI/content/manifest/release 绑定。 | `4/4` 的 seal bit 当前不可实施：实现者只能 A. 信任 S03 receipt（违反“receipt 不替代 bytes”），或 B. 在 S04复制 seal parser/canonicalizer/digest（违反唯一 seal 真相），或 C. 擅自增加未审查的 S02/S03 API。核心完整性 DoD、失败注入和执行机 pull 后复验都无法按现有 LLD实现。 | 在 CP5 前闭合 verifier-only 依赖，且不扩大真实权限：推荐 S04 的数据来源仍只能是 S03 selected replica，但 staging port 必须返回精确 sealed-bundle value；S04 复用 S02 唯一 `verify_sealed_trial_return_bundle`（可经 S03 的窄 verifier facade 委托）完成原 seal bytes 复验。同步 Feature “S04 禁止直接消费 S02”的表述为“禁止绕过 S03 selection 取数，但允许 verifier-library dependency”，并增加 tampered seal bytes 使 4/4 seal=false、pointer advance=0 的测试。S04 不得自建第二 digest。 | `S04 LLD §2 lines 65-76、§5 lines 124-130、§6 lines 138-146、§7/§8 lines 183-200`；`Feature I02 DESIGN lines 53-57/74`；`ADR-004 lines 98-113` |
| F-CR172-CP5-R2-003 | required | `DR-1/3/12`、cross-LLD exact type | S05 已在大部分位置使用 S01 的 `decision_origin=repository_fixture`，但 `ZeroOperationOracleV1`、安全表和风险表仍要求 fixture decision 带 `evidence_kind=fixture`。S01 `ActionDecisionV1` 没有 `evidence_kind`，R2 正是以 `decision_origin` 替代 R1 中未冻结的 evidence-kind 概念；S05 又引入同义标记，形成第二个来源真相。 | CP6 实现者可能在 fixture JSON/test helper 中复制 provenance 规则，导致 S05 与 S01 public type 不一致；验证可能只检查自建 `evidence_kind`，却没有验证真实 `ActionDecisionV1.decision_origin`，产生假 PASS。 | 删除 S05 当前切片中的 `evidence_kind=fixture` 别名，所有 fixture/real 判定只消费 `ActionDecisionV1.decision_origin`、`ActionScopeContextV1.target_kind` 和 `fixture://`/port binding；zero-operation oracle 只记录独立的 real authorized/executed counters，不重定义 decision provenance。 | `S05 LLD §5 ZeroOperationOracleV1 line 161、§9 authorization safety line 291、§12 risk table line 362`；`S01 LLD §5 lines 107-113` |
| F-CR172-CP5-R2-004 | required | `DR-1/2/3/12`、`REQ-CR172-013` | REQ-013 要求新 run 默认根实际替换为 `multifactor-strategy-research/{run_id}`，新 run 写 legacy=`0`；但 R2 已明确 mature runner diff=`0`，S01 只提供无 I/O 的 `decide_run_path`，S05 只验证 value contract。HLD 一面把 REQ-013 trace 标为 `1/1`，一面又声明不修改 runner、native runtime deferred；没有 runtime enforcement point、owner 或 follow-up closure 条件。 | 当前实现最多证明“新路径决策合同存在”，不能证明任何实际新 run 不再写 legacy。若 CP8 把 REQ-013 记为完成，会形成合同 ready 冒充 runtime default switch；若不留显式前置，未来 native producer 仍可能沿用旧默认。 | 最小不扩权整改：保持 runner diff=`0`，把本 CR 的 REQ-013 结论改为 `contract_ready/runtime_enforcement_deferred`；在既有 `FU-S02-NATIVE-PRODUCER` 下登记 path-enforcement 子前置（runtime/data owner，触发于未来真实 producer CR），要求其在 launch/workspace first side effect 前消费 `RunPathDecisionV1`，证明 new default=`1`、legacy write=`0`。S05 增加 `runtime path enforcement/default switch count=0` 并禁止 CP8 声称 REQ-013 runtime delivered；无需当前修改 runner或新增授权。 | `REQUIREMENTS §REQ-CR172-013 lines 981-985`；`HLD §7 trace line 214、§11.6 lines 439-441`；`S01 LLD §5/§6 RunPathDecisionV1`；`S05 LLD SC-CR172-G02/Legacy tests` |

## 3. 汇总结论

- blocking_count: 1
- required_count: 3
- optional_count: 0
- recommended_next_action: `revise-and-resubmit`
- decision_impact: `不能建议 CP5 approve。R1 五项中 4 项已关闭或在原边界关闭，F-003 仍只部分关闭；此外 S04 的 execution-staging seal 复验缺少可实施的唯一 verifier 链，REQ-013 交付口径也把 contract-ready 与 runtime enforcement 混为一谈。完成四项最小整改并重新 correlation/review 前，应继续停留在 CP5 准备阶段。`
- trade_off_note: `整改不需要扩大真实运行权限：S01 只需在当前 slice 机械拒绝 approved-ledger；S04 只需补 verifier-library 依赖而不允许绕过 S03 selection；REQ-013 只需诚实降格为 contract-ready 并绑定未来 native-producer 前置；S05 删除同义 provenance 标记。代价是多一轮设计修订，收益是避免把伪授权、metadata-only seal 比对和未接线的新路径误判为已交付。`

## 4. 待确认项

- None。四项均可按上述最小、不扩权路线机械整改，不需要新增用户授权或改变已批准 PATH-I 范围。

