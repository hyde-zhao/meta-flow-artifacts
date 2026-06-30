---
story_id: "CR139-S34"
title: "M2 lineage_checksum 回填"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "a 已设计未实现，消费既有 HLD 契约"
  rationale: "M2 是 a 类"
wave: "CR139-W2-CLEANUP"
status: "lld-approved"
req_coverage: ["REQ-209"]
depends_on: ["CR139-S33"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-209 (FEAT-02 写侧, a, technical-note); HLD-DATA-LAKE §17.7.1"
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

# CR139-S34 M2 lineage_checksum 回填

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-209。Owner Feature: FEAT-02 写侧。Wave: CR139-W2-CLEANUP。lld_policy: technical-note（a 已设计未实现，消费既有 HLD 契约；M2 是 a 类）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-209 (FEAT-02 写侧, a, technical-note); HLD-DATA-LAKE §17.7.1）
- `process/REQUIREMENTS.md` v1.17（REQ-209）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/manifest.py`
- `market_data/normalization.py`

### 接口约定与设计约束
- 文件所有权：manifest.py lineage_checksum（merge_order 第 2 位，S33 M1 后）；normalization.py lineage 回填切片（merge_order 第 4 位）
- 与 S33 M1 file-conflict，M1 先
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
- 写 canonical 完成时 lineage_checksum 非缺失（17/17 缺失→0 缺失）。

## acceptance_criteria

写 canonical 完成时 lineage_checksum 非缺失（17/17 缺失→0 缺失）。

量化验收：
- 写 canonical 完成时 lineage_checksum 非缺失（17/17 缺失→0 缺失）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-209 (FEAT-02 写侧, a, technical-note); HLD-DATA-LAKE §17.7.1
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- 消费既有 HLD-DATA-LAKE 契约（a 类）

## lld_policy

- required_level: `technical-note`
- trigger_reasons: a 已设计未实现，消费既有 HLD 契约
- rationale: M2 是 a 类
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S33
- dependency_type: ["contract"]
- 文件所有权: manifest.py lineage_checksum（merge_order 第 2 位，S33 M1 后）；normalization.py lineage 回填切片（merge_order 第 4 位）
- 与 S33 M1 file-conflict，M1 先

## 技术说明

### 设计依据

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 REQ-209：M2 lineage_checksum 回填为 technical-note，消费既有 HLD-DATA-LAKE 契约。
- S34 依赖 S33 的 catalog/manifest source-of-truth；先有 catalog 定主，再设计 checksum 回填。
- 本 Story 只设计 fixture/static 回填规则，不写真实 lake。

### 文件影响范围

| 文件 | 设计变更 | 边界 |
|---|---|---|
| `market_data/manifest.py` | 设计 lineage checksum 计算/校验字段，manifest 从 catalog 派生时携带 checksum。 | 不改变 provider catalog，不执行真实回填。 |
| `market_data/normalization.py` | 设计 canonical 写完成时生成 lineage source material 并传给 manifest builder。 | 位于 S31/S32 后、S14 前。 |
| `tests/test_cr139_lineage_checksum_backfill.py` | fixture 覆盖 17/17 缺失到 0 缺失的静态场景。 | 不扫描真实 lake。 |

### 接口 / 数据 / 权限变化

- `lineage_checksum` 使用稳定输入：dataset、run_id、source_uri/ref、schema_version、row_count 或 partition fingerprint。
- checksum 缺失时 fixture writer 必须生成；输入不足时阻断，不用随机值。
- manifest 校验需要区分 `missing_checksum`、`mismatch_checksum`、`unverifiable_source`。
- 权限无变化；不执行真实回填 job。

### 异常和回退

- 来源字段缺失：阻断并输出缺失字段。
- checksum mismatch：阻断，要求重新生成 fixture/source material。
- 真实历史 manifest 缺 checksum：只记录 remediation candidate，不自动写回。

### 测试入口

- `uv run --python 3.11 pytest -q tests/test_cr139_lineage_checksum_backfill.py`
- 覆盖：checksum 生成、缺字段阻断、mismatch 阻断、17/17 fixture 缺失归零。

### 已知风险、偏离记录与重访条件

- 风险：checksum 输入集合过宽导致不稳定。缓解：只选稳定 metadata，禁止把本地绝对路径纳入 checksum。
- 偏离：本 Story 不执行历史 manifest physical backfill。
- 重访条件：S33 catalog schema 变化、manifest 派生规则变化、真实 backfill 授权。

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
