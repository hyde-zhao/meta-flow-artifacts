---
cr_id: "CR-121"
title: "Merge-Induced Directory Routing Regression Hardening"
cr_type: "process"
cr_kind: "requirement-change"
lifecycle_status: "superseded"
readiness_status: "n/a"
gate_status: "closed"
gate_profile: "standard"
status: "superseded"
impact_level: "medium"
workflow_mode_before: "production"
workflow_mode_after_change: "standard"
rollback_to: "FU-CR120-001 follow-up candidate"
source: "cp8-follow-up"
parent_cr: "CR-120"
source_checkpoint: "process/checkpoints/CP8-CR120-DELIVERY-READINESS.md"
source_follow_up_id: "FU-CR120-001"
follow_up_type: "merge-induced-directory-routing-regression-hardening"
risk_class: "medium-repository-governance-git-index-cleanup-authorized-not-executed"
owner: "host-orchestrator"
created_at: "2026-06-23T09:08:54+08:00"
updated_at: "2026-06-23T12:43:37+08:00"
acceptance_criteria: "CP2 approves scope, audit baseline, staged authorization and no-runtime/no-index-cleanup boundary. Any .gitignore, Git index, checker, guardrail or file movement work requires a later CP5 authorization."
close_condition: "CP8 approved by user after scoped remediation and verification, or user rejects / cancels before CP5. Runtime/NAS/credentials/trading/provider/lake/catalog publish remain unauthorized unless separately gated."
cp2_approved_by: "user"
cp2_approved_at: "2026-06-23T09:16:46+08:00"
cp5_authorization_status: "approved"
cp5_authorization_checkpoint: "process/checkpoints/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-REVIEW.md"
cp5_approved_by: "user"
cp5_approved_at: "2026-06-23T09:30:34+08:00"
deferred_by_cr: "CR-125"
deferred_reason: "CR125 CP5 approved at 2026-06-23T12:43:37+08:00. CR121 CP6 index-only remediation is not resumed; remaining directory routing cleanup is superseded by CR125 runner development readiness stop-line. No git rm --cached, .gitignore, source/checker/tests, delete/move, commit/push or runtime work executed under CR121."
---

# CR-121 Merge-Induced Directory Routing Regression Hardening

## 变更来源

用户在 CR120 CP8 approve 同时要求检测 `/home/hyde/workspace/quant-lab/docs/design`、`/home/hyde/workspace/quant-lab/docs/features` 以及相邻目录是否因 2026-06-22 分支合并引入目录结论混乱，并结合当前需要整改的 CR 给出后续整改计划。该问题已在 CR120 后续台账中登记为 `FU-CR120-001`。

用户随后明确启动 `FU-CR120-001 Merge-Induced Directory Routing Regression Hardening`，并限定本轮先做 CP2 范围确认：不改代码、不执行 `git rm --cached`、不删除文件、不启动 runtime。

## 当前授权边界

| 范围 | CP2 当前授权 | 说明 |
|---|---:|---|
| process artifacts 读取 / 写入 / 更新 | true | 仅用于 CR121 变更单、context、precheck、checkpoint、CR index、STATE 和 source follow-up tracking。 |
| repository routing read-only audit | true | 可读取目录类型、symlink 目标、Git tracking / ignored 状态和 merge 证据摘要。 |
| source code modify | false | 不修改源码、checker、tests 或脚本。 |
| `.gitignore` modify | false | CP2 不修改 ignore 规则。 |
| Git index cleanup | false | 不执行 `git rm --cached`、`git add`、commit、checkout、reset、clean 或任何 destructive Git 操作。 |
| file delete / move / overwrite | false | 不删除、不移动、不覆盖 `docs/`、`process`、`checkpoints`、`.claude` 或其他文件。 |
| runtime / NAS / credentials / trading / provider / lake / catalog / publish | false | 本 CR 当前完全不授权。 |

## 五维度影响分析

| 维度 | 影响 | 结论 |
|---|---|---|
| 需求层 | true | 需要明确源码仓库公开文档面、artifact-routed 内部文档面、ignored-but-tracked 路径和 merge 回流路径的治理边界。 |
| 场景层 | true | 覆盖 merge 后 symlink 仍存在但 Git 仍跟踪 ignored 路径、旧文档目录回流、`.claude/**` / `AGENTS.md` 等忽略规则与索引状态不一致的场景。 |
| 计划层 | true | `FU-CR120-001` 从 candidate 转为正式 CR121；本轮仅 CP2 范围确认，后续 CP5 才能授权具体整改。 |
| 安全 / 权限层 | true | Git index cleanup、文件删除/移动、远端写、runtime、NAS、凭据、交易和 publish 都需要明确禁止或单独门禁。 |
| 交付层 | true | 后续若通过 CP5，可能影响源码仓库清洁度、artifact 路由验证、merge guardrail / checker 策略和发布前自检。 |

## CP2 初始检查对象

| 对象 | 纳入原因 | CP2 处理 |
|---|---|---|
| `docs/design` | 已整改为 artifact-routed symlink，需防 merge 结论混乱 | 只确认范围；不改 link / index。 |
| `docs/features` | 已整改为 artifact-routed symlink，需防 merge 结论混乱 | 只确认范围；不改 link / index。 |
| `docs/quality` | 同属内部 artifact-routed 文档面 | 只确认范围。 |
| `process` | 运行态 ledger symlink，Git tracking / merge 回流风险高 | 只确认范围。 |
| `checkpoints` | legacy fallback / artifact-routed 人工检查点入口 | 只确认范围。 |
| `.claude/**` | `.gitignore` 已忽略但当前仍有 tracked 路径风险 | 只确认范围。 |
| `AGENTS.md` | 当前规则忽略与跟踪状态存在治理争议 | 只确认范围。 |
| `docs/qmt/**` | merge 带回旧式文档候选，需要判定是否留在源码公开面 | 只确认范围。 |
| `docs/goldminer/**` | merge 带回旧式文档候选，需要判定是否留在源码公开面 | 只确认范围。 |
| `docs/release/*.md` | 部分 release 文档为源码公开面，部分内部审查文档可能回流 | 只确认范围。 |
| `.gitignore` | 可能需要后续 hardening，但 CP2 不改 | 只确认 owner / 范围。 |
| guardrail / checker scripts | 可能需要后续静态检查，但 CP2 不改源码 | 只确认候选 owner。 |

## 文档处理决策

| 对象 | 决策 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| `process/changes/CR-120-FOLLOW-UP-TRACKING-2026-06-23.md` | 原文档更新 | 保留 `FU-CR120-001` 原候选行并改为 active，链接 CR121 | CR120 后续台账是本 CR 直接来源。 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR120 closure 和 FU row，新增 CR121 active | 当前 CR tracking 机器索引。 |
| `process/STATE.md` | 原文档更新 | 保留历史 audit sections，更新当前 active gate 和待决策队列 | 只同步当前态。 |
| `process/context/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-CONTEXT.yaml` | 新增 | N/A | CP2 默认上下文胶囊。 |
| `process/checks/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-PRECHECK.md` | 新增 | N/A | CP2 自动预检。 |
| `process/checkpoints/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-REVIEW.md` | 新增 | N/A | CP2 人工审查稿。 |

## 待人工决策项

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR121-01 | scope | 是否接受将 `FU-CR120-001` 升格为 CR121，并以 merge-induced directory routing regression hardening 为独立范围？ | 接受 CR121 作为独立 standard CR，CP2 只确认范围和候选清单。 | A. 仅保留 follow-up candidate；B. 拆成多个 CR；C. reject 并取消候选。 | 推荐方案保留追溯并避免目录治理问题被 CR120 关闭状态吞掉；拆分会降低本轮收敛速度。 |
| DQ-CP2-CR121-02 | implementation | 是否接受分阶段路线：CP2 仅做 read-only audit / scope freeze，所有 `.gitignore`、Git index、checker、guardrail 或 `git rm --cached` 等整改必须等 CP5？ | 接受 staged route，CP2 不改代码、不改 `.gitignore`、不清理 index。 | A. design-only 不进入 CP5；B. 直接进入 CP5；C. 拆成 audit CR 和 cleanup CR。 | 推荐方案权限边界最清楚；直接 CP5 容易混淆授权；拆分成本更高。 |
| DQ-CP2-CR121-03 | scope | 是否冻结初始检查对象全集？ | 接受对象全集：`docs/design`、`docs/features`、`docs/quality`、`process`、`checkpoints`、`.claude/**`、`AGENTS.md`、`docs/qmt/**`、`docs/goldminer/**`、`docs/release/*.md`、`.gitignore`、guardrail/checker scripts；CP5 再裁剪整改子集。 | A. 只处理 `docs/design` / `docs/features`；B. 排除 `.claude/**` / `AGENTS.md`；C. 只处理 ignored-but-tracked 路径。 | 推荐方案避免漏掉 merge 回流路径；范围较宽但 CP2 不执行整改。 |
| DQ-CP2-CR121-04 | runtime_authorization | 是否继续禁止 runtime/NAS/credentials/trading/provider/lake/catalog/publish，并禁止 `git rm --cached`、删除、移动、reset/checkout、远端写？ | 继续禁止；只允许本地静态检查和 process 文件更新。 | A. 暂停所有验证命令；B. 另起 git index cleanup CP5；C. 另起 runtime gate。 | 推荐方案足够支撑 CP2；任何真实清理、外部访问或运行都必须单独授权。 |

## 不授权范围

- 不改源码、tests、checker、guardrail 或 `.gitignore`。
- 不执行 `git rm --cached`、`git add`、commit、checkout、reset、clean、remote write、push、branch/default branch/tag/history 操作。
- 不删除、移动、覆盖、重建或替换 `docs/design`、`docs/features`、`docs/quality`、`process`、`checkpoints`、`.claude/**`、`AGENTS.md`、`docs/qmt/**`、`docs/goldminer/**` 或 `docs/release/*.md`。
- 不启动 runtime，不访问 NAS，不读取凭据 / env / token / account / funds / positions / orders / fills / raw logs。
- 不执行 submit / cancel / buy / sell、simulation / live、provider fetch、lake write、catalog publish。

## 当前结论

CR121 已由 CR125 接管并关闭为 `closed-superseded-by-cr125`。原 CP5 授权的 index-only remediation 不再自动恢复；进入 runner development 前只保留 CR125 记录的最小 readiness 结论。后续若真实需要 `git rm --cached`、`.gitignore`、checker/source/tests 或更广目录治理，必须另起独立 CR。

本轮未执行 Git index cleanup、未修改 `.gitignore`、未删除 / 移动 CR121 范围文件、未修改源码 / checker / tests、未 commit/push、未启动 runtime 或外部系统访问。

## CP5 Remediation Authorization 草案

本节记录 2026-06-23T09:20:34+08:00 发起的 CP5 授权门禁准备。用户于 2026-06-23T09:30:34+08:00 回复 `approve`，接受 `DQ-CP5-CR121-01..04` 推荐方案。该批准只授权后续 CP6 最小 `index-only untrack` remediation 子集；本次回填未执行 `.gitignore` 修改、Git index cleanup、`git rm --cached`、删除 / 移动、代码 / checker 修改或 runtime。

### 允许后续整改的候选子集

| 路径 / 对象 | CP5 推荐分类 | 后续 CP6 允许动作 | CP5 前状态 |
|---|---|---|---|
| `docs/design` | allowed-remediation | index-only untrack，保留本地 symlink 和 artifact 目标内容 | 已授权后续 CP6；尚未执行 |
| `docs/features` | allowed-remediation | index-only untrack，保留本地 symlink 和 artifact 目标内容 | 已授权后续 CP6；尚未执行 |
| `docs/quality` | allowed-remediation | index-only untrack，保留本地 symlink 和 artifact 目标内容 | 已授权后续 CP6；尚未执行 |
| `process` | allowed-remediation | index-only untrack，保留外置 process symlink 和 artifact process | 已授权后续 CP6；尚未执行 |
| `checkpoints` | allowed-remediation | index-only untrack，保留本地 symlink 和 artifact checkpoint 内容 | 已授权后续 CP6；尚未执行 |
| `.claude/**` | allowed-remediation | index-only untrack，不删除工作区文件 | 已授权后续 CP6；尚未执行 |

### 仅审计对象

| 路径 / 对象 | CP5 推荐分类 | 本 CR 处理方式 | 后续需要 |
|---|---|---|---|
| `AGENTS.md` | audit-only | 仅记录 tracked / ignored 冲突，不 untrack、不删除、不改写 | 如需改变跟踪策略，另起 CR |
| `docs/qmt/**` | audit-only | 仅记录 merge 回流风险，不做内容治理 | 如需公开文档 / 内部归档判定，另起内容治理 CR |
| `docs/goldminer/**` | audit-only | 仅记录 merge 回流风险，不做内容治理 | 如需公开文档 / 内部归档判定，另起内容治理 CR |
| `docs/release/*.md` | audit-only | 保留公开 release 入口，不在本 CR 迁移或删除 | 如需拆分 release 公开 / 内部边界，另起 CR |
| `.gitignore` | audit-only | 本 CR 不修改 ignore 规则 | 如需 hardening，另起 CP5 实现门禁 |
| guardrail / checker scripts | audit-only | 本 CR 不改源码、tests、checker | 如需 checker hardening，另起实现 CR |

### 冻结禁止项

即使 CP5 后续获得 approve，也仍不授权以下动作：

- 删除、移动、覆盖、重建或替换任何文件 / symlink / artifact 目标内容。
- 修改 `.gitignore`。
- 修改源码、tests、checker、guardrail 或脚本实现。
- 执行 `git add`、commit、push、remote write、branch、tag、history、checkout、reset、clean。
- 启动 runtime，访问 NAS、凭据、交易账户、provider、lake、catalog 或 publish。

### 人工确认结果

CP5 人工审查稿路径：`process/checkpoints/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-REVIEW.md`。

已批准决策：`DQ-CP5-CR121-01`、`DQ-CP5-CR121-02`、`DQ-CP5-CR121-03`、`DQ-CP5-CR121-04`。

下一步建议：启动 CR121 CP6 最小 index-only remediation 执行记录；只允许对 `docs/design`、`docs/features`、`docs/quality`、`process`、`checkpoints`、`.claude/**` 执行 index-only untrack 和后续静态验证。仍不授权删除、移动、覆盖、`.gitignore` 修改、源码 / tests / checker 修改、`git add`、commit、push、remote write、checkout、reset、clean、runtime、NAS、凭据、交易、provider、lake、catalog 或 publish。
