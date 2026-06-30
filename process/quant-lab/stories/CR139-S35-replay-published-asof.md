---
story_id: "CR139-S35"
title: "L5 replay 接通 published as_of 重放"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环"
  rationale: "L5 是 d2，既有 replay 合同接通 published as_of 重放"
wave: "CR139-W3-OPS"
status: "verified"
req_coverage: ["REQ-245"]
depends_on: ["CR139-S04", "CR139-S14"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-245 (FEAT-02 读侧, d2, technical-note)"
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

# CR139-S35 L5 replay 接通 published as_of 重放

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-245。Owner Feature: FEAT-02 读侧。Wave: CR139-W3-OPS。lld_policy: technical-note（d2 既有合同闭环；L5 是 d2，既有 replay 合同接通 published as_of 重放）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-245 (FEAT-02 读侧, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-245）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/replay.py`
- `market_data/cli.py`

### 接口约定与设计约束
- 文件所有权：replay.py :215（独占）；cli.py cmd_p0_replay（merge_order 第 4 位）
- P2；cli.py merge_order 第 4 位
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
- published as_of replay 单日快照重放，不触发 provider。

## acceptance_criteria

published as_of replay 单日快照重放，不触发 provider。

量化验收：
- published as_of replay 单日快照重放，不触发 provider。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-245 (FEAT-02 读侧, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环
- rationale: L5 是 d2，既有 replay 合同接通 published as_of 重放
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 技术说明

### 1. 设计依据

S35 覆盖 REQ-245：replay 必须能够基于已 published 的 as_of 单日快照重放，且不得在缺失快照时触发 provider backfill。S04 已提供 published pointer read selector 语义，S14 已提供 append/pointer advance 合同，本 Story 只在 `market_data/replay.py` 和 `market_data/cli.py` 增加只读合同与 CLI 入口，不读取真实 lake root、不写 catalog/manifest/pointer。

### 2. 文件影响

| File | Change | Boundary |
|---|---|---|
| `market_data/replay.py` | Add published as_of replay request/result contracts and selector. | In-memory pointer records only. |
| `market_data/cli.py` | Add a fixture-friendly CLI command wrapper for published as_of replay. | Requires explicit pointer payload; no lake-root scan. |
| `tests/test_cr139_replay_published_asof.py` | Add fixture tests for exact published snapshot, unpublished block, missing as_of block, and no provider fallback. | In-memory only. |

### 3. Interface Contract

`PublishedAsOfReplayRequest` fields:

- `dataset`
- `as_of_trade_date`
- `run_id`
- `batch_id`

`PublishedAsOfReplayResult` must return:

- `status`: `ready_for_replay` or `blocked`;
- `published_path` and `manifest_ref` only when an exact published as_of snapshot exists;
- `replay_source=published_asof_snapshot`;
- operation counters for provider fetch, credential read, lake write, catalog/manifest write, pointer advance, and runtime all zero.

Validation must fail when:

- no pointer matches requested dataset/as_of;
- matching pointer is not published;
- published path or manifest ref is missing;
- caller supplies only a candidate/unpublished pointer.

### 4. S13/S36/S37 Boundary

This Story does not introduce DuckDB or predicate pushdown. S13/S36/S37 remain separate read-engine tasks and are not required for in-memory published as_of replay contract validation.

### 5. Test Plan

| Test ID | Scenario | Expected |
|---|---|---|
| T-S35-REPLAY-01 | Exact published pointer for dataset/as_of. | Result is `ready_for_replay`, attempts/network/writes all zero. |
| T-S35-REPLAY-02 | Matching pointer is candidate/unpublished. | Result blocks with `published_asof_not_published`. |
| T-S35-REPLAY-03 | No matching as_of pointer. | Result blocks with `published_asof_snapshot_missing`. |
| T-S35-REPLAY-04 | CLI wrapper with explicit pointer JSON. | Returns same contract without lake scan or provider fallback. |

### 6. Non-goals

- No provider fetch, no real lake read/list, no catalog/manifest/pointer write, no physical replay execution, no NAS, no runtime/trading, no Git remote write.

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Technical note | `process/stories/CR139-S35-replay-published-asof.md#技术说明` | ready-for-review | Defines published as_of replay selector and CLI wrapper boundary. |
| CP5 自动预检 | `process/checks/CP5-CR139-S35-replay-published-asof-TECHNICAL-NOTE-IMPLEMENTABILITY.md` | PASS | No blocker; fixture/static replay contract implementation only. |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation evidence | `process/stories/CR139-S21-S35-gates-replay-IMPLEMENTATION.md` | complete | Implements published as_of replay selector and explicit-pointer CLI wrapper. |
| CP6 自动检查 | `process/checks/CP6-CR139-S35-replay-published-asof-CODING-DONE.md` | PASS | Targeted fixture/static tests passed; no provider, runtime, or lake writes. |
| Return packet | `process/returns/STORY-CR139-S35.CP6.return.json` | PASS | Story/stage return metadata for CP6. |
| Evidence index | `process/evidence/STORY-CR139-S35.CP6.index.json` | PASS | CP6 evidence references and validation snapshot. |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A8-S21-S35.md` | PASS | S21/S35 static-fixture verification report. |
| CP7 自动检查 | `process/checks/CP7-CR139-S35-replay-published-asof-VERIFICATION-DONE.md` | PASS | Story can be marked `verified`. |
| Return packet | `process/returns/STORY-CR139-S35.CP7.return.json` | PASS | Story/stage return metadata for CP7. |
| Evidence index | `process/evidence/STORY-CR139-S35.CP7.index.json` | PASS | CP7 evidence references and validation snapshot. |

## 依赖与文件所有权

- depends_on: CR139-S04, CR139-S14
- dependency_type: ["contract"]
- 文件所有权: replay.py :215（独占）；cli.py cmd_p0_replay（merge_order 第 4 位）
- P2；cli.py merge_order 第 4 位

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
