---
story_id: "CR139-S32"
title: "C4 写入去重保证"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "b 设计过期"
  rationale: "C4 是 b 类写入去重"
wave: "CR139-W2-CLEANUP"
status: "lld-approved"
req_coverage: ["REQ-207"]
depends_on: ["CR139-S07"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-207 (FEAT-02 写侧, b, technical-note)"
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

# CR139-S32 C4 写入去重保证

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-207。Owner Feature: FEAT-02 写侧。Wave: CR139-W2-CLEANUP。lld_policy: technical-note（b 设计过期；C4 是 b 类写入去重）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-207 (FEAT-02 写侧, b, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-207）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/normalization.py`
- `market_data/validation.py`

### 接口约定与设计约束
- 文件所有权：normalization.py 写入去重切片（merge_order 第 3 位）；validation.py（独占）
- 与 S07 C2b 读层去重语义对应
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
- 写 canonical 主键重复时去重或校验失败阻断。

## acceptance_criteria

写 canonical 主键重复时去重或校验失败阻断。

量化验收：
- 写 canonical 主键重复时去重或校验失败阻断。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-207 (FEAT-02 写侧, b, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: b 设计过期
- rationale: C4 是 b 类写入去重
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S07
- dependency_type: ["contract"]
- 文件所有权: normalization.py 写入去重切片（merge_order 第 3 位）；validation.py（独占）
- 与 S07 C2b 读层去重语义对应

## 技术说明

### 设计依据

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 REQ-207：C4 写入去重保证为 technical-note。
- S07 已完成读层去重；S32 设计写侧 canonical key 去重或阻断策略，避免继续依赖读层兜底。
- S32 位于 S31 后、S34/S14 前，先冻结写侧去重语义。

### 文件影响范围

| 文件 | 设计变更 | 边界 |
|---|---|---|
| `market_data/normalization.py` | 写 canonical 前按 dataset primary key 执行 dedup policy：确定性去重或阻断。 | 不重写真实 lake，不迁移历史分区。 |
| `market_data/validation.py` | 设计 duplicate key validation result，供 normalization fail closed。 | 只新增校验语义，不改变读层 S07 行为。 |
| `tests/test_cr139_write_dedup_guarantee.py` | fixture 覆盖重复主键去重、不可决重复阻断、无重复通过。 | static-only。 |

### 接口 / 数据 / 权限变化

- 新增 `validate_canonical_primary_key_uniqueness(frame, key_fields, policy)` 语义。
- policy 只允许 `deduplicate_deterministic` 或 `fail_on_duplicate`；禁止 silent append。
- 去重选择必须记录 duplicate_count、key_fields、selected_rule 和 dropped rows count。
- 权限无变化；不写真实 lake。

### 异常和回退

- 主键缺字段：阻断。
- 重复行完全相同：可确定性去重并记录计数。
- 重复主键但非主键字段冲突：阻断，交由上游数据修复。
- 如后续发现 dataset 无稳定主键，转 OPEN/Spike，不伪造主键。

### 测试入口

- `uv run --python 3.11 pytest -q tests/test_cr139_write_dedup_guarantee.py`
- 覆盖：重复完全相同、重复冲突、缺 key、无重复、禁止 silent append。

### 已知风险、偏离记录与重访条件

- 风险：不同 dataset key_fields 不一致。缓解：CP6 实现时使用 dataset contract 显式配置，不使用全局默认。
- 偏离：本 Story 不实现真实数据去重作业。
- 重访条件：新增 canonical dataset、primary key 合同变化、或真实 remediation 作业授权。

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
