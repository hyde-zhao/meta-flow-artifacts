---
cr_id: "CR-050"
checkpoint: "CP7"
review_status: "PASS_WITH_RISK"
review_mode: "host-inline; external CP5 review input; independent CP7 unavailable"
---

# CR-050 Quality Review

## Findings

| ID | 严重度 | 状态 | 发现与处理 |
|---|---|---|---|
| CR050-F-001 | HIGH | RESOLVED | Git `check-ref-format` 会接受部分 shell-like 名称；新增原始 ref allowlist，再调用 Git 语法校验，并覆盖 `;`、`$()`、option-like、空白和换行负例。 |
| CR050-F-002 | HIGH | RESOLVED | open plan 与执行间 remote default 可漂移；执行在 fetch 后按授权 OID fresh reproof，漂移时在创建任何 ref 前 BLOCKED。 |
| CR050-F-003 | HIGH | RESOLVED | finish plan 与删除间 CR tip 可漂移；recovery ref 和每仓 remote delete 前 fresh reproof，漂移时 mutation=0。 |
| CR050-F-004 | MEDIUM | RESOLVED | finish remote-delete 顺序未冻结；固定为 `artifact→project`，并增加首仓已删、次仓失败后的 resume fixture。 |
| CR050-F-005 | LOW | RESOLVED | open→publish 的 operator commit 边界及 CLI 命名不够显式；LLD 已补 operational note，并统一四个 `meta-flow cr branch-*` 命令。 |

BLOCKER/未解决实现 finding：0。实现没有引入 GitPython、Git Town 或名义不唯一的 `gb` 依赖；没有使用 `+`、force、force-with-lease、local merge、rebase、force-delete 或自动 rollback。

## 核心安全判断

- 单仓 artifact default 成功不能推进 paired workflow truth：只有 2/2 exact PASS/NO_CHANGE 才令 `paired_projection_advanced`、`finish_allowed`、`cr_close_allowed` 为 true。
- 任何真实 ref mutation 都要求 operation/repository/ref/OID 绑定的 typed authorization；CP2/CP3/CP5 审批不等于执行授权。
- finish 不信任旧 merge result、CR closed 或 branch absent；必须重新观察 remote refs 与 ancestry。
- 远端不存在只可在 known tip 和 fresh proof 可重建时记为 `NO_CHANGE`，不能单独构成 PASS。

## Remaining risks

`CR050-RISK-WORKING-TREE-ONLY` 已由两仓普通 main 推送证据解除；`CR050-RISK-NO-INDEPENDENT-QA`、`CR050-RISK-BRANCH-PROTECTION`、`CR050-RISK-SQUASH-UNPROVABLE` 与真实 branch lifecycle/platform receipt 未验证仍 OPEN，因此 release ceiling 保持 `READY_WITH_RISK`。
