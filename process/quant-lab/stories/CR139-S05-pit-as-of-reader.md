---
story_id: "CR139-S05"
title: "C1 PIT as-of reader"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "a 已设计未实现，消费既有 HLD 契约"
  rationale: "C1 是 a 类，消费既有 HLD 契约"
wave: "CR139-W1-STRUCTURAL-P0"
status: "lld-ready-for-review"
req_coverage: ["REQ-204"]
depends_on: ["CR139-S04"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-204 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.7.1/§14"
source_hld: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
feature_design_matrix: "docs/design/FEATURE-DESIGN-MATRIX.md v1.13"
story_backlog: "process/STORY-BACKLOG-CR139.md"
development_plan: "process/DEVELOPMENT-PLAN-CR139.yaml"
engagement_mode: "production"
scenario_subject_type: "target-artifact"
created_at: "2026-06-28T18:00:00+08:00"
delegated_by: "host-orchestrator"
---

# CR139-S05 C1 PIT as-of reader

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-204。Owner Feature: FEAT-02 读侧。Wave: CR139-W1-STRUCTURAL-P0。lld_policy: technical-note（a 已设计未实现，消费既有 HLD 契约；C1 是 a 类，消费既有 HLD 契约）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-204 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.7.1/§14）
- `process/REQUIREMENTS.md` v1.17（REQ-204）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/readers.py`
- `market_data/catalog.py`

### 接口约定与设计约束
- 文件所有权：readers.py read_panel_as_of 函数（merge_order 首位）；catalog.py pit_status 字段
- 四道 P0 防线之一（PIT 强制）；readers.py merge_order 首位
- 不授权范围：CP4 不授权 LLD 写作/实现/runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移执行/Git remote write。CP5 通过后才实现。
- 平台目标：日频离线研究 + 模拟盘场景；DuckDB 只读引擎（D4）；parquet 仍是存储。
- 命名规范：遵循 CR-139 §5 Wave 路线 + REQ-247 基线门顺序。

### AI 可执行任务清单（CP5 LLD 阶段）
1. 消费 HLD/ADR/MATRIX 契约，产出 full LLD（technical-note）或 Story 内 ## 技术说明（technical-note）。
2. 核验文件所有权切片与 merge_order，避免 readers.py/publish.py/catalog.py/normalization.py 冲突。
3. 标注跨边界 owner/consumer（如适用），消费方只读。
4. 验收标准量化、可验证。

## validation_context

### 验证入口
- 静态/fixture 验证（CP6/CP7 默认 static-only，runtime 需独立授权）。
- 验证命令：`uv run --python 3.11 pytest -q tests/test_cr139_*.py`（按 Story 测试文件）。

### 验证方式
- technical-note Story：full LLD 14 章 + fixture 测试 + 契约映射。
- technical-note Story：Story 内 ## 技术说明 + 既有合同接通闭环证据。

### 依赖环境
- 不读取 .env/凭据；不启动 runtime；不连接 QMT/NAS；不写真实 lake。
- DuckDB 依赖引入需 CP5 批准（D4，仅 S13 相关）。

### 关键验证场景
- 读 financial_pit/market_cap 时 as_of=T 只返回 available_at<=T 的最新记录；构造'未来财报'用例断言读不到（0 条泄漏）。

## acceptance_criteria

读 financial_pit/market_cap 时 as_of=T 只返回 available_at<=T 的最新记录；构造'未来财报'用例断言读不到（0 条泄漏）。

量化验收：
- 读 financial_pit/market_cap 时 as_of=T 只返回 available_at<=T 的最新记录；构造'未来财报'用例断言读不到（0 条泄漏）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-204 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.7.1/§14
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- 消费既有 HLD-DATA-LAKE 契约（a 类）

## lld_policy

- required_level: `technical-note`
- trigger_reasons: a 已设计未实现，消费既有 HLD 契约
- rationale: C1 是 a 类，消费既有 HLD 契约
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S04
- dependency_type: ["contract"]
- 文件所有权: readers.py read_panel_as_of 函数（merge_order 首位）；catalog.py pit_status 字段
- 四道 P0 防线之一（PIT 强制）；readers.py merge_order 首位

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。

## 技术说明

> 本技术说明是 CR-139 CP5 Wave1 LLD 批次的设计证据。S05 是 **a 类（已设计未实现）**，消费既有 HLD-DATA-LAKE 契约，不重复设计（REQ-249）。既有合同位置已 grep 核验（memory `verify-existing-code-before-claiming-absent`）。本说明不含实现；CP5 全量人工确认 + Wave1 基线门（REQ-247）+ S04 verified 后才允许实现。

### 设计依据

| 来源 | 章节 / ID | 消费内容 |
|---|---|---|
| HLD-DATA-LAKE | §17.7.1 写入时序与读写边界 | reader 只读 catalog current pointer 指向的 Parquet；candidate 未 publish 时返回 `candidate_unpublished`；PIT 字段缺 available_at 进入决策次数=0 |
| HLD-DATA-LAKE | §14 CR-011 因子研究数据补齐 | `available_at_rule`（explicit_timestamp / daily_close_fact / missing_required）；PIT dataset 缺 explicit available_at 不得进 production_strict；非价格类缺 available_at → `pit_incomplete`/`required_missing` |
| HLD-DATA-LAKE | §4.3 available_at_rule | events/PIT 用 `explicit_timestamp`；缺 `available_at` 时 `missing_required`，不得进入决策 |
| companion HLD | `HLD-STRATEGY-DATA-FOUNDATION.md` v0.2 §5/§12.2 | 读侧 PIT as-of reader C1；读侧四道 P0 防线之"PIT 强制" |
| ADR | `ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2 | D7 四组门禁；Wave1 基线门 REQ-247 |
| handoff | `DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3.2 C1 / §1.3 | 既有 `readers.py:2728 read_dataset`（通用，published 门禁）；`readers.py:227/1790 decision_time` 部分支持；`catalog.py` pit_status 14 null + 3 pit_available（§1.3.1） |
| REQ | REQ-204 / REQ-247 / REQ-248 / REQ-249 | C1 PIT as-of reader；基线门；不授权；既有合同闭环 |

### 文件影响

| 文件 | 切片 | merge_order 位 | 所有权动作 | 既有合同位置（已核验） |
|---|---|---|---|---|
| `market_data/readers.py` | 新增 `read_panel_as_of(dataset, as_of, ...)` 单表 PIT as-of 读函数 | readers_py merge_order **首位**（**S05**→S06→S07→S09→S25→S27→S28→S13→S16→S36） | 新增 PIT as-of 读函数；复用 `read_dataset` published 门禁；**不修改** `read_dataset`（`:2728`）/`read_canonical`（`:2691`）既有合同 | `:2728 def read_dataset`、`:2778 if not entry.published`（published 门禁，已核验；handoff 称 :2796 实为 :2778）、`:227/1790 decision_time`（部分支持，已核验）、`:2511/3138 available_at/decision_time 处理` |
| `market_data/catalog.py` | pit_status 字段消费 | catalog_py merge_order **S05 不在列**（S04→S33→S39） | S05 只**读** catalog 既有 `pit_status` 字段做 PIT 过滤判定；**不写** catalog（见偏离记录 open_item） | `:59 pit_status: str \| None`、`:483/505/611 pit_status 处理`（已核验，14 null + 3 pit_available 现状） |

> readers.py 是冲突最密集文件（10 Story 触及）。LCQ-02 已 resolved：函数切片 + merge_order 串行合并。S05 是 readers_py merge_order **首位**，无前置 readers.py 写冲突。S05 切片边界 = 新增 `read_panel_as_of` 单表 PIT 函数，**不修改** S06 的 panel 聚合、S07 的去重层、S09 的兼容回退、S25 的 decision_time 阻断（S25 消费 S05 读路径，merge_order 在 S05 之后）。

### 接口 / 数据变化

- **接口**：新增 `read_panel_as_of(dataset, as_of, ...)` 单表 PIT as-of 读函数。对 financial_pit/market_cap 等 PIT dataset，按 `available_at <= as_of` 取每个 (symbol) 的最新记录；构造"未来财报"用例（`available_at > as_of`）断言读不到（泄漏行=0）。
- **数据**：消费既有 `catalog.py:59 pit_status` 字段判定 dataset 是否 PIT 类（14 null + 3 pit_available 现状）；消费既有 `available_at`/`available_at_rule` 字段做 PIT 过滤。**无 schema 变更**，不写 lake。
- **复用既有合同**：`read_dataset`（`:2728`）published 门禁（`:2778`）—— S05 的 PIT 过滤在 published 门禁**之后**执行，即先保证只读 published current truth，再做 PIT as-of 过滤。`decision_time`（`:227/1790`）部分支持由 S25（X3 lookahead 阻断）后续加强，S05 不修改 decision_time 既有逻辑。
- **跨边界**：S05 是 PIT as-of **生成侧**（owner=FEAT-02 读侧），消费方 = S06 R1 panel reader（调 `read_panel_as_of` 做 PIT 过滤）、S23 X1 复权 PIT、S25 X3 decision_time、S27 L3 读审计、S28 L4 readiness、S29 T2 PIT 正确性回归。消费方只读，不反向修改 reader。

### 失败路径

| 场景 | 前置校验 | 失败行为 | 回退 |
|---|---|---|---|
| dataset 未 published | 复用 `read_dataset` published 门禁（`:2778`） | 返回 structured `unavailable`/`candidate_unpublished` | 等 S04 published pointer 前移（S05 不前移） |
| PIT dataset 缺 `available_at` | `available_at_rule=missing_required`（HLD §4.3） | 返回 `required_missing`/`pit_incomplete`，不静默用当前快照伪装 PIT | HLD §14.4 契约；不进入 production_strict |
| `as_of` 早于任何 `available_at` | PIT 过滤后空集 | 返回空 DataFrame + `required_missing` reason | 调用方调整 as_of 或降级 |
| catalog `pit_status` 为 null（14/17 现状） | S05 读 pit_status 判定 PIT 类 | 对 null pit_status 的 dataset 按 dataset 名单（financial_pit/market_cap/events）显式判定 PIT，不依赖 catalog 标记 | 后续 M1（S33）/ S09 V4 可补 catalog pit_status 标记 |

### 测试入口

- 静态 / fixture 验证（CP6/CP7 默认 static-only，runtime 需独立授权）。
- 验证命令：`uv run --python 3.11 pytest -q tests/test_cr139_pit_as_of_reader.py`。
- 关键用例：读 financial_pit/market_cap 时 as_of=T 只返回 available_at<=T 的最新记录；构造"未来财报"用例（available_at=T+5）断言 read_panel_as_of(as_of=T) 读不到（0 条泄漏）；dataset 未 published 返回 unavailable。
- 既有合同接通闭环证据：`read_dataset` published 门禁复用 + `available_at`/`available_at_rule` PIT 过滤。

### 风险

| 风险 | 级别 | 应对 |
|---|---|---|
| catalog.py 文件所有权：S05 不在 catalog_py merge_order（S04→S33→S39），但 story card 列 catalog.py 为输出 | 中 | S05 只**读** catalog 既有 `pit_status`（`:59`，已核验）；若实现时需"物化 pit_status 值"（写 catalog），必须拆出 file-conflict 依赖到 S33 或独立切片，不能自行写 catalog.py（见偏离记录） |
| 14/17 catalog pit_status=null 现状下 PIT 判定依赖 dataset 名单 | 低 | S05 对 null pit_status 按 dataset 名单（financial_pit/market_cap/events）显式判定 PIT；后续 M1（S33）/ V4（S09）补 catalog pit_status 标记 |
| readers.py merge_order 首位，与 S06/S07 切片边界 | 低 | LCQ-02 已 resolved：S05 只新增 `read_panel_as_of`，不改 S06 panel 聚合 / S07 去重层；CP5 review 确认函数级边界 |
| 基线门时序被绕过 | 高 | REQ-247 硬依赖；S05 depends_on S04（contract）；dev_gate 标注；CP5/CP6 校验 S01/S02/S03/S04 verified |
| published 门禁位置漂移（handoff :2796 vs 实际 :2778） | 低 | 已 grep 核验为 :2778；本说明以 :2778 为准 |

### 偏离记录

- **无设计偏离**。S05 是 a 类，消费既有 HLD-DATA-LAKE §17.7.1/§14 契约，不重复设计（REQ-249）。
- **合同位置修正**：handoff v0.7 §1.3 称 published 门禁在 `readers.py:2796`，grep 核验实际为 `:2778`（HLD Gotchas#1 亦引用 :2778）。本说明以 :2778 为准。
- **catalog.py 文件所有权 open_item**：story card / DEVELOPMENT-PLAN 列 catalog.py 为 S05 输出（"catalog.py pit_status 字段"），但 catalog_py merge_order（S04→S33→S39）不含 S05。S05 对 catalog.py 的触点限定为**只读**既有 `pit_status` 字段（`:59`，已核验存在）。若实现时发现需**写** catalog 物化 pit_status 值，则升级为 file-conflict 依赖（与 S04 current pointer / S33 M1 定主 串行），不自行写 catalog.py。此为 open_item，CP5 review 确认。
- **基线门硬依赖**：S05 depends_on S04（contract），属 Wave1 基线门结构性变更（REQ-247），实现前必须 S01/S02/S03/S04 verified。
- **CP6 硬门禁（CP5 评审升级 2026-06-28）**：S05 对 `catalog.py` 是**只读**（仅读既有 `pit_status` 字段做 PIT 判定），**严禁写 catalog.py**。若实现时需补 catalog 写入（如物化 pit_status 值），必须**拆到 S33（M1 catalog 主从）或独立设计切片**，走 file-conflict 依赖与 S04/S33 串行，不得在 S05 内自行写 catalog。CP6 实现门禁必须校验 S05 的 catalog.py 写操作计数=0，违反则阻断。此约束已写入 CP5 批次 result.json `cp6_hard_gate` 字段 + per-story precheck。

### lld_gate

- required: true
- required_level: technical-note
- status: ready-for-review（本技术说明已写入，等 CP5 全量人工确认）
