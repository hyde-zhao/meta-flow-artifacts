---
story_id: "CR139-S27"
title: "L3 读审计 log+run-id 贯通"
change_id: "CR-139"
owner_feature: "FEAT-02 读侧/11"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: "d1 纯新建对象"
  rationale: "L3 是 d1 纯新建读审计 log"
wave: "CR139-W2-READ-GATE"
status: "verified"
req_coverage: ["REQ-233"]
depends_on: ["CR139-S05", "CR139-S22"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-233 (FEAT-02 读侧/11, d1, full-lld)"
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

# CR139-S27 L3 读审计 log+run-id 贯通

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-233。Owner Feature: FEAT-02 读侧/11。Wave: CR139-W2-READ-GATE。lld_policy: full-lld（d1 纯新建对象；L3 是 d1 纯新建读审计 log）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。

## 跨边界项 owner/consumer

owner=FEAT-02 读侧（读审计 log）；consumer=FEAT-11 RunEvidenceIndex（run-id 关联）

owner 侧负责生成/定义契约，consumer 侧只读消费，不得反向修改（FD-46..53 禁止反向依赖）。


## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-233 (FEAT-02 读侧/11, d1, full-lld)）
- `process/REQUIREMENTS.md` v1.17（REQ-233）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/readers.py`
- `market_data/read_audit.py`

### 接口约定与设计约束
- 文件所有权：readers.py 读审计 hook 切片（merge_order 第 6 位）；新 read_audit.py 模块（独占）
- 跨边界 owner=FEAT-02 读侧；消费 T6 run-id
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
- reader 读完成时读审计 log 与 RunEvidenceIndex 同 run-id 贯通。

## acceptance_criteria

reader 读完成时读审计 log 与 RunEvidenceIndex 同 run-id 贯通。

量化验收：
- reader 读完成时读审计 log 与 RunEvidenceIndex 同 run-id 贯通。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-233 (FEAT-02 读侧/11, d1, full-lld)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `full-lld`
- trigger_reasons: d1 纯新建对象
- rationale: L3 是 d1 纯新建读审计 log
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## CP5 设计证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Full LLD | `process/stories/CR139-S27-read-audit-log-runid-LLD.md` | ready-for-review | 定义 read audit record、reader hook、RunEvidenceIndex 消费边界。 |
| CP5 自动预检 | `process/checks/CP5-CR139-S27-read-audit-log-runid-LLD-IMPLEMENTABILITY.md` | PASS | 无阻断项；不授权 runtime/write。 |

## CP6 实现证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Implementation | `process/stories/CR139-S27-read-audit-log-runid-IMPLEMENTATION.md` | implemented | 新增 read audit 记录 helper 和 reader adapter。 |
| CP6 自动检查 | `process/checks/CP6-CR139-S27-read-audit-log-runid-CODING-DONE.md` | PASS | targeted pytest 18 passed；无禁止操作。 |
| Return packet | `process/returns/STORY-CR139-S27.CP6.return.json` | implemented | touched files 与边界检查已记录。 |
| Evidence index | `process/evidence/STORY-CR139-S27.CP6.index.json` | PASS | CP6 机器证据索引。 |

## CP7 验证证据

| 证据 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Verification report | `docs/quality/VERIFICATION-REPORT-CR139-BACKLOG-A3-S27-S28.md` | PASS | S27/S28 fixture/static 联合验证报告。 |
| CP7 自动检查 | `process/checks/CP7-CR139-S27-read-audit-log-runid-VERIFICATION-DONE.md` | PASS | S27 REQ-233 验证通过。 |
| Return packet | `process/returns/STORY-CR139-S27.CP7.return.json` | verified | CP7 return packet。 |
| Evidence index | `process/evidence/STORY-CR139-S27.CP7.index.json` | PASS | CP7 机器证据索引。 |

## 依赖与文件所有权

- depends_on: CR139-S05, CR139-S22
- dependency_type: ["contract"]
- 文件所有权: readers.py 读审计 hook 切片（merge_order 第 6 位）；新 read_audit.py 模块（独占）
- 跨边界 owner=FEAT-02 读侧；消费 T6 run-id

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
