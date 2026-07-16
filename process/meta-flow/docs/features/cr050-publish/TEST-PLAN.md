---
status: draft
version: "1.0"
feature_id: "FEAT-GB-02"
---

# FEAT-GB-02 Test Plan

| Scope | 来源 | 方法 | 期望 |
|---|---|---|---|
| 2/2 existing commit publish | TC-GB-004 | bare integration | remote CR OID=entry HEAD |
| dirty/wrong/non-FF | TC-GB-005 | negative fixtures | BLOCKED；writes=0 |
| partial/resume/dry-run | TC-GB-009/010 | fault+command spy | facts retained；mutation=0 for dry-run |
| no implicit commit | REQ-GB-C001/C002 | argv spy | add/commit/amend count=0 |

风险：执行时HEAD漂移、remote并发、stderr泄露；分别用captured OID post-check、non-FF remote、redaction fixture验证。
