---
story_id: "CR139-S30"
title: "N2+N3 run_id 命名规约统一"
change_id: "CR-139"
owner_feature: "FEAT-02 写侧"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: "b 设计过期 命名整理类"
  rationale: "N2/N3 是 b 类命名/整理"
wave: "CR139-W2-CLEANUP"
status: "lld-approved"
req_coverage: ["REQ-202", "REQ-203"]
depends_on: ["CR139-S08"]
dependency_type: ["contract"]
feature_design_refs:
  - "MATRIX v1.13 REQ-202/203 (FEAT-02 写侧, b, technical-note)"
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

# CR139-S30 N2+N3 run_id 命名规约统一

## 背景与目标

本 Story 属于 CR-139「Strategy Data Foundation」CP4 Story 拆解，覆盖 REQ-202, REQ-203。Owner Feature: FEAT-02 写侧。Wave: CR139-W2-CLEANUP。lld_policy: technical-note（b 设计过期 命名整理类；N2/N3 是 b 类命名/整理）。

设计来源：companion HLD v0.2（confirmed-cp3）+ ADR v0.2 + FEATURE-DESIGN-MATRIX v1.13。本卡片仅记录 Story 边界与验收，完整 LLD（full-lld）或技术说明（technical-note）在 CP5 后由 meta-dev 写入。



## dev_context

### 输入文件
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§9/§10/§12/§17）
- `process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` v0.2
- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13（MATRIX v1.13 REQ-202/203 (FEAT-02 写侧, b, technical-note)）
- `process/REQUIREMENTS.md` v1.17（REQ-202, REQ-203）
- `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（代码触点）

### 输出文件
- `market_data/normalization.py`
- `market_data/lake_layout.py`

### 接口约定与设计约束
- 文件所有权：run_id 生成处（merge_order 第 3 位，S08 N1 后）
- 聚合 N2+N3；与 S08 N1 file-conflict，N1 先
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
- 新 run 命名符合 run-<purpose>-<window>-<source>-<YYYYMMDD> 统一前缀规约，无 unknown run_id（N2）；新路径不含 CR 编号，存量路径保留（N3）。

## acceptance_criteria

新 run 命名符合 run-<purpose>-<window>-<source>-<YYYYMMDD> 统一前缀规约，无 unknown run_id（N2）；新路径不含 CR 编号，存量路径保留（N3）。

量化验收：
- 新 run 命名符合 run-<purpose>-<window>-<source>-<YYYYMMDD> 统一前缀规约，无 unknown run_id（N2）；新路径不含 CR 编号，存量路径保留（N3）。

## feature_design_refs

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 — MATRIX v1.13 REQ-202/203 (FEAT-02 写侧, b, technical-note)
- `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` v0.2（§5/§10/§17）
- companion HLD v0.2 对应章节

## lld_policy

- required_level: `technical-note`
- trigger_reasons: b 设计过期 命名整理类
- rationale: N2/N3 是 b 类命名/整理
- 重访条件：见 MATRIX v1.13 §重访条件增量（FEAT-02/03/06/11/12/14）。

## 依赖与文件所有权

- depends_on: CR139-S08
- dependency_type: ["contract"]
- 文件所有权: run_id 生成处（merge_order 第 3 位，S08 N1 后）
- 聚合 N2+N3；与 S08 N1 file-conflict，N1 先

## 技术说明

### 设计依据

- `docs/design/FEATURE-DESIGN-MATRIX.md` v1.13 REQ-202/203：N2/N3 为命名/整理类 technical-note。
- `process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml`：S30 先于 S31/S32/S33/S34/S14/S22/S39 收敛 run_id 命名。
- S08 已 verified，S30 只消费 S08 的 run_id partition key 治理结果，不执行物理分区迁移。

### 文件影响范围

| 文件 | 设计变更 | 边界 |
|---|---|---|
| `market_data/normalization.py` | 设计统一 run_id 生成入口，输出 `run-<purpose>-<window>-<source>-<YYYYMMDD>` 格式；禁止新 run_id 使用 `unknown` 或 CR 编号。 | 只设计新写入路径，不重写存量分区。 |
| `market_data/lake_layout.py` | 设计路径 helper 的 run_id 命名校验和 legacy path 容忍策略。 | 存量 CR 命名路径保留只读兼容。 |
| `tests/test_cr139_runid_naming.py` | fixture/static 覆盖合法命名、unknown 阻断、CR 编号路径不再新增。 | 不访问真实 lake。 |

### 接口 / 数据 / 权限变化

- 新增或收敛 `build_run_id(purpose, window, source, as_of_date)` 语义，输出必须匹配 `^run-[a-z0-9-]+-[a-z0-9-]+-[a-z0-9-]+-[0-9]{8}$`。
- 新增 `validate_run_id_name(run_id)`，用于 normalization/lake_layout 共享校验。
- 新写入路径不得包含 `CR\d+`、`unknown` 或临时批次名；存量路径保持可读，不做自动迁移。
- 权限无变化；不读取凭据，不启动 runtime，不写真实 lake。

### 异常和回退

- 缺少 purpose/window/source/date 时 fail closed，返回明确错误，不生成 `unknown`。
- 检测到存量 CR 命名路径时只做 legacy read/diagnostic，不改路径。
- 如发现外部调用方依赖旧 run_id 格式，先保留兼容解析，并把删除旧格式转后续 CR。

### 测试入口

- `uv run --python 3.11 pytest -q tests/test_cr139_runid_naming.py`
- fixture 场景：合法 run_id、缺字段阻断、CR 编号禁止新增、legacy path 只读兼容。

### 已知风险、偏离记录与重访条件

- 风险：命名 helper 若被多个写侧绕过，会继续产生不合规 run_id。缓解：CP6 实现时把 helper 作为唯一新写入入口。
- 偏离：不做真实分区改名，不关闭 legacy 读取。
- 重访条件：新增写侧 dataset、run_id 分区策略变化、或用户授权物理迁移时重开设计。

## 不授权范围声明

本 Story CP4 阶段不授权：LLD 写作（CP5）/ 实现 / runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移执行 / published pointer 前移执行 / broker lake 实盘写 / Git remote write / 真实 lake 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。
