---
checkpoint_id: "CP2-CR162-PRODUCT-BASELINE-REFRESH"
checkpoint_name: "CR162 Product Baseline Refresh Scope"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-10T12:30:31+00:00"
reviewed_at: "2026-07-10T20:43:29+08:00"
reviewed_by: "user"
change_id: "CR-162"
context_ref: "process/context/CP2-CR162-PRODUCT-BASELINE-REFRESH-CONTEXT.yaml"
auto_check_result: "process/checks/CP2-CR162-PRODUCT-BASELINE-REFRESH.result.json"
---

# CP2 CR162 Product Baseline Refresh Scope

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---:|---:|---|
| `process/checks/CP0-CR-162-BOOTSTRAP.result.json` | PASS | 0 | CR162 已登记且无 active CR 冲突。 |
| `process/checks/CP1-CR162-PRODUCT-BASELINE-COMPLETENESS.result.json` | PASS | 0 | 已确认 CR161 承诺的 9 个基线文档未被刷新。 |
| `process/checks/CP2-CR162-PRODUCT-BASELINE-REFRESH.result.json` | PASS / pending human gate | 0 | 范围、历史 reframe、不授权边界和 checker follow-up 已收敛为决策项。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 修复 CR161 已关闭交付中遗漏的产品基线追溯，使后续工作可从当前基线发现七对象 evidence contract、fail-closed ceiling、CR155 negative regression 和 deferred producers。 |
| 推荐动作 | `approve`：仅增量刷新 9 个已承诺文档；保留 CR161 历史关闭状态，并把通用 CP8 checker 作为独立后续候选。 |
| approve 后会发生什么 | Host Orchestrator 更新 6 个产品文档和 3 个 feature 文档，追加修订记录，建立静态追溯验证，随后自动推进 CP7 和 CP8。 |
| approve 不授权什么 | 不授权 HLD/ADR 语义变化、代码/测试/schema/checker、研究引擎 instrumentation、统计/经济/容量计算、数据/NAS/provider/credential/broker、simulation/paper/live/trading、external framework、Git remote、publish。 |
| 不确认会阻塞什么 | 阻塞 CR162 的产品基线写入；CR161 继续保持“设计已交付但当前基线缺失追溯”的状态。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR162-PRODUCT-BASELINE-REFRESH-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 CR162 summary、route plan 和 CP1；仅为逐文档映射或人工审计展开正式文档。 |
| 全文档读取 | `RE-20260710T122812Z0000-cfa7c029`，用于验证 9 个原基线未含 CR161 专属内容。 |
| 缺失 / waived 理由 | N/A。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CR161 post-closure review | 当前用户评审 | scanned | 1 | 1 | 缺失的产品基线刷新进入 DQ-CP2-CR162-001。 |
| CR161 formal CR | 文档处理决策 / affected docs | scanned | 2 | 2 | 9 文档承诺和历史正确性进入 DQ-001/002。 |
| CP1 completeness | CP1 result | scanned | 3 | 2 | 无 runtime 授权作为不授权边界；9 文档范围和 contract truth 进入决策。 |
| CP8 checker gap | 用户建议 | scanned | 1 | 1 | 通用 checker 不与当前文档刷新混做，进入 DQ-CP2-CR162-003。 |
| STATE / CR tracking | `STATE.current.json` / CR ledger | scanned | 0 | 0 | 无 active/blocked formal CR 冲突；legacy tracking errors 与本 CR 无关。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 3 | 范围、历史 reframe 和通用 checker 分拆均进入下方决策表。 |
| 高风险策略确认 | 0 | 本 CR 不授权 runtime、数据、凭据、交易或发布。 |
| agent 默认处理 | 3 | 仅增量编辑；保留旧基线；CP3-CP6 按 route N/A。 |
| 仅审计记录 | 2 | CR161 independent-verifier waiver 和全局 legacy CR tracking errors 不在本 CR 重开。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP2-CR162-001` | scope | 是否刷新 CR161 明确承诺的全部 9 个基线文档？ | 刷新 6 个产品文档和 3 个 `factor-research-loop` feature 文档。 | A. 只刷新 6 个产品文档；B. 不刷新，仅登记 follow-up。 | 推荐方案满足原始文档处理决策且避免 feature baseline 继续漂移；A 留下 3 个明确承诺缺口；B 保留已知缺陷。 | 中风险；后续 FU 无法完整追溯时会重现本次问题。 | 若发现某 feature 文件已由另一 active CR 独占，回到 CP2 缩小或拆分该文件。 |
| `DQ-CP2-CR162-002` | scope | 如何修正已关闭 CR161 的当前需求真相？ | 以 CR162 更新当前基线；完成后标记 CR161 `historical_baseline_status=reframed`，不重开 CP3/CP7/CP8。 | A. 重开 CR161 并重新走门禁；B. 不做 historical reframe。 | 推荐方案保留历史审计并清晰表达新事实；A 重复已完成设计治理；B 让历史交付继续被误读为基线已刷新。 | 中风险；避免篡改既有 waiver、验证结论与远端历史。 | 若刷新发现 HLD/ADR 本身错误，停止并创建 architecture CR，不在 CR162 修正。 |
| `DQ-CP2-CR162-003` | follow_up_tracking | 是否在本 CR 实现通用 CP8 `product-baseline-refresh` checker？ | 不实现；在 CR162 收敛时登记 `FU-CR162-001` candidate。 | A. 当前实现 checker 并重路由 CP3-CP6；B. 不登记控制改进。 | 推荐方案把当前纠错保持为文档范围；A 会扩大到 Meta Flow 工具实现；B 会保留重复缺口风险。 | 低到中风险；不会影响本轮 9 文档的可验证刷新。 | 若后续连续出现同类遗漏，提升 FU-CR162-001 为正式 process CR。 |

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 3 |
| 必须用户决策 | `DQ-CP2-CR162-001`、`DQ-CP2-CR162-002`、`DQ-CP2-CR162-003` |
| 推荐回复 | `approve` |
| 如果你回复 approve | 认可全部 9 个文档刷新、CR161 historical reframe、不在本轮实现通用 checker；随后自动进入文档更新与 CP7/CP8。 |
| 不表示授权 | 不表示授权代码/测试/schema/checker、研究引擎、统计计算、真实数据、运行时、交易、凭据、外部框架、Git remote 或 publish。 |
| 修改 | 回复 `修改: <决策 ID> <内容>`；Host 会回到 CP2 更新范围。 |
| reject | CR162 保持 active / CP2 pending，不修改基线。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR162 formal CR exists | PASS | `process/changes/CR-162.md` | 产品基线 correction 已登记。 |
| CP0 route plan exists | PASS | `process/checks/CP0-CR162.route-plan.json` | CP1/CP2/CP7/CP8 applicable；CP3-CP6 N/A。 |
| CP1 completeness passed | PASS | `process/checks/CP1-CR162-PRODUCT-BASELINE-COMPLETENESS.result.json` | 9 文档、无 overclaim 与不授权边界已确认。 |
| CP2 capsule exists | PASS | `process/context/CP2-CR162-PRODUCT-BASELINE-REFRESH-CONTEXT.yaml` | compact context ready。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 9 文档刷新范围完整 | PASS | DQ-CP2-CR162-001 | 用户批准 6 product + 3 feature 的完整增量刷新。 |
| 2 | CR161 不被重开或改写 | PASS | DQ-CP2-CR162-002 | 用户批准以 reframe 保留历史关闭记录。 |
| 3 | 静态文档范围不被 checker 实现扩大 | PASS | DQ-CP2-CR162-003 | 用户批准仅登记 `FU-CR162-001` candidate。 |
| 4 | `typed_unavailable` 不被误写为 computed proof | PASS | CP1 / capsule requirements | 后续文档更新必须保留 fail-closed ceiling。 |
| 5 | 无任何新授权 | PASS | authorization boundary | approve 不扩大 deny-by-default 边界。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认范围和 reframe 路径 | PASS | 本 checkpoint / Gate ledger | 2026-07-10 用户回复 `批准`。 |
| 不授权边界保持 | PASS | Decision Brief | approve 不授权的对象保持不变。 |
| 可进入产品基线文档更新 | PASS | CP2 approved | 自动推进至 route 的 N/A 记录和文档更新。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR162 formal CR | `process/changes/CR-162.md` | PASS | 五维度影响与文档决策已写入。 |
| CP1 result | `process/checks/CP1-CR162-PRODUCT-BASELINE-COMPLETENESS.result.json` | PASS | baseline completeness passed。 |
| CP2 context | `process/context/CP2-CR162-PRODUCT-BASELINE-REFRESH-CONTEXT.yaml` | PASS | compact capsule。 |
| CP2 result | `process/checks/CP2-CR162-PRODUCT-BASELINE-REFRESH.result.json` | PASS | awaiting human decision。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-10T20:43:29+08:00
- 修改意见：无；按推荐方案执行。
- 已接受决策项：`DQ-CP2-CR162-001`、`DQ-CP2-CR162-002`、`DQ-CP2-CR162-003`
