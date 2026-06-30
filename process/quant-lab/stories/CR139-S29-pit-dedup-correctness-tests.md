---
story_id: "CR139-S29"
title: "T2+T3 PIT/去重正确性回归测试"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: "c 范围扩展 T2/T3"
  rationale: "T2/T3 是 c 类，构造回归测试"
wave: "CR139-W2-CORRECTNESS"
status: "verified"
req_coverage: ["REQ-226", "REQ-227"]
depends_on: ["CR139-S05", "CR139-S07"]
dependency_type: ["runtime"]
feature_design_refs:
  - "MATRIX v1.13 REQ-226/227 (FEAT-02 读侧, c, full-lld)"
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

# CR139-S29 T2+T3 PIT/去重正确性回归测试

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-226, REQ-227。Owner Feature: FEAT-02 读侧。Wave: CR139-W2-CORRECTNESS。lld_policy: full-lld（c 范围扩展 T2/T3；T2/T3 是 c 类，构造回归测试）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-226/227 (FEAT-02 读侧, c, full-lld)）
- `process/REQUIREMENTS.md` v1.17（REQ-226, REQ-227）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `tests/test_cr139_pit_correctness.py`
- `tests/test_cr139_dedup_correctness.py`

### 接口约定与设计约束
- 文件所有权：新测试文件（独占）
- 聚合 T2+T3
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
- 未来财报 fixture as-of read 读不到（T2）；重复键 fixture 去重 read 后唯一（T3）。

## acceptance_criteria

未来财报 fixture as-of read 读不到（T2）；重复键 fixture 去重 read 后唯一（T3）。

量化验收：
- 未来财报 fixture as-of read 读不到（T2）；重复键 fixture 去重 read 后唯一（T3）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-226/227 (FEAT-02 读侧, c, full-lld)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `full-lld`
- trigger_reasons: c 范围扩展 T2/T3
- rationale: T2/T3 是 c 类，构造回归测试
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Full LLD | `process/stories/CR139-S29-pit-dedup-correctness-tests-LLD.md` | ready-for-review | 固化 PIT/dedup correctness standard，供 S23 引用。 |
| CP5 自动预检 | `process/checks/CP5-CR139-S29-pit-dedup-correctness-tests-LLD-IMPLEMENTABILITY.md` | PASS | 无阻断项；不授权 lake/catalog/manifest/pointer/NAS/provider/runtime/Git。 |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation | `process/stories/CR139-S29-pit-dedup-correctness-tests-IMPLEMENTATION.md` | implemented | 新增 PIT/dedup correctness fixture/static 测试。 |
| CP6 自动检查 | `process/checks/CP6-CR139-S29-pit-dedup-correctness-tests-CODING-DONE.md` | PASS | targeted pytest 15 passed；无禁止操作。 |
| Return packet | `process/returns/STORY-CR139-S29.CP6.return.json` | implemented | touched files 与边界检查已记录。 |
| Evidence index | `process/evidence/STORY-CR139-S29.CP6.index.json` | PASS | CP6 机器证据索引。 |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A1-A2-S29-S23.md` | PASS | S29/S23 fixture/static 联合验证报告。 |
| CP7 自动检查 | `process/checks/CP7-CR139-S29-pit-dedup-correctness-tests-VERIFICATION-DONE.md` | PASS | S29 REQ-226/227 验证通过。 |
| Return packet | `process/returns/STORY-CR139-S29.CP7.return.json` | verified | CP7 return packet。 |
| Evidence index | `process/evidence/STORY-CR139-S29.CP7.index.json` | PASS | CP7 机器证据索引。 |

## 依赖与文件所有权

- depends_on: CR139-S05, CR139-S07
- dependency_type: ["runtime"]
- 文件所有权: 新测试文件（独占）
- 聚合 T2+T3

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
