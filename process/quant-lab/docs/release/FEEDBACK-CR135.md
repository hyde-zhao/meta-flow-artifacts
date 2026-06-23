---
title: "CR135 Feedback"
source_cr: "CR-135"
status: "ready_with_risk"
profile: "compact"
created_at: "2026-06-23"
---

# CR135 Feedback

## Observation Signals

| 信号 | 触发条件 | 分流 |
|---|---|---|
| Bundle file mismatch | manifest hash / bytes mismatch | bugfix CR |
| Replay ambiguity | user expects replay to rerun package | docs / naming clarification CR |
| Tracking warning | user requires zero-warning `cr-tracking` | FU-CR135-001 candidate |
| Permission boundary concern | any request to include runtime/NAS/QMT/credentials/trading | runtime authorization CR |

## Feedback Routing

Feedback that changes bundle schema, CLI behavior or replay semantics should start a new formal CR. Runtime, NAS, QMT, credential, provider/lake/catalog and trading feedback must not be folded into CR135 closure.
