---
log_type: clarification
cr_id: CR-139
phase: requirement-clarification
delegated_to: meta-pm
created_at: 2026-06-28T00:00:00+08:00
status: active
---

# CLARIFICATION-LOG — CR-139 Strategy Data Foundation

> meta-pm 在 `requirement-clarification` 委托期间维护。多轮追加，不覆盖。

## 调研发现（2026-06-28，阶段零快速调研）

### 现有可复用资源

- **既有场景基线**：`process/USE-CASES.md` v1.16（50 UC，confirmed 2026-06-24）与 `process/REQUIREMENTS.md` v1.16（REQ-001..200，confirmed）。CR-139 在此基线上增量，保留 UC-01..UC-50 / REQ-001..200 编号，新增 UC-51+ / REQ-201+。
- **既有合同（核验存在，闭环非新建）**：以下对象经 `grep` 核验确认已存在合同或局部实现，整改性质为"既有对象与数据湖 release / lineage / ML feature / run-id 全链路闭环"，**不是新建**（遵守 memory `verify-existing-code-before-claiming-absent`）：

| 对象 | 核验位置 | 核验结果 |
|---|---|---|
| ExperimentManifest | `engine/research_manifest.py:152` `class ExperimentManifest` | 存在，dataclass + 校验 |
| StrategyAdmissionPackage | `engine/strategy_admission_package.py:127` | 存在，dataclass + 序列化 |
| RunEvidenceIndex | `trading/strategy_runner/evidence_index.py:19` `from_run_result` | 存在 |
| BrokerLakeSchema | `trading/broker_lake.py:64` | 存在，schema registry |
| CommissionSchedule | `trading/qmt_gateway_contracts.py:997` | 存在 |
| publish_current_pointer | `market_data/publish.py:605` | 存在（dry-run only） |
| build_readiness_matrix | `market_data/readiness.py:462` | 存在 |
| plan_incremental_refresh | `market_data/incremental.py:248` | 存在（dry-run plan） |
| decision_time 过滤 | `market_data/readers.py:227` | 存在（部分支持） |
| survivorship_bias_note / is_pit_universe | `engine/contracts.py:33/151/152` | 存在（已识别偏差） |
| BenchmarkCoverage / BenchmarkDefinition | `market_data/benchmarks.py:99/114` | 存在，contract/resolver |
| PortfolioRiskPolicy | `engine/mature_multifactor_framework.py:228` | 存在 |
| policy_cycle_coverage | `engine/factor_model_validation.py:69/124` | 存在，评估入口 |
| label 泄漏审计 / embargo / ml_leakage_audit | `engine/factor_model_validation.py:376/561`、`engine/factor_robustness.py:53` | 存在（分散） |
| replay 合同 | `market_data/replay.py:215 run_replay_from_manifest` | 存在（部分） |

- **Skill 模板**：`scenario-expansion`（SCENARIOS-TEMPLATE.yaml / TEST-MATRIX-TEMPLATE.md）、`story-planning`（STORY-MAP / MVP-SCOPE / RELEASE-SLICES / BACKLOG 模板）均可复用。
- **既有 CP2 产物模式**：CR-014/018/019/025/030/046 均采用 CR-scoped `process/` 产物（CP2-CRxxx-REQUIREMENTS-BASELINE.md、CP2-CRxxx-DISCUSSION-CHECKPOINT.json），CR-139 沿用此模式。

### 平台能力约束

- `engagement_mode=production`、`scenario_subject_type=target-artifact`（用户未声明 meta 自优化，按规则 20/21 默认）。
- 产物路由严格按 capsule `产物路由` 章节：写入 `process/` 与子目录，**不新建** `docs/product/`（项目无此约定）。
- 不授权范围（CR-139 §9 + capsule）：runtime / NAS / QMT / trading / provider-lake-catalog 写入 / 物理分区迁移（Wave1 N1 后置到基线冻结后）/ Git remote write。CP2/CP3 通过不等于授权上述操作。
- CP2 不拆 Story / 不写 LLD / 不实现代码（规则 25）；CP3 由 meta-se 刷新 BLUEPRINT / DOMAIN-MAP / DEPENDENCY-MAP / FEATURE-DESIGN-MATRIX 并新开 companion HLD「Strategy Data Foundation」。

### 对需求的初步影响

1. **CR-139 范围与既有 UC/REQ 的关系**：CR-139 是 parent CR，superseded-in-scope CR-017/CR-018（二者从未过 CP3）。既有 UC-09/UC-13/UC-14（数据湖 production current truth）和 REQ-088..137 是 CR-139 的直接前身；CR-139 将数据湖从"市场数据湖"扩展为"策略生产数据底座"，覆盖信息收集→回测→模拟盘→实盘全流程 + 全数据资产层（市场数据 + 研究数据集/因子面板 + ML feature/label + run evidence + broker facts + 交易审计链 + 配置类事实源）。
2. **45 项整改如何映射到四阶段流程**：机械统计 (a)=6/(b)=7/(c)=12/(d1)=6/(d2)=14，总 45，无重复（脚本核验）。映射到 5 块需求（数据湖整改 / 研究数据集·ML / 读侧语义 / 配置类事实源 / 交易审计链）与 3 个 Wave：Wave1 P0（冻结基线+信任基础，解锁回测）、Wave2 P1（接入与规模，解锁 ML+模拟盘）、Wave3 P2（实盘与运维）。四道 P0 防线 C1/C2/V1/R1 在 Wave1。
3. **既有合同清单核验结论**：§1.6 全部 15 类对象核验存在。仅 L3/E4/T7/T8/X1/X2（d1）为纯新建。整改性质是"闭环/版本化扩展"，非"新建"——此结论已固化到 USE-CASES / REQUIREMENTS 增量中，避免"把已有合同写成无"导致重复设计和范围膨胀（handoff 二轮 BLOCKING1 教训）。

### 计数核验（遵守 memory `mechanical-recount-with-script`）

脚本逐项点数：a=6 (C1,R3,R4,V1,M2,T1) + b=7 (N2,N3,C2,C3,C4,M1,M3) + c=12 (N1,R1,R2,V2,V3,V4,M4,E1,E2,T4,T2,T3) + d1=6 (L3,E4,T7,T8,X1,X2) + d2=14 (L1,L2,L4,L5,E3,E5,T5,T6,F1,F2,F3,F4,X3,X4) = 45，无重复。与 handoff §3.12 / CR-139 §4 一致。

---

## 澄清结论（2026-06-28，Step 2 requirement-clarifier）

### D8a — (d1) 纯新建 6 项纳入口径

- **待确认问题**：L3 / E4 / T7 / T8 / X1 / X2（纯新建对象，无既有合同）是否纳入 CR-139 范围？
- **推荐方案**：纳入。按新建能力处理（需完整设计 + LLD）。
- **依据**：handoff v0.7 二轮建议批准。这 6 项是信任基础 / 可审计 / 跨源一致性的必要能力：
  - T7 整改回归基线 + 黄金值快照（Wave1 P0 前置，D6 升级）
  - T8 整改对象自动化清册（Wave1 P0 第 1 步，事实校验前置）
  - L3 读审计 log + run-id 贯通（"可审计"是用户三大目标之一，P2→P1 升级）
  - E4 特征离线/在线一致性校验（ML 门禁组必要项）
  - X1 复权因子本身 PIT 正确性校验（研究门禁组细化）
  - X2 跨源交易日历/时区一致性（全流程必要）
- **影响面**：范围、复杂度、验证、CP3 门禁。
- **当前状态**：建议批准（working basis）。正式确认由 host-orchestrator 在 CP2 人工门禁完成。

### D8b — (d2) 既有合同闭环 14 项纳入口径

- **待确认问题**：L1/L2/L4/L5/E3/E5/T5/T6/F1/F2/F3/F4/X3/X4（既有合同闭环/扩展）是否纳入 CR-139 范围？
- **推荐方案**：纳入，按"既有合同接通闭环 / 版本化扩展"处理，**不重复设计**。
- **依据**：handoff v0.7 二轮建议批准（附口径）。15 类既有合同已 grep 核验存在（见上表）。整改性质是"合同已存在缺执行/版本化/前置闭环"，而非"对象不存在"。
- **影响面**：范围、设计路由（走集成闭环 LLD，不刷新设计）、CP3 Feature 归属。
- **当前状态**：建议批准（working basis）。正式确认由 host-orchestrator 在 CP2 人工门禁完成。
- **核验原则**：遵守 memory `verify-existing-code-before-claiming-absent`——已逐项 grep 核验，未把已有合同写成无。

### D1-D7 追溯确认（已同意，CP2 仅追溯）

| 决策 | 结论 | CP2 处理 |
|---|---|---|
| D1 新开 parent CR | 同意 | CR-139 已建立，追溯确认 |
| D2 新 companion HLD「Strategy Data Foundation」 | 同意 | CP3 由 meta-se，CP2 仅记录范围 |
| D3 不重命名存量 run_id | 同意 | 纳入 REQ 约束 |
| D4 DuckDB 只读引擎 | 同意 | 纳入 REQ（R3/R4/T1） |
| D5 ML feature 层 lake `features/` 子层带版本 | 可接受（附条件：保留切换独立 feature store 条件） | 纳入 REQ（V3），切换条件写入 Deferred |
| D6 C2 前强制基准快照 | 同意（升级为 T7） | 纳入 REQ（T7），Wave1 P0 |
| D7 四组门禁集为 CP3 必过 | 同意（扩展） | 纳入 REQ 约束（REQ-246），CP3 落门禁 |

### BLOCKING 未决项

- **0 项 BLOCKING**。D8a/D8b 已达"建议批准"，作为 working basis 推进产物；正式人工确认路由到 CP2 门禁（host-orchestrator 发起，meta-pm 不得自行发起）。
- 产物按建议批准口径编写；若 CP2 人工门禁修改 D8a/D8b 口径，meta-pm 按"决策修订再发布"规则增量回更。

### 交互工具说明

本次委托未提供 `AskUserQuestion` 平台工具。meta-pm 按"建议批准"working basis 推进产物，SGQ-* 场景确认证据来源为 handoff v0.7 五轮评审 trail + 本澄清结论；D8a/D8b 正式确认与灰区重点选择在最终交还摘要中呈现给用户，由 host-orchestrator 在 CP2 人工门禁统一确认。此处理方式已记录在 `CP2-CR139-DISCUSSION-CHECKPOINT.json` 的 `interaction_mode` 字段。
