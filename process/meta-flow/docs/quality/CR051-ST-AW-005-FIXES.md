---
status: "pending"
version: "1.0"
scope: "CR-051 / ST-AW-005"
created_at: "2026-07-18T16:08:00Z"
created_by: "meta-qa-critical probe + host-orchestrator evidence fallback"
---

# ST-AW-005 回修输入

| Fix ID | 来源 | 修复方向 | 路由 | 验收 |
|---|---|---|---|---|
| FIX-AW005-001 | REV-AW005-001 | 将 deny policy 作为逐对象读取前置门；source/target 当前 portable path 在 lstat/readlink/open/hash/descend 前匹配；命中后不读取、不映射并标记非 READY | meta-dev-debugger | 宽 root 内 denied source/target subtree 的读取、hash、mapping、sentinel 泄漏均为 0；relative symlink 不绕过；既有 suite 不回归 |

## 边界

- 不修改批准的 allow/deny 语义，不新增 glob DSL。
- 不删除 `process/quant-lab/**` deny contract，不把 denied 子树静默当作“正常忽略后 READY”。命中 deny 表示 scope 冲突，必须留下 reason 并非 READY。
- 不扩展 CLI/shared writers，不执行真实迁移、软链接、Git/worktree/ref/remote mutation。
- 若修复要求改变 public manifest/readiness contract，必须写 design delta 并重开 CP5；按当前判断无需。

## CP7-R2 复验

独立 meta-qa-critical 至少复跑：新增 denied descendant 三类探针、23 条定向 suite、相邻回归、Ruff/format/pycompile/diff/危险 API 审计、return/evidence check。关闭 finding 前状态保持 `NEEDS_REWORK`。
