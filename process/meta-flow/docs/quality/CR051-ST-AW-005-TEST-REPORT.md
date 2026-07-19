---
status: "NEEDS_REWORK"
version: "1.0"
scope: "CR-051 / ST-AW-005 / CP7 tests"
created_at: "2026-07-18T16:08:00Z"
---

# ST-AW-005 CP7 测试报告

| 层级 | 结果 | 判定 |
|---|---|---|
| 定向 suite | 23 passed | 既有 happy/negative fixture 全绿 |
| 相邻 suite | 254 passed + 9 subtests | routing/worktree/leg/aggregate 未回归 |
| 静态质量 | Ruff / format / diff-check PASS | 无静态阻断 |
| root-level denied scope | PASS | root 本身是 `process/quant-lab` 时阻断 |
| nested denied subtree | **FAIL** | root=`process` 时 `process/quant-lab/**` 被读取和映射，READY 误报 |

失败并非测试环境或平台差异。它暴露的是测试矩阵缺口：原 suite 只覆盖 deny pattern 与 root 完全/前缀命中，没有覆盖 allowlisted ancestor root 内的 denied descendant。

R2 最小回归必须新增：

1. source 宽 root 内 denied 目录，目录及子项的 lstat/open/hash/descend 次数为 0；
2. target 宽 root 内 denied 目录，同样读取数为 0；
3. 合法 sibling symlink 或相对 symlink 指向 denied portable descendant 时不跟随、不读取；
4. manifest 不含 denied path、hash 或 sentinel，readiness 必须 `BLOCKED` 或 `MANUAL_REVIEW`；
5. 23 条原定向、相邻 suite、Ruff/format/pycompile/危险 API 审计不回归。
