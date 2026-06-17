---
checkpoint_id: CP2-CR067-CLEAN-PUBLICATION-EXECUTION-BASELINE
change_id: CR-067
checkpoint_type: manual
status: approved
created_at: '2026-06-15T12:01:19+08:00'
auto_precheck: process/checks/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-BASELINE.md
context_capsule: process/context/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml
auto_final_authorization: false
---

# CP2 CR067 Clean Publication Execution Baseline

## 自动预检摘要

- 自动预检结论：PASS_WITH_RISK
- 自动预检文件：`process/checks/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-BASELINE.md`
- 本检查点只确认 CR067 的需求基线与授权边界；不执行 Git 写动作、远端操作、凭据读取、数据迁移或运行时连接。
- 风险来源：CR062 post-approval preflight 已发现 current branch 追踪 `reports/**`、`runs/**` 和 `.env.example`，因此 CR067 不允许发布当前分支原样状态，必须走 clean publication execution gate。

## Decision Brief

推荐决策：批准 CR067 作为 CR066 后续的 Clean Publication Execution Gate，范围限定为“干净发布执行门禁”，先冻结需求基线、边界和后续 CP3/CP5 审查输入；在 CP2/CP3/CP5 全部通过前，不授权任何 Git 写动作或外部操作。

备选方案：保持 CR067 为 follow-up candidate；回到 CR062 直接发布 current branch；改为先处理 NAS/data lake/runtime；恢复 CR046。推荐方案优先，因为它能把 current index 污染与 GitHub publication 解耦，且不扩大到数据、运行时或交易权限面。

影响维度：范围、Git publication 安全、证据可追溯、CR062 blocked 状态、CR066 strategy 关闭状态、CR046 paused 状态、后续 CR 排序。

风险与回退：如果用户不批准 CP2，CR067 保持 candidate，不进入 CP3/CP5；如果后续发现范围需要包含 NAS/data lake/runtime/CR046 recovery，必须新建或修订 CR，不得在 CR067 内静默扩大。

### Context Capsule Summary

- CP2 capsule：`process/context/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml`
- 读取策略：compact；默认不读取完整历史文档，除非审计、冲突或用户要求。
- 关键事实：CR066 已 READY_WITH_RISK 关闭；CR062 仍 blocked；CR046 仍 user-paused；CR067 只允许在门禁批准后进入受控 post-approval execution。

### Decision Collection Coverage

- 已扫描来源：`process/STATE.md`、`process/changes/CR-INDEX.yaml`、`process/checkpoints/CP8-CR066-DELIVERY-READINESS.md`、`process/checks/CR062-POST-APPROVAL-PREFLIGHT.md`、`docs/release/CLEAN-PUBLICATION-STRATEGY-CR066.md`、`process/changes/CR-067-CLEAN-PUBLICATION-EXECUTION-GATE-2026-06-15.md`、CP2 context capsule。
- 候选问题数：7
- 纳入待决策数：5
- N/A / 未纳入原因：NAS/data lake/runtime/CR046 recovery 均超出 CR067，列为不授权项；Git push 细节留给 CP5 runtime_authorization。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR067-01 | scope | 是否正式启动 CR067，承接 CR066 的后续执行门禁？ | 启动 CR067，并登记为 active formal CR；CR062 保持 blocked，CR046 保持 paused。 | 保持为 candidate；回到 CR062；恢复 CR046。 | 推荐方案承接 CR066 且不扩大权限；备选会停滞或绕过已发现 blocker。 | 新增一轮 CP2/CP3/CP5，但审计边界清晰。 | 用户 reject 时 CR067 保持 candidate。 |
| DQ-CP2-CR067-02 | scope | CR067 范围是否限定为 clean publication execution gate？ | 限定为 manifest freeze、scan gate、临时 clean snapshot、remote push policy 和 evidence；不含 NAS/lake/runtime/CR046。 | 合并 NAS/lake；合并 runtime；恢复 CR046。 | 推荐方案隔离 GitHub publication；备选扩大权限和回滚面。 | 后续迁移仍需独立 CR。 | 用户要求合并时重新做 CR 影响分析。 |
| DQ-CP2-CR067-03 | security | 是否接受 allowlist/default-exclude/conditional include policy？ | 接受；`.env.example` 仅在 placeholder/no-secret scan 通过后条件纳入。 | 发布 current branch；code-only；排除全部过程证据。 | 推荐方案兼顾安全与可追溯；current branch 风险高；code-only 证据不足。 | 需要额外 manifest 和扫描。 | scan 出现 BLOCKING/HIGH 时 fail-closed。 |
| DQ-CP2-CR067-04 | risk_acceptance | 是否确认不发布 current branch as-is？ | 确认；必须使用临时 clean snapshot。 | 直接 push 当前分支；尝试当前分支清理后发布。 | 推荐方案避免 tracked history 污染；备选可能泄漏过程/数据路径。 | 需要构建临时快照并记录来源。 | 若 clean snapshot 无法构建，CR067 blocked。 |
| DQ-CP2-CR067-05 | follow_up_tracking | 如果后续执行失败，是否保持 fail-closed/forward-fix？ | 是；失败时记录 blocker，forward-fix，不回退到 CR062 as-is。 | 退回 CR062 current branch publication；跳过失败扫描。 | 推荐方案维持安全门禁；备选绕过已知风险。 | 可能延迟 GitHub publication。 | 用户另行批准新 CR 或修订 CR067。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权 `git add`、`git commit`、`git remote add`、`git remote set-url`、`git push`、`git tag`、branch creation / rename、history rewrite、force push 或 remote deletion。
- 不授权 NAS/data lake/provider/catalog/runtime/凭据操作，不读取 `.env`、token、账号、密码、cookie、session、private key 或交易事实。
- 不授权 physical move / rename / delete，不授权 bulk import rewrite。
- 不授权恢复 CR046，不授权 QMT/MiniQMT runtime、交易、账户查询或仿真/真实下单。

## Entry Criteria

- CR066 已 closed-current-delivery / READY_WITH_RISK。
- CR062 post-approval preflight blocker 已记录。
- 用户已说“好的继续推进”，表示可以启动下一步正式 CR 材料。
- CR067 尚未获得 CP2/CP3/CP5 人工批准。

## Checklist

- [x] CR067 正式 CR 已创建。
- [x] CP2 context capsule 已创建。
- [x] CP2 自动预检已完成并标记 PASS_WITH_RISK。
- [x] 决策项已收集并列入本文件。
- [x] 不授权项已显式列出。
- [x] 用户完成人工审查并回复 `approve` / `修改: <具体修改点>` / `reject`。

## Exit Criteria

- 用户回复 `approve`：CP2 需求基线通过，进入 CP3/CP5 组合门禁确认；仍不单独授权 Git 写动作，必须等待 CP2/CP3/CP5 全部通过。
- 用户回复 `修改: <具体修改点>`：更新 CR067 范围和 Decision Brief 后重新发起。
- 用户回复 `reject`：CR067 保持未批准，不执行后续动作。

## Deliverables

- `process/changes/CR-067-CLEAN-PUBLICATION-EXECUTION-GATE-2026-06-15.md`
- `process/context/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml`
- `process/checks/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-BASELINE.md`
- `process/checkpoints/CP2-CR067-CLEAN-PUBLICATION-EXECUTION-BASELINE.md`

## 人工审查结果

- status: approved
- reviewed_by: user
- reviewed_at: '2026-06-15T12:45:02+08:00'
- decision: approve
- notes: 用户回复“同意”，按 CR067 CP2/CP3/CP5 组合门禁 approve 处理；接受 DQ-CP2-CR067-01..05 推荐方案。CP2 批准不单独授权 forbidden operations；后续执行仍受 CP5 不授权项约束。
