---
status: draft
version: "1.0"
feature_id: "FEAT-GB-03"
---

# FEAT-GB-03 Test Plan

| Scope | 来源 | 方法 | 期望 |
|---|---|---|---|
| current 2/2 PASS后cleanup | TC-GB-006 | bare integration | proof→recovery→remote→local |
| PARTIAL/旧PASS/ref drift | TC-GB-007/008/015/017 | negative | delete=0 |
| recovery collision/remote partial | TC-GB-010/011 | fault/idempotence | PARTIAL且恢复面保留 |
| squash/non-ancestor/protected | TC-GB-008 | proof fixture | BLOCKED；force delete=0 |
| dry-run | TC-GB-009 | ref snapshot | local/remote mutation=0 |

真实forge receipt与protected platform不在本轮自动化证据中；风险保持OPEN。
