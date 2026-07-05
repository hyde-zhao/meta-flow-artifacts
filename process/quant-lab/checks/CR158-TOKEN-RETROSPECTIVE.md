---
report_id: "CR158-TOKEN-RETROSPECTIVE"
change_id: "CR-158"
status: "complete"
created_at: "2026-07-05T19:35:00+08:00"
created_by: "host-orchestrator"
measurement_quality: "estimated"
---

# CR158 Token and Redundancy Retrospective

## Measurement Caveat

Exact model/API token telemetry is not exposed in the current tool surface. This report uses available proxies:

- `process/state/READ-EXPANSION-LEDGER.ndjson` `estimated_tokens` for full/read-expansion events.
- CR158 artifact count and byte size as a proxy for generated/readable text volume.
- Observed checker repair loops and gate reissues from ledgers.

Therefore all token numbers below are directional estimates, not billing-grade counts.

## Recorded Read Expansion Tokens

| Stage | Estimated tokens | Share | Interpretation |
|---|---:|---:|---|
| CP0 | 4,200 | 14.1% | Intake read of CR158 and CR157 deferred-item provenance. |
| CP3 | 900 | 3.0% | Architecture discussion evidence check. |
| CP5 | 23,371 | 78.7% | HLD/ADR plus existing event/ML admission style reads for LLD implementability. This is the dominant recorded read cost. |
| CP6 | 1,240 | 4.2% | Implementation evidence audit read expansion. |
| Total recorded read expansion | 29,711 | 100% | Excludes normal file reads, generated output, tool output and user/assistant conversation tokens. |

## Artifact Volume Proxy

| Artifact group | Count | Bytes | Token implication |
|---|---:|---:|---|
| checkpoints | 8 | 68,499 | Human-gate Decision Briefs are one of the largest generated-text surfaces. |
| checks/results | 11 | 59,002 | Machine result JSONs are useful but verbose. |
| context capsules | 8 | 32,146 | Capsule layer is moderate and worth keeping. |
| stories / LLD / implementation | 12 | 86,581 | LLDs dominate written design volume. |
| quality docs | 4 | 9,994 | Compact and acceptable. |
| release docs/context | 6 | 10,838 | Low to moderate; possibly reducible for no-deploy changes. |
| CR158 process subset total | 48 files | 261,508 bytes | Roughly equivalent to tens of thousands of text tokens when read in full. |

## High-Cost Areas

| Rank | Area | Why tokens were spent | Optimization potential |
|---:|---|---|---|
| 1 | CP5 LLD and implementability | Five full LLDs plus broad source/design reads; CP5 read expansion alone recorded 23,371 tokens. | High: use batch-lld for similar adapter extension stories and keep only boundary/security stories full. |
| 2 | Human-gate Decision Brief repair | CP2 required reissue; CP8 required checker-token repair. | High: schema-render the checkpoint and launch message from one template before presenting to the user. |
| 3 | Checkpoint/status bookkeeping | Multiple JSON/YAML/Markdown state surfaces must be synchronized. | Medium: consolidate status-sync into one command and avoid manually touching CR frontmatter, STATE, CURRENT and ledgers separately. |
| 4 | Release readiness for no-deploy slice | Five scoped release docs were generated even though install/migration/deploy were N/A. | Medium: allow minimal profile for internal no-deploy slices. |
| 5 | Skill/rule reads | Current Codex skill protocol requires reading applicable SKILL.md files in full each turn. | Low under current rules; can reduce only by narrower skill selection and compact continuation context. |

## Suggested Optimizations

| ID | Recommendation | Expected token reduction | Risk / Tradeoff |
|---|---|---:|---|
| OPT-CR158-001 | Add a `human-gate render` command that generates CP2/CP3/CP5/CP8 checkpoint and launch message from one schema, including Decision Collection Coverage, CP-specific sections, `优劣分析`, CP8 split table and exact replies. | 15-25% for gate-heavy CRs | Requires maintaining one canonical renderer and tests. |
| OPT-CR158-002 | Permit `batch-lld` for homogeneous adapter extension stories under standard workflow, while requiring full LLD only for shared core and high-risk guardrail stories. | 20-35% for similar multi-story adapter CRs | Must preserve per-story anchors, ownership and acceptance criteria traceability. |
| OPT-CR158-003 | Add `meta-flow cr status-sync --id <CR>` after every CP approval to update CR frontmatter, summary, index, STATE/current and ledgers from the CP result. | 5-10% plus lower error risk | Needs careful overwrite rules to avoid clobbering user edits. |
| OPT-CR158-004 | Introduce `release_artifact_profile=minimal-internal` for no-install/no-deploy/no-migration local/static slices: keep release context and CP8 summary, generate separate release docs only when publish/install/deploy changes exist. | 5-10% for no-deploy CRs | Must still keep rollback/migration N/A and feedback route visible. |
| OPT-CR158-005 | Expand capsule-first discipline in CP5: only read full HLD/ADR/source modules when capsule has missing fields or conflict flags; record targeted section refs instead of full files. | 10-20% in design-heavy CRs | Requires richer CP5 context capsule and better source-symbol summaries. |

## Practical Conclusion

The largest avoidable cost in CR158 was not implementation or tests; it was governance text around CP5 design evidence and human-gate formatting repairs. The most effective next improvement is a validated gate renderer plus a batch-LLD policy for homogeneous Story groups. Together they should reduce similar CR overhead materially without weakening the audit trail.
