---
checkpoint: CP8
checkpoint_id: CP8-CR171-STAGE3-LAUNCH-RELEASE-READINESS
cr_id: CR-171
status: approved
gate_profile: runtime-high-risk
context_ref: process/context/CP8-CR171.context.json
result_ref: process/checks/CP8-CR171-STAGE3-LAUNCH-RELEASE-READINESS.result.json
release_context_ref: process/release/RELEASE-CONTEXT-CR171-STAGE3-LAUNCH-DECISION-GATE.yaml
approved_at: "2026-07-15T22:35:00+08:00"
approved_by: user
---

# CP8 — CR-171 Decision-Gate Release Readiness

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 关闭 CR-171 的决策/设计/验证记录，不是启动 Stage 3。 |
| 推荐动作 | `approve`，接受 `READY_WITH_RISK`：设计合同已完成，但 current entry 仍为 `insufficient_for_current_entry`。 |
| approve 后会发生什么 | 只关闭 CR-171；五字段授权、C1-C3 binding、C4 rework、FU-006 与真实 activation 继续保持 OPEN/独立 follow-up。 |
| approve 不授权什么 | lake/NAS/provider/credential、write、computation、runtime/trading、aggregate、CR155 promotion、repair/rerun、publish/deploy 或 Git remote write。 |
| 不确认会阻塞什么 | 阻塞 CR-171 的正式关闭；Stage 3 仍保持未启动。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP8-CR171.context.json` |
| capsule 状态 | ready / compact / context check PASS |
| read_profile | compact |
| 默认读取策略 | capsule-first；CP7 result、quality summary、revalidation report 与 release context 为最小输入。 |
| 全文档读取扩展 | CP7 已记录 1 条静态 legacy-marker 审计；无外部读取。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---|
| CP7 verification | CP7 result/evidence | complete | 5 | 1 | release readiness/risk acceptance |
| release context/fact diff | release context | complete | 3 | 1 | `READY_WITH_RISK`, not `READY` |
| workspace status | dual worktree status | complete | 2 | 1 | no push/publish authorized |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 3 | 下方 CP8 DQ。 |
| 高风险策略确认 | 1 | READY_WITH_RISK 不得被解释为 Stage 3 entry readiness。 |
| agent 默认处理 | 0 | 无。 |
| 仅审计记录 | 5 | OI-001..005 继续 OPEN。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|
| `DQ-CP8-CR171-001` | risk_acceptance | 是否关闭 CR-171 为 `READY_WITH_RISK`？ | 关闭并保留 OI-001..005 OPEN。 | `reject`，保持 active 以要求额外文档修改。 | 保守地关闭 decision gate，不伪装 current entry。 | approve 不启动 Stage 3。 | reopen/rework 走新 CR 或 CP3 reopen。 |
| `DQ-CP8-CR171-002` | follow_up_tracking | 是否把 Activation、C4、FU-006 与 revalidation gaps 留为 follow-up？ | 明确留在 follow-up，不在本 CR 修复。 | 将其中一项提升为新 CR（需新授权）。 | 防止 scope creep。 | 不得把 CP8 解释为 lake/computation authorization。 | 新 CR 才可改变 owner/authorization。 |
| `DQ-CP8-CR171-003` | security | 是否接受无 push/publish 的本地 readiness 记录？ | 接受；不 push/publish。 | 另行提出双仓提交/推送授权。 | 避免 dirty worktree 下误发布。 | 当前两个 worktree 均 dirty。 | 单独授权后再处理双仓交付。 |

### 用户需决策事项

本轮待人工决策项：3：`DQ-CP8-CR171-001`（READY_WITH_RISK 关闭）、`DQ-CP8-CR171-002`（follow-up 保留）和 `DQ-CP8-CR171-003`（不推送/不发布）。回复 `approve` 表示接受三项推荐；`修改: <DQ-ID> + <具体修改>` 或 `reject` 将保持 CR-171 不关闭。

### CP8 后续跟踪分流表

| 分流类别 | 项目 | 状态 | 处理方式 |
|---|---|---|---|
| 关闭范围 | CR-171 design/verification contract | ready-for-close | CP8 批准后关闭，不代表 Stage 3 启动。 |
| 不授权范围 | real lake/read/write/computation/runtime/trading/publish | not-authorized | 必须新授权。 |
| 风险接受项 | current-entry=`insufficient_for_current_entry` | accepted-risk | `READY_WITH_RISK`，不能改写为 READY/entry-ready。 |
| 后续 CR 候选项 | OI-001..005 | candidate | activation/C4/FU-006/revalidation 通过 future CR 或既有 lane。 |
| 取消 / deferred | push/publish/real activation | deferred | 本门禁不执行。 |

## Entry Criteria

- [x] CP7 PASS，evidence index/return packet 可读。
- [x] release context 与 compact release documents 已生成。
- [x] fact diff 包含 negative/deferred 项，release decision 因而为 `READY_WITH_RISK`。
- [x] 没有真实数据、runtime、write 或交易操作。

## Checklist

| # | Item | Result | Evidence |
|---:|---|---|---|
| 1 | CP7 evidence/return integrity | PASS | CP7 result/evidence index |
| 2 | current-entry insufficiency 不被伪装为 readiness | PASS | revalidation report |
| 3 | 5 个 OPEN item 与 FU-006 guard 被保留 | PASS | HLD §21 / release context |
| 4 | no real release/push authorization | PASS | release context / workspace status |

## Exit Criteria

- [ ] 用户接受 `READY_WITH_RISK` 与 3 项 DQ。
- [ ] 用户确认 CP8 不代表 Stage 3、data access、computation 或真实发布授权。

## Deliverables

- `process/release/RELEASE-CONTEXT-CR171-STAGE3-LAUNCH-DECISION-GATE.yaml`
- `process/docs/release/*-CR171.md`
- `process/checks/CP8-CR171-STAGE3-LAUNCH-RELEASE-READINESS.result.json`

## 人工审查结果

状态：`approved`

用户于 2026-07-15 批准 `DQ-CP8-CR171-001..003`：以 `READY_WITH_RISK` 关闭 CR-171、保留 OI-001..005 follow-up、且不进行提交/推送/发布。该批准不启动 Stage 3 或任何真实数据/计算行为。
