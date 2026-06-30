---
cr_id: "CR-139"
title: "Strategy Data Foundation（数据湖整改 parent CR）"
cr_type: "product-scope"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
cp2_approved_by: "user"
cp2_approved_at: "2026-06-28T16:05:00+08:00"
cp3_approved_by: "user"
cp3_approved_at: "2026-06-28T17:30:00+08:00"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中数据湖架构重写、跨 FEAT-02/03/06/11/12/13/14 文件所有权与依赖方向、多 Story 依赖、实盘/交易审计链安全边界；按规则 43/2 不得使用 fast-lane，必须走 CP2→CP3→CP4/CP5 标准链路。"
rollback_to: "process/context/DATA-LAKE-REMEDIATION-HANDOFF.md v0.7 五轮评审定稿基线 + CR-018 未过 CP3 的现状"
approval_result: "story-backlog-closed-reconciled"
cp3_approval_result: "approved"
created_at: "2026-06-28T00:00:00+08:00"
created_by: "host-orchestrator"
approved_by: ""
approved_at: ""
source: "user-request"
linked_issue: ""
parent_cr: ""
superseded_in_scope:
  - "CR-017"
  - "CR-018"
source_checkpoint: "process/context/DATA-LAKE-REMEDIATION-HANDOFF.md"
source_decision_id: "USER-20260628-START-CR139"
follow_up_type: "strategy-data-foundation"
risk_class: "product-scope-data-trust-foundation"
owner: "host-orchestrator"
revisit_condition: "用户修改全流程四阶段范围、四组门禁集构成、Wave 1 基线冻结顺序或 D8a/D8b 纳入口径时，回退到 handoff v0.7 与 CR-139 CP2 基线后重新评估。"
acceptance_criteria: "冻结数据湖整改 parent CR 审计边界：覆盖市场数据湖 + 研究数据集/因子面板 + ML feature/label + run evidence + broker facts + 交易审计链 + 配置类事实源全数据底座，支撑多因子 + ML 策略从信息收集→回测→模拟盘→实盘全流程可信/可复现/可审计；45 项整改清单按 (a)=6/(b)=7/(c)=12/(d1)=6/(d2)=14 分类锁定，四组门禁集（数据/研究/ML/交易生产）作为 CP3 必过门禁；CR-139 本身只记录范围与决策边界，不进行 Story 拆解、LLD 或实现，CP2/CP3 通过后才进入 CP4/CP5。"
close_condition: "CP2 范围基线通过 + CP3 HLD 人工门禁通过（含 BLUEPRINT/DOMAIN-MAP/DEPENDENCY-MAP 刷新、companion HLD「Strategy Data Foundation」、FEATURE-DESIGN-MATRIX 刷新、四组门禁集落为 CP3 门禁），并记录当前 gate 未授权范围与后续 runtime_authorization 路径。"
conflict_keys:
  - strategy-data-foundation
  - data-lake-read-side-semantics
  - pit-as-of-panel-reader
  - published-current-pointer
  - ml-feature-label-artifact-layer
  - run-evidence-run-id-lineage
  - broker-facts-audit-chain
  - config-fact-source-versioning
  - schema-contract-freeze-pre-simulation
impact_surface:
  - process/context/DATA-LAKE-REMEDIATION-HANDOFF.md
  - process/changes/CR-139-STRATEGY-DATA-FOUNDATION-2026-06-28.md
  - process/changes/CR-INDEX.yaml
  - process/STATE.md
  - process/USE-CASES.md
  - process/REQUIREMENTS.md
  - docs/design/BLUEPRINT.md
  - docs/design/DOMAIN-MAP.md
  - docs/design/DEPENDENCY-MAP.md
  - docs/design/FEATURE-DESIGN-MATRIX.md
  - process/HLD-DATA-LAKE.md
  - process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md
  - docs/features/market-data-lake/
  - market_data/readers.py
  - market_data/lake_layout.py
  - market_data/publish.py
  - market_data/catalog.py
  - market_data/normalization.py
  - market_data/incremental.py
  - market_data/readiness.py
  - market_data/replay.py
  - market_data/duckdb_query.py
  - market_data/benchmarks.py
  - engine/research_manifest.py
  - engine/strategy_admission_package.py
  - engine/mature_multifactor_framework.py
  - engine/factor_model_validation.py
  - engine/factor_robustness.py
  - engine/contracts.py
  - trading/broker_lake.py
  - trading/qmt_gateway_contracts.py
  - trading/strategy_runner/evidence_index.py
  - experiments/run_experiment_15_*.py
  - experiments/run_experiment_23_29_*.py
  - pyproject.toml
authz_policy_refs:
  - NO_CREDENTIAL_READ
  - NO_RUNTIME
  - NO_NAS
  - NO_TRADING
  - NO_PROVIDER_LAKE_CATALOG_WRITE
  - NO_REMOTE_WRITE
  - NO_PHYSICAL_PARTITION_MIGRATION_BEFORE_BASELINE
---

# CR-139 Strategy Data Foundation（数据湖整改 parent CR）

## 状态收敛记录（2026-06-30）

`process/STATE.md` 与 `process/state/STATE.current.json` 已记录 CR139 40 个 Story 全部闭环：31 个 Story-level `verified`，9 个 `gate-reconciled-complete`，`lld-pending=0`。为启动 CR140 前消除 `cr-tracking` 的多 active CR 阻断，本 frontmatter 状态从 `active-cp7-pending-wave1-impl` 收敛为 `closed-current-delivery`。

本次状态收敛不改写 CR139 历史执行证据，不授权任何新的 lake/catalog/manifest/pointer/NAS/provider/runtime/Git 操作；后续 provider catalog、legacy cleanup、NAS、真实湖读写或运行时活动仍需独立授权。

> 本 CR 是**范围 / 需求类 parent CR**，按 Meta Flow 规则 25/26 必须 CR-first。CR 创建后回 `requirement-clarification`，由 `meta-pm` 执行 CR 触发的增量 use-case-discovery / requirement-extraction / requirement-clarifier 与 SCENARIOS / TEST-MATRIX / STORY-MAP / MVP-SCOPE 更新，过 CP1/CP2；CP2 未通过前不得进行 CR Story 分解、LLD 或实现。来源基线为 `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7（经五轮评审定稿，机械统计口径 (a)=6/(b)=7/(c)=12/(d1)=6/(d2)=14，逐项点数 6+7+12+6+14=45）。本 CR 只含判断、范围与决策边界，不含实现。

## 1. 变更描述

将数据湖模块从"已验证的小窗口市场数据链路"推进为**策略生产数据底座**，支撑多因子策略 + 机器学习策略从信息收集 → 回测 → 模拟盘 → 实盘全流程，达到可信、可复现、可审计。

整改对象从 v0.1 仅覆盖市场数据湖，扩展至全数据底座：

- 市场数据湖（写侧 + 读侧）
- 研究数据集 / 因子面板 / LabelWindow
- ML feature / label / 模型 artifact
- run evidence / 读审计
- broker facts / 订单 / 成交 / 持仓审计链
- 配置类事实源（benchmark / commission / universe·risk policy / 政策周期·shortability）
- 跨源与时点一致性

本 CR 收口 CR-017 / CR-018（二者从未过 CP3，与实现漂移）的范围，按评审 D1 新开 parent CR，不再续推 CR-018。整改性质为"既有对象与数据湖 release / lineage / ML feature / run-id 全链路闭环"，**非整体新建**——仅 §3 中 L3 / E4 / T7 / T8 / X1 / X2（d1）为纯新建对象。

## 2. 来源与评审基线

| 项 | 内容 |
|---|---|
| 来源文档 | `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 |
| 评审状态 | v0.1–v0.6 经五轮评审 / 自检返修；v0.7 修最后 3 处文字 / 附录残留，可进入 CR/CP2/CP3 审批链路 |
| 事实核查 | 数据湖目录核查 + 代码耦合分析 + 设计文档比对 + 五轮评审反馈核验 + §3 全表机械统计（脚本逐行解析可复核） |
| 计数口径 | (a)=6 / (b)=7 / (c)=12 / (d1)=6 / (d2)=14，总 45；遗漏分析驱动 d1+d2=20 |
| 收口对象 | CR-017、CR-018（superseded-in-scope，从未过 CP3） |

## 3. 整改目标与防线集

### 3.1 目标分解（按全流程四阶段）

| 阶段 | 数据底座必须提供 | 当前是否满足 |
|---|---|---|
| 信息收集 | 规范写入、PIT 盖戳、去重、CR 审计、血缘、配置类事实源 | 部分（写入在，PIT/去重/血缘/配置层未闭合） |
| 回测 | PIT 安全面板、去重、冻结快照、多表 join、universe 时变、复权正确 | 不满足 |
| 模拟盘 | 日级增量、pointer 前移、读前质量门禁、run evidence 贯通 | 不满足 |
| 实盘 | 低延迟读、schema 契约稳定、读审计、replay、决策时点-新鲜度对齐 | 不满足 |

### 3.2 防线集（四组门禁，CP3 必过 — 评审 D7）

#### 3.2.1 数据门禁组（原四道防线）
| 防线 | 含义 | 当前状态 |
|---|---|---|
| PIT 强制 | 读财报/估值按 `available_at <= as_of` 取最新 | reader 未执行 |
| 去重唯一 | `(symbol, trade_date)` 在 canonical 唯一 | glob-concat 可能多份 |
| 冻结快照 | published current pointer 固定 | candidate/gold/published 三层全空 |
| 多表 as-of join | 价格×财务×估值×行业统一 as-of 宽表 | panel reader 缺失 |

#### 3.2.2 研究门禁组
PIT universe / 幸存者偏差、复权与公司行动、交易状态/涨跌停/停牌、label 泄漏检查。

#### 3.2.3 ML 门禁组
特征离线/在线一致性、训练切分 manifest、模型 artifact/hash。

#### 3.2.4 交易生产门禁组
成本/滑点/成交可得性、读审计与 run-id 贯通、决策时点-新鲜度对齐。

## 4. 整改范围（45 项，五块需求）

CP2 范围基线拆为五块需求：数据湖整改 / 研究数据集·ML / 读侧语义 / 配置类事实源 / 交易审计链。整改清单按类别锁定如下（详见 handoff §3）：

| 类别 | 数量 | 实际项 | 设计动作 |
|---|---|---|---|
| (a) 已设计未实现 | 6 | C1、R3、R4、V1、M2、T1 | 不刷新设计，走 CR + LLD 落地 |
| (b) 设计过期/缺失 | 7 | N2、N3、C2、C3、C4、M1、M3 | 刷新对应设计章节 |
| (c) 范围扩展 | 12 | N1、R1、R2、V2、V3、V4、M4、E1、E2、T4、T2、T3 | CR → CP2 → CP3，刷新蓝图+HLD+矩阵 |
| (d1) 遗漏分析-纯新建 | 6 | L3、E4、T7、T8、X1、X2 | 按新建能力处理（D8a） |
| (d2) 遗漏分析-既有合同闭环 | 14 | L1、L2、L4、L5、E3、E5、T5、T6、F1、F2、F3、F4、X3、X4 | 不重复设计，只做集成闭环/版本化扩展（D8b） |

四道 P0 防线：**C1** PIT as-of reader、**C2** 分区去重、**V1** published 快照、**R1** panel reader。

### 4.1 既有合同清单（闭环非新建，修正"把已有写成无"）

下列对象**已存在合同或局部实现**，整改性质为"既有对象与数据湖 release / lineage / ML feature / run-id 全链路闭环"，不是新建（详见 handoff §1.6）：

| 对象 | 已存在位置 | 整改性质 |
|---|---|---|
| ExperimentManifest | `engine/research_manifest.py:152` | 闭环到 published data release + lineage |
| StrategyAdmissionPackage | `engine/strategy_admission_package.py:127` | 闭环到 catalog current pointer + run-id |
| RunEvidenceIndex | `trading/strategy_runner/evidence_index.py:19` | 闭环到 lake run-id 贯通 |
| BrokerLakeSchema | `trading/broker_lake.py:64` | 闭环到实盘写 + 审计链 |
| CommissionSchedule | `trading/qmt_gateway_contracts.py:997` | 闭环到成本前置（T5） |
| label 泄漏审计 / embargo / ml_leakage_audit | `engine/factor_model_validation.py:561`、`:376`、`engine/factor_robustness.py:53` | 统一接入 data release + cutoff gate（E3） |
| BenchmarkCoverage / BenchmarkDefinition | `market_data/benchmarks.py:99`、`:114` | 补版本化 benchmark + risk-free curve + release 闭环（F1） |
| incremental refresh 计划合同 | `market_data/incremental.py` `plan_incremental_refresh` | 接通真实日级 append 执行链（L1） |
| publish_current_pointer | `market_data/publish.py:605` | 接通真实 pointer 前移执行（L2） |
| readiness matrix | `market_data/readiness.py:462` `build_readiness_matrix` | 前置为实盘读前强制阻断门禁（L4） |
| replay 合同 | `market_data/replay.py`、`cli.py:437 cmd_p0_replay` | 接通 published as_of 单日快照重放（L5） |
| decision_time 过滤 | `market_data/readers.py:227`、`:1790` | 加强制 lookahead 阻断门禁（X3） |
| survivorship_bias_note | `engine/contracts.py:270` | 补按时点构建 PIT universe 成分链（X4） |
| PortfolioRiskPolicy | `engine/mature_multifactor_framework.py:228` | 补版本化 universe/risk policy 事实源 + release 闭环（F3） |
| policy_cycle_coverage | `engine/factor_model_validation.py:444`、`config/policy_cycles.yaml` | 补版本化政策周期/shortability 事实源 + release 闭环（F4） |

> 仅 L3、E4、T7、T8、X1、X2（d1）为纯新建对象。本 CR 涉及上述对象的整改项，"问题"列含义统一为"未与数据湖 release / run-id / artifact ref / lineage 闭环"、"分散存在未统一"或"合同已存在缺执行/版本化/前置"，而非"对象不存在"。

## 5. 设计刷新判断

| 文档 | 刷新原因 | 刷新内容 | 责任角色 |
|---|---|---|---|
| `docs/design/BLUEPRINT.md` | 范围扩展至全数据底座 | 增补"策略生产数据底座"能力地图；明确 ML feature 层、配置类事实源层、run evidence、broker facts、交易审计链的 Feature 归属和数据归属 | meta-se |
| `process/HLD-DATA-LAKE.md` | 停在 CR018 未过 CP3、与实现漂移；范围未覆盖 ML feature/读侧 join/训练快照/读审计/配置层 | **新开 companion HLD「Strategy Data Foundation」**（评审 D2 命名），写侧/读侧分离 | meta-se |
| `docs/design/FEATURE-DESIGN-MATRIX.md` | 缺 ML feature 层/读侧语义/run evidence/配置层 Feature 归属 | 新增或扩展 Feature，补 `lld_policy` 和重访条件；明确 V3/L3/L4/R2/E*/T*/F*/X* 归属 | meta-se |
| `docs/features/market-data-lake/` | stub 停在 CR014，严重过期 | 重写 DESIGN/TASKS/TEST-PLAN 覆盖新整改项 | meta-dev（CP5 后） |

类别 (a) 的 6 项（C1、R3、R4、V1、M2、T1）：HLD 已有契约，LLD 消费即可，不刷新设计。

## 6. 落地路线图（CR 编排，CP3 通过后才进入 CP4/CP5）

### Wave 1 — 冻结基线 + 信任基础（P0，解锁回测）
评审重排：**先冻结现状基线和重复画像，再实现 published pointer/read selector，再做读层去重，最后做候选压缩/迁移。不先改物理分区**，否则"历史数据变化"与"结构修复"混在一起无法归因。

> **CP2 用户追加约束（2026-06-28T16:05:00+08:00）**：Wave1 任何**结构性变更**（V1 pointer、C1 reader、R1 panel reader、C2b 读层去重、N1 物理迁移等改变读/写路径或分区结构的变更）必须在 **T8 清册 → T7 黄金值基线 → C2a 重复画像**三步完成之后才允许执行。基线冻结未完成前不得启动结构性变更；结构性变更后的结果必须能对 T7 黄金值做归因对比。此约束强化 handoff §5 重排口径，写入 REQ-247。

1. T8 整改对象自动化清册（全 dataset 行数/覆盖/重复键/pit/published/lineage）
2. T7 整改回归基线 + 黄金值快照（下游因子/回测结果冻结对比基准）
3. C2a 重复画像（看清 38 分区里哪些 `(symbol,trade_date)` 重复、来自哪些 run）
   — ↑ 基线冻结门：以上三步完成后方可执行以下结构性变更 ↓
4. V1 实现 published pointer / read selector
5. C1 PIT as-of reader（`financial_pit`/`market_cap` 按 `available_at` 过滤）
6. R1 统一 published/as-of panel reader（复用 `read_dataset` published 门禁）
7. C2b 读层去重（按 source_run_id 取最新版本）
8. N1 后置 候选压缩/物理分区迁移（最后做，基于 T7 黄金值对比归因）

### Wave 2 — 接入与规模（P1，解锁 ML + 模拟盘）
评审 HIGH3：**V4 schema 契约冻结必须在模拟盘前完成**，从 Wave3 移入 Wave2。

9. V4 schema 演进规则 + 实盘契约冻结（模拟盘前必过）
10. R2 ML 接入 lake，废除 `--data-dir`/`load_local_frames` 旁路
11. V3 feature/label/artifact 层 + E1/E2/E5 ExperimentManifest/模型 hash/split manifest
12. R3 DuckDB 只读 adapter
13. L1/L2 日级增量 + pointer 前移
14. E3/E4 label 泄漏检查 + 离线/在线一致性
15. F1/F2/F3 配置类事实源层
16. T4/T5/T6 broker facts + 成本前置 + run-id 贯通（既有对象闭环）
17. X1/X2/X3/X4 跨源与时点一致性
18. L3/L4 读审计 + 读前质量门禁
19. T2/T3 PIT/去重正确性测试
20. N1/C3/M1 收尾

### Wave 3 — 实盘与运维（P2）
21. L5 replay、R4 延迟优化、T1、M3/M4、F4

## 7. 决策项（D1–D7 已同意；D8a/D8b 二轮建议批准）

| 决策 ID | 待确认问题 | 推荐方案 | 评审结论 |
|---|---|---|---|
| D1 | 收口 CR018 还是新开 parent CR？ | 新开 parent CR | 同意 |
| D2 | 扩 HLD 还是新 companion HLD？ | 新开，命名「Strategy Data Foundation」 | 同意（附命名） |
| D3 | 存量 run_id 一次性重命名？ | 不重命名，新数据规范化 + 旧数据下次重跑替换 | 同意 |
| D4 | DuckDB 只读引擎还是主存储？ | 只读查询引擎，parquet 仍是存储 | 同意 |
| D5 | ML feature 层 lake 子层还是独立 store？ | lake `features/` 子层带版本 | 可接受（附条件：保留切换到独立 feature store 的条件） |
| D6 | C2 前强制基准快照？ | 是，强制 | 同意（升级为 T7 整改回归基线机制） |
| D7 | 四道防线做 CP3 门禁？ | 是，扩展为"数据/研究/ML/交易生产门禁集" | 同意（附扩展） |
| **D8a** | (d1) 纯新建对象 6 项是否纳入范围？ | 纳入 | 二轮建议批准（L3/E4/T7/T8/X1/X2） |
| **D8b** | (d2) 既有合同闭环/扩展 14 项是否纳入范围？ | 纳入，按"既有合同接通闭环/版本化扩展"处理，**不重复设计** | 二轮建议批准（L1/L2/L4/L5/E3/E5/T5/T6/F1/F2/F3/F4/X3/X4；E1/E2/T4 归 (c) 不计 D8） |

> D8a/D8b 在 CP2 范围基线人工门禁中作为待人工决策项正式确认。

## 8. 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/USE-CASES.md` | 增量更新 | 既有场景基线保留；CR-139 追加策略生产数据底座场景 | `## 修订记录` | pending（CP2） |
| `process/REQUIREMENTS.md` | 增量更新 | 既有需求基线保留；CR-139 追加五块需求 | `## 修订记录` | pending（CP2） |
| `docs/design/BLUEPRINT.md` | 原文档更新 | 既有 Feature 边界保留；增补"策略生产数据底座"能力地图 | `## 修订记录` | pending（CP3） |
| `process/HLD-DATA-LAKE.md` | 标注 superseded-in-scope by CR-139 | 保留为历史追溯，不再续推 | `## 修订记录` | pending（CP3） |
| `process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md` | 新增 | 新 companion HLD，写侧/读侧分离 | `## 修订记录` v0.1 | pending（CP3） |
| `docs/design/FEATURE-DESIGN-MATRIX.md` | 原文档更新 | 既有 Feature 保留；补 ML feature/读侧/run evidence/配置层归属与 `lld_policy` | `## 修订记录` | pending（CP3） |
| `docs/features/market-data-lake/` | 原文档更新 | CR014 stub 不回滚；CP5 后重写 | DESIGN/TASKS/TEST-PLAN | pending（CP5） |
| `docs/USER-MANUAL.md` / `README.md` | 后续 Wave 更新 | 当前仅列为影响对象 | 后续修订记录 | pending（CP8） |

## 9. 不授权范围（non-authorized scope）

CR-139 是范围 / 需求类 parent CR，只记录审计边界，**不授权**以下操作；任何运行时、凭据、外部接口、数据写入、publish、live / 交易类事项必须形成独立 `runtime_authorization` 决策项：

- 真实数据湖写入 / 物理分区迁移 / published pointer 前移执行（Wave 1 N1 物理迁移明确后置到基线冻结之后）
- NAS / 外部存储 / provider / catalog 运行时连接与凭据读取
- QMT / MiniQMT / gateway 运行时、下单 / 撤单 / 模拟盘 / 实盘
- 实盘读审计写、broker facts 实盘写、run evidence 上传
- Git remote write / commit / push / 分支治理 / 历史重写
- 任何 publish、live、production 写入

CP2 / CP3 通过不等于授权上述操作；后续必要验证按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。

## 10. 协议约束

- 本次是范围 / 需求类变更，按规则 25/26 必须 CR-first：CR 后回 `requirement-clarification`，CP2 过基线、CP3 过 HLD 人工门禁后才能拆 Story 和 LLD（规则 1/2）。
- HLD / 蓝图只能 meta-se 改；feature DESIGN 三件套是 meta-dev 在 CP5 后落地（规则 8）。
- CP3 前不得实现，不得命中 fast-lane（规则 2/43）。
- 全阶段 Context Capsule：CP2/CP3/CP5/CP6/CP7/CP8 前后生成或检查 `process/context/*-CONTEXT.yaml`（规则 33）。
- 四组门禁集为 CP3 必过门禁（D7）；V4 schema 契约冻结必须前置到模拟盘前（评审 HIGH3）。

## 11. CR 编排与下一站

```
parent CR「Strategy Data Foundation」(CR-139, D1)
 ├─ CP1: 用户场景完备门（自动）
 ├─ CP2: 范围基线 — 拆成 数据湖整改 + 研究数据集/ML + 读侧语义 + 配置层 + 交易审计链 五块需求
 │        + D8a/D8b 纳入口径人工确认
 ├─ CP3: HLD 人工门禁
 │   ├─ 刷新 BLUEPRINT.md / DOMAIN-MAP.md / DEPENDENCY-MAP.md
 │   ├─ 新建 companion HLD「Strategy Data Foundation」（写侧/读侧分离，D2 命名）
 │   ├─ 刷新 FEATURE-DESIGN-MATRIX.md
 │   └─ 四组门禁集落为 CP3 门禁（D7）
 ├─ CP4/CP5: 按 Wave 1 P0（T8/T7/C2a/V1/C1/R1/C2b）拆 Story + LLD
 └─ CP6-CP8: 实现 → 验证 → 交付
```

**下一站**：CR-139 已建立审计边界。回 `requirement-clarification`，由 `meta-pm` 执行 CR 触发的增量 use-case-discovery / requirement-extraction / requirement-clarifier，产出 / 更新 `SCENARIOS.yaml`、`TEST-MATRIX.md`、`STORY-MAP.md`、`MVP-SCOPE.md`，过 CP1/CP2 范围基线门。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-28 | host-orchestrator | 基于 handoff v0.7 起草 CR-139 parent CR 审计边界；锁定 45 项分类口径、四组门禁、Wave 路线图、D1-D8 决策、文档处理决策与不授权范围；CP2/CP3 待启动 |
| v0.2 | 2026-06-28 | host-orchestrator | CP2 范围基线门通过（用户 2026-06-28T16:05:00+08:00 approve）：D8a 6 项纳入按新建能力处理（d1 完整设计+LLD）、D8b 14 项纳入按既有合同闭环/版本化扩展不重复设计（d2 F1-F4 版本化/其余闭环）；追加 Wave1 结构性变更基线门约束——结构性变更必须在 T8 清册→T7 黄金值基线→C2a 重复画像之后执行（写入 §5 Wave1 与 REQ-247）；USE-CASES/REQUIREMENTS 升 v1.17 ready_for_design=true；CP2 产物 SCENARIOS/TEST-MATRIX/STORY-MAP/MVP-SCOPE/RELEASE-SLICES/BACKLOG 已就绪；下一站 CP3 由 meta-se 刷新蓝图/矩阵/新开 companion HLD |
| v0.3 | 2026-06-28 | host-orchestrator | CP3 HLD 架构评审门通过（用户 approve 2026-06-28T17:30:00+08:00）：CP3-DQ-01 (AGA-1)=A1 分层同属 FEAT-02、CP3-DQ-02 (AGA-3)=C1 lake features/ 子层、CP3-DQ-03 (AGA-5)=E1 复用 V1 pointer 语义；BLUEPRINT v1.14/DOMAIN-MAP v1.13/DEPENDENCY-MAP v1.13/FEATURE-DESIGN-MATRIX v1.13/companion HLD v0.2/ADR v0.2 全部 pending-cp3 → confirmed-cp3；HLD/ADR confirmed=true；下一站 CP4/CP5 story-planning（按 Wave1 P0 拆 Story+LLD）；CP3 不授权实现/runtime/真实写入/物理迁移/Git remote write |
| v0.4 | 2026-06-28 | host-orchestrator | CP5-W1 LLD 批次通过（用户首轮 changes_requested 6 项返修闭环 + 二轮 approve 2026-06-28T19:20:00+08:00）：Wave1 8 Story 设计证据完成（6 full-lld 14章 + 2 technical-note），S02 最小 MetricSpec 7 类冻结、S06 异粒度物化算法 7 条冻结、S05/S06 catalog CP6 硬门禁、状态全同步；进入 Wave1 实现（CP6/CP7，基线门 S01→S02→S03 verified 后 S04-S08 结构性变更才实现）；按 Wave 分批 CP5，Wave1 verified 后再写 Wave2 LLD |
