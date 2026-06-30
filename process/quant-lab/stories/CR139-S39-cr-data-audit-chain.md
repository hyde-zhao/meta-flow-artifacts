---
story_id: "CR139-S39"
title: "M4 CR→数据审计链"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "c 范围扩展（MATRIX 逐项标 technical-note，审计链字段补全低风险）"
  rationale: "M4 是 c 类但 MATRIX 标 technical-note"
wave: "CR139-W3-OPS"
status: "lld-approved"
req_coverage: ["REQ-211"]
depends_on: ["CR139-S08", "CR139-S33"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-211 (FEAT-02 写侧, c, technical-note)"
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

# CR139-S39 M4 CR→数据审计链

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-211。Owner Feature: FEAT-02 写侧。Wave: CR139-W3-OPS。lld_policy: technical-note（c 范围扩展（MATRIX 逐项标 technical-note，审计链字段补全低风险）；M4 是 c 类但 MATRIX 标 technical-note）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-211 (FEAT-02 写侧, c, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-211）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/catalog.py`

### 接口约定与设计约束
- 文件所有权：catalog.py triggered_by_cr/run_lineage（merge_order 第 3 位，S04/S33 后）
- P2；与 S04/S33 file-conflict
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
- catalog 查询可追溯 triggered_by_cr 与 run_lineage。

## acceptance_criteria

catalog 查询可追溯 triggered_by_cr 与 run_lineage。

量化验收：
- catalog 查询可追溯 triggered_by_cr 与 run_lineage。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-211 (FEAT-02 写侧, c, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: c 范围扩展（MATRIX 逐项标 technical-note，审计链字段补全低风险）
- rationale: M4 是 c 类但 MATRIX 标 technical-note
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S08, CR139-S33
- dependency_type: ["contract"]
- 文件所有权: catalog.py triggered_by_cr/run_lineage（merge_order 第 3 位，S04/S33 后）
- P2；与 S04/S33 file-conflict

## 技术说明

### 设计依据

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 REQ-211：M4 CR->数据审计链为 technical-note。
- S39 消费 S33 catalog source-of-truth 和 S08/S30 run_id/path 规则。
- `CR139-W2-DATA-CONTRACTS` 准备门禁明确：S39 只负责 `triggered_by_cr` / `run_lineage`，不承担 CR014/CR018 命名清理主体。

### 文件影响范围

| 文件 | 设计变更 | 边界 |
|---|---|---|
| `market_data/catalog.py` | 为 catalog record 设计 `triggered_by_cr`、`run_lineage`、`audit_refs` 只读查询字段。 | 不改 CR014/CR018 API 命名主体，不写 provider catalog。 |
| `tests/test_cr139_cr_data_audit_chain.py` | fixture 覆盖 CR 到 dataset/run lineage 可追溯、缺字段阻断、S33 alias 兼容不受影响。 | static-only。 |

### 接口 / 数据 / 权限变化

- `triggered_by_cr` 推荐格式：`CR-139` 或后续正式 CR ID；不得使用临时会话 ID 代替。
- `run_lineage` 至少包含 data_run_id、source_run_id、publish_run_id 或对应缺失状态。
- catalog 查询必须能从 CR -> dataset release -> run lineage 追溯；缺字段返回断链状态，不伪造。
- 权限无变化；不写真实 lake，不写 provider catalog，不推进 pointer。

### 异常和回退

- 缺 `triggered_by_cr`：记录 `audit_status=missing_trigger` 并 fail closed 审计检查。
- 缺 run lineage：记录断点，交由 S22/S30/S14 修复，不在 S39 补造。
- 发现 CR014/CR018 命名问题：回指 S33，不在 S39 处理。

### 测试入口

- `uv run --python 3.11 pytest -q tests/test_cr139_cr_data_audit_chain.py`
- 覆盖：CR 追溯、run lineage 追溯、缺字段断链、S33 命名 alias 不破坏。

### 已知风险、偏离记录与重访条件

- 风险：S39 与 S33 同改 `catalog.py` 产生边界漂移。缓解：merge order S33 -> S39；S39 只增审计链字段。
- 偏离：不执行 provider catalog 写入，不做命名清理主体。
- 重访条件：S33 catalog schema 变化、S22 run lineage 字段变化、或新增 provider catalog runtime 授权。

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
