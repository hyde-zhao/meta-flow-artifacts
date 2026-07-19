---
status: "NEEDS_REWORK"
version: "1.0"
scope: "CR-051 / ST-AW-005 / independent review"
created_at: "2026-07-18T16:08:00Z"
---

# ST-AW-005 独立质量评审

## Findings

### REV-AW005-001 — denied descendant 未在逐对象读取前阻断

- 严重度：HIGH / BLOCKING
- 影响：跨项目内容隔离、隐私边界、readiness 可信度、NF-AW-005、TP-AW05-005/006。
- 根因：`_portable_scope()` 只校验显式 root；`_enumerate_root()`、`_target_state()` 与内容读取端口没有在每个 portable descendant 上再次应用 `_matches_denied_path()`。
- 触发条件：允许 root 是 deny pattern 的祖先，例如 allow `process/`、deny `process/quant-lab/**`。
- 实际结果：denied source/target 子树被枚举，sentinel 文件被 hash 并进入 mapping，readiness 可错误为 `READY`。
- 期望结果：在任何对象读取或 descend 前 fail closed；denied path 读取数、mapping 数与内容泄漏数均为 0，readiness 非 READY。
- 路由：`meta-dev-debugger`，同 Story R2；无需需求/HLD/LLD 变更，除非修复需要改变已批准 read-scope contract。

## 非阻断事项

- regular-file `lstat→open` swap 已由 `O_NOFOLLOW + fstat` 关闭。
- 对抗式并发目录 rename/replace 尚未由 dirfd/openat 全链路证明，保持 `R-AW005-DIR-ENUM-TOCTOU`。
- 真实迁移、Windows native no-follow 与 hosted remote 未授权或未验证，继续路由 CP8。

## 结论

在 `REV-AW005-001` 关闭前，不得把 ST-AW-005 标为 verified，不得进入 documentation / CP8。
