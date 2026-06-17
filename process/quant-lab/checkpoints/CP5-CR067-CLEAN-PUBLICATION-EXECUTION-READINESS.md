---
checkpoint_id: CP5-CR067-CLEAN-PUBLICATION-EXECUTION-READINESS
change_id: CR-067
checkpoint_type: manual
status: approved
created_at: '2026-06-15T12:01:19+08:00'
auto_precheck: process/checks/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-READINESS.md
context_capsule: process/context/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml
auto_final_authorization: false
---

# CP5 CR067 Clean Publication Execution Readiness

## 自动预检摘要

- 自动预检结论：PASS_WITH_RISK
- 自动预检文件：`process/checks/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-READINESS.md`
- CP5 关注点：是否允许在 CP2/CP3/CP5 全部批准后执行受控 post-approval sequence。
- 重要边界：CP5 approve 只授权本文件列出的最小 clean publication execution；仍不授权 force push、tag、history rewrite、remote deletion、NAS/data lake/runtime/credential、physical move、bulk import rewrite 或 CR046 recovery。

## Decision Brief

推荐决策：批准 CR067 的 CP5 执行准备，以便在 CP2/CP3/CP5 均通过后，按固定顺序执行：重新读取批准状态、冻结 manifest、创建临时 clean snapshot、复制 allowlist、运行 fail-closed scan、在临时 snapshot 中执行 Git 初始化/提交/精确 remote add/push、记录证据。该授权不是立即在当前文件中执行，也不是无限授权；执行前必须再次确认本检查点为 approved。

备选方案：继续只保留 strategy-only；直接发布当前分支；允许 history rewrite/force push；把发布改为 NAS/data lake/runtime 前置。推荐方案优先，因为它是 CR066 策略之后最小的可执行出口，并能避开 CR062 blocker。

影响维度：运行授权、manifest 变更控制、扫描失败处理、推送失败处理、验证范围、rollback/forward-fix、证据路径。

风险与回退：scan 或 push 前置校验失败时，CR067 标记 blocked，不执行后续 Git 写动作；如果 push 后发现问题，只允许 forward-fix 或停止，不允许 force push、history rewrite 或 remote deletion。

### Context Capsule Summary

- CP5 capsule：`process/context/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml`
- 关键事实：CP5 批准前不得创建临时 snapshot、不得运行 scan、不得 `git init/add/commit/remote add/push`。
- post-approval 执行仍必须不读取 `.env` 或凭据，不接触 NAS/data lake/runtime。

### Decision Collection Coverage

- 已扫描来源：CR067 正式 CR、CR067 execution plan、CR067 manifest、CR067 scan gate、CP2/CP3 自动检查、CP5 自动检查、CP5 context capsule、CR062 blocker evidence。
- 候选问题数：9
- 纳入待决策数：5
- N/A / 未纳入原因：具体命令执行证据在批准后生成；GitHub 远端认证、token、SSH 凭据读取不属于 CR067 授权项；真实 release/tag 不属于当前交付。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR067-01 | runtime_authorization | CP2/CP3/CP5 全部 approve 后，是否授权受控 post-approval sequence？ | 授权临时 clean snapshot、fail-closed scan、临时 repo git init/add/commit、exact remote add 和 push。 | 继续不执行；直接发布 current branch；授权 force/tag/rewrite。 | 推荐方案最小可执行；备选要么停滞，要么越过风险边界。 | 将产生 Git 写动作和外部 push，但仅限临时 snapshot 与 exact remote。 | 执行前发现批准状态不一致即停止。 |
| DQ-CP5-CR067-02 | implementation | 是否冻结 CR067 manifest allowlist/exclude/conditional include？ | 冻结当前 manifest；`.env.example` 仅 placeholder/no-secret scan PASS 时纳入。 | 运行时临时增减文件；发布 full process；只发 code-only。 | 推荐方案可审计；临时增减难追踪；full process 风险高。 | 可能漏掉非核心文档，需要后续 forward-fix。 | manifest 需改动时回到 CP5 修订。 |
| DQ-CP5-CR067-03 | security | 是否采用 no-secret/path/large/topology/cache 扫描 fail-closed？ | 采用；任一 BLOCKING/HIGH 即停止，不 commit/push。 | 只做 secret scan；警告但继续；push 后补扫。 | 推荐方案安全优先；备选可能公开敏感路径或大文件。 | 扫描规则可能产生 false positive。 | false positive 需记录并修订 scan gate 后重审。 |
| DQ-CP5-CR067-04 | risk_acceptance | 执行后的验证范围是否限定为 scoped validation？ | 限定为 import smoke / py_compile / manifest-vs-snapshot / scan evidence / push evidence，不跑全量 pytest。 | 跑全量 pytest；不跑验证。 | 推荐方案匹配 publication gate；全量 pytest 成本高且非本 CR 核心；不验证风险高。 | 无法证明全部业务行为，只证明发布包基础可用。 | 用户要求更强验证时扩展 CP7。 |
| DQ-CP5-CR067-05 | follow_up_tracking | scan/push 失败或后续问题如何处理？ | fail-closed；记录 blocker；forward-fix；不回 CR062 as-is。 | force push 修复；history rewrite；删除 remote；回 CR062。 | 推荐方案最小破坏；备选高风险或绕过 blocker。 | 可能需要新增 CR 或多轮修订。 | 用户另批高风险 CR 才能改变边界。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项 CP5 推荐方案；在 CP2/CP3/CP5 全部批准后，才授权“受控 post-approval sequence”。即使批准，也仍不授权以下操作：

- 不授权在当前仓库 current branch 上直接 publication。
- 不授权 force push、tag、history rewrite、branch creation / rename、remote deletion 或 remote set-url。
- 不授权读取或使用 `.env`、token、账号、密码、cookie、session、private key、SSH agent 细节或交易事实。
- 不授权 NAS/data lake/provider/catalog/runtime 操作。
- 不授权 physical move / rename / delete，不授权 bulk import rewrite。
- 不授权恢复 CR046，不授权 QMT/MiniQMT runtime、账户查询、下单、撤单、simulation/live。

## Entry Criteria

- CR067 正式 CR 已创建。
- CR067 CP2/CP3 context、自动预检和人工检查点均已准备。
- CR067 manifest、execution plan、scan gate 已准备。
- 用户尚未批准 CP2/CP3/CP5。

## Checklist

- [x] 执行顺序已固定。
- [x] manifest freeze 已定义。
- [x] scan fail-closed 已定义。
- [x] scoped validation 已定义。
- [x] rollback/forward-fix 已定义。
- [x] 不授权项已列出。
- [x] 用户完成人工审查并回复 `approve` / `修改: <具体修改点>` / `reject`。

## Exit Criteria

- 用户回复 `approve`：CP2/CP3/CP5 组合门禁通过；允许后续在重新读取批准状态后执行受控 post-approval sequence。
- 用户回复 `修改: <具体修改点>`：更新 CP5 执行准备、manifest 或 scan gate 后重新发起。
- 用户回复 `reject`：CR067 保持未批准，不执行任何 Git 写动作或外部 publication。

## Deliverables

- `docs/release/CLEAN-PUBLICATION-EXECUTION-PLAN-CR067.md`
- `docs/release/CLEAN-PUBLICATION-EXECUTION-MANIFEST-CR067.yaml`
- `docs/release/CLEAN-PUBLICATION-SCAN-GATE-CR067.md`
- `process/context/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml`
- `process/checks/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-READINESS.md`
- `process/checkpoints/CP5-CR067-CLEAN-PUBLICATION-EXECUTION-READINESS.md`

## 人工审查结果

- status: approved
- reviewed_by: user
- reviewed_at: '2026-06-15T12:45:02+08:00'
- decision: approve
- notes: 用户回复“同意”，按 CR067 CP2/CP3/CP5 组合门禁 approve 处理；接受 DQ-CP5-CR067-01..05 推荐方案。批准后仅授权受控 post-approval sequence：临时 clean snapshot、allowlist copy、fail-closed scan、临时 repo git init/add/commit、exact remote add/push；仍不授权 force push、tag、history rewrite、branch creation/rename、remote deletion、remote set-url、NAS/data lake/runtime/credential、physical move、bulk import rewrite 或 CR046 recovery。
