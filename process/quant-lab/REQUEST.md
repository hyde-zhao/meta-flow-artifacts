---
status: captured
created_at: "2026-05-13"
created_by: "meta-po"
request_type: "new_workflow_init"
engagement_mode: "production"
scenario_subject_type: "target-artifact"
scenario_subject_id: "local-daily-backtest-layer"
---

# 原始请求登记

## 用户目标

开发一个本地量化回测项目，先不选择大型量化框架，而是在项目内实现一个轻量、透明、可调试的“本地日频组合回测层”。第一阶段重点服务实践六动量策略的本地复现、参数扫描和报告输出，后续再扩展 RSI、MACD、样本外测试、市场环境分段和聚宽少量候选参数验证。

## 建议目录结构

```text
work/studies/quant-trading/local_backtest/
  data/
    prices.parquet
    index_members.parquet
    trade_calendar.parquet
  engine/
    data_loader.py
    metrics.py
    portfolio.py
    backtest.py
  strategies/
    momentum.py
    rsi.py
    macd.py
  notebooks/
    01_momentum_local_test.ipynb
  reports/
    momentum_param_sweep.csv
```

## 核心分层

- 信号层：根据历史价格计算买卖信号或目标权重。
- 组合层：根据目标权重、当前持仓、交易成本生成组合净值。
- 分析层：计算收益、回撤、Sharpe、Alpha、Beta、分段表现。

## 第一版本地回测器最小能力

1. 读取多股票日线收盘价。
2. 读取沪深 300 成分股池，第一版可简化为当前成分股。
3. 每 N 个交易日调仓。
4. 计算过去 `lookback_days` 收益率。
5. 选排名前 `top_fraction`。
6. 持仓跌出 `sell_buffer` 后卖出。
7. 目标持仓等权。
8. 扣除手续费和滑点。
9. 输出净值曲线、累计收益、年化收益、最大回撤、Sharpe。
10. 支持参数扫描。

## 第一版明确不处理

1. 涨跌停。
2. 停牌。
3. 成分股历史变化。
4. 分红送转。
5. 分钟级撮合。
6. 真实成交量约束。

## 数据路线

学习阶段采用 AKShare 拉取数据并保存为 parquet 本地缓存。第一次联网拉取后保存到本地；后续回测只读取本地 parquet，避免每次回测都联网取数。数据至少包括股票日线行情、沪深 300 指数行情、交易日历、指数成分股、复权因子、停牌信息和涨跌停价格；其中第一版本地回测只强依赖收盘价、固定沪深 300 股票池和交易日历。

## 框架路线

第一版不直接上 RQAlpha、Backtrader 或完整事件驱动框架。当前目标是快速验证动量、RSI、MACD 等策略参数，不是搭建完整交易系统。后续当策略稳定并需要订单、账户、撮合、风控、实盘模拟或多品种支持时，再考虑迁移到 RQAlpha 或聚宽。

## 策略核心共用要求

动量策略应抽出可复用的纯函数，供本地回测器和聚宽策略复用或复制同一逻辑，避免本地和平台代码出现两套不同策略逻辑。用户给出的核心函数示意：

```python
def compute_momentum_returns(close_df, lookback_days):
    return close_df.iloc[-1] / close_df.iloc[-lookback_days - 1] - 1


def select_momentum_targets(
    momentum,
    current_positions,
    top_fraction=0.10,
    sell_buffer=0.30,
):
    ranked = momentum.dropna().sort_values(ascending=False)
    buy_count = max(1, int(len(ranked) * top_fraction))
    buy_set = set(ranked.head(buy_count).index)

    sell_threshold = max(buy_count, int(len(ranked) * sell_buffer))
    keep_set = set(ranked.head(sell_threshold).index)

    target = buy_set | (set(current_positions) & keep_set)
    return sorted(target)
```

## 推荐路线图

1. 做一个最小本地动量回测器：读取本地 parquet、每 20 个交易日调仓、计算过去 20 日收益率、买入前 10%、跌出前 30% 卖出、等权、输出净值曲线和指标。
2. 补齐参数扫描和报告：扫描 `lookbacks = [5, 10, 20, 30, 60]`、`freqs = [5, 10, 20, 30]`、`fractions = [0.05, 0.10, 0.20]` 共 60 组参数，输出 `momentum_param_sweep_local.csv`，并补充热力图、收益排名、Sharpe 排名、最大回撤排名、样本内/样本外衰减。
3. 将本地最优参数回填聚宽验证：只跑默认参数、本地 Sharpe 最优、本地收益最优、保守低换手参数，避免在聚宽上跑完整 60 组扫描。

## 第一版验收线索

1. 能跑完整 2019-2025。
2. 能输出累计收益、年化收益、最大回撤、Sharpe。
3. 能跑 60 组参数扫描。
4. 参数扫描耗时明显低于聚宽。

## 外部依据待核验项

- AKShare 适合作为学习阶段免费数据接口，但其数据声明为学术研究与参考用途，接口可能受不可控因素影响。
- RQAlpha 更适合事件驱动、订单、账户、撮合、风控、分析和更接近真实交易的后续阶段。
- vectorbt 可在参数扫描规模变大后用于指标与持仓向量化。
- bt 可在资产配置、ETF 轮动、多资产组合和定期再平衡场景中作为后续候选。

## CR156 原始请求

- 时间：2026-07-04T23:43:42+08:00
- 请求：启动 CR156。请先按 meta-flow current state 恢复上下文：读取 AGENTS.md、process/current/CURRENT.json、process/state/STATE.current.json、process/changes/CR-INDEX.yaml，确认当前无 active formal CR 后，从 CP0 开始受理。
- 初始处理：已创建 CR156 受理入口；CP0 曾因缺少具体变更目标、范围、验收标准、优先级和授权边界而 `BLOCKED`。

## CR156 目标补充

- 时间：2026-07-05T00:07:22+08:00
- 补充：CR156 的目标是 hygiene 收束：`FU-CR154-001` packaging → `FU-CR152-001` test taxonomy。
- 当前处理：CP0 已按该补充更新为 `PASS`；CR156 当前 scope 为本地 hygiene consolidation，不改变产品基线，不授权 Git remote write、true release execution、publish、runtime、真实数据、凭据、NAS/provider、broker 或交易动作。

## CR156 命名整改与 CP2 推进

- 时间：2026-07-05T00:21:55+08:00
- 请求：修改为长命名后，继续推进项目。
- 处理：正式 CR 文件已从短名迁移为 `process/changes/CR-156-HYGIENE-PACKAGING-TEST-TAXONOMY-2026-07-05.md`，并同步引用；CP1 自动检查已通过，CP2 compact scope baseline 已生成。
- 下一步准确提示词：`请审查 process/checkpoints/CP2-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE.md，并回复 approve / 修改: <具体修改点> / reject`

## CR156 CP2 评审意见处理

- 时间：2026-07-05T00:38:02+08:00
- 请求：按照评审意见继续推进项目；重点修正 Checkpoint Index、CR-INDEX 路径、串行约束和 CP3-CP7 形式开销。
- 处理：CP2 已回填为 `approved`；CR 正文与 summary / index 改为 `CR-INDEX.json`；CP3 / CP4 / CP5 已标记为 N/A；CP6 / CP7 已标记为 WAIVED；两项 follow-up evidence 可并行只读核验并统一 closure；已生成 CP8 READY_WITH_RISK closure gate。
- 下一步准确提示词：`请审查 process/checkpoints/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-RELEASE-READINESS.md，并回复 approve / 修改: <具体修改点> / reject`

## CR156 CP8 批准关闭与流程复盘

- 时间：2026-07-05T09:09:46+08:00
- 请求：批准 CR156 关闭；关闭后审核 CR156 开发过程中 Meta Flow 流程遵守情况、流程可改进点、token 消耗集中阶段和优化空间。
- 处理：CP8 已回填为 `approved`；CR156 已关闭为 `READY_WITH_RISK`；`FU-CR154-001` 与 `FU-CR152-001` 已标记为 closed under CR156；当前无 active formal CR / pending gate。
- 复盘产物：`process/checks/CR156-META-FLOW-PROCESS-RETROSPECTIVE-2026-07-05.md`

## CR168 原始请求与评审修正

- 时间：2026-07-13T16:00:00+08:00
- 请求：将 `FU-CR161-004` 启动为下一个正式变更，建议编号 `CR-168`，建立 fixture/static-only 的 C3 Economic Cost / Slippage / Impact computable evidence producer foundation；只推进到 CP2 人工门禁。
- 评审修正：Gate 4 明确为 C3+C4 联合门禁；C3 只投影 `impact_model_family`、`impact_model_ref`、`cost_underestimation_status`、`no_real_tca_claim`，C4 字段保持 `typed_unavailable` 并 fail-closed；把成本低估风险状态纳入 C3 合同；fixture 调整为 daily multifactor 与 ML 的 multi-strategy-type compatibility；跨字段 currency/price-basis/calendar 混用仅在缺少显式转换声明时触发失败。
- 授权边界：允许双仓库远端读取/fetch、本地安全分支创建和必要冲突处理；不允许 Git 远端写入、真实数据/凭据访问、真实 TCA/impact calibration、C4 计算、交易/runtime、Stage 3 启动或 CR-155 admission promotion。
- 编排方式：用户于 2026-07-13 明确要求不拉起子 Agent；产品基线刷新由当前 Host Orchestrator 以内联方式执行，并记录为用户批准的 `meta-pm inline-fallback`。

## CR170 原始请求与评审修正

- 时间：2026-07-15T12:05:00+08:00
- 请求：核实 FU-CR161-007 canonical-hardening 方案中两个评审关注点，必要时修改范围与实施方案，然后正式开始该 CR。
- 评审修正一：`build_shared_gate_summary` 已传播 artifact `NEEDS_REVIEW`，因此该底层 merge 必须先回归验证、无失败证据时保持修改数为 0；不能笼统重写 Gate 6。
- 评审修正二：`resolve_admission_policy` 是独立 admission 集成点；当前 probe 证实 T0/T1/T2 的 mandatory Gate `NEEDS_REVIEW` 均会落到 PASS，CP3 必须冻结 lower-level merge 与 tier/admission decision 的职责边界。
- 正式范围：Gate 1-5 N/A evidence 五态语义 + Gate 6 admission policy hardening；Gate 2/3/4/5 generic reason escape 已复现，Gate 1 必须采用字段判定、blocked claim、最终 worst-state 三层断言。
- 跟踪拆分：`FU-CR161-007` 的 canonical-hardening 子切片转为 `CR-170`；aggregate + CR155 regression 剩余切片按数字型规范保留为 `FU-CR161-009`。
- 授权边界：允许双仓库 fetch/read、本地分支和 fixture/static probe；不授权真实数据、凭据、Stage 3、aggregate、CR155 promotion、runtime/trading、publish/release 或 Git 远端写入。
- 编排方式：用户此前明确要求不拉起子 Agent，本轮继续使用已批准的 `meta-pm inline-fallback`；不得伪造 subagent dispatch。
