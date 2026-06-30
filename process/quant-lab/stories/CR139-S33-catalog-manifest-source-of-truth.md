---
story_id: "CR139-S33"
title: "M1 catalog/manifest 定主"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "b 设计过期"
  rationale: "M1 是 b 类真相源定主"
wave: "CR139-W2-CLEANUP"
status: "lld-approved"
req_coverage: ["REQ-208"]
depends_on: ["CR139-S04"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-208 (FEAT-02 写侧, b, technical-note)"
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

# CR139-S33 M1 catalog/manifest 定主

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-208。Owner Feature: FEAT-02 写侧。Wave: CR139-W2-CLEANUP。lld_policy: technical-note（b 设计过期；M1 是 b 类真相源定主）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-208 (FEAT-02 写侧, b, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-208）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/catalog.py`
- `market_data/manifest.py`

### 接口约定与设计约束
- 文件所有权：catalog.py（merge_order 第 2 位，S04 V1 后）；manifest.py 派生化（与 S34 M2 串行）
- 与 S04 V1/S39 M4 file-conflict
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
- catalog 与 manifest 一致性校验，catalog 为真相源，manifest 派生。

## acceptance_criteria

catalog 与 manifest 一致性校验，catalog 为真相源，manifest 派生。

量化验收：
- catalog 与 manifest 一致性校验，catalog 为真相源，manifest 派生。
- 验收补充（CR139-W2-DATA-CONTRACTS）：`market_data/catalog.py` 中历史 CR014/CR018 命名 API 必须纳入 S33 设计证据清理范围。S33 技术说明需设计领域命名 API / constants，保留旧 CR014/CR018 alias 兼容，补充领域命名测试，并明确旧 alias 的 deprecation / 迁移路径；不得直接删除旧 API。
- 验收补充边界：S39 只负责 `triggered_by_cr` 与 `run_lineage` 审计链，不承担 CR014/CR018 命名清理主体；本补充不授权 provider catalog 写入、真实 lake 写入、published pointer advance execution 或物理分区迁移执行。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-208 (FEAT-02 写侧, b, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: b 设计过期
- rationale: M1 是 b 类真相源定主
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S04
- dependency_type: ["contract"]
- 文件所有权: catalog.py（merge_order 第 2 位，S04 V1 后）；manifest.py 派生化（与 S34 M2 串行）
- 与 S04 V1/S39 M4 file-conflict

## 技术说明

### 设计依据

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 REQ-208：M1 catalog/manifest 定主为 technical-note。
- `process/checks/CP5-CR139-W2-DATA-CONTRACTS-DESIGN-EVIDENCE-CONVERGENCE-PLAN.md`：S33 是 `catalog.py` CR014/CR018 命名清理 owner，S39 只负责审计链字段。
- S04 V1 已提供 published pointer/read selector 基线；S33 在其后冻结 catalog source-of-truth。

### 文件影响范围

| 文件 | 设计变更 | 边界 |
|---|---|---|
| `market_data/catalog.py` | 设计 catalog 为真相源；新增领域命名 constants/API；保留 CR014/CR018 alias。 | 不写 provider catalog，不推进 pointer。 |
| `market_data/manifest.py` | 设计 manifest 从 catalog 派生/校验，不再作为冲突真相源。 | lineage checksum 由 S34 消费。 |
| `tests/test_cr139_catalog_manifest_source_of_truth.py` | 覆盖 catalog/manifest 一致性、领域命名 API、legacy alias 兼容。 | fixture/static-only。 |

### 接口 / 数据 / 权限变化

- 推荐领域命名：`CATALOG_POINTER_REQUIRED_FIELDS`、`RELEASE_EVIDENCE_REQUIRED_FIELDS`、`build_catalog_pointer_record()`、`build_release_evidence_record()`、`validate_catalog_manifest_consistency()`。
- 旧 `CR014_*`、`CR018_*`、`build_cr018_*()` 和 `_cr018_*()` 作为 alias 或 compatibility wrapper 保留，标注 deprecated，但不得在本 Story 直接删除。
- catalog 为 source-of-truth；manifest 只存派生 view 或 release-specific snapshot，冲突时以 catalog 记录为准并阻断。
- 权限无变化；不写 provider catalog、不写真实 lake、不执行 pointer advance。

### 异常和回退

- catalog/manifest 字段不一致：阻断并输出 dataset/release_id/field diff。
- legacy alias 被调用：继续返回同等行为，并在测试中证明兼容。
- 发现旧 API 外部依赖广泛：保持 alias，无 deprecation removal 日期，后续 CR 再迁移调用方。

### 测试入口

- `uv run --python 3.11 pytest -q tests/test_cr139_catalog_manifest_source_of_truth.py`
- 覆盖：catalog 为真相源、manifest 派生、领域 API 与 legacy alias 等价、禁止 provider catalog write。

### 已知风险、偏离记录与重访条件

- 风险：命名清理误删 public API。缓解：本 Story 只新增领域 API 和 alias，不删除旧符号。
- 偏离：不把 S39 审计链字段纳入命名清理主体。
- 重访条件：外部调用方迁移完成、provider catalog 写入另行授权、或 S39 需要新增审计字段影响 catalog schema。

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
