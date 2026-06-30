---
story_id: "CR139-S19"
title: "F3 版本化 universe/risk policy"
change_id: "CR-139"
owner_feature: "FEAT-14"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环"
  rationale: "F3 是 d2，既有 PortfolioRiskPolicy 合同接通版本化"
wave: "CR139-W2-CONFIG-FACTS"
status: "verified"
req_coverage: ["REQ-238"]
depends_on: ["CR139-S04"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-238 (FEAT-14, d2, technical-note)"
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

# CR139-S19 F3 版本化 universe/risk policy

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-238。Owner Feature: FEAT-14。Wave: CR139-W2-CONFIG-FACTS。lld_policy: technical-note（d2 既有合同闭环；F3 是 d2，既有 PortfolioRiskPolicy 合同接通版本化）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-238 (FEAT-14, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-238）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `engine/mature_multifactor_framework.py`

### 接口约定与设计约束
- 文件所有权：mature_multifactor_framework.py :228（独占）
- X4 PIT universe 消费 F3
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
- PortfolioRiskPolicy 版本化，universe/risk policy 带 version + release 闭环 + 退市/ST/容量约束。

## acceptance_criteria

PortfolioRiskPolicy 版本化，universe/risk policy 带 version + release 闭环 + 退市/ST/容量约束。

量化验收：
- PortfolioRiskPolicy 版本化，universe/risk policy 带 version + release 闭环 + 退市/ST/容量约束。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-238 (FEAT-14, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环
- rationale: F3 是 d2，既有 PortfolioRiskPolicy 合同接通版本化
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计依据

S19 覆盖 REQ-238，将现有 `PortfolioRiskPolicy` 扩展为版本化 universe/risk policy config fact 合同，并显式携带退市/ST/容量约束字段。S19 为后续 S26 PIT universe constituent chain 和 S40 policy-cycle/shortability 提供版本化 policy ref，但本 Story 不实现二者。

### 2. 文件影响

| File | Change |
|---|---|
| `engine/mature_multifactor_framework.py` | Add `effective_from`, `release_id`, `universe_policy_ref`, `delisting_policy`, `st_policy` to `PortfolioRiskPolicy`; include required validation for version metadata. |
| `tests/test_cr139_config_facts_versioning.py` | Add S19 assertions for serialized and validated risk policy. |

### 3. Interface Contract

`PortfolioRiskPolicy.to_dict()` must include `version`, `effective_from`, `release_id`, and universe/risk fields. `build_stage2_portfolio_risk_policy()` remains backward compatible through defaults. `validate_portfolio_risk_policy()` must fail closed when required version fields are absent from a mapping payload.

### 4. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S19-CONFIG-01 | Builder-created risk policy. | Version metadata and delisting/ST/capacity policy fields exist. |
| T-S19-CONFIG-02 | Mapping missing version metadata. | Validation blocks with risk policy invalid reason. |

### 5. Non-goals

No PIT universe chain build, no policy-cycle/shortability implementation, no real lake/config write, no catalog/manifest/pointer/NAS/provider/runtime/Git operation.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S19-versioned-universe-risk-policy.md#技术说明` | ready-for-review | Defines versioned PortfolioRiskPolicy and universe/risk constraints. |
| CP5 自动预检 | `process/checks/CP5-CR139-S17-S18-S19-config-facts-versioning-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | Batch has independent file owners and fixture/static scope. |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation | `process/stories/CR139-S17-S18-S19-config-facts-versioning-IMPLEMENTATION.md` | implemented | Added versioned universe/risk policy fields and validation. |
| CP6 自动检查 | `process/checks/CP6-CR139-S19-versioned-universe-risk-policy-CODING-DONE.md` | PASS | Implementation complete; ready for CP7. |
| Return packet | `process/returns/STORY-CR139-S19.CP6.return.json` | PASS | Story return for CP6. |
| Evidence index | `process/evidence/STORY-CR139-S19.CP6.index.json` | PASS | Includes changed files and targeted pytest evidence. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A6-S17-S18-S19.md` | PASS | Verifies REQ-238 config fact metadata and validation. |
| CP7 自动检查 | `process/checks/CP7-CR139-S19-versioned-universe-risk-policy-VERIFICATION-DONE.md` | PASS | Story can be marked `verified`. |
| Return packet | `process/returns/STORY-CR139-S19.CP7.return.json` | PASS | Story return for CP7. |
| Evidence index | `process/evidence/STORY-CR139-S19.CP7.index.json` | PASS | Includes pytest evidence. |

## 依赖与文件所有权

- depends_on: CR139-S04
- dependency_type: ["contract"]
- 文件所有权: mature_multifactor_framework.py :228（独占）
- X4 PIT universe 消费 F3

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
