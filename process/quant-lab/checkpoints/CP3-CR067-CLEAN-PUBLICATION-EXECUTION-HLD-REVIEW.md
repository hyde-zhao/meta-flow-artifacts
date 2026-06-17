---
checkpoint_id: CP3-CR067-CLEAN-PUBLICATION-EXECUTION-HLD-REVIEW
change_id: CR-067
checkpoint_type: manual
status: approved
created_at: '2026-06-15T12:01:19+08:00'
auto_precheck: process/checks/CP3-CR067-CLEAN-PUBLICATION-EXECUTION-HLD-CONSISTENCY.md
context_capsule: process/context/CP3-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml
auto_final_authorization: false
---

# CP3 CR067 Clean Publication Execution HLD Review

## 自动预检摘要

- 自动预检结论：PASS_WITH_RISK
- 自动预检文件：`process/checks/CP3-CR067-CLEAN-PUBLICATION-EXECUTION-HLD-CONSISTENCY.md`
- 推荐架构：temporary clean snapshot + allowlist manifest + fail-closed scan + exact remote push policy。
- 本检查点只确认架构方向；在 CP2/CP3/CP5 全部批准前，不执行临时快照创建、扫描、Git 初始化、remote add 或 push。

## Decision Brief

推荐决策：批准 CR067 的高层架构采用“临时 clean snapshot 发布”而非“当前分支发布”。该架构把发布内容从当前 Git history 中隔离出来，先根据 allowlist 构建临时目录，再通过 secret/path/large/topology/cache 扫描后，才允许进入受控 Git publication sequence。

备选方案：直接发布当前分支；在当前仓库上做历史清理；只发布 minimal code-only；把 GitHub publication 推迟到 NAS/data lake/runtime 之后。推荐方案优先，因为 current branch 已有 forbidden tracked path blocker，而 history rewrite、force push 或大规模清理均超出当前授权。

影响维度：包导入形态、legacy root 保留、发布内容完整性、扫描失败处理、remote conflict policy、后续回滚/forward-fix。

风险与回退：如果架构不通过，CR067 不进入 CP5；如果用户要求改为 history rewrite 或 branch rename，必须另起高风险 CR 并重新审批。

### Context Capsule Summary

- CP3 capsule：`process/context/CP3-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml`
- 关键事实：`pyproject.toml` 中 `[tool.uv] package=false`，当前 `quant_lab` 兼容命名空间仍桥接 `engine`、`market_data`、`strategies`、`trading` 等 legacy roots；clean snapshot 不能只发布 `quant_lab/**`。
- 读取策略：compact；如需审计完整 CR066 strategy，可读取 `docs/release/CLEAN-PUBLICATION-STRATEGY-CR066.md`。

### Decision Collection Coverage

- 已扫描来源：`pyproject.toml`、`docs/release/CR061-BATCH-B-CANDIDATE-MANIFEST.yaml`、`docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md`、`docs/release/CLEAN-PUBLICATION-EXECUTION-PLAN-CR067.md`、`docs/release/CLEAN-PUBLICATION-EXECUTION-MANIFEST-CR067.yaml`、`docs/release/CLEAN-PUBLICATION-SCAN-GATE-CR067.md`、CP3 context capsule。
- 候选问题数：8
- 纳入待决策数：5
- N/A / 未纳入原因：具体远端 URL 已继承 CR062 candidate，但实际 remote add/push 授权留给 CP5；tag/default branch rename/history rewrite 被列为不授权项。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR067-01 | architecture | 是否采用 temporary clean snapshot + exact remote push 架构？ | 采用临时 clean snapshot，扫描通过后再按 exact remote policy 推送。 | 当前分支发布；history rewrite 后发布；code-only 发布。 | 推荐方案隔离污染且可审计；备选要么高风险，要么证据不足。 | 需要临时目录和 manifest freeze。 | snapshot 无法通过扫描时 blocked。 |
| DQ-CP3-CR067-02 | implementation | 是否保留 legacy roots 作为发布必需内容？ | 保留 `engine/**`、`market_data/**`、`strategies/**`、`trading/**` 与 `quant_lab/**`。 | 只发布 `quant_lab/**`；立即 bulk import rewrite。 | 推荐方案维持现有 import 契约；备选可能破坏运行或越权重写。 | 发布包仍保留 legacy alias 语义。 | 未来另起 CR 做 bulk rewrite。 |
| DQ-CP3-CR067-03 | security | scan gate 是否必须在任何 Git 写动作前 PASS？ | 必须；scan 任一 BLOCKING/HIGH 即停止。 | Git init/commit 后再 scan；只做 warning。 | 推荐方案 fail-closed；备选可能产生污染提交。 | 执行会多一轮扫描证据。 | scan fail 时不执行 Git 写动作。 |
| DQ-CP3-CR067-04 | implementation | remote conflict policy 是否采用 fail-safe？ | 采用 exact remote candidate，不做 `remote set-url`、force push、remote deletion；冲突即停止。 | 自动覆盖 remote；force push；删除重建 remote。 | 推荐方案最小外部变更；备选破坏远端状态或越权。 | 可能因远端冲突 blocked。 | 用户另批 remote remediation CR。 |
| DQ-CP3-CR067-05 | risk_acceptance | 是否接受 no history rewrite / no force / no tag / no remote deletion 边界？ | 接受，CR067 只允许 forward-fix。 | 允许 rewrite；允许 force push；允许 tag。 | 推荐方案降低破坏性；备选需要更高风险授权。 | 首次 publication 可能需要多轮修正。 | 用户明确请求时新建高风险 CR。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项架构推荐；不表示授权以下操作：

- 不授权任何 Git 写动作或远端变更，直到 CP2/CP3/CP5 全部批准。
- 不授权 history rewrite、force push、tag、branch creation / rename、remote deletion 或 remote set-url。
- 不授权 NAS/data lake/provider/catalog/runtime/凭据操作。
- 不授权 physical move / rename / delete、bulk import rewrite 或 CR046 recovery。

## Entry Criteria

- CP2 需求基线文件已准备，等待用户统一批准。
- CR067 execution plan、manifest 和 scan gate 文档已创建。
- CP3 context capsule 已创建。
- 当前 HLD 方向不依赖外部网络或 Git 远端写入。

## Checklist

- [x] 推荐架构已明确。
- [x] legacy roots 发布必要性已说明。
- [x] scan gate 与 Git write 顺序已约束。
- [x] remote conflict policy 已 fail-safe 化。
- [x] no rewrite / no force / no tag / no remote deletion 边界已列出。
- [x] 用户完成人工审查并回复 `approve` / `修改: <具体修改点>` / `reject`。

## Exit Criteria

- 用户回复 `approve`：CP3 架构通过，可与 CP5 一并作为 post-approval execution 前置；仍不单独授权执行。
- 用户回复 `修改: <具体修改点>`：更新架构和决策表后重新发起。
- 用户回复 `reject`：CR067 停止在架构门禁。

## Deliverables

- `docs/release/CLEAN-PUBLICATION-EXECUTION-PLAN-CR067.md`
- `docs/release/CLEAN-PUBLICATION-EXECUTION-MANIFEST-CR067.yaml`
- `docs/release/CLEAN-PUBLICATION-SCAN-GATE-CR067.md`
- `process/context/CP3-CR067-CLEAN-PUBLICATION-EXECUTION-CONTEXT.yaml`
- `process/checkpoints/CP3-CR067-CLEAN-PUBLICATION-EXECUTION-HLD-REVIEW.md`

## 人工审查结果

- status: approved
- reviewed_by: user
- reviewed_at: '2026-06-15T12:45:02+08:00'
- decision: approve
- notes: 用户回复“同意”，按 CR067 CP2/CP3/CP5 组合门禁 approve 处理；接受 DQ-CP3-CR067-01..05 推荐方案。CP3 批准不授权 history rewrite、force push、tag、branch rename、remote deletion 或 remote set-url。
