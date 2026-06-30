---
story_id: "CR139-S21"
title: "T5 CommissionSchedule 前置成本门禁"
change_id: "CR-139"
owner_feature: "FEAT-12"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环"
  rationale: "T5 是 d2，既有 CommissionSchedule 接通成本前置门禁"
wave: "CR139-W2-AUDIT-CHAIN"
status: "verified"
req_coverage: ["REQ-241"]
depends_on: ["CR139-S18"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-241 (FEAT-12, d2, technical-note)"
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

# CR139-S21 T5 CommissionSchedule 前置成本门禁

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-241。Owner Feature: FEAT-12。Wave: CR139-W2-AUDIT-CHAIN。lld_policy: technical-note（d2 既有合同闭环；T5 是 d2，既有 CommissionSchedule 接通成本前置门禁）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-241 (FEAT-12, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-241）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `trading/qmt_gateway_contracts.py`

### 接口约定与设计约束
- 文件所有权：qmt_gateway_contracts.py :997（merge_order 第 2 位，S18 F2 后）
- 与 S18 F2 file-conflict，F2 先；消费 F2 版本化 commission
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
- 回测/实盘成本门禁前置，成本/滑点/成交可得性前置阻断。

## acceptance_criteria

回测/实盘成本门禁前置，成本/滑点/成交可得性前置阻断。

量化验收：
- 回测/实盘成本门禁前置，成本/滑点/成交可得性前置阻断。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-241 (FEAT-12, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环
- rationale: T5 是 d2，既有 CommissionSchedule 接通成本前置门禁
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计依据

S21 覆盖 REQ-241：回测/模拟/实盘前置阶段必须在本地下单前确认成本、滑点和成交可得性合同，缺失或不可用时 fail-closed。S18 已在 `CommissionSchedule` 中补齐 `version`、`effective_from`、`release_id`、`slippage_bps`、`stamp_duty_rate` 和 `transfer_fee_rate`，本 Story 在 `trading/qmt_gateway_contracts.py` 追加纯合同层 pretrade gate，不调用 QMT、broker adapter、账户查询或 runtime。

### 2. 文件影响

| File | Change | Boundary |
|---|---|---|
| `trading/qmt_gateway_contracts.py` | Add `CostPretradeGateInput`, `CostPretradeGateDecision`, and `evaluate_cost_pretrade_gate()`. | Contract-only, no QMT/runtime/account read. |
| `tests/test_cr139_commission_cost_pretrade_gate.py` | Add fixture tests for pass, missing versioned schedule, unavailable tradability, and side-effect counters. | In-memory only. |

### 3. Interface Contract

`CostPretradeGateInput` fields:

- `order_intent_ref`
- `notional`
- `side`
- `tradability_status`
- `schedule`
- `cost_model_ref`
- `liquidity_ref`

`CostPretradeGateDecision` must return:

- `status`: `allowed` or `blocked`;
- `blocked_reasons`: stable reason codes;
- `estimated_total_cost`;
- `schedule_version`, `schedule_release_id`, `cost_model_ref`, `liquidity_ref`;
- zero operation counters for QMT/runtime/broker/account/provider/lake/publish.

Validation must fail when:

- schedule is missing;
- S18 version metadata is missing;
- tradability is not available;
- cost/liquidity refs are missing;
- notional is not positive.

### 4. S40 Boundary

S40 remains out of this batch because its card records an explicit `factor_model_validation.py` merge order after S15. S21 consumes S18 only and does not modify factor model validation or policy-cycle/shortability contracts.

### 5. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S21-COST-01 | Versioned commission schedule, cost ref, liquidity ref, available tradability. | Gate allows and computes total estimated cost. |
| T-S21-COST-02 | Schedule lacks version/release metadata. | Gate blocks with `cost_schedule_version_missing`. |
| T-S21-COST-03 | Tradability unavailable. | Gate blocks with `tradability_unavailable`. |
| T-S21-COST-04 | Operation counters. | All real operation counters remain zero. |

### 6. Non-goals

- No QMT runtime, broker/account query, credential read, order submit, adapter call, or trading execution.
- No config_facts physical write, active catalog/manifest write, pointer advance, provider fetch, NAS, or Git remote write.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S21-commission-cost-pretrade-gate.md#技术说明` | ready-for-review | Defines local cost pretrade gate and S18 schedule dependency. |
| CP5 自动预检 | `process/checks/CP5-CR139-S21-commission-cost-pretrade-gate-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | No blocker; fixture/static contract implementation only. |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation evidence | `process/stories/CR139-S21-S35-gates-replay-IMPLEMENTATION.md` | complete | Implements local cost pretrade gate contract. |
| CP6 自动检查 | `process/checks/CP6-CR139-S21-commission-cost-pretrade-gate-CODING-DONE.md` | PASS | Targeted fixture/static tests passed; no runtime or lake writes. |
| Return packet | `process/returns/STORY-CR139-S21.CP6.return.json` | PASS | Story/stage return metadata for CP6. |
| Evidence index | `process/evidence/STORY-CR139-S21.CP6.index.json` | PASS | CP6 evidence references and validation snapshot. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A8-S21-S35.md` | PASS | S21/S35 static-fixture verification report. |
| CP7 自动检查 | `process/checks/CP7-CR139-S21-commission-cost-pretrade-gate-VERIFICATION-DONE.md` | PASS | Story can be marked `verified`. |
| Return packet | `process/returns/STORY-CR139-S21.CP7.return.json` | PASS | Story/stage return metadata for CP7. |
| Evidence index | `process/evidence/STORY-CR139-S21.CP7.index.json` | PASS | CP7 evidence references and validation snapshot. |

## 依赖与文件所有权

- depends_on: CR139-S18
- dependency_type: ["contract"]
- 文件所有权: qmt_gateway_contracts.py :997（merge_order 第 2 位，S18 F2 后）
- 与 S18 F2 file-conflict，F2 先；消费 F2 版本化 commission

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
