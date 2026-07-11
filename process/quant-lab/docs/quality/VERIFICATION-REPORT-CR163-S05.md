---
status: complete
version: "1.2"
story_id: "CR163-S05-integrity-recovery-permission-regression"
validation_mode: "mixed"
verification_result: "PASS_WITH_RISK"
reverification_round: 2
created_by: "meta-qa-critical"
created_at: "2026-07-11T17:28:00+08:00"
updated_at: "2026-07-11T17:56:00+08:00"
---

# CR163-S05 Final CP7 Reverification Report

## 1. Decision

`PASS_WITH_RISK`. QA-CR163-S05-001..003 are independently closed in the normal final round. The authorization installation manifest is exact 13/13; every category has a concrete/path-aware target and predicate; all wrappers are active during the synthetic S01→S05 path and the observed counters are 13/13 zero. Each marker is invoked through its actual monkeypatched Path/os/socket/subprocess/urllib API, increments the intended counter, blocks before the substituted original spy, and performs no real I/O. Independent non-marker probes pass through 13/13 boundaries to safe originals while counters remain zero.

The focused S05 suite and full seven-suite subset pass. Structured trace and authorization schema regressions remain closed. No new finding, waiver, design clarification or workflow-health escalation is required.

The risk qualifier retains accepted limitations: future producer retry-loop changes require an end-to-end same-session ordinal fixture; upstream effective/C1 and S02 recovery/capacity claim ceilings remain unchanged.

## 2. Verification inventory

| Object / contract | Method | Result |
|---|---|---|
| authorization manifest | source review and exact-set assertions | PASS — 13/13 target+predicate |
| guarded synthetic path | focused native test | PASS — 13/13 zero under simultaneous wrappers |
| marker boundary hits | 13 actual monkeypatched API cases | PASS — counter++ / original spy 0 / no real I/O |
| non-marker pass-through | independent safe-spy matrix | PASS — 13/13 originals called, counters zero |
| exact evidence schema | five-class negative matrix | PASS — missing/unknown/bool/negative/nonzero blocked |
| structured scenario trace | AST resolver and bad-ref test | PASS — 12/12 |
| integrity/recovery/CR155 | focused and full suites | PASS_WITH_RISK |
| CP6 return/evidence | machine validation | OK / OK |

## 3. Traceability and quantitative exits

| Requirement / metric | Result |
|---|---|
| requirements / scenarios | 8/8; 12/12 structured and resolvable |
| CPI mappings / chains | 4/4; 2/2 |
| deterministic seals | 10 seals → 1 hash |
| negative classes | 5/5 blocked |
| boundary installation manifest | 13/13 |
| guarded-path forbidden counters | 13/13 zero |
| marker boundary hits | 13/13 blocked before original; original spy calls 0 |
| non-marker pass-through | 13/13; safe original calls 13; forbidden counts 0 |
| authorization schema negative classes | 5/5 |
| actual/synthetic CR155 | 2/2 blocked; paper false; reconstruction/backfill 0 |
| effective/C1/runtime-ready positive claims | 0 / 0 / 0 |

## 4. Execution evidence

| Check | Result |
|---|---|
| authorization suite | 19 passed in 0.60s |
| focused S05 suites | 33 passed in 0.93s |
| schema/trace selectors | 5 passed + 2 passed |
| independent non-marker safe-spy matrix | 13/13 pass-through, zero counters |
| full seven-suite subset | 262 passed in 29.59s |
| CP6 return/evidence | OK / OK |
| `git diff --check` | PASS |

## 5. Finding history

| ID | Initial | Round 1 | Round 2 final |
|---|---|---|---|
| QA-CR163-S05-001 | OPEN BLOCKER: no observed sentinels | OPEN BLOCKER: only 3/13 connected | RESOLVED: 13/13 installed, marker and pass-through proven |
| QA-CR163-S05-002 | OPEN HIGH: missing schema failed open | RESOLVED | RESOLVED, regression retained |
| QA-CR163-S05-003 | OPEN HIGH: trace not locatable | RESOLVED | RESOLVED, regression retained |

## 6. Manual review, risks and route

- Requirement consistency, negative coverage, error actionability, authorization isolation and CR155 claim ceiling: PASS.
- Accepted `R-CR163-S05-001`: any future producer retry-loop change must add an end-to-end same-session/same-trial ordinal fixture.
- Carry upstream effective/C1 non-computability and S02 recovery/capacity observation-only limits to CP8.
- Workflow-health escalation: not required; final round found no new defect.
- Route: `verified-with-risk`.
