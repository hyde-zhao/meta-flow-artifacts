---
checkpoint_id: CP2-CR068-REPOSITORY-HYGIENE-BASELINE
change_id: CR-068
checkpoint_type: manual
status: approved
created_at: '2026-06-15T14:13:45+08:00'
auto_precheck: process/checks/CP2-CR068-REPOSITORY-HYGIENE-BASELINE.md
context_capsule: process/context/CP2-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml
auto_final_authorization: false
---

# CP2 CR068 Repository Hygiene Baseline

## 自动预检摘要

- 自动预检结论：PASS_WITH_RISK
- 自动预检文件：`process/checks/CP2-CR068-REPOSITORY-HYGIENE-BASELINE.md`
- 本检查点只确认 CR068 范围基线；不授权修改 `.gitignore`、clone/fetch remote、commit、push 或 branch/default branch governance。
- 风险来源：CR067 public baseline 缺少 `.gitignore`，后续若批准需要普通 fast-forward forward-fix。

## Decision Brief

推荐决策：批准 CR068 作为 CR067 后续的 Repo Hygiene / `.gitignore` Forward-fix Gate，范围限定为“把当前 tracked `.gitignore` 作为 public baseline 的受控补丁候选”，先冻结需求基线、边界和后续 CP3/CP5 审查输入；在 CP2/CP3/CP5 全部通过前，不授权任何实现或远端操作。

备选方案：保持 CR068 为候选；把 `.gitignore` 风险继续留给后续；把 branch/default branch governance 合并进本 CR；恢复 CR046。推荐方案优先，因为它关闭 CR067 已知低成本残余风险，同时不扩大到分支治理、NAS/lake/runtime 或交易权限面。

影响维度：public repository hygiene、安全扫描、CR067 residual risk、CR062 blocked 状态、CR046 paused 状态、后续 CR 排序。

风险与回退：如果用户不批准 CP2，CR068 保持 active pending，不进入执行；如果后续发现需要 branch governance、NAS/data lake/runtime 或 CR046 recovery，必须新建或修订 CR，不得在 CR068 内静默扩大。

### Context Capsule Summary

- CP2 capsule：`process/context/CP2-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml`
- 读取策略：compact；默认不读取完整历史文档或 `.env`。
- 关键事实：CR067 已 READY_WITH_RISK 关闭；public baseline 缺少 `.gitignore`；CR068 CP2 只确认 scope。

### Decision Collection Coverage

- 已扫描来源：`process/STATE.md`、`process/changes/CR-INDEX.yaml`、`process/checkpoints/CP8-CR067-DELIVERY-READINESS.md`、`docs/release/RELEASE-NOTES-CR067.md`、`.gitignore` tracked 状态、CR068 formal CR、CP2 context capsule。
- 候选问题数：7
- 纳入待决策数：5
- N/A / 未纳入原因：branch/default branch governance、NAS/data lake/runtime/CR046 recovery 均超出 CR068，列为不授权项；具体执行授权留给 CP5 runtime_authorization。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR068-01 | scope | 是否正式启动 CR068，承接 CR067 的 `.gitignore` residual risk？ | 启动 CR068，并登记为 active formal CR；CR062 保持 blocked，CR046 保持 paused。 | 保持为候选；继续接受风险；恢复 CR046。 | 推荐方案关闭明确残余风险；备选会延迟 public hygiene。 | 新增一轮 CP2/CP3/CP5，但边界清晰。 | 用户 reject 时 CR068 保持 pending，不执行。 |
| DQ-CP2-CR068-02 | scope | CR068 范围是否限定为 `.gitignore` forward-fix？ | 限定为当前 tracked `.gitignore` 的 public baseline forward-fix。 | 合并 branch governance；合并 NAS/lake；合并 runtime。 | 推荐方案最小；备选扩大权限面。 | 后续治理仍需独立 CR。 | 用户要求合并时重新做 CR 影响分析。 |
| DQ-CP2-CR068-03 | security | 是否接受 `.gitignore` 作为需要扫描的 public hygiene 文件？ | 接受；CP5 后必须先扫描再 commit/push。 | 不发布 `.gitignore`；直接发布不扫描。 | 推荐方案保留安全门；跳过扫描风险高。 | 需要额外 scan gate。 | scan 出现 BLOCKING/HIGH 时 fail-closed。 |
| DQ-CP2-CR068-04 | follow_up_tracking | CR062 blocked baseline 如何处理？ | 保持 CR062 blocked；CR068 只做 CR067 baseline forward-fix。 | 回到 CR062；将 CR062 标记 superseded。 | 推荐方案不重开 current branch publication；备选需更多台账处理。 | CR062 仍作为历史 blocker。 | 后续可单独做 CR062 supersede/close hygiene。 |
| DQ-CP2-CR068-05 | risk_acceptance | 是否确认 CP2 不授权任何远端或文件修改？ | 确认；CP2 只批准范围。 | CP2 直接授权执行；跳过 CP3/CP5。 | 推荐方案遵守门禁；备选越权。 | 执行延后一轮确认。 | CP3/CP5 approved 后再执行。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权修改 `.gitignore` 本体。
- 不授权 git clone/fetch、git add、git commit、git push、git tag、remote deletion、remote set-url。
- 不授权 branch creation / rename 或 default branch governance。
- 不授权 force push、history rewrite 或 current branch as-is publication。
- 不授权读取 `.env`、token、账号、密码、cookie、session、private key 或交易事实。
- 不授权 NAS/data lake/provider/catalog/runtime 操作、physical move、bulk import rewrite 或 CR046 recovery。

## Entry Criteria

- CR067 已 closed-current-delivery / READY_WITH_RISK。
- CR067 release notes 记录 `.gitignore` 未纳入 public baseline。
- 用户已说“按照你的建议继续推进”，表示可以启动 Repo Hygiene follow-up CR 材料。
- 用户已回复“同意”，CR068 CP2/CP3/CP5 组合门禁批准已收到。

## Checklist

- [x] CR068 正式 CR 已创建。
- [x] CP2 context capsule 已创建。
- [x] CP2 自动预检已完成并标记 PASS_WITH_RISK。
- [x] 决策项已收集并列入本文件。
- [x] 不授权项已显式列出。
- [x] 用户完成人工审查并回复 `approve` / `修改: <具体修改点>` / `reject`。

## Exit Criteria

- 用户回复 `approve`：CP2 需求基线通过，进入 CP3/CP5 组合门禁确认；仍不授权执行。
- 用户回复 `修改: <具体修改点>`：更新 CR068 范围和 Decision Brief 后重新发起。
- 用户回复 `reject`：CR068 保持未批准，不执行后续动作。

## Deliverables

- `process/changes/CR-068-REPOSITORY-HYGIENE-GITIGNORE-FORWARD-FIX-2026-06-15.md`
- `process/context/CP2-CR068-REPOSITORY-HYGIENE-CONTEXT.yaml`
- `process/checks/CP2-CR068-REPOSITORY-HYGIENE-BASELINE.md`
- `process/checkpoints/CP2-CR068-REPOSITORY-HYGIENE-BASELINE.md`

## 人工审查结果

- status: approved
- reviewed_by: user
- reviewed_at: 2026-06-15T14:40:47+08:00
- decision: approve
- notes: 用户回复“同意”，按本轮门禁 `approve` 兼容处理；接受 DQ-CP2-CR068-01..05 推荐方案。CP2 仅批准范围基线，不单独授权远端执行；执行授权由 CP5 组合门禁限定。
