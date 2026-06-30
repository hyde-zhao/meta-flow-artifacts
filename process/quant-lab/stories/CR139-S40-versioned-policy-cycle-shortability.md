---
story_id: "CR139-S40"
title: "F4 版本化政策周期/shortability"
change_id: "CR-139"
owner_feature: "FEAT-03"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环"
  rationale: "F4 是 d2，既有 policy_cycle_coverage 接通版本化"
wave: "CR139-W3-OPS"
status: "verified"
req_coverage: ["REQ-239"]
depends_on: ["CR139-S04", "CR139-S19"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-239 (FEAT-03, d2, technical-note)"
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

# CR139-S40 F4 版本化政策周期/shortability

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-239。Owner Feature: FEAT-03。Wave: CR139-W3-OPS。lld_policy: technical-note（d2 既有合同闭环；F4 是 d2，既有 policy_cycle_coverage 接通版本化）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-239 (FEAT-03, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-239）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `engine/factor_model_validation.py`
- `config/policy_cycles.yaml`

### 接口约定与设计约束
- 文件所有权：factor_model_validation.py :444（merge_order 第 2 位，S15 E3 后）；policy_cycles.yaml（独占）
- P2；与 S15 E3 file-conflict，E3 先；消费 S19 F3 参照
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
- 政策周期版本化带 version + release 闭环 + shortability。

## acceptance_criteria

政策周期版本化带 version + release 闭环 + shortability。

量化验收：
- 政策周期版本化带 version + release 闭环 + shortability。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-239 (FEAT-03, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环
- rationale: F4 是 d2，既有 policy_cycle_coverage 接通版本化
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S04, CR139-S19
- dependency_type: ["contract"]
- 文件所有权: factor_model_validation.py :444（merge_order 第 2 位，S15 E3 后）；policy_cycles.yaml（独占）
- P2；与 S15 E3 file-conflict，E3 先；消费 S19 F3 参照

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。

## 技术说明

### 范围

S40 闭环既有 `policy_cycle_coverage()` 与 `short_feasibility()` 合同，补齐 policy cycle 配置的 `version` / `release_id` 元数据，并增加一个 fixture/static 组合 gate，证明策略研究在声明 long-short/market-neutral 时同时具备版本化政策周期覆盖和 shortability 证据。

### 实现切片

- `config/policy_cycles.yaml`：增加顶层 `version`、`release_id`，并为每个 cycle 补齐 `version`、`release_id`。
- `engine/factor_model_validation.py`：`load_policy_cycle_config()` 保留向后兼容并填充 release 元数据；新增 `policy_cycle_shortability_gate()` 组合检查。
- `tests/test_cr139_versioned_policy_cycle_shortability.py`：验证版本化 release、shortability pass/block、零 operation counters。

### 验收

- policy cycle config 每行带 `version` + `release_id`。
- `policy_cycle_shortability_gate()` 在 policy coverage pass 且 long-short shortability pass 时返回 `pass`。
- 缺 shortable universe 或 policy cycle release 元数据时返回 `blocked`。
- 不读写真实 lake/catalog/manifest/pointer/NAS/provider/runtime/Git。

### CP5 结论

技术说明可实现；仅授权本地代码、fixture/static 测试和过程证据。

## CP6/CP7 实现与验证证据

- Implementation：`process/stories/CR139-S40-versioned-policy-cycle-shortability-IMPLEMENTATION.md`
- CP6 return：`process/returns/STORY-CR139-S40.CP6.return.json`
- CP6 evidence：`process/evidence/STORY-CR139-S40.CP6.index.json`
- CP7 report：`docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A14-S40.md`
- CP7 return：`process/returns/STORY-CR139-S40.CP7.return.json`
- CP7 evidence：`process/evidence/STORY-CR139-S40.CP7.index.json`
- 结论：`verified`；versioned policy cycle release 与 shortability gate 合同已通过 fixture/static 验证。
