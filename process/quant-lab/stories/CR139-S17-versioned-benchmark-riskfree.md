---
story_id: "CR139-S17"
title: "F1 版本化 benchmark+risk-free curve"
change_id: "CR-139"
owner_feature: "FEAT-02"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环"
  rationale: "F1 是 d2，既有 BenchmarkCoverage/BenchmarkDefinition 合同接通版本化"
wave: "CR139-W2-CONFIG-FACTS"
status: "verified"
req_coverage: ["REQ-236"]
depends_on: ["CR139-S04"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-236 (FEAT-02, d2, technical-note)"
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

# CR139-S17 F1 版本化 benchmark+risk-free curve

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-236。Owner Feature: FEAT-02。Wave: CR139-W2-CONFIG-FACTS。lld_policy: technical-note（d2 既有合同闭环；F1 是 d2，既有 BenchmarkCoverage/BenchmarkDefinition 合同接通版本化）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-236 (FEAT-02, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-236）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/benchmarks.py`

### 接口约定与设计约束
- 文件所有权：benchmarks.py :99/114（独占）
- config_facts release 复用 V1 pointer 语义
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
- benchmark 版本化带 version + effective_from + release_id + risk-free curve；release 闭环复用 V1 pointer 语义（AGA-5 E1）。

## acceptance_criteria

benchmark 版本化带 version + effective_from + release_id + risk-free curve；release 闭环复用 V1 pointer 语义（AGA-5 E1）。

量化验收：
- benchmark 版本化带 version + effective_from + release_id + risk-free curve；release 闭环复用 V1 pointer 语义（AGA-5 E1）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-236 (FEAT-02, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环
- rationale: F1 是 d2，既有 BenchmarkCoverage/BenchmarkDefinition 合同接通版本化
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计依据

S17 覆盖 REQ-236，将现有 `BenchmarkDefinition` 扩展为版本化 config fact 合同。HLD AGA-5 已确认 E1：配置类事实源复用 V1 published pointer 语义，但本 Story 只实现内存/fixture 合同字段，不写 `config_facts/` 物理目录、不发布 pointer。

### 2. 文件影响

| File | Change |
|---|---|
| `market_data/benchmarks.py` | Add `version`, `effective_from`, `release_id`, `risk_free_curve_ref` to `BenchmarkDefinition`; keep `list_required_benchmarks()` backward compatible via defaults. |
| `tests/test_cr139_config_facts_versioning.py` | Add S17 assertions for versioned benchmark metadata and risk-free curve ref. |

### 3. Interface Contract

`BenchmarkDefinition.to_dict()` must include:

- `version`
- `effective_from`
- `release_id`
- `risk_free_curve_ref`

Existing callers constructing `BenchmarkDefinition` without these fields remain valid through default values.

### 4. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S17-CONFIG-01 | `list_required_benchmarks()` returns registry rows. | Every row includes non-empty `version`, `effective_from`, `release_id`, and `risk_free_curve_ref`. |
| T-S17-CONFIG-02 | Custom benchmark definition. | `to_dict()` preserves explicit version metadata. |

### 5. Non-goals

No real `config_facts/` write, no catalog/manifest/pointer update, no provider fetch, no runtime/trading/NAS/Git operation.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S17-versioned-benchmark-riskfree.md#技术说明` | ready-for-review | Defines versioned benchmark + risk-free curve contract. |
| CP5 自动预检 | `process/checks/CP5-CR139-S17-S18-S19-config-facts-versioning-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | Batch has independent file owners and fixture/static scope. |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation | `process/stories/CR139-S17-S18-S19-config-facts-versioning-IMPLEMENTATION.md` | implemented | Added versioned benchmark config fact fields. |
| CP6 自动检查 | `process/checks/CP6-CR139-S17-versioned-benchmark-riskfree-CODING-DONE.md` | PASS | Implementation complete; ready for CP7. |
| Return packet | `process/returns/STORY-CR139-S17.CP6.return.json` | PASS | Story return for CP6. |
| Evidence index | `process/evidence/STORY-CR139-S17.CP6.index.json` | PASS | Includes changed files and targeted pytest evidence. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A6-S17-S18-S19.md` | PASS | Verifies REQ-236 config fact metadata. |
| CP7 自动检查 | `process/checks/CP7-CR139-S17-versioned-benchmark-riskfree-VERIFICATION-DONE.md` | PASS | Story can be marked `verified`. |
| Return packet | `process/returns/STORY-CR139-S17.CP7.return.json` | PASS | Story return for CP7. |
| Evidence index | `process/evidence/STORY-CR139-S17.CP7.index.json` | PASS | Includes pytest evidence. |

## 依赖与文件所有权

- depends_on: CR139-S04
- dependency_type: ["contract"]
- 文件所有权: benchmarks.py :99/114（独占）
- config_facts release 复用 V1 pointer 语义

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
