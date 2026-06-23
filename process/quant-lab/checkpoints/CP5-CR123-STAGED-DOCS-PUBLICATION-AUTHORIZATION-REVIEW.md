---
checkpoint_id: "CP5-CR123"
checkpoint_name: "CR123 Staged Docs Publication Authorization Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-23T10:34:24+08:00"
updated_at: "2026-06-23T11:04:01+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-23T11:21:01+08:00"
auto_check_result: "process/checks/CP5-CR123-STAGED-DOCS-PUBLICATION-AUTHORIZATION-PRECHECK.md"
target:
  phase: "story-planning"
  story_id: ""
  artifacts:
    - "process/context/CP5-CR123-STAGED-DOCS-PUBLICATION-AUTHORIZATION-CONTEXT.yaml"
    - "process/checks/CP2-CR123-STAGED-DOCS-PUBLICATION-PRECHECK.md"
---

# CP5 CR123 Staged Docs Publication Authorization Review

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR123-STAGED-DOCS-PUBLICATION-AUTHORIZATION-PRECHECK.md` | PASS | 0 | dirty worktree allowlist 与 `docs/features` CRxxx cleanup allowlist 可确认；后续 CP6 执行仍需用户 approve。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR123-STAGED-DOCS-PUBLICATION-AUTHORIZATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或用户要求时读取完整来源。 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP2 scope | `process/checkpoints/CP2-CR123-STAGED-DOCS-PUBLICATION-REVIEW.md` | scanned | 3 | 0 | CP2 DQ 已在 CP2 review 中列出。 |
| CP5 precheck | `process/checks/CP5-CR123-STAGED-DOCS-PUBLICATION-AUTHORIZATION-PRECHECK.md` | scanned | 5 | 5 | local commit authorization, dirty allowlist, message, forbidden scope, follow-up。 |
| Git index audit | `git status --short`; `git diff --cached --name-status -- docs` | scanned | 3 | 2 | allowlist + excluded dirty。 |
| `docs/features` artifact audit | `/home/hyde/workspace/meta-flow-artifacts/process/quant-lab/docs/features` | scanned | 3 | 3 | CP6-B cleanup allowlist、CR102/103/104 重点处理、执行切片顺序。 |
| runtime / remote / credentials | N/A | n/a | 0 | 1 | 不授权项进入 DQ-CP5-CR123-04。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR123-01 | implementation | 后续 CP6 是否可执行 local git commit？ | 授权 CP6 只在 index 精确等于 29 个 allowlisted docs 路径且无额外 staged 文件时执行一次 local commit。 | A. 不提交，仅保持 staged；B. 要求先清理其他 dirty 文件；C. 分两次 commit。 | 推荐方案可固化 CR122 结果；A 保留 staged 风险；B 会阻塞无关工作；C 增加审计复杂度。 | 会写本地 Git 历史，但不写远端。 | CP6 前任何 staged 非 allowlist 文件出现时停止。 |
| DQ-CP5-CR123-02 | security | dirty worktree allowlist 如何冻结？ | 允许既有 excluded dirty 文件保留未暂存 / 未跟踪，但若它们进入 index 或新增 staged 非 allowlist 文件，CP6 必须停止。 | A. 要求工作区完全干净后再 commit；B. 把 CR118/CR119 文件也纳入 commit；C. 暂停。 | 推荐方案风险可控且不破坏既有工作；A 成本高；B 越界；C 不解决 staged docs。 | 依赖 CP6 前重新检查 index。 | excluded 文件被 staged 或 allowlist 改变时回 CP5。 |
| DQ-CP5-CR123-03 | implementation | local commit message 如何冻结？ | 使用 `CR123: publish CR122 staged docs restore`。 | A. `Publish CR122 docs baseline restore`; B. `CR122 docs baseline restore`; C. CP6 前再确认 message。 | 推荐方案可追溯 CR123 与 CR122；备选 message 也可行但追溯弱。 | message 不影响内容，但影响审计。 | 用户修改 message 时只更新 CP5 决策，不改内容范围。 |
| DQ-CP5-CR123-04 | runtime_authorization | CP5 approve 是否授权 push、remote write、runtime 或外部系统？ | 不授权；只允许未来 CP6 local commit。 | A. 另起 remote publication gate；B. 另起 runtime gate；C. 暂停。 | 推荐方案权限最小；A/B 均超出当前目标。 | 防止 local commit 被误解为远端发布。 | 任何 push/runtime 需求必须另起 gate。 |
| DQ-CP5-CR123-05 | follow_up_tracking | CP6 local commit 后如何收敛？ | CP6 后做静态验证并进入 CP8；CR121 仍等待用户另行选择。 | A. CP6 后立即恢复 CR121；B. CP6 后启动 push gate；C. CP6 后暂停。 | 推荐方案先关闭本 CR；A/B 影响面不同，需单独选择。 | 避免自动串联 CR121 或 push。 | 用户明确选择后再启动相应 gate。 |
| DQ-CP5-CR123-06 | implementation | 后续 CP6-B 是否可删除非重点 `docs/features` CRxxx 目录？ | 授权 CP6-B 仅在 artifact repo 前置检查通过时删除 7 个 allowlisted 目录：`CR093-ledger-hygiene`、`CR094-warning-cleanup`、`CR095-standalone-checker-cli-output-convergence`、`cr045-goldminer-bridge`、`cr061-package-import-layout`、`cr098-runner-readonly-integration`、`cr099-runner-real-readonly-smoke`。 | A. 另起 CR124 删除；B. 只删除空目录或无 VERIFICATION 的目录；C. 暂不删除仅记录。 | 推荐方案符合用户“其他文本基本可以直接删除”；A 增加轮次；B 会留下历史 CR 文档残余；C 不整改。 | 会删除 artifact repo 中已跟踪文档内容；CP6-B 前必须重跑 allowlist，且不影响主仓库 29 个 staged docs commit。 | 任一目录清单不匹配、出现未审查修改或用户要求保留时停止并回 CP5。 |
| DQ-CP5-CR123-07 | implementation | CR102/CR103/CR104 三个 `docs/features` 目录如何重点处理？ | 默认保留并进入重点审查：CR102 的 DESIGN/TASKS/TEST-PLAN、CR103 的 VERIFICATION、CR104 的 INPUT-CHECKLIST 不在 CP6-B 默认删除范围；后续只做内容复盘/必要压缩或迁移建议。 | A. 三个目录全部保留不处理；B. 只保留 CR103/CR104 验证记录，删除 CR102 设计文档；C. 三个目录另起 CR 单独逐文件处理。 | 推荐方案避免误删 CR102/103/104 验证链；A 不解决重点处理；B 可能提前删除授权设计上下文；C 增加轮次但更保守。 | 保留这些文件会留下少量 artifact docs，但可防止误删近期验证与授权证据。 | 审查发现某文件只是重复设计残余时，另行更新 allowlist 后再删除。 |
| DQ-CP5-CR123-08 | implementation | CR123 后续执行顺序如何冻结？ | 拆成两个 CP6 切片：先 CP6-A 处理主仓库 29 个 staged docs local commit；再 CP6-B 处理 artifact repo `docs/features` cleanup；两者均不 push。 | A. 先删 feature 再 commit docs；B. 两个 repo 混在一个 commit/步骤；C. 只做 CP6-A 暂停 CP6-B。 | 推荐方案让主仓库发布与 artifact 删除分别可验证；A 会让 staged 风险继续悬挂；B 难审计；C 不完成用户新增整改。 | 需要在 CP6-B 后另行处理 artifact repo 的本地变更状态；不自动 push。 | 任一切片前置检查失败则停止，不继续下一切片。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP5 八项推荐；后续 CP6 可执行 CP6-A allowlist-only local commit 与 CP6-B allowlist-only artifact docs/features cleanup。 |
| 备选方案 | `修改: <具体修改点>` 指明要调整的 DQ；或 `reject` 停止授权。 |
| 影响维度 | local Git history、dirty worktree allowlist、artifact docs/features cleanup、commit message、remote/runtime boundary、CR121 sequencing |
| 风险与回退 | CP6 前重跑 allowlist 检查；任何非 allowlist staged 文件或 artifact delete allowlist drift 出现即停止。 |
| 用户需决策事项 | 是否批准 `DQ-CP5-CR123-01..08`。 |
| 自动终验授权 | `auto_final_authorization: false`；CP5 approve 只授权未来 CP6-A local commit 与 CP6-B artifact cleanup，不执行当前 commit/delete，也不授权 push / runtime。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 precheck PASS | 待审查 | CP2 precheck | PASS。 |
| CP5 precheck PASS | 待审查 | CP5 precheck | PASS。 |
| dirty allowlist 可读 | 待审查 | CP5 context | ready。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | local commit 授权范围明确 | 待审查 | DQ-CP5-CR123-01 | allowlist-only。 |
| 2 | dirty worktree excluded 文件明确 | 待审查 | DQ-CP5-CR123-02 | scripts/tests 不提交。 |
| 3 | commit message 明确 | 待审查 | DQ-CP5-CR123-03 | 推荐 message。 |
| 4 | push/runtime 禁止项明确 | 待审查 | DQ-CP5-CR123-04 | 不授权。 |
| 5 | 后续收敛路径明确 | 待审查 | DQ-CP5-CR123-05 | CP6 -> CP8。 |
| 6 | `docs/features` 删除候选明确 | 待审查 | DQ-CP5-CR123-06 | 7 个目录 allowlist-only。 |
| 7 | CR102/103/104 重点处理边界明确 | 待审查 | DQ-CP5-CR123-07 | 默认保留，不在删除范围。 |
| 8 | CP6 切片顺序明确 | 待审查 | DQ-CP5-CR123-08 | CP6-A 主仓库 commit，CP6-B artifact cleanup。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认 CP5 | 待审查 | 本文件人工审查结果 | 等待用户回复。 |
| 可进入 CP6 执行切片 | 待审查 | DQ-CP5-CR123-01..08 | approve 后仍需 CP6-A / CP6-B 各自前置检查。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 context | `process/context/CP5-CR123-STAGED-DOCS-PUBLICATION-AUTHORIZATION-CONTEXT.yaml` | 待审查 |  |
| CP5 precheck | `process/checks/CP5-CR123-STAGED-DOCS-PUBLICATION-AUTHORIZATION-PRECHECK.md` | 待审查 |  |
| CR123 formal CR | `process/changes/CR-123-CR122-STAGED-DOCS-PUBLICATION-GATE-2026-06-23.md` | 待审查 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-23T11:21:01+08:00
- 修改意见：用户回复 approve，接受 `DQ-CP5-CR123-01..08` 推荐方案。
- 风险接受项：授权后续 CP6-A 仅提交 29 个 allowlisted staged docs 变更；授权 CP6-B 仅删除 7 个 allowlisted 非重点 `docs/features` CRxxx artifact 目录。CR102/CR103/CR104 默认保留重点复盘；不授权 push/runtime/CR121 cleanup。
