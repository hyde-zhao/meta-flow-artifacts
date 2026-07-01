---
title: "Quant Research Production Roadmap — Review v0.1"
status: "review"
reviewer: "host-orchestrator (review pass)"
created_at: "2026-07-01"
reviewed_doc: "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md"
review_version: "v0.1"
verdict: "approve-with-revisions"
verdict_summary: "范围与阶段方向合理且与项目依赖链一致；阶段目标存在量化不足与遗漏项；可行性整体合适但 NAS 多节点约束未落地到阶段。建议补强后作为 v0.2 baseline。"
---

# Roadmap 评审意见

本评审针对 `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` v0.1，从三个维度评审：**范围是否合理**、**阶段目标是否合理**、**可行性是否合适**。评审基于代码库实际资产核验，非纯文档推断。

## 0. 总体结论

| 维度 | 评价 | 一句话 |
|---|---|---|
| 范围 | ✅ 合理，有 2 项遗漏 | 覆盖数据/研究/回测/paper/live 全链路且依赖方向正确，但漏了 NAS 多节点治理和既有 CR 资产衔接 |
| 阶段目标 | ⚠️ 方向对，量化不足 | 多数 exit 条件是定性表述，违反规则 2「目标必须量化」；几项缺失门 |
| 可行性 | ⚠️ 整体合适，1 处硬卡点 | 与既有资产契合度高，但 NAS 多节点约束（用户已决策）未落进任何 Phase |

**verdict：approve-with-revisions**——不推翻，但需补强后作为 v0.2 baseline 才能据此起 CR-147。

---

## 1. 范围评审

### 1.1 范围合理之处

- **依赖方向正确（§3.2）**：Data Lake → Research Builder → Strategy → Backtest → Promotion → Live 的单向链路，与项目既有依赖表（CR-016 §依赖：CR-015 broker → CR-016 sim/live → CR-017 复权口径）一致。
- **禁止依赖表（§3.2）命中真实风险点**：「Data cleanup 自动选择 business-conflict 主记录」「Event 策略用 event_time 替代 available_at」「ML 直读非 PIT lake」三条都是本项目历史上踩过/防御过的坑（CR-139 S05 PIT reader、CR-139 S31 events schema repair 均有证据）。
- **不优先执行项明确（§4 Phase 1）**：N1 物理迁移、legacy delete/archive、provider/NAS sync 被显式后置，与我两轮分析一致——非交付阻断。
- **CR 切分克制（§5）**：不直接起「实盘平台 CR」，而是 CR-147 数据底座优先，符合规则 26「大块集中需求入口分流」。

### 1.2 范围遗漏（需补）

**G1（中）— NAS 多节点治理完全缺席**

用户已明确决策「多节点/灾备/团队共享」，但路线图仅在 §frontmatter 和 §121、§160、§431 把 NAS 作为「不授权项」提及，**没有任何 Phase 把多节点一致性作为目标**。这是真实缺口：

- 多节点读同一 NAS → catalog/pointer 多节点一致视图必须保证（published pointer 前移从可选变必须）
- 多节点不能并发写 → 需要明确 write-primary 单点 + 同步协议
- NAS 故障降级路径需演练

建议在 Phase 1 新增 `Multi-node consistency` 能力行，或在 Phase 4/5 补 `NAS consistency gate`。否则到了 Phase 5 readonly live 会发现多节点没有一致视图。

**G2（中）— 未衔接既有 CR 资产，存在重复造轮子风险**

路线图把 Feature Store / Label Store / Event Store / Experiment Registry / Strategy Artifact / Paper Simulation 当作「待建」，但代码库已有大量沉淀：

| 路线图对象 | 既有资产（已核验存在） |
|---|---|
| Feature Store | `market_data/features/artifacts.py`（FeatureArtifactSpec、LabelArtifactSpec、FeatureStoreSwitchPolicy） |
| Experiment registry | `engine/research_manifest.py`、`engine/training_snapshot_contract.py` |
| Strategy artifact | `engine/strategy_admission_package.py`、`engine/strategy_readiness_admission.py` |
| Paper simulation | `engine/paper_simulation.py` |
| Backtest / cost / portfolio | `engine/backtest.py`、`engine/mature_multifactor_framework.py` |
| Strategy runner / sim admission / kill switch | `trading/strategy_runner/`（simulation_admission、live_switch_deferred、simulation_operator 等 20+ 模块） |
| scripts/data_lake 入口 | `check_market_data_readiness.py`、`profile_duplicate_keys.py`、`publish_market_data_release.py` 等 9 个 |

路线图应明确这些是**收敛/闭环既有资产**而非新建（对应 handoff §1.6 的 d2 类）。否则 CR-147 会被当成「从零建 Feature Store」，实际应是「把 FeatureArtifactSpec 升级为正式 Feature Store contract」。

**G3（低）— deferred 能力 register 未引用**

CR-019 S09 已交付 `docs/CR019-DEFERRED-CAPABILITIES.md`（Backtrader/Qlib/minute/Level2 后置）。路线图 §4 多次提到 ML/事件驱动扩展，应显式声明「日频多因子 baseline 不触发 Qlib/minute/Level2」，引用该 register 作为边界，避免范围被撑大。

---

## 2. 阶段目标评审

### 2.1 方向合理之处

- **Phase 0→1 顺序正确**：先固化基线再治理，符合 REQ-247 基线门。
- **Phase 1 治理优先级合理**：`trade_status > prices_limit > prices > events > adj_factor` 的排序有据（business-conflict 量级 + 影响面）。
- **Phase 2→3→4 渐进**：先 PIT-safe 样本，再统一回测，再策略生产化，逻辑自洽。
- **Phase 5/6 实盘分档（L1-L5）合理**：与 CR-016 的 live_readonly/small_live/scale_up later-gated 结构吻合。

### 2.2 量化不足（违反规则 2）

规则 2 要求「成功标准必须含可度量值，禁止『不少于/尽可能/更完整』」。路线图多处违反：

| Phase | exit 条件原文 | 问题 |
|---|---|---|
| Phase 1 | 「核心 dataset 不再只有『可读』状态」 | 「核心」未定义（5 个还是 17 个？） |
| Phase 1 | 「有明确 production readiness」 | readiness matrix 覆盖率门槛未量化 |
| Phase 2 | 「可复现相同样本」 | 复现容差未量化（bit-exact？checksum 一致？） |
| Phase 3 | 「至少各有一个 baseline 回测」 | baseline 通过标准未量化 |
| Phase 5 | 「连续运行多个交易日」 | 「多个」无下限 |
| Phase 6 | 「完成一个评估窗口」 | 窗口长度、对账容忍未量化 |

建议每条 exit 补量化门槛，例如：
- Phase 1：「17/17 datasets 有 readiness matrix；trade_status/prices_limit/prices 三者 conflict policy 通过 human gate」
- Phase 5：「连续 ≥10 个交易日，live signal vs offline replay 差异可解释率 100%」
- Phase 6：「评估窗口 ≥20 交易日，每日对账 0 失败」

### 2.3 缺失门（需补）

**M1（中）— Phase 0 缺 fail-closed fallback 加固门**

Phase 0 把 CR-146 close 作为终点，但未包含我两轮指出的 `readers.py` rglob-all fallback 加固。这个潜伏正确性风险（catalog 回归时按 `source_run_id` 字典序选 survivor 而非时间最新）是 Phase 1 任何「动 catalog/写数据」操作的安全前提，必须在 Phase 0 或 Phase 1 早期完成。

**M2（中）— Phase 1 缺 published pointer 前移门**

用户决策多节点后，published pointer 前移（CR-139 S04，当前 not-authorized）从可选变必须。Phase 1 的「Run registry」只替代 source_run_id fallback，未提 pointer 前移。多节点一致视图依赖 published pointer，应在 Phase 1 补。

**M3（低）— Phase 2 缺 leakage gate 量化**

Phase 2 把 Leakage Gate 列为能力，但未定义 leakage 检测的量化标准（feature availability 偏移量、label window 重叠率阈值）。

---

## 3. 可行性评审

### 3.1 可行性合适之处

- **与既有资产契合度高**：FeatureArtifactSpec、ExperimentManifest、StrategyAdmissionPackage、paper_simulation、strategy_runner 全部已存在，CR-147 多数是收敛而非新建（见 G2 表）。
- **CR 切分粒度合理**：CR-147 数据底座优先、CR-148 回测引擎、CR-149 paper/readonly、CR-150 small-live gate，每块可独立 CP2-CP8 闭环，不强行塞一个巨型 CR。
- **授权边界清晰**：frontmatter `authorization_boundary` 显式排除 runtime/broker/provider/NAS/credential/live，符合 REQ-248。

### 3.2 可行性卡点

**C1（高）— NAS 多节点是硬卡点，未在阶段落地**

用户已决策多节点，但：
- Phase 1 「不优先执行 provider/NAS sync」
- Phase 5 readonly live 需要多节点一致视图，却无 NAS 一致性门
- Phase 6 small-live 需要灾备，却无灾备验证

**这是可行性最大风险**：路线图当前隐含「单机推进到 Phase 5 再处理 NAS」，但多节点下 Phase 1 的 published pointer、Phase 2 的 dataset snapshot 多节点可见性、Phase 5 的 live signal 多节点一致性都需要 NAS 提前就位。建议把 NAS 接入提前到 Phase 1 或 Phase 2，作为 Phase 5 的前置依赖，而非后置。

**C2（中）— Phase 1 conflict policy 与 4.27M 冲突的现实张力**

Phase 1 要求「4,272,624 business-conflict groups 有处理策略」，但 CR-146 CP2 已决定 DQ-002「不做语义择优，保持 quarantine」。两者一致（quarantine 算处理策略），但需明确：Phase 1 的产出是 **quarantine policy + dataset-level human gate 流程**，不是「解决 4.27M 冲突」。措辞应避免被误读为 Phase 1 要清零冲突。

**C3（低）— Phase 3 baseline 回测的 ML/事件策略依赖 Phase 2 未就绪**

Phase 3 exit「多因子/ML/事件策略至少各有一个 baseline」依赖 Phase 2 的 Feature/Label/Event Store。但 §7 近期建议又说「先多因子 baseline，再扩展 ML 和事件驱动」。需明确 Phase 3 exit 是否含 ML/事件，还是仅多因子——建议 Phase 3 exit 仅要求多因子 baseline，ML/事件延后到 Phase 3.5 或 Phase 4。

---

## 4. 与用户三个决策的对齐检查

| 用户决策 | 路线图对齐情况 |
|---|---|
| 暂定日频，保留日内 | ✅ 隐含日频（未提 minute/Level2），但未显式声明「日频为基线、日内为 Spike」 |
| 多节点/灾备/团队共享 | ❌ 完全未落地（见 G1/C1） |
| 先不锁策略，推数据底座到生产级 | ⚠️ CR-147 scope 含「baseline multi-factor backtest」，与「先不锁策略」轻微冲突——baseline 不是锁定策略，但措辞易误读。建议改为「以多因子 baseline 验证 research→backtest→registry 链路通」，明确 baseline 是链路验证而非策略上线 |

---

## 5. 修订建议（按优先级）

| 优先级 | 修订项 | 落点 |
|---|---|---|
| P0 | 补 NAS 多节点一致性能力到 Phase 1/2，作为 Phase 5 前置依赖 | §4 Phase 1 新增行 + Phase 5 前置条件 |
| P0 | Phase 0/1 补 fail-closed fallback 加固门 | §4 Phase 0 或 Phase 1 早期 |
| P0 | Phase 1 补 published pointer 前移门（多节点必需） | §4 Phase 1 |
| P1 | 全部 exit 条件补量化门槛 | §4 各 Phase |
| P1 | 补既有资产衔接表（FeatureArtifactSpec 等收敛非新建） | §5 CR-147 scope |
| P1 | 明确 Phase 1 冲突产出是 quarantine policy 而非清零 | §4 Phase 1 措辞 |
| P2 | 显式声明日频基线 + 引用 CR-019 deferred register 边界 | §1 目标 + §4 |
| P2 | 明确 Phase 3 exit 仅多因子 baseline，ML/事件延后 | §4 Phase 3 |
| P2 | 补 Gotchas 章节（规则 9，路线图类产出必须） | 新增 §8 |

---

## 6. 一句话结论

**路线图方向正确、依赖链自洽、CR 切分克制，作为 v0.1 draft 是合格的起点；但要据此起 CR-147，必须先补 NAS 多节点落地（P0）、量化全部 exit 条件（P1）、衔接既有资产避免重复造轮子（P1）。** 补强后作为 v0.2 baseline，CR-147 可据此进入 CP2。

不阻塞当前 Phase 0（CR-146 approve + fail-closed 加固）的推进——这两项在路线图补强期间可并行执行。
