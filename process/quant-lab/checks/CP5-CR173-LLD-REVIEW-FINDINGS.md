---
artifact: "CP5-CR173-LLD-REVIEW-FINDINGS"
reviewer: "meta-qa-critical"
lane: "lane-quality"
input_type: "review_findings"
round: 1
status: final
governance_mode: review-gated
cr_id: "CR-173"
review_scope: "design-only"
---

# Review Findings

## 1. 审查范围

- 目标对象：`process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md`、`process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md`、`process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md`
- 审查目标：LLD↔HLD/ADR/Domain Map↔Feature↔Story 一致性；exact PSD/participation-ratio 可实现性；七字段、failure reason、identity/audit；6×3、8/8、7/7、8+12、10 类 zero counter；public/real/runtime deny-default 边界。
- 审查依据：`process/context/CP5-CR173.context.json`、`process/checks/CP4-CR173-STORY-DAG-PARALLEL-SAFETY.result.json`、Feature DESIGN/TEST-PLAN/TASKS、三张 Story 卡、HLD §5.2/§9、Domain Map §七字段/状态机、ADR-003/005、Dependency Map §Public C1 contract touch classification，以及 `AGENTS.md` Design Review 规则 1/3/5/8/12。
- 审查对象 SHA-256：S01 `7f677686e783a6dcc48e12cf2bb7e3dfce8fa6fecc010ba5182574ddcd29b12b`；S02 `9ba8f5dc71c2fd8b60db04c1b0e17488630c051c368a681c8e78bf0c34769f2a`；S03 `3fd2218bff0c0dddf61e4c8cfc3b3dc85caddee29aaf8a70ba0a166ac728fd7f`。
- 执行边界：未修改 LLD/Story/Feature/STATE/ledger；未实现源码、测试或 fixture；未运行 native/public tests；未发起或批准 CP5。

## 2. Findings

### Advisor Table（CP3 方案形成输入适用）

> 本产物 `input_type=review_findings`，不适用 CP3 advisor formation；保留模板结构，不倒填方案形成输入。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| N/A | N/A | N/A | CP5 design-only review | N/A | N/A |

<!-- findings-table -->

| ID | Severity | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|----------|----------|--------|------------|--------|
| F-CR173-CP5-001 | 一般（required） | Design Review 1/3/8/12；HLD §5.2；Domain Map §七字段；ADR-003/005 | HLD 要求同一 normalized input + method 的 `computation ref` 与 evidence hash 稳定；Domain Map/Feature 却把第七字段定义为 append-only computation/attempt ref；S01 LLD 将其改写为 stable content-addressed identity，并新增未进入 HLD/Domain Map/ADR/Feature 对象清单的外置 `ComputationAttemptAudit`，S03 又要求每次执行产生唯一 `attempt_audit_ref`。该分层在技术上可解 3/3 determinism 与逐执行审计冲突，但当前属于 LLD 对已批准权威合同的语义重分配。 | CP5 若直接批准，会让生产实现、fixture oracle、未来 persistence/projection owner 对“第七字段是逻辑 computation identity 还是执行 attempt identity”持不同解释；七字段 hash、failure recovery、append-only 审计与 future migration 均不可形成单一可验证合同。 | 必须在 CP5 前显式选择并回写权威基线。推荐保留 HLD 的 stable content-addressed `effective_trial_computation_ref`，在 HLD/Domain Map/ADR/Feature 中新增并定义外置 `ComputationAttemptAudit` 的 owner、identity 生成、与 evidence/hash 的链接、append-only/retention/persistence（当前可明确 N/A）及 F01-F08 attempt-basis canonical schema；同步修正第七字段措辞和 T01-T04 映射。备选是让第七字段逐执行唯一，但必须同步重写 3/3 hash/canonical evidence 规则，代价更高。 | `docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md:201`；`docs/design/DOMAIN-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md:60`；`docs/design/ARCHITECTURE-DECISION-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md:52,76-78`；`docs/features/effective-trial-offline-estimator/DESIGN.md:131`；`docs/features/effective-trial-offline-estimator/TEST-PLAN.md:145`；`process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md:100-102,147,155-159,297,313`；`process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md:154,162-171` |
| F-CR173-CP5-002 | 一般（required） | Design Review 1/5；Feature TEST-PLAN §4/§5；8/8 exact failure contract | Feature DESIGN/TEST-PLAN 把 nonfinite 同时放入 F03 numeric-grammar unsupported 与 F04 invalid matrix domain；S01/S02 接口只接受 canonical decimal string，`"NaN"`/`"Inf"` 在 parser 前即违反 grammar，因此 S03 的 GV-ET-05 实际固定为 F03，但 S03 的 failure matrix 仍声明 nonfinite→F04。 | 同一 nonfinite fixture 可因实现层次不同得到两个 reason，破坏 8/8 reason 精确断言、fixed precedence、method hash 覆盖的 reason enum，以及 recovery/audit 的 canonical identity。 | 冻结唯一可达映射并同步 HLD/Feature/S01-S03。推荐：所有非 canonical string token（含 `NaN`/`Inf`）归 F03；F04 仅接收已经成功解析为有限 exact rational、但 shape/symmetry/diag/range/PSD 失败的矩阵。若坚持 nonfinite→F04，则必须新增被合同允许的 nonfinite representation 和到达路径，并版本化 grammar/spec/hash。 | `docs/features/effective-trial-offline-estimator/DESIGN.md:140-153`；`docs/features/effective-trial-offline-estimator/TEST-PLAN.md:55,66-67`；`process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md:218-230`；`process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md:154,228-229,382` |
| F-CR173-CP5-003 | 一般（required） | Design Review 1/3/5；public deny-default boundary；8+12 inventory contract | S03 同时要求 `public C1 production call/write=0`，又要求 CP7 runner 对权威 12/12 existing regression/authorization paths 做 read-only selected run；这些既有回归会合法调用 current public types/adapters/gates。LLD 虽把 static boundary lane 与 regression runner 分开，但 zero counter 没有定义调用来源、阶段、采集边界或豁免，因此按全局字面口径无法同时为 0 和完成 12/12。 | CP7 可出现必然假失败，或实现者通过不运行 12/12 来伪造 zero-call；同时无法区分“CR173 新代码不得形成 public integration call edge”与“只读回归为了证明不恶化而执行既有 public 调用”。 | 将计数拆成可机械验证的作用域：`cr173_new_code_public_dependency_edges=0`、`cr173_new_code_public_calls=0`、`public_production_diff=0`、`public_writes=0`；另立 `cp7_read_only_public_regression_inventory=12/12` 和 `existing_expected_edits=0`。明确 selected regression 的既有 public 调用只属于 read-only verification lane，不计入 integration-call counter；任一写/expected relaxation 仍 deny-default。 | `docs/features/effective-trial-offline-estimator/TEST-PLAN.md:32,111-124`；`process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md:98,109,183,289-299,338` |
| F-CR173-CP5-004 | 轻微（optional） | Design Review 1/10；Story lifecycle consistency | 三张 Story 卡顶层 `status` 仍为 `planned-pending-cp4`，但各自 `lld_gate.status` 已是 `ready-for-cp5-review`，CP4 result 已 PASS；S01 卡仍称“本轮不创建 LLD”，Feature DESIGN 仍称 LLD 数为 0，而三份 LLD 已存在且 ready-for-review。 | 不改变算法或授权边界，但会让 CP5 queue、人工 Decision Brief 与后续机器/人工审计出现相互矛盾的 point-in-time 状态。 | 由 Story/Feature owner 在不改变设计语义的前提下刷新生命周期陈述，或显式标记这些句子为历史快照并引用当前 LLD；不得让顶层状态继续表示 CP4 pending。 | `process/stories/STORY-CR173-S01-contract-evidence-canonicalization.md:5,51,109`；`process/stories/STORY-CR173-S02-exact-spectral-estimator.md:5,56`；`process/stories/STORY-CR173-S03-golden-failure-boundary-verification.md:5,73`；`docs/features/effective-trial-offline-estimator/DESIGN.md:192` |

## 3. 汇总结论

- blocking_count: 0
- required_count: 3
- optional_count: 1
- recommended_next_action: `revise-and-resubmit`
- decision_impact: 三项 required finding 均可在设计层修正，不需要扩大 CR173 estimator-only 授权；但修正前不得把三份 LLD提交 CP5 人工批准。F-001 必须由权威设计 owner 明确 identity/audit 单一语义并回写基线，不能仅在 LLD 内解释。
- trade_off_note: exact-rational participation-ratio、pivot total order、fraction-free LDLT、双 invariant、一次 half-even、8+12 路径清单与 deny-default 文件所有权总体可实现；返工集中在权威语义一致性和验证计数边界，不要求放宽 exact/fixture-only/public-zero 目标。

已通过但不抵消 required finding 的重点审查结果：

- O-PSD-03 可达：对 `ρ=-0.9` 的 3×3 等相关矩阵乘 10 后，首轮 residual 为 `[[19,-171],[-171,19]]`；第二轮最后 residual 为 `(19·19-171²)/10=-2888`，确定进入 negative-pivot F04。
- O-PSD-04 可达：unit-diagonal 矩阵 `[[1,1,1],[1,1,-1],[1,-1,1]]` 首轮 pivot 后 residual 为 `[[0,-2],[-2,0]]`，确定进入 zero-diagonal + nonzero coupling F04。
- `q=n²/ΣRᵢⱼ²` 的 exact rational、未舍入/舍入后双 `[1,n]` invariant 与一次 half-even 设计闭合；pivot comparator 是 exact total order。
- Dependency Map 与 S03 的 public inventory 精确一致且文件均存在：production `8/8`、regression/authorization `12/12`，missing/duplicate/extra=`0/0/0`。
- Feature/Story/Wave/Task=`1/3/3/12`，七字段=`7/7`，public projection Feature/Story/Task=`0/0/0`；文件 owner 主路径 `7/7` 无重叠。

## 4. 待确认项

- required：由 architecture/methodology owner 决定并回写 F-001 的权威语义；推荐 stable `effective_trial_computation_ref` + 显式外置 append-only attempt audit 两层。
- required：由 contract/numeric owner确认 nonfinite 归 F03（推荐）还是新增可达 F04 representation。
- required：由 QA/verification owner冻结 public call zero counter 的来源/阶段作用域与 12/12 read-only regression 例外边界。

