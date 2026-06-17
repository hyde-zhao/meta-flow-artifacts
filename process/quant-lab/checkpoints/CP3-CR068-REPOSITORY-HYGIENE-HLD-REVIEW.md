---
checkpoint_id: CP3-CR068-REPOSITORY-HYGIENE-HLD-REVIEW
change_id: CR-068
checkpoint_type: manual
status: approved
created_at: '2026-06-15T14:13:45+08:00'
auto_precheck: process/checks/CP3-CR068-REPOSITORY-HYGIENE-HLD-CONSISTENCY.md
context_capsule: process/context/CP3-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml
auto_final_authorization: false
---

# CP3 CR068 Repository Hygiene HLD Review

## 自动预检摘要

- 自动预检结论：PASS_WITH_RISK
- 自动预检文件：`process/checks/CP3-CR068-REPOSITORY-HYGIENE-HLD-CONSISTENCY.md`
- 推荐架构：temporary exact remote checkout + `.gitignore` delta + scan gate + ordinary fast-forward push。
- CP3 approve 只确认架构和安全边界；不授权执行。

## Decision Brief

推荐决策：采用“精确远端基线 + 单文件 `.gitignore` delta + 先扫描后提交 + 普通 fast-forward push”的 repo hygiene forward-fix 架构。该架构避免重开 current branch publication，也避免 force/rewrite/branch governance。

备选方案：直接在当前仓库 push；新建无父提交覆盖远端；把 branch/default branch governance 合并；不发布 `.gitignore`。推荐方案优先，因为它保持 CR067 公共基线连续性，并把变更面限制为 `.gitignore`。

影响维度：远端基线连续性、扫描前置、delta 范围、冲突处理、回滚策略、branch governance 分离。

风险与回退：remote baseline mismatch 或非 fast-forward 需求出现时 fail-closed；不 force、不 rewrite、不 rename branch；后续只能新 CR 或普通 forward-fix。

### Context Capsule Summary

- CP3 capsule：`process/context/CP3-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml`
- 读取策略：compact。
- 关键事实：CR068 只允许 `.gitignore` delta；branch/default branch governance 另起 CR。

### Decision Collection Coverage

- 已扫描来源：CR068 formal CR、CR068 plan、CR068 manifest、CR068 scan gate、CP3 context capsule、CR067 release notes、`.gitignore` tracked 状态。
- 候选问题数：8
- 纳入待决策数：5
- N/A / 未纳入原因：remote credentials、branch settings、NAS/lake/runtime、CR046 recovery 均非 CP3 架构范围，列为不授权项。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR068-01 | architecture | 是否采用 exact remote baseline + single-file delta 架构？ | 采用，从 CR067 remote baseline 做 `.gitignore` fast-forward。 | 当前分支发布；新无父提交；不发布。 | 推荐方案连续、最小；备选风险高或不解决问题。 | 需要 remote baseline 校验。 | baseline mismatch 时 fail-closed。 |
| DQ-CP3-CR068-02 | implementation | 是否冻结 delta 为 `.gitignore` 单文件？ | 冻结为 `.gitignore` 单文件。 | 同时追加 docs；允许运行时增减文件。 | 推荐方案可审计；备选扩大范围。 | 后续若需 docs 需新 DQ。 | delta 非单文件时回 CP5 修订。 |
| DQ-CP3-CR068-03 | security | scan gate 是否必须在 commit/push 前 PASS？ | 必须；任一 BLOCKING/HIGH 停止。 | commit 后再扫；只警告。 | 推荐方案 fail-closed；备选可能公开弱规则。 | 可能产生 false positive。 | false positive 需修订 scan gate。 |
| DQ-CP3-CR068-04 | architecture | branch/default branch governance 是否排除？ | 排除；保持 `master` 事实，不在 CR068 内治理。 | 合并 main/default branch 治理；立即 rename。 | 推荐方案降低权限面；备选需额外远端设置授权。 | `master` residual risk 保留。 | 用户另起 branch governance CR。 |
| DQ-CP3-CR068-05 | risk_acceptance | 是否接受 forward-fix only rollback？ | 接受；问题只能用新普通 commit 修复。 | 允许 force/rewrite；删除远端重建。 | 推荐方案低破坏；备选高风险。 | 错误修复需要新 commit。 | 用户另批高风险 CR 才能改变。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权执行 clone/fetch/commit/push；执行授权留给 CP5。
- 不授权 branch/default branch governance、force push、tag、history rewrite、remote deletion、remote set-url。
- 不授权读取 `.env`、token、账号、密码、cookie、session、private key 或交易事实。
- 不授权 NAS/data lake/provider/catalog/runtime、physical move、bulk import rewrite 或 CR046 recovery。

## Entry Criteria

- CR068 formal CR、plan、manifest、scan gate 已存在。
- CP3 context capsule 已创建。
- CP3 自动预检为 PASS_WITH_RISK。
- 用户已回复“同意”，CR068 CP2/CP3/CP5 组合门禁批准已收到。

## Checklist

- [x] 推荐架构已说明。
- [x] 备选方案与切换条件已说明。
- [x] branch governance out of scope 已说明。
- [x] scan-before-commit 已说明。
- [x] rollback/forward-fix 已说明。
- [x] 不授权项已显式列出。
- [x] 用户完成人工审查并回复 `approve` / `修改: <具体修改点>` / `reject`。

## Exit Criteria

- 用户回复 `approve`：CP3 架构评审通过，仍等待 CP5 执行准备确认。
- 用户回复 `修改: <具体修改点>`：更新架构和 Decision Brief 后重新发起。
- 用户回复 `reject`：CR068 保持未批准，不执行任何 Git 或远端操作。

## Deliverables

- `docs/release/REPOSITORY-HYGIENE-FORWARD-FIX-PLAN-CR068.md`
- `docs/release/REPOSITORY-HYGIENE-FORWARD-FIX-MANIFEST-CR068.yaml`
- `docs/release/REPOSITORY-HYGIENE-SCAN-GATE-CR068.md`
- `process/context/CP3-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml`
- `process/checks/CP3-CR068-REPOSITORY-HYGIENE-HLD-CONSISTENCY.md`
- `process/checkpoints/CP3-CR068-REPOSITORY-HYGIENE-HLD-REVIEW.md`

## 人工审查结果

- status: approved
- reviewed_by: user
- reviewed_at: 2026-06-15T14:40:47+08:00
- decision: approve
- notes: 用户回复“同意”，按本轮门禁 `approve` 兼容处理；接受 DQ-CP3-CR068-01..05 推荐方案。CP3 只批准 exact remote baseline、single-file delta、scan-before-commit 和 no branch/default branch governance 架构边界。
