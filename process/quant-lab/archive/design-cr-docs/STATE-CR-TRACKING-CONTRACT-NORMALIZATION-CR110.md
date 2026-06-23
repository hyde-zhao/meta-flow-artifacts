---
title: "CR110 State / CR Tracking Contract Normalization"
cr_id: "CR-110"
status: "closed-current-delivery"
created_at: "2026-06-22T14:34:43+08:00"
owner: "host-orchestrator"
source_cr: "CR-110"
scope_policy: "no-code-no-runtime"
---

# CR110 State / CR Tracking Contract Normalization

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 初稿：把 FU-CR109-001 转为 State / CR tracking 合同归一化计划门。 |
| v0.2 | 2026-06-22 | host-orchestrator | 用户批准 CP2，接受 no-code/no-runtime 范围、默认优先级和高风险不授权边界。 |
| v1.0 | 2026-06-22 | host-orchestrator | 用户批准 CP8，CR110 关闭为 READY；STATE 摘要清洁和 checker expectation notes 保留为后续 candidate。 |

## 1. 决策目标

CR110 的目标是在不改源码、不改 checker implementation 的前提下，定义 State / CR tracking / follow-up row 的合同归一化计划，降低后续恢复上下文、换机器和 CR 状态查询时的误判风险。

## 2. 当前输入

| 输入 | 状态 | 说明 |
|---|---|---|
| CR109 | closed-current-delivery / READY | Governance cleanup gate 已关闭，用户选择 FU-CR109-001。 |
| 当前 active formal CR | CR110 | 本 CR 是唯一新启动的 active formal CR。 |
| 源码整改需求 | false | 当前授权不包含源码或 checker implementation 修改。 |
| 高风险路线 | not-authorized | CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish 均未授权。 |

## 3. Contract 候选池

| 候选 | 类型 | 默认动作 | 说明 |
|---|---|---|---|
| active formal CR source of truth | governance | 推荐候选 | 明确 CR-INDEX、formal CR frontmatter、STATE 摘要之间的优先级。 |
| gate_status enum contract | governance | 推荐候选 | 固化 `cp2_pending`、`cp8_pending`、`closed`、`not_started` 等机器枚举。 |
| follow-up row lifecycle contract | governance | 推荐候选 | 统一台账中 formal CR、candidate、blocked/deferred 的生命周期字段。 |
| state summary staleness policy | governance | 候选 | 明确 `STATE.md` 人类摘要字段可能陈旧时的处理方式。 |
| CR tracking checker expectation notes | governance | 候选 | 记录 checker 当前字段预期和常见失败模式。 |

## 4. 默认推荐

| 排名 | 推荐候选 | 推荐动作 | 理由 |
|---:|---|---|---|
| 1 | active formal CR source of truth | 默认下一步 | 直接降低 active CR 锁和 candidate 转正式 CR 的误判风险。 |
| 2 | gate_status enum + follow-up lifecycle | 并列推进 | 与 CR tracking checker 强相关，能减少格式失败。 |
| 3 | state summary staleness policy | 延后 | 重要但可在主合同明确后补充。 |

## 5. 非范围

| 非范围 | 说明 |
|---|---|
| 源码修改 | CR110 不修改 Python / test / package / checker code。 |
| checker implementation | 不修改 `meta-flow check cr-tracking` 或其他工具实现。 |
| HLD / LLD 正式设计 | CR110 不直接进入 architecture-only HLD。 |
| runtime / trading | 不启动 QMT/MiniQMT/XtQuant/gateway，不下单、不撤单、不 simulation/live。 |
| NAS / credentials / publish | 不访问 NAS，不读取凭据，不执行 provider/lake/catalog publish。 |

## 6. CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR110-01 | scope | 接受 CR110 只做 no-code/no-runtime State / CR tracking contract normalization plan | A. 暂停 CR110；B. 改为 context capsule / human gate consistency review | 若用户要求实现或 runtime，另起独立 CR |
| DQ-CP2-CR110-02 | follow_up_tracking | 默认优先整理 active formal CR source of truth、gate_status enum 和 follow-up row lifecycle | A. 只整理 CR-INDEX；B. 只整理 STATE 摘要；C. 只盘点不排序 | 若用户调整排序，更新本文件并重新发起 CP2 |
| DQ-CP2-CR110-03 | runtime_authorization | 继续禁止源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish | A. 另起源码实现 CR；B. 另起 runtime/NAS gate | 若用户明确要求高风险验证，停止 CR110 默认路线 |

## 7. CP2 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | approved |
| 审批时间 | 2026-06-22T14:47:29+08:00 |
| 接受决策 | DQ-CP2-CR110-01、DQ-CP2-CR110-02、DQ-CP2-CR110-03 |
| 下一门禁 | CP8 delivery readiness |

## 8. 合同归一化计划

CR110 CP2 已获用户批准。当前合同归一化计划如下：

| 合同项 | 当前结论 | 后续处理 |
|---|---|---|
| active formal CR source of truth | `CR-INDEX.active_crs[]` 是活动正式 CR 汇总入口；formal CR frontmatter、follow-up tracking 和 `STATE.md.active_change` 必须同步。 | 在 CP8 中作为 CR110 当前交付关闭。 |
| gate_status enum contract | `gate_status` 只使用机器枚举：`not_started`、`cp2_pending`、`cp3_pending`、`cp5_pending`、`cp8_pending`、`closed`、`blocked`。 | 在 CP8 中作为 CR110 当前交付关闭。 |
| follow-up row lifecycle contract | candidate 不占执行锁；候选转正式 CR 后，正式 CR 行和 legacy candidate 行必须同步；正式 CR 关闭后，上游候选别名也必须同步关闭或保留明确 candidate。 | 在 CP8 中作为 CR110 当前交付关闭。 |
| state summary staleness policy | 仅作为后续候选，不在 CR110 中重写 `STATE.md` 历史长表。 | 保留候选。 |
| CR tracking checker expectation notes | 仅作为后续候选；若涉及 checker implementation change，必须另起 CR。 | 保留候选。 |

CR110 不授权源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、账户 / raw log 读取或 provider/lake/catalog publish。

## 9. CP8 关闭结果

CR110 已于 2026-06-22T15:04:13+08:00 经用户批准关闭为 `closed-current-delivery / READY`。当前交付只关闭合同归一化计划，不启动代码层整改。

| 分流 | 项目 | 状态 | 说明 |
|---|---|---|---|
| 关闭范围 | active formal CR source of truth | closed | 作为当前治理合同计划关闭。 |
| 关闭范围 | gate_status enum contract | closed | 作为当前治理合同计划关闭。 |
| 关闭范围 | follow-up row lifecycle contract | closed | 作为当前治理合同计划关闭。 |
| 后续候选 | FU-CR110-001 STATE Summary Staleness Cleanup Candidate | candidate | 仅登记，不启动。 |
| 后续候选 | FU-CR110-002 CR Tracking Checker Expectation Notes Candidate | candidate | 仅登记，不启动；若涉及 checker implementation change，必须另起实现 CR。 |
