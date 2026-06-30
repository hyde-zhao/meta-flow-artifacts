---
story_id: "CR139-S31"
title: "C3 events schema 修复"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "b 设计过期"
  rationale: "C3 是 b 类 schema 修复"
wave: "CR139-W2-CLEANUP"
status: "lld-approved"
req_coverage: ["REQ-206"]
depends_on: ["CR139-S01", "CR139-S02", "CR139-S03"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-206 (FEAT-02 写侧, b, technical-note)"
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

# CR139-S31 C3 events schema 修复

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-206。Owner Feature: FEAT-02 写侧。Wave: CR139-W2-CLEANUP。lld_policy: technical-note（b 设计过期；C3 是 b 类 schema 修复）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-206 (FEAT-02 写侧, b, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-206）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/normalization.py`

### 接口约定与设计约束
- 文件所有权：normalization.py events 分支切片（merge_order 第 2 位，S08 N1 后）
- 基线门后结构性变更
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
- events 数据集 schema 类型非全 null，字段类型正确。

## acceptance_criteria

events 数据集 schema 类型非全 null，字段类型正确。

量化验收：
- events 数据集 schema 类型非全 null，字段类型正确。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-206 (FEAT-02 写侧, b, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: b 设计过期
- rationale: C3 是 b 类 schema 修复
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S01, CR139-S02, CR139-S03
- dependency_type: ["contract"]
- 文件所有权: normalization.py events 分支切片（merge_order 第 2 位，S08 N1 后）
- 基线门后结构性变更

## 技术说明

### 设计依据

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 REQ-206：C3 events schema 修复为 technical-note。
- S01/S02/S03 已建立整改对象和黄金值基线；S31 只设计 events 写侧 schema 修复，不真实重跑 lake。
- `normalization.py` merge order：S08 -> S31 -> S32 -> S34 -> S14。

### 文件影响范围

| 文件 | 设计变更 | 边界 |
|---|---|---|
| `market_data/normalization.py` | 在 events dataset 分支定义字段类型 canonical map，禁止全 null schema 推断。 | 只覆盖 events 分支，不改其他 dataset schema。 |
| `tests/test_cr139_events_schema_repair.py` | 使用小型 fixture 覆盖 events 字段类型、全 null 输入阻断和缺字段错误。 | 不读取真实 lake。 |

### 接口 / 数据 / 权限变化

- events schema 至少明确事件时间、事件类型、标的代码、来源、run_id/available_at 等字段的类型和必填性。
- 当输入样本全 null 或无法推断类型时，normalization 必须使用 explicit schema 或阻断，不得写出全 null typed parquet。
- 权限无变化；无 runtime、无真实 lake 写入、无 provider catalog 写入。

### 异常和回退

- `events` 必填字段缺失：阻断并给出字段名。
- 字段全 null 但 explicit schema 可用：按 schema cast 后继续 fixture 写入。
- 字段全 null 且无 explicit schema：阻断，避免产生不可读 schema。
- 发现存量 events schema 异常时只报告，不执行真实修复。

### 测试入口

- `uv run --python 3.11 pytest -q tests/test_cr139_events_schema_repair.py`
- 覆盖：正常 events、全 null with schema、全 null without schema、类型不匹配阻断。

### 已知风险、偏离记录与重访条件

- 风险：events 来源字段差异可能超出当前 fixture。缓解：新增来源必须先补 schema map 和 fixture。
- 偏离：不重跑历史 events 数据，不修复真实 parquet。
- 重访条件：新增 events dataset 来源、字段类型变化、或真实回填授权。

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
