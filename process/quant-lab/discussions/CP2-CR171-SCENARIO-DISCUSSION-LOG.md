# CP2 CR171 Scenario / Requirement Discussion Log

## 记录

| 字段 | 值 |
|---|---|
| CR | CR-171 — Stage 3 Launch / Real-Lake Entry Decision Gate |
| 阶段 | CP1 complete; CP2 human decision pending |
| 日期 | 2026-07-15 |
| 记录方 | meta-pm |
| 用户交互状态 | 本轮未提问；交接限制要求 Host Orchestrator 集中发起 CP2 正式选择。 |
| CP1 澄清结论 | 无阻断歧义；三个决策都可由 CP2 在不触发数据、runtime 或实现操作的情况下作出。 |

## Scenario Gray Areas

| Gray area | 为什么重要 | 影响面 | CP2 处理 |
|---|---|---|---|
| 证据路线 | 决定 current runner 的边界冻结义务，或 C1-C4 后续 activation CR。 | architecture / authorization / future delivery | 必选 `CP2-CR171-DQ-ROUTE` |
| verifier 路径 | 决定 FU-006 是否阻断 entry，以及何时必须收紧为完整 verifier。 | risk acceptance / verification / gate | 必选 `CP2-CR171-DQ-VERIFIER` |
| future read scope | 决定后续可能读取的严格五元组与 deny-default 边界，避免从 CR 创建或 CP8 推断授权。 | authorization / audit / runtime safety | 必选 `CP2-CR171-DQ-READ-SCOPE` |

## Host CP2 Relay Items

### CP2-CR171-DQ-ROUTE — Stage 3 Evidence Route

- 推荐：`c1_c4_real_producer`。
- 备选：`current_runner`。
- 影响：推荐路线形成 2 个 entry CR（本 CR + Real-Evidence Activation CR），把 computation/producer binding 保留给后续显式授权；备选路线只有 1 个 entry CR，但 CP3 必须先冻结 runner 的完整 read-only execution boundary。
- 不能由本次选择授权：真实数据读取、computation、producer binding、aggregate、runtime、trading。

### CP2-CR171-DQ-VERIFIER — FU-006 Disposition

- 推荐：`event_bounded_waiver`。
- 备选：`fu006_first`。
- 影响：推荐路线只在两个机械到期事件之前有效：任何 real-evidence admission 可为 PASS/PASS_WITH_RISK 前；或 Stage 3 exit gate 可启动前。备选路线形成 3 个 entry CR（本 CR + FU-006 + activation）。
- 不继承：CR170 的历史 verifier 风险接受不构成 CR171 waiver。

### CP2-CR171-DQ-READ-SCOPE — Frozen Read Contract

- 推荐：`scoped_research_data_lake_read_only`，以 `data_release`、datasets、date range、read identity、output directory 五元组冻结。
- 备选：不批准任何 read scope，保持 Stage 3 未启动。
- 无条件拒绝：credentials/environment read、provider fetch、lake/NAS write、catalog/current-pointer mutation、runtime、trading。
- 说明：本 CP2 决策文本本身不执行读取、计算或运行时行为；任何后续操作仍须遵守获批范围与后续检查点。

## Confirmed Boundaries

- 历史 Stage 3 运行只能表述为 `legacy / require-revalidation`；允许 verdict 仅为 `reaffirmed_as_legacy_only`、`insufficient_for_current_entry`、`incompatible_rework_required`。
- 任何 repair、backfill、rerun、manifest rewrite 或 defect remediation 必须另立 follow-up/CR。
- CR010、CR018、CR032 仅披露，不在 CR171 内修复；风险消费者使用 `R-CR170-RUNNER-GAP`。
- CP8 交付成功不等于 `stage3_entry_ready`，也不等于真实数据、computation、runtime 或 trading 授权。
