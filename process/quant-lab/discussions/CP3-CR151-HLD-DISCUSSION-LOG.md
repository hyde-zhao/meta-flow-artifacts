---
discussion_id: "CP3-CR151-HLD-DISCUSSION"
cr_id: "CR-151"
phase: "solution-design"
status: "ready-for-cp3-review"
owner: "host-orchestrator"
created_at: "2026-07-01T22:53:50+08:00"
authorization_boundary: "design-only; no source implementation, no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote"
---

# CP3 CR151 HLD Discussion Log

## Purpose

This log records the Architecture Gray Areas used before drafting the CR151 HLD. It is an audit and recovery artifact, not the downstream implementation authority. The implementation authority after CP3 approval will be the HLD, ADR, Feature Design Matrix, Story cards, and CP5 design evidence.

## Inputs Scanned

| Source | Path | Result |
|---|---|---|
| CP2 scope baseline | `process/checkpoints/CP2-CR151-MULTIFACTOR-STRATEGY-E2E-SCOPE.md` | CR151 scope, Wave A first, and no-runtime boundary approved. |
| Roadmap | `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` | CR151 is the next formal work after CR150; data-lake production closure deferred. |
| Strategy E2E review | `docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` | C1/C2 and MF-GAP-1/2/4 remain valid; ResearchDatasetSpec and BacktestRunSpec absence claim is stale. |
| Remediation plan | `docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md` | CR151 Wave A is admission gate hard prerequisite set. |
| Existing contracts | `engine/research_production_contracts.py`, `engine/backtest_production_contracts.py`, `engine/strategy_admission_package.py`, `engine/mature_multifactor_research.py` | Existing contract sources should be extended and linked, not replaced. |

## Architecture Gray Areas

| AGA ID | Key Question | Why It Changes Architecture | Impact Surface | Recommendation | Status |
|---|---|---|---|---|---|
| AGA-CR151-01 | Should statistical admission be a dedicated contract module or embedded directly into `mature_multifactor_research.py`? | This decides ownership, reuse by CR152/CR153, and whether tests can validate admission semantics without touching the runtime runner. | module boundary, dependency map, tests, future reuse | Use a dedicated metadata-only contract module consumed by mature multifactor linkage. | selected |
| AGA-CR151-02 | Should CR151 implement Wave A only or include Wave B reports now? | Wave B adds capacity, benchmark-relative reporting, extended factor diagnostics, regime-aware validation, correlation clustering / de-duplication and PIT universe audit, increasing CP5 and implementation risk. | scope, Story count, release risk | Keep Wave A only: multiple testing, minimum robust statistics, walk-forward/OOS, PBO/DSR, statistical gate. Track Wave B as follow-up / CR154 input. | selected |
| AGA-CR151-03 | How strict should the gate be when fixture data is incomplete? | The gate must avoid false runtime claims while still allowing deterministic unit tests and static evidence. | status model, failure path, CP7/CP8 wording | Use `PASS / FAIL / NEEDS_REVIEW / BLOCKED`; missing mandatory evidence is `BLOCKED`, threshold uncertainty is `NEEDS_REVIEW`. | selected |
| AGA-CR151-04 | Should validation remain static-only? | Real lake, provider, NAS, runtime and external framework access would change security and evidence obligations. | authorization, verification, release readiness | Keep static/fixture-only validation. Any real data or runtime validation must be a separate gate/CR. | selected |

## Advisor Tables

### AGA-CR151-01 Module Boundary

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Dedicated `engine.strategy_admission_statistical_gate` contract module | Reusable, testable in isolation, avoids enlarging mature runner, supports future ML/event reuse by adapter. | Adds one new module and explicit linkage object. | module, tests, HLD, Story split | Recommended. | Switch only if CP5 shows the object is purely internal and has no cross-strategy value. |
| Embed in `engine.mature_multifactor_research.py` | Minimal file count; simple local wiring. | Harder to reuse, increases already broad runner surface, makes static fixture tests less isolated. | module, maintainability | Not recommended. | Accept only for emergency tiny patch, not full E2E framework completion. |
| Extend `engine.strategy_admission_package.py` directly | Keeps admission in one place. | Existing package also covers QMT/no-real-op semantics; mixing statistical gate internals risks conflating research admission with runtime readiness. | safety, claims, tests | Conditional fallback. | Use only if final design needs a small reference field, not statistical calculation contracts. |

### AGA-CR151-02 Wave Boundary

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Wave A only | Smallest complete admission gate; aligns CP2; easier CP5/CP6/CP7. | Leaves capacity, benchmark-relative and PIT universe audit for later. | scope, Story count, tests | Recommended. | Switch if user requires production-grade strategy turnover before CR152/153. |
| Wave A + Wave B | More production-like output. | Too broad; duplicates CR154 cross-strategy reliability scope. | scope, test burden, release risk | Not recommended. | Use after CR151 closes and CR154 starts. |
| Documentation-only | Lowest code risk. | Does not complete the E2E framework. | value, verification | Not recommended. | Use only if source implementation becomes blocked by a newly discovered contract conflict. |

### AGA-CR151-03 Gate Status Model

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| `PASS / FAIL / NEEDS_REVIEW / BLOCKED` | Distinguishes hard missing evidence from statistical gray areas; supports fail-closed. | Slightly more states than binary gate. | tests, release wording | Recommended. | Switch only if downstream admission package already has an equivalent enum. |
| Binary pass/fail | Simple. | Forces missing evidence and ambiguous threshold into same bucket. | debugging, audit | Not recommended. | N/A. |
| Always blocked until real data | Very safe. | Prevents local fixture completion and contradicts CR151 scope. | value, CP7 | Not recommended. | Use only if static statistical gate cannot be proven with fixtures. |

### AGA-CR151-04 Validation Boundary

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Static/fixture-only | Matches CP2, no credentials, no lake/NAS/provider/runtime risk. | Cannot claim real strategy performance. | security, CP7/CP8 | Recommended. | Switch only via separate runtime authorization gate. |
| Read real lake snapshots | Stronger empirical evidence. | Requires runtime authorization and data-lake/NAS readiness decisions. | security, data lake | Not authorized. | Future CR only. |
| Use external framework / provider | Potentially richer validation. | Violates CR151 boundary and adds dependency risk. | security, dependency | Not authorized. | Future Spike/CR only. |

## Adopted Inputs

| Input | HLD Effect |
|---|---|
| CP2 Wave A first approval | HLD limits implementation scope to statistical admission core. |
| E2E review C1/C2/MF-GAP-1/2/4 | HLD makes multiple testing, minimum robust statistics, walk-forward/OOS and overfit risk first-class reports; MF-GAP-2 extended diagnostics and MF-GAP-4 regime split are deferred rather than implied covered. |
| Current source fact correction | HLD reuses `ResearchDatasetSpec` and `BacktestRunSpec` rather than rebuilding them. |
| User route A | HLD defers data-lake production closure and runtime validation. |
| E2E review MF-GAP-7 | HLD records factor correlation clustering / redundancy de-duplication as deferred Wave B / CR154 scope. |

## Deferred Architecture Ideas

| ID | Idea | Deferred Reason | Trigger |
|---|---|---|---|
| DAI-CR151-01 | Capacity / impact report | Cross-strategy production reliability concern, better suited to CR154. | User requires production-grade turnover or capital sizing. |
| DAI-CR151-02 | IR / tracking error / Active Share | Benchmark-relative evaluation, not admission hard prerequisite for CR151 Wave A. | CR154 or CP3 scope expansion request. |
| DAI-CR151-03 | PIT universe audit | Important but intersects data-lake facts and FU-CR139/FU-CR149 governance. | Real lake validation or production turnover path resumes. |
| DAI-CR151-04 | External Qlib / mlfinlab / pyfolio integration | Adds external dependency and runtime/installation surface. | Separate Spike with explicit dependency authorization. |
| DAI-CR151-05 | Extended factor evaluation diagnostics: IC decay by lag, half-life, turnover, liquidity/capacity view, orthogonalization against known risk factors, monotonicity and quantile spread | Full factor quality reporting, not the minimum Wave A admission gate. | CR154, CR151 follow-up, or user explicitly expands CP3 scope. |
| DAI-CR151-06 | Regime-aware validation / regime-stratified backtest | Requires regime definitions and may intersect real-data / production reliability work. | User requires policy-cycle or market-state-specific admission. |
| DAI-CR151-07 | Factor correlation clustering and redundancy de-duplication | Portfolio construction / factor set governance item, not mandatory to prove first statistical gate semantics. | CR154, post-CR151 Story, or explicit CP3 scope expansion. |
