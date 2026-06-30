---
story_id: "CR139-S36"
title: "R4 列裁剪/谓词下推"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "a 已设计未实现，消费既有 HLD 契约"
  rationale: "R4 是 a 类"
wave: "CR139-W3-OPS"
status: "verified"
req_coverage: ["REQ-217"]
depends_on: ["CR139-S13"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-217 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.6"
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

# CR139-S36 R4 列裁剪/谓词下推

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-217。Owner Feature: FEAT-02 读侧。Wave: CR139-W3-OPS。lld_policy: technical-note（a 已设计未实现，消费既有 HLD 契约；R4 是 a 类）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-217 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.6）
- `process/REQUIREMENTS.md` v1.17（REQ-217）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/readers.py`

### 接口约定与设计约束
- 文件所有权：readers.py 列裁剪切片（merge_order 第 10 位）
- P2；readers.py merge_order 末位
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
- 查询指定列/谓词时下推生效。

## acceptance_criteria

查询指定列/谓词时下推生效。

量化验收：
- 查询指定列/谓词时下推生效。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-217 (FEAT-02 读侧, a, technical-note); HLD-DATA-LAKE §17.6
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- 消费既有 HLD-DATA-LAKE 契约（a 类）

## lld_policy

- required_level: `technical-note`
- trigger_reasons: a 已设计未实现，消费既有 HLD 契约
- rationale: R4 是 a 类
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计依据

S36 覆盖 REQ-217：DuckDB readonly adapter 在查询指定列和谓词时必须把 projections 与 partition filters 传入只读 SQL 模板。S13 已提供 `read_dataset_via_duckdb_contract()` façade，本 Story 不新增依赖、不改事实源，只补 CR139 级 fixture 验证，证明 `projection_scan` 模板渲染出指定列和 `WHERE` 谓词。

### 2. 文件影响

| File | Change | Boundary |
|---|---|---|
| `tests/test_cr139_duckdb_pushdown_e2e.py` | Add S36 projection/predicate pushdown contract tests. | `tmp_path` only; no real lake. |
| `market_data/readers.py` | Consumed existing S13 façade; no additional code required unless tests expose a gap. | Reader-side contract only. |

### 3. Interface Contract

The S13 façade must preserve:

- requested `projections`;
- requested `partition_filters`;
- rendered read-only SQL with selected columns;
- rendered read-only SQL with deterministic equality predicates;
- zero operation counters and no `.duckdb` persistent files.

### 4. S37 Boundary

S36 validates pushdown contract shape. S37 validates the broader read-only e2e/audit side-effect chain.

### 5. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S36-PUSHDOWN-01 | Request `trade_date,symbol,close` with `trade_date=20260526`. | Rendered SQL contains selected columns and `WHERE trade_date = ...`. |
| T-S36-PUSHDOWN-02 | Forbidden SQL template with projection request. | Blocked before execution. |

### 6. Non-goals

- No DuckDB hard dependency, no performance benchmark, no real parquet scan, no catalog/manifest/pointer write, no provider/runtime/NAS/Git operation.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S36-column-pruning-predicate-pushdown.md#技术说明` | ready-for-review | Defines projection/predicate pushdown validation over S13 façade. |
| CP5 自动预检 | `process/checks/CP5-CR139-S36-column-pruning-predicate-pushdown-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | No blocker; fixture/static contract tests only. |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation summary | `process/stories/CR139-S36-S37-duckdb-pushdown-e2e-IMPLEMENTATION.md` | complete | Adds S36/S37 fixture/static tests over existing S13 façade. |
| Work packet | `process/context/stories/STORY-CR139-S36.CP6.work-packet.json` | complete | CP6 execution context. |
| Return packet | `process/returns/STORY-CR139-S36.CP6.return.json` | PASS | Boundary metadata and command evidence. |
| Evidence index | `process/evidence/STORY-CR139-S36.CP6.index.json` | PASS | References pytest evidence. |
| CP6 自动检查 | `process/checks/CP6-CR139-S36-column-pruning-predicate-pushdown-CODING-DONE.md` | PASS | Ready for CP7. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A10-S36-S37.md` | PASS | Covers S36/S37 traceability and boundary verification. |
| Work packet | `process/context/stories/STORY-CR139-S36.CP7.work-packet.json` | complete | CP7 verification context. |
| Return packet | `process/returns/STORY-CR139-S36.CP7.return.json` | verified | Stage decision PASS. |
| Evidence index | `process/evidence/STORY-CR139-S36.CP7.index.json` | PASS | Includes pytest, return-check, and diff-check evidence. |
| CP7 自动检查 | `process/checks/CP7-CR139-S36-column-pruning-predicate-pushdown-VERIFICATION-DONE.md` | PASS | Story moved to verified. |

## 依赖与文件所有权

- depends_on: CR139-S13
- dependency_type: ["contract"]
- 文件所有权: readers.py 列裁剪切片（merge_order 第 10 位）
- P2；readers.py merge_order 末位

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
