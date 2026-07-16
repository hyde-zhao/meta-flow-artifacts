---
story_id: "ST-GB-004"
cr_id: "CR-050"
stage: "CP6"
status: "implemented"
execution_mode: "inline-fallback"
implemented_at: "2026-07-16"
---

# ST-GB-004 Implementation

## 实现对象

- `OperationAuthorization`/loader/validator 绑定 operation、CR、branch、remote、repository fingerprint、default branch、fresh default/published full OID、issuer、expiry 与 single-use 语义；CP gate approval 不能替代该对象。
- `plan_merge` 重新观察 2/2 remote CR/default refs，要求 matching `PublishEvidence`、fresh exact OID 与 `default <= published tip`；任一 preflight 失败时 default writes=0。
- `execute_merge` 固定 artifact→project，用 `git push --porcelain <remote> <published_oid>:refs/heads/<default>` 普通更新并逐仓 post-query；不含 `+`、force、merge、rebase 或自动冲突处理。
- `project_merge` 只有 2/2 `PASS|NO_CHANGE` 且 after=expected 时才把 `paired_complete/paired_projection_advanced/finish_allowed/cr_close_allowed` 全部置 true。

## 安全不变量与恢复

Fault fixture 注入 artifact PASS、project rejection，断言 overall=`PARTIAL`、两仓 CR refs 均保留、三个下游授权布尔均 false、automatic rollback=0。fresh retry 重新读取 refs，artifact exact 时为 `NO_CHANGE`，只继续 project；不复用旧 PASS 绕过观察。

## 验证与边界

- Merge/projection fixtures：3 passed，8 deselected。
- legacy workspace/CR lifecycle：36 passed。
- CLI 明确要求 publish result/authz；Ruff/diff PASS。
- 只对临时 bare remote 执行 default ref 更新；当前真实 default write 数为 0。

## 设计差异与交接

无设计差异；artifact-first PARTIAL 投影 firewall 已形成机器断言，因此未触发 project-first/重开 CP2/CP3。真实 branch protection/merge queue/receipt 仍未验证。
