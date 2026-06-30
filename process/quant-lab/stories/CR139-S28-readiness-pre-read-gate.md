---
story_id: "CR139-S28"
title: "L4 readiness 读前强制门禁"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧/14"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环"
  rationale: "L4 是 d2，既有 build_readiness_matrix 前置为强制 gate"
wave: "CR139-W2-READ-GATE"
status: "verified"
req_coverage: ["REQ-234"]
depends_on: ["CR139-S05"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-234 (FEAT-02 读侧/14, d2, technical-note)"
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

# CR139-S28 L4 readiness 读前强制门禁

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-234。Owner Feature: FEAT-02 读侧/14。Wave: CR139-W2-READ-GATE。lld_policy: technical-note（d2 既有合同闭环；L4 是 d2，既有 build_readiness_matrix 前置为强制 gate）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。

## 跨边界项 owner/consumer

owner=FEAT-02 读侧（门禁）；consumer=FEAT-14 模拟盘

owner 侧负责生成/定义契约，consumer 侧只读消费，不得反向修改（FD-46..53 禁止反向依赖）。


## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-234 (FEAT-02 读侧/14, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-234）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/readiness.py`
- `market_data/readers.py`

### 接口约定与设计约束
- 文件所有权：readiness.py :462（独占）；readers.py gate 调用切片（merge_order 第 7 位）
- 跨边界 owner=FEAT-02 读侧；FEAT-14 模拟盘消费
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
- 读前 readiness gate 不通过时阻断（coverage/新鲜度/PIT）。

## acceptance_criteria

读前 readiness gate 不通过时阻断（coverage/新鲜度/PIT）。

量化验收：
- 读前 readiness gate 不通过时阻断（coverage/新鲜度/PIT）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-234 (FEAT-02 读侧/14, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环
- rationale: L4 是 d2，既有 build_readiness_matrix 前置为强制 gate
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计目标

S28 将现有 `build_readiness_matrix` / reader readiness issue 合同收敛为读前强制门禁：当目标 dataset 的 coverage、freshness、published/readiness/PIT 前置条件不满足时，reader 在实际读取前返回阻断型 `ReaderResult`，并给出可行动 remediation。

本 Story 不实现 S25 的 decision_time/lookahead 语义。S25 继续独立拥有 signal/decision time 与 `available_at` 的违规判定；S28 只预留可插拔 hook，避免后续 S25 返工。

### 2. 现有基础

| Existing object | Current behavior | S28 design use |
|---|---|---|
| `market_data/readiness.py::build_readiness_matrix` | Builds P0 readiness matrix and gap codes from catalog/manifest/candidate/audit inputs. | Pre-read gate input contract. |
| `market_data/readers.py::_readiness_issues` | Blocks after frame read when readiness/pit fields in the frame are not available. | Retain as post-read row-level safety net. |
| `read_dataset` catalog/published/quality checks | Already block missing catalog, unpublished pointer, quality failure/warn. | S28 should not duplicate; compose before physical parquet read where readiness matrix is supplied. |

### 3. Proposed CP6 Slice

| Action | File | Detail |
|---|---|---|
| Add pure gate helper | `market_data/readiness.py` | `evaluate_pre_read_readiness_gate(dataset, matrix_or_rows, *, require_pit=True)` returns pass/block payload with gap codes and remediation; no IO. |
| Add reader integration hook | `market_data/readers.py` | Optional `readiness_gate` / `pre_read_readiness` parameter or internal helper call before `_entry_paths()` read. Default behavior remains compatible when no gate input is supplied. |
| Add tests | `tests/test_cr139_readiness_pre_read_gate.py` | Fixture matrix pass and fail cases; fail blocks before fixture reader is invoked. |

### 4. S25-vs-S28 Boundary

| Boundary | Owner | S28 rule |
|---|---|---|
| Dataset readiness: coverage, freshness, quality, published/readiness/PIT status | S28 | Implement/readiness gate can block. |
| Signal/decision time lookahead: `decision_time` vs row `available_at` | S25 | Do not implement in S28; expose hook name or extension slot only. |
| Shared reader hook | FEAT-02 read side | S28 must not consume S25 under a hidden name. Later S25 can plug into the hook without changing S28 failure codes. |

### 5. Failure Contract

| Condition | Status | Issue code |
|---|---|---|
| Readiness matrix missing for required dataset | `required_missing` | `pre_read_readiness_missing` |
| Matrix row has `gap_codes` | `unavailable` or `required_missing` when required | `pre_read_readiness_blocked` plus gap codes |
| PIT required but row has `pit_incomplete` | `unavailable` / `required_missing` | `pit_incomplete` |
| Gate passes | continue reader path | no issue |

### 6. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S28-GATE-01 | Passing readiness row. | Reader continues and returns available fixture result. |
| T-S28-GATE-02 | Row with `GAP_READINESS_NOT_AVAILABLE`. | Reader returns blocked result before physical/frame read. |
| T-S28-GATE-03 | PIT required and row has `pit_incomplete`. | Reader blocks with PIT issue. |
| T-S28-S25-BOUNDARY-01 | CP5/static scan. | S28 tests and helper do not implement decision_time/lookahead failure semantics. |

### 7. Non-goals

- No real lake scan or write.
- No active catalog/manifest write.
- No pointer advance.
- No NAS/provider/runtime/trading/credential/Git operation.
- No S25 decision_time/lookahead implementation.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S28-readiness-pre-read-gate.md#技术说明` | ready-for-review | 定义 readiness pre-read gate、failure contract、S25 boundary。 |
| CP5 自动预检 | `process/checks/CP5-CR139-S28-readiness-pre-read-gate-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | 无阻断项；S25 边界保留。 |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation | `process/stories/CR139-S28-readiness-pre-read-gate-IMPLEMENTATION.md` | implemented | 新增 pre-read readiness gate helper 与 read_dataset 可选集成参数。 |
| CP6 自动检查 | `process/checks/CP6-CR139-S28-readiness-pre-read-gate-CODING-DONE.md` | PASS | targeted pytest 18 passed；S25 lookahead 语义未混入。 |
| Return packet | `process/returns/STORY-CR139-S28.CP6.return.json` | implemented | touched files 与边界检查已记录。 |
| Evidence index | `process/evidence/STORY-CR139-S28.CP6.index.json` | PASS | CP6 机器证据索引。 |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A3-S27-S28.md` | PASS | S27/S28 fixture/static 联合验证报告。 |
| CP7 自动检查 | `process/checks/CP7-CR139-S28-readiness-pre-read-gate-VERIFICATION-DONE.md` | PASS | S28 REQ-234 验证通过，S25 边界保留。 |
| Return packet | `process/returns/STORY-CR139-S28.CP7.return.json` | verified | CP7 return packet。 |
| Evidence index | `process/evidence/STORY-CR139-S28.CP7.index.json` | PASS | CP7 机器证据索引。 |

## 依赖与文件所有权

- depends_on: CR139-S05
- dependency_type: ["contract"]
- 文件所有权: readiness.py :462（独占）；readers.py gate 调用切片（merge_order 第 7 位）
- 跨边界 owner=FEAT-02 读侧；FEAT-14 模拟盘消费

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
