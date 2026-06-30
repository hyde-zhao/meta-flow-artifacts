---
story_id: "CR139-S13"
title: "R3 DuckDB 只读 adapter"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "a 已设计未实现，消费既有 HLD 契约"
  rationale: "R3 是 a 类，D4 只读引擎"
wave: "CR139-W2-ML-SIMULATION"
status: "verified"
req_coverage: ["REQ-216"]
depends_on: ["CR139-S09"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-216 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.6/§17.7"
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

# CR139-S13 R3 DuckDB 只读 adapter

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-216。Owner Feature: FEAT-02 读侧。Wave: CR139-W2-ML-SIMULATION。lld_policy: technical-note（a 已设计未实现，消费既有 HLD 契约；R3 是 a 类，D4 只读引擎）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-216 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.6/§17.7）
- `process/REQUIREMENTS.md` v1.17（REQ-216）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/duckdb_query.py`
- `pyproject.toml`
- `market_data/readers.py`

### 接口约定与设计约束
- 文件所有权：duckdb_query.py（独占）；pyproject.toml 依赖声明（CP5 批准才引入）；readers.py adapter 接入切片（merge_order 第 8 位）
- DuckDB 依赖引入需 CP5 批准（D4）
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
- DuckDB adapter 只读查询返回结果且不写持久事实源（FD-49）；parquet 仍是存储；.duckdb 持久事实源/DuckDB 写 lake/DuckDB 替代 catalog 均为违规。

## acceptance_criteria

DuckDB adapter 只读查询返回结果且不写持久事实源（FD-49）；parquet 仍是存储；.duckdb 持久事实源/DuckDB 写 lake/DuckDB 替代 catalog 均为违规。

量化验收：
- DuckDB adapter 只读查询返回结果且不写持久事实源（FD-49）；parquet 仍是存储；.duckdb 持久事实源/DuckDB 写 lake/DuckDB 替代 catalog 均为违规。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-216 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.6/§17.7
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- 消费既有 HLD-DATA-LAKE 契约（a 类）

## lld_policy

- required_level: `technical-note`
- trigger_reasons: a 已设计未实现，消费既有 HLD 契约
- rationale: R3 是 a 类，D4 只读引擎
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计依据

S13 覆盖 REQ-216：DuckDB 只能作为只读 query/audit adapter，Parquet + catalog current pointer 仍是事实源。当前仓库已存在 CR014 的 `market_data/duckdb_query.py` 和 `market_data/audit.py` 合同，实现了无硬依赖、可注入 adapter、默认 fallback 的只读边界；`pyproject.toml` 未声明 DuckDB。S13 采用该既有合同，补 `readers.py` 显式 adapter façade 与 CR139 Story 级 fixture 验证，不新增 DuckDB 依赖。

### 2. 文件影响

| File | Change | Boundary |
|---|---|---|
| `market_data/readers.py` | Add `read_dataset_via_duckdb_contract()` façade delegating to `duckdb_query.py`. | Reader-side adapter only; no write path. |
| `market_data/duckdb_query.py` | Reuse existing read-only query contract. | No new hard dependency. |
| `pyproject.toml` | No change. | DuckDB dependency change remains 0. |
| `tests/test_cr139_duckdb_readonly_adapter.py` | Add CR139 S13 fixture tests. | `tmp_path` only; no real lake. |

### 3. Interface Contract

`read_dataset_via_duckdb_contract()` accepts:

- `dataset`
- `catalog_pointer`
- `sql_template_id`
- `projections`
- `partition_filters`
- `policy`
- optional injected `adapter`
- `fallback_rows`

It must return either `ReadOnlyQueryResult` or `DuckDBBoundaryError` from `duckdb_query.py`.

Required behavior:

- published mode must read only from a valid catalog current pointer `published_path`/`canonical_path`;
- SQL must come from whitelisted read-only templates;
- no hard DuckDB import or dependency declaration is required;
- default no-adapter path uses fallback evidence with `duckdb_dependency_change=0`;
- `.duckdb` persistent files and source-of-truth updates remain 0.

### 4. S36/S37 Boundary

S36/S37 depend on S13 and can consume this façade after S13 is verified. S13 does not implement column pruning/predicate pushdown acceptance or DuckDB e2e test closure beyond the adapter contract.

### 5. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S13-DUCKDB-01 | Valid published pointer, no DuckDB adapter. | Fallback result returns rows, `duckdb_dependency_change=0`, no writes. |
| T-S13-DUCKDB-02 | Forbidden SQL template. | `DuckDBBoundaryError(code=forbidden_sql)` before execution. |
| T-S13-DUCKDB-03 | `.duckdb` sentinel. | No persistent `.duckdb` files created under fixture tmp path. |
| T-S13-DUCKDB-04 | Dependency declaration. | `pyproject.toml` contains no `duckdb` dependency. |

### 6. Non-goals

- No DuckDB package installation, no `pyproject.toml` / `uv.lock` dependency addition.
- No real lake read/list/write, no catalog/manifest/pointer write, no provider fetch, no NAS, no runtime/trading, no Git remote write.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S13-duckdb-readonly-adapter.md#技术说明` | ready-for-review | Defines no-hard-dependency DuckDB readonly reader façade. |
| CP5 自动预检 | `process/checks/CP5-CR139-S13-duckdb-readonly-adapter-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | No blocker; fixture/static implementation with dependency change 0. |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation evidence | `process/stories/CR139-S13-duckdb-readonly-adapter-IMPLEMENTATION.md` | complete | Implements reader-side DuckDB readonly façade without hard dependency. |
| CP6 自动检查 | `process/checks/CP6-CR139-S13-duckdb-readonly-adapter-CODING-DONE.md` | PASS | Targeted fixture/static tests passed; no dependency or lake writes. |
| Return packet | `process/returns/STORY-CR139-S13.CP6.return.json` | PASS | Story/stage return metadata for CP6. |
| Evidence index | `process/evidence/STORY-CR139-S13.CP6.index.json` | PASS | CP6 evidence references and validation snapshot. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A9-S13.md` | PASS | S13 static-fixture verification report. |
| CP7 自动检查 | `process/checks/CP7-CR139-S13-duckdb-readonly-adapter-VERIFICATION-DONE.md` | PASS | Story can be marked `verified`. |
| Return packet | `process/returns/STORY-CR139-S13.CP7.return.json` | PASS | Story/stage return metadata for CP7. |
| Evidence index | `process/evidence/STORY-CR139-S13.CP7.index.json` | PASS | CP7 evidence references and validation snapshot. |

## 依赖与文件所有权

- depends_on: CR139-S09
- dependency_type: ["contract"]
- 文件所有权: duckdb_query.py（独占）；pyproject.toml 依赖声明（CP5 批准才引入）；readers.py adapter 接入切片（merge_order 第 8 位）
- DuckDB 依赖引入需 CP5 批准（D4）

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
