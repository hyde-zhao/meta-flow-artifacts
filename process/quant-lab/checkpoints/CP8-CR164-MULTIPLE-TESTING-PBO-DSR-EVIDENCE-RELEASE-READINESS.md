---
checkpoint_id: "CP8-CR164-MULTIPLE-TESTING-PBO-DSR-EVIDENCE-RELEASE-READINESS"
checkpoint_name: "CR164 Release Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-12T21:50:00+08:00"
reviewed_at: "2026-07-12T21:51:00+08:00"
reviewed_by: "user"
change_id: "CR-164"
release_decision: "READY_WITH_RISK"
release_context_ref: "process/release/RELEASE-CONTEXT-CR164-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.yaml"
context_ref: "process/context/CP8-CR164-MULTIPLE-TESTING-PBO-DSR-EVIDENCE-CONTEXT.yaml"
auto_check_result: "process/checks/CP8-CR164-MULTIPLE-TESTING-PBO-DSR-EVIDENCE-RELEASE-READINESS.result.json"
---

# CP8 CR-164 Release Readiness

## 自动预检摘要

| 项目 | 结论 | 量化证据 |
|---|---|---|
| 5 个 Story CP6/CP7 | PASS_WITH_RISK | S01-S04 PASS；S05 PASS_WITH_RISK；blocker=0 |
| 受影响代码与回归 | PASS | 109/109；compile PASS；diff-check PASS |
| 量化验收 | PASS | provenance 7/7；三消费者 3/3；授权越界 0；evidence 总账差异 0 |
| 全仓测试 | RISK | 1935 passed，14 个既有流程/产物卫生失败 |
| QA 隔离 | RISK | 用户禁止子 Agent，Host inline fresh verification，无独立 Agent/模型隔离 |
| Release execution | NOT_EXECUTED | 双仓 dirty；commit/push/tag/deploy/publish 均未授权、未执行 |

## Decision Brief

CR-164 已交付可计算的 BH/WRC/SPA/PBO/CSCV/raw-count DSR 证据、统一合同和保守聚合，并接入三个既有 admission 消费者。所有正向声明都受 sealed-lineage binding、candidate membership hash、method provenance 和 fail-closed validator 约束。

推荐以 `READY_WITH_RISK` 批准 CP8。该结论只关闭本地 CR 交付门禁，不等于真实发布或运行授权。

### 审批者摘要

- 本次确认服务的整体目标：交付可计算、可追溯且 fail-closed 的 multiple-testing / PBO / raw-count DSR evidence，并接入既有 admission 消费者。
- 推荐动作：批准 `READY_WITH_RISK`；CR-164 功能与量化验收已通过，0 blocker、0 waiver。
- approve 后会发生什么：回填 CP8 人工结果、同步 CR/state，并把本地工作流标记为 delivered-with-risk。
- approve 不授权什么：不授权真实数据、runtime、交易、部署、发布、commit、push、tag 或 package publish。
- 不确认会阻塞什么：CR-164 将保持 active，不能关闭 CP8 或形成最终交付结论。

### 决策分层

| 层级 | 内容 | 本次动作 |
|---|---|---|
| 必须用户决策 | 是否以 READY_WITH_RISK 接受功能结论和两项风险 | DQ-001、DQ-002 |
| 高风险策略确认 | runtime/data/trading/remote-write 继续 deny-default | 不随 CP8 扩权 |
| agent 默认处理 | approval 后回填 checkpoint、ledger、CR/state | Host 自动处理 |
| 仅审计记录 | 109/109、7/7、0 次越权、全仓 14 项既有失败 | 不新增用户决策 |

### Context Capsule Summary

CP8 capsule 位于 `process/context/CP8-CR164-MULTIPLE-TESTING-PBO-DSR-EVIDENCE-CONTEXT.yaml`，`read_profile=process-lite`。默认读取策略使用 state/current/read-policy 与 CR summary，再按需读取 CR164 release context；全文档读取仅在 capsule 缺失、字段冲突、人工审计、deep review 或 schema 校验失败时允许，不默认展开全部 HLD/LLD/diff/transcript。

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---:|---:|---:|---|
| 五个 CP7 results | scanned | 2 | 2 | 两项风险纳入 DQ-002 |
| Aggregate quality / QAC | scanned | 1 | 1 | 功能与 claim ceiling 纳入 DQ-001 |
| Release context / docs | scanned | 1 | 0 | remote delivery 分流为另行授权，不隐式并入 CP8 |
| Open blockers / waivers | scanned | 0 | 0 | N/A：blocker=0，waiver=0 |

### 待人工决策清单

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP8-CR164-001 | scope | 是否接受当前功能、量化验收与 claim ceiling？ | approve CP8 / READY_WITH_RISK | reject 或 request changes | 批准可按冻结范围闭环；拒绝可保留进一步修改空间但延迟交付 | 不授权 runtime/remote write | request changes 时回到相应 Story；范围变化回 CP2/CP3/CP5 |
| DQ-CP8-CR164-002 | risk_acceptance | 是否接受两项剩余风险？ | 接受 inline QA 独立性限制与 14 项既有全仓失败，后续分开治理 | 解除 no-subagent 后独立复核；或扩展全仓治理 CR | 接受可避免将既有债务塞入本 CR；不接受可提高隔离/全仓洁净度但扩大时间和范围 | 风险已披露，0 CR164 blocker | 独立性不接受则重新 CP7；全仓清零要求则新建/扩展治理 CR |

用户需决策事项 summary：共 2 项：`DQ-CP8-CR164-001` 为发布范围与 claim ceiling 接受，`DQ-CP8-CR164-002` 为两项剩余风险接受；repository delivery 不在本表中隐式授权。

### CP8 后续跟踪分流表

| 分类 | 内容 | 状态 / 动作 |
|---|---|---|
| 关闭范围 | CR164 五个 Story、统计 evidence、三个既有消费者投影 | CP8 approve 后关闭 |
| 不授权范围 | 真实数据、runtime、simulation/paper/live、交易、commit/push/tag/deploy/publish | 继续 deny-default |
| 风险接受项 | R-CR164-INLINE-INDEPENDENCE；R-REPO-GLOBAL-14 | accepted by user under DQ-002 |
| 后续 CR 候选项 | 独立 Agent 复核；全仓 14 项卫生治理；automatic block selector；effective trials/C1 | candidate，不自动启动 |
| 取消 / deferred | real-data characterization、runtime integration | deferred，需新授权与新 CR |

双仓 commit/push 不包含在上述批准中；如需交付到远端，必须另行明确授权“paired source + artifact commit/push”。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP5 已批准 | PASS | `process/checkpoints/CP5-CR164-ALL-STORIES-LLD-BATCH.md` |
| 五个 CP6 完成 | PASS | `process/checks/CP6-CR164-S01..S05.result.json` |
| 五个 CP7 完成 | PASS_WITH_RISK | `process/checks/CP7-CR164-S01..S05.result.json` |
| Quality 与 release docs 完整 | PASS | `docs/quality/*CR164.md`、`docs/release/*CR164*.md` |
| CP8 capsule / release context 存在 | PASS | 本门禁 frontmatter 引用 |

## Checklist

| # | 检查项 | 结果 | 说明 |
|---|---|---|---|
| 1 | 四项原评审缺口全部闭合 | PASS | QAC、raw-count 声明、fixed window、disagreement priority |
| 2 | mandatory evidence fail-closed | PASS | missing/orphan/conflict/untrusted 均不得 PASS |
| 3 | raw/effective DSR non-alias | PASS | effective fields unavailable |
| 4 | 三消费者保守投影 | PASS | 3/3，不改善更差状态 |
| 5 | CR155 negative regression | PASS | blocked；paper_candidate=false |
| 6 | Forbidden operations | PASS | 0 次真实数据/运行时/外部写操作 |
| 7 | 独立性声明准确 | PASS_WITH_RISK | fresh tests，但同 Host inline |
| 8 | 全仓测试状态准确 | PASS_WITH_RISK | 14 项既有失败未隐藏、未纳入 CR164 PASS |
| 9 | Release overclaim | PASS | READY_WITH_RISK != RELEASED；无 commit/push/deploy |

## Exit Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| 自动预检通过 | PASS_WITH_RISK | 0 blocker，0 waiver，2 risks |
| 人工接受 READY_WITH_RISK | PASS | 用户已明确 `approve CP8` |
| 不授权边界明确 | PASS | runtime/data/trading/remote write deny-default |
| CR 与 state 同步关闭 | PASS | 本次由 Host 执行 status-sync |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| Verification / test / review / fixes | `docs/quality/*CR164.md` | ready |
| Release context | `process/release/RELEASE-CONTEXT-CR164-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.yaml` | ready |
| Release docs | `docs/release/*CR164-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md` | ready |
| CP6 / CP7 machine results | `process/checks/CP6-CR164-*`、`process/checks/CP7-CR164-*` | validated / ledgered |

## 人工审查结果

- 结论：`approved / READY_WITH_RISK`
- 审查人：user
- 审查时间：2026-07-12T21:51:00+08:00
- 接受决策：`DQ-CP8-CR164-001`、`DQ-CP8-CR164-002`
- 风险接受：`R-CR164-INLINE-INDEPENDENCE`、`R-REPO-GLOBAL-14`
- 修改意见：无；用户原文为 `approve CP8`。
- 不授权项：真实数据、runtime、simulation/paper/live、交易、commit、push、tag、deploy、publish 继续 deny-default。
