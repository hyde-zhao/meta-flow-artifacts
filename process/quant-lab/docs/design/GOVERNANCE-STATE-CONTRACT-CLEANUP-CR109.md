---
title: "CR109 Governance / State Contract Cleanup"
cr_id: "CR-109"
status: "closed-current-delivery"
created_at: "2026-06-22T14:13:19+08:00"
owner: "host-orchestrator"
source_cr: "CR-109"
scope_policy: "no-code-no-runtime"
---

# CR109 Governance / State Contract Cleanup

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 初稿：把 FU-CR108-002 转为治理 / 状态契约清洁决策门。 |
| v0.2 | 2026-06-22 | host-orchestrator | 用户批准 CP2，接受 no-code/no-runtime 范围、默认优先整理 State / CR tracking / follow-up row 契约和不授权边界。 |
| v1.0 | 2026-06-22 | host-orchestrator | 用户批准 CP8，CR109 关闭为 READY；仅推进候选 FU-CR109-001 为正式 CR110，FU-CR109-002/003 保持 candidate。 |

## 1. 决策目标

CR109 的目标是在 redesign 继续展开前，先把治理和状态契约清洁项排好优先级。本 CR 不改源码、不修改 checker 实现、不启动 runtime、不访问 NAS 或凭据。

## 2. 当前输入

| 输入 | 状态 | 说明 |
|---|---|---|
| CR108 | closed-current-delivery / READY | Backlog decision 已关闭，用户选择 FU-CR108-002。 |
| 当前 active formal CR | CR109 | 本 CR 是唯一新启动的 active formal CR。 |
| 源码整改需求 | false | 当前证据不要求源码整改。 |
| 高风险路线 | not-authorized | CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish 均未授权。 |

## 3. Cleanup 候选池

| 候选 | 类型 | 默认动作 | 说明 |
|---|---|---|---|
| State / active_change contract normalization | governance | 推荐候选 | 收敛 `STATE.md` 顶层 active_change、cr_tracking 摘要和 CR-INDEX 的一致性口径。 |
| CR-INDEX item schema normalization | governance | 推荐候选 | 收敛 active_crs / recent_items / items 的重复字段、状态枚举和 source_tracking 规则。 |
| Follow-up tracking row consistency | governance | 推荐候选 | 统一 formal CR、candidate、blocked/deferred 在台账中的生命周期字段。 |
| Context capsule naming and read-profile consistency | governance | 候选 | 统一 CP2 / CP8 context capsule 的命名、read_profile 和扩展读取记录。 |
| Human gate launch / checkpoint consistency | governance | 候选 | 收敛人工门禁文件状态、发起消息和 approved 回填语义。 |
| CP result JSON / ledger gap analysis | governance | 延后 | 若需要补机器 result / ledger，另行启动更大治理 CR。 |

## 4. 默认推荐

| 排名 | 推荐候选 | 推荐动作 | 理由 |
|---:|---|---|---|
| 1 | State / CR tracking / follow-up row contract | 默认下一步 | 直接降低 CR 恢复、换机器和上下文压缩时的状态误判风险。 |
| 2 | Context capsule / human gate consistency | 备选 | 对后续 CP2 / CP8 有帮助，但可在状态契约之后处理。 |
| 3 | CP result JSON / ledger gap analysis | 延后 | 范围更大，可能引出工具实现或迁移工作。 |

## 5. 非范围

| 非范围 | 说明 |
|---|---|
| 源码修改 | CR109 不修改 Python / test / package / checker code。 |
| HLD / LLD 正式设计 | CR109 不直接进入 architecture-only HLD。 |
| 实现计划执行 | 不创建 Story 实现批次。 |
| runtime / trading | 不启动 QMT/MiniQMT/XtQuant/gateway，不下单、不撤单、不 simulation/live。 |
| NAS / credentials / publish | 不访问 NAS，不读取凭据，不执行 provider/lake/catalog publish。 |

## 6. CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR109-01 | scope | 接受 CR109 只做 no-code/no-runtime governance cleanup gate | A. 暂停治理清洁；B. 改为 HLD gate | 若用户要求实现或 runtime，另起独立 CR |
| DQ-CP2-CR109-02 | follow_up_tracking | 默认优先整理 State / CR tracking / follow-up row 契约 | A. 优先 context capsule；B. 优先 human gate；C. 只盘点不排序 | 若用户调整排序，更新本文件并重新发起 CP2 |
| DQ-CP2-CR109-03 | runtime_authorization | 继续禁止源码修改、CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish | A. 另起源码实现 CR；B. 另起 runtime/NAS gate | 若用户明确要求高风险验证，停止 CR109 默认路线 |

## 7. 当前结论

CR109 的 CP8 已获用户批准，当前交付关闭为 `READY`。用户选择“仅需推进候选的 CR”，因此候选 `FU-CR109-001 State / CR Tracking Contract Normalization Plan` 已转为正式活动 CR110；`FU-CR109-002 Context Capsule / Human Gate Consistency Review` 和 `FU-CR109-003 CP Result JSON / Ledger Gap Analysis` 继续保留为 candidate。

本 CR 未授权源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、账户 / raw log 读取或 provider/lake/catalog publish。
