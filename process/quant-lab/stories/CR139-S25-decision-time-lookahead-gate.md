---
story_id: "CR139-S25"
title: "X3 decision_time lookahead 阻断"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环"
  rationale: "X3 是 d2，既有 decision_time 部分支持加强制阻断"
wave: "CR139-W2-CONSISTENCY"
status: "verified"
req_coverage: ["REQ-235"]
depends_on: ["CR139-S05"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-235 (FEAT-02 读侧, d2, technical-note)"
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

# CR139-S25 X3 decision_time lookahead 阻断

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-235。Owner Feature: FEAT-02 读侧。Wave: CR139-W2-CONSISTENCY。lld_policy: technical-note（d2 既有合同闭环；X3 是 d2，既有 decision_time 部分支持加强制阻断）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-235 (FEAT-02 读侧, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-235）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/readers.py`

### 接口约定与设计约束
- 文件所有权：readers.py decision_time 切片 :227/1790（merge_order 第 5 位）
- readers.py merge_order 第 5 位
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
- 信号时刻 vs available_at lookahead 违规时阻断。

## acceptance_criteria

信号时刻 vs available_at lookahead 违规时阻断。

量化验收：
- 信号时刻 vs available_at lookahead 违规时阻断。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-235 (FEAT-02 读侧, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环
- rationale: X3 是 d2，既有 decision_time 部分支持加强制阻断
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计依据

S25 覆盖 REQ-235：当调用方给出用于信号、过滤、股票池或成交判断的 `decision_time` 时，reader 必须证明参与决策的数据满足 `available_at <= decision_time`。若任一参与行缺少 `available_at`、时间不可解析，或存在 `available_at > decision_time`，reader fail-closed 返回阻断型 `ReaderResult`。

S28 已完成 `pre_read_readiness_gate`，只覆盖 coverage/freshness/published/readiness/PIT 前置状态。S25 不复用或修改 S28 的 failure codes；S25 只新增 decision-time lookahead gate。

### 2. 文件影响

| File | Change | Owner boundary |
|---|---|---|
| `market_data/readers.py` | Add optional `decision_time` keyword to `read_dataset`; add pure helper for available-at lookahead validation. | S25 owns readers.py decision_time slice; S28 readiness hook remains unchanged. |
| `tests/test_cr139_decision_time_lookahead_gate.py` | Add fixture/static tests for pass, future `available_at`, missing `available_at`, and invalid `decision_time`. | Test-only fixture writes under `tmp_path`. |

No new dependency, no catalog/manifest/pointer write, no lake mutation beyond pytest fixture files.

### 3. Interface Contract

`read_dataset(..., decision_time: str | None = None)` remains backward compatible:

- `decision_time is None`: behavior is unchanged.
- `decision_time` is supplied and the read frame has zero rows after ordinary filters: no lookahead issue is emitted.
- `decision_time` is supplied and `available_at` is missing: return `status=required_missing` when `required=True`, else `unavailable`, with issue code `decision_time_available_at_missing`.
- `decision_time` is unparsable or any `available_at` is unparsable: return blocked `ReaderResult` with parse-specific issue code.
- Any row with `available_at > decision_time`: return blocked `ReaderResult` with issue code `decision_time_lookahead_blocked`, including `future_row_count`, `decision_time`, and `max_available_at`.

### 4. Failure And Remediation

| Condition | Status | Issue code | Remediation action |
|---|---|---|---|
| Missing `available_at` while `decision_time` supplied | `required_missing` / `unavailable` | `decision_time_available_at_missing` | `provide_explicit_available_at` |
| Unparsable `decision_time` | `required_missing` / `unavailable` | `decision_time_unparseable` | `fix_decision_time` |
| Unparsable `available_at` | `required_missing` / `unavailable` | `decision_time_available_at_unparseable` | `fix_available_at_timestamps` |
| `available_at > decision_time` | `required_missing` / `unavailable` | `decision_time_lookahead_blocked` | `use_data_available_at_or_before_decision_time` |

### 5. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S25-LOOKAHEAD-01 | All rows have `available_at <= decision_time`. | Reader returns `available`; frame is returned. |
| T-S25-LOOKAHEAD-02 | One row has `available_at > decision_time`. | Reader blocks with `decision_time_lookahead_blocked`. |
| T-S25-LOOKAHEAD-03 | `decision_time` supplied but `available_at` missing. | Reader blocks with `decision_time_available_at_missing`. |
| T-S25-LOOKAHEAD-04 | `decision_time` unparsable. | Reader blocks with `decision_time_unparseable`. |

### 6. Risks And Non-goals

- This Story does not infer decision time from `trade_date`; callers must pass it explicitly.
- This Story does not change S28 readiness gate, PIT as-of reader filtering, or post-read readiness checks.
- This Story does not start runtime/trading/QMT, read credentials, connect providers, write real lake, write catalog/manifest, advance pointer, operate NAS, or write Git remote.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S25-decision-time-lookahead-gate.md#技术说明` | ready-for-review | Defines explicit `decision_time` gate and S25/S28 boundary. |
| CP5 自动预检 | `process/checks/CP5-CR139-S25-decision-time-lookahead-gate-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | No blocker; fixture/static implementation only. |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation | `process/stories/CR139-S25-decision-time-lookahead-gate-IMPLEMENTATION.md` | implemented | Added explicit `decision_time` gate in `read_dataset`. |
| CP6 自动检查 | `process/checks/CP6-CR139-S25-decision-time-lookahead-gate-CODING-DONE.md` | PASS | Implementation complete; ready for CP7. |
| Return packet | `process/returns/STORY-CR139-S25.CP6.return.json` | PASS | Story return for CP6. |
| Evidence index | `process/evidence/STORY-CR139-S25.CP6.index.json` | PASS | Includes changed files and targeted pytest evidence. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A5-S25.md` | PASS | Verifies REQ-235, S28 boundary, and read path regressions. |
| CP7 自动检查 | `process/checks/CP7-CR139-S25-decision-time-lookahead-gate-VERIFICATION-DONE.md` | PASS | Story can be marked `verified`. |
| Return packet | `process/returns/STORY-CR139-S25.CP7.return.json` | PASS | Story return for CP7. |
| Evidence index | `process/evidence/STORY-CR139-S25.CP7.index.json` | PASS | Includes pytest, CP6 return-check, and diff-check evidence. |

## 依赖与文件所有权

- depends_on: CR139-S05
- dependency_type: ["contract"]
- 文件所有权: readers.py decision_time 切片 :227/1790（merge_order 第 5 位）
- readers.py merge_order 第 5 位

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
