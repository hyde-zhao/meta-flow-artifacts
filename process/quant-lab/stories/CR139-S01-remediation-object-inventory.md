---
story_id: "CR139-S01"
title: "T8 整改对象自动化清册"
change_id: "CR-139"
owner_feature: "FEAT-02"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: "d1 纯新建对象，无既有合同"
  rationale: "T8 是 d1 遗漏分析纯新建，必须 full-lld"
wave: "CR139-W1-BASELINE-FREEZE"
status: "lld-ready-for-review"
req_coverage: ["REQ-213"]
depends_on: []
dependency_type: []
feature_design_refs:
  - "MATRIX v1.13 REQ-213 (FEAT-02, d1, full-lld)"
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

# CR139-S01 T8 整改对象自动化清册

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-213。Owner Feature: FEAT-02。Wave: CR139-W1-BASELINE-FREEZE。lld_policy: full-lld（d1 纯新建对象，无既有合同；T8 是 d1 遗漏分析纯新建，必须 full-lld）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-213 (FEAT-02, d1, full-lld)）
- `process/REQUIREMENTS.md` v1.17（REQ-213）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `scripts/lake_inventory.py`
- `market_data/remediation_inventory.py`

### 接口约定与设计约束
- 文件所有权：新清册脚本（独占）
- Wave1 基线门第 1 步，无前置
- 不授权范围：CP4 不授权 LLD 写作/实现/runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移执行/Git remote write。CP5 通过后才实现。
- 平台目标：日频离线研究 + 模拟盘场景；DuckDB 只读引擎（D4）；parquet 仍是存储。
- 命名规范：遵循 CR-139 §5 Wave 路线 + REQ-247 基线门顺序。

### AI 可执行任务清单（CP5 LLD 阶段）
1. 消费 HLD/ADR/MATRIX 契约，产出 full LLD（full-lld）或 Story 内 ## 技术说明（technical-note）。
2. 核验文件所有权切片与 merge_order，避免 readers.py/publish.py/catalog.py/normalization.py 冲突。
3. 标注跨边界 owner/consumer（如适用），消费方只读。
4. 验收标准量化、可验证。

## validation_context

### 验证入口
- 静态/fixture 验证（CP6/CP7 默认 static-only，runtime 需独立授权）。
- 验证命令：`uv run --python 3.11 pytest -q tests/test_cr139_*.py`（按 Story 测试文件）。

### 验证方式
- full-lld Story：full LLD 14 章 + fixture 测试 + 契约映射。
- technical-note Story：Story 内 ## 技术说明 + 既有合同接通闭环证据。

### 依赖环境
- 不读取 .env/凭据；不启动 runtime；不连接 QMT/NAS；不写真实 lake。
- DuckDB 依赖引入需 CP5 批准（D4，仅 S13 相关）。

### 关键验证场景
- 执行清册命令扫描全 dataset，输出行数/覆盖/重复键/pit_status/published/lineage 缺失情况；17→N dataset 全覆盖。

## acceptance_criteria

执行清册命令扫描全 dataset，输出行数/覆盖/重复键/pit_status/published/lineage 缺失情况；17→N dataset 全覆盖。

量化验收：
- 执行清册命令扫描全 dataset，输出行数/覆盖/重复键/pit_status/published/lineage 缺失情况；17→N dataset 全覆盖。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-213 (FEAT-02, d1, full-lld)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `full-lld`
- trigger_reasons: d1 纯新建对象，无既有合同
- rationale: T8 是 d1 遗漏分析纯新建，必须 full-lld
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: 无
- dependency_type: []
- 文件所有权: 新清册脚本（独占）
- Wave1 基线门第 1 步，无前置

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
