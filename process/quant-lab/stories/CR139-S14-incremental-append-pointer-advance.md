---
story_id: "CR139-S14"
title: "L1+L2 日级增量+pointer 前移"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环 L1/L2"
  rationale: "L1/L2 是 d2，既有 incremental/publish 合同接通闭环"
wave: "CR139-W2-ML-SIMULATION"
status: "lld-approved"
req_coverage: ["REQ-243", "REQ-244"]
depends_on: ["CR139-S04", "CR139-S08"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-243/244 (FEAT-02 写侧, d2, technical-note)"
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

# CR139-S14 L1+L2 日级增量+pointer 前移

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-243, REQ-244。Owner Feature: FEAT-02 写侧。Wave: CR139-W2-ML-SIMULATION。lld_policy: technical-note（d2 既有合同闭环 L1/L2；L1/L2 是 d2，既有 incremental/publish 合同接通闭环）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-243/244 (FEAT-02 写侧, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-243, REQ-244）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/incremental.py`
- `market_data/publish.py`
- `market_data/normalization.py`
- `market_data/cli.py`

### 接口约定与设计约束
- 文件所有权：incremental.py（独占）；publish.py pointer 前移（merge_order 第 2 位，S04 V1 后）；normalization.py 增量盖戳（merge_order 第 5 位）；cli.py 增量命令（merge_order 第 3 位）
- 聚合 L1+L2；published pointer 前移执行不授权
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
- 日级增量 append 幂等+当日去重+available_at 盖戳（L1）；每日 promote 后 current pointer 前移+门禁通过（L2）。

## acceptance_criteria

日级增量 append 幂等+当日去重+available_at 盖戳（L1）；每日 promote 后 current pointer 前移+门禁通过（L2）。

量化验收：
- 日级增量 append 幂等+当日去重+available_at 盖戳（L1）；每日 promote 后 current pointer 前移+门禁通过（L2）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-243/244 (FEAT-02 写侧, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环 L1/L2
- rationale: L1/L2 是 d2，既有 incremental/publish 合同接通闭环
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S04, CR139-S08
- dependency_type: ["contract"]
- 文件所有权: incremental.py（独占）；publish.py pointer 前移（merge_order 第 2 位，S04 V1 后）；normalization.py 增量盖戳（merge_order 第 5 位）；cli.py 增量命令（merge_order 第 3 位）
- 聚合 L1+L2；published pointer 前移执行不授权

## 技术说明

### 设计依据

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 REQ-243/244：L1/L2 为既有 incremental/publish 合同闭环 technical-note。
- S14 消费 S04 published pointer/read selector 和 S08 run_id partition key 治理；建议在 S31/S32/S34 后冻结 normalization 切片。
- 本 Story 只设计日级增量 append 与 pointer plan，不执行 published pointer advance。

### 文件影响范围

| 文件 | 设计变更 | 边界 |
|---|---|---|
| `market_data/incremental.py` | 设计日级 append plan、idempotency key、当日去重和 available_at 盖戳。 | 不写真实 lake。 |
| `market_data/publish.py` | 设计 pointer advance plan 和 precheck result；执行入口保持未授权。 | 不移动 current pointer。 |
| `market_data/normalization.py` | 消费 S30/S31/S32/S34 结果，设计增量写入 metadata。 | 不覆盖前置 Story 规则。 |
| `market_data/cli.py` | 设计 plan-only / fixture-only CLI 入口，默认禁止 execute。 | 不新增真实执行默认入口。 |
| `tests/test_cr139_incremental_append_pointer_plan.py` | fixture 覆盖幂等、去重、available_at、pointer plan。 | static-only。 |

### 接口 / 数据 / 权限变化

- 新增 `IncrementalAppendPlan` 语义：dataset、trade_date、run_id、input_fingerprint、expected_partitions、idempotency_key、available_at。
- 新增 `PointerAdvancePlan` 语义：from_pointer、to_pointer、precheck_status、evidence_refs、execute_allowed=false。
- CLI 默认只产出 plan/fixture evidence；必须显式 runtime_authorization 才能执行 pointer advance，且本批不提供该授权。
- 权限无变化；不读凭据、不启动 runtime、不写真实 lake。

### 异常和回退

- 同一 idempotency key 重复提交：返回 existing plan，不重复 append。
- 当日重复 canonical key：按 S32 规则去重或阻断。
- pointer precheck 不通过：只输出 blocked plan，不推进 pointer。
- 用户要求真实前移：停止当前流程，另建 runtime_authorization 决策项。

### 测试入口

- `uv run --python 3.11 pytest -q tests/test_cr139_incremental_append_pointer_plan.py`
- 覆盖：幂等 append、当日去重、available_at 必填、pointer plan blocked、execute forbidden。

### 已知风险、偏离记录与重访条件

- 风险：plan-only 被误当成真实发布。缓解：`execute_allowed=false` 写入计划、测试和 CP5 Brief。
- 偏离：不执行真实 pointer advance，不写真实 lake。
- 重访条件：用户另行授权 scoped real operation、S33/S34 manifest 合同变化、S22 run-id lineage 字段变化。

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
