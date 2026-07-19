---
status: "closed"
version: "2.0"
scope: "CR-051 / ST-AW-005 / CP7-R2"
created_at: "2026-07-18T16:36:53Z"
---

# ST-AW-005 CP7-R2 回修状态

| Fix ID | Finding | 状态 | 独立复验证据 |
|---|---|---|---|
| `FIX-AW005-001` | `REV-AW005-001` | CLOSED | 3 类独立 probe + 3 closure + 26 directed + 95 adjacent + 280/9 combined；probe/mapping/leak=0 |

当前没有新的回修输入，不需要再修改源码或测试。

## 后续仅保留风险跟踪

目录枚举 TOCTOU、真实迁移、Windows native no-follow 和 hosted remote 四项不属于本 R2 修复范围。它们应由 CP8 分别作为 follow-up/risk acceptance/not-authorized/runtime-authorization 处理，不得包装成已完成或已授权。

## 复验触发条件

若后续改动 deny glob 语义、manifest/readiness schema、目录访问模型、Windows no-follow、真实 migration 或 remote 执行边界，必须重新打开对应测试与设计门；否则本 finding 保持 CLOSED。
