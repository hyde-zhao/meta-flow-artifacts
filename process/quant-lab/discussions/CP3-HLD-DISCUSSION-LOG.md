---
discussion_id: "CP3-CR139-HLD-DISCUSSION"
cr_id: CR-139
phase: solution-design
delegated_to: meta-se
status: pending-user-decision
created_at: 2026-06-28T16:40:00+08:00
previous_cycle: "CP3-CR138-HLD-DISCUSSION (completed; audit trail preserved in process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md and CR-138 ADR)"
---

# CP3 HLD Discussion Log — CR-139 Strategy Data Foundation

> 本日志记录 CR-139 companion HLD「Strategy Data Foundation」**方案形成前**的 CP3 Architecture Gray Areas 讨论输入。advisor lane 使用 table-first 结构（`Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch`）。本日志为审计与恢复用，不替代 BLUEPRINT/DOMAIN-MAP/DEPENDENCY-MAP/HLD/ADR/FEATURE-DESIGN-MATRIX（规则 46）。

## 讨论范围与来源

- 来源基线：`process/context/CP3-CR139-STRATEGY-DATA-FOUNDATION-CONTEXT.yaml` + handoff v0.7 + CR-139 v0.2 + USE-CASES v1.17 UC-51..57 + REQUIREMENTS v1.17 REQ-201..249 + BLUEPRINT v1.12 + DOMAIN-MAP v1.11 + DEPENDENCY-MAP v1.11 + FEATURE-DESIGN-MATRIX v1.11。
- 既有合同已 grep 核验（见 §0），闭环非新建（REQ-249）。
- 用户三大目标：可信 / 可复现 / 可审计。四组门禁为 CP3 必过（D7/REQ-246）。Wave1 基线门（REQ-247）。V4 schema 冻结前置到模拟盘前（HIGH3）。

## §0 既有合同代码位置核验（memory: verify-existing-code-before-claiming-absent）

| 对象 | 核验位置 | 核验结果 |
|---|---|---|
| ExperimentManifest | `engine/research_manifest.py:152 class ExperimentManifest` | 存在 |
| StrategyAdmissionPackage | `engine/strategy_admission_package.py:127` | 存在（CR-139 §4.1） |
| RunEvidenceIndex | `trading/strategy_runner/evidence_index.py:19 class RunEvidenceIndex` + `from_run_result:33` | 存在 |
| broker lake schema registry | `trading/broker_lake.py`（BrokerLakeEventType:23 / BrokerLakePlanStatus:36 / BrokerLakeErrorCode:51 + dry-run writer） | 存在（schema registry 形态，非单一 class 名 BrokerLakeSchema） |
| CommissionSchedule | `trading/qmt_gateway_contracts.py:997 class CommissionSchedule` | 存在 |
| publish_current_pointer | `market_data/publish.py:605 def publish_current_pointer` | 存在（dry-run only） |
| read_dataset + published 门禁 | `market_data/readers.py:2728 def read_dataset` + `:2778 if not entry.published` | 存在 |
| plan_incremental_refresh | `market_data/incremental.py:248 def plan_incremental_refresh` | 存在 |
| build_readiness_matrix | `market_data/readiness.py:462 def build_readiness_matrix` | 存在 |
| run_replay_from_manifest | `market_data/replay.py:215 def run_replay_from_manifest` + `cli.py cmd_p0_replay` | 存在 |
| BenchmarkCoverage / BenchmarkDefinition | `market_data/benchmarks.py:99 / :114` | 存在 |
| PortfolioRiskPolicy | `engine/mature_multifactor_framework.py:228 class PortfolioRiskPolicy` | 存在 |
| policy_cycle_coverage | `engine/factor_model_validation.py:69/124/143` + `config/policy_cycles.yaml` | 存在 |
| label 泄漏 / embargo | `engine/factor_model_validation.py:376 embargo` + `factor_robustness.py:53 ml_leakage_audit` | 存在（分散） |
| decision_time 过滤 | `market_data/readers.py:227 / :278 / :1670` | 存在（部分支持） |
| survivorship_bias_note / is_pit_universe | `engine/contracts.py:151/152/270` | 存在（已识别偏差） |

结论：d2 14 项既有合同真实存在，整改性质为"接通闭环 / 版本化扩展 / 前置门禁"，非新建（REQ-249）。仅 L3/E4/T7/T8/X1/X2（d1）为纯新建。

---

## AGA-1：写侧 / 读侧分离边界（companion HLD 如何切写侧与读侧）

> 真正影响架构走向。决定 companion HLD 的章节结构、FEAT-02 内部职责切分、读侧 Story owner 与文件所有权。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A1 写侧/读侧分层同属 FEAT-02，HLD 分写侧章 + 读侧章 | 边界清晰、读侧语义（PIT/dedup/panel/selector/audit）独立收口、不新增 Feature、与既有 HLD-DATA-LAKE 写侧重叠最小 | FEAT-02 内部需明确读写子域边界与文件所有权（readers.py 内 read-side 段 vs publish/normalize write-side 段） | FEAT-02 模块表、companion HLD 章节、读侧 Story owner、readers.py 文件段 | **推荐** | 假设 readers.py 可按 read-side / write-side 分段而不拆文件。**When to switch**：CP5 发现读侧 Story 与写侧文件所有权冲突频繁、或读侧语义膨胀超出单 Feature → 升级 A3 拆 FEAT-02R |
| A2 写侧读侧同归 FEAT-02 单一职责，HLD 不分读写章 | Feature 数量不增、最简单 | 读侧语义（PIT/panel/dedup/audit）与写侧 normalize/publish/lineage 在同一章重叠，companion HLD 难收口"写侧/读侧分离"决策 D2 | companion HLD 结构、读侧 Story 归属模糊 | 不推荐 | D2 决策已要求"写侧/读侧分离"，A2 与 D2 冲突 |
| A3 读侧拆出独立 FEAT-02R（read-side）Feature | 边界最清、读侧独立演进 | 新增 Feature、与 FEAT-03 panel reader 消费边界重叠、增加蓝图复杂度 | BLUEPRINT Feature 表、FEATURE-DESIGN-MATRIX、依赖图 | 备选（fallback） | 仅当 A1 在 CP5 出现读写冲突时升级 |

**推荐理由**：D2 决策明确要求 companion HLD 写侧/读侧分离；A1 在不新增 Feature 的前提下用 HLD 章节结构 + 模块表 + 文件段归属实现分离，最贴合既有蓝图且可回退。读侧统一面板 reader（R1）虽同时被 FEAT-02/03 消费，但 owner 仍在 FEAT-02 读侧子域，FEAT-03 只读消费。

---

## AGA-2：published current pointer 与 catalog 主从（M1 双真相源）

> 已由 handoff §3.5 M1 + REQ-208 基本锁定，列此仅作 advisor 留痕与切换条件记录。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| B1 catalog.json 为主真相源，manifest.jsonl 为派生 | 单一真相源、publish gate 语义一致、与 V1 pointer 闭环 | catalog 单文件写入需保证原子性 | catalog.py、manifest.py、publish.py | **推荐（既定）** | 假设 catalog 单文件可承载 17→N dataset。**When to switch**：catalog 单文件成为写入瓶颈 → 引入 catalog 分片（按 dataset）但仍保持单一真相源语义，不退回双真相源 |
| B2 manifest 为主 | manifest 已有增量写入链路 | 与 publish gate / current pointer 语义不一致 | manifest.py | 不推荐 | 与 RULE-02 冲突 |
| B3 双向同步无主 | 无 | 双真相源漂移、归因困难 | 全流程 | 不推荐 | 现状问题根源 |

---

## AGA-3：ML feature 层归属与切换条件（D5 lake `features/` 子层 vs 独立 feature store）

> 真正影响架构走向。决定 V3 feature/label/artifact 层的物理归属、版本化 schema、artifact hash 闭环形态与未来切换路径。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| C1 lake `features/` 子层带版本 + schema，feature/label/artifact 审计链，保留切换独立 store 条件（DEF-139-01） | 与 lake publish/release 闭环天然一致、不引入新依赖、artifact hash 可引用 dataset snapshot、切换条件显式记录 | parquet 子层对高频/在线 serving 不友好；feature 复用规模大时查询延迟可能上升 | FEAT-03 feature 层、lake_layout.py 新增 features/ 子层、E1/E2/E5 审计链、DEF-139-01 | **推荐（D5 既定，附条件）** | 假设当前为日频离线研究 + 模拟盘，无在线特征 serving。**When to switch**：(a) feature 数量/复用规模使 parquet 子层查询延迟不可接受；(b) 引入在线特征 serving / 实盘低延迟特征读取需求 → 切换 C2 独立 feature store，届时另起 architecture CR + V3 重设计 |
| C2 直接独立 feature store（Feast / 自建） | 在线 serving 友好、feature 复用强 | 引入新依赖、与 lake release 闭环需额外桥接、CP3/CP5 需评估依赖 | 新依赖、FEAT-03 边界 | 备选（deferred） | DEF-139-01 记录切换条件；当前 CP3 不引入 |
| C3 不持久化 feature 层，每次从 raw lake 重算（现状） | 无新存储 | V3 现状问题：不可复现、ML 被迫旁路、训练-实盘漂移 | — | 不推荐 | 现状缺陷，V3 整改目标即消除 |

**推荐理由**：D5 已定 lake `features/` 子层带版本；当前为日频离线 + 模拟盘场景，parquet 子层 + 版本化 schema + artifact hash 闭环足以满足"可复现/训练-实盘一致"，且不引入新依赖。切换条件必须显式写入 ADR（DEF-139-01），避免未来无依据切换。

---

## AGA-4：run-id 贯通与读审计（L3/T6）

> 影响模块边界与跨 Feature 契约。L3 为 d1 纯新建，T6 跨 FEAT-02/03/06/11。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| D1 reader 读审计 log + 数据 run-id 贯穿 RunEvidenceIndex → broker event，同 run-id 贯通 | 满足"可审计"目标、复盘可从信号回溯到数据版本/订单/成交/对账、与既有 RunEvidenceIndex 闭环 | 读审计需新建模块（L3 d1）；run-id 贯通跨 4 Feature 需统一契约 | readers.py 读审计、evidence_index.py、broker_lake.py、FEAT-06/11 事件 schema | **推荐** | 假设 run-id 作为贯穿主键。**When to switch**：若 run-id 贯通性能/复杂度开销过大 → 读审计异步落盘但 run-id 关联不丢；不退回 D2/D3 |
| D2 读审计与 run-id 解耦，各自独立 id | 实现简单 | 复盘断链、不满足"可审计" | — | 不推荐 | 与用户三大目标冲突 |
| D3 只在 broker 侧审计，reader 不落读审计 | 最少改动 | 信号→数据回溯断、L3 缺失 | — | 不推荐 | L3 已 P2→P1 升级 |

**跨边界归属**：T6 run-id 贯通跨 FEAT-02/03/06/11，owner 为数据 run-id 生成侧（FEAT-02 写侧），消费方为 RunEvidenceIndex（FEAT-11）与 broker event（FEAT-06）；读审计 log owner 为 FEAT-02 读侧（L3 d1 新建）。

---

## AGA-5：配置类事实源版本化机制（F1-F4）— 复用 V1 published pointer 还是独立

> 真正影响架构走向。决定 benchmark/commission/universe·risk policy/政策周期四类配置事实源的版本化与 release 闭环机制是否统一。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| E1 复用 V1 published pointer 语义统一版本化 + release 闭环；配置类事实源物理存储独立子目录（如 `config_facts/`）而非混入 market data canonical | 机制统一、四类配置与 dataset 一样可版本化/可回溯到 release、减少机制分叉、复用既有 publish/current pointer 门禁 | 配置类语义（如 policy_cycle 的周期性）与 dataset pointer 略有差异，需在 schema 上区分 version 语义 | FEAT-02/12/14/03 配置层、publish.py 复用、新增 config_facts 子层、F1-F4 release 闭环 | **推荐** | 假设四类配置的版本化语义可用 `version + effective_from + release_id` 统一表达。**When to switch**：若某类配置（如 policy_cycle 的周期覆盖）版本化语义与 dataset pointer 差异过大、或复用 publish 门禁造成配置变更被数据 publish 节奏卡死 → 该类独立版本化机制，其余仍复用 E1 |
| E2 各类配置独立版本化机制（benchmark registry / commission registry / policy registry 各一套） | 每类可量身定制 | 四套机制、维护成本高、release 闭环分叉 | 四个独立 registry | 备选 | 仅当 E1 对某类配置不适用时局部切换 |
| E3 配置不入 lake，只存 config 文件 + git 版本 | 最简单 | 无 release 闭环、归因不可复现、与"配置类事实源"目标冲突 | — | 不推荐 | UC-56 目标即消除此现状 |

**推荐理由**：UC-56 要求四类配置事实源"像数据 dataset 一样可版本化、可回溯到某次 release"。复用 V1 published pointer 语义（version + effective_from + release_id + current pointer）最一致且减少机制分叉；物理存储独立子目录避免与 market data canonical 混淆。policy_cycle（F4 P2）若周期语义差异大可局部独立，但 F1/F2/F3 优先复用。

---

## AGA-6：schema 演进与契约冻结（V4）

> 已由评审 HIGH3 + REQ-220 基本锁定（移入 Wave2 模拟盘前必过），列此作 advisor 留痕。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| F1 schema 演进规则 + reader 兼容回退 + 模拟盘前契约冻结 | 实盘契约稳定、reader 不因 schema 变更崩、模拟盘前到位 | 兼容回退需维护、演进规则需定义 | contracts.py、readers.py、V4 | **推荐（既定 HIGH3）** | 假设 schema 变更可分级（兼容/破坏）。**When to switch**：若兼容回退维护成本过高 → 收紧为 F2（冻结不演进，变更需重跑）但需评估重跑成本 |
| F2 schema 冻结不演进，任何变更需重跑 | 无兼容负担 | 重跑成本高、不灵活 | — | 备选 | 极端情况 |
| F3 schema 自由演进，reader 不做兼容 | 无 | runner 读湖会崩 | — | 不推荐 | 实盘风险 |

---

## 待用户决策项（AskUserQuestion）

> 以下三项为真正影响架构走向的灰区，由 meta-se 通过 AskUserQuestion 向用户确认。AGA-2/4/6 为既定或半既定，不升级为用户必须决策项，仅作审计记录。

| 决策 ID | 灰区 | 推荐方案 | 备选 | 影响 |
|---|---|---|---|---|
| DQ-CP3-AGA1 | AGA-1 写侧/读侧分离边界 | A1 分层同属 FEAT-02 + HLD 分读写章 | A3 拆 FEAT-02R | companion HLD 结构、Feature 边界、Story owner |
| DQ-CP3-AGA3 | AGA-3 ML feature 层归属与切换条件 | C1 lake features/ 子层带版本 + DEF-139-01 切换条件 | C2 独立 feature store（deferred） | V3 物理归属、依赖、未来切换路径 |
| DQ-CP3-AGA5 | AGA-5 配置类事实源版本化机制 | E1 复用 V1 published pointer 语义 + 独立 config_facts 子目录 | E2 各类独立 registry | F1-F4 release 闭环机制、publish.py 复用 |

## 用户决策记录

> 待 AskUserQuestion 完成后回填。

## Live 用户决策记录（host-orchestrator 补充，2026-06-28T17:10:00+08:00）

> meta-se 委托期间 AskUserQuestion 工具不可用，AGA-1/3/5 标记 pending-cp3。host-orchestrator 回收后按规则 45 补 live 用户交互确认三项架构走向决策。用户全部采纳推荐方案，与 working basis 一致，无需按规则 36 回退刷新设计产物（precheck #16 PARTIAL → PASS）。

| 灰区 | 推荐方案 | 备选 | 用户回答 | 结果 |
|---|---|---|---|---|
| AGA-1 写侧/读侧分离边界 | A1 分层同属 FEAT-02 + HLD 分读写章 | A3 拆 FEAT-02R 读侧 | A1 分层同属 FEAT-02（推荐） | 采纳，不新增 Feature，CP5 读写冲突频繁时可升级 A3（回退条件已写 ADR） |
| AGA-3 ML feature 层归属 | C1 lake `features/` 子层带版本 + DEF-139-01 切换条件 | C2 独立 feature store（deferred） | C1 lake features/ 子层（推荐） | 采纳，符合 D5 附条件决策；feature 规模/在线 serving 触发时另起 CR 切换 C2 |
| AGA-5 配置类事实源版本化机制 | E1 复用 V1 published pointer 语义 + config_facts 子目录 | E2 各类独立 registry | E1 复用 V1 pointer 语义（推荐） | 采纳，机制统一复用 publish.py；policy_cycle 语义差异过大时该类独立，其余复用 |

**结论**：3 项架构走向决策全部采纳推荐方案，与 meta-se working basis 一致，0 项需回退刷新。CP3 precheck #16（架构灰区讨论）由 PARTIAL 收敛为 PASS。AGA-2/4/6 既定，AGA-1/3/5 经 live 确认，6 灰区全部 resolved。
