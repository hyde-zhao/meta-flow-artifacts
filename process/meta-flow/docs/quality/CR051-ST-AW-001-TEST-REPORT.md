---
document_type: story-test-report
cr_id: CR-051
story_id: ST-AW-001
checkpoint: CP7
decision: NEEDS_REWORK
---

# CR-051 / ST-AW-001 测试报告

## 汇总

| 类别 | 通过 | 失败 | N/A |
|---|---:|---:|---:|
| 定向 pytest | 36 | 0 | 0 |
| 静态 / 编译 / diff | 3 | 0 | 0 |
| 独立新增负例 | 0 | 2 | 0 |
| 仓库级边界检查 | 0 | 0 | 2 |

## 结果

- pytest：36 passed。
- Ruff、`py_compile`、`git diff --check`：PASS。
- module-boundaries/import：因 canonical manifest 缺失而 N/A；不是实现 PASS 证据。
- 新增负例失败：跨项目 config/decision mismatch 未阻断；legacy 非字符串 `owned_paths` 泄漏裸 `TypeError`。
- 真实 artifact、Git、worktree、ref、remote、link、migration mutation：0。

## 重验要求

回修后必须把两个负例固化为自动测试，并重跑 36 个既有测试、两个新增测试、Ruff、`py_compile` 和 diff 检查。若 module manifest 仍缺失，继续显式记录 N/A 与 CP8 风险，不得伪造通过。
