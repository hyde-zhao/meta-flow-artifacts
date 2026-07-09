---
discussion_id: "CP3-CR160-HLD-DISCUSSION"
cr_id: "CR-160"
stage: "CP3"
status: "completed"
created_at: "2026-07-08T23:36:39+08:00"
owner: "meta-se"
source_context: "process/context/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-DESIGN-CONTEXT.yaml"
question_permission: "relay-via-host-orchestrator; no direct user questions in this delegation"
---

# CP3 CR160 HLD Discussion Log

This log records pre-HLD architecture gray areas and advisor table-first analysis. It is not a CP3 human gate result.

## Architecture Gray Areas

| ID | 问题 | 为什么重要 | 推荐方案 | 备选方案 | 状态 |
|---|---|---|---|---|---|
| AGA-CR160-01 | Stage 4 是否接受 contract-only artifacts？ | 决定 review input lane 和最高可输出结论，直接影响 overclaim 风险。 | 接受 contract-only，但只允许低等级 fail-closed review conclusion。 | 完全排除 contract-only；或直接做 checker。 | selected |
| AGA-CR160-02 | CR155 existing real-data evidence 能否作为 seed？ | 需要用真实历史 evidence 证明 fail-closed 分类，同时不能授权新 lake access。 | 可作为 `blocked_admission_failed` seed，只消费 existing evidence。 | 不使用 CR155；或把 CR155 作为 observation candidate。 | selected |
| AGA-CR160-03 | observation plan 是 template 还是 instance？ | Template/instance 混淆会让 future Stage 3 package 只引用模板也被误判通过。 | CR160 定义 `observation_plan_template`；future Stage 3 产出 `observation_plan_instance`。 | 让 `observation_plan_ref` 直接指向模板；或暂不定义 plan。 | selected |
| AGA-CR160-04 | Review 是 manual、automatic 还是 mixed？ | 决定当前 CR 是否需要 code/schema/checker 和 CP4/CP5/CP6。 | Manual checklist + post-review decision table；checker out of scope。 | Automatic checker now；或 Stage 4/5 combined gate。 | selected |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Manual layered review + fail-closed decision table | 最小关闭 Stage 4 语义缺口；符合 design-only；直接控制 no-overclaim。 | 人工一致性成本高。 | HLD, checklist, CP3 result, CP7 verification inputs | Recommended | 若后续 review 频繁或差异大，切换到 checker CR。 |
| Automatic checker now | 一致性更强，可批量执行。 | 超出 CP2 deliverable shape；需要 Story/LLD/code/test。 | engine, tests, CP4/CP5/CP6, schema ownership | Deferred | 仅在用户授权 implementation CR 且 instance schema 稳定时切换。 |
| Stage 4 + Stage 5 combined gate | 一次性连接 paper/simulation admission。 | 授权边界过宽，容易 runtime leak。 | Stage 5, runtime risk, authorization gate | Rejected for CR160 | 独立 Stage 5 CR 和 runtime/simulation authorization gate 通过后再考虑。 |
| Contract-only exclusion | 最保守，避免低证据被误用。 | 无法给 early-stage artifacts 提供 remediation route。 | Product workflow, evidence classification | Governance fallback | 若 contract-only 仍被误读为 readiness，切换到 exclusion。 |

## Lane Summary

| Lane | Finding | HLD impact |
|---|---|---|
| lane-product | Stage 4 需要给用户一个可解释分类，而不是只说 pass/fail。 | Decision table 输出 classification + escalation route。 |
| lane-architecture | `observation_plan_template` 和 `observation_plan_instance` 必须分离。 | HLD §7 规定 producer/reviewer/compliance relation。 |
| lane-quality | Checklist 必须覆盖 Stage 1/2/3/cross-cutting controls，不只看 Stage 3 admission package。 | Checklist 分四层，并含 fail-closed rules。 |
| lane-docs-check | Non-authorization wording 必须出现在 HLD、checklist、seed classification 和 CP3 result。 | 所有产物重复声明不授权 runtime/data/paper/simulation。 |

## Selected Defaults

| Decision | Selected default | Reason |
|---|---|---|
| Evidence profile | `contract_only`, `real_data_validated`, `runtime_authorized`, `unknown` | Matches CP2 decision and HLD decision table. |
| Contract-only ceiling | No paper/simulation/runtime readiness | Prevents overclaim. |
| CR155 classification | `blocked_admission_failed` | CR155 has `BLOCKED`, statistical `FAIL`, `paper_candidate=false`. |
| Observation plan object | Template now, future instance later | Avoids false PASS on template-only refs. |
| Review mechanism | Human checklist, post-review decision table | CR160 is design-only. |

## Deferred Ideas

| ID | 内容 | 延后原因 | 后续触发 |
|---|---|---|---|
| DAI-CR160-01 | Automatic observation review checker | Requires code/schema/tests and implementation gate. | Follow-up CR. |
| DAI-CR160-02 | Persistent observation review ledger | Requires storage ownership and migration/rollback design. | Audit/scale requirement. |
| DAI-CR160-03 | Stage 5 paper/simulation authorization gate | Separate authorization boundary. | Stage 5 CR. |
| DAI-CR160-04 | Data-lake governance remediation workflow | Would involve data operations/governance routes. | Data governance CR. |

## Human Discussion Summary

No direct user questions were asked in this meta-se delegation because the handoff set `question_permission.can_ask_user=false`. CP2 already approved the five decisions needed for CP3 drafting. Remaining CP3 decisions are for host-orchestrator to present at the formal CP3 human gate.

