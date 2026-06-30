---
title: 数据湖模块整改交接评审文档
status: revised-after-review-r5
version: "0.7"
audience: 评审人
created_at: 2026-06-28
revised_at: 2026-06-28
revision_reason: 接受五轮评审返修；修 3 处文字/附录残留：§3.10"本层为新建对象"与 F1-F4 (d2) 冲突、附录 B 标题"类别 a 来源"误导（含已改 d2 的 L2/L5）、§8.7"v0.5 口径"应为 v0.6
source_investigation: 数据湖目录核查 + 代码耦合分析 + 设计文档比对 + 五轮评审反馈核验 + §3 全表机械统计
review_status: v0.1–v0.6 经五轮评审/自检返修；v0.7 修最后 3 处文字残留，可进入 CR/CP2/CP3 审批链路
---

# 数据湖模块整改交接评审文档

> 本文档用于外部评审。v0.1–v0.6 经五轮评审/自检返修。v0.7 修正五轮评审 3 处文字/附录残留（§3.10 小节说明、附录 B 标题、§8.7 版本口径），机械统计口径不变：d1 纯新建 6 + d2 既有闭环 14 = 20，逐项点数 6+7+12+6+14=45。评审目标：判断整改对象是否完整、防线是否到位、目标是否对齐"多因子 + ML 策略全流程生产"、落地路线是否可行。本文不含实现，只含判断与依据。

## 0. 评审请求摘要

| 项 | 内容 |
|---|---|
| **整改目标** | 数据湖模块支撑多因子策略 + 机器学习策略，从信息收集 → 回测 → 模拟盘 → 实盘，全流程可信、可复现、可审计 |
| **整改对象** | 市场数据湖 + 研究数据集/因子面板 + ML feature/label + run evidence + broker facts + 交易审计链（v0.1 仅覆盖市场数据湖，v0.2 扩展至全数据底座） |
| **评审重点** | (1) 整改是否覆盖全流程全数据资产层；(2) 防线集是否构成充分集；(3) 设计是否需要刷新；(4) 落地路线是否可行；(5) v0.1 事实错误是否已修正 |
| **不评审** | 具体代码实现、CR 编号分配、发布判定 |

---

## 1. 现状核查（整改对象的事实依据，v0.1 事实错误已修正）

### 1.1 数据湖物理结构

```
data/lake/ (软链 → 外部存储, 已 gitignore)
├── raw/          18G   原始抓取, Hive 分区 run_id=
├── canonical/    3.5G  规范化 parquet, dataset/schema_version/run_id=
├── candidate/    空    medallion 中间层
├── gold/         空    medallion 中间层
├── published/    空    medallion 终态
├── quality/      81 目录, 1.2M  质量报告, 平铺, 混 smoke/probe
├── catalog/      32K   catalog.json 单文件真相源
└── manifest/     6.8M  market_data_manifest.jsonl 第二真相源
```

prices 下 38 个 run、5886 个 parquet 文件；`read_canonical` 会 glob 全量 parquet（`market_data/readers.py:375`）。

### 1.2 数据集清单（catalog 已登记 17 个）

价格类：`prices`、`prices_hfq`、`prices_limit`、`adj_factor`、`hs300_index`
财务/估值类：`financial_pit`、`market_cap`、`events`
参考类：`stock_basic`、`trade_calendar`、`trade_status`、`index_members`、`index_weights`、`industry_classification`、`bse_code_mapping`、`lifecycle_code_change`、`liquidity_capacity`

**字段底子是好的**：
- `prices` = `symbol × trade_date`，含 raw + adjusted OHLCV + `adj_factor` + `available_at` + `available_at_rule`
- `financial_pit` 带 `ann_date`、`report_period`、`available_at`、`available_at_rule`（PIT 原料齐全）
- `market_cap` 带 pe/pb/ps/turnover + `available_at`
- 全部 parquet 列式存储

### 1.3 关键代码事实（v0.1 三条错误已修正，见 §1.3.1）

| 事实 | 位置 | 含义 |
|---|---|---|
| `read_canonical` 只认 prices，非 prices raise | `market_data/readers.py:2691` | prices 专用 fast reader |
| **通用 `read_dataset` 已存在，且阻断未发布 catalog**（v0.1 漏判） | `market_data/readers.py:2728`、`:2796 if not entry.published` | 问题不是"全库 reader 只认 prices"，而是"无统一 published/as-of **panel** reader + ML 接入缺失" |
| reader glob-concat 全部分区 | `readers.py:375 _canonical_paths`、`:409` | prices 下 38 个 `run_id` 分区全读，`(symbol,trade_date)` 可能重复 |
| DuckDB 只读护栏已写，adapter 空 | `market_data/duckdb_query.py` | `import duckdb` 全库 0 处；`pyproject.toml` 无 duckdb 依赖；策略默认 `ENGINE_FALLBACK` |
| ML 实验绕过 lake | `experiments/run_experiment_15_*.py load_local_frames`、`run_experiment_23_29_*.py:132 --data-dir` | ML 从本地目录读，数据湖对 ML 是摆设 |
| 路径构造集中在 `lake_layout.py` | `market_data/lake_layout.py` | 单一咽喉，结构改动代码触点可控 |
| `events` 数据集 schema 全 `null` 类型 | `data/lake/canonical/events/` | 8 part 坏数据，写入侧类型推断 bug |
| `quality/` 含 `cr010-...-smoke-v2/v3`、`unknown` run_id | `data/lake/quality/` | 一次性产物混入生产报告，命名不一致 |

#### 1.3.1 catalog 元数据实际分布（v0.1 修正）

v0.1 称"catalog 全部 pit_status: null"为**过度泛化错误**。实测（`uv run python` 一次统计）：

| 字段 | 实际分布 |
|---|---|
| `published` | 17/17 全 `false`（v0.1 属实） |
| `pit_status` | 14 `null` + 3 `pit_available`（index_members、index_weights、stock_basic）—— v0.1 称"全部 null"**错误** |
| `lineage_checksum` | 17/17 全缺失（v0.1 属实） |

**修正表述**：catalog 元数据对 PIT 的覆盖是**部分**的（3 个参考类已 pit_available，14 个价格/财务类未标记），非全 null。lineage_checksum 与 published 确实全缺。

### 1.4 设计文档现状

| 文档 | 版本/状态 | 问题 |
|---|---|---|
| `process/HLD-DATA-LAKE.md` | v0.8, `confirmed: false`, `draft-pending-cp3-cr018` | **停在 2026-05-29，CR017/CR018 从未过 CP3，与实现漂移** |
| `docs/design/BLUEPRINT.md` | v1.12, CR-138 | FEAT-02 范围是"数据生产"，未覆盖"策略生产数据底座"全貌 |
| `docs/design/FEATURE-DESIGN-MATRIX.md` | v1.11, CR-131 | 缺 ML feature 层 / 读侧语义 / run evidence 的 Feature 归属 |
| `docs/features/market-data-lake/` | 2026-06-21 stub (3KB/1.4KB/2KB) | 停在 CR014 时代，既不反映 CR018 也不反映新目标 |

### 1.5 蓝图 Feature 边界（v0.2 新增，作为范围 checklist）

整改对象分散在多个 Feature，不能只归 FEAT-02：

| Feature | 与策略生产数据底座的关系 |
|---|---|
| FEAT-02 生产级市场数据湖 | 市场数据事实源（写侧 + 读侧） |
| FEAT-03 研究数据集与多因子研究闭环 | FactorPanel、LabelWindow、ExperimentManifest、StrategyAdmissionPackage、因子目录 |
| FEAT-06 OMS/风控/Broker Lake/阶段激活 | BrokerLakeRecord、订单/成交/持仓审计链、stage gate |
| FEAT-11 Runner Control Plane | RunPlan、RunEvidence、RunState、IncidentRecord |
| FEAT-12 QMT Gateway Service Layer | CommissionSchedule、PnLSnapshot、ExecutionReport、GatewayAuditRecord、TradingCalendar |
| FEAT-13 策略类型适配与统一研究合同 | StrategyTypeAdapter、SignalSet、ResearchEvidenceIndex |
| FEAT-14 成熟策略生产与模拟盘门禁 | UniversePolicy、PortfolioRiskPolicy、SimulationObservationPlan、ReadinessDecision |

### 1.6 既有合同存在性（v0.3 新增，修正二轮 BLOCKING1）

> 二轮评审指出：v0.2 把"已有合同"误写成"无"，会导致重复设计和范围膨胀。核验后确认下列对象**已存在合同或局部实现**，整改性质应为"既有对象与数据湖 release / lineage / ML feature / run-id 全链路闭环"，**不是新建**。

| 对象 | 已存在位置 | 现状 | 整改性质 |
|---|---|---|---|
| ExperimentManifest | `engine/research_manifest.py:152` | 有 dataclass + 校验 | 闭环到 published data release + lineage |
| StrategyAdmissionPackage | `engine/strategy_admission_package.py:127` | 有 dataclass + 序列化 | 闭环到 catalog current pointer + run-id |
| RunEvidenceIndex | `trading/strategy_runner/evidence_index.py:19` | 有 from_run_result | 闭环到 lake run-id 贯通 |
| BrokerLakeSchema | `trading/broker_lake.py:64` | 有 schema registry + dry-run writer | 闭环到实盘写 + 审计链 |
| CommissionSchedule | `trading/qmt_gateway_contracts.py:997` | 有合同定义 | 闭环到成本前置（T5） |
| FactorPanel / LabelWindow / FactorModelValidationReport | FEAT-03 蓝图对象 | 蓝图已登记 | 闭环到 panel reader + PIT |
| label 泄漏审计 / embargo / ml_leakage_audit | `engine/factor_model_validation.py:561`、`:376`、`engine/factor_robustness.py:53` | 分散存在 | 统一接入 data release + cutoff gate（E3） |
| BenchmarkCoverage / BenchmarkDefinition | `market_data/benchmarks.py:99`、`:114` | contract/resolver 已存在 | 补版本化 benchmark + risk-free curve 事实源 + release 闭环（F1） |
| incremental refresh 计划合同 | `market_data/incremental.py:1` CR014-S06 `plan_incremental_refresh` | dry-run plan 合同已存在 | 接通真实日级 append 执行链（L1，v0.5 自检新增） |
| publish_current_pointer | `market_data/publish.py:605` | dry-run only | 接通真实 pointer 前移执行（L2，v0.5 自检新增） |
| readiness matrix | `market_data/readiness.py:462` `build_readiness_matrix` | 已存在 | 前置为实盘读前强制阻断门禁（L4，v0.5 自检新增） |
| replay 合同 | `market_data/replay.py`、`cli.py:437 cmd_p0_replay` | 部分存在 | 接通 published as_of 单日快照重放（L5，v0.5 自检新增） |
| decision_time 过滤 | `market_data/readers.py:227`、`:1790` | 已部分支持 | 加强制 lookahead 阻断门禁（X3，v0.5 自检新增） |
| survivorship_bias_note | `engine/contracts.py:270` `is_pit_universe=false` | 已识别偏差 | 补按时点构建 PIT universe 成分链（X4，v0.5 自检新增） |
| PortfolioRiskPolicy | `engine/mature_multifactor_framework.py:228` | 合同已存在 | 补版本化 universe/risk policy 事实源 + release 闭环（F3，v0.5 自检新增） |
| policy_cycle_coverage | `engine/factor_model_validation.py:444`、`config/policy_cycles.yaml` | 评估已存在 | 补版本化政策周期/shortability 事实源 + release 闭环（F4，v0.5 自检新增） |

**修正口径**：本 handoff 中凡涉及上述对象的整改项，"问题"列应理解为"未与数据湖 release / run-id / artifact ref / lineage 闭环"、"分散存在未统一"或"合同已存在缺执行/版本化/前置"，而非"对象不存在"。仅 §3 中 L3、E4、T7、T8、X1、X2 标 (d1) 者为纯新建。

---

## 2. 整改目标与防线集

### 2.1 目标分解（按全流程四阶段）

| 阶段 | 数据底座必须提供 | 当前是否满足 |
|---|---|---|
| **信息收集** | 规范写入、PIT 盖戳、去重、CR 审计、血缘、配置类事实源 | 部分（写入在，PIT/去重/血缘/配置层未闭合） |
| **回测** | PIT 安全面板、去重、冻结快照、多表 join、universe 时变、复权正确 | **不满足** |
| **模拟盘** | 日级增量、pointer 前移、读前质量门禁、run evidence 贯通 | 不满足 |
| **实盘** | 低延迟读、schema 契约稳定、读审计、replay、决策时点-新鲜度对齐 | 不满足 |

### 2.2 防线集（v0.2 扩展为"数据/研究/ML/交易生产门禁集"）

> 评审核心：v0.1 的四道防线（PIT/去重/快照/join）是**必要但不充分**集。v0.2 扩展为四组门禁，覆盖策略生产全链路失真源。

#### 2.2.1 数据门禁组（原四道防线）
| 防线 | 含义 | 缺失后果 | 当前状态 |
|---|---|---|---|
| PIT 强制 | 读财报/估值按 `available_at <= as_of` 取最新，杜绝前视 | 标签泄露，模型虚高 | reader 未执行 |
| 去重唯一 | `(symbol, trade_date)` 在 canonical 唯一 | 重复行污染标签 | glob-concat 可能多份 |
| 冻结快照 | published current pointer 固定 | 调参偷看未来 | 三层全空 |
| 多表 as-of join | 价格×财务×估值×行业统一 as-of 宽表 | ML 被迫旁路 | panel reader 缺失 |

#### 2.2.2 研究门禁组（v0.2 新增）
| 防线 | 含义 | 缺失后果 |
|---|---|---|
| PIT universe / 幸存者偏差 | universe 按时点成分构建，含退市/ST | 幸存者偏差，回测虚高 |
| 复权与公司行动 | 除权除息事件按 PIT 应用，复权因子本身 PIT 正确 | 收益序列断裂、复权断点 |
| 交易状态/涨跌停/停牌 | 信号生成排除不可交易标的 | 不可成交信号进入回测 |
| label 泄漏检查 | 标签窗口与特征 cutoff 严格分离 | 标签泄漏 |

#### 2.2.3 ML 门禁组（v0.2 新增）
| 防线 | 含义 | 缺失后果 |
|---|---|---|
| 特征离线/在线一致性 | 训练特征与实盘特征同 schema 同计算 | 训练-实盘偏差 |
| 训练切分 manifest | split cutoff 冻结、可复现 | 不可复现、信息穿越 |
| 模型 artifact/hash | 模型版本化、hash 可追溯 | 模型版本漂移 |

#### 2.2.4 交易生产门禁组（v0.2 新增）
| 防线 | 含义 | 缺失后果 |
|---|---|---|
| 成本/滑点/成交可得性 | 成本模型与成交约束前置 | 回测不可执行化 |
| 读审计与 run-id 贯通 | 数据→研究→执行审计链同 run-id | 无法复盘、不可审计 |
| 决策时点-新鲜度对齐 | 信号时刻 vs available_at 不可前视 | 实盘 lookahead |

---

## 3. 整改清单（v0.2：维度扩至 9，含新增项）

> 标注：**优先级** P0/P1/P2；**类别** (a)已设计未实现 / (b)设计过期或缺失 / (c)范围扩展 / **(d) 遗漏分析驱动新增**。优先级分两套语义：`[湖]`=数据湖内部整洁度、`[产]`=策略生产解锁价值；冲突时以 `[产]` 为准。

### 3.1 命名与分区层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| N1 | `run_id` 当长期分区键，prices 38 分区膨胀 | 全流程 | 新数据 `dataset/schema_version/` + current + archive；CR 转 catalog 元数据 `triggered_by_cr` | `lake_layout.py`、`cli.py:1871`、`normalization.py:808` | P0 | (c) |
| N2 | 前缀不一致 + `unknown` run_id | 收集 | 新 run 统一 `run-<purpose>-<window>-<source>-<YYYYMMDD>`；修 `unknown` bug | run_id 生成处 | P1 | (b) |
| N3 | CR 编号烧进路径 | 维护 | 新规约不放 CR；存量不主动改名 | 无 | P1 | (b) |

### 3.2 数据正确性层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| C1 | **PIT 未强制**，无 as-of panel reader → 前视偏差 | 回测/ML 红线 | 实现 PIT as-of reader；写时盖戳、读时过滤 | `readers.py` 新增 `read_panel_as_of`、`catalog.py` 物化 `pit_status` | **P0** | (a) HLD §17.7.1/§14 |
| C2 | **分区重叠**，glob-concat 38 分区 | 全流程 | **读层去重**优先（按 source_run_id 取最新）；物理分区收敛后置 | `readers.py:375/409` | **P0** | (b) |
| C3 | `events` schema 全 `null` 类型 | 收集/特征 | 修写入侧类型推断，重跑 events | `normalization.py` events 分支 | P1 | (b) |
| C4 | 写入无去重保证 | 收集/模拟盘 | 写 canonical 前按主键去重或写入后校验唯一 | `normalization.py`、`validation.py` | P1 | (b) |

### 3.3 读取与查询层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| R1 | 无统一 published/as-of **panel** reader（非"全库只认 prices"） | 回测/ML | 新增 `read_panel(datasets, as_of)`，复用 `read_dataset` published 门禁 | `readers.py:2728` 复用、新增 panel 聚合 | **P0** | (c) |
| R2 | ML 绕过 lake 走 `--data-dir`/`load_local_frames` | ML | ML 接入 `read_panel_as_of`，废除旁路 | `experiments/run_experiment_15_*.py`、`23_29_*.py` | P1 | (c) |
| R3 | pandas concat 全量加载撑不住 | 回测/ML/实盘 | 引 DuckDB 只读 adapter（护栏已就位） | `duckdb_query.py`、`pyproject.toml`、`readers.py` | P1 | (a) HLD §17.6/§17.7 |
| R4 | 无列裁剪/谓词下推 | 实盘 | DuckDB 接入自然解决；过渡期 reader 支持下推 | `readers.py` | P2 | (a) |

### 3.4 版本与快照层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| V1 | candidate/gold/published 全空，`published: false` | 回测/ML/模拟盘 | 接通 promote：canonical → published + catalog current pointer 固定；补完 `publish.py` | `publish.py`、`catalog.py`、`cli.py` | **P0** | (a) HLD §5/§17.4 |
| V2 | 无训练快照概念 | ML | ML 只读 published 快照，training cutoff 固定 | `readers.py` published-only、ML 脚本 | P1 | (c) |
| V3 | 无特征/标签持久化层，每次从原始 lake 重算 | ML | 新增 `features/` 层带版本 + schema；feature/label/artifact 审计链 | 新模块 | **P1**（v0.1 P2，评审升） | (c) |
| V4 | schema 版本有字段无演进/兼容策略 | 实盘契约 | 定义 schema 演进规则 + reader 兼容回退；**模拟盘前契约冻结**（评审 HIGH3：不放实盘运维尾部） | `contracts.py`、`readers.py` | **P1**（v0.1 P2，遗漏分析升；v0.3 排期移入 Wave2） | (c) |

### 3.5 元数据与血缘层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| M1 | `catalog.json` 与 `manifest.jsonl` 双真相源 | 全流程 | 定 catalog 为主、manifest 为派生 | `catalog.py` 或 `manifest.py` 写入方 | P1 | (b) |
| M2 | `lineage_checksum` 17/17 缺失，raw→canonical 未闭合 | 收集 | 写 canonical 时回填 `lineage_checksum` | `manifest.py`、`normalization.py` | P2 | (a) |
| M3 | `quality/` 平铺 81 项 + smoke/probe 混入 | 维护 | 按数据集/日期分区；smoke/probe 进 `_scratch/` + 保留策略 | 质量报告写入处 | P2 | (b) |
| M4 | CR→数据审计链断 | 审计 | catalog 加 `triggered_by_cr`、`run_lineage` | `catalog.py` | P2 | (c) |

### 3.6 增量与实盘层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| L1 | 增量刷新计划合同已存在（`incremental.py` CR014-S06 `plan_incremental_refresh`），但未接通真实日级 append + 幂等写 + 当日去重的执行链 | 模拟盘/实盘 | 既有计划合同接通执行：`available_at` 盖戳 + 幂等写 + 当日去重 | `incremental.py`、`normalization.py`、`cli.py` 增量命令 | P1 | (d2) |
| L2 | published pointer 合同已存在（`publish.py:605 publish_current_pointer`，当前 dry-run only），未接通真实前移执行 | 模拟盘/实盘 | 既有合同接通真实执行：每日 promote → current pointer 前移 + 门禁 | `publish.py`、`catalog.py` | P1 | (d2) |
| L3 | 无读审计（reader 未记录 read audit log，与 run evidence/broker facts run-id 未贯通） | 实盘/可审计 | 新增 reader 读审计 log；与既有 RunEvidenceIndex 同 run-id 贯通 | `readers.py`、新 audit 模块 | **P1** | (d1) |
| L4 | readiness matrix 已存在（`readiness.py:462 build_readiness_matrix`），未前置为实盘信号生成前的强制阻断门禁 | 实盘 | 既有 readiness 前置为读前强制 gate（coverage/新鲜度/PIT），不通过阻断 | `readiness.py`、`readers.py` | P1 | (d2) |
| L5 | replay 合同已部分存在（`replay.py`、`cli.py:437 cmd_p0_replay`、`backup_restore.py:321`），未接通按 published as_of 重放单日快照 | 实盘 | 既有 replay 接通 published as_of 单日快照重放 | `replay.py`、`readers.py` | P2 | (d2) |

### 3.7 研究与 ML 审计链层（v0.2 新增维度，v0.3 修正既有对象表述）

> 类别细化：(c)=范围扩展闭环、(d1)=遗漏分析驱动的新建对象、(d2)=遗漏分析驱动的既有对象闭环。

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| E1 | ExperimentManifest 已存在但未与 published data release + lineage 闭环 | 全流程可复现 | 既有 `ExperimentManifest`（`engine/research_manifest.py:152`）补 dataset snapshot + as_of + split + 产出引用 | FEAT-03/11 产物 | P1 | (c) |
| E2 | 模型 artifact/hash 未与 dataset snapshot 闭环 | ML 可复现 | 模型 artifact 带 hash 引用 dataset snapshot（既有 StrategyAdmissionPackage 可承载） | FEAT-03 | P1 | (c) |
| E3 | 泄漏检查已分散存在但未统一接入 data release / feature-label cutoff / purge-embargo gate | ML | 统一既有泄漏审计（`factor_model_validation.py:561` label_available_at、`:376` embargo、`factor_robustness.py:53` ml_leakage_audit、`validation.py` 执行价泄漏防护）到 data release + cutoff gate | 既有校验统一 | P1 | (d2) |
| E4 | 无特征离线/在线一致性校验 | ML/实盘 | 训练特征与实盘特征同 schema 同计算的校验 | 新校验 | P1 | (d1) |
| E5 | 训练切分 manifest 未冻结（既有 embargo/split 策略未与 release 闭环） | ML 可复现 | split cutoff 入既有 ExperimentManifest，与既有 purge-embargo 统一 | FEAT-03 | P1 | (d2) |

### 3.8 交易生产层（v0.2 新增维度，v0.3 修正既有对象表述）

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| T4 | BrokerLakeSchema 已存在但未与实盘写 + 审计链闭环 | 实盘可审计 | 既有 `trading/broker_lake.py:64` schema 接通实盘写 + 订单/成交/持仓审计链 | FEAT-06 | P1 | (c) |
| T5 | CommissionSchedule 已存在但未前置到回测/实盘成本门禁 | 回测可执行化 | 既有 `trading/qmt_gateway_contracts.py:997` 接通成本/滑点/成交可得性前置 | FEAT-12 | P1 | (d2) |
| T6 | 数据 run-id 未贯穿既有 RunEvidenceIndex / broker event | 全流程可审计 | 数据 run-id 贯穿既有 `trading/strategy_runner/evidence_index.py:19` | 跨 FEAT-02/03/06/11 | P1 | (d2) |

### 3.9 测试与护栏层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| T1 | DuckDB 只读边界有测试但 adapter 空 | ML | adapter 实现后补 e2e 只读测试 | `tests/test_cr014_duckdb_*.py` | P2 | (a) |
| T2 | 无 PIT 正确性回归测试 | 回测/ML | 构造"未来财报"用例，断言 as-of reader 读不到 | 新测试 | P1 | (c) |
| T3 | 无去重正确性测试 | 全流程 | 断言 `(symbol,trade_date)` 唯一 | 新测试 | P1 | (c) |
| **T7** | 无整改回归基线/黄金值对比机制 | 整改可归因 | 整改前后跑下游因子/回测黄金值快照对比 | 新机制 | P1 | (d1) |
| **T8** | 无整改对象自动化清册/审计基线 | 整改可核查 | 一行命令输出全 dataset 清册（行数/覆盖/重复键/pit/published/lineage） | 新脚本 | P1 | (d1) |

### 3.10 配置类事实源层（v0.2 新增维度，遗漏分析驱动）

> 这是 v0.1 完全遗漏的策略生产配置事实源层，被"数据湖=市场数据"锚定而遗漏。但对象性质为**既有合同闭环/版本化扩展，不是纯新建对象**——F1-F4 对应的 benchmark contract、CommissionSchedule、PortfolioRiskPolicy、policy_cycle_coverage 合同均已存在，整改是补版本化事实源 + release 闭环。

| ID | 问题 | 影响 | 修法 | 归属 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| F1 | benchmark contract/resolver 已存在但缺版本化 benchmark / risk-free curve 事实源及与 release/回测/归因闭环 | 回测/因子评价 | 既有 `market_data/benchmarks.py:99 BenchmarkCoverage`/`:114 BenchmarkDefinition`/`:177 dataset requirements` 基础上，补版本化 benchmark 与无风险利率曲线事实源 + release 闭环 | FEAT-02 benchmark policy | P1 | (d2) |
| F2 | CommissionSchedule 合同已存在但缺版本化费用模型事实源及与 release 闭环 | 回测/实盘 | 既有 `trading/qmt_gateway_contracts.py:997` CommissionSchedule 基础上，补版本化佣金/费用/滑点模型事实源 + release 闭环 | FEAT-12 | P1 | (d2) |
| F3 | PortfolioRiskPolicy 合同已存在（`engine/mature_multifactor_framework.py:228`），缺版本化 universe/risk policy 事实源及与 release 闭环 | 模拟盘/实盘 | 既有 PortfolioRiskPolicy 基础上补版本化 universe policy + risk policy 事实源（退市/ST/容量约束）+ release 闭环 | FEAT-14 | P1 | (d2) |
| F4 | 政策周期评估已存在（`engine/factor_model_validation.py:444 policy_cycle_coverage`、`config/policy_cycles.yaml`），缺版本化政策周期/broker shortability 事实源及与 release 闭环 | 因子/交易 | 既有政策周期评估基础上补版本化政策周期 + shortability 事实源 + release 闭环 | FEAT-03 | P2 | (d2) |

### 3.11 跨源与时点一致性层（v0.2 新增维度，遗漏分析驱动）

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| X1 | 复权因子本身 PIT 正确性未校验 | 回测 | 除权除息事件按 PIT 应用；复权断点回归测试 | `normalization.py`、CR017 复权 | P1 | (d1) |
| X2 | 跨源（tushare/jqdata/QMT）交易日历/时区一致性 | 全流程 | 跨源交易日历对齐校验；时区归一 | `trade_calendar`、跨源 | P1 | (d1) |
| X3 | decision_time 已部分支持（`readers.py:227/1790` decision_time 过滤），缺实盘信号生成时刻 vs available_at 的强制 lookahead 门禁 | 实盘 | 既有 decision_time 基础上加强制 lookahead 阻断门禁 | `readers.py`、实盘读 | P1 | (d2) |
| X4 | 幸存者偏差已识别（`engine/contracts.py:270` survivorship_bias_note、`is_pit_universe=false`），缺按时点构建 universe 的 PIT 成分链 | 回测 | index_members snapshot 按时点构建 PIT universe，消除固定成分快照偏差 | FEAT-14 | P1 | (d2) |

### 3.12 类别统计（v0.6 机械统计口径，修正 v0.5 (a)/(b) 未同步 + F1/F2 标签矛盾）

> v0.5 把 L1/L2/L4/L5/X3/X4/F3/F4 改归 d2，但 (a)/(b) 计数和 F1/F2 标签未同步。v0.6 用脚本对 §3 全表逐行机械统计重算。

| 类别 | 数量 | 实际项 | 设计动作 |
|---|---|---|---|
| (a) 已设计未实现 | **6 项** | C1、R3、R4、V1、M2、T1 | 不刷新设计，走 CR + LLD 落地 |
| (b) 设计过期/缺失 | **7 项** | N2、N3、C2、C3、C4、M1、M3 | 刷新对应设计章节 |
| (c) 范围扩展 | **12 项** | N1、R1、R2、V2、V3、V4、M4、E1、E2、T4、T2、T3 | CR → CP2 → CP3，刷新蓝图+HLD+矩阵 |
| **(d1) 遗漏分析-纯新建对象** | **6 项** | L3、E4、T7、T8、X1、X2 | 需评审确认是否纳入范围（D8a） |
| **(d2) 遗漏分析-既有合同闭环/扩展** | **14 项** | L1、L2、L4、L5、E3、E5、T5、T6、F1、F2、F3、F4、X3、X4 | 不重复设计，只做集成闭环/版本化扩展（D8b） |

> 计数口径（v0.6 机械统计，逐项点数 6+7+12+6+14=45）：(a)(b) 由 v0.5 的"9/9"修正为实际"6/7"——v0.5 把 L2/L5 从 (a) 改 d2、L1/L4 从 (b) 改 d2 时未同步 (a)/(b) 计数。F1/F2 由 v0.5 的 (d1) 修正为 (d2)（benchmark 合同 `benchmarks.py:99`、CommissionSchedule `qmt_gateway_contracts.py:997` 已存在）。E1/E2/T4 归 (c) 不计 D8。遗漏分析驱动合计 d1+d2 = **20 项**。计数演进链：13(v0.2误) → 15(v0.4误) → 20(v0.6 机械统计确认)。

---

## 4. 设计刷新判断

### 4.1 需要刷新的文档

| 文档 | 刷新原因 | 刷新内容 | 责任角色 |
|---|---|---|---|
| `docs/design/BLUEPRINT.md` | 范围扩展至全数据底座 | 增补"策略生产数据底座"能力地图；明确 ML feature 层、配置类事实源层、run evidence、broker facts、交易审计链的 Feature 归属和数据归属 | meta-se |
| `process/HLD-DATA-LAKE.md` | (1) 停在 CR018 未过 CP3、与实现漂移；(2) 范围未覆盖 ML feature/读侧 join/训练快照/读审计/配置层 | **新开 companion HLD `Strategy Data Foundation`**（评审 D2 命名），覆盖市场数据湖 + 研究数据集 + ML feature/label + run evidence 边界；写侧/读侧分离 | meta-se |
| `docs/design/FEATURE-DESIGN-MATRIX.md` | 缺 ML feature 层 / 读侧语义 / run evidence / 配置层 Feature 归属 | 新增或扩展 Feature，补 `lld_policy` 和重访条件；明确 V3/L3/L4/R2/E*/T*/F*/X* 归属 | meta-se |
| `docs/features/market-data-lake/` | stub 停在 CR014，严重过期 | 重写 DESIGN/TASKS/TEST-PLAN 覆盖新整改项 | meta-dev（CP5 后） |

### 4.2 不需要刷新设计的项

类别 (a) 的 6 项（C1、R3、R4、V1、M2、T1）：HLD 已有契约，LLD 消费即可。

> 注：L2、L5 在 v0.5 已从 (a) 改归 (d2) 既有合同闭环（`publish.py:605`、`replay.py` 已有合同），不再属"已设计未实现"，而是"既有合同缺执行闭环"；N1 是 (c) 范围扩展。这些归类变化影响"哪些只走 LLD、哪些需补集成闭环"的路由，v0.6 同步修正。

### 4.3 协议约束（评审需知）

- 本次是**范围/需求类变更**，按 Meta Flow 必须 CR-first（规则 25/26）。
- CR 后回 `requirement-clarification`，CP2 过基线、CP3 过 HLD 人工门禁后才能拆 Story 和 LLD（规则 1/2）。
- HLD/蓝图只能 meta-se 改；feature DESIGN 三件套是 meta-dev 在 CP5 后落地（规则 8）。
- CP3 前不得实现，除非 fast-lane 且不命中架构/文件所有权冲突（规则 2/HLD 锁）。

---

## 5. 落地路线图（v0.2 Wave1 重排：先冻结基线，不先改物理分区）

### Wave 1 — 冻结基线 + 信任基础（P0，解锁回测）

> 评审重排：**先冻结现状基线和重复画像，再实现 published pointer/read selector，再做读层去重，最后做候选压缩/迁移。不要先改物理分区**，否则"历史数据变化"和"结构修复"混在一起无法归因。

1. **T8** 整改对象自动化清册（全 dataset 行数/覆盖/重复键/pit/published/lineage）
2. **T7** 整改回归基线 + 黄金值快照（下游因子/回测结果冻结对比基准）
3. **C2a** 重复画像（先看清 38 分区里哪些 `(symbol,trade_date)` 重复、来自哪些 run）
4. **V1** 实现 published pointer / read selector（先建立"读哪个"的确定性，再谈"读得对不对"）
5. **C1** PIT as-of reader（`financial_pit`/`market_cap` 按 `available_at` 过滤）
6. **R1** 统一 published/as-of panel reader（复用 `read_dataset` published 门禁）
7. **C2b** 读层去重（按 source_run_id 取最新版本）
8. **N1 后置** 候选压缩/物理分区迁移（最后做，且基于 T7 黄金值对比归因）

### Wave 2 — 接入与规模（P1，解锁 ML + 模拟盘）

> 评审 HIGH3：**V4 schema 契约冻结必须在模拟盘前完成**，从 Wave3 移入 Wave2，不放实盘运维尾部。

9. **V4** schema 演进规则 + 实盘契约冻结（模拟盘前必过）
10. **R2** ML 接入 lake，废除 `--data-dir`/`load_local_frames` 旁路
11. **V3** feature/label/artifact 层 + E1/E2/E5 ExperimentManifest/模型 hash/split manifest
12. **R3** DuckDB 只读 adapter
13. **L1/L2** 日级增量 + pointer 前移
14. **E3/E4** label 泄漏检查 + 离线/在线一致性
15. **F1/F2/F3** 配置类事实源层
16. **T4/T5/T6** broker facts + 成本前置 + run-id 贯通（既有对象闭环）
17. **X1/X2/X3/X4** 跨源与时点一致性
18. **L3/L4** 读审计 + 读前质量门禁
19. **T2/T3** PIT/去重正确性测试
20. N1/C3/M1 收尾

### Wave 3 — 实盘与运维（P2）

21. L5 replay、R4 延迟优化、T1、M3/M4、F4

### CR 编排建议

```
新 parent CR「Strategy Data Foundation」(评审 D1 同意新开)
 ├─ CP2: 范围基线 — 拆成 数据湖整改 + 研究数据集/ML + 读侧语义 + 配置层 + 交易审计链 五块需求
 ├─ CP3:
 │   ├─ 刷新 BLUEPRINT.md
 │   ├─ 新建 HLD「Strategy Data Foundation」（写侧/读侧分离，D2 命名）
 │   ├─ 刷新 FEATURE-DESIGN-MATRIX.md
 │   └─ 过 CP3 人工门禁（D7：四组门禁为 CP3 必过）
 ├─ CP4/CP5: 按 Wave 1 P0（T8/T7/V1/C1/R1/C2）拆 Story + LLD
 └─ CP6-CP8: 实现 → 验证 → 交付
```

---

## 6. 待评审决策项（D1-D7 已同意或附条件；D8 经二轮拆分）

| 决策 ID | 待确认问题 | 推荐方案 | 评审结论 | 备注 |
|---|---|---|---|---|
| D1 | 收口 CR018 还是新开 parent CR？ | 新开 parent CR | 同意 | — |
| D2 | 扩 HLD 还是新 companion HLD？ | 新开，命名「Strategy Data Foundation」 | 同意（附命名） | 覆盖市场数据湖+研究数据集+ML feature/label+run evidence |
| D3 | 存量 run_id 一次性重命名？ | 不重命名，新数据规范化 + 旧数据下次重跑替换 | 同意 | — |
| D4 | DuckDB 只读引擎还是主存储？ | 只读查询引擎，parquet 仍是存储 | 同意 | — |
| D5 | ML feature 层 lake 子层还是独立 store？ | lake `features/` 子层带版本 | 可接受（附条件） | 保留切换到独立 feature store 的条件 |
| D6 | C2 前强制基准快照？ | 是，强制 | 同意 | v0.2 升级为 T7 整改回归基线机制 |
| D7 | 四道防线做 CP3 门禁？ | 是，但扩展为"数据/研究/ML/交易生产门禁集" | 同意（附扩展） | v0.2 §2.2 已扩为四组门禁 |
| **D8a** | (d1) 纯新建对象 6 项是否纳入范围？ | 纳入 | 二轮建议批准 | L3/E4/T7/T8/X1/X2（逐项点数 1×6=6），按新建能力处理 |
| **D8b** | (d2) 既有合同闭环/扩展 14 项是否纳入范围？ | 纳入，按"既有合同接通闭环/版本化扩展"处理，**不重复设计** | 二轮建议批准（附口径） | L1/L2/L4/L5/E3/E5/T5/T6/F1/F2/F3/F4/X3/X4（逐项点数 1×14=14）；E1/E2/T4 归 (c) 不计 D8 |

---

## 7. 评审清单（逐项核对）

- [ ] §1 现状核查事实是否准确（v0.1–v0.4 事实错误是否已全部修正、有无新遗漏）
- [ ] §1.5/§1.6 顺序与既有合同存在性是否准确（含 v0.5 自检新增 8 行：L1/L2/L4/L5/X3/X4/F3/F4）
- [ ] §2.2 四组门禁是否构成信任基础的充分集
- [ ] §3 整改是否覆盖信息收集/回测/模拟盘/实盘全流程全数据资产层
- [ ] §3 每项"问题"列是否已避免"把已有写成无"（v0.5 自检修正 8 项后是否还有残留）
- [ ] §3.12 (a)(b)(c)(d1)(d2) 分类与计数是否正确，逐项点数 d1 6 + d2 14 = 20 是否对得上
- [ ] §4 设计刷新范围是否恰当
- [ ] §5 Wave 2 V4 契约冻结是否前置到模拟盘前（评审 HIGH3 是否已修正）
- [ ] §6 D8a（纯新建 6）/ D8b（既有闭环 14）互斥口径是否正确，E1/E2/T4 是否已归 (c) 不计 D8
- [ ] §8 遗漏分析是否还有未发现的整改面

---

## 8. 遗漏原因分析驱动的整改面核查（v0.2 新增，v0.6 机械统计口径）

> 本节回答用户要求：用六条遗漏原因逐条当镜头反查，确认是否还有未分析到的整改面。结论：有，已并入 §3。按 v0.6 机械统计口径，遗漏分析驱动整改面共 **20 项**，拆为 (d1) 纯新建 **6 项**（L3/E4/T7/T8/X1/X2）+ (d2) 既有合同闭环 **14 项**（L1/L2/L4/L5/E3/E5/T5/T6/F1/F2/F3/F4/X3/X4）；E1/E2/T4 归 (c) 不计 D8。逐条核查如下。

### 8.1 原因1「范围锚定过窄」→ 反查：还有哪些数据资产层未列
- **已并入**：研究数据集/因子面板（E1）、feature/label（V3）、模型 artifact/hash（E2）、run evidence（E1）、broker facts（T4）、订单/成交/持仓审计链（T4）。
- **反查新发现**：**配置类事实源的版本化与 release 闭环缺失**（F1 benchmark/无风险利率、F2 commission/费用、F3 universe/risk policy、F4 政策周期/shortability）。v0.6 修正：benchmark（`benchmarks.py:99`）、CommissionSchedule（`qmt_gateway_contracts.py:997`）、PortfolioRiskPolicy（`mature_multifactor_framework.py:228`）、policy_cycle_coverage（`factor_model_validation.py:444`）已有合同或评估入口，**非"完全缺失"**，而是缺版本化事实源 + release 闭环。→ 新增 §3.10，四项均归 (d2)。

### 8.2 原因2「事实核查不彻底/过度泛化」→ 反查：核查机制本身缺什么
- **已并入**：v0.1 三条事实错误修正（§1.3.1）。
- **反查新发现**：**整改清单本身缺乏"事实校验前置 + 自动化清册"**。评审 Wave1 重排已要求先冻结基线，但更深一层是缺一个"整改对象自动化清册"（一行命令输出全 dataset 行数/覆盖/重复键/pit/published/lineage），以及"整改回归基线/黄金值对比机制"。→ 新增 T7、T8。

### 8.3 原因3「防线视角错（正向推 vs 反向推）」→ 反查：还有哪些失真源未覆盖
- **已并入（评审已提）**：幸存者偏差、复权、交易状态、label 泄漏、离线/在线一致性、split manifest、成本可得性、读审计 run-id 贯通。
- **反查新发现**：
  - **PIT 复权**（X1）：复权因子本身有 PIT 问题，除权除息事件时点正确性未校验——评审提了"复权与公司行动"但未细化到复权因子 PIT。
  - **跨源一致性**（X2）：tushare/jqdata/QMT 跨源交易日历/时区对齐——评审未提。
  - **决策时点-新鲜度对齐**（X3）：实盘信号时刻 vs available_at 的 lookahead 门禁——L4 提了新鲜度但未明确决策时点对齐。
  - **universe 时变成分**（X4）：index_members snapshot 按时点构建 universe（幸存者偏差的细分）——评审提了幸存者偏差但未细化到成分时变。
  → 新增 §3.11。

### 8.4 原因4「优先级按湖修复难度排」→ 反查：还有哪些优先级误判
- **已并入**：V3 P2→P1（评审提）。
- **反查新发现**：
  - **V4 schema 演进 P2→P1**：实盘前必须 schema 契约冻结，否则 runner 读湖会因 schema 变更崩——从生产视角应 P1。→ V4 已升。
  - **L3 读审计 P2→P1**："可审计"是用户三大目标之一（可信/可复现/可审计），读审计贯穿全流程，不应只 P2——v0.1 误判。→ L3 已升。
- **结构改进**：优先级分两套语义 `[湖]`/`[产]`，冲突以生产价值为准——避免再次混排。

### 8.5 原因5「Wave1 缺先冻结基线」→ 反查：基线机制是否完整
- **已并入（评审重排）**：先冻结基准+重复画像，不先改物理分区。
- **反查新发现**：**基线不止"重复画像"，还缺"下游结果黄金值对比 + 整改归因"**。v0.1 说"先跑基准快照对比"但没定义快照存哪、怎么对比、谁负责、如何归因"结构修复 vs 历史数据变化"。→ 升级为 T7 整改回归基线机制（含黄金值快照、对比、归因）。

### 8.6 原因6「没拿蓝图当范围 checklist」→ 反查：Feature 归属是否系统化
- **已并入**：§1.5 蓝图 Feature 边界表。
- **反查新发现**：**整改清单缺"Feature 归属映射"**。30+ 项整改散落，但没系统标注每项归 FEAT-0X、跨边界项怎么归属。→ §3 各表已加"归属/代码触点"列；跨边界项（T6 run-id 贯通跨 FEAT-02/03/06/11）显式标注。

### 8.7 反查总结

| 遗漏原因 | 已并入项 | 反查新发现（(d) 类） |
|---|---|---|
| 1 范围锚定 | E1/E2/V3/T4 | F1/F2/F3/F4（配置类事实源层） |
| 2 核查不彻底 | §1.3.1 修正 | T7/T8（基线与清册机制） |
| 3 防线视角 | §2.2 研究门禁组 | X1/X2/X3/X4（跨源与时点一致性） |
| 4 优先级误判 | V3 升 | V4 升、L3 升、优先级两套语义 |
| 5 缺冻结基线 | Wave1 重排 | T7 升级（黄金值对比+归因） |
| 6 没用蓝图 | §1.5 Feature 表 | Feature 归属列系统化 |

**六条原因反查后净新增遗漏分析驱动整改面**（计数演进：v0.2 误计 13 → v0.3 误计 11/4 括号对不上 → v0.4 互斥口径 15 → v0.5 自检发现 8 项误把已有写成"无"改归 d2 → v0.6 机械统计确认 20）。按 v0.6 机械统计口径拆分（E1/E2/T4 归 (c) 不计 D8）：
- **(d1) 纯新建 6 项** = L3 + E4 + T7 + T8 + X1 + X2（无既有合同）
- **(d2) 既有合同闭环/扩展 14 项** = L1 + L2 + L4 + L5 + E3 + E5 + T5 + T6 + F1 + F2 + F3 + F4 + X3 + X4（既有合同已存在，缺闭环/版本化/前置）
- 合计 20 项，逐项点数 6+14=20 可复核。

> v0.5 自检新增归 d2 的 8 项：L1（incremental.py 已有）、L2（publish_current_pointer 已有）、L4（build_readiness_matrix 已有）、L5（replay.py/cmd_p0_replay 已有）、X3（decision_time 已有）、X4（survivorship_bias_note 已识别）、F3（PortfolioRiskPolicy 已有）、F4（policy_cycle_coverage 已有）。v0.6 进一步把 F1（benchmark 合同）、F2（CommissionSchedule）从 d1 改归 d2。这 10 项 v0.2–v0.5 曾误写成"无"或误归 d1，v0.6 修正为"既有合同缺闭环"。

评审需确认 D8a（纯新建 6 项）/ D8b（既有合同闭环 14 项，不重复设计）是否纳入范围。

---

## 附录 A：核查依据文件路径

- 数据湖：`data/lake/`（canonical/quality/catalog/manifest/raw）
- 读写代码：`market_data/readers.py`（`read_canonical:2691`、`read_dataset:2728`、published 门禁 `:2796`、`_canonical_paths:375`）、`market_data/lake_layout.py`、`market_data/duckdb_query.py`、`market_data/publish.py`、`market_data/catalog.py`、`market_data/normalization.py`
- ML 实验：`experiments/run_experiment_15_*.py`（`load_local_frames`）、`experiments/run_experiment_23_29_ml_factor_suite.py:132`（`--data-dir`）
- 设计文档：`process/HLD-DATA-LAKE.md`、`docs/design/BLUEPRINT.md`、`docs/design/FEATURE-DESIGN-MATRIX.md`、`docs/features/market-data-lake/`
- 依赖声明：`pyproject.toml`
- 蓝图 Feature 边界：`docs/design/BLUEPRINT.md:84-95,175-176`

## 附录 B：HLD-DATA-LAKE 历史覆盖项

> 注：本表记录 HLD-DATA-LAKE 各章节历史上覆盖的整改项，用于追溯设计来源。是否属于当前类别 (a) 以 §3.12 机械统计为准——其中 L2、L5 经 v0.5/v0.6 自检已改归 (d2) 既有合同闭环（`publish.py:605`、`replay.py` 合同已存在），不再属 (a)。

| HLD 章节 | 历史覆盖整改项 | 当前类别 |
|---|---|---|
| §5 Quality/Readiness/Publish | V1、L2 | V1=(a)；L2=(d2) |
| §14 PIT universe / `available_at_rule` | C1 | C1=(a) |
| §17.4 catalog current pointer / 增量 / replay | V1、L2、L5 | V1=(a)；L2/L5=(d2) |
| §17.6/§17.7 DuckDB 只读 query/audit | R3、R4、T1 | 均=(a) |
| §17.7.1 写入时序与读写边界 | C1、M2 | 均=(a) |
| §6 真实回补与恢复 | L5 | L5=(d2) |

## 附录 C：v0.1 → v0.2 变更摘要（历史记录；计数口径已演进，当前 v0.6 机械统计为 d1 6 + d2 14 = 20，见附录 G。下表中"13/15/11/4"均为各历史版本当时口径，非当前）

| 变更点 | v0.1 | v0.2 |
|---|---|---|
| §1.3 read_dataset | 漏判"全库只认 prices" | 修正：通用 `read_dataset` 已存在且 published 阻断，问题是 panel reader+ML 接入缺失 |
| §1.3.1 pit_status | "全部 null"（错误） | 修正：14 null + 3 pit_available |
| §2.2 防线 | 4 道（必要不充分） | 扩为 4 组门禁（数据/研究/ML/交易生产） |
| §3 维度 | 7 维 30 项 | 9 维 + 配置层/跨源一致性，含 (d) 类新增（v0.2 误计 13，v0.4 修正 15） |
| §3.4 V3 | P2 | P1（评审升） |
| §3.4 V4 | P2 | P1（遗漏分析升） |
| §3.6 L3 | P2 | P1（遗漏分析升） |
| §4 D2 HLD | "HLD-STRATEGY-DATA-BASE" | "Strategy Data Foundation"（评审命名） |
| §5 Wave1 | C2 先收敛分区 | 重排：T8 清册→T7 基线→C2a 画像→V1 pointer→C1→R1→C2b 读层去重→N1 后置 |
| §6 D8 | 无 | 新增：(d) 类是否纳入（v0.2 误计 13，v0.4 修正为 D8a 11 + D8b 4） |
| §8 | 无 | 新增：遗漏原因反查（v0.2 误计 13，v0.4 修正 15） |

## 附录 D：v0.2 → v0.3 变更摘要（二轮评审返修）

| 变更点 | v0.2 | v0.3 |
|---|---|---|
| §1.4 文档版本 | BLUEPRINT v1.11 / MATRIX v1.10（陈旧） | 修正：v1.12 / v1.11（评审 HIGH4） |
| §1.6 既有合同 | 缺失，v0.2 把已有对象写成"无" | 新增 §1.6：ExperimentManifest / StrategyAdmissionPackage / RunEvidenceIndex / BrokerLakeSchema / CommissionSchedule 已存在，整改为"既有对象闭环"非新建（评审 BLOCKING1） |
| §3.7/§3.8 表述 | "无 ExperimentManifest/broker facts/CommissionSchedule" | 修正为"既有对象未与数据湖 release/lineage/run-id 闭环"；类别细化 (c)/(d1)/(d2) |
| §3.10/§3.11 类别 | (d) 笼统 | 拆 (d1) 新建 / (d2) 既有闭环 |
| §3.12 计数 | "(d) 13 项"（错误） | 修正：遗漏分析驱动 15 项 = 新建 11 (d1) + 既有闭环 4 (d2)（评审 BLOCKING2） |
| §3.4 V4 排期 | P1 但 Wave3 放运维 | 修正：移入 Wave2，模拟盘前契约冻结（评审 HIGH3） |
| §5 Wave2/3 | V4 在 Wave3 | V4 移入 Wave2 第 9 项 |
| §6 D8 | 单项"13 项是否纳入" | 拆 D8a（新建 11 项）/ D8b（既有闭环 4 项，不重复设计）（评审 MEDIUM5） |
| §7 评审清单 | 未含二轮修正项 | 补 BLOCKING1/2、HIGH3、MEDIUM5 核对项 |
| §8.7 计数 | "净新增 13 项" | 修正：15 项 = 11 新建 + 4 既有闭环 |

## 附录 E：v0.3 → v0.4 变更摘要（三轮评审返修）

| 变更点 | v0.3 | v0.4 |
|---|---|---|
| §3.12 / §6 / §8.7 D8 计数 | "d1 11/d2 4"但括号列项数对不上（11 列了 12 项、4 列了 6 项），连续两轮同类错误 | 互斥口径修正：d1 新建 11 = F1-4+X1-4+E4+T7+T8（逐项点数 4+4+1+1+1）；d2 既有闭环 4 = E3+E5+T5+T6（1+1+1+1）；E1/E2/T4 归 (c) 不计 D8（评审 BLOCKING） |
| §3.7 E3 | "无 label 泄漏检查机制"（把已有写成无，同类问题） | 修正为"泄漏检查分散存在未统一接入 data release/cutoff/embargo gate"；E3 从 (d1) 改 (d2)；补既有位置 `factor_model_validation.py:561/376`、`factor_robustness.py:53`（评审 HIGH） |
| §3.10 F1 | "无 benchmark/无风险利率曲线事实源"（过宽） | 修正为"benchmark contract/resolver 已存在，缺版本化事实源 + release 闭环"；补既有位置 `benchmarks.py:99/114/177`（评审 MEDIUM） |
| §1.5/§1.6 顺序 | §1.6 出现在 §1.5 前 | 对调：§1.5 在前，§1.6 在后（评审 LOW） |
| §1.6 既有合同表 | 缺泄漏检查、benchmark 行 | 补两行（E3、F1 既有能力） |
| 附录 C | 保留 v0.2 "13 项"未标注 | 标注"v0.2 误计 13，v0.4 修正 15"（评审 LOW） |

## 附录 F：v0.4 → v0.5 变更摘要（自检返修，非评审驱动）

> v0.5 是应"自检是否还有同类遗漏和错误"要求做的全量代码核验，发现 BLOCKING1/E3/F1 同类病系统性复发。

| 变更点 | v0.4 | v0.5 |
|---|---|---|
| §3.6 L1 | "无规范当日增量更新语义" | 修正：`incremental.py` CR014-S06 `plan_incremental_refresh` 已有，缺执行链；(b)→(d2) |
| §3.6 L2 | "无 published pointer 前进机制" | 修正：`publish.py:605 publish_current_pointer` 已有（dry-run），缺真实前移；(a)→(d2) |
| §3.6 L4 | "无读前质量门禁" | 修正：`readiness.py:462 build_readiness_matrix` 已有，缺前置阻断；(c)→(d2) |
| §3.6 L5 | "无 replay 能力" | 修正：`replay.py`/`cli.py:437 cmd_p0_replay` 已有，缺 published as_of 单日重放；(a)→(d2) |
| §3.10 F3 | "无 universe/risk policy 事实源" | 修正：`mature_multifactor_framework.py:228 PortfolioRiskPolicy` 已有，缺版本化事实源 + release 闭环；(d1)→(d2) |
| §3.10 F4 | "无政策周期/shortability 事实源" | 修正：`factor_model_validation.py:444 policy_cycle_coverage` 已有，缺版本化事实源 + release 闭环；(d1)→(d2) |
| §3.11 X3 | "无决策时点对齐门禁" | 修正：`readers.py:227/1790 decision_time` 已部分支持，缺强制 lookahead 门禁；(d1)→(d2) |
| §3.11 X4 | "无 universe 时变成分对齐" | 修正：`engine/contracts.py:270 survivorship_bias_note` 已识别，缺 PIT 成分链；(d1)→(d2) |
| §3.12 计数 | d1 11 + d2 4 = 15 | 重算：d1 纯新建 6 + d2 既有闭环 14 = 20（8 项从 d1/他类改归 d2） |
| §6 D8a/D8b | 11/4 | 6/14 |
| §1.6 既有合同表 | 8 行 | 补 8 行（L1/L2/L4/L5/X3/X4/F3/F4 既有能力） |
| §8.7 | 15 项 | 20 项，附计数演进链 13→15→20 |

**自检根因**：v0.2–v0.4 写"问题"列时未逐项核验代码，导致"把已有合同写成无"反复出现。v0.5 对 §3 全表 30+ 项逐项 grep 代码核验，修正 8 项。剩余仍标"无"的项（L3 读审计、E4 离线/在线一致性、T7/T8 基线机制、X1 复权 PIT 校验、X2 跨源对齐、C1/C2/R1/V1 等）经核验确认代码确无对应能力，表述准确。

## 附录 G：v0.5 → v0.6 变更摘要（四轮评审返修，机械统计消除计数矛盾）

> v0.5 改了 d2 但未同步 (a)/(b) 计数和 F1/F2 标签，导致全文口径矛盾。v0.6 用脚本对 §3 全表逐行机械统计，以统计结果为准统一全文。

| 变更点 | v0.5 | v0.6 |
|---|---|---|
| §3.12 (a) 计数 | 9 项（未同步 L2/L5 改 d2） | 机械统计 6 项：C1/R3/R4/V1/M2/T1 |
| §3.12 (b) 计数 | 9 项（未同步 L1/L4 改 d2） | 机械统计 7 项：N2/N3/C2/C3/C4/M1/M3 |
| §3.10 F1 标签 | (d1) | (d2)（benchmark 合同 `benchmarks.py:99` 已存在，评审 BLOCKING2） |
| §3.10 F2 标签 | (d1) | (d2)（CommissionSchedule `qmt_gateway_contracts.py:997` 已存在，评审 BLOCKING2） |
| §4.2 不需刷新设计 | "类别 (a) 9 项含 L2/L5/N1" | 修正：(a) 6 项 C1/R3/R4/V1/M2/T1；L2/L5 已改 (d2)，N1 是 (c)（评审 HIGH3） |
| §8 开头口径 | "15 项 = d1 11 + d2 4" | "20 项 = d1 6 + d2 14"（评审 MEDIUM4） |
| §8.1 配置类表述 | "配置类事实源层完全缺失" | "配置类事实源的版本化与 release 闭环缺失"（已有合同，评审 MEDIUM5） |
| §8.7 末行 | "D8a 新建 11 / D8b 既有 4" | "D8a 纯新建 6 / D8b 既有闭环 14" |
| 附录 C 标题 | "v0.4 修正为 15" | 标注"历史口径，当前 v0.6 为 20" |

**v0.6 机械统计结果（脚本逐行解析 §3 表格标签，可复核）**：(a)=6, (b)=7, (c)=12, (d1)=6, (d2)=14，总计 45。遗漏分析驱动 d1+d2=20。此结果与 §3.12、§6 D8a/D8b、§8.7 全文一致。

## 附录 H：v0.6 → v0.7 变更摘要（五轮评审返修，文字/附录残留定稿）

> 五轮评审确认核心口径已收敛，仅余 3 处文字/附录残留。v0.7 修完后可进入审批链路。

| 变更点 | v0.6 | v0.7 |
|---|---|---|
| §3.10 小节说明 | "本层为新建对象"（与 F1-F4 已归 d2 冲突，评审 HIGH） | 改为"既有合同闭环/版本化扩展，不是纯新建对象" |
| 附录 B 标题 | "HLD-DATA-LAKE 已设计覆盖项（类别 a 来源）"（表内含已改 d2 的 L2/L5，误导，评审 MEDIUM） | 改为"HLD-DATA-LAKE 历史覆盖项"，加注"是否属当前 (a) 以 §3.12 为准"，补"当前类别"列 |
| §8.7 版本口径 | "按 v0.5 互斥口径拆分"（应为 v0.6，评审 LOW） | 改为"按 v0.6 机械统计口径拆分" |

**v0.7 不改机械统计口径**：(a)=6, (b)=7, (c)=12, (d1)=6, (d2)=14, 总 45，d1+d2=20，与 v0.6 一致。本轮仅修文字/附录残留。
