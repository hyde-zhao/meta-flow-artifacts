---
story_id: "CR139-S38"
title: "M3 quality/ 分区整理"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "b 设计过期"
  rationale: "M3 是 b 类分区整理"
wave: "CR139-W3-OPS"
status: "verified"
req_coverage: ["REQ-210"]
depends_on: []
dependency_type: []
feature_design_refs:
  - "MATRIX v1.13 REQ-210 (FEAT-02 写侧, b, technical-note)"
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

# CR139-S38 M3 quality/ 分区整理

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-210。Owner Feature: FEAT-02 写侧。Wave: CR139-W3-OPS。lld_policy: technical-note（b 设计过期；M3 是 b 类分区整理）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-210 (FEAT-02 写侧, b, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-210）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/quality_writer.py`

### 接口约定与设计约束
- 文件所有权：质量报告写入模块（独占）
- P2；无强依赖
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
- quality/ 按数据集/日期分区，smoke/probe 隔离到 _scratch/。

## acceptance_criteria

quality/ 按数据集/日期分区，smoke/probe 隔离到 _scratch/。

量化验收：
- quality/ 按数据集/日期分区，smoke/probe 隔离到 _scratch/。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-210 (FEAT-02 写侧, b, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: b 设计过期
- rationale: M3 是 b 类分区整理
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计目标

S38 为 quality report 写入提供新的分区整理合同：

- 正式 quality report 路径必须按 `quality/<dataset>/<as_of_date>/...` 分区；
- smoke/probe/test 运行的输出必须隔离到 `quality/_scratch/<run_id>/...`；
- 新模块为 `market_data/quality_writer.py`，不替代现有 `market_data.validation.write_quality_reports()`，避免影响既有 CLI/validation 行为。

### 2. 现有基础

| Existing object | Current behavior | S38 design use |
|---|---|---|
| `LakeLayout.quality_root` | Resolves `<lake_root>/quality`. | Root path source. |
| `market_data.validation.write_quality_reports()` | Writes legacy shape `quality/<run_id>/<dataset>_quality.*`. | Keep unchanged; S38 adds new writer contract. |
| W3 publish guard / quality paths evidence | Confirms quality path existence in current truth. | Evidence input only; not sufficient to close S38 writer contract. |

### 3. Proposed CP6 Slice

| Action | File | Detail |
|---|---|---|
| Create writer contract | `market_data/quality_writer.py` | Add `QualityWriteRequest`, `QualityWriteResult`, `quality_partition_path()`, `write_partitioned_quality_report()`. |
| Add tests | `tests/test_cr139_quality_partition_writer.py` | Validate official dataset/date partition and `_scratch` isolation for smoke/probe. |
| Export optional API | no package-level import required | Tests can import direct module to keep blast radius low. |

### 4. Data And Path Contract

| Mode | Path pattern | Notes |
|---|---|---|
| `official` | `quality/<dataset>/<as_of_date>/<run_id>-quality.json` | `as_of_date` normalized to `YYYY-MM-DD`; dataset/run_id sanitized for path segment safety. |
| `smoke` / `probe` | `quality/_scratch/<run_id>/<dataset>-<as_of_date>-quality.json` | Never writes under official dataset/date partition. |

The writer accepts JSON-serializable payloads only. It returns path metadata and operation counters. It does not update catalog, manifest, pointer, or provider state.

### 5. Failure Contract

| Condition | Error |
|---|---|
| Missing dataset/run_id/as_of_date | `ValueError("quality_write_required_field_missing: ...")` |
| Unsupported mode | `ValueError("unsupported_quality_write_mode: ...")` |
| Unsafe path segment (`/`, `..`, empty) | `ValueError("unsafe_quality_path_segment: ...")` |
| Non JSON-serializable payload | `TypeError` from JSON serialization |

### 6. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S38-PART-01 | Official quality write. | Path is under `quality/<dataset>/<as_of_date>/`; file exists; result marks `scratch=false`. |
| T-S38-SCRATCH-01 | Smoke/probe write. | Path is under `quality/_scratch/<run_id>/`; no official dataset/date file created. |
| T-S38-SAFE-01 | Unsafe path segment. | Writer fails before write. |

### 7. Non-goals

- No migration of existing quality files.
- No real lake quality write outside pytest `tmp_path` fixtures.
- No catalog/manifest/pointer/NAS/provider/runtime/Git operation.
- No change to existing `write_quality_reports()` callers.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S38-quality-partition-reorg.md#技术说明` | ready-for-review | Defines partition contract and `_scratch` isolation. |
| CP5 自动预检 | `process/checks/CP5-CR139-S38-quality-partition-reorg-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | 无阻断项；不授权真实 lake/catalog/manifest/pointer 操作。 |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation | `process/stories/CR139-S38-quality-partition-reorg-IMPLEMENTATION.md` | implemented | Added `market_data/quality_writer.py` and fixture/static tests. |
| CP6 自动检查 | `process/checks/CP6-CR139-S38-quality-partition-reorg-CODING-DONE.md` | PASS | Implementation complete; ready for CP7. |
| Return packet | `process/returns/STORY-CR139-S38.CP6.return.json` | PASS | Story return for CP6. |
| Evidence index | `process/evidence/STORY-CR139-S38.CP6.index.json` | PASS | Includes changed files and targeted pytest evidence. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A4-S38.md` | PASS | Verifies REQ-210 and legacy validation writer compatibility. |
| CP7 自动检查 | `process/checks/CP7-CR139-S38-quality-partition-reorg-VERIFICATION-DONE.md` | PASS | Story can be marked `verified`. |
| Return packet | `process/returns/STORY-CR139-S38.CP7.return.json` | PASS | Story return for CP7. |
| Evidence index | `process/evidence/STORY-CR139-S38.CP7.index.json` | PASS | Includes pytest, CP6 return-check, and diff-check evidence. |

## 依赖与文件所有权

- depends_on: 无
- dependency_type: []
- 文件所有权: 质量报告写入模块（独占）
- P2；无强依赖

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
