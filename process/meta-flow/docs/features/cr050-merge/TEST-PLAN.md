---
status: draft
version: "1.0"
feature_id: "FEAT-GB-04"
---

# FEAT-GB-04 Test Plan

| Scope | 来源 | 方法 | 期望 |
|---|---|---|---|
| exact 2/2 FF | TC-GB-012 | bare integration | defaults=published tips；artifact→project |
| non-FF/forbidden commands | TC-GB-013 | DAG+command spy | BLOCKED；merge/rebase/force=0 |
| authz/policy rejection | TC-GB-014 | typed authz negative+remote hook | writes=0或PARTIAL；不绕过 |
| artifact PASS/project fail | TC-GB-015 | fault injection | projection/finish/close=false；branches retained |
| dry-run | TC-GB-016 | ref snapshot/golden plan | mutation=0；plan确定 |
| merge-finish gate | TC-GB-017 | integration | 2/2 current proof前delete=0 |

安全断言：argv refspec无`+`，无`--force*`，无本地checkout/merge；URL/stderr redacted。切换断言：无法隔离projection即测试FAIL而非waive。
