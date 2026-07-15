# CR-170 CP2 场景讨论日志

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-07-15 | host-orchestrator-inline-meta-pm | 固化用户评审输入、两个代码事实关注点及五项 CP2 待决策问题。 |
| v0.2 | 2026-07-15 | host-orchestrator-inline-meta-pm | 吸收 UC-58 评审：独立验证者标为 `future consumer`，当前验证由 Gate 维护者自验证代行；补充 consumer 端到端调用视角。用户在该修订条件下批准 CP2 五项推荐方案。 |

## 本轮用户输入复述

用户要求再次审查两个关注点是否需要修改，并在修改后启动正式 CR：

1. Gate 6 已有 `NEEDS_REVIEW` worst-state 传播，避免无依据重写。
2. `resolve_admission_policy` 是独立 admission 集成点，CP3 必须明确其与底层 merge 的边界。

复核结果：两点均成立，但第一点只适用于 `build_shared_gate_summary` 的底层合并；实际 probe 同时证明 `resolve_admission_policy` 会在 T0/T1/T2 把 mandatory `NEEDS_REVIEW` 落为 PASS。因此 CR170 的目标修改为“保留并回归验证底层 merge + 最小硬化 admission policy”，而不是笼统重写 Gate 6。

## Scenario Gray Areas

| ID | 问题 | 推荐方案 | 备选 | 状态 |
|---|---|---|---|---|
| SGQ-CR170-001 | hardening 覆盖 Gate 4 还是 Gate 1-5？ | Gate 1-5 一次完成 inventory 与 N/A semantics。 | 只 Gate 4，其他另起 CR。 | OPEN-CP2 |
| SGQ-CR170-002 | 五态是业务语义还是代码类型？ | CP2 冻结业务语义，CP3 决定最小代码形态。 | CP2 直接冻结 enum/dataclass。 | OPEN-CP2 |
| SGQ-CR170-003 | tier policy 如何处理 mandatory NEEDS_REVIEW？ | T0 NR/no-PASS；T1/T2 BLOCKED；T3 NOT_AUTHORIZED。 | 所有 tier 一律 BLOCKED。 | OPEN-CP2 |
| SGQ-CR170-004 | Gate 6 应重写还是分层？ | 保护现有 bottom-up merge，单独硬化 `resolve_admission_policy`。 | 重写聚合与 admission。 | OPEN-CP2 |
| SGQ-CR170-005 | CR168/169 adapter 是否可删？ | 本 CR 保留并回归；FU-CR161-009 满足四条件后再评估。 | 本 CR 同步简化。 | OPEN-CP2 |

## 产品复述检查

- 目标：mandatory evidence 缺失、generic reason escape、边界不完整或 NEEDS_REVIEW 时，无条件 PASS 数为 0。
- 范围：Gate 1-5 N/A semantics + Gate 6 protected merge/admission policy；不是 Stage 3 Launch 或 aggregate。
- 验收：21/21 policy units、5/5 语义、20/20 场景、15/15 QAC、adapter 2/2 regression。
- 不授权：真实数据、current runner 接入、aggregate、CR155 promotion、runtime/trading、远端写入。

## UC-58 用户评审补强与人工结论

- “独立验证者”不作为 CR170 当前已实现的 consumer；它是 `FU-CR161-006` 的 future consumer。
- CR170 当前验证主体为可靠性 Gate 维护者自验证；这不构成 verifier independence claim。
- UC-58 增补准入策略维护者的端到端调用期望，但 Use Case、Requirement、Scenario 与 QAC 数量均不改变。
- 用户批准 CP2 五项推荐方案；批准效果仅为进入 CP3 solution-design。

当前状态：`CP2 approved`；五项 DQ 均 `resolved-by-user`，下一阶段为 solution-design。
