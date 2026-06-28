---
title: 数据湖模块整改交接评审文档
status: draft-for-review
audience: 评审人
created_at: 2026-06-28
source_investigation: 数据湖目录核查 + 代码耦合分析 + 设计文档比对
review_scope: 整改对象 / 防线 / 目标 / 细节 / 设计刷新必要性 / 落地路线
---

# 数据湖模块整改交接评审文档

> 本文档用于外部评审。评审目标：判断整改对象是否完整、防线是否到位、目标是否对齐"多因子 + ML 策略全流程生产"、落地路线是否可行。本文不含实现，只含判断与依据。

## 0. 评审请求摘要

| 项 | 内容 |
|---|---|
| **整改目标** | 数据湖模块支撑多因子策略 + 机器学习策略，从信息收集 → 回测 → 模拟盘 → 实盘，全流程可信、可复现、可审计 |
| **整改对象** | `data/lake/` 数据湖（外部存储，软链入仓，已 gitignore）+ `market_data/` 读写代码 + 相关设计文档 |
| **评审重点** | (1) 30 项整改是否覆盖全流程；(2) 四道防线（PIT/去重/快照/join）是否到位；(3) 设计是否需要刷新；(4) 落地路线是否可行 |
| **不评审** | 具体代码实现、CR 编号分配、发布判定 |

---

## 1. 现状核查（整改对象的事实依据）

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

### 1.2 数据集清单（catalog 已登记 17 个）

价格类：`prices`、`prices_hfq`、`prices_limit`、`adj_factor`、`hs300_index`
财务/估值类：`financial_pit`、`market_cap`、`events`
参考类：`stock_basic`、`trade_calendar`、`trade_status`、`index_members`、`index_weights`、`industry_classification`、`bse_code_mapping`、`lifecycle_code_change`、`liquidity_capacity`

**字段底子是好的**：
- `prices` = `symbol × trade_date`，含 raw + adjusted OHLCV + `adj_factor` + `available_at` + `available_at_rule`
- `financial_pit` 带 `ann_date`、`report_period`、`available_at`、`available_at_rule`（PIT 原料齐全）
- `market_cap` 带 pe/pb/ps/turnover + `available_at`
- 全部 parquet 列式存储

### 1.3 关键代码事实

| 事实 | 位置 | 含义 |
|---|---|---|
| `read_canonical` 只认 prices，其余 raise | `market_data/readers.py:2700` | 多数据集 join reader 缺失 |
| reader glob-concat 全部分区 | `readers.py:376 _canonical_paths`、`:409` | prices 下 38 个 `run_id` 分区全读，`(symbol,trade_date)` 可能重复 |
| DuckDB 只读护栏已写，adapter 空 | `market_data/duckdb_query.py` | `import duckdb` 全库 0 处；`pyproject.toml` 无 duckdb 依赖；策略默认 `ENGINE_FALLBACK` |
| ML 实验绕过 lake | `experiments/run_experiment_15_*.py load_local_frames` | ML 从本地目录读，数据湖对 ML 是摆设 |
| 路径构造集中在 `lake_layout.py` | `market_data/lake_layout.py` | 单一咽喉，结构改动代码触点可控 |
| catalog 全部 `published: false` | `data/lake/catalog/catalog.json` | medallion 后三层 + publish 流未接通 |
| catalog 全部 `pit_status: null` | 同上 | PIT 字段采到但未强制执行 |
| catalog `lineage_checksum: null` | 同上 | raw→canonical 血缘校验链未闭合 |
| `events` 数据集 schema 全 `null` 类型 | `data/lake/canonical/events/` | 8 part 坏数据，写入侧类型推断 bug |
| `quality/` 含 `cr010-...-smoke-v2/v3`、`unknown` run_id | `data/lake/quality/` | 一次性产物混入生产报告，命名不一致 |

### 1.4 设计文档现状

| 文档 | 版本/状态 | 问题 |
|---|---|---|
| `process/HLD-DATA-LAKE.md` | v0.8, `confirmed: false`, `draft-pending-cp3-cr018` | **停在 2026-05-29，CR017/CR018 从未过 CP3，与实现漂移** |
| `docs/design/BLUEPRINT.md` | v1.11, CR-138 | FEAT-02 范围是"数据生产"，未覆盖"策略生产数据底座" |
| `docs/design/FEATURE-DESIGN-MATRIX.md` | v1.10, CR-131 | 缺 ML feature 层 / 读侧语义的 Feature 归属 |
| `docs/features/market-data-lake/` | 2026-06-21 stub (3KB/1.4KB/2KB) | 停在 CR014 时代，既不反映 CR018 也不反映新目标 |

---

## 2. 整改目标与四道防线

### 2.1 目标分解（按全流程四阶段）

| 阶段 | 数据湖必须提供 | 当前是否满足 |
|---|---|---|
| **信息收集** | 规范写入、PIT 盖戳、去重、CR 审计、血缘 | 部分（写入在，PIT/去重/血缘未闭合） |
| **回测** | PIT 安全面板、去重、冻结快照、多表 join | **不满足**（PIT 未强制、分区重叠、无快照、join 缺失） |
| **模拟盘** | 日级增量、pointer 前移、读前质量门禁 | 不满足（增量非规范化、pointer 无前移、无门禁） |
| **实盘** | 低延迟读、schema 契约稳定、读审计、replay | 不满足（pandas concat 慢、无读审计、replay 未通） |

### 2.2 四道防线（信任基础，P0）

> 评审核心：这四道防线若不补，下游模拟盘/实盘都建立在不可信数据上。这是整改进度的咽喉。

| 防线 | 含义 | 缺失后果 | 当前状态 |
|---|---|---|---|
| **防线一：PIT 强制** | 读财报/估值按 `available_at <= as_of` 取最新，杜绝前视 | ML 标签泄露，模型虚高、实盘崩 | 字段采到，reader 未执行，`pit_status: null` |
| **防线二：去重唯一** | `(symbol, trade_date)` 在 canonical 唯一 | 重复行污染标签和特征，不可复现 | reader glob-concat 38 分区，可能多份 |
| **防线三：冻结快照** | published current pointer 固定，回测只读快照 | 反复调参偷看未来，结果不可复现 | 三层全空，`published: false` |
| **防线四：多表 join** | 价格 × 财务 × 估值 × 行业统一 as-of join 宽表 | ML 无法用 lake，被迫旁路 | `read_canonical` 只认 prices，ML 走 `load_local_frames` |

---

## 3. 整改清单（30 项，七维度）

> 标注：**优先级** P0/P1/P2；**类别** (a)已设计未实现 / (b)设计过期或缺失 / (c)范围扩展。

### 3.1 命名与分区层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| N1 | `run_id` 当长期分区键，prices 38 分区膨胀 | 全流程 | 新数据 `dataset/schema_version/` + current + archive；CR 转 catalog 元数据 `triggered_by_cr` | `lake_layout.py`、`cli.py:1871`、`normalization.py:808` | P0 | (c) |
| N2 | 前缀不一致（`cr010-`/`run-cr010-`/`run-stage3-`）+ `unknown` run_id | 收集 | 新 run 统一 `run-<purpose>-<window>-<source>-<YYYYMMDD>`；修 `unknown` bug | run_id 生成处 | P1 | (b) |
| N3 | CR 编号烧进路径 | 维护 | 新规约不放 CR；存量不主动改名 | 无（规约+元数据） | P1 | (b) |

### 3.2 数据正确性层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| C1 | **PIT 未强制**，无 as-of reader → 前视偏差 | 回测/ML 红线 | 实现 PIT as-of reader；写时盖戳、读时过滤 | `readers.py` 新增 `read_panel_as_of`、`catalog.py` 物化 `pit_status` | **P0** | (a) HLD §17.7.1/§14 已设计 |
| C2 | **分区重叠**，glob-concat 38 分区 | 全流程 | 收敛分区 + reader 按 source_run_id 取最新版本去重 | `readers.py:376/409` | **P0** | (b) |
| C3 | `events` schema 全 `null` 类型（8 part 坏数据） | 收集/特征 | 修写入侧类型推断，重跑 events | `normalization.py` events 分支 | P1 | (b) |
| C4 | 写入无去重保证 | 收集/模拟盘 | 写 canonical 前按主键去重或写入后校验唯一 | `normalization.py`、`validation.py` | P1 | (b) |

### 3.3 读取与查询层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| R1 | `read_canonical` 只认 prices，其余 raise | 回测/ML | 放开全数据集 或 新增 `read_panel(datasets, as_of)` | `readers.py:2700` | **P0** | (c) |
| R2 | ML 绕过 lake 走 `load_local_frames` | ML | ML 接入 `read_panel_as_of`，废除旁路 | `experiments/run_experiment_15_*.py`、`23_29_*.py` | P1 | (c) |
| R3 | pandas concat 全量加载撑不住 | 回测/ML/实盘 | 引 DuckDB 只读 adapter（护栏已就位）：谓词下推+列裁剪+并行扫描 | `duckdb_query.py` 填 adapter、`pyproject.toml` 加可选组、`readers.py` 接调用 | P1 | (a) HLD §17.6/§17.7 已设计 |
| R4 | 无列裁剪/谓词下推 | 实盘 | DuckDB 接入自然解决；过渡期 reader 支持下推 | `readers.py` | P2 | (a) |

### 3.4 版本与快照层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| V1 | candidate/gold/published 全空，`published: false` | 回测/ML/模拟盘 | 接通 promote：canonical → published + catalog current pointer 固定；补完 `publish.py` | `publish.py`、`catalog.py`、`cli.py` | **P0** | (a) HLD §5/§17.4 已设计 |
| V2 | 无训练快照概念，每次读"当前最新" | ML | ML 只读 published 快照，training cutoff 固定 | `readers.py` published-only 模式、ML 脚本 | P1 | (c) |
| V3 | 无特征/标签持久化层，每次从原始 lake 重算 | ML | 新增 `features/` 层带版本 + schema；feature store | 新模块 | P2 | (c) |
| V4 | schema 版本有字段无演进/兼容策略 | 实盘 | 定义 schema 演进规则 + reader 兼容回退 | `contracts.py`、`readers.py` | P2 | (c) |

### 3.5 元数据与血缘层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| M1 | `catalog.json` 与 `manifest.jsonl` 双真相源 | 全流程 | 定 catalog 为主、manifest 为派生 | `catalog.py` 或 `manifest.py` 写入方 | P1 | (b) |
| M2 | `lineage_checksum: null`，raw→canonical 未闭合 | 收集 | 写 canonical 时回填 `lineage_checksum` | `manifest.py`、`normalization.py` | P2 | (a) HLD 已设计 |
| M3 | `quality/` 平铺 81 项 + smoke/probe 混入 | 维护 | 按数据集/日期分区；smoke/probe 进 `_scratch/` + 保留策略 | 质量报告写入处 | P2 | (b) |
| M4 | CR→数据审计链断 | 审计 | catalog 加 `triggered_by_cr`、`run_lineage` | `catalog.py` | P2 | (c) |

### 3.6 增量与实盘层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| L1 | 无规范"当日增量更新"语义 | 模拟盘/实盘 | 规范日级 append：`available_at` 盖戳 + 幂等写 + 当日去重 | `normalization.py`、`cli.py` 增量命令 | P1 | (b) |
| L2 | 无 published pointer 前进机制 | 模拟盘/实盘 | 每日 promote：canonical → published current pointer 前移 + 门禁 | `publish.py`、`catalog.py` | P1 | (a) HLD §17.4 已设计 |
| L3 | 无读审计 | 实盘 | reader 记录 read audit log（dataset, as_of, run_id, caller） | `readers.py`、新 audit 模块 | P2 | (c) |
| L4 | 无读前质量门禁前置到信号生成 | 实盘 | 实盘读前强制 quality gate（coverage/新鲜度/PIT），不通过阻断 | `readiness.py`、`readers.py` | P1 | (c) |
| L5 | 无 replay/重跑单日能力 | 实盘 | 支持按 `as_of` 重放某日 published 快照 | `readers.py` as-of 基础 | P2 | (a) HLD §17.4 已设计 |

### 3.7 测试与护栏层

| ID | 问题 | 影响 | 修法 | 代码触点 | 优先级 | 类别 |
|---|---|---|---|---|---|---|
| T1 | DuckDB 只读边界有测试但 adapter 空 | ML | adapter 实现后补 e2e 只读测试 | `tests/test_cr014_duckdb_*.py` | P2 | (a) |
| T2 | 无 PIT 正确性回归测试 | 回测/ML | 构造"未来财报"用例，断言 as-of reader 读不到 | 新测试 | P1 | (c) |
| T3 | 无去重正确性测试 | 全流程 | 断言 `(symbol,trade_date)` 唯一 | 新测试 | P1 | (c) |

### 3.8 类别统计

| 类别 | 数量 | 含义 | 设计动作 |
|---|---|---|---|
| (a) 已设计未实现 | 9 项 | HLD-DATA-LAKE 已写明，代码没做 | **不刷新设计**，走 CR + LLD 落地 |
| (b) 设计过期/缺失 | 9 项 | 与现状脱节或没写 | **刷新对应设计章节** |
| (c) 范围扩展 | 12 项 | 超出原 HLD"数据生产"范围 | **必须 CR → CP2 → CP3，刷新蓝图+HLD+矩阵** |

---

## 4. 设计刷新判断

### 4.1 需要刷新的文档

| 文档 | 刷新原因 | 刷新内容 | 责任角色 |
|---|---|---|---|
| `docs/design/BLUEPRINT.md` | 范围扩展：数据生产 → 策略生产数据底座，跨 FEAT-02/03/06 | 增补"策略生产数据底座"能力地图；明确 ML feature 层、simulation/live 读路径的 Feature 归属和数据归属 | meta-se |
| `process/HLD-DATA-LAKE.md` | (1) 停在 CR018 未过 CP3、与实现漂移；(2) 范围未覆盖 ML feature/读侧 join/训练快照/读审计 | 修正漂移；或新开 companion HLD（`HLD-STRATEGY-DATA-BASE`）专管策略消费读侧 + ML feature 层（写侧/读侧分离更干净） | meta-se |
| `docs/design/FEATURE-DESIGN-MATRIX.md` | 缺 ML feature 层 / 读侧语义 Feature 归属 | 新增或扩展 Feature，补 `lld_policy` 和重访条件；明确 V3/L3/L4/R2 归属 | meta-se |
| `docs/features/market-data-lake/` | stub 停在 CR014，严重过期 | 重写 DESIGN/TASKS/TEST-PLAN 覆盖新整改项 | meta-dev（CP5 后） |

### 4.2 不需要刷新设计的项

类别 (a) 的 9 项（C1 PIT、R3 DuckDB、R4 下推、V1 publish、M2 血缘、L2 pointer 前移、L5 replay、T1 DuckDB 测试、N1 部分已有）：HLD-DATA-LAKE 已有契约，**LLD 消费即可，不重设计**。

### 4.3 协议约束（评审需知）

- 本次是**范围/需求类变更**，按 Meta Flow 必须 CR-first（规则 25/26）。
- CR 后回 `requirement-clarification`，CP2 过基线、CP3 过 HLD 人工门禁后才能拆 Story 和 LLD（规则 1/2）。
- HLD/蓝图只能 meta-se 改；feature DESIGN 三件套是 meta-dev 在 CP5 后落地（规则 8）。
- CP3 前不得实现，除非 fast-lane 且不命中架构/文件所有权冲突（规则 2/HLD 锁）。

---

## 5. 落地路线图

### Wave 1 — 信任基础（P0，解锁回测）

1. **C2** 收敛 prices 分区 + reader 去重 → 先跑基准快照对比，区分"结构修复带来的数据变化"vs"真 bug"
2. **C1** 实现 PIT as-of reader（`financial_pit`/`market_cap` 按 `available_at` 过滤）
3. **R1** 放开多数据集 reader / 新增 `read_panel_as_of`
4. **V1** 接通 published 快照 + catalog current pointer

> 评审要点：C2 收敛前 `read_canonical` 是"读全部叠一起"，收敛后下游因子/回测结果可能变化。**必须先跑基准快照**，否则无法区分"结构修复"与"真 bug"。

### Wave 2 — 接入与规模（P1，解锁 ML + 模拟盘）

5. **R2** ML 实验接入 lake，废除 `load_local_frames` 旁路
6. **R3** 引 DuckDB 只读 adapter
7. **L1/L2** 规范日级增量 + pointer 前移
8. **N1/C3/M1/L4/T2/T3** 配套收尾

### Wave 3 — 实盘与运维（P2）

9. V4 schema 演进、L3 读审计、L5 replay、R4 延迟优化、T1、M3/M4、V3 feature store

### CR 编排建议

```
新 parent CR「策略生产数据底座」(或收口 CR018)
 ├─ CP2: 范围基线 — 拆成 数据湖整改 + ML 接入 + 读侧语义 三块需求
 ├─ CP3:
 │   ├─ 刷新 BLUEPRINT.md
 │   ├─ 刷新/新建 HLD（写侧/读侧分离）
 │   ├─ 刷新 FEATURE-DESIGN-MATRIX.md
 │   └─ 过 CP3 人工门禁
 ├─ CP4/CP5: 按 Wave 1 P0（C1/C2/V1/R1）拆 Story + LLD
 └─ CP6-CP8: 实现 → 验证 → 交付
```

---

## 6. 待评审决策项

请评审人对以下决策给出意见：

| 决策 ID | 待确认问题 | 推荐方案 | 备选 |
|---|---|---|---|
| D1 | 整改收口到现有 CR018 还是新开 parent CR？ | 新开 parent CR（目标/节奏/审计边界与 CR018 不同） | 收口 CR018 |
| D2 | HLD 是扩 HLD-DATA-LAKE 还是新开 companion HLD？ | 新开 `HLD-STRATEGY-DATA-BASE`（写侧/读侧分离） | 扩展现有 |
| D3 | 存量 `run_id` 分区是否一次性重命名？ | 不重命名，新数据规范化 + 旧数据下次重跑自然替换 | 一次性改名 |
| D4 | DuckDB 作为只读查询引擎还是主存储？ | 只读查询引擎，parquet 仍是存储 | DuckDB 主存储（不推荐） |
| D5 | ML feature 层是 lake 子层还是独立 feature store？ | lake `features/` 子层带版本（V3） | 独立 feature store |
| D6 | C2 收敛分区前是否强制基准快照？ | 是，强制 | 否 |
| D7 | 四道防线（PIT/去重/快照/join）是否定为 CP3 必过门禁？ | 是 | 否 |

---

## 7. 评审清单（逐项核对）

评审人请逐项确认：

- [ ] §1 现状核查事实是否准确（是否有遗漏的整改对象）
- [ ] §2.2 四道防线是否构成信任基础的充分必要集
- [ ] §3 30 项整改是否覆盖信息收集/回测/模拟盘/实盘全流程
- [ ] §3.8 (a)(b)(c) 分类是否正确（哪些其实已设计、哪些真缺设计）
- [ ] §4 设计刷新范围是否恰当（有没有过度刷新或漏刷）
- [ ] §5 Wave 1 的 P0 排序是否合理（C2→C1→R1→V1 依赖是否正确）
- [ ] §6 七个决策项的推荐方案是否可接受
- [ ] 是否存在评审人认为缺失的整改维度或防线

---

## 附录 A：核查依据文件路径

- 数据湖：`data/lake/`（canonical/quality/catalog/manifest/raw）
- 读写代码：`market_data/readers.py`、`market_data/lake_layout.py`、`market_data/duckdb_query.py`、`market_data/publish.py`、`market_data/catalog.py`、`market_data/normalization.py`
- ML 实验：`experiments/run_experiment_15_*.py`、`experiments/run_experiment_23_29_ml_factor_suite.py`
- 设计文档：`process/HLD-DATA-LAKE.md`、`docs/design/BLUEPRINT.md`、`docs/design/FEATURE-DESIGN-MATRIX.md`、`docs/features/market-data-lake/`
- 依赖声明：`pyproject.toml`

## 附录 B：HLD-DATA-LAKE 已设计覆盖项（类别 a 来源）

| HLD 章节 | 覆盖整改项 |
|---|---|
| §5 Quality/Readiness/Publish | V1、L2 |
| §14 PIT universe / `available_at_rule` | C1 |
| §17.4 catalog current pointer / 增量 / replay | V1、L2、L5 |
| §17.6/§17.7 DuckDB 只读 query/audit | R3、R4、T1 |
| §17.7.1 写入时序与读写边界 | C1、M2 |
| §6 真实回补与恢复 | L5 |
