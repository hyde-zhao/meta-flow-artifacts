---
artifact: "CP5-CR172-LLD-REVIEW-R3-FINDINGS"
reviewer: "meta-qa-critical"
lane: "lane-quality"
input_type: "review_findings"
round: 3
status: complete
governance_mode: review-gated
---

# Review Findings

## 1. 审查范围

- 目标对象：CR-172 五份 full LLD v1.2、CR172-specific HLD/ADR v1.3、Feature v1.2、R2 findings 与 `CP5-CR172-LLD-R3-CROSS-CONTRACT-CORRELATION.result.json`。
- 审查目标：逐项判定 R2 的 `1 blocking + 3 required` 是否关闭；复核 approved-ledger hard deny、fixture provenance 单一真相、S03 selected-replica read、S04 bytes-level seal verifier、REQ-013 runtime-deferred 与零真实操作边界；查找新缺陷。
- 审查依据：`AGENTS.md` Design Review 1/3/5/8/10/12、R3 handoff 七项强制审查点、`review-artifact-protocol`、`quality-review` 和 R3 correlation result。
- 静态验证：五份 LLD 均重新通过 `meta-flow story lld-check --evidence-type full-lld`；S04 test ID total/unique/duplicate=`16/16/0`；S05 `evidence_kind` occurrence=`0`。本轮未读取/执行 mature runner，未执行实现测试或任何真实操作。

### 1.1 R2 Findings 关闭判定

| R2 Finding | R3 判定 | 证据与结论 |
|---|---|---|
| F-CR172-CP5-R2-001 approved-ledger self-assert | `CLOSED` | S01 current-v1 在 record/path/predecessor 判定前对任何 `approved_ledger` 固定 `authorized=false`、`eligible_to_execute=false`，reason only=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；六动作测试与 S05 oracle 均覆盖 caller enum/record 不解锁。 |
| F-CR172-CP5-R2-002 S04 bytes-level seal verifier | `CLOSED_AT_LLD` | S03 current selection 返回 immutable bundle/source-selection/receipt=`3/3`；S04 数据只来自 S03，S02 只作 verifier-library，typed staging=`3/3`，verifier exactly once；tampered seal→seal=false/pointer=0；bypass/receipt-only/secondary digest=`0/0/0`。上层 HLD/ADR 仍有一处不一致，作为新 finding F-R3-001 处理。 |
| F-CR172-CP5-R2-003 S05 evidence-kind second truth | `CLOSED` | S05 `evidence_kind` 字段/helper/assertion occurrence=`0`；唯一 provenance 为 `decision_origin + target_kind + fixture URI/port`。 |
| F-CR172-CP5-R2-004 REQ-013 runtime overclaim | `CLOSED` | 当前状态固定为 `contract_ready/runtime_enforcement_deferred`；runner diff/default switch/runtime enforcement/runtime-delivered=`0/0/0/0`；既有 future native-producer path-enforcement 子前置=`1`，CP8 禁止 runtime-delivered claim。 |

## 2. Findings

### Advisor Table（CP3 方案形成输入适用）

> 不适用。本轮为 CP5 full-LLD R3 独立复审，不是 CP3 方案形成输入。

<!-- findings-table -->

| ID | Severity | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|----------|----------|--------|------------|--------|
| F-CR172-CP5-R3-001 | required | `DR-1/3/8/12`、R3 handoff items 3/4 | 五份 LLD v1.2 与 Feature I02 v1.2 已明确选择：S03 verifier facade=`0`，S04 直接复用 S02 唯一 verifier-library；S03 LLD 把“提出 S03 verifier facade”列为 `NEEDS_DESIGN_CLARIFICATION` 触发条件。但 HLD v1.3 §10.2/§11.4 和 ADR-004 仍保留“或调用 S03 窄 verifier facade”的未限定备选。该备选没有 switch condition，且与 R3 handoff“不得提供 verifier facade”、S03/S04 LLD、Feature 已选依赖方向直接矛盾。 | CP5 同时批准 HLD/ADR 与 LLD 时会留下两种合法实现解读：实现者可按上层文档在 S03 新增 facade，而 LLD/测试要求其数量为 0。即使 facade 只委托 S02，它仍改变 S03 public surface、依赖边界和 `digest/verifier facade/data bypass=0/0/0` 的验收口径，导致 cross-contract correlation 可能假 PASS。 | 最小、不扩权整改：HLD §10.2/§11.4 和 ADR-004 删除“S03 窄 verifier facade”选项，唯一冻结为“S04 数据只来自 S03 current selection，验证只直接调用 S02 verifier-library”；同步修订记录并重跑定向文本/契约 correlation。五份 LLD、Feature、Story/Plan、源码、测试和权限范围均无需改变。 | `HLD v1.3 lines 351/430`；`ADR v1.3 line 114`；`S03 LLD §2 F-S03-11、§6/§12`；`S04 LLD §3/§6/§12`；`Feature I02 DESIGN §2/§5` |

## 3. 汇总结论

- blocking_count: 0
- required_count: 1
- optional_count: 0
- recommended_next_action: `revise-and-resubmit`
- decision_impact: `R2 四项实质缺陷均已在五份 LLD v1.2 中关闭，但 R3 的 proceed 条件要求 required=0。HLD/ADR 仍允许被 R3 明确禁止的 S03 verifier facade，因此当前不能建议 CP5 approve。完成上层文档单点收敛并定向复核后即可重新判断，不需要修改 LLD 或扩大实现/授权范围。`
- trade_off_note: `保留 facade 备选没有当前收益，却会制造 public surface 双解释；删除该备选只收敛契约，不改变数据流、文件 owner、Story DAG、测试数量或真实操作边界。`

## 4. 待确认项

- None。采用已由 Feature/LLD 明确选择的“直接 S02 verifier-library”即可，无需用户新增架构或授权决策。

