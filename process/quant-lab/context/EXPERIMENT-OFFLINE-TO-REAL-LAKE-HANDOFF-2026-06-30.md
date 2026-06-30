# Experiment Offline To Real Lake Handoff

## 目的

本交接文档供后续在可连接数据湖的 PC 上继续 E4：真实 lake readonly semantic validation。它只说明下一阶段应读什么、验证什么、禁止什么；不授权当前机器连接真实数据湖。

## 当前离线完成状态

| 项 | 状态 |
|---|---|
| CR140 | experiments 归位、helper 收敛、turnover synthetic parity 已完成 |
| CR141 | 46 条红基线已清零，最终全量 pytest 通过 |
| CR142 | 17_21 / 23_29 safe engine convergence 已完成 |
| CR143 | E3 非 lake 收尾：IC engine contract、factor ownership、交接文档 |
| E2 | source 和 artifact 已执行远端持久化 |

## E4 进入条件

1. 在数据湖 PC 上重新拉取 source 和 artifact 远端最新提交。
2. 先运行 full pytest，确认离线基线仍为全绿。
3. 明确授权范围为真实 lake readonly validation。
4. 继续禁止 lake write、catalog pointer write、provider fetch、NAS 修改、QMT/runtime、simulation/live/trading。
5. 只验证研究语义，不发布生产策略，不推进实盘。

## 必读文件

| 文件 | 用途 |
|---|---|
| `process/changes/CR-140-EXPERIMENTS-REFACTOR-2026-06-28.md` | turnover synthetic parity 边界 |
| `process/changes/CR-142-REMAINING-EXPERIMENTS-ENGINE-CONVERGENCE-2026-06-30.md` | 17_21 / 23_29 engine convergence 边界 |
| `process/changes/CR-143-EXPERIMENT-OFFLINE-HARDENING-2026-06-30.md` | E3 非 lake 收尾边界 |
| `process/checks/CR143-EXPERIMENT-OFFLINE-INVENTORY-2026-06-30.md` | experiment inventory 和 factor ownership 入口 |
| `experiments/run_experiment_turnover_factor.py` | E4 turnover 真实湖只读验证候选 |
| `experiments/run_experiment_15_factor_framework.py` | E4 多因子研究输入验证候选 |
| `experiments/run_experiment_16_momentum_factor.py` | E4 IC / Rank IC 真实数据形状验证候选 |
| `experiments/run_experiment_17_21_factor_suite.py` | E4 多因子套件真实数据形状验证候选 |
| `experiments/run_experiment_23_29_ml_factor_suite.py` | E4 ML 套件只读 smoke 候选，不做 ML 生产化 |

## E4 建议验证顺序

1. **只读预检**：确认 lake root、as-of、dataset availability，只读读取，不写 manifest 或 pointer。
2. **turnover 真实数据形状验证**：验证 CR140 synthetic parity 覆盖不到的真实字段、日期、缺失、极值、停牌和横截面大小。
3. **实验 15/16 多因子基础验证**：验证 factor panel、forward return、IC/Rank IC、group return 在真实数据形状下不报错，口径可解释。
4. **17_21/23_29 smoke**：只做研究套件真实数据形状 smoke，不声明生产准入。
5. **差异登记**：所有 synthetic 与真实 lake 差异必须分类为数据形状、字段语义、统计口径、因子定义、性能或权限问题。

## 禁止项

- 不写 lake。
- 不写 catalog pointer。
- 不 publish。
- 不 provider fetch。
- 不执行 NAS 修改。
- 不执行 QMT / MiniQMT / xtquant / gateway runtime。
- 不启动 simulation/live/trading。
- 不把 research-only summary 升格为 production admission package。

## E4 产出建议

| 产出 | 说明 |
|---|---|
| E4 readonly validation CR | 单独登记真实 lake 只读授权和边界 |
| 真实 lake 差异表 | 按数据形状/字段语义/统计口径分类 |
| experiment real-lake smoke report | 只声明只读研究验证结果 |
| 后续生产化候选 | 只登记，不直接启动模拟盘或实盘 |
