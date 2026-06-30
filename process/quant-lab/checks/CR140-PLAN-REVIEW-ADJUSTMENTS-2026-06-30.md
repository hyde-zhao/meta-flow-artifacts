# CR140 Plan Review Adjustments

## Entry Criteria

- CR140 已登记并完成 Phase 0 baseline。
- 用户从 quant-lab 端到端定位（信息收集 -> 策略研究回测 -> 模拟盘 -> 实盘）提交计划评审。
- 当前不得进入 Phase 1，除非本文件中的调整项已写回 CR140 状态。

## Review Decision

结论：CR140 方向正确，但必须调整叙事和执行门禁后再进入 Phase 1。

CR140 的目标收敛为：

> 整理研究实验层与生产/数据工具层的代码边界，并用离线 fixture 证明 turnover 实验评估侧迁移到 engine 后的重构等价性；本轮不验证真实数据湖、不产生模拟盘/实盘准入、不改变生产 current truth。

## Required Adjustments

| ID | 类型 | 调整项 | 决策 |
|---|---|---|---|
| ADJ-CR140-01 | governance | CR139 状态收敛是 CR140 Phase 0 的治理副作用，必须独立审计说明。 | 采用独立审计说明，不新开 mini CR；CP8 终验必须列为范围外附带治理修正。 |
| ADJ-CR140-02 | baseline | 46 条 Phase 0 失败必须分类。 | 新增分类文件；后续 Phase 报告必须列出新增失败和修复的基线失败。 |
| ADJ-CR140-03 | phase1 | Phase 1 三个迁移不得合并成一个回退单元。 | 1.1/1.2/1.3 分别形成独立验证与可独立 commit 的变更组。 |
| ADJ-CR140-04 | phase1 | `production_current_truth_rerun.py` 归位目标缺少职责判定。 | Phase 1.2 前先判定 reusable engine contract vs data governance script；不得默认搬到 `engine/`。 |
| ADJ-CR140-05 | phase1 | `lake_input_contract.py` 与 `training_snapshot_contract.py` 也是非实验文件。 | 本轮必须明确：纳入 Phase 1 归位，或列为后续 CR；不得声称 Phase 1 后 `experiments/` 只剩实验入口。 |
| ADJ-CR140-06 | phase2 | helper 收敛前缺少行为等价检查。 | 删除本地 helper 前必须 diff 本地副本与基座；非 byte-equivalent 或语义不等价则保留并记录。 |
| ADJ-CR140-07 | phase3 | golden fixture 只能证明合成 fixture 上的新旧等价。 | close condition 必须写明：不证明真实数据研究语义、不证明模拟盘/实盘准入。 |
| ADJ-CR140-08 | phase3 | turnover 产物当前无下游模拟盘/实盘消费者需要固化。 | 作为 Phase 3 前置证据；未来新增 consumer 需独立准入审查。 |
| ADJ-CR140-09 | phase5 | 回退后缺少复验要求。 | revert 后必须重跑全量 pytest 并对比 Phase 0 failure set，确认没有新增失败后才能继续。 |

## Evidence Notes

- `experiments/` 当前还包含非实验文件：`lake_input_contract.py`、`production_current_truth_rerun.py`、`reporting.py`、`run_data_lake_readiness_audit.py`、`training_snapshot_contract.py`。
- `production_current_truth_rerun.py` docstring 声明为 CR018-S08 production current truth research rerun dry-run contract；归位前需要职责判定。
- `lake_input_contract.py` 和 `training_snapshot_contract.py` docstring 均声明为 CR139 contract / fixture validation helper；它们不是实验入口。
- `rg` 检查未发现 `experiments/run_experiment_turnover_factor.py` 的产物被 `engine/`、`scripts/`、`strategies/` 或 `trading/` 作为模拟盘/实盘准入输入直接消费；当前风险是未来 consumer 误读，而不是现有 runtime 破坏。

## Exit Criteria

- CR140 正文已补充目标降调、Phase 1/2/3/5 调整和治理副作用说明。
- Phase 0 failure set 已分类。
- CR140 summary / STATE 已引用本文件。
- `meta-flow check cr-tracking` 仍为 PASS。

## Deliverables

- `process/checks/CR140-PLAN-REVIEW-ADJUSTMENTS-2026-06-30.md`
- `process/checks/CR140-CR139-STATUS-RECONCILIATION-AUDIT-2026-06-30.md`
- `process/checks/CR140-PHASE0-BASELINE-FAILURE-CLASSIFICATION-2026-06-30.md`
