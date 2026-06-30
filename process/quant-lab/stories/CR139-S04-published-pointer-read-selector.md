---
story_id: "CR139-S04"
title: "V1 published pointer / read selector"
change_id: "CR-139"
owner_feature: "FEAT-02 写/读"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "a 已设计未实现，消费既有 HLD 契约"
  rationale: "V1 是 a 类，HLD 已有契约，LLD 消费即可"
wave: "CR139-W1-STRUCTURAL-P0"
status: "lld-ready-for-review"
req_coverage: ["REQ-232"]
depends_on: ["CR139-S01", "CR139-S02", "CR139-S03"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-232 (FEAT-02 写/读, a, technical-note); HLD-DATA-LAKE §5/§17.4"
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

# CR139-S04 V1 published pointer / read selector

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-232。Owner Feature: FEAT-02 写/读。Wave: CR139-W1-STRUCTURAL-P0。lld_policy: technical-note（a 已设计未实现，消费既有 HLD 契约；V1 是 a 类，HLD 已有契约，LLD 消费即可）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-232 (FEAT-02 写/读, a, technical-note); HLD-DATA-LAKE §5/§17.4）
- `process/REQUIREMENTS.md` v1.17（REQ-232）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/publish.py`
- `market_data/catalog.py`
- `market_data/cli.py`

### 接口约定与设计约束
- 文件所有权：publish.py pointer 固定（merge_order 首位）；catalog.py current pointer；cli.py promote 命令
- 四道 P0 防线之一；基线门硬依赖 REQ-247；published pointer 前移执行不授权
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
- candidate/gold/published promote 后 catalog current pointer 固定；reader 只读 published（未发布行=0）；validate pass 不自动前移 pointer（SM-27 非法转换阻断）。

## acceptance_criteria

candidate/gold/published promote 后 catalog current pointer 固定；reader 只读 published（未发布行=0）；validate pass 不自动前移 pointer（SM-27 非法转换阻断）。

量化验收：
- candidate/gold/published promote 后 catalog current pointer 固定；reader 只读 published（未发布行=0）；validate pass 不自动前移 pointer（SM-27 非法转换阻断）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-232 (FEAT-02 写/读, a, technical-note); HLD-DATA-LAKE §5/§17.4
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- 消费既有 HLD-DATA-LAKE 契约（a 类）

## lld_policy

- required_level: `technical-note`
- trigger_reasons: a 已设计未实现，消费既有 HLD 契约
- rationale: V1 是 a 类，HLD 已有契约，LLD 消费即可
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S01, CR139-S02, CR139-S03
- dependency_type: ["contract"]
- 文件所有权: publish.py pointer 固定（merge_order 首位）；catalog.py current pointer；cli.py promote 命令
- 四道 P0 防线之一；基线门硬依赖 REQ-247；published pointer 前移执行不授权

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。

## 技术说明

> 本技术说明是 CR-139 CP5 Wave1 LLD 批次的设计证据。S04 是 **a 类（已设计未实现）**，消费既有 HLD-DATA-LAKE 契约，不重复设计（REQ-249）。既有合同位置已 grep 核验（memory `verify-existing-code-before-claiming-absent`）。本说明不含实现；CP5 全量人工确认 + Wave1 基线门（REQ-247）满足后才允许实现。

### 设计依据

| 来源 | 章节 / ID | 消费内容 |
|---|---|---|
| HLD-DATA-LAKE | §5 Quality / Readiness / Publish | publish gate 语义、catalog current truth 字段、`quality_status=fail` 阻断 publish |
| HLD-DATA-LAKE | §17.4 推荐方案总览与系统架构图 | Explicit Publish Gate → catalog current pointer 单一入口；validate pass 不自动更新 current pointer |
| HLD-DATA-LAKE | §17.7.1 写入时序与读写边界 | publish 阶段写入 catalog current pointer（唯一使 reader 默认可见的新版本入口）；validate/quality 只生成 candidate，不更新 pointer |
| companion HLD | `HLD-STRATEGY-DATA-FOUNDATION.md` v0.2 §5/§12.1/§12.2 | 写侧 publish 闭环（V1/L2/M1/M2）；读侧四道 P0 防线之"冻结快照" |
| ADR | `ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2 | D7 四组门禁；Wave1 基线门 REQ-247；AGA-5 E1（config_facts 复用 V1 pointer 语义） |
| handoff | `DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3.4 V1 / §1.6 既有合同 | 既有 `publish.py:605 publish_current_pointer`（dry-run only）；candidate/gold/published 三层全空、17/17 published=false |
| REQ | REQ-232 / REQ-247 / REQ-248 / REQ-249 | V1 published pointer / read selector；基线门；不授权；既有合同闭环 |

### 文件影响

| 文件 | 切片 | merge_order 位 | 所有权动作 | 既有合同位置（已核验） |
|---|---|---|---|---|
| `market_data/publish.py` | pointer 固定 / promote 闭环 | publish_py merge_order **首位**（S04→S14→S22） | 接通 `publish_current_pointer` 既有 dry-run 合同到 publish gate 闭环；**不前移执行**（dev_gate.published_pointer_advance_execution_allowed=false） | `:605 def publish_current_pointer`（已 grep 核验，dry-run only） |
| `market_data/catalog.py` | current pointer 固定 + published 标记 | catalog_py merge_order **首位**（S04→S33→S39） | catalog entry `published`/`published_at`/current pointer 字段固化；validate pass 不自动前移（SM-27） | `:56 published: bool`、`:57 published_at`、`:248 _missing_pointer_value`、`:256 _catalog_pointer_payload`（已 grep 核验） |
| `market_data/cli.py` | promote 命令 | cli_py merge_order **第 2 位**（S08→S04→S14→S35） | promote 命令接通 publish gate（显式动作，非自动） | 既有 CLI 生命周期（HLD-DATA-LAKE §3.3 `publish` 子命令） |

> readers.py **不在** S04 文件所有权范围（readers.py merge_order S05→S06→S07→...，S04 不在列）。S04 的 published selector 语义通过 `read_dataset` 既有 published 门禁（`readers.py:2778 if not entry.published`，已核验；handoff 称 :2796 实为 :2778）被读侧消费，S04 不修改 readers.py。

### 接口 / 数据变化

- **接口**：`publish_current_pointer`（`publish.py:605`）既有 dry-run 合同接通为 publish gate 闭环：candidate → gold → published promote 后，catalog current pointer 固定指向已发布 release。reader 只消费 published current truth。
- **数据**：catalog entry `published`（`:56`）从 17/17 `false` 状态，在显式 promote 后固化为 `true` + `published_at` + current pointer 指向。**不自动前移**：validate pass ≠ published current truth（SM-27 非法转换阻断，HLD Gotchas#1）。
- **跨边界**：S04 是 published pointer **生成侧**（owner=FEAT-02 写侧），消费方 = S05 C1 PIT reader（read published）、S06 R1 panel reader（复用 published 门禁）、S17-S19 F1-F3 config_facts（复用 V1 pointer 语义，AGA-5 E1）、S22 T6 run-id（publish 时生成 run-id）。消费方只读，不反向修改 pointer。

### 失败路径

| 场景 | 前置校验 | 失败行为 | 回退 |
|---|---|---|---|
| validate pass 自动前移 pointer | SM-27 非法转换阻断 | 阻断，current pointer 保持旧值 | 显式 promote 才前移 |
| `quality_status=fail` | publish gate 前置 | 阻断 publish，current pointer 不变 | 回退 validate / 重新 plan |
| promote 输入不完整（缺 lineage checksum / known_limitations） | publish gate 前置 | current pointer 保持旧值，candidate 标记 `candidate_unpublished` | 回退 validate |
| published pointer 前移执行（未授权） | dev_gate.published_pointer_advance_execution_allowed=false | 阻断 | 等独立 runtime_authorization（REQ-248） |

### 测试入口

- 静态 / fixture 验证（CP6/CP7 默认 static-only，runtime 需独立授权）。
- 验证命令：`uv run --python 3.11 pytest -q tests/test_cr139_published_pointer.py`（按 Story 测试文件）。
- 关键用例：candidate/gold/published promote 后 catalog current pointer 固定；reader 只读 published（未发布行=0）；validate pass 不自动前移 pointer（SM-27 阻断）。
- 既有合同接通闭环证据：`publish_current_pointer`（`:605`）从 dry-run 接通到 publish gate；catalog `published`/`current_pointer` 字段固化。

### 风险

| 风险 | 级别 | 应对 |
|---|---|---|
| 17/17 published=false 现状下 published selector 对所有 dataset 返回 unavailable，被误判为 bug | 中 | 这是正确行为（无 published current truth）；S04 实现前基线门 S01/S02/S03 verified；fixture 须构造 published=true catalog entry 验证 |
| publish.py / catalog.py / cli.py 三文件 merge_order 协调（S04 在 publish_py/catalog_py 首位、cli_py 第 2 位） | 低 | S04 是三文件 merge_order 首位/靠前，无前置写冲突；S14/S22/S33/S39 后续基于 S04 契约串行 |
| published pointer 前移执行被误授权 | 高 | dev_gate.published_pointer_advance_execution_allowed=false；REQ-248 不授权；本 Story 只 pointer 固定/selector，不前移 |
| 既有合同位置漂移（handoff 称 published 门禁 :2796，实际 :2778） | 低 | 已 grep 核验为 :2778；HLD Gotchas#1 亦引用 :2778；本说明以核验位置为准 |

### 偏离记录

- **无设计偏离**。S04 是 a 类，消费既有 HLD-DATA-LAKE §5/§17.4 契约，不重复设计（REQ-249）。
- **合同位置修正**：handoff v0.7 §1.3 称 published 门禁在 `readers.py:2796`，grep 核验实际为 `:2778`（HLD Gotchas#1 亦引用 :2778）。本说明以 :2778 为准，不传播 :2796 错误。
- **基线门硬依赖**：S04 depends_on S01/S02/S03（contract + runtime），属 Wave1 基线门结构性变更（REQ-247），实现前必须 S01/S02/S03 verified。

### lld_gate

- required: true
- required_level: technical-note
- status: ready-for-review（本技术说明已写入，等 CP5 全量人工确认）
