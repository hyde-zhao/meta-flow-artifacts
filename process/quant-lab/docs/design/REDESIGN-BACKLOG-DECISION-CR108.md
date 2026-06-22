---
title: "CR108 Redesign Backlog Decision"
cr_id: "CR-108"
status: "approved-closed-ready"
created_at: "2026-06-22T14:01:49+08:00"
owner: "host-orchestrator"
source_cr: "CR-108"
scope_policy: "no-code-no-runtime"
---

# CR108 Redesign Backlog Decision

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 初稿：把 CR107 scope intake 转换为 redesign backlog 决策队列。 |
| v1.0 | 2026-06-22 | host-orchestrator | 用户批准 CR108 backlog decision，并明确选择 `FU-CR108-002` 作为下一条正式 CR。 |

## 1. 决策目标

CR108 的目标是把 CR107 已确认的候选排序转换为 2-4 个可执行后续 CR 候选，并为下一条 CR 给出明确推荐。本 CR 不做架构设计、不写 LLD、不改代码、不启动 runtime。

## 2. 当前输入

| 输入 | 状态 | 说明 |
|---|---|---|
| CR107 | closed-current-delivery / READY | Scope intake 和候选排序已关闭。 |
| 当前 active formal CR | none | 启动 CR108 前 `cr-tracking` OK。 |
| 源码整改需求 | false | CR106 / CR107 均未发现必须源码整改的证据。 |
| 高风险路线 | not-authorized | CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish 均未授权。 |

## 3. Backlog 候选池

| 候选 | 类型 | 默认动作 | 说明 |
|---|---|---|---|
| `FU-CR108-001 Architecture-only Redesign HLD Gate` | architecture | 条件启动 | 仅当用户决定先做模块 / 状态 / ledger / context / routing 的结构设计时启动。 |
| `FU-CR108-002 Governance / State Contract Cleanup Gate` | governance | 推荐候选 | 可继续 no-code / no-runtime，收敛状态契约、context capsule、CR indexing 和 human gate 规则缺口。 |
| `FU-CR108-003 Redesign Implementation Planning Gate` | planning | 延后 | 只有当 HLD 或治理队列明确后再规划实现，不直接进入代码。 |
| `CR-026 Qlib isolated runner optional Spike` | spike | 延后 / 可取消 | 与 redesign 默认路线关系弱，只有仍需要 Qlib 对照 runner 时保留。 |
| `CR-027 Minute data feasibility Spike` | spike | 延后 | 需要明确日频证据不足的触发条件。 |
| `CR-028 Level2 rights and microstructure Spike` | spike / high-risk | 延后 | 权利、成本、schema 和 replay fixture 风险高。 |
| `RA-CR097-001` | runtime | 不启动 | 真实 runtime readonly retest，不属于 redesign 默认路线。 |
| `CR-089` | blocked runtime | 保持 blocked | 不由 CR108 恢复。 |
| `FU-CR101-001 / CR105` | trading high-risk | 不启动 | order-write / simulation / live 独立高风险门禁。 |

## 4. 默认推荐

| 排名 | 推荐候选 | 推荐动作 | 理由 |
|---:|---|---|---|
| 1 | `FU-CR108-002 Governance / State Contract Cleanup Gate` | 默认下一步 | 继续 no-code / no-runtime，能直接降低后续流程风险。 |
| 2 | `FU-CR108-001 Architecture-only Redesign HLD Gate` | 备选 | 适合用户决定先做结构设计，但成本比治理收敛更高。 |
| 3 | `FU-CR108-003 Redesign Implementation Planning Gate` | 延后 | 实现规划必须等 backlog / HLD /治理边界更清楚。 |

## 5. 非范围

| 非范围 | 说明 |
|---|---|
| 源码修改 | CR108 不修改 Python / test / package code。 |
| HLD / LLD 正式设计 | CR108 只决定是否启动 HLD，不直接写完整 HLD。 |
| 实现计划执行 | 不创建 Story 实现批次。 |
| runtime / trading | 不启动 QMT/MiniQMT/XtQuant/gateway，不下单、不撤单、不 simulation/live。 |
| NAS / credentials / publish | 不访问 NAS，不读取凭据，不执行 provider/lake/catalog publish。 |

## 6. CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR108-01 | scope | 接受 CR108 只做 backlog decision，不改代码、不启动 runtime | A. 暂停 redesign backlog；B. 直接进入 HLD gate | 若用户要求实现或 runtime，另起独立 CR |
| DQ-CP2-CR108-02 | follow_up_tracking | 默认把 `FU-CR108-002 Governance / State Contract Cleanup Gate` 作为下一步推荐 | A. 优先 HLD gate；B. 全部候选等待；C. 取消 Qlib / data Spike 候选 | 若用户调整排序，更新本文件并重新发起 CP2 |
| DQ-CP2-CR108-03 | runtime_authorization | 继续禁止 CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish | A. 另起 CR105；B. 另起 runtime/NAS gate | 若用户明确要求高风险验证，停止 CR108 默认路线 |

## 7. 当前结论

CR108 已完成 backlog decision 收口。用户在 2026-06-22T14:13:19+08:00 回复 `approve，并启动FU-CR108-002 Governance / State Contract Cleanup Gate`，表示：

- 接受 CR108 只做 no-code/no-runtime backlog decision。
- 选择 `FU-CR108-002 Governance / State Contract Cleanup Gate` 作为下一条正式 CR。
- `FU-CR108-001 Architecture-only Redesign HLD Gate` 保持候选。
- `FU-CR108-003 Redesign Implementation Planning Gate` 延后。
- 不授权源码修改、CR105、CR089 恢复、runtime、交易写、NAS、凭据、账户 / raw log 读取或 provider/lake/catalog publish。
