---
cr_id: "CR-171"
checkpoint: "CP3"
phase: "solution-design"
status: "ready-for-human-review"
created_at: "2026-07-15T21:50:00+08:00"
---

# CP3 CR-171 HLD Discussion Log

## Purpose and method

This is the Architecture Gray Areas table-first design record for the CP3 package. It consumes the CP2-approved values, UC-58-CR171 and REQ-CR171-001..005. No additional reviewer lane is claimed; no lake, provider, credential, runtime or implementation fact was read or produced.

## Architecture Gray Areas

| ID | Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|---|
| AGA-CR171-001 | C1–C4 two-CR activation separation | canonical-first producer route; isolates real computation | later activation CR required | architecture / authorization / audit | select | reopen CP2 only if all producer paths become statically incompatible |
| AGA-CR171-002 | current runner single-CR route | fewer entry CRs | runner-gap remains, full execution boundary moves into CR-171 | runner / canonical / risk | reject | only on explicit CP2 reopen and accepted runner gap |
| AGA-CR171-003 | five-field candidate values left pending human authorization | avoids inferred release/path permission | activation cannot start until human freezes values | data governance / authorization | select | human may freeze exact 5/5 at CP3 or a later authorization gate |
| AGA-CR171-004 | infer values from legacy run | fast-looking | converts historical documentation into read permission | security / audit | reject | never; needs explicit user authorization |
| AGA-CR171-005 | revalidation classification only | preserves historical truth and narrow CR boundary | repair needs a later CR | evidence / scope | select | new rework CR on `incompatible_rework_required` |
| AGA-CR171-006 | event-bounded waiver | does not block design and has 2 mechanical expiry guards | FU-006 still mandatory before maturity | verification / gate | select | choose FU-006 first only via CP2 reopen |

## Discussion outcome

- The HLD adopts `c1_c4_real_producer`, `event_bounded_waiver`, and a `scoped_research_data_lake_read_only` *candidate*, exactly as CP2 approved.
- Four CP3 decisions remain for the human: two-CR split; candidate semantics; 3-verdict historical-report contract; 2-event waiver enforcement.
- All five read fields intentionally remain `pending_human_authorization`. This is a successful, truthful design outcome—not a blocker for CP3 review—and prevents any accidental real read authorization.
- Static C1–C4 feasibility verdicts are C1/C2/C3 `insufficient`, C4 `incompatible`; they neither activate any producer nor permit a current-runner fallback.

## Deferred ideas and boundary

| Idea | Status | Reason / next owner |
|---|---|---|
| Freeze exact release/dataset/date/identity/output values | pending human authorization | CP3 decision or later activation-CR gate; no lake inspection allowed to infer them |
| Real producer mapping and computation | deferred | Real-Evidence Activation CR only |
| FU-006 independent verifier | deferred but mandatory before E1/E2 action | FU-006 lane |
| C1–C4 aggregation and CR155 promotion | deferred | FU-007b / later maturity gate |
| Repair historical defects | out of scope | separate follow-up CR only |

## Inputs and traceability

- CP2 result: `process/checks/CP2-CR171-STAGE3-LAUNCH-SCOPE.result.json`
- CP2 manual record: `process/checkpoints/CP2-CR171-STAGE3-LAUNCH-SCOPE.md`
- Candidate memo: `process/plans/STAGE3-LAUNCH-DECISION-MEMO-2026-07-15.yaml`
- Product traceability: UC-58-CR171, REQ-CR171-001..005, QAC-CR171-01..06.

## Review remediation and impact record — 2026-07-15

The CP3 review identified two documentation-governance omissions: (1) the HLD lacked an explicit OPEN/RESOLVED state register for residual items; (2) ADR-002..004 relied on the general HLD rollback paragraph instead of giving per-decision reopen conditions. Both are accepted as **same-CR CP3 design-document remediation**, not a new requirement or authorization change.

| Impact dimension | Assessment | Document decision |
|---|---|---|
| Requirement / product | no change to UC-58, REQ-CR171-001..005, scenarios or CP2 decisions | unchanged; no CP1/CP2 replay |
| Architecture / contract | adds explicit residual-item status and per-ADR reopen semantics only | update HLD §18/§19/§21 and ADR-002..004; v0.2 revision records retain v0.1 baseline |
| Security / authorization | no allow-list, deny-default, release value, credential, read, write or computation rule is relaxed | unchanged; all claim ceilings remain false |
| Verification / workflow | CP3 auto precheck gains checks for status closure and 4/4 ADR reopen conditions; CP7 verifies them | update CP3 result/checklist; CP4–CP6 remain N/A |
| Delivery / rollback | no Story, code, data, runtime, external write or release impact | remain in CR-171 CP3; rollback is CP3 design revision only |

Review disposition: `OI-CR171-006=RESOLVED` on 2026-07-15 after the documented artifacts and CP3 checks are updated. Existing residual entry items remain explicitly `OPEN` in HLD §21; their owners and closure routes are unchanged.
