---
checkpoint_id: "CP2"
checkpoint_name: "CR108 Redesign Backlog Decision Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-22T14:01:49+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-22T14:13:19+08:00"
auto_check_result: "process/checks/CP2-CR108-REDESIGN-BACKLOG-DECISION-PRECHECK.md"
target:
  phase: "requirement-scope-baseline"
  change_id: "CR-108"
  artifacts:
    - "process/changes/CR-108-REDESIGN-BACKLOG-DECISION-GATE-2026-06-22.md"
    - "docs/design/REDESIGN-BACKLOG-DECISION-CR108.md"
    - "process/context/CP2-CR108-REDESIGN-BACKLOG-DECISION-CONTEXT.yaml"
---

# CP2 CR108 Redesign Backlog Decision 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---|---|
| `process/checks/CP2-CR108-REDESIGN-BACKLOG-DECISION-PRECHECK.md` | PASS | 0 | 可进入人工确认。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR108-REDESIGN-BACKLOG-DECISION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。 |
| 全文档读取扩展 | 0 次。 |
| 缺失 / waived 理由 | N/A。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 本轮决策来自用户当前指令和 CR108 scope。 |
| 自动预检结果 | `process/checks/CP2-CR108-REDESIGN-BACKLOG-DECISION-PRECHECK.md` | scanned | 3 | 3 | scope、follow_up_tracking、runtime_authorization。 |
| CR formal file | `process/changes/CR-108-REDESIGN-BACKLOG-DECISION-GATE-2026-06-22.md` | scanned | 3 | 3 | 与本表去重后保留 3 项。 |
| Backlog decision draft | `docs/design/REDESIGN-BACKLOG-DECISION-CR108.md` | scanned | 3 | 3 | 覆盖候选池、默认推荐和不授权边界。 |
| CR107 release context | `process/release/RELEASE-CONTEXT-CR107.yaml` | scanned | 1 | 1 | FU-CR107-001 来源已确认。 |
| discussion log / checkpoint | N/A | n/a | 0 | 0 | 本 CR 是 backlog decision，不新增产品场景讨论。 |
| 下游正式产物 | N/A | n/a | 0 | 0 | 尚未进入 HLD / LLD / 实现。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | 用户明确启动 FU-CR107-001。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR108-01 | scope | 是否批准 CR108 只做 no-code/no-runtime 的 redesign backlog decision？背景：CR107 已关闭 READY，当前需要把 backlog 固化为后续 CR 队列。 | 批准 CR108 只做 backlog decision、候选排序和台账同步，不改源码。 | A. 暂停 redesign backlog；B. 直接进入 architecture-only HLD。 | 推荐方案先把后续队列定清楚；A 最保守但不推进；B 更快进入设计但可能跳过 backlog 收敛。 | 推荐方案风险低；主要成本是一轮文档 / 台账维护。 | 若用户要求代码实现，另起实现 CR；若用户要求架构设计，转 FU-CR108-001。 |
| DQ-CP2-CR108-02 | follow_up_tracking | 是否接受当前 backlog 候选池，并默认把 `FU-CR108-002 Governance / State Contract Cleanup Gate` 作为 CR108 后下一步推荐？ | 接受排序：治理 / state contract cleanup 优先；architecture-only HLD 作为备选；implementation planning 延后；CR-026/027/028 延后或后续取消。 | A. 优先 architecture-only HLD；B. 全部候选等待；C. 取消 Qlib / data Spike 候选。 | 推荐方案延续低风险 no-code/no-runtime；A 适合用户已决定先做结构设计；B 最保守但会停滞；C 可减少 backlog 噪音。 | 推荐方案不触发 runtime 和源码修改；风险是后续仍需单独 CR 才能执行。 | 若用户指定不同优先级，更新 backlog decision 并重新发起 CP2。 |
| DQ-CP2-CR108-03 | runtime_authorization | 是否继续禁止 CR105、CR089 恢复、QMT/MiniQMT/XtQuant/gateway runtime、交易写、NAS、凭据和 provider/lake/catalog publish 被 CR108 隐式启动？ | 是。CR108 不启动任何高风险 runtime 或交易写门禁。 | A. 另起 CR105 高风险门禁；B. 另起 runtime/NAS authorization gate。 | 推荐方案符合当前低风险 redesign 路线；A/B 可处理高风险需求但需要独立审查和授权。 | 推荐方案避免误触交易写、账户、凭据和外部资源边界；高风险验证继续延期。 | 若用户明确要求 order-write / submit-cancel / simulation-live 或真实 runtime，停止 CR108 默认路线并创建独立 gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`，接受 DQ-CP2-CR108-01..03 的推荐方案。 |
| 备选方案 | `修改: <具体修改点>` 可指定优先 HLD、暂停 backlog、取消 Spike 或另起高风险 gate。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、安全 / 权限、交付影响。 |
| 优劣分析 | 见上表。 |
| 风险与回退 | CR108 不改源码、不启动 runtime；发现需要实现或高风险验证时另起 CR。 |
| 用户需决策事项 | DQ-CP2-CR108-01、DQ-CP2-CR108-02、DQ-CP2-CR108-03。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确启动 FU-CR107-001 | approved | 当前对话 | 用户已要求启动。 |
| CR107 已关闭 READY | approved | CR107 CP8 / release context | 来源候选已确认。 |
| 不改代码 / 不启动 runtime 边界清晰 | approved | CR108 authorization_policy | 不修改源码、不启动 CR105 / CR089 / runtime / NAS / 凭据 / publish。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否接受 CR108 作为当前 backlog decision gate | approved | DQ-CP2-CR108-01 | 用户回复 `approve`，接受推荐方案。 |
| 2 | 是否接受当前 backlog 候选排序 | approved | DQ-CP2-CR108-02 | 用户回复 `approve` 并明确启动 `FU-CR108-002`。 |
| 3 | 是否接受高风险边界继续禁止 | approved | DQ-CP2-CR108-03 | 用户回复 `approve`，不授权高风险范围。 |
| 4 | 是否允许 CR108 后续只做文档 / 台账收敛 | approved | CR108 执行链路 | 不进入实现 / runtime。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 决策项全部有结论 | approved | DQ-CP2-CR108-01..03 | 用户回复 `approve`，接受全部推荐方案。 |
| 无未授权高风险动作 | approved | 不授权项 | approve 不授权源码修改、CR105、CR089 恢复或 runtime。 |
| 可进入 CR108 backlog decision 收敛 | approved | CP2 auto precheck PASS | CP2 approved，可进入 CP8 收口。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR108 formal CR | `process/changes/CR-108-REDESIGN-BACKLOG-DECISION-GATE-2026-06-22.md` | approved | CP2 scope gate accepted. |
| CR108 backlog decision | `docs/design/REDESIGN-BACKLOG-DECISION-CR108.md` | approved | 默认下一步选择 `FU-CR108-002`。 |
| CP2 auto precheck | `process/checks/CP2-CR108-REDESIGN-BACKLOG-DECISION-PRECHECK.md` | approved | PASS。 |
| CP2 context capsule | `process/context/CP2-CR108-REDESIGN-BACKLOG-DECISION-CONTEXT.yaml` | approved | ready / minimal。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-22T14:13:19+08:00
- 修改意见：无。
- 风险接受项：接受 CR108 只做 no-code/no-runtime backlog decision；接受默认下一候选为 `FU-CR108-002 Governance / State Contract Cleanup Gate`；继续不授权源码修改、CR105、CR089 恢复、QMT/MiniQMT/XtQuant/gateway runtime、交易写、NAS、凭据、账户 / raw log 读取或 provider/lake/catalog publish。
