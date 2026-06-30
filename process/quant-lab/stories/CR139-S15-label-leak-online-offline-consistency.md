---
story_id: "CR139-S15"
title: "E3+E4 label 泄漏+离线在线一致性"
change_id: "CR-139"
owner_feature: "FEAT-03"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: "d1 纯新建 E4 + d2 既有合同 E3，取最高"
  rationale: "聚合 Story 取 full-lld（含 E4 d1 full-lld）"
wave: "CR139-W2-ML-SIMULATION"
status: "verified"
req_coverage: ["REQ-223", "REQ-224"]
depends_on: ["CR139-S11", "CR139-S12"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-223/224 (FEAT-03, d2/d1)"
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

# CR139-S15 E3+E4 label 泄漏+离线在线一致性

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-223, REQ-224。Owner Feature: FEAT-03。Wave: CR139-W2-ML-SIMULATION。lld_policy: full-lld（d1 纯新建 E4 + d2 既有合同 E3，取最高；聚合 Story 取 full-lld（含 E4 d1 full-lld））。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-223/224 (FEAT-03, d2/d1)）
- `process/REQUIREMENTS.md` v1.17（REQ-223, REQ-224）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `engine/factor_model_validation.py`
- `engine/factor_robustness.py`

### 接口约定与设计约束
- 文件所有权：factor_model_validation.py label 泄漏 :561/376（merge_order 首位）；factor_robustness.py :53；新一致性校验模块
- 聚合 E3+E4
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
- label 泄漏检查统一接入 data release + cutoff gate（E3）；离线/在线特征同 schema 同计算，不一致阻断（E4）。

## acceptance_criteria

label 泄漏检查统一接入 data release + cutoff gate（E3）；离线/在线特征同 schema 同计算，不一致阻断（E4）。

量化验收：
- label 泄漏检查统一接入 data release + cutoff gate（E3）；离线/在线特征同 schema 同计算，不一致阻断（E4）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-223/224 (FEAT-03, d2/d1)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `full-lld`
- trigger_reasons: d1 纯新建 E4 + d2 既有合同 E3，取最高
- rationale: 聚合 Story 取 full-lld（含 E4 d1 full-lld）
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S11, CR139-S12
- dependency_type: ["contract"]
- 文件所有权: factor_model_validation.py label 泄漏 :561/376（merge_order 首位）；factor_robustness.py :53；新一致性校验模块
- 聚合 E3+E4

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。

## CP5 full-lld 设计证据

- LLD：`process/stories/CR139-S15-label-leak-online-offline-consistency-LLD.md`
- 自动预检：`process/checks/CP5-CR139-S15-label-leak-online-offline-consistency-LLD-IMPLEMENTABILITY.md`
- 结论：PASS；仅授权本地代码、fixture/static 测试和过程证据，不授权真实 lake/catalog/NAS/provider/runtime/Git remote 写入。

## CP6/CP7 实现与验证证据

- Implementation：`process/stories/CR139-S15-S16-label-training-cutoff-IMPLEMENTATION.md`
- CP6 return：`process/returns/STORY-CR139-S15.CP6.return.json`
- CP6 evidence：`process/evidence/STORY-CR139-S15.CP6.index.json`
- CP7 report：`docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A13-S15-S16.md`
- CP7 return：`process/returns/STORY-CR139-S15.CP7.return.json`
- CP7 evidence：`process/evidence/STORY-CR139-S15.CP7.index.json`
- 结论：`verified`；label/cutoff gate 与 offline/online feature consistency 合同已通过 fixture/static 验证。
