---
checkpoint_id: "CP2-CR050-R2"
checkpoint_name: "CR-050 Explicit Paired Fast-Forward Merge Product Baseline R2"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-16T14:05:22Z"
reviewed_by: "user"
reviewed_at: "2026-07-16T14:23:16Z"
supersedes_checkpoint_ref: "process/checkpoints/CP2-CR050-GIT-BRANCH-BASELINE.md"
auto_check_result: "process/checks/CP2-CR050-GIT-BRANCH-BASELINE-R2.result.json"
context_ref: "process/context/CP2-CR050-GIT-BRANCH-CONTEXT-R2.yaml"
decision_brief_profile: "compact"
target:
  phase: "requirement-clarification"
  artifacts: ["process/docs/product/USE-CASES.md", "process/docs/product/REQUIREMENTS.md", "process/docs/product/SCENARIOS.yaml", "process/docs/product/TEST-MATRIX.md", "process/docs/product/STORY-MAP.md", "process/docs/product/MVP-SCOPE.md", "process/docs/product/RELEASE-SLICES.md", "process/docs/product/BACKLOG.md"]
---

# CP2 R2 — CR-050 显式成对 fast-forward merge 产品基线

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR050-GIT-BRANCH-USE-CASE-R2.result.json` | PASS | 0 | 4 个 Use Case、17 个场景、4 个 Story 完成追踪 |
| `process/checks/CP2-CR050-GIT-BRANCH-BASELINE-R2.result.json` | PASS | 0 | merge 范围、安全、partial 与授权边界可判定 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 为 project 与 artifact 两仓提供 `open → publish → 显式 merge → finish` 的确定、可恢复、可审计 Git CR 分支生命周期。 |
| 本轮为何回到 CP2 | 用户要求增加 merge，推翻了 CP2 R1 的 DQ-03“工具不 merge”；既有 HLD/ADR 只能保留为 changes-requested 历史证据。 |
| 推荐动作 | `approve`：接受 CP2-R2-DQ-01..04，随后生成 HLD/ADR/CP3 R2。 |
| approve 后会发生什么 | 仅推进设计、Story 规划和下一个 CP3 人工门禁；不会实现代码或改变真实 Git refs。 |
| approve 不授权什么 | stage/commit/push、CR branch create/delete、默认分支写入、真实 merge、force/history rewrite、forge API、凭据、runtime、production write。 |
| 不确认会阻塞什么 | CR-050 保持在 requirement-clarification；不得生成 R2 架构、Story 设计或实现。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule | `process/context/CP2-CR050-GIT-BRANCH-CONTEXT-R2.yaml` |
| read_profile | `compact` |
| 默认读取策略 | capsule first；默认只消费 `must_read` / `allowed_reads`，全文产品文档仅按需读取。 |
| 全文档读取 | 本轮范围变更的全文读取已逐项追加到 `process/state/READ-EXPANSION-LEDGER.ndjson`。 |
| 产品范围 | UC-GB-001..004；REQ-GB-001..014/C001..004/NF001..003；TC-GB-001..017；ST-GB-001..004 |
| 生命周期 | `open → publish → explicit merge → finish`；任何相邻动作不得隐式 merge |
| merge 核心 | 两仓先全部 preflight，按 artifact→project，且只能 fast-forward |
| partial 核心 | 保留两仓 CR branches、阻断 finish、不自动回滚已经成功的默认分支更新 |
| 调度披露 | 用户要求不使用子 Agent；本轮由 Host inline 完成，不是独立 meta-pm 或平台 attestation |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| state | `process/state/STATE.current.json` | scanned | 1 | 0 | 路由回 CP2，不形成额外产品选择 |
| inline handoff | `process/handoffs/CR050-CP1-CP2-META-PM-R2.md` | scanned | 4 | 4 | scope/order/partial/authz 全部进入 DQ |
| auto checks | CP1/CP2 R2 result | scanned | 0 | 0 | blocker=0、waiver=0 |
| discussion | `process/discussions/CP2-CR050-SCENARIO-DISCUSSION-LOG.md` | scanned | 2 | 2 | SGA-GB-03 reopened；新增 SGA-GB-06 |
| product baseline | 八个 `process/docs/product/**` 文件 | scanned | 4 | 4 | 四阶段、ff-only、partial、授权边界 |
| stale design | HLD/ADR/CP3 v1.0 | scanned | 2 | 0 | 仅作为 changes-requested 历史输入，不作为 R2 设计结论 |
| repository evidence | `git_sync.py` 与 bare-repo fixture | scanned | 1 | 0 | 证明 native Git 可扩展；实现细节留给 CP3/CP5 |
| user input | merge 请求、approve、no-subagent | scanned | 3 | 4 | scope change 已转化为四项正式决策 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | CP2-R2-DQ-01..04 |
| 高风险策略确认 | 3 | 默认分支写、跨仓 partial、不自动回滚 |
| agent 默认处理 | 5 | result 字段、错误文案、timeout、fixture 名称、CLI help 排版 |
| Deferred | 3 | forge/API merge adapter、merge queue receipt、Git Town/stacked branch adapter |
| 仅审计记录 | 4 | no-subagent inline fallback、CP3 v1.0 changes-requested、源码仓 clean、当前无真实 Git mutation |
| 明确禁止 | 7 | 隐式 merge、merge commit、rebase、squash、force、自动冲突解决、history rewrite |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-R2-DQ-01 | scope | merge 是否成为 lifecycle 的独立显式动作？ | 增加独立 `merge`；`publish` 只推已提交 CR tip，`finish` 只证明并清理，不得隐式 merge。 | A：继续由外部平台 merge；B：让 finish 隐式 merge。 | 推荐方案补齐 CLI 生命周期又保持每个动作单一职责；A 保守但未满足当前目标；B 隐藏默认分支写副作用。 | 隐式 merge 会扩大授权并模糊失败恢复。 | 若原生 Git 无法遵守远端策略，`merge` fail closed，切换到后续 forge adapter CR；不得退化为隐式 merge。 |
| CP2-R2-DQ-02 | architecture | 两仓 merge 的 preflight 与执行顺序是什么？ | 对 project + artifact 先完成全部无写 preflight，再按 `artifact → project` 串行 fast-forward。 | A：project→artifact；B：并行；C：单仓。 | artifact-first 先固化过程证据，且顺序确定；代价是 project 失败时形成 partial。并行无法提供可解释顺序。 | Git 无跨仓事务，任何顺序都有一仓成功另一仓失败的窗口。 | 只有存在可验证跨仓事务服务才重评；当前必须逐仓 terminal result。 |
| CP2-R2-DQ-03 | risk_acceptance | artifact 成功而 project 失败时如何处理？ | 整体 `PARTIAL`；保留两仓 CR branches、阻断 finish、不自动回滚 artifact 默认分支；恢复时重新 fetch/观测后只继续未完成仓。 | A：自动回滚 artifact；B：继续 finish；C：删除成功仓 branch。 | 推荐方案保留真实完成 fact，避免用第二次远端写放大风险；代价是需要显式 resume。自动回滚可能同样被保护规则拒绝。 | partial 期间两仓默认分支暂时不一致。 | 只有用户另行授权且具备可验证安全反向更新时，才讨论补偿；当前 CR 禁止自动补偿。 |
| CP2-R2-DQ-04 | security | 如何约束默认分支写与远端保护？ | CP2 只批准产品契约；每次真实 merge 仍需 operation-specific 明示授权。保护规则、hook 或远端拒绝必须保持原状并报告 `BLOCKED/PARTIAL`。 | A：CP2 一次性授权所有未来默认分支写；B：现在实现 forge API/merge queue adapter；C：仍完全外部 merge。 | 推荐方案权限最小并兼容当前原生 Git MVP；A 授权过宽；B 扩大依赖与平台范围；C 不满足当前功能目标。 | 原生 push 可能被远端策略拒绝，不能伪装为成功或自动绕过。 | 若策略要求 PR/merge queue，原生命令 fail closed，另立 adapter CR 并验证 receipt。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP2-R2-DQ-01..04 推荐方案 |
| 备选方案 | `修改: CP2-R2-DQ-xx <具体要求>`；或 `reject` 停止 CR-050 |
| 风险与回退 | 所有真实 Git 写仍未授权；CP3 R2 前可继续修改产品范围 |
| 用户需决策事项 | CP2-R2-DQ-01、CP2-R2-DQ-02、CP2-R2-DQ-03、CP2-R2-DQ-04 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP3 v1.0 已记录 changes_requested | PASS | CP3 checkpoint；不覆盖旧 result |
| CR 已路由回 requirement-clarification | PASS | CR-050 frontmatter / checkpoint index |
| 八个产品基线增量更新 | PASS | v1.4..1.8 current revisions |
| CP1/CP2 R2 自动预检 | PASS | 两份 R2 result；blocker=0 |
| Context ready | PASS | CP2 R2 capsule |

## Checklist

| # | 检查项 | 当前结果 | 证据 / 待审查意见 |
|---:|---|---|---|
| 1 | 四阶段 lifecycle 与非隐式 merge 边界 | approved | DQ-01；用户批准推荐方案 |
| 2 | preflight-all、artifact→project、ff-only | approved | DQ-02；用户批准推荐方案 |
| 3 | partial 保留事实、分支与 finish 阻断 | approved-with-risk | DQ-03；用户明确接受非原子 partial 风险 |
| 4 | scope/implementation/真实 default write 授权分离 | approved | DQ-04；仅批准产品契约，未授权真实写入 |
| 5 | UC/REQ/TC/ST/MVP/Slice 全链追踪 | PASS | TEST-MATRIX / STORY-MAP |
| 6 | inline fallback 与不授权边界诚实 | PASS | handoff / context / CR |

## Exit Criteria

| 条目 | 当前结果 | 证据 |
|---|---|---|
| R2 产品基线 blocker=0 | PASS | CP2 R2 result |
| 四项范围/风险决策关闭 | PASS | CP2-R2-DQ-01..04 已批准 |
| 用户明确允许进入 R2 设计 | PASS | 2026-07-16 用户回复“批准” |
| 真实 Git ref mutation=0 | PASS | 源码仓 clean；本轮仅外置 process 变化 |

## Deliverables

| 交付物 | 路径 | 当前结果 |
|---|---|---|
| 产品基线 | `process/docs/product/**` | R2 ready |
| CP1/CP2 结果 | `process/checks/*CR050*R2.result.json` | PASS |
| Context | `process/context/CP2-CR050-GIT-BRANCH-CONTEXT-R2.yaml` | ready-for-human-gate |
| Discussion | `process/discussions/CP2-CR050-SCENARIO-DISCUSSION-LOG.md` | R2 complete |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-16T14:23:16Z
- 修改意见：无；批准 CP2-R2-DQ-01..04 的推荐方案，并允许进入 HLD/ADR/CP3 R2。
- 风险接受项：接受 CP2-R2-DQ-03 的跨仓非原子 `PARTIAL`、保留成功事实且不自动补偿；接受 native Git 可能被 branch protection 拒绝并 fail closed。
- 授权边界：本批准不授权源码实现、stage/commit/push、真实 CR branch/default branch mutation、merge、branch delete、forge API、凭据、force 或 history rewrite。

## 可接受回复

- `approve`
- `修改: CP2-R2-DQ-xx <具体修改点>`
- `reject`
