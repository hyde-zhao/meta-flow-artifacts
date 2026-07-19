---
status: "NEEDS_REWORK"
version: "1.0"
scope: "CR-051 / ST-AW-005 / CP7"
created_at: "2026-07-18T16:08:00Z"
created_by: "meta-qa-critical probe + host-orchestrator evidence fallback"
---

# ST-AW-005 CP7 验证报告

## 阶段结论

`NEEDS_REWORK`。独立 meta-qa-critical 探针发现 `REV-AW005-001`（HIGH/BLOCKING）：较宽 allowlisted root 内的 denied 子树未在递归读取前重新匹配 deny policy，违反 sibling / `process/quant-lab/**` 内容读取数为 0 的硬门，并可错误输出 `READY`。

## 验证对象与追踪

| 验证对象 | 结论 | 证据 |
|---|---|---|
| 11 分区 immutable/portable manifest | PASS | 23 条定向测试、canonical layout fixture |
| missing target 非 READY | PASS | root-missing BLOCKED、mapping-missing MANUAL_REVIEW |
| static symlink 与 regular-file swap no-follow | PASS | `O_NOFOLLOW + fstat` fixture |
| O-AW-03 candidate-only | PASS | T1/T2/T3 边界与多阈值去重 fixture |
| denied 子树逐对象读取隔离 | **FAIL** | 独立宽 root 探针：`process/quant-lab/secret.txt` 被读取、映射且 readiness=`READY` |
| 真实 mutation / publication | PASS | 实际执行数 0；本轮只读 fixture |

## 独立复现

构造 portable root `process/`，同时设置 `denied_paths=("process/quant-lab/**", ...)`。在 source/target 的 `process/quant-lab/secret.txt` 放置 sentinel。`_portable_scope()` 因 root 自身 `process` 不命中 deny 而放行；`_enumerate_root()` 递归时未对 `process/quant-lab` 及其子项再次调用 deny matcher，最终 manifest 包含 denied mapping、读取了 sentinel，且 `reason_codes=()`、`readiness=READY`。

该缺陷证明现有“root 本身等于 `process/quant-lab` 时阻断”的测试不足以覆盖“宽 root 内含 denied 子树”的真实策略语义。既有测试全绿不能覆盖此失败路径。

## 质量结果

- 定向测试：23 passed。
- 独立 QA 相邻回归：254 passed + 9 subtests。
- Ruff、format、diff-check：通过。
- 独立 denied-subtree 探针：FAIL，形成 `REV-AW005-001`。
- 源码/测试修改：0；真实迁移、link、Git/worktree/ref/remote mutation：0。

## 路由

回 `meta-dev-debugger` 做 bounded R2：在任何 `lstat`、`readlink`、file open/hash 与 directory descend 前对当前 portable source/target path 应用 deny policy；命中后不得读取、不得写入 mapping，并必须 fail closed 为非 READY。补宽 root 下 source/target denied 子树及相关 symlink 负向覆盖。修复后由独立 meta-qa-critical 复验。

## 剩余风险

`R-AW005-DIR-ENUM-TOCTOU`、真实迁移未授权、Windows native no-follow 与 hosted remote 未验证继续保留；但这些非阻断风险不能替代当前 finding 的回修。
