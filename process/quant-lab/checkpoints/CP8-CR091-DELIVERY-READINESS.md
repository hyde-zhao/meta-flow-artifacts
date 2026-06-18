---
checkpoint_id: "CP8"
checkpoint_name: "CR091 QMT Strategy Runner Delivery Readiness"
type: "auto_then_manual"
status: "pending"
owner: "host-orchestrator"
created_at: "2026-06-18T15:24:12+08:00"
reviewed_by: ""
reviewed_at: ""
auto_check_result: "process/checks/CP8-CR091-DELIVERY-READINESS.md"
target:
  phase: "documentation"
  story_id: "CR091"
  artifacts:
    - "process/release/RELEASE-CONTEXT-CR091.yaml"
    - "process/context/CP8-CR091-DELIVERY-CONTEXT.yaml"
    - "process/docs/release/RELEASE-NOTES-CR091.md"
    - "process/docs/release/DEPLOY-CHECKLIST-CR091.md"
    - "process/docs/release/ROLLBACK-CR091.md"
    - "process/docs/release/MIGRATION-CR091.md"
    - "process/docs/release/FEEDBACK-CR091.md"
---

# CP8 CR091 QMT Strategy Runner 交付就绪人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP8-CR091-DELIVERY-READINESS.md` | PASS_WITH_RISK | 0 | 推荐进入 CP8 人工风险接受；剩余风险为真实 runtime 未验证、历史 cr-tracking 旧账和 future evidence output gate。 |

## Decision Brief

CR091 当前交付是 QMT strategy runner 的离线合同、设计、实现切片和验证收口。它证明多因子优先 adapter、通用 adapter 扩展、package/cache intake、fake readonly gateway 和 redacted evidence 在离线 fixture 范围内可用；它不证明真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 可用，也不授权 NAS、凭据、账户或交易动作。

推荐 CP8 接受 `READY_WITH_RISK`，关闭 CR091 当前离线交付为 `closed-current-delivery / READY_WITH_RISK`。后续真实 runtime、NAS package exchange、submit/cancel、simulation/live 和历史账本治理均保留为独立候选，不自动启动。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP8-CR091-DELIVERY-CONTEXT.yaml` |
| capsule 状态 | `ready-for-human-review` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次；本轮只读取 CP7 summary、quality summary、CR091 变更单和 release context |
| 缺失 / waived 理由 | N/A |
| Release Context | `process/release/RELEASE-CONTEXT-CR091.yaml` |
| CP6 | `PASS` |
| CP7 | `PASS_WITH_RISK` |
| CP8 自动预检 | `PASS_WITH_RISK` |
| 推荐 | `READY_WITH_RISK` |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 本轮 CP8 决策在本 checkpoint 中新增。 |
| 自动预检结果 | `process/checks/CP8-CR091-DELIVERY-READINESS.md` | scanned | 3 | 3 | `R-CR091-CP7-001..003` 纳入风险接受与分流。 |
| CP7 验证结果 | `process/checks/CP7-CR091-QMT-STRATEGY-RUNNER-VERIFICATION-DONE.md` | scanned | 3 | 3 | CP7 风险与不授权项纳入 DQ。 |
| 下游正式产物 | `docs/quality/CR091-QMT-STRATEGY-RUNNER-VERIFICATION-REPORT.md` / `TEST-REPORT` / `REVIEW` / `FIXES` | scanned | 5 | 4 | findings none-found；风险和 follow-up 合并为 4 类。 |
| Release Context | `process/release/RELEASE-CONTEXT-CR091.yaml` | scanned | 5 | 5 | scope、risk、non-authorized、follow-up、rollback 均纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | 用户要求继续推进 CP8；未授权 runtime。 |
| 委托 Agent 交还摘要 | `process/handoffs/META-QA-CR091-OFFLINE-RUNNER-VERIFY-2026-06-18.md` | scanned | 1 | 1 | QA 子 agent 完成 CP7，结论为 `PASS_WITH_RISK`。 |
| discussion log / checkpoint | N/A | n/a | 0 | 0 | CP8 无新增场景 / 架构灰区讨论。 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP8-CR091-01 | risk_acceptance | 是否接受 CR091 当前 CP8 结论？ | 接受 `READY_WITH_RISK`，关闭 CR091 当前离线 runner 交付。 | 补真实 runtime gate 后再关闭；reject 回退到 CP7 pending CP8。 | 推荐方案与 CP6/CP7 证据一致，可收口离线合同；补 runtime 会进入当前未授权范围。 | 接受后仍不声明真实 runtime ready。 | 用户要求真实验证或 reject 时回退。 |
| DQ-CP8-CR091-02 | scope | CR091 关闭范围是否限定为研究 / 设计 / 离线实现 / fixture 验证？ | 是，仅关闭离线 runner 合同和实现切片。 | 扩大到真实 QMT runner；扩大到 NAS package exchange；扩大到 submit/cancel。 | 推荐方案边界清楚；扩大范围需要新设计和授权。 | 真实交易主机和策略可交易性仍未证明。 | 用户明确扩大范围时另起 gate / CR。 |
| DQ-CP8-CR091-03 | runtime_authorization | CP8 approve 是否授权 QMT/MiniQMT/XtQuant/gateway/runner、NAS、凭据、账户或交易动作？ | 不授权。 | 授权只读 runtime smoke；授权 NAS package exchange；授权 submit/cancel。 | 推荐方案符合当前安全边界；备选风险高且需要独立门禁。 | 后续真实验证仍 blocked。 | 需要真实动作时进入独立 runtime authorization。 |
| DQ-CP8-CR091-04 | follow_up_tracking | 后续候选如何分流？ | 保留 runtime readonly smoke、NAS package exchange、order-write、ledger hygiene 为候选，不自动启动。 | 本轮强制启动某个候选；取消全部候选。 | 推荐方案保留路线且不扩大执行面；立即启动会越过当前 CP8 授权。 | 后续仍需用户明确选择和新门禁。 | 用户指定候选时另起 CR / gate。 |
| DQ-CP8-CR091-05 | implementation | 是否接受当前无需额外用户手工 runtime 验证即可关闭离线切片？ | 接受；CP8 只关闭离线验证通过的 runner 合同。 | 要求用户在交易主机手工验证后再关；取消 runner 交付。 | 推荐方案不要求用户执行未授权真实环境；手工验证会引入 runtime / 凭据 / 账户边界。 | 无真实 runtime 证据风险进入接受项。 | 用户要求手工真实验证时暂停关闭并新建 runtime gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP8-CR091-01..05，关闭 CR091 当前离线交付为 `closed-current-delivery / READY_WITH_RISK` |
| 备选方案 | `修改: <具体修改点>` 后重出 CP8；或 `reject` 回退到 CP7 pending CP8 |
| 影响维度 | 用户价值、可验证性、维护成本、安全 / 权限、后续 runtime 路线、账本治理 |
| 优劣分析 | 推荐方案收口离线 runner 合同并保留真实验证路线；备选方案更保守但会延迟关闭并需要额外授权 |
| 风险与回退 | 接受 `R-CR091-CP7-001..003`；reject 时回退到 `active-cp7-pass-with-risk-pending-cp8` |
| 用户需决策事项 | DQ-CP8-CR091-01 是否接受 READY_WITH_RISK；DQ-CP8-CR091-02 是否限定关闭范围；DQ-CP8-CR091-03 是否继续不授权 runtime；DQ-CP8-CR091-04 是否只保留候选不启动；DQ-CP8-CR091-05 是否无需用户手工 runtime 验证即可关闭离线切片 |

### CP8 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 交付范围 | CR091 research / HLD / LLD / TEST-PLAN、offline runner code、checker、fixtures/tests、CP6/CP7/CP8 evidence |
| 安装验证 | N/A；无安装脚本、Agent、Skill 或平台部署 |
| 文档缺口 | 无阻断文档缺口；全局 `TEST-STRATEGY.md` 未新增，CR091 scoped TEST-PLAN 被 CP7 接受为等价输入 |
| 遗留风险 | 真实 runtime 未验证、历史 CR tracking 旧账、future evidence output gate |
| 风险接受项 | `R-CR091-CP7-001`、`R-CR091-CP7-002`、`R-CR091-CP7-003` |
| 推荐处理方案 | 接受 `READY_WITH_RISK`，关闭当前 CR091 离线 runner 交付 |
| 备选处理方案 | 修改后重出 CP8；或 reject 回退到 CP7 pending CP8；或新建 runtime authorization gate 后再关闭 |
| 回退方式 | 使用 `process/docs/release/ROLLBACK-CR091.md`，仅回滚代码 / fixture / checker / 台账状态，不执行 destructive action |

### CP8 后续跟踪分流表

| 分流类别 | 项目 ID | 状态 | 处理方式 | 台账 / CR 路径 | 说明 |
|---|---|---|---|---|---|
| 关闭范围 | CLOSE-CR091-01 | pending-close | CP8 approve 后关闭本轮离线 runner 交付 | `process/checkpoints/CP8-CR091-DELIVERY-READINESS.md` | 不包含真实 runtime、NAS、凭据或交易写。 |
| 不授权范围 | NA-CR091-01 | not-authorized | 不进入本轮执行授权 | `process/checkpoints/CP8-CR091-DELIVERY-READINESS.md` | QMT/MiniQMT/XtQuant/gateway/runner、NAS、凭据、账户、submit/cancel、simulation/live 必须独立授权。 |
| 风险接受项 | RA-CR091-01 | pending-acceptance | 用户接受风险后放行 | `docs/quality/CR091-QMT-STRATEGY-RUNNER-VERIFICATION-REPORT.md` | `R-CR091-CP7-001..003`。 |
| 后续 CR 候选项 | CR091-FU-01 | candidate-not-started | 保留为 runtime authorization 候选 | `process/docs/release/FEEDBACK-CR091.md` | 真实只读 runner smoke。 |
| 后续 CR 候选项 | CR091-FU-02 | candidate-not-started | 保留为 NAS package exchange 候选 | `process/docs/release/FEEDBACK-CR091.md` | NAS package exchange / pull / publish。 |
| 后续 CR 候选项 | CR091-FU-03 | candidate-not-started | 保留为 high-risk order-write 候选 | `process/docs/release/FEEDBACK-CR091.md` | submit/cancel 或 simulation/live。 |
| 后续 CR 候选项 | CR091-FU-04 | candidate-not-started | 保留为 ledger hygiene 候选 | `process/docs/release/FEEDBACK-CR091.md` | 历史 CR025 / CR019 台账旧账。 |
| 取消 / deferred 项 | DEF-CR091-01 | deferred | 真实交易、真实账户、真实运行全部后置 | `process/release/RELEASE-CONTEXT-CR091.yaml` | 不删除，保留追溯。 |

## 不授权范围

本 CP8 即使通过，也不授权：

- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089。
- 恢复 CR020 用户删除的 gateway 路线。

## Entry Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP6 implementation done | 待审查 | `process/checks/CP6-CR091-QMT-STRATEGY-RUNNER-CODING-DONE.md` |  |
| CP7 verification done | 待审查 | `process/checks/CP7-CR091-QMT-STRATEGY-RUNNER-VERIFICATION-DONE.md` |  |
| Release Context ready | 待审查 | `process/release/RELEASE-CONTEXT-CR091.yaml` |  |
| Release docs ready | 待审查 | `process/docs/release/*-CR091.md` |  |
| CP8 auto check ready | 待审查 | `process/checks/CP8-CR091-DELIVERY-READINESS.md` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | Decision Brief 覆盖推荐、备选、影响、风险和回退 | 待审查 | 本文件 Decision Brief |  |
| 2 | Decision Collection Coverage 已列出 | 待审查 | 本文件覆盖表 |  |
| 3 | 用户视角不授权项已列出 | 待审查 | 本文件“不授权范围” |  |
| 4 | 风险接受项已列出 | 待审查 | `R-CR091-CP7-001..003` |  |
| 5 | 后续路线不自动启动 | 待审查 | CP8 后续跟踪分流表 |  |
| 6 | Release docs 和 rollback 可读 | 待审查 | `process/docs/release/*-CR091.md` |  |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 5 项推荐方案，关闭 CR091 当前离线交付为 `closed-current-delivery / READY_WITH_RISK`。 |
| `修改: <具体修改点>` | 按修改点修订 CP8 材料并重新发起确认。 |
| `reject` | 回退到 `active-cp7-pass-with-risk-pending-cp8`，不关闭 CR091。 |

## Deliverables

- `process/context/CP8-CR091-DELIVERY-CONTEXT.yaml`
- `process/release/RELEASE-CONTEXT-CR091.yaml`
- `process/docs/release/RELEASE-NOTES-CR091.md`
- `process/docs/release/DEPLOY-CHECKLIST-CR091.md`
- `process/docs/release/ROLLBACK-CR091.md`
- `process/docs/release/MIGRATION-CR091.md`
- `process/docs/release/FEEDBACK-CR091.md`
- `process/checks/CP8-CR091-DELIVERY-READINESS.md`
- `process/checks/CP8-CR091-HUMAN-GATE-LAUNCH-MESSAGE.md`

## 人工审查结果

- 结论：`pending`
- 审查人：
- 审查时间：
- 用户回复：
- 接受的决策项：
- 风险接受项：
- 修改意见：
- 不授权边界：
