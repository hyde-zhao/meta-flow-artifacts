---
checkpoint_id: CP5-CR068-REPOSITORY-HYGIENE-READINESS
change_id: CR-068
checkpoint_type: manual
status: approved
created_at: '2026-06-15T14:13:45+08:00'
auto_precheck: process/checks/CP5-CR068-REPOSITORY-HYGIENE-READINESS.md
context_capsule: process/context/CP5-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml
auto_final_authorization: false
---

# CP5 CR068 Repository Hygiene Readiness

## 自动预检摘要

- 自动预检结论：PASS_WITH_RISK
- 自动预检文件：`process/checks/CP5-CR068-REPOSITORY-HYGIENE-READINESS.md`
- CP5 关注点：是否允许在 CP2/CP3/CP5 全部批准后执行受控 repo hygiene forward-fix。
- 重要边界：CP5 approve 只授权本文件列出的最小 `.gitignore` forward-fix；仍不授权 branch/default branch governance、force push、tag、history rewrite、remote deletion、remote set-url、NAS/data lake/runtime/credential、physical move、bulk import rewrite 或 CR046 recovery。

## Decision Brief

推荐决策：批准 CR068 的 CP5 执行准备，以便在 CP2/CP3/CP5 均通过后，按固定顺序执行：重新读取批准状态、临时 exact remote checkout、验证 CR067 baseline、复制当前 tracked `.gitignore`、运行 fail-closed scan、commit only `.gitignore`、普通 fast-forward push、记录证据。该授权不是无限授权；执行前必须再次确认本检查点为 approved。

备选方案：继续只保留风险；把 `.gitignore` 合并到 branch governance；允许 force/rewrite；把 repo hygiene 改为 NAS/data lake/runtime 前置。推荐方案优先，因为它是 CR067 后最小 forward-fix。

影响维度：运行授权、manifest freeze、扫描失败处理、远端 baseline mismatch 处理、验证范围、rollback/forward-fix、证据路径。

风险与回退：scan 或 remote baseline 校验失败时，CR068 标记 blocked，不执行 commit/push；如果 push 后发现问题，只允许 forward-fix 或停止，不允许 force push、history rewrite 或 remote deletion。

### Context Capsule Summary

- CP5 capsule：`process/context/CP5-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml`
- 关键事实：CP5 批准前不得 clone/fetch/commit/push。
- post-approval 执行仍必须不读取 `.env` 或凭据，不接触 NAS/data lake/runtime。

### Decision Collection Coverage

- 已扫描来源：CR068 正式 CR、CR068 plan、CR068 manifest、CR068 scan gate、CP2/CP3 自动检查、CP5 自动检查、CP5 context capsule、CR067 release evidence、`.gitignore` tracked 状态。
- 候选问题数：9
- 纳入待决策数：5
- N/A / 未纳入原因：GitHub branch settings、default branch、tag/release、credential details、NAS/lake/runtime/CR046 均不属于 CR068 授权项。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR068-01 | runtime_authorization | CP2/CP3/CP5 全部 approve 后，是否授权受控 forward-fix sequence？ | 授权 exact remote checkout、baseline verify、`.gitignore` copy、scan、commit only `.gitignore`、fast-forward push。 | 继续不执行；合并 branch governance；授权 force/rewrite。 | 推荐方案最小可执行；备选要么停滞，要么越过风险边界。 | 将产生 Git 读写和外部 push，但仅限 `.gitignore` forward-fix。 | 执行前发现批准状态不一致即停止。 |
| DQ-CP5-CR068-02 | implementation | 是否冻结 CR068 manifest delta 为 `.gitignore`？ | 冻结为当前 tracked `.gitignore`。 | 运行时临时增减文件；同时追加 docs。 | 推荐方案可审计；临时增减难追踪。 | 可能漏掉其他 hygiene 文件。 | manifest 需改动时回到 CP5 修订。 |
| DQ-CP5-CR068-03 | security | 是否采用 repo hygiene scan fail-closed？ | 采用；任一 BLOCKING/HIGH 即停止，不 commit/push。 | 只做 warning；push 后补扫。 | 推荐方案安全优先；备选可能公开弱规则。 | 扫描规则可能产生 false positive。 | false positive 需记录并修订 scan gate 后重审。 |
| DQ-CP5-CR068-04 | risk_acceptance | 执行后的验证范围是否限定为 scoped validation？ | 限定为 `.gitignore` delta、scan evidence、remote baseline、fast-forward push evidence。 | 跑全量测试；不跑验证。 | 推荐方案匹配 repo hygiene；全量测试非核心；不验证风险高。 | 无法证明业务行为，只证明 repo hygiene forward-fix。 | 用户要求更强验证时扩展 CP7。 |
| DQ-CP5-CR068-05 | follow_up_tracking | push 失败或后续问题如何处理？ | fail-closed；记录 blocker；forward-fix；不 force/rewrite/delete remote。 | force push 修复；history rewrite；删除 remote；回 CR062。 | 推荐方案最小破坏；备选高风险或绕过 blocker。 | 可能需要新增 CR 或多轮修订。 | 用户另批高风险 CR 才能改变边界。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项 CP5 推荐方案；在 CP2/CP3/CP5 全部批准后，才授权“受控 forward-fix sequence”。即使批准，也仍不授权以下操作：

- 不授权 branch creation / branch rename / default branch governance。
- 不授权 force push、tag、history rewrite、remote deletion 或 remote set-url。
- 不授权发布 current branch as-is。
- 不授权读取或使用 `.env`、token、账号、密码、cookie、session、private key、SSH agent 细节或交易事实。
- 不授权 NAS/data lake/provider/catalog/runtime 操作。
- 不授权 physical move / rename / delete，不授权 bulk import rewrite。
- 不授权恢复 CR046，不授权 QMT/MiniQMT runtime、账户查询、下单、撤单、simulation/live。

## Entry Criteria

- CR068 正式 CR 已创建。
- CR068 CP2/CP3 context、自动预检和人工检查点均已准备。
- CR068 plan、manifest、scan gate 已准备。
- 用户已回复“同意”，CR068 CP2/CP3/CP5 组合门禁批准已收到。

## Checklist

- [x] 执行顺序已固定。
- [x] manifest delta 已定义为 `.gitignore`。
- [x] scan fail-closed 已定义。
- [x] scoped validation 已定义。
- [x] rollback/forward-fix 已定义。
- [x] 不授权项已列出。
- [x] 用户完成人工审查并回复 `approve` / `修改: <具体修改点>` / `reject`。

## Exit Criteria

- 用户回复 `approve`：CP2/CP3/CP5 组合门禁通过；允许后续在重新读取批准状态后执行受控 `.gitignore` forward-fix sequence。
- 用户回复 `修改: <具体修改点>`：更新 CP5 执行准备、manifest 或 scan gate 后重新发起。
- 用户回复 `reject`：CR068 保持未批准，不执行任何 Git 写动作或外部 publication。

## Deliverables

- `docs/release/REPOSITORY-HYGIENE-FORWARD-FIX-PLAN-CR068.md`
- `docs/release/REPOSITORY-HYGIENE-FORWARD-FIX-MANIFEST-CR068.yaml`
- `docs/release/REPOSITORY-HYGIENE-SCAN-GATE-CR068.md`
- `process/context/CP5-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml`
- `process/checks/CP5-CR068-REPOSITORY-HYGIENE-READINESS.md`
- `process/checkpoints/CP5-CR068-REPOSITORY-HYGIENE-READINESS.md`

## 人工审查结果

- status: approved
- reviewed_by: user
- reviewed_at: 2026-06-15T14:40:47+08:00
- decision: approve
- notes: 用户回复“同意”，按本轮门禁 `approve` 兼容处理；接受 DQ-CP5-CR068-01..05 推荐方案。批准后仅授权受控 `.gitignore` forward-fix sequence，不授权 branch/default branch governance、force/tag/rewrite、remote deletion/set-url、NAS/data lake/runtime/credential、physical move、bulk import rewrite 或 CR046 recovery。
