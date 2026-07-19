---
document_type: story-test-report
cr_id: CR-051
story_id: ST-AW-002
checkpoint: CP7
decision: PASS_WITH_RISK
---

# ST-AW-002 CP7 测试报告

- 83/83 tests PASS（CAP/DUR/WT/PORT/TC-AW）。
- Ruff、py_compile、diff PASS。
- 危险恢复 argv=0；唯一 `stash` 文本为错误提示，不会执行。
- 所有 Git/worktree/remote mutation 仅临时 pytest fixture/local bare remote；真实目标=0。
- 未执行：真实 shared remote/worktree、Windows durability pilot。
