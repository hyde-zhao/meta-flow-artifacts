# CR075 Feedback

## Observation Signals

| Signal | Meaning | Route |
|---|---|---|
| Import smoke fails later | Target env drift or package layout regression | New CR or reopen CR075 with evidence |
| Tests require reports/data | CR075 scope insufficient for data readiness | Start CR076 candidate |
| Old root no longer needed | Legacy root retirement question | Start CR077 candidate |
| Remote branch/default branch issue | Git governance remains unresolved | Start CR078 candidate |
| Need QMT/MiniQMT verification | CR046 recovery requested | Explicitly resume CR046 through a separate gate |

## Feedback Boundary

Feedback can reference CR075 evidence, but must not include credentials, `.env` contents, account identifiers, trading facts, private paths from reports/data, or production samples.
