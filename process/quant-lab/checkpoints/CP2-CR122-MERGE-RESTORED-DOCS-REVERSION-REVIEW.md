---
checkpoint_id: "CP2-CR122"
checkpoint_name: "Merge-Restored Docs Reversion Scope Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-23T09:44:04+08:00"
updated_at: "2026-06-23T09:58:42+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-23T10:07:48+08:00"
auto_check_result: "process/checks/CP2-CR122-MERGE-RESTORED-DOCS-REVERSION-PRECHECK.md"
target:
  phase: "story-planning"
  story_id: ""
  artifacts:
    - "process/changes/CR-122-MERGE-RESTORED-DOCS-REVERSION-TO-REMEDIATED-BRANCH-BASELINE-2026-06-23.md"
    - "process/context/CP2-CR122-MERGE-RESTORED-DOCS-REVERSION-CONTEXT.yaml"
---

# CP2 CR122 Merge-Restored Docs Reversion Scope Review

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR122-MERGE-RESTORED-DOCS-REVERSION-PRECHECK.md` | PASS | 0 | 可发起 CP2 范围确认；已纳入用户修正后的 precheck docs 基线、work 分支 CR102/103/104 验证证据保留例外和后续 CR 修改保留规则；本轮无 docs 内容恢复动作。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR122-MERGE-RESTORED-DOCS-REVERSION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或用户要求时读取完整来源。 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |
| 基线修正 | 主基线为 `precheck/ql-rd-000-redesign-baseline` 的 pre-merge docs：`234b3d3^1` / `47a5eee6...`；`work/chapter3-factor-gap-remediation-20260608` 仅作为 CR102/103/104 验证记录 / 结果保留例外来源。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| 用户显式请求 | 当前对话 | scanned | 6 | 4 | 用户指定 baseline、恢复目标、CP2 禁止项、CP5 后范围；合并为 4 项 DQ。 |
| CR122 formal CR | `process/changes/CR-122-MERGE-RESTORED-DOCS-REVERSION-TO-REMEDIATED-BRANCH-BASELINE-2026-06-23.md` | scanned | 4 | 4 | 范围、恢复目标、冲突预检和禁止项已消费。 |
| CR121 formal CR | `process/changes/CR-121-MERGE-INDUCED-DIRECTORY-ROUTING-REGRESSION-HARDENING-2026-06-23.md` | scanned | 3 | 2 | 与 CR121 的 scope overlap 纳入 DQ-CP2-CR122-02/03。 |
| directory routing audit | `process/checks/CR120-DIRECTORY-ROUTING-MERGE-AUDIT-2026-06-23.md` | scanned | 3 | 2 | merge-restored docs 和 symlink 路由分层已纳入。 |
| Git read-only evidence | `git diff --name-status 234b3d3^1 234b3d3 -- docs` | scanned | 29 | 2 | 28 added / 1 modified 映射到目标 docs 清单。 |
| work branch validation evidence scan | `git ls-tree -r --name-only work/chapter3-factor-gap-remediation-20260608 -- docs` + CR102/103/104 搜索 | scanned | 3 | 1 | tracked docs 中未发现 CR103/CR104 验证记录 / 结果；仅发现 CR102 HLD，默认不保留为验证结果。纳入 DQ-CP2-CR122-03 的保留例外规则。 |
| post-merge docs modification scan | `git log 234b3d3..HEAD -- docs` | scanned | 0 | 1 | 当前无提交；纳入 DQ-CP2-CR122-03 的 CP5/CP6 前重扫与保留规则。 |
| runtime / NAS / credentials | N/A | n/a | 0 | 0 | 本 CP2 不需要外部运行时；明确列为不授权。 |

### Scope Summary

| 类别 | 路径 / 对象 | CP2 结论 |
|---|---|---|
| baseline | `precheck/ql-rd-000-redesign-baseline` pre-merge docs at `234b3d3^1` / `47a5eee6...` | 推荐作为主 docs 基线。 |
| merge commit | `234b3d3` | 只作为差异来源，不作为目标状态。 |
| work branch exception | `work/chapter3-factor-gap-remediation-20260608` | 仅保留 CR102/103/104 验证记录 / 验证结果；设计 / HLD / runbook 默认不保留。 |
| later CR modifications | `git log 234b3d3..HEAD -- docs` 当前为空 | CP5/CP6 前必须重扫；若存在后续合法 CR 修改，保留修改后的内容。 |
| added docs | 28 个 `docs/**` 路径 | CP5 approve 后恢复为不存在；CP2 不执行。 |
| modified docs | `docs/USER-MANUAL.md` | CP5 approve 后恢复为 `234b3d3^1` 版本；CP2 不执行。 |
| CR121 overlap | `docs/design`、`docs/features`、`docs/quality`、`.claude/**` | CR122 不处理；CR121 保持 CP5 approved but deferred/not-executed 的 index-only 路由治理。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR122-01 | scope | 是否接受 `precheck/ql-rd-000-redesign-baseline` 的 pre-merge docs 作为主基线？ | 接受：主基线为 `234b3d3^1` / `47a5eee6...` 上的 precheck docs，而不是 work 分支或当前 merge 结果。 | A. 以当前 merge 结果为准；B. 逐文件人工判定；C. 暂缓 CR122。 | 推荐方案符合用户修正后的目标且可机械验证；当前 merge 结果会保留回流文档；逐文件判定成本高；暂缓不修复。 | 决定后续 CP5/CP6 的恢复真相源。 | 若用户选择逐文件判定，回到 CP2 修改目标清单。 |
| DQ-CP2-CR122-02 | scope | 是否接受 CR122 仅处理 docs 内容恢复？ | 接受：CR122 只处理列出的 docs 新增 / 修改路径；不处理 `.claude/**`、`.gitignore`、checker、源码、tests。 | A. 同时纳入 `.claude/**`；B. 同时修改 `.gitignore`；C. 合并到 CR121。 | 推荐方案与 CR121 分层最清楚；A/B 扩大到平台产物或代码治理；C 会污染 CR121 index-only 授权。 | 降低与 CR121 的执行冲突。 | 若用户要求扩大范围，必须重写 CP2 和 CP5 授权边界。 |
| DQ-CP2-CR122-03 | implementation | CP5 后允许的恢复动作如何冻结？ | 后续 CP5 approve 后，CP6 仅允许对目标 docs 路径执行内容恢复：新增文件默认恢复为不存在，`docs/USER-MANUAL.md` 恢复 precheck 基线版本；但必须保留已确认的 CR102/103/104 验证记录 / 验证结果和后续合法 CR 修改。 | A. 只恢复 `docs/qmt/**` / `docs/goldminer/**`；B. 保留 release 公开入口；C. design-only 不执行恢复。 | 推荐方案完整覆盖用户目标并保留验证证据例外；A/B 可能保留 merge 回流；C 不产生整改。 | 后续会产生 docs 工作区 / index 变更；提交 / 推送仍需独立授权。 | 若 CP5 前发现清单不完整，或用户给出具体验证记录路径，回 CP2/CP5 修订保留例外清单。 |
| DQ-CP2-CR122-04 | runtime_authorization | 是否继续禁止 runtime、NAS、凭据、交易、provider、lake、catalog、publish，以及提交 / 推送？ | 继续禁止；CP2 不执行任何恢复命令，CP5 前不允许删除、`git rm` 或 `git restore`。 | A. 暂停所有验证命令；B. CP5 后同时授权 commit；C. 拆成 docs 删除和 USER-MANUAL 恢复两个 CR。 | 推荐方案权限最小；暂停验证会降低可审计性；commit/push 需独立发布门禁；拆分会增加门禁轮次。 | 防止内容恢复被误解为发布或外部动作授权。 | 如需 commit/push/runtime，另起高风险或发布门禁。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP2 四项推荐方案，冻结 CR122 范围、precheck docs 主基线、work 分支验证证据保留例外和后续 CR 修改保留策略。 |
| 备选方案 | `修改: <具体修改点>` 指明要调整的决策；或 `reject` 停止 CR122。 |
| 影响维度 | docs 内容恢复、CR121 scope separation、Git 工作区 / index、发布边界、安全权限 |
| 风险与回退 | 当前为中风险 docs governance；CP2 approve 不执行恢复，CP5 才能授权具体命令。 |
| 用户需决策事项 | 是否批准 `DQ-CP2-CR122-01..04`。 |
| 自动终验授权 | `auto_final_authorization: false`；CP2 approve 不等于 CP5/CP8、commit/push 或 runtime 授权。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| 用户请求明确 | 待审查 | 当前对话 | 已指定 CR122 目标和 CP2 禁止项。 |
| process 路由健康 | 待审查 | `meta-flow workspace check` | `process_link_health: ok`。 |
| merge docs diff 可读 | 待审查 | `git diff --name-status 234b3d3^1 234b3d3 -- docs` | 28 added / 1 modified。 |
| CR121 overlap 已识别 | 待审查 | CR121 变更单 | 通过 scope separation 处理。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | baseline 清楚 | 待审查 | `DQ-CP2-CR122-01` | 推荐 `precheck/ql-rd-000-redesign-baseline` pre-merge docs at `234b3d3^1` / `47a5eee6...`。 |
| 2 | CR122 范围清楚 | 待审查 | `DQ-CP2-CR122-02` | 只处理 docs 内容恢复。 |
| 3 | CP5 后恢复动作清楚 | 待审查 | `DQ-CP2-CR122-03` | CP2 不执行；CP5/CP6 前必须重扫 CR102/103/104 验证证据和后续 CR 修改。 |
| 4 | 不授权项清楚 | 待审查 | `DQ-CP2-CR122-04` | 删除 / git rm / restore / commit / runtime 均未授权。 |
| 5 | Decision Collection Coverage 完整 | 待审查 | 本文件 Decision Brief | 6 个来源已扫描。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认 CP2 | 待审查 | 本文件人工审查结果 | 等待用户回复。 |
| 可进入 CP5 authorization | 待审查 | DQ-CP2-CR122-01..04 | CP2 approve 后可准备 CP5；仍不执行恢复。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR122 formal CR | `process/changes/CR-122-MERGE-RESTORED-DOCS-REVERSION-TO-REMEDIATED-BRANCH-BASELINE-2026-06-23.md` | 待审查 |  |
| CP2 context capsule | `process/context/CP2-CR122-MERGE-RESTORED-DOCS-REVERSION-CONTEXT.yaml` | 待审查 |  |
| CP2 precheck | `process/checks/CP2-CR122-MERGE-RESTORED-DOCS-REVERSION-PRECHECK.md` | 待审查 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-23T10:07:48+08:00
- 修改意见：用户回复“批准，按照你推荐的方案执行。”接受 DQ-CP2-CR122-01..04 推荐方案；推荐方案为选择性恢复 docs 基线，不重跑 precheck CR。
- 风险接受项：接受 CP2 只确认范围和方案；不授权 docs 内容恢复、删除、`git rm`、`git restore`、`.gitignore`、源码 / checker / tests、commit / push 或 runtime。后续进入 CP5 授权准备。
