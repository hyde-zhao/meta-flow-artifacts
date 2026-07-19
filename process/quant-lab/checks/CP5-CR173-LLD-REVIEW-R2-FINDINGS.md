---
artifact: "CP5-CR173-LLD-REVIEW-R2-FINDINGS"
reviewer: "meta-qa-critical"
lane: "lane-quality"
input_type: "review_findings"
round: 2
status: final
governance_mode: review-gated
cr_id: "CR-173"
review_scope: "design-only-remediation-recheck"
---

# Review Findings

## 1. 审查范围

- 目标对象：Round 1 findings/summary、`process/handoffs/CR173-CP5-META-SE-DESIGN-REMEDIATION-RETURN-SUMMARY.md`、HLD/Domain Map/ADR v1.1、Feature DESIGN/TEST-PLAN/TASKS v0.2、三张 Story 卡、三份 LLD v1.2。
- 审查目标：逐项关闭 F-CR173-CP5-001..004；复核 stable computation ref、`EffectiveTrialAttemptBasisV1`、外置 audit owner/linkage/N/A lifecycle、F03/F04、public 双 lane、freshness，以及 exact PSD、7/7、6×3、8/8、8+12 和 authorization ceiling 是否回归。
- 审查依据：`review-artifact-protocol`、Round 1 findings/summary、`AGENTS.md` Design Review 规则 1/2/3/5/8/10/12，以及上述权威设计和 Story/LLD 证据。
- 审查对象 SHA-256：HLD `87a489e0b3290ed5b0652b2a264d6a3472ad7a3835aeb5e8dc93ae637cd327e8`；Domain Map `59f5d798915463e35861071fa07c15bd56a760a3f8681c35f0514189efbbac6b`；ADR `82c1dd850090caa0580dd2a1122a84d53edbba9d4326b00ed8688cb9c1ac8bb3`；S01/S02/S03 LLD 分别为 `ff3eeff27be91d385f6c705e0f40de20493925b3362e6e76a6c68b54d8219052`、`ac634d1eb03fc222098ad32284d436897e9a247182ee1b63b877140b58aab023`、`2861eab93aeb866e5f8a6def56c4435646721d41dc3837c3d773ee3f8448a9d2`。
- 执行边界：未修改 HLD/Domain/ADR/Feature/Story/LLD/STATE/checkpoint/ledger；未实现源码、测试或 fixture；未运行 native/public tests；未发起或批准 CP5。

## 2. Findings

### Round 1 关闭映射

| Round 1 Finding | Round 2 状态 | 证据与结论 | 对应 R2 finding |
|---|---|---|---|
| F-CR173-CP5-001 identity/audit | `NOT_CLOSED` | 权威 HLD/Domain/ADR/Feature 与 S01 已冻结 stable computation ref、七项 basis、外置 audit owner/linkage、persistence/retention=N/A；但 S03 把三位 presence bitmap 改成四组件，并在实际 outcome 产生前构造包含 outcome/failure ID 的完整 basis。 | F-CR173-CP5-R2-001 |
| F-CR173-CP5-002 F03/F04 | `CLOSED` | HLD/Domain/ADR、Feature 0.2 与 S01/S02/S03 统一：non-canonical string token（含 NaN/Inf）只走 F03；F04 只接全部 token 已解析为 finite exact rational 后的 matrix-domain failure；F03→F04/estimator call=0。 | None |
| F-CR173-CP5-003 public-call scope | `CLOSED_WITH_NEW_QUANTITY_FINDING` | 四个 CR173 new-code zero counters与 `12/12` read-only regression + expected edits 0 已分 lane，existing public calls 不再混入 new-code counter；但 S03 的“10 类 forbidden counter”数量与实际九项枚举不一致。 | F-CR173-CP5-R2-002 |
| F-CR173-CP5-004 freshness | `PARTIALLY_CLOSED` | 三张 Story 顶层已改为 `ready-for-cp5-review`，Feature 已承认三份 LLD 存在；但 HLD/Domain/ADR/Feature 状态仍为 awaiting LLD resync，三张 Story 正文仍称 LLD v1.1 待同步，实际已是 v1.2 ready-for-review。 | F-CR173-CP5-R2-003 |

### Advisor Table（CP3 方案形成输入适用）

> 本产物 `input_type=review_findings`，不适用 CP3 advisor formation；保留模板结构，不倒填方案形成输入。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| N/A | N/A | N/A | CP5 design-only remediation recheck | N/A | N/A |

<!-- findings-table -->

| ID | Severity | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|----------|----------|--------|------------|--------|
| F-CR173-CP5-R2-001 | 一般（required） | Round 1 F-001；Design Review 1/3/5；exact identity contract | HLD、Domain Map、Feature TEST-PLAN 和 S01 LLD 均定义 `presence_bitmap` 为 identity/envelope/method 三个布尔位，attempted evidence 只属于第四个 component snapshot digest；S03 LLD v1.2 却把 bitmap 写成 identity/dependency/method/evidence 四组件。S03 流程还在调用 S01/S02 estimator 之前先“构造完整七项 basis”，但 basis 本身含 `primary_failure_id` 和 actual outcome，因此发生结果前无法合法完成；把 expected outcome 预灌给 facade 会形成自证 oracle。 | S03 按四位 bitmap 计算的 canonical basis/computation ref 与 S01 生产合同不一致，3/3 identity/hash oracle 会跨 Story 漂移；前置构造 actual-outcome basis 会导致循环依赖或让验证使用期望值生成被验证对象，破坏 F01-F08 fail-closed 证据独立性。 | S03 必须严格复用三位 bitmap；attempted-evidence 只保留在 `component_snapshot_digests`，不得新增第四 presence bit。流程改为：case/raw components→S01/S02 validation/estimator 得到 actual outcome→finalize seven-item basis→stable computation ref→seven-field evidence/hash→external audit；expected basis 只用于事后比对，绝不能作为 facade 输入。同步修正 mermaid、步骤 2、API/fixture oracle，并明确 F01-F08 returned basis 与 expected oracle逐字段比较。 | `docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md:217`；`docs/design/DOMAIN-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md:75`；`process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md:163,230-237`；`process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md:170,226,246,274-284` |
| F-CR173-CP5-R2-002 | 一般（required） | Design Review 1/2/5；authorization inventory；Round 1 F-003 follow-through | S03 Goal/NFR/API/流程/测试/DoD 反复要求“10 类 forbidden/zero-operation counters”；但 §8.6 只枚举 credential、real data、lake/NAS、provider/network、catalog/store/pointer、strategy runtime、QMT/trading、publish/deploy、Git remote 共九项。§8.7 的四个 new-code public zero counters另行列示，且多处文本写成“10 counters + new-code 四计数”，不能机械推断为第十类。 | `10/10=0` 无法由设计直接计算，CP7 可能报告 9/10、重复计算 public 类，或静默遗漏一个授权类别；这违反量化目标和高风险 deny-default 清单的单一真相源要求。 | 冻结一张权威 operation-class inventory。若本次拆 lane 后非 public 类确实只有九项，统一改为 `9/9`，并单列 public new-code 四个 zero counters与 read-only两个计数；若仍要求 `10/10`，必须明确第十类名称、owner、采集边界及其与 §8.7 的非重复关系。同步 Story S03、LLD Goal/NFR/API/§7/§8.6/测试/DoD 和 Feature 验收计数。 | `process/stories/STORY-CR173-S03-golden-failure-boundary-verification.md:167`；`process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md:76,99,110,216,251,340-348,398,471` |
| F-CR173-CP5-R2-003 | 轻微（optional） | Round 1 F-004；Design Review 1/10；Story lifecycle consistency | 三张 Story 顶层已刷新为 `ready-for-cp5-review`，但正文仍声明“LLD v1.1 已存在且必须同步后重新评审”；实际三份 LLD frontmatter 均为 v1.2、`ready-for-review`。HLD/Domain/ADR 及 Feature triplet 的状态仍含 `awaiting-lld-resync`，Feature DESIGN 仍说 LLD 尚待 resync，TASKS 还写“CP5 前不创建 LLD”。 | 不改变数值、安全或授权语义，但会让 CP5 Decision Brief、current discovery 和人工审计误判 resync 尚未发生，F-004 的 point-in-time 一致性只完成了一半。 | 由权威设计/Story owner在 R2 结论落定后刷新为“LLD v1.2 resynced，R2 revise/re-review 状态”；保留 `confirmed=false` 与实现锁定。将 TASKS 的“也不创建 LLD”改为“不再创建/修改实现资产；LLD 已存在且只按审查结论修订”，或明确该句为历史快照。 | `docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md:2,378`；`docs/design/DOMAIN-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md:2`；`docs/design/ARCHITECTURE-DECISION-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md:2`；`docs/features/effective-trial-offline-estimator/DESIGN.md:4,214`；`docs/features/effective-trial-offline-estimator/TEST-PLAN.md:3`；`docs/features/effective-trial-offline-estimator/TASKS.md:3,64`；三张 Story `:109/:113/:131` |

## 3. 汇总结论

- blocking_count: 0
- required_count: 2
- optional_count: 1
- recommended_next_action: `revise-and-resubmit`
- decision_impact: F-001 的权威方向已正确，但 S03 对 canonical basis 的 exact schema 和生成顺序仍未落地一致；authorization counter cardinality 也未形成可计算清单。因此 Round 2 不能 proceed，也不得发起 CP5。两项 required 均可在设计层小范围修正，不需要扩大 CR173 estimator-only 授权。
- trade_off_note: 保留 stable computation identity + external audit 的两层设计仍是推荐方案；修复 S03 basis 消费顺序不会改变七字段或 hash domain。public 双 lane已解决原始 call-scope 冲突；只需把 non-public operation class 数量与 public 六计数的关系精确化。

已通过且未回归的复核结果：

- F03/F04 已完全唯一化：NaN/Inf/exponent/负零等 non-canonical token只走 F03，F04 从 finite exact-rational matrix 开始；F03→F04/estimator calls=`0/0`。
- exact PSD comparator、fraction-free LDLT、双 `[1,n]` invariant、一次 half-even未回归；O-PSD-03 仍可推导最后 residual `-2888`，O-PSD-04 首轮 residual 仍为 `[[0,-2],[-2,0]]`，目标分支均可达。
- 七字段=`7/7`、golden=`6/6×3/3`、failure=`8/8`、seven-field mutation=`7/7` 的设计覆盖仍完整；本 finding 只要求修正 S03 basis 构造，不否定这些目标。
- public inventory 仍精确为 production `8/8` + regression/authorization `12/12`；四个 new-code public counters目标均 0，read-only inventory=`12/12`、existing expected edits=`0`，existing public calls lane 解释正确。
- authorization claim ceiling未扩张：strategy/real/provider/credential/lake/runtime/trading/publish/deploy/remote、public projection、Gate1/DSR/admission、CR172 auto-resume 均保持 deny-default/0；当前缺陷是 operation-class 数量自洽，不是授权放宽。
- Feature/Story/Wave/Task=`1/3/3/12`，public projection Feature/Story/Task=`0/0/0`，文件所有权与串行 DAG 未回归。

## 4. 待确认项

- required：meta-dev/S03 owner按 F-CR173-CP5-R2-001 修正三位 bitmap 和 outcome 后置 basis 生成流程。
- required：QA/verification owner按 F-CR173-CP5-R2-002 冻结 9+public-six 或显式 10-class 的唯一计数清单。
- optional：权威设计/Story owner在下一轮前刷新 resync/re-review 状态。

