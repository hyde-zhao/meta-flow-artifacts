# CP3 CR155 HLD Discussion Log

## Context

- CR: `CR-155`
- Stage: CP3 solution-design
- Source CP2: `process/checkpoints/CP2-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-SCOPE.md`
- Purpose: record Architecture Gray Areas and advisor table inputs for the HLD draft.

## Architecture Gray Areas

| ID | Question | Selected Direction | Reason |
|---|---|---|---|
| AGQ-CR155-001 | Should CR155 produce a standalone strategy artifact or only a report/admission package? | Standalone artifact consumed by admission package. | Stable audit identity is required by CP2 scope. |
| AGQ-CR155-002 | How should CP2-approved governed lake readonly be represented? | Isolated readonly validation pipeline. | Prevents readonly from becoming write/runtime permission. |
| AGQ-CR155-003 | How should the factor combination be chosen? | Simple deterministic factor set, no optimizer. | Baseline is reference, not optimal strategy discovery. |
| AGQ-CR155-004 | How should CR151 and CR154 gates compose? | Separate statistical, reliability and package decisions. | Keeps failure reasons auditable. |
| AGQ-CR155-005 | How should rerun consistency be enforced? | Mandatory two-rerun comparison. | CP2 required core metrics consistency. |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Artifact contract plus readonly validation pipeline | Concrete audit artifact; uses CP2-approved readonly boundary; supports rerun evidence | Requires CP5 story split | artifact, data, validation, security | Recommended | Switch to fixture-only if readonly provenance cannot be proven. |
| Report-pack-only baseline | Faster | Weak auditability and implicit strategy definition | reports, admission | Not recommended | Useful only outside CR155 formal artifact scope. |
| Admission-package-only extension | Fewer objects | Mixes strategy definition and decision | admission, ownership | Not recommended | Package should consume artifact summary. |
| External framework recipe | Familiar workflow | Violates dependency/runtime boundary | dependency, runtime, licensing | Rejected | Requires separate Spike. |

## Outcome

The HLD draft uses a standalone artifact contract plus isolated readonly validation pipeline, with CR151/CR154 gate refs preserved separately and rerun consistency as a first-class evidence object.
