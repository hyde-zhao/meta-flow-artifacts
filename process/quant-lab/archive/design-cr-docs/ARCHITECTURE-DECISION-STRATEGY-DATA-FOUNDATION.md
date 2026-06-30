---
title: "Architecture Decision Record — Strategy Data Foundation (CR-139)"
cr_id: "CR-139"
companion_hld: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
status: "confirmed-cp3"
complexity: "complex"
confirmed: true
confirmed_by: "user (CP3 approved via host-orchestrator)"
confirmed_at: "2026-06-28T17:30:00+08:00"
created_at: "2026-06-28T17:10:00+08:00"
---

# Architecture Decision Record — Strategy Data Foundation (CR-139)

> 本 ADR 落地 CR-139「Strategy Data Foundation」parent CR 的 D1-D8 决策 + Wave1 基线门（REQ-247）+ V4 模拟盘前冻结（HIGH3）+ AGA-1/3/5 架构灰区决策。ADR 结论已回写到 companion HLD 模块表/流程图（规则 12，不孤立 ADR）：D2 → HLD §1/§5 写侧/读侧分离结构；D4 → HLD §11/§10 DuckDB 只读；D5 → HLD §5/§10 ML feature 层；D7 → HLD §12.2 四组门禁 CP3 必过；D8a/D8b → HLD §17 lld_policy；Wave1 基线门 → HLD §12.3；V4 冻结 → HLD §12.6；AGA-1/3/5 → HLD §4/§5（CP3 已确认 A1/C1/E1）。

## Agent/Skill 组合方案

| 决策 ID | 决策 | 推荐方案 | 备选 | 影响 | 回退 / 切换条件 | 状态 |
|---|---|---|---|---|---|---|
| D1 | 收口 CR-018 还是新开 parent CR | 新开 parent CR CR-139 | 续推 CR-018 | 审计边界、范围 | — | 同意（CR-139 已建） |
| D2 | 扩 HLD 还是新 companion HLD | 新开 companion HLD「Strategy Data Foundation」，写侧/读侧分离 | 扩 HLD-DATA-LAKE | HLD 结构、范围收口 | — | 同意（附命名） |
| D3 | 存量 run_id 一次性重命名 | 不重命名，新数据规范化 + 旧数据下次重跑替换 | 一次性重命名 | N1/N2/N3 路由 | — | 同意 |
| D4 | DuckDB 只读引擎还是主存储 | 只读查询引擎，parquet 仍是存储 | DuckDB 主存储 | 依赖、事实源 | — | 同意 |
| D5 | ML feature 层 lake 子层还是独立 store | lake `features/` 子层带版本，附条件保留切换 | 直接独立 feature store | V3 物理归属 | feature 规模/在线 serving → 切换独立 store（DEF-139-01） | 可接受（附条件） |
| D6 | C2 前强制基准快照 | 强制（升级为 T7 整改回归基线 + 黄金值） | 不强制 | Wave1 归因 | — | 同意 |
| D7 | 四道防线做 CP3 门禁 | 是，扩展为"数据/研究/ML/交易生产门禁集" | 仅四道防线 | CP3 必过门禁 | — | 同意（附扩展） |
| D8a | (d1) 纯新建 6 项是否纳入 | 纳入（L3/E4/T7/T8/X1/X2），按新建能力 full-lld | 不纳入 | 范围 | — | CP2 用户 approve（2026-06-28T16:05） |
| D8b | (d2) 既有合同闭环 14 项是否纳入 | 纳入，按"既有合同接通闭环/版本化扩展"处理，不重复设计（technical-note） | 不纳入 | 范围 | — | CP2 用户 approve |
| Wave1 基线门 | 结构性变更顺序 | T8→T7→C2a 基线冻结后才能执行 V1/C1/R1/C2b/N1；N1 物理迁移后置 | 不设基线门 | Wave1 归因、REQ-247 | — | CP2 用户追加约束 |
| V4 冻结 | schema 契约冻结时机 | 模拟盘前必过（HIGH3，从 Wave3 移入 Wave2） | 实盘运维尾部 | 模拟盘稳定 | — | 评审 HIGH3 |
| AGA-1 | 写侧/读侧分离边界 | A1 分层同属 FEAT-02 + HLD 分读写章 | A3 拆 FEAT-02R | Feature 边界、Story owner | CP5 读写冲突 → 升级 A3 | **confirmed-cp3 (A1/C1/E1, user approved 2026-06-28T17:30:00+08:00)** |
| AGA-3 | ML feature 层归属与切换条件 | C1 lake features/ 子层 + DEF-139-01 | C2 独立 feature store（deferred） | V3 物理归属、依赖 | feature 规模/在线 serving → 切换 C2 | **confirmed-cp3 (A1/C1/E1, user approved 2026-06-28T17:30:00+08:00)** |
| AGA-5 | 配置类事实源版本化机制 | E1 复用 V1 pointer 语义 + config_facts 子目录 | E2 各类独立 registry | F1-F4 release 闭环 | policy_cycle 语义差异过大 → 该类独立 | **confirmed-cp3 (A1/C1/E1, user approved 2026-06-28T17:30:00+08:00)** |

## 平台适配差异

| 维度 | 当前平台 | 适配说明 |
|---|---|---|
| 数据湖存储 | parquet（`data/lake/` 软链外部存储，已 gitignore） | parquet 仍是存储；DuckDB 只读查询不替代 |
| DuckDB | 护栏已写（`duckdb_query.py`），adapter 空，`pyproject.toml` 无依赖 | CP3/CP5 批准才引入；只读 parquet/catalog |
| 读引擎 | `read_canonical`（prices 专用 fast reader）+ `read_dataset`（通用，published 门禁 `:2778`） | 新增 `read_panel_as_of` 复用 `read_dataset` published 门禁 |
| ML 接入 | `experiments/run_experiment_15_*.py load_local_frames`、`run_experiment_23_29_*.py:132 --data-dir` 旁路 | R2 废除旁路，接入 `read_panel_as_of`（FD-48） |
| 路径构造 | `lake_layout.py` 单一咽喉 | 新增 `features/` + `config_facts/` 子层，结构改动触点可控 |
| 配置类事实源 | benchmark（`benchmarks.py:99/114`）、commission（`qmt_gateway_contracts.py:997`）、PortfolioRiskPolicy（`mature_multifactor_framework.py:228`）、policy_cycle（`factor_model_validation.py:444`）合同已存在 | 复用 V1 pointer 语义版本化 + release 闭环（AGA-5 E1） |

## 设计确认点（需人工确认）

> CP3 人工门禁需确认的决策项。AGA-1/3/5 因 meta-se 当前环境 AskUserQuestion 工具不可用，已路由 host-orchestrator CP3 Decision Brief。

| 确认点 ID | 类型 | 待确认问题 | 推荐方案 | 备选 | 优劣 | 影响 / 风险 | 回退 / 切换 |
|---|---|---|---|---|---|---|---|
| CP3-DQ-01 (AGA-1) | architecture | companion HLD 写侧/读侧分离是否分层同属 FEAT-02 | A1 分层同属 FEAT-02 + HLD 分读写章 | A3 拆 FEAT-02R | A1 不增 Feature、可回退；A3 边界最清但增复杂度 | companion HLD 结构、Story owner、文件所有权 | CP5 读写冲突频繁 → 升级 A3 |
| CP3-DQ-02 (AGA-3) | architecture | ML feature 层是否 lake features/ 子层带版本（D5） | C1 lake features/ 子层 + DEF-139-01 切换条件 | C2 独立 feature store（deferred） | C1 不引入依赖、与 release 闭环一致；C2 在线 serving 友好但增依赖 | V3 物理归属、依赖、未来切换 | feature 规模/在线 serving → 切换 C2 另起 CR |
| CP3-DQ-03 (AGA-5) | architecture | 配置类事实源版本化是否复用 V1 published pointer 语义 | E1 复用 V1 pointer 语义 + config_facts 子目录 | E2 各类独立 registry | E1 机制统一、减分叉；E2 量身定制但四套机制 | F1-F4 release 闭环、publish.py 复用 | policy_cycle 语义差异过大 → 该类独立，其余复用 |

其余 D1-D8 + Wave1 基线门 + V4 冻结已在 CP2 用户 approve（2026-06-28T16:05:00+08:00）或评审定稿，CP3 仅作架构门禁复核，不重复决策。

## ADR 回写映射（规则 12，不孤立 ADR）

| ADR 决策 | 回写到 HLD 章节 | 回写到 BLUEPRINT/DOMAIN-MAP/DEPENDENCY-MAP |
|---|---|---|
| D2 写侧/读侧分离 | HLD §1/§5/§9/§10 | BLUEPRINT §CR-139 Feature/数据归属增量；DOMAIN-MAP OBJ-91..99 |
| D4 DuckDB 只读 | HLD §11/§10/§Gotchas#6 | DEPENDENCY-MAP FD-49 |
| D5 ML feature 子层 | HLD §5/§10/§17 | BLUEPRINT CAP-16/FEAT-03；DOMAIN-MAP OBJ-100..104 |
| D7 四组门禁 | HLD §12.2/§1.3 G-1 | DOMAIN-MAP RULE-54..56 |
| D8a/D8b lld_policy | HLD §17 | FEATURE-DESIGN-MATRIX §lld_policy 判定规则 |
| Wave1 基线门 | HLD §12.3 | DOMAIN-MAP RULE-60；DEPENDENCY-MAP FD-53 |
| V4 冻结 | HLD §12.6/§Gotchas#7 | DOMAIN-MAP OBJ-116/117/SM-31/RULE-59 |
| AGA-1/3/5 | HLD §4/§5/§20 | BLUEPRINT DQ-BP-CR139-01..03；FEATURE-DESIGN-MATRIX DQ-FD-CR139-01..03 |

## 不授权范围声明（REQ-248）

CP3 通过不授权：runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移（Wave1 N1 后置到基线冻结之后）/ Git remote write / 真实 lake 写入 / published pointer 前移执行 / broker lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。

## 变更记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-28 | meta-se | CR-139 CP3 首版 ADR：落 D1-D8 + Wave1 基线门 + V4 冻结 + AGA-1/3/5；ADR 回写映射；不授权范围声明。AGA-1/3/5 confirmed-cp3 (A1/C1/E1)。 |
| 0.2 | 2026-06-28 | host-orchestrator | CP3 approved，AGA-1/3/5 确认 A1/C1/E1，pending-cp3 → confirmed-cp3。 |
