---
checkpoint_id: "CP2"
checkpoint_name: "CR107 Redesign Scope Intake Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-22T13:38:58+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-22T13:50:24+08:00"
auto_check_result: "process/checks/CP2-CR107-REDESIGN-SCOPE-INTAKE-PRECHECK.md"
target:
  phase: "requirement-scope-baseline"
  change_id: "CR-107"
  artifacts:
    - "process/changes/CR-107-REDESIGN-SCOPE-INTAKE-AND-CANDIDATE-PRIORITIZATION-2026-06-22.md"
    - "docs/design/REDESIGN-SCOPE-INTAKE-CR107.md"
    - "process/context/CP2-CR107-REDESIGN-SCOPE-INTAKE-CONTEXT.yaml"
---

# CP2 CR107 Redesign Scope Intake 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---|---|
| `process/checks/CP2-CR107-REDESIGN-SCOPE-INTAKE-PRECHECK.md` | PASS | 0 | 可进入人工确认。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR107-REDESIGN-SCOPE-INTAKE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。 |
| 全文档读取扩展 | 0 次。 |
| 缺失 / waived 理由 | N/A。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 本轮决策来自用户当前指令和 CR107 scope。 |
| 自动预检结果 | `process/checks/CP2-CR107-REDESIGN-SCOPE-INTAKE-PRECHECK.md` | scanned | 3 | 3 | scope、follow_up_tracking、runtime_authorization。 |
| CR formal file | `process/changes/CR-107-REDESIGN-SCOPE-INTAKE-AND-CANDIDATE-PRIORITIZATION-2026-06-22.md` | scanned | 3 | 3 | 与本表去重后保留 3 项。 |
| Scope intake draft | `docs/design/REDESIGN-SCOPE-INTAKE-CR107.md` | scanned | 3 | 3 | 覆盖候选排序、默认下一步和不授权边界。 |
| CR tracking index | `process/changes/CR-INDEX.yaml` | scanned | 6 | 3 | 高风险和 Spike 候选已分类；不全部纳入阻断决策。 |
| discussion log / checkpoint | N/A | n/a | 0 | 0 | 本 CR 是 scope intake，不新增产品场景讨论。 |
| 下游正式产物 | N/A | n/a | 0 | 0 | 尚未进入 HLD / LLD / 实现。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | 用户明确启动 CR107，并限制不改代码、不启动高风险 runtime。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR107-01 | scope | 是否批准 CR107 只做 no-code/no-runtime 的 redesign scope intake 和候选项排序？背景：CR106 已关闭 READY，当前没有源码整改证据。 | 批准 CR107 只做范围收口、候选排序和台账同步，不改源码。 | A. 暂停 redesign，保持 CR106 baseline；B. 直接进入 architecture-only HLD。 | 推荐方案能先把工作队列和风险边界定清楚；A 最保守但不推进；B 更快进入设计但可能跳过候选收敛。 | 推荐方案风险低；主要成本是一轮文档 / 台账维护。 | 若用户要求代码实现，另起实现 CR；若用户要求架构设计，转 FU-CR107-002。 |
| DQ-CP2-CR107-02 | follow_up_tracking | 是否接受当前候选排序，并把 `FU-CR107-001 Redesign Backlog Decision Gate` 作为 CR107 后默认下一步？ | 接受排序：低风险 backlog decision gate 优先；CR-026/027/028 延后；RA-CR097-001、CR089、CR105 不启动。 | A. 优先 architecture-only HLD；B. 优先 CR-026 Qlib Spike；C. 保持全部候选等待。 | 推荐方案延续低风险路线；A 适合用户已决定做结构设计；B 适合仍需要 Qlib 对照；C 最保守但会停滞。 | 推荐方案不触发 runtime 和源码修改；风险是需要后续再细化 backlog。 | 若用户指定不同优先级，更新 scope intake 并重新发起 CP2。 |
| DQ-CP2-CR107-03 | runtime_authorization | 是否继续禁止 CR105、CR089 恢复、QMT/MiniQMT/XtQuant/gateway runtime、交易写、NAS、凭据和 provider/lake/catalog publish 被 CR107 隐式启动？ | 是。CR107 不启动任何高风险 runtime 或交易写门禁。 | A. 另起 CR105 高风险门禁；B. 另起 runtime/NAS authorization gate。 | 推荐方案符合用户“暂不启动高风险 runtime”的限制；A/B 可处理高风险需求但需要独立审查和授权。 | 推荐方案避免误触交易写、账户、凭据和外部资源边界；高风险验证继续延期。 | 若用户明确要求 order-write / submit-cancel / simulation-live 或真实 runtime，停止 CR107 默认路线并创建独立 gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`，接受 DQ-CP2-CR107-01..03 的推荐方案。 |
| 备选方案 | `修改: <具体修改点>` 可指定优先 HLD、优先 CR-026、暂停 redesign 或另起高风险 gate。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、安全 / 权限、交付影响。 |
| 优劣分析 | 见上表。 |
| 风险与回退 | CR107 不改源码、不启动 runtime；发现需要实现或高风险验证时另起 CR。 |
| 用户需决策事项 | DQ-CP2-CR107-01、DQ-CP2-CR107-02、DQ-CP2-CR107-03。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确启动 CR107 | 待审查 | 当前对话 | 用户已要求启动 CR107。 |
| 不改代码边界清晰 | 待审查 | CR107 / scope intake | 不修改源码。 |
| 不启动高风险 runtime 边界清晰 | 待审查 | CR107 authorization_policy | 不启动 CR105 / runtime / 交易写 / NAS / 凭据 / publish。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否接受 CR107 作为当前 redesign 入口 | 待审查 | DQ-CP2-CR107-01 | 推荐接受。 |
| 2 | 是否接受当前候选排序 | 待审查 | DQ-CP2-CR107-02 | 推荐接受。 |
| 3 | 是否接受高风险边界继续禁止 | 待审查 | DQ-CP2-CR107-03 | 推荐接受。 |
| 4 | 是否允许 CR107 后续只做文档 / 台账收敛 | 待审查 | CR107 执行链路 | 不进入实现 / runtime。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 决策项全部有结论 | 待审查 | DQ-CP2-CR107-01..03 | 需用户 approve / 修改 / reject。 |
| 无未授权高风险动作 | 待审查 | 不授权项 | approve 不授权 CR105、CR089 恢复或 runtime。 |
| 可进入 CR107 CP8 收敛 | 待审查 | CP2 auto precheck PASS | CP2 approved 后进入候选路由收敛。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR107 formal CR | `process/changes/CR-107-REDESIGN-SCOPE-INTAKE-AND-CANDIDATE-PRIORITIZATION-2026-06-22.md` | 待审查 |  |
| CR107 scope intake | `docs/design/REDESIGN-SCOPE-INTAKE-CR107.md` | 待审查 |  |
| CP2 auto precheck | `process/checks/CP2-CR107-REDESIGN-SCOPE-INTAKE-PRECHECK.md` | 待审查 |  |
| CP2 context capsule | `process/context/CP2-CR107-REDESIGN-SCOPE-INTAKE-CONTEXT.yaml` | 待审查 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-22T13:50:24+08:00
- 修改意见：无；用户回复 `approve`，接受 DQ-CP2-CR107-01..03 的推荐方案。
- 风险接受项：接受 CR107 仅执行 no-code/no-runtime 的 redesign scope intake 和候选排序；不授权 CR105、CR089 恢复、QMT/MiniQMT/XtQuant/gateway runtime、交易写、NAS、凭据、账户原文或 provider/lake/catalog publish。
