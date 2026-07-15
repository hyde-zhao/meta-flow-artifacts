---
title: "Stage 3 Launch / Real-Lake Entry Decision Gate — HLD"
cr_id: "CR-171"
stage: "CP3"
status: "approved-cp3"
scope: "design-only authorization boundary, historical-evidence classification, and later activation-CR contract"
---

# Stage 3 Launch / Real-Lake Entry Decision Gate — HLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-07-15 | meta-se-critical | 新建 §1–§23：将 CP2 已批准的 C1–C4 路线、事件锚定 waiver、候选只读合同和历史证据边界固化为 CP3 可审查架构；不引入实现或真实数据动作。 |
| v0.2 | 2026-07-15 | host-orchestrator | 评审修订：新增 §21 Open Items 状态台账（OPEN/RESOLVED、日期、owner、闭环条件）；§18/§19 增加状态闭环与 ADR 逐项重开条件的验证/回退约束。 |
| v0.3 | 2026-07-15 | user / host-orchestrator | 用户接受 v0.2 修订并批准 CP3 的 4 项推荐；仅解锁 CP7 design verification，未产生运行授权。 |

## 1. 问题定义、目标与当前事实

CR-170 已完成 canonical hardening，但并未把当前 runner 接入 canonical gate，`R-CR170-RUNNER-GAP` 仍受控且 `stage3_started=false`。CR-171 的职责不是启动 Stage 3，而是在未读取真实数据、未运行 producer 的情况下，把下一步的架构、授权边界和历史事实处理方式变成可审计的决策合同。

CP2 已批准三个输入：

| 决策 | 已批准值 | 本 HLD 的后果 |
|---|---|---|
| 证据路线 | `c1_c4_real_producer` | 采用两个入口 CR：CR-171 为决策/边界/事实收敛；后续 Real-Evidence Activation CR 才处理 producer mapping、release/PIT/lineage binding、real computation authorization 与 run identity。 |
| verifier | `event_bounded_waiver` | FU-006 可在此决策 CR 后延后，但两个事件任一临近即失效，不能继承 CR-170 waiver。 |
| future read scope | `scoped_research_data_lake_read_only` | 本 HLD 只定义五字段候选和 deny-default 规则；不形成 read authorization。 |

本设计的量化成功标准为：五字段 allow contract 5/5、deny-default 类别 6/6、C1–C4 feasibility verdict 4/4、waiver 失效事件 2/2、revalidation 合法 current-entry verdict 3/3、CP3 设计期间真实 read / computation / write / runtime / trading 计数均为 0。

## 2. 目标与非目标

### 2.1 目标

1. 明确 CR-171 与未来 Real-Evidence Activation CR 的职责、所有权和输入输出契约。
2. 以静态、已声明的合同事实为依据，完成 C1–C4 对“冻结 release schema + PIT + lineage”候选的设计时可行性判定。
3. 给 CP3 人工门禁一份可冻结、但尚未授权执行的五字段 read contract candidate。
4. 将 historical Stage 3 evidence 的处理限制为 classification、legacy annotation 和 report，并以事件式 waiver 防止成熟准入绕过 FU-006。

### 2.2 非目标

- 不访问 lake/NAS/provider、凭据、环境或已有真实 research 输出；不创建 output directory。
- 不执行 C1–C4、current runner、aggregate orchestration、CR155 promotion、simulation、paper/live/QMT/broker/trading。
- 不编写代码、测试、schema、Story、LLD 或执行工具；CP4–CP6 因本 HLD 仍为 N/A。
- 不修复 CR-032、CR-010、CR-018、CR-031、CR-033 或 CR-168 的 tracking debt；仅在风险隔离中披露。
- 不修复、回填、重跑、重写历史 data/schema/PIT/lineage/code/manifest/evidence。

## 3. Blueprint 适用性与 HLD 拆分判定

当前核心产物只有一个：**Stage 3 entry decision-and-boundary contract**。其中 revalidation、waiver 和 activation boundary 都服务于同一份授权/claim-ceiling 合同；没有独立可交付实现、Story 或运行时组件。因此 standalone BLUEPRINT/DOMAIN-MAP/DEPENDENCY-MAP 及 HLD 拆分均为 `N/A`。若未来 activation CR 产生超过 5 个可独立交付的 producer/adapter Story 或独立 runtime contract，必须重新运行 blueprint 和 HLD 拆分判定。

## 4. Architecture Gray Areas 与 advisor table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| AGA-1：C1–C4 后续 activation | 让 canonical hardening 被真实 producer 消费；映射、PIT/lineage 和 computation 可独立审计 | 需要第二个入口 CR，当前不产生任何运行结果 | architecture、authorization、future research | 推荐，CP2 已选 | 若 C1–C4 静态合同判定全体 incompatible，回到 CP2/新 CR 重新评估 current runner，不可就地切换。 |
| AGA-2：current runner 直接路线 | CR 数最少 | `R-CR170-RUNNER-GAP` 持续；必须在 CR-171 冻结完整 runner read boundary | runner、canonical、安全 | 不选 | 仅在用户重开 CP2 且接受 runner gap 时可用。 |
| AGA-3：event-bounded FU-006 waiver | 不阻塞决策/设计；失效点可机械判定 | 成熟准入前必须完成独立 verifier | verification、admission gate | 推荐，CP2 已选 | 如希望在首个 admission 前消除 waiver，则独立提升 FU-006。 |
| AGA-4：五字段值由 CP3 推断 | 快 | 从文档推断真实 release/目录会构成伪授权 | authorization、data governance | 不选 | 只有人类在 CP3 以明确授权对象冻结 5/5 字段，才可进入后续 activation 决策。 |
| AGA-5：revalidation 顺手修复 | 可能减少后续 CR | 将决策门膨胀为实现/数据修复 CR，破坏事实边界 | historical evidence、scope | 不选 | 仅通过 `incompatible_rework_required` 路由一个新 CR。 |

上述表是方案形成输入，不虚构额外 reviewer lane。其事实来源为 CP2 result、`REQ-CR171-001..005`、`UC-58-CR171` 及 `MULTIFACTOR-RESEARCH.md` 的 legacy marker。

## 5. 候选方案比较与推荐

| 方案 | 结构 | 优点 | 主要代价 | 适用条件 | 结论 |
|---|---|---|---|---|---|
| A（推荐） | CR-171 决策门 + 后续 Real-Evidence Activation CR | 将授权决策、静态可行性与真实 binding/computation 分离；canonical 可被 C1–C4 消费 | 需要独立 activation CR | C1–C4 至少存在可通过新 binding 补齐的路径；每项真实操作另行授权 | 采用 |
| B（备选） | CR-171 + current runner 受限只读路径 | 入口 CR 少 | runner gap 持续，CR-171 必须承担完整执行边界 | 用户重开 CP2 并明确接受 runner 路线 | 不采用 |
| C（不采用） | 将 activation/FU-006/aggregate 并入 CR-171 | 表面 CR 数少 | 混合 high-risk authorization、implementation、aggregation 和 CR155 promotion 风险 | 无；与 CR-171 decision-only 宗旨冲突 | 拒绝 |

**推荐方案 A**。它保留 CP2 已批准的 `C1–C4 + event_bounded_waiver`，并把真实操作放到可独立审查的后续 CR。切换/回退必须通过新的 CP2 决策或 follow-up CR，不能在 CP3 文档内更换路线。

## 6. 目标架构与职责边界

```text
CP2 approved decisions
        |
        v
CR-171 (this HLD / CP3)
  - decision record & claim ceiling
  - candidate read contract (no permission)
  - C1-C4 static feasibility verdict
  - revalidation classification contract
  - event-bounded FU-006 waiver state machine
        |
        | only after separate human-authorized CR
        v
Real-Evidence Activation CR (future)
  - real-data-to-producer mapping
  - frozen release + schema/PIT/lineage binding
  - real computation authorization
  - producer-result <-> Stage 3 run identity binding
        |
        v
later FU-007b / maturity gates
  - C1-C4 aggregate orchestration
  - CR155 regression / promotion decision
```

| Object / owner | CR-171 owns | Future activation CR owns | Explicitly not owned here |
|---|---|---|---|
| research-data read boundary | candidate selection rules and deny-default | exact runtime use only after separate authorization | credentials, provider fetch, write, catalog pointer mutation |
| C1–C4 | static contract feasibility verdicts | real input mapping, computation authorization, output/run binding | execution, test, aggregate |
| historical evidence | classify, annotate legacy, report | no ownership unless new CR says otherwise | repair/backfill/rerun/manifest rewrite |
| verifier | waiver state machine | consumes only after FU-006 completion | independent verifier implementation |
| aggregate / CR155 | no ownership | no ownership until FU-007b | aggregate and promotion |

## 7. Five-field future read-contract candidate

The following is a *candidate specification*, not an authorization, credential, command, path creation instruction or read permission. A value may not be inferred from historical documents or legacy run paths.

| Field | CP3 candidate selection rule | CP3 value | Activation-CR acceptance criterion |
|---|---|---|---|
| `data_release` | Immutable release identifier plus manifest/version/hash reference, selected by authorized data owner | `pending_human_authorization` | one immutable release ID and one corresponding provenance reference |
| `datasets` | Explicit named dataset list; no wildcard family or implicit catalog expansion | `pending_human_authorization` | finite allow-list with schema/version per dataset |
| `date_range` | Closed start/end inclusive range within selected release availability and PIT policy | `pending_human_authorization` | one bounded range and declared PIT cut-off semantics |
| `read_identity` | Named least-privilege research identity, read-only and non-credential-discovery | `pending_human_authorization` | identity owner, read-only grant reference and revocation route |
| `output_directory` | Pre-existing approved research evidence destination; no lake/NAS/catalog/current-pointer location | `pending_human_authorization` | exact approved path plus write/retention authority, separately authorized if writing is needed |

Denied by default (6/6): credential/environment read; provider fetch; lake/NAS write; catalog/current-pointer mutation; runtime/real computation; trading/broker activity. The fifth field does not authorize a write; if future evidence must be written, its storage authority is a separate explicit decision.

**CP3 human action requested:** freeze each of the 5 values or intentionally retain it as `pending_human_authorization`. The latter is a valid CP3 result and means activation cannot open.

## 8. C1–C4 static producer feasibility matrix

This matrix uses only declared/static contract facts: C1 has an effective-count `typed_unavailable` ceiling; Stage 2 handoff requires data release, PIT, lineage and evidence refs; C3/C4 boundaries reserve calculation/aggregate work; no lake content, schema, manifest or output was inspected.

| Producer | Required future inputs under selected contract | Static fact available now | Design-time verdict | Why | Required future disposition |
|---|---|---|---|---|---|
| C1 | frozen release reference, PIT-aware research input, sealed lineage/family and effective-count semantics | lineage contract exists but `effective_trial_count=typed_unavailable`; no real release binding | `insufficient` | release/schema/PIT/lineage binding and required input sufficiency cannot be proved statically | activation CR must map inputs and obtain computation authorization; no current-entry claim |
| C2 | frozen release fields, PIT/availability semantics, evidence lineage and deterministic input contract | static contract/refs-only evidence boundary exists; no real schema or PIT mapping | `insufficient` | static description cannot show the selected release satisfies C2 inputs | activation CR must map and validate named input contract |
| C3 | frozen release fields for cost/impact evidence, PIT/lineage and authorization boundary | C3 is a typed/static boundary; real calibration/computation remains unapproved | `insufficient` | no authorized real inputs, calibration or schema binding exists | activation CR must define mapping and separately authorize real computation |
| C4 | frozen release fields for capacity/liquidity, PIT/lineage and aggregation relation | C4 calculator/aggregate remains deferred; `FU-007b` owns aggregate exit work | `incompatible` | current declared boundary has no C4 real producer computation/aggregate path available for activation as-is | new C4-specific implementation/authorization CR is required before C4 can change verdict |

`insufficient` is not a failure of CR-171: it is the truthful design outcome when facts are missing. `incompatible` likewise routes to rework; neither permits a workaround through current runner, aggregate, CR155 promotion or a historical run.

## 9. Historical-evidence revalidation contract

Historical evidence remains annotated `legacy / require-revalidation`. The CR-171 revalidation lane has exactly three operations: **classification, legacy annotation, report**. It may not repair, backfill, rerun, rewrite manifests, mutate schema/PIT/lineage or remediate defects.

| Report verdict | Meaning | Current claim effect | Required route |
|---|---|---|---|
| `revalidated_for_current_entry` | An independently authorized future audit found the declared current-entry evidence contract satisfied | still does **not** set `stage3_started`, `stage3_entry_ready`, `real_data_read_authorized` or `real_computation_authorized`; it is evidence input only | requires later activation/admission controls |
| `insufficient_for_current_entry` | required current evidence is absent, stale or cannot be proven | all entry/maturity claims remain false | record gap; no repair in CR-171 |
| `incompatible_rework_required` | evidence/contract conflicts with the approved current-entry contract | all entry/maturity claims remain false | separate rework CR only |

`reaffirmed_as_legacy_only` remains a permissible annotation of the historic record, not a fourth current-entry verdict. This resolves the product-document legacy narrative without expanding the CP2/CP3 current-entry report schema.

## 10. Event-bounded FU-006 waiver state machine

```text
DESIGN_ONLY (CP2 approval) --> WAIVER_ACTIVE_FOR_DESIGN
WAIVER_ACTIVE_FOR_DESIGN --[E1: before first real-evidence admission PASS/PASS_WITH_RISK]--> EXPIRED_BLOCK_ADMISSION
WAIVER_ACTIVE_FOR_DESIGN --[E2: before Stage 3 exit-gate start]-------------------------> EXPIRED_BLOCK_EXIT
EXPIRED_* --[independent FU-006 evidence accepted]--------------------------------------> VERIFIED_REQUIRED_GATE
```

| State | Permitted | Forbidden / mechanical guard |
|---|---|---|
| `WAIVER_ACTIVE_FOR_DESIGN` | CR-171 decisions, design documentation, static analysis | admission PASS/PASS_WITH_RISK, exit-gate start, real computation |
| `EXPIRED_BLOCK_ADMISSION` | await FU-006 independent evidence | any first real-evidence admission PASS/PASS_WITH_RISK |
| `EXPIRED_BLOCK_EXIT` | await FU-006 independent evidence | Stage 3 exit-gate start |
| `VERIFIED_REQUIRED_GATE` | later gate may assess FU-006 evidence | does not itself authorize data, runtime or trading |

For either E1 or E2, `FU-006 evidence absent => requested maturity action blocked`. There is no calendar extension and no inheritance of the CR-170 verifier waiver.

## 11. Claim ceiling and pure-analysis precommitment

The following must remain false throughout CR-171 and after a successful CP8: `stage3_started`, `stage3_entry_ready`, `real_data_read_authorized`, `real_computation_authorized`, `aggregate_orchestration_implemented`, `cr155_promoted`, `runtime_ready`, `simulation_ready`, `trading_ready`.

Revalidation in this CR is pure analysis/manual reading of already-authorized review materials and introduces **no new executable tool**. If someone proposes a script, executable revalidation tool, producer invocation or any data action, the route must be reassessed before CP4–CP6 applicability can change. Until then CP4, CP5 and CP6 remain N/A.

## 12. Integration contract

| Consumer | Invocation / timing | Input contract | Output contract | Failure / downgrade |
|---|---|---|---|---|
| future activation CR | only after its own CP0/CP2 authorization | CP3-approved route, five-field candidate, C1–C4 verdicts, waiver state | mapping/binding/authorization decisions, never inferred from this HLD | any pending/insufficient/incompatible field blocks activation scope |
| future revalidation report | only under a separately authorized audit boundary | legacy evidence references and CP3 verdict schema | one of 3/3 current-entry verdicts plus annotations | repair need routes new CR |
| admission/exit gate | later maturity stage | FU-006 state and evidence references | allow/block decision | E1/E2 without FU-006 blocks action |
| risk consumer | any new CR joining CR-170 risk | `R-CR170-RUNNER-GAP`, with alias policy resolution | canonical risk ID | do not consume historical alias as a separate risk |

## 13. Scenario simulations

| Scenario | Steps | Expected design outcome | Result |
|---|---|---|---|
| permitted design-only flow | choose C1–C4 route; assess static contracts; leave five fields pending | no lake/runtime action; HLD can reach CP3 | PASS |
| insufficient historical evidence | auditor cannot prove current PIT/lineage from declared facts | `insufficient_for_current_entry`; no repair/rerun; claims stay false | PASS |
| incompatible historical evidence | declared contract conflicts with current required schema/PIT/lineage | `incompatible_rework_required`; separate CR required | PASS |
| waiver expiry | attempt admission PASS or exit-gate start while FU-006 absent | E1/E2 mechanically block requested action | PASS |
| denied read/computation boundary | infer credential/read/computation authority from CP2/CP3 documents | denied; all corresponding authorization/claim flags remain false | PASS |

## 14. NFR, risks and failure paths

| Area | Measurable control | Failure behavior |
|---|---|---|
| authorization | allow fields 5/5 and deny classes 6/6 present; no wildcard | missing/ambiguous field means no future read contract |
| feasibility | C1–C4 4/4 verdicts, each static evidence-backed | no `compatible` claim without release/schema/PIT/lineage binding |
| revalidation | 3/3 legal current-entry verdicts, repair actions 0 | invalid verdict/action blocks CP7/CP8 wording |
| waiver | E1/E2 2/2 and no extension branch | event triggers deny maturity action |
| claims | forbidden readiness flags 9/9 false | any positive claim is a high-risk overclaim and stops route |

Key risks: `R-CR170-RUNNER-GAP` remains controlled out of scope; `R-CR171-HISTORICAL-EVIDENCE-OVERCLAIM` is controlled by the 3-verdict ceiling; `R-CR171-LAKE-READ-SCOPE-ESCAPE` by 5/5+6/6 contract; `R-CR171-PIT-LINEAGE-INFEASIBILITY` by the C1–C4 matrix and activation-CR split. CR-032/010/018/031/033/168 are disclosed tracking debt only and are not remediated here.

## 15. Use-case and requirement traceability

| Product source | HLD consumption |
|---|---|
| UC-58-CR171 | §1–§2, §5–§13: decision-only journey and no-execution boundary |
| REQ-CR171-001 | §5–§6 and §8: two-CR route and producer boundary |
| REQ-CR171-002 | §10: 2/2 event-bounded waiver guards |
| REQ-CR171-003 | §7 and §14: 5/5 candidate + 6/6 deny-default |
| REQ-CR171-004 | §9 and §13: legacy annotation, 3/3 verdicts, no repair |
| REQ-CR171-005 | §11 and §14: claim ceiling, risk alias and debt isolation |

## 16. ADR candidates and CP3 decisions

The companion ADR records four candidate decisions: two-CR route separation, five-field pending-human candidate semantics, revalidation/legacy report ceiling, and event-bounded waiver enforcement. CP3 asks the human to approve these as a coherent design; approval does not grant any runtime/data authority.

## 17. Follow-up boundary

| Follow-up | Trigger | Scope | Not authorized by this HLD |
|---|---|---|---|
| Real-Evidence Activation CR | human freezes sufficient five-field input contract and authorizes its CP0 | real-data mapping, binding, computation authorization, run identity | aggregate, CR155 promotion, runtime/trading |
| FU-006 lane | before E1 or E2 | independent verifier evidence | waiver extension or inferred PASS |
| FU-007b | Stage 3 exit/mature SAP or CR155 promotion | aggregate orchestration and independent CR155 regression | early entry activation |
| C4 rework CR | C4 matrix `incompatible` persists | implementation/contract reconstruction under new authorization | bypass via aggregation |

## 18. CP7 verification intent

CP7 verifies design artefacts only: 5/5 and 6/6 contract completeness; 4/4 C1–C4 verdict consistency; 3/3 revalidation verdict ceiling and no-repair condition; E1/E2 guard logic; legacy marker; risk alias usage; all no-authorization flags; **§21 每一项 OPEN/RESOLVED 状态、owner、route 和日期是否自洽**；以及 ADR-001..004 的逐项重开条件是否存在且未被误触发。It will not test data reads, computations, producers, runtime, writes or trading.

## 19. CP3 approval effect and rollback

If approved, this HLD permits the route to continue to CP7 design verification with CP4–CP6 N/A. It still does not permit real-lake reading or Stage 3 start. If modified/rejected, return to CP3 design. The detailed per-ADR reopen conditions in the ADR are authoritative: a changed 5-field candidate or revalidation/waiver rule cannot be silently absorbed by §19. If future static facts invalidate the selected route, a new CR or CP2 re-open is required; simply editing five-field values or switching to current runner is not an allowed rollback.

## 20. Gotchas

- A named historical release path is not a current read authorization.
- `revalidated_for_current_entry` is evidence classification, not a readiness or runtime flag.
- An output directory candidate does not authorize writing it; output authorization is separate.
- Waiver “active” applies only to design; it never creates an admission PASS exception.
- C4 `incompatible` must not be hidden by C1–C3 results or converted into aggregate success.
- A successful CR-171 CP8 confirms decision/verification closure only, never Stage 3 entry readiness.

## 21. Open Items Status Register

This register is the explicit state closure for residual facts. `OPEN` means the item is intentionally unresolved and has a named owner, route and re-review condition; it does **not** permit a fallback, an inferred authorization or a positive Stage 3 claim. `RESOLVED` records the date and evidence that closed the item. All rows are reviewed at CP7 and CP8; none creates data, computation or runtime authority.

| ID | Item / why it remains open | Status | Opened / last reviewed | Owner | Closure evidence / route | Effect while open |
|---|---|---|---|---|---|---|
| `OI-CR171-001` | Exact 5/5 `data_release / datasets / date_range / read_identity / output_directory` values cannot be inferred without creating a false authorization. | **OPEN** | 2026-07-15 / 2026-07-15 | authorized data owner + future Activation CR approver | a separately authorized gate freezes 5/5 finite values and their acceptance criteria; if writing is needed, a distinct write authority is recorded | Activation scope blocked; no read/write/computation. |
| `OI-CR171-002` | C1/C2/C3 are statically `insufficient`: no named release/schema/PIT/lineage binding has been authorized or proven. | **OPEN** | 2026-07-15 / 2026-07-15 | future Real-Evidence Activation CR | activation CR supplies explicit mapping, binding and separately reviewed computation authorization; any failed binding stays insufficient | No producer execution or current-entry claim. |
| `OI-CR171-003` | C4 is statically `incompatible`: no C4 real producer calculation/aggregate path is presently available. | **OPEN** | 2026-07-15 / 2026-07-15 | future C4-specific rework / authorization CR | a separate CR establishes a compatible C4 contract and required authorization; FU-007b remains responsible for aggregate/CR155 regression | C4 cannot be activated or hidden by C1–C3/aggregate. |
| `OI-CR171-004` | FU-006 independent verifier evidence is not yet complete. | **OPEN** | 2026-07-15 / 2026-07-15 | FU-006 independent verifier lane | independent verifier evidence accepted before E1 or E2; no calendar extension and no CR-170 waiver inheritance | E1 blocks first real-evidence admission PASS/PASS_WITH_RISK; E2 blocks Stage 3 exit-gate start. |
| `OI-CR171-005` | Historical evidence has not been independently classified under the current-entry contract. | **OPEN** | 2026-07-15 / 2026-07-15 | separately authorized revalidation/audit lane | report gives exactly one allowed current-entry verdict plus legacy annotation; repair need becomes a new CR | Stage 3/read/computation readiness remains false. |
| `OI-CR171-006` | This review found no explicit state register and incomplete per-ADR reopen wording. | **RESOLVED** | 2026-07-15 / **2026-07-15** | host-orchestrator | this §21 table, ADR-002..004 reopen clauses, CP3 result items 11–12 and CP3 approval trace | No scope or authorization change; the CP3 gate may proceed. |
