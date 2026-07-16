---
story_id: "ST-GB-003"
cr_id: "CR-050"
stage: "CP6"
status: "implemented"
execution_mode: "inline-fallback"
implemented_at: "2026-07-16"
---

# ST-GB-003 Implementation

## 实现对象

- `plan_finish` 要求 current matching 2/2 `PairedMergeProjection`、独立 delete authz、clean/attached worktree、非 protected CR ref、fresh known-tip identity 与 `known_tip <= remote default` ancestry。
- `execute_finish` 在任何 remote delete 前为 2/2 repositories 创建或复用 local-only `refs/meta-flow/recovery/<cr>/<fingerprint>`；同名不同 OID fail closed。
- 远端删除固定 artifact→project，使用普通 exact delete refspec并逐仓验证 absent；第一仓成功、第二仓失败时返回 `PARTIAL`，两仓 local branch 均保留。
- fresh resume 对有 recovery/known-tip/ancestry 证明的 artifact absent 返回 `NO_CHANGE`，再处理 project。只有 2/2 remote absent 后，才从包含 known tip 的 detached commit 执行安全 `branch -d`，随后恢复 local default；不隐式更新 default。

## 安全边界

Branch absent 本身不是合并证明；old/partial projection、unknown tip、drift、non-ancestor、protected branch 与 authz mismatch 均在 delete 前阻断。禁止 `branch -D`、force、reset、自动 recovery-ref push/delete、CR closed 反向作证。

## 验证与边界

- Finish/proof/resume fixtures：3 passed，8 deselected。
- legacy workspace/CR lifecycle：36 passed。
- CLI 明确要求 merge result/authz；Ruff/diff PASS。
- remote delete 只发生在 pytest 临时 bare remote；当前两仓 remote ref 未修改。

## 设计差异与交接

无设计差异。评审要求的 artifact→project remote-delete 顺序和 PARTIAL resume fixture 已机器验证。Git ancestry 对 squash/rebase 仍按批准设计 fail closed；真实平台自动删 branch/receipt 未验证。
