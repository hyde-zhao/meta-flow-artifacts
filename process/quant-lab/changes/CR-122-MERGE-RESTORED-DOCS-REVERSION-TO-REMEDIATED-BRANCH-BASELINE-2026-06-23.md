---
cr_id: "CR-122"
title: "Merge-Restored Docs Reversion to Remediated Branch Baseline"
cr_type: "process"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "production"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR122 state; no docs content restored before CP5 approval"
source: "user-request"
parent_cr: "CR-121"
source_checkpoint: "process/checkpoints/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-REVIEW.md"
source_follow_up_id: ""
follow_up_type: "merge-restored-docs-content-reversion"
risk_class: "medium-docs-content-reversion-pending-authorization"
owner: "host-orchestrator"
created_at: "2026-06-23T09:44:04+08:00"
updated_at: "2026-06-23T10:22:20+08:00"
cp2_approved_by: "user"
cp2_approved_at: "2026-06-23T10:07:48+08:00"
cp2_approved_checkpoint: "process/checkpoints/CP2-CR122-MERGE-RESTORED-DOCS-REVERSION-REVIEW.md"
cp2_approval_input: "批准，按照你推荐的方案执行。"
cp5_authorization_status: "approved"
cp5_authorization_checkpoint: "process/checkpoints/CP5-CR122-MERGE-RESTORED-DOCS-REVERSION-AUTHORIZATION-REVIEW.md"
cp5_approved_by: "user"
cp5_approved_at: "2026-06-23T10:15:47+08:00"
cp5_approval_input: "approve"
cp6_status: "PASS"
cp6_check: "process/checks/CP6-CR122-MERGE-RESTORED-DOCS-REVERSION-CODING-DONE.md"
cp7_status: "PASS"
cp7_check: "process/checks/CP7-CR122-MERGE-RESTORED-DOCS-REVERSION-VERIFICATION-DONE.md"
cp8_status: "approved"
cp8_checkpoint: "process/checkpoints/CP8-CR122-DELIVERY-READINESS.md"
cp8_approved_by: "user"
cp8_approved_at: "2026-06-23T10:22:20+08:00"
cp8_approval_input: "approve"
acceptance_criteria: "CP2 confirms docs target list, precheck branch docs baseline, work branch CR102/103/104 validation-record/result preservation rule, later-CR baseline-doc modification preservation rule, CR121 scope separation and no-execution boundary. CP5 must approve before any docs content restore, git rm or git restore."
close_condition: "CP8 approved by user after scoped docs reversion and verification, or user rejects / cancels before CP5. Runtime/NAS/credentials/trading/provider/lake/catalog publish remain unauthorized."
---

# CR-122 Merge-Restored Docs Reversion to Remediated Branch Baseline

## 变更来源

用户要求启动 CR122：把 `234b3d3` merge 从主线带回的 `docs` 文档全部恢复到本分支第一父提交 `234b3d3^1` 的整改后状态；新增文档恢复为不存在，`docs/USER-MANUAL.md` 恢复为 `234b3d3^1` 版本。

用户明确限定本轮先做 CP2 范围确认和冲突预检：不执行删除、不执行 `git rm`、不修改 `.gitignore`、不改源码 / checker / tests、不提交 / 推送、不启动 runtime。

用户随后修正基线规则：`precheck/ql-rd-000-redesign-baseline` 这个分支的 `docs` 为基线；`work/chapter3-factor-gap-remediation-20260608` 这个分支仅保留 `CR102`、`CR103` 和 `CR104` 的验证记录和验证结果；同时需要考虑分支合并后后续 CR 对原基线文档的修改，若存在修改必须保持修改后的内容。

## 当前授权边界

| 范围 | CP2 当前授权 | 说明 |
|---|---:|---|
| process artifacts 读取 / 写入 / 更新 | true | 仅用于 CR122 变更单、context、precheck、checkpoint、CR index 和 STATE。 |
| repository read-only audit | true | 可读取 merge diff、Git tracking 状态和 CR121 既有授权边界。 |
| docs content restore | false | CP2 不恢复 `docs` 内容，不执行 `git restore`。 |
| docs file deletion / git rm | false | CP2 不删除文件，不执行 `git rm`。 |
| `.gitignore` modify | false | 本 CR CP2 不修改 ignore 规则。 |
| source / checker / tests modify | false | 不修改源码、checker、tests 或脚本。 |
| git add / commit / push / remote write | false | 不提交、不推送、不写远端。 |
| runtime / NAS / credentials / trading / provider / lake / catalog / publish | false | 本 CR 完全不授权。 |

## 五维度影响分析

| 维度 | 影响 | 结论 |
|---|---|---|
| 需求层 | true | 明确 merge-restored docs 的恢复目标是 `precheck/ql-rd-000-redesign-baseline` 的 pre-merge docs 基线，并叠加 CR102/103/104 验证证据保留例外与后续合法 CR 修改保留规则，而不是当前 merge 结果或主线版本。 |
| 场景层 | true | 覆盖新增 docs 路径恢复为不存在、`docs/USER-MANUAL.md` 恢复 precheck 基线版本、空目录处理、验证证据保留例外和与 CR121 路由治理分层。 |
| 计划层 | true | CR122 为独立 standard CR；CP2 仅确认范围，CP5 才能授权实际 docs 内容恢复。 |
| 安全 / 权限层 | true | `git rm`、删除、`git restore`、提交、推送、runtime 和外部系统动作都必须显式门禁。 |
| 交付层 | true | 后续若执行，会改变源码仓库 `docs` 内容面，使其回到本分支整改后状态；需要 CP7 验证与 CP8 关闭。 |

## 冲突预检

| 冲突对象 | 状态 | 处理结论 |
|---|---|---|
| CR121 | active-cp5-approved-cp6-authorized | 存在重叠影响面，但可分层：CR121 保持 index-only 路由治理；CR122 接管 docs 内容恢复。不得并行执行两个 cleanup。 |
| `docs/design` / `docs/features` / `docs/quality` | symlink / artifact-routed | 不进入 CR122；仍由 CR121 index-only cleanup 处理。 |
| `.claude/**` | ignored-but-tracked | 不进入 CR122；仍由 CR121 或独立平台产物治理处理。 |
| `docs/qmt/**` / `docs/goldminer/**` / top-level QMT docs | merge-restored docs content | 默认进入 CR122 docs 内容恢复范围；仅 CR102/103/104 的验证记录 / 验证结果可作为保留例外，设计 / HLD / runbook 默认不保留。 |
| `docs/release/*.md` | merge-restored public release entry candidates | 按用户目标进入 CR122 恢复范围；若用户要保留，必须修改 CP2 决策。 |

## 目标恢复基线

| 项 | 值 |
|---|---|
| merge commit | `234b3d3` |
| 基线分支 | `precheck/ql-rd-000-redesign-baseline` |
| pre-merge docs 基线提交 | `234b3d3^1` = `47a5eee6ad601ee13e1a193eb86116430ab498bc` |
| work 分支候选来源 | `work/chapter3-factor-gap-remediation-20260608` |
| work 分支保留策略 | 仅保留 `CR102`、`CR103`、`CR104` 的验证记录和验证结果；设计、HLD、runbook、计划类文档默认不保留。 |
| 后续 CR 修改保留策略 | 若 merge 后后续 CR 修改过原基线 docs，CP5/CP6 前必须重算并保留修改后的内容。当前只读检查 `git log 234b3d3..HEAD -- docs` 无提交。 |
| 冲突处理策略 | docs 内容以 precheck docs 基线 + 已确认验证证据保留例外 + 后续合法 CR 修改为准 |
| CP2 执行策略 | 只确认范围和冲突预检，不做恢复 |
| CP5 后可授权策略 | 仅对本 CR 列出的 docs 目标执行内容恢复 |

## 分支差异与保留例外预检

| 检查项 | 只读证据 | 结论 |
|---|---|---|
| 当前分支 | `git rev-parse --abbrev-ref HEAD` | `precheck/ql-rd-000-redesign-baseline` |
| merge commit | `git show --no-patch 234b3d3` | `Merge master into precheck redesign baseline`；第一父为 `47a5eee6...`，第二父为 `13b8ebc...`。 |
| work 分支 | `git branch --list --all ...` | 本地和 origin 均存在 `work/chapter3-factor-gap-remediation-20260608`。 |
| work 分支 docs 中 CR102/103/104 匹配 | `git ls-tree -r --name-only work/chapter3-factor-gap-remediation-20260608 -- docs` + CR102/103/104 搜索 | 仅发现 `docs/qmt/CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-HLD.md`，它是 HLD，不是验证记录 / 验证结果；未发现 CR103/CR104 命名 docs。 |
| work 分支 CR104 匹配 | 全仓库只读搜索 | 发现 `scripts/build_cr104_runtime_preflight_evidence.py`、`scripts/run_cr104_qmt_miniqmt_validation.ps1`、`tests/test_cr104_runtime_validation_scripts.py`，均非 docs，且本 CR 不改源码/tests/scripts。 |
| merge 后后续 docs 修改 | `git log 234b3d3..HEAD -- docs` | 当前无提交；CP5/CP6 前仍需重跑，防止覆盖后续合法修改。 |

## 目标 docs 清单

### 新增文档默认恢复为不存在

以下为 `234b3d3^1..234b3d3` 带入的 docs 新增路径。默认恢复为不存在；只有被 CP2/CP5 明确分类为 `CR102/CR103/CR104 validation record/result` 的路径才可从 work 分支保留。当前只读预检未在 tracked docs 中发现 CR103/CR104 验证记录 / 结果，且 `docs/qmt/CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-HLD.md` 默认仍属于 HLD，不属于验证结果。

| 路径 | 恢复目标 |
|---|---|
| `docs/CR025-BACKTRADER-MODULE-REFERENCE.md` | absent in `234b3d3^1` |
| `docs/CR030-FACTOR-RESEARCH-QUICKSTART.md` | absent in `234b3d3^1` |
| `docs/QMT-C-S-BRIDGE-RUNBOOK.md` | absent in `234b3d3^1` |
| `docs/QMT-GATEWAY-INSTALL.md` | absent in `234b3d3^1` |
| `docs/QMT-INCIDENT-PLAYBOOK.md` | absent in `234b3d3^1` |
| `docs/QMT-SIMULATION-LIVE-RUNBOOK.md` | absent in `234b3d3^1` |
| `docs/QMT-TRADING-RUNBOOK.md` | absent in `234b3d3^1` |
| `docs/goldminer/CR045-BRIDGE-RUNBOOK.md` | absent in `234b3d3^1` |
| `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | absent in `234b3d3^1` |
| `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md` | absent in `234b3d3^1` |
| `docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md` | absent in `234b3d3^1` |
| `docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md` | absent in `234b3d3^1` |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml` | absent in `234b3d3^1` |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-HLD.md` | absent in `234b3d3^1` |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-MANUAL-GUIDE.md` | absent in `234b3d3^1` |
| `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-TEST-PLAN.md` | absent in `234b3d3^1` |
| `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md` | absent in `234b3d3^1` |
| `docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md` | absent in `234b3d3^1` |
| `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-HLD.md` | absent in `234b3d3^1` |
| `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-RECOVERY-RUNBOOK.md` | absent in `234b3d3^1` |
| `docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md` | absent in `234b3d3^1` |
| `docs/qmt/CR101-VALIDATION-AND-FOLLOW-UP-GATES.md` | absent in `234b3d3^1` |
| `docs/qmt/CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-HLD.md` | absent in `234b3d3^1` |
| `docs/release/DEPLOY-CHECKLIST.md` | absent in `234b3d3^1` |
| `docs/release/FEEDBACK.md` | absent in `234b3d3^1` |
| `docs/release/MIGRATION.md` | absent in `234b3d3^1` |
| `docs/release/RELEASE-NOTES.md` | absent in `234b3d3^1` |
| `docs/release/ROLLBACK.md` | absent in `234b3d3^1` |

### 修改文档恢复 precheck 基线版本

| 路径 | 恢复目标 |
|---|---|
| `docs/USER-MANUAL.md` | restore from precheck baseline `234b3d3^1:docs/USER-MANUAL.md` unless a later legitimate CR modification is found before CP6 |

### 保留例外候选

| 类别 | 当前结论 | CP5/CP6 前动作 |
|---|---|---|
| CR102 验证记录 / 验证结果 | tracked docs 中未发现明确验证记录 / 结果；`CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-HLD.md` 默认不保留。 | 若用户指出具体验证结果路径，加入保留例外；否则按默认恢复为 absent。 |
| CR103 验证记录 / 验证结果 | tracked docs 中未发现命名路径。 | CP5 前重扫 work 分支和 process ledger；仅保留验证记录 / 结果。 |
| CR104 验证记录 / 验证结果 | tracked docs 中未发现命名路径；发现 scripts/tests，不属于 CR122 docs 范围。 | 不纳入 CR122 docs 内容恢复；如需保留 process 验证证据，另列 process evidence preservation，不改源码/tests。 |
| merge 后后续 CR 对原基线 docs 的修改 | 当前 Git 提交历史无 `234b3d3..HEAD -- docs` 修改。 | CP6 前重跑检查；若存在，保留后续 CR 修改后的内容，不机械覆盖。 |

## 文档处理决策

| 对象 | 决策 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| 28 个 merge-restored docs 新增路径 | 原文档更新 / 删除候选 | CR122 记录完整路径清单；Git 历史保留主线版本 | CP5 approve 后才可删除 / `git rm`。 |
| `docs/USER-MANUAL.md` | 原文档更新 | Git 历史保留 merge 版本；CR122 记录恢复基线 | CP5 approve 后才可按 precheck 基线恢复；若 CP6 前发现后续合法 CR 修改，保留修改后内容。 |
| `docs/design` / `docs/features` / `docs/quality` | 不变 | 由 CR121 路由治理处理 | CR122 不处理 symlink / index-only route。 |
| `.claude/**` | 不变 | 由 CR121 或后续平台产物治理处理 | CR122 不处理平台 agent / skill 产物。 |
| `.gitignore` / checker / source / tests | 不变 | N/A | 本 CR 不改代码或规则。 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR121 active 记录，新增 CR122 active | 当前 CR tracking 机器索引。 |
| `process/STATE.md` | 原文档更新 | 保留历史状态，更新当前 active gate 和待决策队列 | 只同步当前态。 |

## 待人工决策项

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR122-01 | scope | 是否接受 `precheck/ql-rd-000-redesign-baseline` 的 pre-merge docs 作为主基线？ | 接受：主基线为 `234b3d3^1` / `47a5eee6...` 上的 precheck docs，而不是 work 分支或当前 merge 结果。 | A. 以当前 merge 结果为准；B. 逐文件人工判定；C. 暂缓 CR122。 | 推荐方案符合用户修正后的目标且可机械验证；逐文件判定成本更高。 |
| DQ-CP2-CR122-02 | scope | 是否接受 CR122 仅处理 docs 内容恢复？ | 接受：CR122 只处理列出的 docs 新增 / 修改路径；不处理 `.claude/**`、`.gitignore`、checker、源码、tests。 | A. 同时纳入 `.claude/**`；B. 同时修改 `.gitignore`；C. 合并到 CR121。 | 推荐方案与 CR121 分层最清楚；扩大范围会混入平台产物或代码治理。 |
| DQ-CP2-CR122-03 | implementation | CP5 后允许的恢复动作如何冻结？ | 后续 CP5 approve 后，CP6 仅允许对目标 docs 路径执行内容恢复：新增文件默认恢复为不存在，`docs/USER-MANUAL.md` 恢复 precheck 基线版本；但必须保留已确认的 CR102/103/104 验证记录 / 验证结果和后续合法 CR 修改。 | A. 只恢复 `docs/qmt/**` / `docs/goldminer/**`；B. 保留 release 公开入口；C. design-only 不执行恢复。 | 推荐方案完整覆盖用户目标且保留验证证据例外；备选可能保留 merge 回流。 |
| DQ-CP2-CR122-04 | runtime_authorization | 是否继续禁止 runtime、NAS、凭据、交易、provider、lake、catalog、publish，以及提交 / 推送？ | 继续禁止；CP2 不执行任何恢复命令，CP5 前不允许删除、`git rm` 或 `git restore`。 | A. 暂停所有验证命令；B. CP5 后同时授权 commit；C. 拆成 docs 删除和 USER-MANUAL 恢复两个 CR。 | 推荐方案权限最小；commit/push 需独立发布门禁。 |

## 不授权范围

- CP2 不执行删除、`git rm`、`git restore`、移动、覆盖或 docs 内容恢复。
- 不修改 `.gitignore`。
- 不修改源码、tests、checker、guardrail 或脚本实现。
- 不执行 `git add`、commit、push、remote write、branch、tag、history、checkout、reset、clean。
- 不启动 runtime，不访问 NAS、凭据、交易账户、provider、lake、catalog 或 publish。

## 当前结论

CR122 已由用户批准通过 CP8 并关闭为 `closed-current-delivery / READY_WITH_RISK`。CP6 已按 CP5 授权执行 docs-only restore；CP7 静态验证确认 staged docs tree 与 `234b3d3^1` docs 基线一致，工作区 docs 无未暂存漂移。

残余风险：29 个 docs 变更仍处于本地 staged 状态，尚未 commit / push。CP8 关闭不授权 commit、push、remote write、`.gitignore`、source/checker/tests、CR121 cleanup、runtime、NAS、credentials、trading、provider、lake、catalog 或 publish；这些动作如需推进必须另起或恢复对应 gate。
