---
handoff_id: "HO-CR030-CP3-STORY-PLANNING-20260716"
type: "phase-delegation"
canonical_role: "meta-se"
status: "spawn-requested"
created_at: "2026-07-16T00:00:00+00:00"
source_phase: "solution-design"
target_phase: "story-planning"
source_cp: "CP3"
target_cp: "CP4"
artifacts:
  input:
    - "docs/design/HLD.md"
    - "docs/design/BLUEPRINT.md"
    - "docs/design/DOMAIN-MAP.md"
    - "docs/design/DEPENDENCY-MAP.md"
    - "docs/design/ARCHITECTURE-DECISION.md"
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
  output:
    - "docs/design/FEATURE-DESIGN-MATRIX.md"
    - "docs/features/*/DESIGN.md (if required)"
    - "docs/features/*/TEST-PLAN.md (if required)"
    - "docs/features/*/TASKS.md (if required)"
    - "process/DEVELOPMENT-PLAN.yaml"
  context:
    - "process/context/CP3-DESIGN-CONTEXT.yaml"
dispatch:
  mode: "subagent"
  canonical_role: "meta-se"
  codex_agent_name: "se-chu"
  reasoning_profile: "high"
  dispatch_trigger: "CP3 approved → story-planning"
  tool_name: "Agent"
---

# Handoff: Story Planning for CR-030

## Phase Summary

CP3 has been approved. The user confirmed REV-03's four-boundary model:

```
itr-ticket-ingestion Skill → SQLite Data Owner → reverse-analysis Skill → improvement-tracker Skill → human reviewer
```

With S1 (new analysis) / S2 (update analysis) dual data links, six analysis dimensions, YoY/MoM comparison, and 7 trust governance contracts.

## Task

Execute story-planning phase:

1. **FEATURE-DESIGN-MATRIX.md** (`docs/design/FEATURE-DESIGN-MATRIX.md`)
   - Map each HLD module to Feature/Epic
   - For each Story: assign `feature_design_refs`, `lld_policy.required_level` (full-lld / technical-note / waived)
   - Mark which Features require `docs/features/<feature>/DESIGN.md` / TEST-PLAN.md / TASKS.md

2. **Feature Design** (only for required Features)
   - `docs/features/<feature>/DESIGN.md`
   - `docs/features/<feature>/TEST-PLAN.md`
   - `docs/features/<feature>/TASKS.md`

3. **Story Decomposition**
   - Use existing STORY-MAP.md v1.3 as input (ST-RA-01..06, ST-NRA-01..03)
   - Assign Wave, dependencies, file ownership
   - Generate full DEVELOPMENT-PLAN.yaml

4. **DEVELOPMENT-PLAN.yaml** (`process/DEVELOPMENT-PLAN.yaml`)
   - Machine truth for Story management
   - Story IDs, Wave, status, tasks, dependencies, file ownership

## Key Architecture (from CP3-approved HLD REV-03)

- **4 boundaries**: itr-ticket-ingestion / SQLite / reverse-analysis / improvement-tracker
- **S1/S2 dual links**: new analysis + update analysis with change detection
- **6 analysis dimensions**: root cause, product quality, escape, missed-test, improvement, YoY/MoM
- **7 trust contracts**: IngestionQualityReport, AnalysisRunManifest, MetricDefinition, root cause 4-layer, escape control evidence, MeasureBaseline, sensitive field strategy
- **Data owner**: SQLite is sole canonical store; raw snapshots not in Git/process
- **Security**: deny-by-default, no credentials, no external write, no auto execution

## Existing Stories (from STORY-MAP.md v1.3)

| Story ID | Summary | Priority |
|---|---|---|
| ST-RA-01 | Launch decision + trusted input | P0 |
| ST-RA-02 | Root cause/escape explanation | P0 |
| ST-RA-03 | Convert lessons to improvement input | P0 |
| ST-RA-04 | Effectiveness-based closure | P1 |
| ST-RA-05 | Controlled ITR ingestion + S1 analysis | P0 |
| ST-RA-06 | S2 update analysis + change detection | P0 |
| ST-NRA-01 | Insufficient evidence → gaps | — |
| ST-NRA-02 | Permission boundary enforcement | — |
| ST-NRA-03 | ITR ingestion failure protection | — |

## Output Requirements

All output files must include proper frontmatter with `status: draft`, `source_cr: "CR-030"`, `version: "1.0"`.

DEVELOPMENT-PLAN.yaml must use the standard schema: stories with id/slug/wave/dependencies/file_ownership/tasks/status.

When done, the meta-se agent should report back with:
- List of all generated files
- Any open architecture questions discovered during decomposition
- Recommended Wave ordering and parallelism
- CP4 pre-check readiness assessment
