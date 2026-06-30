---
story_id: "CR139-S26"
title: "X4 PIT universe 成分链"
change_id: "CR-139"
owner_feature: "FEAT-14"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环"
  rationale: "X4 是 d2，既有 survivorship_bias_note 接通 PIT 成分链"
wave: "CR139-W2-CONSISTENCY"
status: "verified"
req_coverage: ["REQ-231"]
depends_on: ["CR139-S09", "CR139-S19"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-231 (FEAT-14, d2, technical-note)"
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

# CR139-S26 X4 PIT universe 成分链

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-231。Owner Feature: FEAT-14。Wave: CR139-W2-CONSISTENCY。lld_policy: technical-note（d2 既有合同闭环；X4 是 d2，既有 survivorship_bias_note 接通 PIT 成分链）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-231 (FEAT-14, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-231）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `engine/contracts.py`

### 接口约定与设计约束
- 文件所有权：contracts.py :270（merge_order 第 2 位，S09 V4 后）
- 与 S09 V4 file-conflict，V4 先；消费 S19 F3
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
- index_members 按时点构建 PIT universe 成分链，消除固定成分快照偏差。

## acceptance_criteria

index_members 按时点构建 PIT universe 成分链，消除固定成分快照偏差。

量化验收：
- index_members 按时点构建 PIT universe 成分链，消除固定成分快照偏差。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-231 (FEAT-14, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环
- rationale: X4 是 d2，既有 survivorship_bias_note 接通 PIT 成分链
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计依据

S26 覆盖 REQ-231：`index_members` 必须能按时点表达 PIT universe 成分链，避免固定当前成分快照被误用为生产 PIT。S09 schema freeze 已完成，S19 已提供 versioned `PortfolioRiskPolicy.universe_policy_ref`，本 Story 在 `engine/contracts.py` 中补齐消费侧合同对象与 fail-closed validation。

### 2. 文件影响

| File | Change | Boundary |
|---|---|---|
| `engine/contracts.py` | Add `PitUniverseConstituent`, `PitUniverseConstituentChain`, and `validate_pit_universe_constituent_chain()`. | Contract-only, no data read/write. |
| `tests/test_cr139_pit_universe_constituent_chain.py` | Add fixture/static tests for valid chain, fixed snapshot block, and future `available_at` block. | In-memory rows only. |

### 3. Interface Contract

`PitUniverseConstituent` fields:

- `symbol`
- `effective_from`
- `effective_to`
- `available_at`
- `source_run_id`
- `lineage_checksum`
- `membership_status`

`PitUniverseConstituentChain` fields:

- `universe_id`
- `as_of_date`
- `policy_version`
- `universe_policy_ref`
- `constituents`
- `is_pit_universe`
- `survivorship_bias_note`

Validation must fail when:

- required fields are missing;
- `is_pit_universe=false`;
- `survivorship_bias_note` indicates fixed/current snapshot;
- any constituent has `available_at > decision_time` when decision time is supplied.

### 4. S24 Boundary

S24 owns calendar/timezone normalization. S26 may consume normalized dates as strings but does not implement source calendar comparison or timezone parsing beyond timestamp validation needed for `available_at`.

### 5. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S26-PIT-01 | PIT chain with valid constituent fields and policy ref. | Validation passes. |
| T-S26-PIT-02 | Chain marked fixed/current snapshot. | Validation blocks with `pit_universe_fixed_snapshot_forbidden`. |
| T-S26-PIT-03 | `available_at > decision_time`. | Validation blocks with `pit_universe_available_after_decision_time`. |

### 6. Non-goals

- No provider fetch, no `index_members` lake read/write, no physical chain materialization.
- No S24 calendar/timezone implementation.
- No catalog/manifest/pointer/NAS/runtime/trading/Git operation.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S26-pit-universe-constituent-chain.md#技术说明` | ready-for-review | Defines PIT constituent chain contract and S24 boundary. |
| CP5 自动预检 | `process/checks/CP5-CR139-S26-pit-universe-constituent-chain-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | No blocker; fixture/static contract implementation only. |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation evidence | `process/stories/CR139-S24-S26-consistency-contracts-IMPLEMENTATION.md` | complete | Implements PIT constituent dataclasses and `validate_pit_universe_constituent_chain()`. |
| CP6 自动检查 | `process/checks/CP6-CR139-S26-pit-universe-constituent-chain-CODING-DONE.md` | PASS | Targeted fixture/static tests passed; no runtime or lake writes. |
| Return packet | `process/returns/STORY-CR139-S26.CP6.return.json` | PASS | Story/stage return metadata for CP6. |
| Evidence index | `process/evidence/STORY-CR139-S26.CP6.index.json` | PASS | CP6 evidence references and validation snapshot. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A7-S24-S26.md` | PASS | S24/S26 static-fixture verification report. |
| CP7 自动检查 | `process/checks/CP7-CR139-S26-pit-universe-constituent-chain-VERIFICATION-DONE.md` | PASS | Story can be marked `verified`. |
| Return packet | `process/returns/STORY-CR139-S26.CP7.return.json` | PASS | Story/stage return metadata for CP7. |
| Evidence index | `process/evidence/STORY-CR139-S26.CP7.index.json` | PASS | CP7 evidence references and validation snapshot. |

## 依赖与文件所有权

- depends_on: CR139-S09, CR139-S19
- dependency_type: ["contract"]
- 文件所有权: contracts.py :270（merge_order 第 2 位，S09 V4 后）
- 与 S09 V4 file-conflict，V4 先；消费 S19 F3

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
