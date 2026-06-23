---
checkpoint_id: "CP5-CR122"
checkpoint_name: "Merge-Restored Docs Reversion Authorization Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-23T10:07:48+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-23T10:15:47+08:00"
auto_check_result: "process/checks/CP5-CR122-MERGE-RESTORED-DOCS-REVERSION-AUTHORIZATION-PRECHECK.md"
target:
  phase: "story-planning"
  story_id: ""
  artifacts:
    - "process/changes/CR-122-MERGE-RESTORED-DOCS-REVERSION-TO-REMEDIATED-BRANCH-BASELINE-2026-06-23.md"
    - "process/context/CP5-CR122-MERGE-RESTORED-DOCS-REVERSION-AUTHORIZATION-CONTEXT.yaml"
---

# CP5 CR122 Merge-Restored Docs Reversion Authorization Review

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR122-MERGE-RESTORED-DOCS-REVERSION-AUTHORIZATION-PRECHECK.md` | PASS | 0 | 可发起 CP5 授权确认；本文件只授权后续 CP6 范围，不执行 docs 恢复。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR122-MERGE-RESTORED-DOCS-REVERSION-AUTHORIZATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或用户要求时读取完整来源。 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |
| CP2 结论 | 用户已批准选择性恢复 docs 基线；不重跑 precheck CR。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| 用户 CP2 approve | 当前对话 | scanned | 1 | 1 | “批准，按照你推荐的方案执行”转为 CP5 授权准备，不直接执行 CP6。 |
| CP2 checkpoint | `process/checkpoints/CP2-CR122-MERGE-RESTORED-DOCS-REVERSION-REVIEW.md` | scanned | 4 | 3 | 基线、范围、恢复动作进入 CP5 DQ。 |
| CP5 precheck | `process/checks/CP5-CR122-MERGE-RESTORED-DOCS-REVERSION-AUTHORIZATION-PRECHECK.md` | scanned | 5 | 5 | CP6 范围、不授权项、重扫规则进入待决策。 |
| Git docs diff | `git diff --name-status 234b3d3^1 234b3d3 -- docs` | scanned | 29 | 1 | 29 个路径归入 DQ-CP5-CR122-01。 |
| work branch evidence scan | `work/chapter3-factor-gap-remediation-20260608` docs scan | scanned | 3 | 1 | 当前无 confirmed docs validation record paths；纳入 DQ-CP5-CR122-02。 |
| runtime / NAS / credentials | N/A | n/a | 0 | 0 | 本 CP5 不需要外部运行时；明确列为不授权。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR122-01 | implementation | 是否授权后续 CP6 执行 docs-only 内容恢复？ | 授权 CP6 仅对 CR122 context 中列出的 29 个 docs 目标执行内容恢复：28 个 merge-restored 新增 docs 默认恢复为不存在，`docs/USER-MANUAL.md` 恢复 precheck 基线版本。 | A. 只恢复 `docs/USER-MANUAL.md`；B. 只恢复 docs/qmt 与 docs/goldminer；C. design-only，不执行恢复。 | 推荐方案完整覆盖用户目标；A/B 会保留部分 merge 回流；C 不修复。 | CP6 会产生 docs 工作区 / index 变更；提交 / 推送仍需独立授权。 | 若 CP6 前路径清单变化，停止并回 CP5 修订。 |
| DQ-CP5-CR122-02 | scope | CR102/103/104 验证记录 / 结果如何保留？ | 保留规则而非预置路径：CP6 前重扫 work 分支和 process evidence；只保留被确认是 CR102/103/104 验证记录 / 验证结果的 docs 路径。当前 confirmed docs 保留路径为空，CR102 HLD 默认不保留。 | A. 保留所有 CR102/103/104 命名文件；B. 保留 `docs/qmt/CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-HLD.md`；C. 不保留任何 work 分支 docs。 | 推荐方案符合用户“验证记录和验证结果”限定；A/B 会保留设计/HLD；C 可能误删真实验证证据。 | CP6 前若发现真实验证结果路径，会缩小删除集合。 | 用户提供具体验证记录路径时，更新 allowlist 后再 CP6。 |
| DQ-CP5-CR122-03 | implementation | 后续 CR 对原基线 docs 的合法修改如何处理？ | CP6 前必须重跑 `git log 234b3d3..HEAD -- docs` 和工作区 diff 检查；若存在后续合法 CR 修改，则保留修改后的内容，不机械覆盖。 | A. 忽略后续修改，严格回到 precheck 基线；B. 对所有当前 docs 保守保留；C. 暂停等待人工逐文件确认。 | 推荐方案兼顾恢复和不覆盖合法变更；A 可能丢修改；B 会保留 merge 回流；C 成本高。 | 需要 CP6 前再做一次只读预检。 | 发现不确定修改时暂停 CP6 并重提 CP5。 |
| DQ-CP5-CR122-04 | risk_acceptance | 是否接受 CP6 后只留下本地 docs 工作区 / index 变更，不自动提交 / 推送？ | 接受：CP6 只做本地 docs 内容恢复和验证；不执行 commit、push、remote write、branch/tag/history 操作。 | A. CP6 后立即另起 publication gate；B. 本轮不执行 CP6；C. 把恢复和 commit 合并到一个高风险门禁。 | 推荐方案权限最小；publication gate 更完整但另需审查；不执行不修复；合并 commit 范围过大。 | 本地变更需要后续发布治理。 | 若不能接受 dirty worktree，选择不执行 CP6 或先设计 publication gate。 |
| DQ-CP5-CR122-05 | runtime_authorization | CP5 approve 是否授权 `.gitignore`、source/checker/tests、CR121 index cleanup、runtime/NAS/credentials/trading/provider/lake/catalog/publish？ | 不授权；CP5 只授权后续 CP6 docs-only 内容恢复。CR121 index-only cleanup 保持 separate/deferred。 | A. 同时授权 CR121 CP6；B. 同时授权 `.gitignore` 或 checker hardening；C. 同时授权 commit/push。 | 推荐方案保持边界清晰；备选都会扩大到其他 CR 或高风险发布治理。 | 防止 CP5 被误读为源码、索引清理、外部运行或发布授权。 | 任一禁止项都必须另起或重开对应 CR / gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP5 五项推荐方案，授权后续 CP6 执行 docs-only 内容恢复。 |
| 备选方案 | `修改: <具体修改点>` 指明要调整的 DQ；或 `reject` 停止 CR122。 |
| 影响维度 | docs 内容恢复、验证证据保留、后续 CR 修改保留、Git 工作区 / index、发布边界、安全权限 |
| 风险与回退 | 中风险 docs governance；CP5 approve 后 CP6 仍需重扫保留例外和后续 CR 修改。 |
| 用户需决策事项 | 是否批准 `DQ-CP5-CR122-01..05`。 |
| 自动终验授权 | `auto_final_authorization: false`；CP5 approve 不等于 CP8、commit/push 或 runtime 授权。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 已批准 | 待审查 | CP2 review 人工审查结果 | 用户批准推荐方案。 |
| CP5 预检通过 | 待审查 | CP5 precheck | PASS，阻断项 0。 |
| CP5 context ready | 待审查 | CP5 context capsule | `read_profile=minimal`。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | docs-only 恢复范围明确 | 待审查 | `DQ-CP5-CR122-01` | 29 个 docs 目标。 |
| 2 | 验证证据保留例外明确 | 待审查 | `DQ-CP5-CR122-02` | 当前 confirmed docs allowlist 为空，CP6 前重扫。 |
| 3 | 后续 CR 修改保留规则明确 | 待审查 | `DQ-CP5-CR122-03` | CP6 前重扫。 |
| 4 | dirty worktree 风险可接受 | 待审查 | `DQ-CP5-CR122-04` | commit/push 不授权。 |
| 5 | 不授权边界清楚 | 待审查 | `DQ-CP5-CR122-05` | source/checker/tests/.gitignore/runtime/CR121 cleanup 不授权。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认 CP5 | 待审查 | 本文件人工审查结果 | 等待用户回复。 |
| 可进入 CP6 docs-only 恢复 | 待审查 | DQ-CP5-CR122-01..05 | CP5 approve 后可进入 CP6；仍不 commit/push/runtime。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 context capsule | `process/context/CP5-CR122-MERGE-RESTORED-DOCS-REVERSION-AUTHORIZATION-CONTEXT.yaml` | 待审查 |  |
| CP5 precheck | `process/checks/CP5-CR122-MERGE-RESTORED-DOCS-REVERSION-AUTHORIZATION-PRECHECK.md` | 待审查 |  |
| CR122 formal CR | `process/changes/CR-122-MERGE-RESTORED-DOCS-REVERSION-TO-REMEDIATED-BRANCH-BASELINE-2026-06-23.md` | 待审查 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-23T10:15:47+08:00
- 修改意见：用户回复 `approve`，接受 DQ-CP5-CR122-01..05 推荐方案。
- 风险接受项：授权后续 CP6 仅对 CR122 context 中列出的 29 个 docs 目标执行 docs-only 内容恢复；接受 CP6 后仅产生本地 docs 工作区 / index 变更，不自动 commit / push。仍不授权 `.gitignore`、source/checker/tests、CR121 index cleanup、runtime/NAS/credentials/trading/provider/lake/catalog/publish。
