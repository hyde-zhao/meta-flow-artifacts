---
validation_mode: mixed
applies_to: ptm-team 全部 6 个 Agent
note: 语义见 §2 验证模式；本字段为规则 12 要求的机器可读声明
---

# PTM Team 测试策略

> 版本：v1.0 · 更新：2026-06-08 · 适用范围：ptm-team 全部 6 个 Agent

---

## 1. 测试分层

| 层级 | 说明 | 适用对象 | 当前状态 |
|---|---|---|---|
| **L0 静态检查** | lint、format、结构检查、guardrail | 全部 Skill/Agent/脚本 | ptm-tde 已覆盖 |
| **L1 单元/契约测试** | Skill 输入输出契约、Prompt 模板变量 | 全部 Skill | ptm-tde CR 级 CP6 已覆盖 |
| **L2 集成测试** | Agent 流程端到端、Skill 调用链 | ptm-tde 12 步流程 | CR 级 CP7 已覆盖 |
| **L3 dry-run / 人工审查** | 场景覆盖、语义质量、Gate 门控 | 人工检查点 | ptm-tde 5 个人工检查点 |
| **L4 运行时验证** | 真实防火墙环境测试 | ptm-te/ptm-tae | 不授权（依赖硬件） |

## 2. 验证模式

| 模式 | 说明 | ptm-tde 应用 |
|---|---|---|
| `static-only` | 仅静态检查，无需运行时环境 | CP6 编码完成检查 |
| `review-only` | 仅人工语义审查 | CP3/CP5/CP8 人工门控 |
| `dry-run-only` | 模拟执行，不接触真实环境 | Skill 输入输出验证 |
| `mixed` | 组合多种模式 | CP7 验证完成检查 |

> ptm-tde 当前验证模式为 `mixed`（static + dry-run + review），**不包含运行时验证**。运行时验证依赖防火墙硬件/拓扑/仪表，不在本交付范围内。

## 3. 各 Agent 测试策略

### ptm-tde（已交付）

| 维度 | 策略 |
|---|---|
| Skill 单元 | 每个 Skill 的输入输出契约通过 CP6 检查 |
| 流程集成 | 12 步主流程 + 扩展分支通过 CR 级 CP7 验证 |
| 人工审查 | 5 个检查点通过 CP3/CP5/CP8 人工门控 |
| 覆盖追溯 | SCENARIOS.yaml → TEST-MATRIX.md 待 P2 创建 |

### ptm-tm/tse/te/tae/qa（待开发）

| 维度 | 策略 |
|---|---|
| 启动标准 | 各自启动时重新执行 CP0-CP2 流程 |
| 测试标准 | 遵循本策略 L0-L3 分层 |
| 验证证据 | CP6/CP7 检查点 + Agent Dispatch Evidence |

## 4. 质量门控

| 门控 | 准出条件 | ptm-tde 状态 |
|---|---|---|
| CP3 HLD | 自动预检 PASS + 人工 approved | ✅ 4/4 CR 通过 |
| CP5 LLD | 全量 Story 设计证据 batch approved | ✅ 4/4 batch 通过 |
| CP6 编码 | 单 Story PASS，Agent Dispatch Evidence | ✅ 28/28 Story 通过 |
| CP7 验证 | PASS/PASS_WITH_RISK/WAIVED | ✅ 12/12 全局通过 |
| CP8 交付 | 自动预检 PASS + 人工 approved | ✅ 4/4 CR 通过 |

## 5. 风险与限制

| 风险 | 等级 | 缓解 |
|---|---|---|
| CP0-CP5 gate_inheritance 未在本仓库重跑 | MEDIUM | 后续发现基线问题发起 CR 回溯 |
| 低风险 Story 缺少 IMPLEMENTATION.md | LOW | CR 级 CP6 已覆盖，P1 补充 |
| 无真实运行时验证环境 | MEDIUM | 不在交付范围内，明确不授权 |
| 其余 5 个 Agent 测试策略待定 | LOW | 各自启动时执行 CP0-CP2 |

## 6. 测试工具

| 工具 | 用途 | 状态 |
|---|---|---|
| `scripts/check_delivery_guardrails.py` | 交付护栏静态检查 | ptm-tde 已集成 |
| `uv run --python 3.11 python <script>` | Python 脚本验证 | 可用 |
| `git diff --check` | 空白字符检查 | 可用 |
| atomic-ops CLI | 原子操作查询验证 | 可用（79 ops） |

---

*本策略覆盖 ptm-team 全部 6 个 Agent。ptm-tde 已按此策略完成交付，其余 Agent 启动时参考执行。*

---

## 7. CR-030 ptm-tse 逆向分析测试策略

> 适用范围：CR-030（FEAT-RA-INGESTION + FEAT-RA-ANALYSIS + FEAT-RA-TRACKING），即 ptm-tse Agent 的 ITR 问题单摄取、清洗、逆向分析与改进跟踪能力。
> 本章节于 2026-07-16 CP7 回修时追加，补充原 TEST-STRATEGY.md 中 CR-030 的覆盖缺失。

### 7.1 覆盖范围

| 覆盖对象 | 类型 | 实现形态 | 关联 Story |
|---------|------|---------|-----------|
| `skills/itr-ticket-ingestion/SKILL.md` | Skill 文本定义 | 10 章（allowlist 校验、HTTP GET、快照保存、批次清单、数据库写入、字段映射清洗、质量报告、失败保护、变更检测、Gotchas） | ST-RA-05.1-INGEST、ST-RA-05.2-CLEAN、ST-RA-06.1-DETECT、ST-NRA-03 |
| `skills/reverse-analysis/SKILL.md` | Skill 文本定义 | 10 章（资格检查、证据分类、六维分析、根因状态机、指标降级、S1 管线、证据保护、权限边界、S2 增量重算、Gotchas） | ST-RA-01、ST-RA-02、ST-RA-05.3-ANALYZE、ST-RA-06.2-REFRESH、ST-NRA-01、ST-NRA-02 |
| `skills/improvement-tracker/SKILL.md` | Skill 文本定义 | CA/PA 治理：批准输入不可变、下游只读消费、审计追溯 | ST-RA-03 |
| `data/dao.py` | 数据库接口层 | 公共 DAO 函数（ticket/batch/version/change_history/analysis_run CRUD）；SQLite WAL 模式 | ST-RA-INGEST-DB |
| `data/schema.sql` | 数据库 Schema | ticket、ingestion_batch、ticket_version、change_history、analysis_run 表定义 | ST-RA-INGEST-DB |
| `delivery/agents/ptm-tse.md` | Agent 定义 | ptm-tse Agent 的触发条件、能力边界、Skill 编排规则 | ST-RA-AGENT-DEF |
| 模板文件 | 配置/模板 | allowlist-config.yaml、field-mapping.yaml、quality-report.yaml、batch-manifest.yaml、conflict-queue.yaml、ra-report.yaml、metric-definition.yaml、analysis-run-manifest.yaml | 各 Skill 私有模板 |

### 7.2 验证模式

**`validation_mode: static-only`**

CR-030 的交付物包含 Skill 文本定义、模板文件、数据库 Schema 定义（schema.sql，可执行 DDL）和 Python 数据访问层（dao.py，Python 代码）。本轮验证模式为 `static-only`：仅执行静态/内存 DDL 验证（sqlite3 executescript）和 Python AST 解析，未做 ITR 网络和真实数据运行验证。

| 验证层 | 适用？ | 说明 |
|--------|:----:|------|
| L0 静态检查 | **是** | Skill 文件结构完整性（章节数、必选段）、模板 YAML schema 校验、数据库 DDL 语法检查 |
| L1 单元/契约测试 | **是（静态）** | Skill 输入输出契约的静态一致性检查（参数定义 vs 算法伪代码/流程图）、DAO 函数签名与调用方契约对齐 |
| L2 集成测试 | N/A | 无运行时集成环境；Skill 间调用链通过文档引用契约（shared_writers + 章节引用）验证 |
| L3 dry-run / 人工审查 | **是（review-only）** | CP5 全量设计证据人工确认、CP7 独立复核 |
| L4 运行时验证 | N/A | 无实际 HTTP 调用、无 SQLite 运行测试、无 reviewer 交互 |

### 7.3 N/A 的运行验证项

以下运行验证因 CR-030 本轮验证模式为 static-only（仅执行静态/内存 DDL 验证和 Python AST 解析，未做 ITR 网络和真实数据运行验证）而标记为 N/A：

| 验证项 | N/A 原因 | 风险 | 缓解 |
|--------|---------|------|------|
| ITR HTTP GET 实际调用 | ITR 服务 `http://10.113.53.108/itr/v1/itrs` 为内网地址，本地开发环境不可达 | MEDIUM — allowlist 校验逻辑、超时处理和错误分类无法通过真实 HTTP 调用验证 | allowlist 校验算法（§2.2）和 HTTP GET 执行伪代码（§2.3）通过静态逻辑审查 + 人工 walkthrough 验证；首次部署到可访问内网的环境时执行一次受控冒烟测试 |
| SQLite 数据库运行测试 | 数据库写入通过 DAO 公共接口调用，DAO 实现本身不在 CR-030 范围（ST-RA-INGEST-DB 提供） | LOW — DAO 函数签名和调用契约已在 Skill 文件中以表格形式明确（§5.1、§9.1），静态一致性可检查 | 通过 `data/dao.py` 的公共函数签名与 Skill 调用代码的交叉引用审查验证 |
| 质量报告 QCOMB 规则触发 | 无真实 ITR 数据源，无法生成含 blocked/anomaly/incomplete quality_flag 的记录以触发 QCOMB 组合规则 | LOW — QCOMB-01~05 的判定逻辑以伪代码形式定义（§7.4），触发条件和阈值均为显式常量 | 通过人工构造的静态 quality_report 样例验证判定路径的代码分支覆盖 |
| S2 增量重算实际执行 | 变更检测（ST-RA-06.1-DETECT）的 change_set 产出和 S2 管线消费的串联逻辑无法运行验证 | LOW — 受影响维度映射表（§9.2.1）和增量重算策略（§9.3）以表格和伪代码形式定义 | 通过静态映射表交叉引用（变更字段 → 维度 → 重算范围）验证覆盖完整性 |
| reviewer 交互流程 | analysis_run 的 `completed → published` 跃迁需要 reviewer 显式调用 `reviewer_publish_analysis_run()`，无法在无交互环境中验证 | LOW — DAO 接口已定义安全约束（§6.7.3），包括不可自动发布、不可从未完成状态发布、不可回退 | 通过 DAO 接口契约的静态审查 + 人工确认流程的文档审查验证 |
| conflict-queue 人工处理 | 冲突队列的 reviewer 回填 `reviewer_decision` + `resolution_timestamp` 为人工操作，无法自动化验证 | LOW — conflict-queue.yaml 模板结构已定义（§9.4.2），人工回填字段和格式已明确 | 通过模板格式校验 + reviewer 操作说明文档审查验证 |
| 环比同比窗口计算 | 同比环比（mom/yoy）需要两个完整自然月/自然季度的 SQLite 数据，当前无历史数据积累 | LOW — 比较逻辑（§3.2.6）和 N/A 判定规则（§9.5.6）已以伪代码和表格形式定义 | 通过静态 N/A 条件表验证：所有标记条件均有对应的 `na_reason` |

### 7.4 验证证据路径

CR-030 各 Story 的验证证据通过以下路径追溯：

| Story | CP6 检查文件 | Skill 文件章节 | 验证类型 |
|-------|------------|--------------|---------|
| ST-RA-05.1-INGEST | `process/checks/CP6-ST-RA-05.1-INGEST-*-CODING-DONE.md` | `itr-ticket-ingestion/SKILL.md` §1-§5 | static review |
| ST-RA-05.2-CLEAN | `process/checks/CP6-ST-RA-05.2-CLEAN-*-CODING-DONE.md` | `itr-ticket-ingestion/SKILL.md` §6-§7 | static review |
| ST-RA-05.3-ANALYZE | `process/checks/CP6-ST-RA-05.3-ANALYZE-*-CODING-DONE.md` | `reverse-analysis/SKILL.md` §6 | static review |
| ST-RA-06.1-DETECT | `process/checks/CP6-ST-RA-06.1-DETECT-*-CODING-DONE.md` | `itr-ticket-ingestion/SKILL.md` §9 | static review |
| ST-RA-06.2-REFRESH | `process/checks/CP6-ST-RA-06.2-REFRESH-*-CODING-DONE.md` | `reverse-analysis/SKILL.md` §9 | static review |
| ST-RA-01 | `process/checks/CP6-ST-RA-01-*-CODING-DONE.md` | `reverse-analysis/SKILL.md` §1-§2 | static review |
| ST-RA-02 | `process/checks/CP6-ST-RA-02-*-CODING-DONE.md` | `reverse-analysis/SKILL.md` §3-§5 | static review |
| ST-NRA-01 | `process/checks/CP6-ST-NRA-01-*-CODING-DONE.md` | `reverse-analysis/SKILL.md` §7 | static review |
| ST-NRA-02 | `process/checks/CP6-ST-NRA-02-*-CODING-DONE.md` | `reverse-analysis/SKILL.md` §8 | static review |
| ST-NRA-03 | `process/checks/CP6-ST-NRA-03-*-CODING-DONE.md` | `itr-ticket-ingestion/SKILL.md` §8 | static review |
| ST-RA-03 | `process/checks/CP6-ST-RA-03-*-CODING-DONE.md` | `improvement-tracker/SKILL.md` | static review |

### 7.5 质量门控（CR-030 状态）

| 门控 | 准出条件 | CR-030 状态 |
|------|---------|:----------:|
| CP3 HLD | 自动预检 PASS + 人工 approved | PASS（HLD REV-03） |
| CP5 LLD | 全量 Story 设计证据 batch approved | PASS（full-lld + technical-note + waived batch） |
| CP6 编码 | 单 Story PASS，Agent Dispatch Evidence | PASS（14/14 CP6 检查文件均 PASS，含 4 个前期缺失 Story 的 CP6 补齐） |
| CP7 验证 | PASS/PASS_WITH_RISK/WAIVED | PASS（14/14 Story static review 完成，CP7 独立复核整改轮次已关闭） |
| CP8 交付 | 自动预检 PASS + 人工 approved | 待人工终验 |

### 7.6 风险与限制（CR-030 专属）

| 风险 | 等级 | 缓解 |
|------|:----:|------|
| ITR 内网服务不可达 → allowlist/HTTP GET 逻辑未运行验证 | MEDIUM | 首次部署到可访问内网环境时执行一次受控冒烟测试；静态 allowlist 校验算法已通过 walkthrough 审查 |
| 无真实 ITR 数据 → 清洗管线 QCOMB 规则未触发验证 | LOW | QCOMB 判定逻辑（伪代码）已通过静态分支覆盖分析；首次摄取真实数据后监控 quality_report 输出 |
| S1/S2 管线串联逻辑未运行验证（Skill 仅定义文本，无执行引擎） | LOW | 管线步骤以编号流程（Step IR-1~5 等）定义，静态顺序一致性可检查；实际执行依赖 ptm-tse Agent 编排 |
| 环比同比无历史数据积累 → N/A 条件仅静态验证 | LOW | 所有 N/A 触发条件在 §9.5.6 中以表格形式枚举，可与实际运行时的 na_reasons 输出对照验证 |
| 4 个 Story 的 CP6 检查文件已补齐 | — | 见 FIXES.md B2，已由另一 agent 修复，CP7 复验通过 |
| 2 个 Skill 的 Gotchas 章节已补齐 | — | Gotchas 内容来自设计契约的显式约束和已知踩坑记录，已通过 CP7 回修轮次审查 |

---

*CR-030 测试策略章节结束。后续 Story 实现或 CR 变更时更新本节状态。*
