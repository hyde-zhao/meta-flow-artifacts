---
story_id: "CR139-S22"
title: "T6 数据 run-id 贯通（跨边界）"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧/11/06"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "d2 既有合同闭环，跨 4 Feature"
  rationale: "T6 是 d2，既有 RunEvidenceIndex 合同接通 run-id 贯通"
wave: "CR139-W2-AUDIT-CHAIN"
status: "lld-approved"
req_coverage: ["REQ-242"]
depends_on: ["CR139-S04", "CR139-S14"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-242 (FEAT-02 写侧/11/06, d2, technical-note)"
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

# CR139-S22 T6 数据 run-id 贯通（跨边界）

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-242。Owner Feature: FEAT-02 写侧/11/06。Wave: CR139-W2-AUDIT-CHAIN。lld_policy: technical-note（d2 既有合同闭环，跨 4 Feature；T6 是 d2，既有 RunEvidenceIndex 合同接通 run-id 贯通）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。

## 跨边界项 owner/consumer

owner=FEAT-02 写侧（run-id 生成）；consumer=FEAT-11 RunEvidenceIndex + FEAT-06 broker event；规则=run-id 单向贯通，消费方只读，FD-51 禁止断链静默成功

owner 侧负责生成/定义契约，consumer 侧只读消费，不得反向修改（FD-46..53 禁止反向依赖）。


## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-242 (FEAT-02 写侧/11/06, d2, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-242）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/publish.py`
- `market_data/manifest.py`
- `trading/strategy_runner/evidence_index.py`

### 接口约定与设计约束
- 文件所有权：生成侧 publish.py/manifest.py（merge_order 第 3/2 位，S04/S14 后）；消费侧 evidence_index.py :19（独占）
- 跨边界 owner=FEAT-02 写侧；T4/L3 消费
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
- 数据→研究→执行同 run-id 贯穿 RunEvidenceIndex/broker event；断链标注断点，伪造贯通次数=0。

## acceptance_criteria

数据→研究→执行同 run-id 贯穿 RunEvidenceIndex/broker event；断链标注断点，伪造贯通次数=0。

量化验收：
- 数据→研究→执行同 run-id 贯穿 RunEvidenceIndex/broker event；断链标注断点，伪造贯通次数=0。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-242 (FEAT-02 写侧/11/06, d2, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: d2 既有合同闭环，跨 4 Feature
- rationale: T6 是 d2，既有 RunEvidenceIndex 合同接通 run-id 贯通
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S04, CR139-S14
- dependency_type: ["contract"]
- 文件所有权: 生成侧 publish.py/manifest.py（merge_order 第 3/2 位，S04/S14 后）；消费侧 evidence_index.py :19（独占）
- 跨边界 owner=FEAT-02 写侧；T4/L3 消费

## 技术说明

### 设计依据

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 REQ-242：T6 为既有 RunEvidenceIndex 合同接通 run-id 贯通 technical-note。
- owner=FEAT-02 写侧生成 run-id，FEAT-11 RunEvidenceIndex 和 FEAT-06 broker event 只读消费。
- S22 消费 S14 的 incremental/publish run-id 输出；断链不得静默成功。

### 文件影响范围

| 文件 | 设计变更 | 边界 |
|---|---|---|
| `market_data/publish.py` | 发布计划和 manifest 输出必须携带 data_run_id / source_run_id。 | 不执行 pointer advance。 |
| `market_data/manifest.py` | 设计 run lineage 字段向 evidence index 可消费格式映射。 | 不接管 S33/S34 source-of-truth/checksum。 |
| `trading/strategy_runner/evidence_index.py` | 只读消费 data run lineage，标注断链点。 | 不反向修改 lake 元数据。 |
| `tests/test_cr139_runid_lineage_penetration.py` | fixture 覆盖 data->research->execution run-id 贯通和断链阻断。 | 不启动交易 runtime。 |

### 接口 / 数据 / 权限变化

- 数据侧输出 `data_run_id`、`source_run_id`、`publish_run_id`、`manifest_ref`。
- RunEvidenceIndex 消费字段为只读引用，不生成或修正数据侧 run_id。
- broker event 如缺数据 run_id，只能标注 `lineage_status=broken` 和断点，不得伪造贯通。
- 权限无变化；不启动 QMT/gateway/trading runtime。

### 异常和回退

- run-id 缺失：RunEvidenceIndex 标记断链并 fail closed 当前贯通检查。
- run-id 格式不合规：回退到 S30 校验错误，不做消费端修补。
- S14 pointer plan 未完成：S22 只可基于 fixture manifest 设计，不进入实现。

### 测试入口

- `uv run --python 3.11 pytest -q tests/test_cr139_runid_lineage_penetration.py`
- 覆盖：完整贯通、缺数据 run-id、格式不合规、消费端只读约束、伪造贯通次数=0。

### 已知风险、偏离记录与重访条件

- 风险：FEAT-11/06 消费侧试图补写 run_id。缓解：明确 owner=FEAT-02 写侧，消费侧只读。
- 偏离：本 Story 不实现 broker lake 实盘写和交易事件真实接入。
- 重访条件：RunEvidenceIndex schema 变化、broker event schema 变化、真实 runtime evidence 接入授权。

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
