---
cr_id: "CR-123"
title: "CR122 Staged Docs Publication Gate"
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
rollback_to: "pre-CR123 state; CR122 staged docs changes remain local and uncommitted"
source: "user-request"
parent_cr: "CR-122"
source_checkpoint: "process/checkpoints/CP8-CR122-DELIVERY-READINESS.md"
source_follow_up_id: "FU-CR122-001"
follow_up_type: "staged-docs-publication-gate"
risk_class: "medium-local-git-commit-publication-pending-authorization"
owner: "host-orchestrator"
created_at: "2026-06-23T10:34:24+08:00"
updated_at: "2026-06-23T11:41:53+08:00"
acceptance_criteria: "CP2 confirms CR123 covers current CR122 29 staged docs changes plus docs/features CRxxx artifact directory remediation, while excluding CR121 cleanup, .gitignore, source/checker/tests, runtime and push. CP5 confirms dirty worktree allowlist, authorizes only a future CP6-A local git commit of the exact staged docs set, and authorizes only a future CP6-B deletion of the explicit non-priority docs/features CRxxx directory allowlist; no git add or push."
close_condition: "CP8 approved by user after CP6-A local commit, CP6-B artifact docs/features cleanup if approved, and verification, or user rejects / cancels before CP5. Remote push, runtime, NAS, credentials, trading, provider, lake, catalog and publish remain unauthorized unless separately gated."
---

# CR-123 CR122 Staged Docs Publication Gate

## 变更来源

用户要求启动 CR123：仅审查并决定是否提交当前 CR122 产生的 29 个 staged docs 变更；不纳入 CR121 cleanup，不修改 `.gitignore` / source / checker / tests，不启动 runtime，不 push。当前先做 CP2 / CP5 发布范围确认和 dirty worktree allowlist 预检。

CR122 已在 2026-06-23T10:22:20+08:00 经 CP8 approve 关闭为 `closed-current-delivery / READY_WITH_RISK`。其结果是：29 个 docs 变更保持本地 staged，尚未 commit / push。

2026-06-23T11:04:01+08:00 用户追加范围：`docs/features` 中 CRxxx 格式命名目录需要添加进来一起整改；除 CR102 / CR103 / CR104 三个文档目录需要重点处理外，其他文本基本可以直接删除。本 CR 因此修订为两个 CP6 切片：CP6-A 处理主仓库 29 个 staged docs local commit，CP6-B 处理 artifact repo 下 `docs/features` CRxxx 目录清理。

## 当前授权边界

| 范围 | 当前授权 | 说明 |
|---|---:|---|
| process artifacts 读取 / 写入 / 更新 | true | 仅用于 CR123 变更单、context、precheck、checkpoint、CR index 和 STATE。 |
| repository read-only audit | true | 可读取 Git status、index、diff 和 CR122/CR121 证据。 |
| local git commit | false | CP2/CP5 草案阶段不提交；后续必须 CP5 approve 后才可在 CP6 执行。 |
| artifact docs/features cleanup | false | CP2/CP5 草案阶段不删除；后续必须 CP5 approve 后才可在 CP6-B 执行 allowlist-only 删除候选目录处理。 |
| git add / stage 操作 | false | CR123 推荐策略是不执行 `git add`，只消费当前 CR122 已 staged 的 docs index。 |
| git push / remote write | false | 本 CR 不授权远端写；如需 push，必须另起 remote/publication gate。 |
| CR121 cleanup | false | 不执行 CR121 CP6 index-only cleanup。 |
| `.gitignore` / source / checker / tests 修改 | false | 不修改这些文件，也不把既有 dirty 文件纳入 CR123 commit。 |
| runtime / NAS / credentials / trading / provider / lake / catalog / publish | false | 本 CR 完全不授权。 |

## 五维度影响分析

| 维度 | 影响 | 结论 |
|---|---|---|
| 需求层 | true | 将 CR122 的本地 staged docs restore 从“未发布风险”推进到“可决定是否本地提交”的独立 publication gate，并把 `docs/features` CRxxx 文档目录整改纳入同一 CR。 |
| 场景层 | true | 覆盖精确 staged docs allowlist、排除非 CR123 dirty 文件、`docs/features` CRxxx 删除候选 / 重点处理分流、禁止 CR121 cleanup 和禁止 push/runtime 等场景。 |
| 计划层 | true | CR123 为 standard gate；CP2/CP5 当前只做范围确认和 dirty worktree / artifact cleanup 预检，CP6 才可在批准后执行 CP6-A local commit 与 CP6-B artifact cleanup。 |
| 安全 / 权限层 | true | local commit 会写 Git 历史，artifact cleanup 会删除文档内容，均必须单独授权；push、runtime、凭据和外部系统继续禁止。 |
| 交付层 | true | 若后续 CP6 执行，将把 CR122 docs restore 固化为本地 Git commit，并清理 artifact-routed feature CR 文档；远端发布仍需独立门禁。 |

## 冲突预检

| 冲突对象 | 状态 | 处理结论 |
|---|---|---|
| CR122 | closed-current-delivery / READY_WITH_RISK | CR123 仅发布 CR122 已 staged docs 结果，不改变 CR122 内容。 |
| CR121 | blocked-awaiting-user-next-step | 不纳入本 CR；CR121 CP6 index-only cleanup 不自动恢复。 |
| `scripts/check_human_gate_decision_brief.py` | unstaged modified | dirty worktree exclude，只审计，不提交。 |
| `tests/test_cr118_human_gate_path_alias_checker.py` | untracked | dirty worktree exclude，只审计，不提交。 |
| `tests/test_cr119_human_gate_launch_message_checker.py` | untracked | dirty worktree exclude，只审计，不提交。 |
| `docs/features` | symlink to artifact repo | 纳入 CR123 CP6-B 候选；本轮只审计和冻结 allowlist，不删除。 |

## `docs/features` CRxxx Artifact Directory 范围

`docs/features` 当前指向 `/home/hyde/workspace/meta-flow-artifacts/process/quant-lab/docs/features`。只读清点确认存在 10 个 CRxxx 格式目录；artifact repo 当前 `process/quant-lab/docs/features` 无 unstaged dirty 输出。

### 删除候选目录

| # | 路径 | 文件数 | 处理建议 |
|---:|---|---:|---|
| 1 | `docs/features/CR093-ledger-hygiene` | 4 | CP6-B allowlist-only 删除候选 |
| 2 | `docs/features/CR094-warning-cleanup` | 1 | CP6-B allowlist-only 删除候选 |
| 3 | `docs/features/CR095-standalone-checker-cli-output-convergence` | 1 | CP6-B allowlist-only 删除候选 |
| 4 | `docs/features/cr045-goldminer-bridge` | 3 | CP6-B allowlist-only 删除候选 |
| 5 | `docs/features/cr061-package-import-layout` | 8 | CP6-B allowlist-only 删除候选 |
| 6 | `docs/features/cr098-runner-readonly-integration` | 7 | CP6-B allowlist-only 删除候选 |
| 7 | `docs/features/cr099-runner-real-readonly-smoke` | 7 | CP6-B allowlist-only 删除候选 |

### 重点处理目录

| # | 路径 | 文件 | 默认处理 |
|---:|---|---|---|
| 1 | `docs/features/cr102-nas-real-package-exchange-authorization` | `DESIGN.md`, `TASKS.md`, `TEST-PLAN.md` | 默认保留，后续复盘是否压缩 / 迁移 / 删除设计残余。 |
| 2 | `docs/features/cr103-qmt-miniqmt-non-trading-day-validation` | `VERIFICATION.md` | 默认保留，作为验证记录重点复盘。 |
| 3 | `docs/features/cr104-qmt-miniqmt-trading-day-validation` | `INPUT-CHECKLIST.md` | 默认保留，作为验证输入 / checkpoint 重点复盘。 |

## 发布候选 allowlist

CR123 当前唯一允许纳入后续 CP6 local commit 的对象，是当前 Git index 中的 29 个 docs 路径：

| # | 路径 | index 状态 |
|---:|---|---|
| 1 | `docs/CR025-BACKTRADER-MODULE-REFERENCE.md` | staged delete |
| 2 | `docs/CR030-FACTOR-RESEARCH-QUICKSTART.md` | staged delete |
| 3 | `docs/QMT-C-S-BRIDGE-RUNBOOK.md` | staged delete |
| 4 | `docs/QMT-GATEWAY-INSTALL.md` | staged delete |
| 5 | `docs/QMT-INCIDENT-PLAYBOOK.md` | staged delete |
| 6 | `docs/QMT-SIMULATION-LIVE-RUNBOOK.md` | staged delete |
| 7 | `docs/QMT-TRADING-RUNBOOK.md` | staged delete |
| 8 | `docs/USER-MANUAL.md` | staged modify |
| 9 | `docs/goldminer/CR045-BRIDGE-RUNBOOK.md` | staged delete |
| 10 | `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | staged delete |
| 11 | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md` | staged delete |
| 12 | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md` | staged delete |
| 13 | `docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md` | staged delete |
| 14 | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml` | staged delete |
| 15 | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-HLD.md` | staged delete |
| 16 | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-MANUAL-GUIDE.md` | staged delete |
| 17 | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-TEST-PLAN.md` | staged delete |
| 18 | `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md` | staged delete |
| 19 | `docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md` | staged delete |
| 20 | `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-HLD.md` | staged delete |
| 21 | `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-RECOVERY-RUNBOOK.md` | staged delete |
| 22 | `docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md` | staged delete |
| 23 | `docs/qmt/CR101-VALIDATION-AND-FOLLOW-UP-GATES.md` | staged delete |
| 24 | `docs/qmt/CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-HLD.md` | staged delete |
| 25 | `docs/release/DEPLOY-CHECKLIST.md` | staged delete |
| 26 | `docs/release/FEEDBACK.md` | staged delete |
| 27 | `docs/release/MIGRATION.md` | staged delete |
| 28 | `docs/release/RELEASE-NOTES.md` | staged delete |
| 29 | `docs/release/ROLLBACK.md` | staged delete |

## Dirty Worktree Allowlist 预检

| 检查项 | 证据 | 结论 |
|---|---|---|
| staged docs count | `git diff --cached --name-only -- docs \| wc -l` = 29 | PASS |
| unstaged docs drift | `git diff --name-only -- docs \| wc -l` = 0 | PASS |
| staged docs baseline | `git diff --cached --quiet 234b3d3^1 -- docs` exit 0 | PASS |
| excluded unstaged tracked | `scripts/check_human_gate_decision_brief.py` | EXCLUDED / NOT FOR COMMIT |
| excluded untracked | `tests/test_cr118_human_gate_path_alias_checker.py`; `tests/test_cr119_human_gate_launch_message_checker.py` | EXCLUDED / NOT FOR COMMIT |
| process route | `meta-flow workspace check --project-root .` | PASS; artifact repo dirty because process artifacts are being updated. |

## 待人工决策项

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR123-01 | scope | 是否确认 CR123 仅覆盖 CR122 的 29 个 staged docs 变更？ | 确认，仅允许这 29 个 docs 路径进入后续 CP6 local commit。 | A. 暂停 publication；B. 回到 CR122 重新恢复；C. 改为 CR121 cleanup 优先。 | 推荐方案最小化发布范围；备选会延后或改变当前目标。 |
| DQ-CP2-CR123-02 | implementation | 是否接受 CP2/CP5 当前只做预检，不立即提交？ | 接受 staged route：本轮不 commit；CP5 approve 后 CP6 才能 local commit。 | A. design-only 不进入 CP5；B. 直接取消 CR123；C. 拆成 CP2 后另起 CP5。 | 推荐方案保持门禁完整；直接提交会越权。 |
| DQ-CP2-CR123-03 | runtime_authorization | 是否继续禁止 push、runtime、`.gitignore`、source/checker/tests 和 CR121 cleanup？ | 继续禁止；本 CR 不授权这些动作。 | A. 另起 push gate；B. 另起 CR121 resume；C. 另起 checker/.gitignore CR。 | 防止 local commit gate 被误解为远端发布或其他 cleanup 授权。 |
| DQ-CP2-CR123-04 | scope | 是否把 `docs/features` CRxxx artifact directory 纳入 CR123 一起整改？ | 纳入同一 CR，但拆成 CP6-B；7 个非重点 CRxxx 目录作为删除候选，CR102/103/104 三个目录作为重点处理对象。 | A. 另起 CR124；B. 只记录不整改；C. 并入 CR121 cleanup。 | 推荐方案响应当前范围变更且避免混淆 index-only cleanup。 |
| DQ-CP5-CR123-01 | implementation | 后续 CP6 是否可执行 local git commit？ | 授权 CP6 只在 index 精确等于 29 个 allowlisted docs 路径且无额外 staged 文件时执行一次 local commit。 | A. 不提交，仅保持 staged；B. 要求先清理其他 dirty 文件；C. 分两次 commit。 | 推荐方案可固化 CR122 结果；其他 dirty 文件不会进入 commit。 |
| DQ-CP5-CR123-02 | security | dirty worktree allowlist 如何冻结？ | 允许既有 excluded dirty 文件保留未暂存 / 未跟踪，但若它们进入 index 或新增 staged 非 allowlist 文件，CP6 必须停止。 | A. 要求工作区完全干净后再 commit；B. 把 CR118/CR119 文件也纳入 commit；C. 暂停。 | 推荐方案风险可控且不破坏既有工作；纳入其他文件会越界。 |
| DQ-CP5-CR123-03 | implementation | local commit message 如何冻结？ | 使用 `CR123: publish CR122 staged docs restore`。 | A. `Publish CR122 docs baseline restore`; B. `CR122 docs baseline restore`; C. CP6 前再确认 message。 | 推荐方案可追溯 CR123 与 CR122；message 不影响内容但影响审计。 |
| DQ-CP5-CR123-04 | runtime_authorization | CP5 approve 是否授权 push、remote write、runtime 或外部系统？ | 不授权；只允许未来 CP6 local commit。 | A. 另起 remote publication gate；B. 另起 runtime gate；C. 暂停。 | 防止 local commit 被误解为发布到远端。 |
| DQ-CP5-CR123-05 | follow_up_tracking | CP6 local commit 后如何收敛？ | CP6 后做静态验证并进入 CP8；CR121 仍等待用户另行选择。 | A. CP6 后立即恢复 CR121；B. CP6 后启动 push gate；C. CP6 后暂停。 | 推荐方案先关闭本 CR；后续工作不自动串联。 |
| DQ-CP5-CR123-06 | implementation | 后续 CP6-B 是否可删除非重点 `docs/features` CRxxx 目录？ | 授权 CP6-B 仅在 artifact repo 前置检查通过时删除 7 个 allowlisted 非重点目录。 | A. 另起 CR124 删除；B. 只删除部分目录；C. 只记录不删除。 | 推荐方案符合用户新增要求，但删除前必须重跑 allowlist。 |
| DQ-CP5-CR123-07 | implementation | CR102/CR103/CR104 三个 `docs/features` 目录如何重点处理？ | 默认保留并进入重点审查，不在 CP6-B 默认删除范围。 | A. 全部保留不处理；B. 只保留 CR103/CR104；C. 另起 CR 单独逐文件处理。 | 推荐方案防止误删近期授权 / 验证证据。 |
| DQ-CP5-CR123-08 | implementation | CR123 后续执行顺序如何冻结？ | CP6-A 先主仓库 29 staged docs local commit，CP6-B 再 artifact docs/features cleanup；两者均不 push。 | A. 先删 feature；B. 混在一个步骤；C. 只做 CP6-A。 | 推荐方案可审计且先解除 staged docs 悬挂风险。 |

## 不授权范围

- 本轮不执行 local git commit。
- 本轮不删除 `docs/features` 内容。
- 不执行 `git add`、`git rm`、`git restore`、checkout、reset、clean。
- 不执行 push、remote add、remote set-url、tag、branch/default branch、force/history 操作。
- 不修改 `.gitignore`、source、checker、tests。
- 不执行 CR121 CP6 index-only cleanup。
- 不启动 runtime，不访问 NAS、credentials、trading、provider、lake、catalog 或 publish。

## 当前结论

CR123 CP6 已执行完成。CP6-A 已创建本地提交 `940bcd9 CR123: publish CR122 staged docs restore`，只包含 CR122 的 29 个 docs 变更。CP6-B 已删除 7 个 allowlisted 非重点 `docs/features` CRxxx artifact 目录；CR102 / CR103 / CR104 三个重点目录默认保留并仍存在。当前未执行 push、runtime、CR121 cleanup、`.gitignore`、source、checker 或 tests 修改。

CP8 已由用户在 2026-06-23T11:41:53+08:00 回复 approve。CR123 关闭为 `closed-current-delivery / READY_WITH_RISK`。

关闭结论：

- 主仓库本地提交 `940bcd9 CR123: publish CR122 staged docs restore` 已完成。
- artifact repo 中 7 个非重点 `docs/features` CRxxx 目录已本地删除，共 31 个文件删除项。
- CR102 / CR103 / CR104 三个重点目录已保留。
- CP8 approve 不授权 push、runtime、CR121 cleanup、`.gitignore`、source、checker 或 tests 修改。

后续分流：

- artifact repo 31 个 feature docs 删除项由 CR124 承接 publication gate。
- CR121 详细清理与 CR102/103/104 重点目录复盘写入后续 runner 开发相关 CR，不在 CR124 处理。
