---
cr_id: "CR-124"
title: "Artifact Feature Docs Cleanup Publication Gate"
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
rollback_to: "pre-CR124 artifact commit state; revert artifact local commit 144a7b3 only if explicitly authorized by a rollback gate"
source: "user-request"
parent_cr: "CR-123"
source_checkpoint: "process/checkpoints/CP8-CR123-DELIVERY-READINESS.md"
source_follow_up_id: "FU-CR123-001"
follow_up_type: "artifact-feature-docs-cleanup-publication-gate"
risk_class: "medium-artifact-repo-local-commit-publication-pending-authorization"
owner: "host-orchestrator"
created_at: "2026-06-23T11:41:53+08:00"
updated_at: "2026-06-23T12:06:50+08:00"
acceptance_criteria: "CP2 confirms CR124 only covers the existing 31 artifact repo docs/features deletion entries produced by CR123 CP6-B and excludes CR121, CR102/103/104 priority directory cleanup, source/checker/tests, runtime and push. CP5 confirms dirty worktree allowlist and authorizes only a future CP6 local artifact repo commit of the exact 31 deletion entries; no push."
close_condition: "CP8 approved by user after artifact repo local commit and verification, or user rejects / cancels before CP5. Remote push, runtime, NAS, credentials, trading, provider, lake, catalog and publish remain unauthorized unless separately gated."
---

# CR-124 Artifact Feature Docs Cleanup Publication Gate

## 变更来源

用户在 CR123 CP8 approve 时要求启动 CR124：将 CR123 CP6-B 产生的 artifact repo `docs/features` 删除结果纳入独立 publication gate。用户同时明确：CR121 与 CR102/103/104 详细清理写入后续 runner 开发相关 CR，不在本 CR 中处理。

CR123 已关闭为 `closed-current-delivery / READY_WITH_RISK`；主仓库提交 `940bcd9` 已完成。当前 CR124 仅处理 `/home/hyde/workspace/meta-flow-artifacts` 中 `process/quant-lab/docs/features` 的 31 个删除项是否固化为本地 artifact commit。

## 当前授权边界

| 范围 | 当前授权 | 说明 |
|---|---:|---|
| process artifacts 读取 / 写入 / 更新 | true | 仅用于 CR124 变更单、context、precheck、checkpoint、CR index 和 STATE。 |
| artifact repo read-only audit | true | 可读取 artifact repo status / diff。 |
| artifact repo local commit | false | CP2/CP5 草案阶段不提交；后续必须 CP5 approve 后才可在 CP6 执行。 |
| artifact repo git add | false | 本轮不 stage；CP5 approve 后才可 CP6 stage 精确 allowlist。 |
| git push / remote write | false | 不授权任何远端写。 |
| CR121 cleanup | false | 不执行 CR121 index-only cleanup。 |
| CR102/103/104 详细清理 | false | 写入后续 runner 开发相关 CR，本 CR 不处理。 |
| `.gitignore` / source / checker / tests 修改 | false | 不修改这些文件。 |
| runtime / NAS / credentials / trading / provider / lake / catalog / publish | false | 本 CR 完全不授权。 |

## 五维度影响分析

| 维度 | 影响 | 结论 |
|---|---|---|
| 需求层 | true | 将 CR123 已执行的 artifact feature docs 删除从 local dirty state 推进到可决定是否本地提交的 publication gate。 |
| 场景层 | true | 覆盖 31 个删除项 allowlist、排除 process artifacts dirty、排除 CR102/103/104 重点目录、禁止 push/runtime/CR121。 |
| 计划层 | true | 当前只做 CP2/CP5 范围与授权预检；CP5 approve 后 CP6 才可执行 artifact repo local commit。 |
| 安全 / 权限层 | true | artifact repo commit 会写本地 Git 历史；push/runtime/外部系统继续禁止。 |
| 交付层 | true | 若后续 CP6 执行，将把 CR123 feature docs cleanup 固化为 artifact repo local commit。 |

## Artifact 删除候选 allowlist

当前唯一允许纳入后续 CP6 artifact commit 的对象，是 artifact repo 中 `process/quant-lab/docs/features` 下 31 个删除项，全部来自 7 个非重点 CRxxx 目录：

| 目录 | 删除文件数 | 处理 |
|---|---:|---|
| `process/quant-lab/docs/features/CR093-ledger-hygiene` | 4 | allowlist |
| `process/quant-lab/docs/features/CR094-warning-cleanup` | 1 | allowlist |
| `process/quant-lab/docs/features/CR095-standalone-checker-cli-output-convergence` | 1 | allowlist |
| `process/quant-lab/docs/features/cr045-goldminer-bridge` | 3 | allowlist |
| `process/quant-lab/docs/features/cr061-package-import-layout` | 8 | allowlist |
| `process/quant-lab/docs/features/cr098-runner-readonly-integration` | 7 | allowlist |
| `process/quant-lab/docs/features/cr099-runner-real-readonly-smoke` | 7 | allowlist |

明确排除：

- `process/quant-lab/docs/features/cr102-nas-real-package-exchange-authorization`
- `process/quant-lab/docs/features/cr103-qmt-miniqmt-non-trading-day-validation`
- `process/quant-lab/docs/features/cr104-qmt-miniqmt-trading-day-validation`

## 冲突预检

| 冲突对象 | 状态 | 处理结论 |
|---|---|---|
| CR123 | closed-current-delivery / READY_WITH_RISK | CR124 只发布 CR123 CP6-B 的 artifact deletion 结果。 |
| CR121 | blocked-awaiting-user-next-step | 不纳入本 CR；后续 runner 开发相关 CR 再处理。 |
| CR102/103/104 priority review | deferred-to-runner-development-cr | 不纳入本 CR。 |
| artifact repo process artifacts dirty | present | process artifacts dirty 只审计，不纳入 CR124 artifact feature deletion commit。 |
| main repo dirty scripts/tests | present | 不纳入本 CR。 |

## 待人工决策项

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR124-01 | scope | 是否确认 CR124 仅覆盖 artifact repo 中 31 个 feature docs 删除项？ | 确认，仅允许这 31 个 deletion entries 进入后续 CP6 artifact commit。 | A. 暂停 publication；B. 回到 CR123 恢复 feature docs；C. 合并 CR121。 | 推荐方案最小；合并 CR121 会扩大范围。 |
| DQ-CP2-CR124-02 | scope | CR121 与 CR102/103/104 详细清理如何处理？ | 写入后续 runner 开发相关 CR，不在 CR124 执行。 | A. 立即启动单独 review CR；B. 纳入 CR124；C. 暂停记录。 | 推荐方案保持 CR124 可快速收口。 |
| DQ-CP2-CR124-03 | runtime_authorization | 是否继续禁止 push、runtime、source/checker/tests、CR121 cleanup？ | 继续禁止；本 CR 不授权这些动作。 | A. 另起 push gate；B. 另起 CR121 resume；C. 另起 runner cleanup CR。 | 防止 artifact commit gate 被误解为远端发布或治理总清理。 |
| DQ-CP5-CR124-01 | implementation | 后续 CP6 是否可执行 artifact repo local commit？ | 授权 CP6 只 stage/commit 31 个 allowlisted feature docs 删除项。 | A. 不提交，仅保持 dirty；B. 要求先提交全部 process artifacts；C. 分目录 commit。 | 推荐方案固化 cleanup 且不混入 process artifacts。 |
| DQ-CP5-CR124-02 | security | artifact repo dirty allowlist 如何冻结？ | 允许 process artifacts dirty 保持未提交；若 CP6 stage 中出现非 31 allowlist 路径则停止。 | A. 要求 artifact repo 全干净；B. 把 process artifacts 一并提交；C. 暂停。 | 推荐方案避免混合运行态文件和 feature docs cleanup。 |
| DQ-CP5-CR124-03 | implementation | artifact commit message 如何冻结？ | 使用 `CR124: publish artifact feature docs cleanup`。 | A. `Publish CR123 artifact feature cleanup`; B. `CR123 feature docs cleanup`; C. CP6 前再确认。 | message 影响审计。 |
| DQ-CP5-CR124-04 | runtime_authorization | CP5 approve 是否授权 push 或外部系统？ | 不授权；只允许 future CP6 local artifact commit。 | A. 另起 artifact push gate；B. 另起 runtime gate；C. 暂停。 | 防止 local commit 被误解为远端发布。 |

## 不授权范围

- 本轮不执行 artifact repo local commit。
- 本轮不执行 artifact repo `git add`。
- 不执行 push、remote write、tag、branch/default branch、force/history 操作。
- 不执行 CR121 cleanup。
- 不处理 CR102/103/104 详细清理。
- 不修改 `.gitignore`、source、checker、tests。
- 不启动 runtime，不访问 NAS、credentials、trading、provider、lake、catalog 或 publish。

## 当前结论

CR124 已通过 CP2/CP5 approve，并在 CP6 执行 artifact repo local commit：

- artifact repo：`/home/hyde/workspace/meta-flow-artifacts`
- commit：`144a7b3`
- message：`CR124: publish artifact feature docs cleanup`
- 内容：31 个 allowlisted `process/quant-lab/docs/features` deletion entries，合计 2051 deletions。

CP7 静态验证结论为 `PASS_WITH_RISK`：artifact staged 已清空，`docs/features/cr102-*`、`docs/features/cr103-*`、`docs/features/cr104-*` 仍保留；未执行 push/runtime/CR121 cleanup，也未修改 `.gitignore`、source、checker 或 tests。

用户已于 2026-06-23T12:06:50+08:00 批准 CP8，CR124 关闭为 `closed-current-delivery / READY_WITH_RISK`。

关闭结论：

- artifact repo local commit `144a7b3` 已作为当前交付接受，未 push。
- `FU-CR124-001` 已转入后续 `CR-125 Runner Development Readiness Documentation Cleanup Gate`。
- 关闭 CR124 不授权 push、remote write、runtime、NAS、credentials、trading、provider、lake、catalog、publish、CR121 cleanup、`.gitignore` 修改、source/checker/tests 修改或 CR102/103/104 默认删除。
