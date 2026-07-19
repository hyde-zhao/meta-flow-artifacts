---
status: "PASS_WITH_RISK"
version: "2.0"
scope: "CR-051 / ST-AW-005 / CP7-R2 tests"
created_at: "2026-07-18T16:36:53Z"
---

# ST-AW-005 CP7-R2 测试报告

## 摘要

`REV-AW005-001` 的三类复现路径全部转绿，精确回归计数满足验证包要求，未发现新缺陷。结论为 `PASS_WITH_RISK`。

## 结果

| 测试层 | 命令摘要 | 结果 |
|---|---|---|
| finding closure | pytest `-k 'denied_descendant or denied_target_descendant or relative_symlink_to_denied'` | 3 passed, 23 deselected |
| directed | pytest `tests/test_artifact_migration.py` | 26 passed |
| adjacent | pytest migration + routing + worktree | 95 passed |
| exact combined | pytest 9 个 CR-051 相关模块 | 280 passed, 9 subtests passed |
| lint | Ruff check | PASS |
| format | Ruff format check | PASS |
| syntax | py_compile | PASS |
| whitespace | git diff --check（SUT/test scoped） | PASS |
| dangerous API/import | `rg` mutator/subprocess patterns | PASS：无命中 |

## 独立负向夹具

| Fixture | 关键断言 | 结果 |
|---|---|---|
| 宽 `process/` source root + denied `process/quant-lab/**` | denied lstat/open/scandir-descend/hash/mapping=0；sentinel/hash leak=0；非 READY | PASS |
| source `legacy/meta-flow` allowed、mapped target `process/quant-lab` denied | 在 source object probe/hash 前阻断；source/target denied probe=0；mapping/leak=0 | PASS |
| allowed relative symlink → mapped denied descendant | denied target probe=0；`link_target=null`；`relative-denied`；payload 无 link text/sentinel | PASS |

## Acceptance 覆盖

验证包 6 项 acceptance 覆盖率为 100%。既有 11 分区 manifest、portable digest、missing target、EACCES、absolute/broken symlink、regular-file symlink-swap、无 `O_NOFOLLOW` fail-closed、zero mutation 与 O-AW-03 单一不可执行 candidate 均由 26 条 directed suite 保持覆盖。

## 覆盖缺口

未做真实项目迁移、真实 worktree/ref/remote、Windows native no-follow、对抗式并发目录 rename/replace。前三项未授权或平台不可用；最后一项是 R2 明确 non-goal。它们已作为四项剩余风险进入 CP8，不影响 `REV-AW005-001` 关闭。

## 阶段决策

`PASS_WITH_RISK`。没有需要 meta-dev 再回修的测试失败。
