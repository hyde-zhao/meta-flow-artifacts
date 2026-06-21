---
check_id: "CR108-LAYERED-DOCS-DRIFT-MATRIX"
cr_id: "CR-108"
type: "static_inventory"
status: "w5_complete"
owner: "host-orchestrator"
created_at: "2026-06-21T22:05:41+08:00"
context_ref: "process/context/CP2-CR108-LAYERED-DOCS-SCOPE-CONTEXT.yaml"
---

# CR108 Layered Docs Drift Matrix

## Scope

This matrix is the first CR108 inventory pass. It does not modify business code, does not read credentials, and does not authorize runtime, SaaS, production writes, trading, CR-033, or CR102-CR104 closure.

## Scan Inputs

| Surface | Command | Result |
|---|---|---|
| docs file list | `find -L docs -maxdepth 2 -type f` | Followed symlinked `docs/design`, `docs/features`, and `docs/quality`; active docs surface is non-empty. |
| current docs drift | `rg -n "meta-po|local_backtest|local-backtest|console script|runtime_authorization gate|uv run --env-file|Host Orchestrator" README.md docs/USER-MANUAL.md docs/design docs/features docs/quality` | Found current-facing drift plus historical audit references. |
| process historical drift | `rg -n "meta-po|local_backtest|local-backtest|runtime_authorization gate" process/HLD.md process/REQUIREMENTS.md process/STORY-BACKLOG.md process/checkpoints process/context process/release` | Found many historical process references; default handling is inventory-only / legacy audit keep. |

## Classification Rules

| Class | Meaning | Default Action |
|---|---|---|
| `must_fix_current_truth` | Current-facing docs contradict package identity, CLI, Host Orchestrator, or current authorization boundary. | Update in CR108. |
| `current_truth_boundary_note` | Current docs mention `.env`, provider commands, runtime gates, or legacy names in a way that may be valid but needs explicit no-agent-access boundary. | Add or tighten wording only if current-facing. |
| `legacy_audit_keep` | CR-scoped or historical evidence records old role names, old paths, runtime gates, or previous package names as historical fact. | Do not rewrite; preserve audit meaning. |
| `inventory_only` | Process historical HLD/requirements/checkpoints/context/release files may be read accidentally but are not current truth. | Record as legacy/read-only; do not bulk rewrite. |
| `follow_up_candidate` | Potentially valid drift but outside CR108 docs/current-truth scope, or needs business-code/test/tooling analysis. | Route to CR110 or later follow-up. |

## Matrix

| ID | Surface | Evidence | Classification | Action |
|---|---|---|---|---|
| CR108-DM-001 | `README.md` | Line 49 says there is no console script entry, but CR105 added `quant_lab` CLI and package identity now exposes `quant_lab`. | `must_fix_current_truth` | Update README current-facing command/entrypoint section. |
| CR108-DM-002 | `README.md` | Line 68 says `meta-po` orchestrates CR/checkpoints; current AGENTS.md defines Host Orchestrator main process. | `must_fix_current_truth` | Replace current-facing orchestration wording with Host Orchestrator; preserve historical CR mentions elsewhere. |
| CR108-DM-003 | `docs/USER-MANUAL.md` | Line 21 says `meta-po` orchestrates CR/checkpoints. | `must_fix_current_truth` | Replace current-facing orchestration wording with Host Orchestrator. |
| CR108-DM-004 | `docs/USER-MANUAL.md` | Line 531 points runtime requests back to `meta-po` authorization path. | `must_fix_current_truth` | Replace with Host Orchestrator CR/CP/human-gate authorization path and keep runtime denied by default. |
| CR108-DM-005 | `README.md`, `docs/USER-MANUAL.md` | Current examples include `uv run --env-file .env` commands. | `current_truth_boundary_note` | Do not remove commands mechanically; add wording that agents must not read `.env` and these commands require explicit user/runtime authorization. |
| CR108-DM-006 | `docs/design/HLD-CR107-QUANT-LAB-REMEDIATION-DESIGN.md` | Intentionally records CR108 scope and old drift examples. | `legacy_audit_keep` | No change; this is the approved design record. |
| CR108-DM-007 | `docs/design/HLD.md`, `ARCHITECTURE-DECISION.md`, `FEATURE-DESIGN-MATRIX.md`, `BLUEPRINT.md`, `DOMAIN-MAP.md`, `DEPENDENCY-MAP.md` | Revision history and some owner fields still mention `meta-po`; many sections already state `quant-lab` canonical and `local_backtest` legacy alias. | `current_truth_boundary_note` | Review current owner/consumer rows only. Do not rewrite revision history. |
| CR108-DM-008 | `docs/design/FEATURE-DESIGN-MATRIX.md` | Current matrix row references `meta-po / meta-se` ownership. | `must_fix_current_truth` | Completed in W2: current owner language updated to Host Orchestrator / meta-se; revision history preserved. |
| CR108-DM-009 | `docs/design/PACKAGE-IDENTITY.yaml` | Contains `local-backtest` and `local_backtest` aliases. | `legacy_audit_keep` | No change; aliases are expected compatibility policy. |
| CR108-DM-010 | `docs/design/HLD-CR045-*`, `ARCHITECTURE-DECISION-CR045.md`, CR046/CR051/CR053 CR-scoped design docs | Historical role names, runtime authorization paths, and `local_backtest` references are CR-scoped facts. | `legacy_audit_keep` | Do not bulk rewrite; only add current index notes if these docs are consumed as current truth. |
| CR108-DM-011 | `docs/features/*/DESIGN.md`, `TEST-PLAN.md`, `TASKS.md` | Many feature docs have revision history authored by `meta-po`; CR045 feature docs route future runtime gates to `meta-po`. | `legacy_audit_keep` | Preserve CR-scoped historical records; review current feature indexes for owner wording separately. |
| CR108-DM-012 | `docs/features/cr061-package-import-layout/DESIGN.md` | The design says restore project metadata to `local-backtest`, while later implementation evidence says `quant-lab` was added. | `must_fix_current_truth` | Completed in W3: added CR108 current-truth addendum; original 1.0 design and rollback wording preserved as historical design context. |
| CR108-DM-013 | `docs/quality/REVIEW-*`, `TEST-REPORT-*`, `VERIFICATION-REPORT-*` | Historical quality evidence uses `project_id: local_backtest`, `meta-po`, old roots, runtime-not-authorized wording. | `legacy_audit_keep` | Completed in W4: added `docs/quality/README.md` current index; historical test reports are not rewritten. |
| CR108-DM-014 | `docs/quality/FACTOR-RESEARCH-GUARDRAILS.md` | Guardrail doc appears current-facing; scan did not identify it as a major old-role hit in first pass. | `current_truth_boundary_note` | Review in CR108 before final docs scan; update only if current owner/authorization wording drifts. |
| CR108-DM-015 | `process/HLD.md`, `process/REQUIREMENTS.md`, `process/STORY-BACKLOG.md` | Historical process design and requirements mention `meta-po`, `local_backtest`, and prior roots extensively. | `inventory_only` | Do not bulk rewrite; mark as legacy process evidence and avoid using as current truth without context capsule. |
| CR108-DM-016 | `process/checkpoints`, `process/context`, `process/release` | Historical gates/context/release capsules mention old roles, old roots, and old runtime gates. | `inventory_only` | Do not bulk rewrite; keep as audit trail. If a current capsule points to wrong role/path, fix that specific current capsule only. |
| CR108-DM-017 | tests/scripts/config/tooling | Not fully scanned in CR108 docs inventory; CR107 plan assigns static scan to CR110. | `follow_up_candidate` | Defer to CR110 non-runtime verification unless a docs edit depends on it. |

## Initial Fix Order

| Wave | Target | Matrix IDs | Notes |
|---|---|---|---|
| CR108-W1 | README / USER-MANUAL | CR108-DM-001..005 | Current-facing user docs first. |
| CR108-W2 | docs/design current indexes | CR108-DM-007..010 | Avoid revision-history rewrites. |
| CR108-W3 | docs/features current indexes | CR108-DM-011..012 | Preserve CR-scoped design history. |
| CR108-W4 | docs/quality current entrypoints | CR108-DM-013..014 | Preserve historical reports. |
| CR108-W5 | process historical surfaces | CR108-DM-015..016 | Inventory-only. |

## Current Decision

- CR108-W1 completed for README / USER-MANUAL current-facing package identity, Host Orchestrator, and no-credential-read wording.
- CR108-W2 completed for `docs/design/FEATURE-DESIGN-MATRIX.md` current owner wording.
- CR108-W3 completed for `docs/features/cr061-package-import-layout/DESIGN.md` current-truth addendum.
- CR108-W4 completed by adding `docs/quality/README.md` current index.
- CR108-W5 completed as inventory-only: process historical HLD/requirements/story/checkpoint/context/release materials are preserved as audit evidence.
- Proceed to final CR108 verification and close readiness.
- Do not modify business code.
- Do not run runtime or credential-dependent commands.

## W1 Completion Evidence

| Check | Result | Evidence |
|---|---|---|
| Old current-facing role wording removed | PASS | `rg -n "meta-po|没有 console script|meta-po 授权路径|meta-po 负责编排" README.md docs/USER-MANUAL.md` returned no matches. |
| New Host Orchestrator / CLI wording present | PASS | `rg -n "quant_lab --help|Host Orchestrator|Agent 不得读取|--env-file .env" README.md docs/USER-MANUAL.md` returned expected current-facing entries. |
| CLI help smoke | PASS | `uv run python -m quant_lab.cli --help` returned `usage: quant_lab [-h] [--json]`; no `.env` read. |

## W2 Completion Evidence

| Check | Result | Evidence |
|---|---|---|
| Current design index owner wording | PASS | `docs/design/FEATURE-DESIGN-MATRIX.md` v1.4 uses Host Orchestrator / meta-se for current responsibility. |
| Historical revision records preserved | PASS | Revision rows that mention `meta-po` were not rewritten. |

## W3 Completion Evidence

| Check | Result | Evidence |
|---|---|---|
| CR061 feature design current truth | PASS | `docs/features/cr061-package-import-layout/DESIGN.md` v1.1 adds CR108 current-truth addendum. |
| Historical design context preserved | PASS | Original 1.0 design and rollback wording remain as historical CR061 context. |

## W4 Completion Evidence

| Check | Result | Evidence |
|---|---|---|
| Quality current entrypoint exists | PASS | `docs/quality/README.md` added as current index. |
| Historical reports preserved | PASS | Existing `docs/quality/*CR*` reports were not bulk rewritten. |

## W5 Completion Evidence

| Check | Result | Evidence |
|---|---|---|
| Process historical surfaces classified | PASS | CR108-DM-015 and CR108-DM-016 classify `process/HLD.md`, `process/REQUIREMENTS.md`, `process/STORY-BACKLOG.md`, `process/checkpoints`, `process/context`, and `process/release` as inventory-only. |
| Historical process materials preserved | PASS | No bulk rewrite of process historical design/readiness materials. |
