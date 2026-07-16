---
story_id: "ST-GB-002"
cr_id: "CR-050"
stage: "CP6"
status: "implemented"
execution_mode: "inline-fallback"
implemented_at: "2026-07-16"
---

# ST-GB-002 Implementation

## 实现对象

- `git_branch_lifecycle.py` 增加 `PublishEvidence`、`plan_publish` 与 `execute_publish`：入口捕获 2/2 local CR branch full OID，要求 clean tree、current branch、upstream 和 non-FF 关系全部匹配后，才允许普通 exact-OID ref push。
- `branch-publish` CLI 使用与 open 相同的 typed authz/result writer；actual 强制 `--output`，dry-run 只输出 deterministic plan。
- bare fixtures 覆盖 2/2 exact publication、matching evidence 与 dirty-tree 负例。

## 契约映射

Publish 不调用 `git add/commit/amend`，也不根据执行期间 HEAD 重新选择 OID。已匹配 remote tip 返回 `NO_CHANGE`；后仓失败保留前仓事实并返回 `PARTIAL`。`PublishEvidence` 只有 fresh post-query exact 等于 captured local OID 时才生成，供 ST-GB-004 消费。

## 验证与边界

- Publish fixtures：2 passed，9 deselected。
- legacy workspace/CR lifecycle：36 passed。
- CLI help 可发现 `branch-publish`；Ruff 和 diff hygiene 通过。
- 仅临时 bare remote 被写；当前仓 stage/commit/push 数为 0。

## 设计差异与交接

无设计差异。真实 remote policy、credential 与 receipt 仍未验证；inline fallback 不构成独立 meta-dev attestation。
