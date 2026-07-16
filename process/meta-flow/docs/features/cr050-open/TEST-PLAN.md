---
status: draft
version: "1.0"
feature_id: "FEAT-GB-01"
---

# FEAT-GB-01 Test Plan

| Scope ID | 覆盖 | 来源 | 层级 | 状态 |
|---|---|---|---|---|
| TP-GB-01 | fresh 2/2 default到同名branch/upstream | TC-GB-001 | bare integration | planned |
| TP-GB-02 | dirty/detached/collision/default未知 | TC-GB-002/003 | unit+bare | planned |
| TP-GB-03 | dry-run零mutation与artifact中途失败 | TC-GB-009/010 | command-spy/fault | planned |

| Case | 条件 | 期望 |
|---|---|---|
| SEC-OPEN-01 | 非法ref/NUL/选项前缀 | BLOCKED；subprocess=0 |
| SEC-OPEN-02 | 任一全局preflight失败 | project/artifact new refs=0 |
| FAIL-OPEN-01 | local准备后artifact push拒绝 | PARTIAL；不自动删除已建ref；resume完整 |

手工验收仅审查CLI help和机器JSON；真实remote branch创建需要另行授权，不属于CP7必需证据。
