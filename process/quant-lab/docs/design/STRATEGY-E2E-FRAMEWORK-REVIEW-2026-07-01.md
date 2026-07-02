---
title: "Strategy End-to-End Production Framework Review"
status: "review-v1.0"
owner: "host-orchestrator"
created_at: "2026-07-01"
scope: "UC-58 多因子 / UC-59 机器学习 / UC-60 事件驱动 端到端生产框架评审"
authorization_boundary: "analysis-only; review of proposed use-case framework, no implementation / runtime / lake / NAS / provider / trading / broker authorization"
reviews: "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md"
methodology: "对照 GitHub 量化实践(Microsoft Qlib、vnpy、QuantConnect Lean)与学术文献(López de Prado AFML、MacKinlay 事件研究、Bailey-López de Prado PBO/DSR、Harvey-Liu-Zhu 多重检验),经并行研究 agent 逐条核实引用后综合"
---

# 三类策略端到端生产框架评审报告

> 评审对象:UC-58 多因子、UC-59 机器学习、UC-60 事件驱动三类策略的端到端生产全流程(信息收集 → 研究 → 回测 → 准入 → 模拟盘 → 实盘 → 生命周期),以及三类策略共用的横切生产实践。
>
> 评审基准:Microsoft Qlib 流水线、WorldQuant 101 Alphas 生命周期、López de Prado《Advances in Financial Machine Learning》(AFML)、MacKinlay (1997) 事件研究方法、Bailey-López de Prado PBO/DSR、Harvey-Liu-Zhu 多重检验校正、Almgren-Chriss/Gatheral 市场冲击。
>
> 当前可实现边界:CR150 只实现多因子 metadata linkage completion;ML/事件驱动作为后续 CR 的用户场景和设计输入,不并入 CR150 实现范围。
>
> 当前不授权:真实数据湖读写、NAS sync、provider fetch、QMT/MiniQMT/xtquant runtime、模拟盘、实盘、broker write、凭据读取。

> 2026-07-02 后评审编号映射:本评审 §6 / §9 使用的是 2026-07-01 初始建议编号。实际落地中,多因子统计门已由 CR151 关闭为 `READY_WITH_RISK`;ML 框架基础已由 CR152 关闭为 `READY_WITH_RISK`;事件驱动基础顺延为下一推荐 CR153。本文方法论缺口判断保留,但后续实施计划以 roadmap v0.8 和 remediation plan v0.4 的 CR 编号与状态为准。

## 0. 执行摘要

### 0.1 总判断

框架在**工程治理层**达到了生产级,但在**统计/方法论层**存在系统性共性缺口。三类策略共用同一条生产主线(信息收集 → 研究 → 回测 → 准入 → 模拟盘 → 实盘 → 生命周期),这条主线的分级门控、PIT/leakage 防护、可复现哈希链、kill switch/reconciliation 做得扎实,显著优于 Qlib 文档级实现。但三类策略在**样本外验证、多重检验校正、回测过拟合检测、市场冲击建模**这四个横切维度上全部偏弱——而这正是"研究编排框架"与"生产级量化框架"的分水岭。

### 0.2 统一口径判定矩阵

| 维度 | UC-58 多因子 | UC-59 ML | UC-60 事件驱动 |
|---|---|---|---|
| (1) 流程正确性 | **中** — 分级门控强,统计验证门弱 | **中** — 排序对,缺 AFML 强制阶段 | **中** — 三时间分离优秀,事件方法论缺 |
| (2) 步骤完备性 | **中** — 生命周期全,统计项缺 | **中** — 4/10 步仅规范未实现 | **弱** — 估计窗/检验族/冲击全缺 |
| (3) 资料/工件完备性 | **中** — 哈希强,统计工件缺 | **混合** — 部分 .py 不存在 | **中** — 架构好,缺量化工件 |
| (4) 输出质量可靠性 | **弱** — 哈希可复现≠统计可靠 | **弱** — 高收益无过拟合门拦截 | **弱** — 成本低估致收益虚高 |

> **最该警惕的点**:四维度里**输出质量可靠性三类全弱**。框架看起来严谨(有 hash/manifest/blocked claims),但 Sharpe/IC 未经多重检验校正、未经 PBO/DSR、未含冲击成本,作为准入门不可信——"看似严谨(有哈希/manifest)实则统计不可信(Sharpe/IC 未防偷窥校正)"。

### 0.3 横切共性 Blocker(三类策略共享,优先级最高)

以下四个 blocker 在 UC-58/59/60 中**反复独立出现**,说明是主线的系统性缺口而非单个用例的疏漏:

| 共性 Blocker | 表现 | 为何致命 | 最佳实践 |
|---|---|---|---|
| **C1 多重检验/数据偷窥门缺失** | UC-58 步骤 7、UC-60 步骤 7、UC-59 步骤 6-7 均无 FDR(BH)/White Reality Check/Hansen SPA/PBO/DSR | 数百候选因子在 α=0.05 下即使全无效也预期约 5% 假阳性通过;无校正则噪声因子进入组合,OOS 失效。Harvey-Liu-Zhu 建议新因子 t≥3.0 | Benjamini-Hochberg FDR;Bailey-López de Prado PBO/CSCV(纯噪声 PBO≈74%);DSR |
| **C2 walk-forward / OOS 非一等阶段** | 三类策略的回测步骤均为单全样本,无 rolling/walk-forward,ML 缺 purged k-fold+embargo | 单历史期回测无法区分预测力与 curve-fit;标准 k-fold 因标签窗跨 fold 边界泄漏 | walk-forward optimization;AFML Ch.7 purged CV with embargo |
| **C3 市场冲击模型仅 commission/tax** | UC-58 步骤 5/6、UC-60 步骤 6 均列"成本/滑点/税"但无 square-root / Almgren-Chriss 模型 | 机构订单 >1% ADV 时冲击主导(比佣金高一个数量级),AUM 放大后净 alpha 被高估,实盘显著跑输回测 | Almgren-Chriss 最优执行;Gatheral square-root(注意:Almgren 2005 实测为 3/5 幂律,非 β=1/2) |
| **C4 容量/流动性感知 sizing 缺失** | 三类均无 ADV 参与率上限、容量美元上限、fill 概率 | 容量超阈值后 alpha 衰减;纸面 Sharpe 不可实现 | Bucci et al. 2019;Kissell |

---

## 1. UC-58 多因子策略端到端框架评审

评审基准:Microsoft Qlib 流水线、WorldQuant 101 Alphas 生命周期、López de Prado 多重检验方法、Hou-Xue-Zhang/Jensen-Kelly 因子复现危机文献。

### 1.1 四维度判定表

| 维度 | 判定 | 证据(UC-58 涉及/未涉及) | 来源 |
|---|---|---|---|
| **(1) 流程正确性** | **adequate**(分级门控强,统计验证门弱) | 做对:步骤 7→8→10→11 显式 research→admission→paper→live 分级,人工授权 + kill switch + reconciliation(步骤 8/10/11);"不把研究结果自动解释为可交易策略"排除项;CR150 边界隔离真实 IO/live。**缺**:步骤 4(评价)与步骤 7(准入)之间无显式多重检验/数据偷窥统计门——FDR/BH、WRC/SPA、PBO/CSCV、DSR 全部缺失;步骤 6 无 walk-forward/rolling OOS 协议,单全样本回测不足以判定准入 | Qlib 工作流仅有 Data→Model→Evaluation(IC+PortAna),亦无 admission/paper/live 分层,故 UC-58 在分级上**优于** Qlib;但 Qlib 文档亦未含多重检验,UC-58 未补此工业短板 |
| **(2) 步骤完备性** | **adequate**(生命周期覆盖强,步骤内统计/经济项缺失多) | 做对:覆盖 1-12 全生命周期,含步骤 12 复盘/漂移/退役;步骤 2/3 显式 available_at、PIT/leakage gate、FactorSpec lineage。**缺**:步骤 4 因子评价仅列 IC/RankIC、覆盖率、缺失率、稳定性、权重、中性化/标准化——缺 IC 衰减(随 lag)、半衰期、换手、容量/流动性、正交化(对已知风险因子)、单调性、分位差、Newey-West 调整 t 统计;步骤 3 有 PIT gate 但未显式要求**退市/ST/被并购股票 PIT universe**;步骤 6 无 regime 分层回测;步骤 6 有 benchmark 但无 IR/TE/Active Share 三件套;步骤 5/6 有 cost 参数但无市场冲击模型(square-root/Almgren-Chriss)与容量上限 | 101 Alphas 评估含持有期 0.6-6.4 天、pairwise corr 15.9%(换手/相关性为标配);生存偏差年化虚高约 0.9%(Elton-Gruber-Blake);walk-forward 为策略验证 gold standard |
| **(3) 资料/工件完备性** | **adequate**(可复现哈希强,统计验证工件缺) | 做对:步骤 2 config hash/factor run refs/lineage、步骤 6 BacktestRunSpec hash、步骤 7 ExperimentManifest/StrategyAdmissionPackage、步骤 3/7 blocked claims/forbidden counters(显式拒绝审计)。**缺**:步骤 7 manifest 无"多重检验工件"(FDR 校正 p 值、PBO 评分、DSR 值、有效独立试验数 N);无 regime 归因报告/walk-forward 报告工件;无容量/冲击报告工件(ADV 参与率、容量美元上限);无 universe PIT 审计工件(退市股纳入日志) | Qlib 输出含年化收益/IR/maxDD 且区分 with/without cost 两档;DSR 需报告 SR₀(期望最大 Sharpe)、N、偏度峰度 |
| **(4) 输出质量可靠性** | **weak**(哈希可复现≠统计可靠) | 做对:config/run hash 保证确定性复现;步骤 7 pass/fail/needs_review 显式决策输出;步骤 9/10/11 reconciliation 活 vs 离线核对。**致命缺**:无 DSR/FDR/PBO,报告的 Sharpe/IC 经多重试验膨胀而未校正→作为准入标准不可信;无 survivorship-free universe,权益曲线系统性上偏;无容量/冲击,净 alpha 在 AUM 放大后被高估;无 Newey-West,IC 显著性误判 | DSR 三重校正(选择偏差+非正态+样本长度);False Strategy Theorem 证明 N 次试验下真实 SR=0 仍观测到显著正 Sharpe;100 个独立检验 α=0.05 全真零假设时至少一个假阳性概率≈99.4% |

### 1.2 优先级 GAPS(多因子)

| # | Gap | 严重度 | 建议放入 |
|---|---|---|---|
| MF-GAP-1 | 无多重检验/数据偷窥统计门(步骤 4/7 之间无 FDR/BH、WRC/SPA、PBO/CSCV、DSR 任何一项作为准入硬指标) | **blocker** | 步骤 4(FDR/BH + Newey-West t 阈值)+ 步骤 7(PBO/CSCV + DSR 作为 pass/fail 硬门) |
| MF-GAP-2 | 因子评价统计项不完备(缺 IC 衰减、半衰期、换手、容量/流动性、正交化、单调性、分位差、Newey-West t) | **blocker** | 步骤 4(扩充评价项)+ 步骤 6(换手/容量作为回测约束) |
| MF-GAP-3 | 无 survivorship-free PIT universe 显式要求(未要求历史 universe 含未来退市/ST/被并购股票及指数再平衡 PIT 成员) | **major**(小盘/微盘策略为 blocker) | 步骤 3(universe PIT 构建硬要求 + 退市股纳入日志工件) |
| MF-GAP-4 | 无 regime-aware / walk-forward 稳健性(步骤 6 无 regime 分层与 walk-forward/rolling OOS) | **major** | 步骤 6(regime 分层 + walk-forward 子步)+ 步骤 7(regime 归因报告工件) |
| MF-GAP-5 | 无市场冲击模型 / 容量上限(仅有 cost 参数,无 square-root / Almgren-Chriss,无 ADV 参与率与容量美元上限) | **major**(高换手策略为 blocker) | 步骤 5(容量上限/冲击模型入约束)+ 步骤 6(回测含冲击,输出容量报告工件) |
| MF-GAP-6 | 无相对基准指标三件套(IR / TE / Active Share) | **major** | 步骤 6(回测指标集)+ 步骤 7(report pack) |
| MF-GAP-7 | 无因子相关性聚类/去重(权重策略未要求因子间相关性聚类与冗余去重,101 Alphas 报告 pairwise corr 15.9% 为行业惯例) | **major** | 步骤 4(组合信号构建子步) |
| MF-GAP-8 | 无 universe PIT 审计工件(步骤 3/7 无显式"退市股票纳入日志"工件供审查,PIT gate 是否真正执行不可验证) | **minor** | 步骤 3(工件)+ 步骤 7(manifest) |

### 1.3 框架做对的地方(多因子)

1. **研究-生产分离原则显式**:排除项"不把研究结果自动解释为可交易策略"是工业级正确——多数框架失败于此反模式。Qlib 文档亦未显式此分离,UC-58 在此**优于** Qlib。
2. **全生命周期覆盖含步骤 12 复盘/漂移/退役**:多数框架止于回测;UC-58 含 drift/decay、risk breach、PnL attribution、continue/pause/rollback 决策,符合生产运维现实。
3. **PIT / available_at / leakage gate 显式(步骤 2/3)**:available_at 时间戳严格 < 决策时点是防 lookahead 的工程硬要求。Qlib 直至 2022-03(PR #343)才发布 PIT database,UC-58 将 PIT 作为契约级 gate 更严格。
4. **可复现哈希链**:步骤 2 config hash、步骤 6 BacktestRunSpec hash、factor run refs/lineage 形成端到端可复现链,超越 Qlib 的隐式可复现。
5. **分级门控 + 人工授权 + kill switch + reconciliation(步骤 8/10/11)**:per-run authorization(最小权限)、readonly/paper gate、kill switch drill、live-vs-offline reconciliation——符合实盘上线工业门控。
6. **禁止/阻断声明显式审计(步骤 1/3/7)**:不可用/阻断声明、blocked claims、forbidden counters 形成拒绝审计轨迹,优于"静默通过"。
7. **CR150 边界隔离真实 IO/live**:将真实 factor panel 构建/数据湖读写/paper/simulation/live 排除出 CR150,防止研究态污染生产态。
8. **成本/滑点/税/benchmark 在回测显式(步骤 6)**:对照 Qlib 亦有 with/without cost 双档,UC-58 在契约层显式声明。
9. **pass/fail/needs_review 三态准入(步骤 7)**:优于二值 pass/fail,留出人工复核窗口。
10. **Qlib/Backtrader/分钟/L2 不作主路径默认**:正确避免将研究玩具当作生产主路径。

### 1.4 多因子引用清单

1. Qlib 工作流文档 — https://qlib.readthedocs.io/en/latest/component/workflow.html
2. Qlib GitHub(PIT database PR #343、with/without cost 双档) — https://github.com/microsoft/qlib
3. Qlib 论文(arXiv:2009.11189) — https://arxiv.org/abs/2009.11189
4. 101 Formulaic Alphas(arXiv:1601.00991,持有期/pairwise corr) — https://arxiv.org/abs/1601.00991
5. False Discovery Rate(Benjamini-Hochberg) — https://en.wikipedia.org/wiki/False_discovery_rate
6. Data dredging / 数据偷窥总览 — https://en.wikipedia.org/wiki/Data_dredging
7. Multiple comparisons problem — https://en.wikipedia.org/wiki/Multiple_comparisons_problem
8. PBO/CSCV(López de Prado) — https://en.wikipedia.org/wiki/Marcos_L%C3%B3pez_de_Prado
9. Deflated Sharpe Ratio — https://en.wikipedia.org/wiki/Deflated_Sharpe_ratio
10. DSR 公式详解(ThinkNewFound,False Strategy Theorem/MinTRL) — https://blog.thinknewfound.com/2019/11/the-deflated-sharpe-ratio/
11. Hansen SPA 原始论文 — https://users.ssc.wisc.edu/~bhansen/papers/spa.pdf
12. Survivorship bias — https://en.wikipedia.org/wiki/Survivorship_bias
13. Market impact(square-root 模型) — https://en.wikipedia.org/wiki/Market_impact
14. Algorithmic trading(Almgren-Chriss 最优执行) — https://en.wikipedia.org/wiki/Algorithmic_trading
15. Information ratio — https://en.wikipedia.org/wiki/Information_ratio
16. Tracking error — https://en.wikipedia.org/wiki/Tracking_error
17. Walk-forward optimization — https://en.wikipedia.org/wiki/Walk_forward_optimization
18. Active Share(Cremers-Petajisto, JF 2009) — https://onlinelibrary.wiley.com/doi/10.1111/j.1540-6261.2009.01507.x
19. Harvey-Liu-Zhu "...and the Cross-Section of Expected Returns"(RFS 2016,t≥3.0) — https://academic.oup.com/rfs/article-abstract/29/1/5/1594347

---

## 2. UC-59 机器学习策略端到端框架评审

评审基准:López de Prado《Advances in Financial Machine Learning》(AFML)及其后续、"The 10 Reasons Most Machine Learning Funds Fail"、Bailey-López de Prado PBO/DSR。

### 2.1 四维度判定表

| 维度 | 判定 | 关键证据 | 引用 |
|---|---|---|---|
| **(1) 流程正确性** | **部分通过** — 流水线排序(研究→准入→模拟→实盘→漂移)方向正确,但缺失 AFML 强制的 6 个核心阶段:purge+embargo CV、triple-barrier 标记、meta-labeling、fractional differentiation、特征重要性(MDI/MDA)、样本唯一性加权。leakage gate 仅在 spec/panel 层校验 available_at 与 label-overlap,未实现 purge+embargo 切分机制本身 | AFML Ch.3/5/7/8/10/12;10 Reasons #4/#5/#6/#7/#8 | |
| **(2) 步骤完备性** | **部分通过** — 10 步结构完整定义于路线图,但 4 步(训练/模拟/实盘/漂移重训)明确推迟未实现;已实现的步骤内部缺失 AFML 子阶段:回测无 PBO/DSR/CSCV、模型训练无特征重要性产物、漂移无 PSI/KS/KL 计算 | PBO/CSCV;DSR;Evidently drift | |
| **(3) 资料/工件完备性** | **混合** — 已实现 spec(FeatureArtifactSpec/LabelArtifactSpec/TrainingSnapshotSpec/SignalSet/ModelArtifactRef/StrategyAdmissionPackage)均为 frozen dataclass 且字段完整;但 CR-147/CR-148 声称新增的 `engine/research_production_contracts.py`(ResearchDatasetSpec/LeakagePolicy)和 `engine/backtest_production_contracts.py`(BacktestRunSpec)**.py 文件在仓库中不存在**,其字段仅存于证据 JSON 样本。缺失:特征重要性产物、实验试验计数 manifest(DSR 必需输入)、drift_report 类 | DVC+MLflow 四元组 | |
| **(4) 输出质量可靠性** | **部分通过** — fail-closed 准入(StrategyAdmissionPackage 默认 blocked)、snapshot 冻结(published_only/mutable_ref_forbidden/cutoff<=as_of)、signal-conversion 分离(SignalSet `not_order=True`)均可靠;但因无 PBO/DSR/CSCV 门禁,高回测收益仍可通过 Research Gate;漂移监控未实现意味着实盘后无量化衰减检测 | AFML 第一/第三定律;PBO 拒绝阈值 0.05 | |

> **工程硬伤(与 CR150 评审呼应)**:ResearchDatasetSpec、BacktestRunSpec 两个核心 .py 文件在 `/home/hyde/workspace` 下任何位置均不存在,字段仅存于 evidence JSON 样本(`evidence/CR147-...json`、`evidence/CR148-...json`)。这意味着契约无编译时校验、无运行时验证、无回归测试——任何人修改字段名都不会被捕获。CR150 真正该补的是"让已存在契约落到可执行 .py + linkage",而非重做。

### 2.2 优先级 GAPS(ML)

| # | Gap | 严重度 | 建议放入 |
|---|---|---|---|
| ML-GAP-1 | 无 purge+embargo 交叉验证机制实现(leakage gate 仅 panel/spec 层前视防护,不是 AFML 的 purged k-fold CV with embargo;TrainingSnapshotSpec.split_policy_id 仅引用字段无实现) | **严重** | 步骤 3(TrainingSnapshot 内嵌 split policy + purge_log/embargo_log) |
| ML-GAP-2 | 无回测过拟合检测(PBO / CSCV / DSR),步骤 7 准入八门禁中无 Overfitting Gate,"不把高回测收益自动解释为可上线模型"仅为文字约束无量化强制 | **严重** | 步骤 6(PBO/DSR 作 report pack 字段)+ 步骤 7(Overfitting Gate:PBO≤0.05 且 DSR≥0.95)+ 步骤 4(experiment manifest 记录 trial_count N) |
| ML-GAP-3 | 无 triple-barrier 标记与 meta-labeling(LabelArtifactSpec 仅固定时间窗口前向收益标记,无路径依赖三重障碍与二级下注模型) | **严重** | 步骤 2(label_method 支持 triple_barrier)+ 步骤 4(meta-labeling 双模型配置) |
| ML-GAP-4 | ResearchDatasetSpec / BacktestRunSpec 核心 .py 文件不存在(字段仅存于证据 JSON 样本,无 frozen dataclass/验证函数/测试) | **严重** | 步骤 2/6(补齐 .py 源码与验证函数,与 FeatureArtifactSpec 同等 rigor) |
| ML-GAP-5 | 无特征重要性分析产物(MDI/MDA/SHAP),AFML 第一定律"回测不是研究工具,特征重要性才是" | **高** | 步骤 4(feature_importance.json)+ 步骤 7(Robustness Gate 增特征重要性稳定性检查) |
| ML-GAP-6 | 无样本唯一性加权(金融标签时间重叠违反 IID 假设,无加权则对最频繁重叠市场状态过拟合) | **高** | 步骤 3/4(sample_weights + weighting_method) |
| ML-GAP-7 | 漂移监控(步骤 10)完全未实现(无 DriftReport 类、无 PSI/KS/KL、无重训练触发逻辑;源码中 drift 命中均为持仓对账漂移非模型漂移) | **高** | 步骤 10(DriftReport + retrain_threshold) |
| ML-GAP-8 | 无 fractional differentiation 特征工程(整数差分破坏记忆,原始价格非平稳;d∈(0,1) 平稳性最大化记忆) | **高** | 步骤 2(stationarity_method: fracdiff + fracdiff_d + ADF 检验) |
| ML-GAP-9 | 无分类标签类不平衡处理(长期偏多市场致正/负标签不平衡,朴素 0.5 阈值几乎从不是最优) | **中** | 步骤 4(class_balance)+ 步骤 5(threshold CV 调优) |
| ML-GAP-10 | 无实验试验计数与多重检验校正(experiment manifest 无 trial_count,DSR 的 N 输入缺失) | **中** | 步骤 4(强制 trial_count + 候选策略逐期收益矩阵) |
| ML-GAP-11 | 无 MinBTL / 最小回测长度门禁(T_min≈2 ln N/SR̂²,低 Sharpe 策略需极长样本) | **中** | 步骤 7(MinBTL gate) |
| ML-GAP-12 | 无特征版本与代码版本的四元组强绑定(现有 hash 是文件级而非 DVC 式 commit-data 耦合) | **中** | 步骤 3(dvc_tag / mlflow_run_id / feature_store_version 四元组指纹) |

### 2.3 框架做对的地方(ML)

1. **Leakage gate 真实存在且 fail-closed** — `label_cutoff_gate` 强制 `leakage_count==0` 且 `cutoff_violation_count==0` 否则 STATUS_BLOCKED;`factor_panel_contracts.py` 校验 `available_at > decision_time` 与 label window overlap。对应 AFML 失败原因 #8 的 spec 层防护。
2. **训练快照冻结严格** — TrainingSnapshotSpec 为 frozen dataclass,验证函数禁止 mutable ref(`"latest"/"current"` → `mutable_snapshot_ref_forbidden`)、强制 `published_only=True`、校验 `training_cutoff <= as_of`。对应 AFML 数据不可变性与 DVC 式版本冻结。
3. **Signal-conversion 分离** — SignalSet 强制 `not_order=True` + `not_runtime_authorization=True`;路线图 Phase 3 明确 SignalStrategy → PortfolioPolicy → ExecutionPolicy 三级分离。直接满足排除条款"不允许模型绕过 signal/portfolio/OMS/risk gate 直接下单"。
4. **Fail-closed 准入** — StrategyAdmissionPackage 默认 blocked,在任何 blocker severity 原因时返回 BLOCKED;FORBIDDEN_OPERATION_COUNTERS 含 13 个计数器(含 provider_fetch/lake_write/qmt_operation/real_order/credential_read),非零即 blocked。8 门禁覆盖面广。
5. **多版本 hash 绑定** — TrainingSnapshotSpec 含 content_hash/feature_schema_hash/lineage_refs;ModelArtifactRef 含 artifact_hash/dataset_snapshot_hash/feature_artifact_refs;虽非 DVC 式但已实现可追溯 hash 链。
6. **实盘分层 L1→L5** — readonly live → paper/shadow → small-live → controlled-live → production-live,前置条件"Phase 1-5 全部通过 + 独立 runtime authorization + kill switch 演练通过"。对应 AFML 策略生命周期(embargo→paper→graduate→reallocate→retire)。
7. **禁止依赖明确** — "ML training 直接读非 PIT lake 文件 | 易引入 lookahead/leakage"被列为禁止依赖;废除 `--data-dir/load_local_frames` 旁路,ML 改用 read_panel_as_of。从数据接入层杜绝旁路泄漏。

### 2.4 ML 引用清单

1. López de Prado, *Advances in Financial Machine Learning* (Wiley, 2018) — https://www.wiley.com/en-us/Advances+in+Financial+Machine+Learning-p-9781119482086
2. López de Prado, "The 10 Reasons Most Machine Learning Funds Fail" (JPM 44(6), 2018; GARP) — https://www.garp.org/hubfs/Whitepapers/a1Z1W0000054x6lUAA.pdf
3. Bailey, Borwein, López de Prado, Zhu, "The Probability of Backtest Overfitting" (J. Computational Finance 20(4), 2017) — https://www.davidhbailey.com/dhbpapers/backtest-prob.pdf
4. Bailey & López de Prado, "The Deflated Sharpe Ratio" (JPM 40(5), 2014) — https://www.davidhbailey.com/dhbpapers/deflated-sharpe.pdf
5. Wikipedia, "Purged cross-validation" — https://en.wikipedia.org/wiki/Purged_cross-validation
6. Hudson Thames, "An Introduction to the Triple Barrier Method" — https://hudsonthames.org/an-introduction-to-the-triple-barrier-method/
7. Hudson Thames, "An Introduction to Meta-Labeling" — https://hudsonthames.com/an-introduction-to-meta-labeling/
8. Hudson Thames, "ML Trading Essentials Part 2"(fractional differentiation、uniqueness 加权) — https://hudsonthames.org/machine-learning-trading-essentials-part-2-fractionally-differentiated-features-filtering-and-labelling/
9. skfolio, `CombinatorialPurgedCV` — https://skfolio.org/generated/skfolio.model_selection.CombinatorialPurgedCV.html
10. Evidently AI, "Data Drift Detection on Large Datasets"(PSI/KS/KL/Jensen-Shannon/Wasserstein 阈值) — https://www.evidentlyai.com/blog/data-drift-detection-large-datasets
11. Evidently AI, "Machine Learning Monitoring: Data and Concept Drift" — https://www.evidentlyai.com/blog/machine-learning-monitoring-data-and-concept-drift
12. Lundberg & Lee, "A Unified Approach to Interpreting Model Predictions" (NIPS 2017, SHAP) — https://arxiv.org/abs/1705.07874
13. scikit-learn, "Permutation Importance" — https://scikit-learn.org/stable/modules/permutation_importance.html
14. scikit-learn, "Common pitfalls and recommendations" — https://scikit-learn.org/stable/common_pitfalls.html
15. DVC, "Get Started" — https://doc.dvc.org/start
16. MLflow, "Model Registry" — https://mlflow.org/docs/latest/model-registry/index.html
17. Qlib, "Workflow" — https://qlib.readthedocs.io/en/latest/component/workflow.html
18. Bailey & López de Prado, "The Sharpe Ratio Efficient Frontier" (J. Risk 15(2), 2012, PSR/MinTRL) — https://www.davidhbailey.com/dhbpapers/sharpe-frontier.pdf

---

## 3. UC-60 事件驱动策略端到端框架评审

评审基准:MacKinlay (1997) 事件研究方法、Brown-Warner、Lyon-Barber-Tsai 长窗方法、事件内生性/自选择文献、PIT 修订偏差(Kothari-Shanken-Sloan)。

### 3.1 四维度判定表

| 维度 | 判定 | 证据 | 引用 |
|---|---|---|---|
| **(1) 流程正确性** | **8/10 合格有重大改进空间** | event_time/available_at/decision_time 三时间分离精确对应 MacKinlay 事件窗 + "decision time vs information available"前视防护,available_at 审计直接阻断 PIT look-ahead;但事件内生性/自选择、聚类方差、PIT 修订 gate 三处方法论缺位 | MacKinlay (1997) JEL 35(1):13-39;Kothari-Shanken-Sloan (1995) JF 50(1):185-224 |
| **(2) 步骤完备性** | **6/10 有关键缺失** | 缺估计窗长度/基准模型选择、BHAR vs CAR 选择与检验族、市场冲击模型族(commission/tax 之外)、walk-forward 一等阶段、容量 gate、归因方法、聚类型方差调整 | Brown-Warner (1985) JFE 14(1);López de Prado (2018) JPM 44(6):120-133 |
| **(3) 资料完备性** | **7/10** | 有 EventResearchBrief/quality report/leakage gate/admission result/blocked claims 架构良好;缺多重检验 p 值校正报告、容量曲线、归因报告、对账 break 报告、机制(regime)分解报告 | Lyon-Barber-Tsai (1999) JF 54(1):165-201;Bailey-López de Prado DSR (2014) JPM 40(5):94-107 |
| **(4) 输出质量可靠性** | **6/10** | event-to-order trace + audit + 不可用/延迟声明优秀;但成本模型低估(commission/tax only)致回测收益不可靠,准入门缺 DSR/PBO/White-Hansen 多重检验校正,"失效/拥挤风险"无量化定义 | Perold (1988) JPM 14(3):4-9;White (2000) Econometrica 68(5):1097-1126;PBO Bailey et al. (2017) JCF 20(4):39-69 |

### 3.2 优先级 GAPS(事件驱动)

| # | Gap | 严重度 | 建议放入 |
|---|---|---|---|
| EV-GAP-1 | 估计窗 + 正常收益基准模型未显式选择(市场模型/FF3/FF5/Carhart/DGTW);估计窗长度(-200 至 -11 惯例)与基准模型选择直接决定 AR/CAR 数值与显著性 | major | 步骤 4(事件研究) |
| EV-GAP-2 | 显著性检验族未指定(BMP 事件引起方差调整/Patell/generalized sign/rank/bootstrap);事件日常伴随方差跳升,普通 t/Patell 过度拒绝,长窗 BHAR 严重右偏需偏度调整 | major | 步骤 4 |
| EV-GAP-3 | 事件内生性/自选择偏差无修正机制(Heckman/PSM/IV/匹配基准);公司自选择进入事件(回购/SEO/股利),条件抽样使"异常收益"混淆事件效应与选择效应 | major | 步骤 4/7 |
| EV-GAP-4 | 重叠/聚类事件窗无方差调整(聚类标准误 / 组合时间序列法);同日/同周事件残差横截面相关,标准统计量低估 SE | major | 步骤 4 |
| EV-GAP-5 | 长周期收益默认 BHAR 有偏,未规定 CAR/calendar-time 选择与再平衡偏差校正 | major | 步骤 4 |
| EV-GAP-6 | PIT/事件修订 look-ahead gate 不完整(CRSP/Compustat 当前快照 vs 当期);数据库事后反映重述/重分类,使用当前快照引入前视偏差 | **blocker** | 步骤 2(治理)+ 6(回测) |
| EV-GAP-7 | 多重检验/数据窥探无校正(多事件窗尝试只报显著者);需 White Reality Check/Hansen SPA/Romano-Wolf/Bonferroni/FDR | **blocker** | 步骤 7(准入) |
| EV-GAP-8 | Walk-forward/OOS/Purged-embargoed CV 非一等阶段;标准 k-fold 时序因标签窗重叠测试集而泄漏 | **blocker** | 步骤 6/7 |
| EV-GAP-9 | 幸存者偏差/股票池重构在回测无显式 gate(事件池 + 退市 + 指数再平衡);退市标的从历史样本消失,存活样本收益系统性偏高 | major | 步骤 3/6 |
| EV-GAP-10 | 成本模型仅 commission/tax,缺市场冲击/滑点族(square-root/3-5 幂律;线性永久 + 非线性瞬时);机构订单 >1% ADV 时冲击主导 | **blocker** | 步骤 6 |
| EV-GAP-11 | 容量/流动性感知 sizing 无显式 gate(ADV 上限/容量衰减/fill 概率/队列位);AUM 上升冲击放大、alpha 衰减 | major | 步骤 5/6 |
| EV-GAP-12 | Kill switch/fat-finger/pre-trade risk 阈值族不完整(频次/规模/notional/价格偏离/持仓上限 + 双人重启);仅 decision_time 隔离不足以防护故障 | major | 步骤 9 |
| EV-GAP-13 | 对账模式仅"post-trade reconciliation"未展开(position/cash/live-vs-offline/intraday-vs-daily/break 管理) | major | 步骤 9 |
| EV-GAP-14 | 业绩归因方法未指定(Brinson-Fachler vs 因子归因;多期链接);无归因则"alpha 来源"不可解构 | minor | 步骤 10(复盘) |
| EV-GAP-15 | regime 分析未显式(牛/熊/波动率/流动性 regime 条件化策略表现);事件 alpha 常具 regime 依赖 | minor-major | 步骤 6/10 |
| EV-GAP-16 | 相对基准指标未显式(vs 指数/同业/事件基准组合;信息比率/跟踪误差) | minor | 步骤 6/10 |
| EV-GAP-17 | event-to-order trace 未指定状态机/合规报告(FINRA CAT/MiFID RTS-22 全生命周期);仅"trace"不足以满足监管重建要求 | major | 步骤 9 |
| EV-GAP-18 | 准入门"失效/拥挤风险"无量化定义(拥挤度指标/alpha 衰减检测/McLean-Pontiff 样本外衰减) | major | 步骤 7 |

### 3.3 框架做对的地方(事件驱动)

- **event_time / available_at / decision_time 三时间分离是核心优秀设计**。它精确对应 MacKinlay 事件窗方法论与"information available at decision time"前视防护——多数回测框架缺失的一等概念。available_at 审计直接阻断 PIT look-ahead(Kothari-Shanken-Sloan 1995 所述 Compustat 前视偏差),decision_time 驱动回测与 signal/execution 分离,是 anti-leakage 骨架。
- 10 步链路(假设→数据治理→特征→研究→信号→回测→准入→模拟盘→实盘→复盘)+ 准入闸门(步骤 7)+ 模拟盘(步骤 8)前置,符合"回测不是研究"(López de Prado)+ paper-trading 验证的生产最佳实践。
- leakage gate(步骤 3)+ decision_time 回测(步骤 6)+ event-to-order trace(步骤 9)+ 不可用/延迟声明(步骤 2)构成强 anti-leakage/可追溯骨架。
- "事件监听器不直接下单"的排除约束符合 signal vs execution 分离 + pre-trade risk gate 生产最佳实践(Lean/vnpy OMS 模式)。
- "blocked event claims + source reliability 准入"对事件修订/PIT/数据质量有显式防护。

### 3.4 事件驱动引用清单

1. MacKinlay, A.C. (1997). "Event Studies in Economics and Finance." *J. Economic Literature* 35(1):13-39. https://www.jstor.org/stable/2729691
2. Brown, S.J. & Warner, J.B. (1985). "Using daily stock returns: The case of event studies." *JFE* 14(1):3-31. https://doi.org/10.1016/0304-405X(85)90042-X
3. Boehmer, E.; Masumeci, J.; Poulsen, A.B. (1991). "Event-study methodology under conditions of event-induced variance." *JFE* 30(2):253-272. https://doi.org/10.1016/0304-405X(91)90032-F
4. Lyon, J.D.; Barber, B.M.; Tsai, C. (1999). "Improved Methods for Tests of Long-Run Abnormal Stock Returns." *J. Finance* 54(1):165-201. https://doi.org/10.1111/0022-1082.00101
5. Li, K. & Prabhala, N.R. (2007). "Self-Selection Models in Corporate Finance." *Handbook of Corporate Finance* ch.2. https://doi.org/10.1016/b978-0-444-53265-7.50016-0
6. Thompson, S.B. (2011). "Simple formulas for standard errors that cluster by both firm and time." *JFE* 99(1):1-10. https://doi.org/10.1016/j.jfineco.2010.08.016
7. Kothari, S.P.; Shanken, J.; Sloan, R.G. (1995). "Another Look at the Cross-section of Expected Stock Returns." *JF* 50(1):185-224. https://doi.org/10.1111/j.1540-6261.1995.tb05171.x
8. White, H. (2000). "A Reality Check for Data Snooping." *Econometrica* 68(5):1097-1126. https://doi.org/10.1111/1468-0262.00152
9. Hansen, P.R. (2005). "A Test for Superior Predictive Ability." *JBES* 23(4):365-380. https://doi.org/10.1198/073500105000000063
10. Bailey, D.H. & López de Prado, M. (2014). "The Deflated Sharpe Ratio." *J. Portfolio Management* 40(5):94-107. https://doi.org/10.3905/jpm.2014.40.5.094
11. Bailey et al. (2017). "The Probability of Backtest Overfitting." *J. Computational Finance* 20(4):39-69. https://doi.org/10.21314/jcf.2016.322
12. López de Prado, M. (2018). "The 10 Reasons Most Machine Learning Funds Fail." *JPM* 44(6):120-133. https://doi.org/10.3905/jpm.2018.44.6.120
13. Almgren, R. & Chriss, N. (2001). "Optimal execution of portfolio transactions." *J. Risk* 3(2):5-39. https://doi.org/10.21314/jor.2001.041
14. Gatheral, J. (2010). "No-Dynamic-Arbitrage and Market Impact." *Quantitative Finance* 10(7):749-759. https://doi.org/10.1080/14697680903373692
15. Huberman, G. & Stanzl, W. (2004). "Price manipulation and quasi-arbitrage." *Econometrica* 72(1):255-281. https://doi.org/10.1111/j.1468-0262.2004.00531.x

---

## 4. 横切生产实践评审(三类策略共用)

### 4.1 四维度判定表

| 维度 | 判定 | 证据 |
|---|---|---|
| **(1) 流程正确性** | **7/10** | 提及 walk-forward/对账/kill switch/pre-trade risk/Brinson/regime/相对基准 9 个面,方向正确;但多数仅列名未给出方法论选择与门 |
| **(2) 步骤完备性** | **5/10** | 市场冲击仅 commission/tax(未含 square-root/Almgren-Chriss/IS);回测陷阱五件套(lookahead/survivorship/data-snooping/regime overfit/成本低估)未成一等阶段;容量衰减无建模;walk-forward 非 OOS 一等阶段;对账未分层 |
| **(3) 资料完备性** | **6/10** | 市场冲击/执行落差/容量/归因/对账/kill switch 各有概念提及但无具体模型/阈值/报告模板;滑点分解(spread+impact+timing+机会+费用)未展开 |
| **(4) 输出质量可靠性** | **6/10** | OMS/pre-trade risk/kill switch 作为概念存在但未绑定生产 OMS 模式(vnpy RiskManager/Lean RiskManagement/nautilus)的具体阈值族与状态机;回测-实盘 parity 未作为硬约束 |

### 4.2 优先级 GAPS(横切)

| # | Gap | 严重度 | 说明 |
|---|---|---|---|
| XC-GAP-1 | 市场冲击模型族仅 commission/tax,缺 square-root/3-5 幂律 + 线性永久 + 非线性瞬时 + IS 分解 + TCA 校准 | **blocker** | 机构规模成本被低估一个数量级,回测收益不可靠 |
| XC-GAP-2 | 回测陷阱五件套(lookahead/survivorship/data-snooping/regime overfit/成本低估)未成一等 gate 阶段 | **blocker** | 这是 ML 基金失败主因 |
| XC-GAP-3 | Walk-forward/OOS 非 OOS 一等阶段 + 无 Purged/Embargoed CV + 无 PBO/DSR 多重检验 | **blocker** | 标准 k-fold 时序泄漏;无多重校正致伪 Sharpe |
| XC-GAP-4 | 容量衰减 + ADV sizing + fill 概率 + 流动性感知回测未建模 | major | 容量超阈值后 alpha 衰减;纸面策略不可实现 |
| XC-GAP-5 | 对账未分层(position/cash/live-vs-offline/intraday-vs-daily/break 工作流) | major | 单点对账无法定位 drift 与 break |
| XC-GAP-6 | OMS/kill switch 缺 pre-trade risk 阈值族 + 自动取消全单 + 双人重启 + 审计状态机 | major | 实盘故障无防护 |
| XC-GAP-7 | 业绩归因方法未指定(Brinson-Fachler + 因子归因 + 多期链接) | minor | 无法解构 alpha 来源 |
| XC-GAP-8 | regime 分析(牛/熊/波动率/流动性 regime 条件化)未一等化 | minor-major | 样本期外推失败 |
| XC-GAP-9 | 相对基准指标(vs 指数/同业/事件基准组合;信息比率/跟踪误差/Sharpe Deflation)未显式 | minor | 绝对收益不区分 beta/alpha |

---

## 5. 共同人读主线点评

提炼的六阶段主线(信息收集 → 研究 → 回测 → 准入 → 模拟盘 → 实盘 → 生命周期)方向正确,且把"准入""模拟盘"作为独立阶段、强调"离线与在线口径一致""独立授权+风控+OMS+kill switch+审计"——这是工业级正确的。问题在于主线本身缺一个**横切的"统计验证层"**,它应当渗透到研究、回测、准入三个阶段:

```text
信息收集 → 研究 ──┐
                  ├─ [统计验证层: 多重检验 / walk-forward / PBO-DSR] ← 当前缺
回测 ────────────┘
准入 ── [过拟合门 / 容量门 / 冲击门] ← 当前缺
模拟盘 → 实盘 → 生命周期
```

---

## 6. 优先级建议与落地顺序

考虑 CR150 只做多因子 linkage completion、不扩边界,落地顺序如下:

1. **CR150 内可做(契约/工件层,不依赖数据湖)**:在 `MultifactorFrameworkCompletionMap` 中**显式登记**这些缺口为 deferred contract 项——即把"多重检验门""walk-forward""容量/冲击""survivorship-free universe""IR/TE/Active Share"作为**待补契约的字段位**写入 map,gap_count 计入。这样不扩大实现边界,但让缺口机器可读、可审计。
2. **CR151(多因子统计门,已关闭 READY_WITH_RISK)**:FDR/BH + Newey-West t + PBO/DSR + walk-forward,接入步骤 4/6/7。blocker 已以本地/static/fixture capability 形式完成。
3. **CR152(ML 框架基础,已关闭 READY_WITH_RISK)**:已完成 first wave: PIT feature matrix、label policy、purged k-fold+embargo、training snapshot / model artifact metadata、prediction artifact、ML admission gate。triple-barrier/meta-label 仅预留 slot,active 选择 fail-closed；feature importance、fractional diff、样本加权、drift 仍可作为后续 wave。
4. **CR153(事件驱动基础,下一推荐项)**:估计窗/基准模型、检验族、内生性修正、聚类方差、event revision PIT gate 和 event-to-order trace。
5. **横切生产 CR**:市场冲击模型族、容量 sizing、分层对账、kill switch 阈值族、Brinson 归因、regime 分析——这三类策略共用,应作为独立横切 CR 而非塞进单个策略 CR。

---

## 7. 引用更正(资深审稿人务必注意)

研究 agent 在核实中发现并纠正的常见引用错误,后续写入 CR/evidence 时以这些纠正后的口径为准:

- **DSR 非 Journal of Risk**,实为 *J. Portfolio Management* 40(5):94-107(2014);Journal of Risk 15(2):3-44(2012)是其前作"Sharpe Ratio Efficient Frontier"。
- **Hansen SPA 非 J. Econometrics**,实为 *JBES* 23(4):365-380(2005)。
- **Mitchell-Stafford (2000) 在 *Journal of Business* 73(3)** 非 JFE;标题为"Managerial Decisions and Long-Term Stock Price Performance"(非"...Firm Value")。
- **Lyon-Barber-Tsai (1999) 在 *Journal of Finance* 54(1)** 非 RFS。
- **Almgren et al. (2005) 实际拒绝平方根(β=1/2),采用 3/5 幂律(β=0.600±0.038)**——平方根是更广文献现象,非该文具体结论。
- **Huberman-Stanzl:永久影响必须线性**(非"凸或线性")以排除价格操纵;凸永久会允许套利。
- **Eckbo-Masulis-第三作者是 Norli**(非 Vermaalen);**Boehmer 第二作者拼写 Masumeci**(非 Musumeci)。
- **McLean-Pontiff (2016) 在 *JF* 71(1)** 非 JFE;**Brock-Lakonishok-LeBaron (1992) 与 Sullivan-Timmermann-White (1999) 均在 *JF*** 非 JFE。
- **Cowan (1992) 标题为"Nonparametric event study tests"**(无"...and partially parametric")。

---

## 8. 置信度与限制说明

- **事件研究方法、事件研究偏差、回测陷阱、市场冲击四簇**:经 Crossref/一手 PDF 逐条核实,高置信度。
- **多因子多重检验、容量/基准/regime**:经 Qlib 官方文档 + 101 Alphas + 可访问二手权威源(Wikipedia/ThinkNewFound/Google Scholar)交叉验证,中-高置信度;SSRN/Wiley/AMS/Oxford 付费墙(403/402)未直接获取一手正文。
- **Brinson 归因、SEC 80B/80C 熔断**:经 Wikipedia 核实,中-高置信度。
- **vnpy RiskManager / Lean RiskManagement 具体类名与阈值**:作为"公认生产 OMS 模式"引用,未能逐字核到源码(gh search/直接 blob 路径 404),建议后续以 vnpy.com 文档与 Lean 源码 `Common/Algorithm/Framework/Risk/` 终审。
- **容量衰减、对账分层、事件驱动框架(zipline/backtrader)细节**:基于领域公知 + 已验证文献(Kissell, Bucci, Perold)综合,中置信度。

---

## 9. 评审结论摘要

UC-58/59/60 三类策略端到端框架在**流程分级门控**(research→admission→paper→live)、**生命周期覆盖**(含退役)、**PIT/leakage 契约**、**可复现哈希链**、**人工授权/kill switch** 上达到 strong/adequate,显著优于 Qlib 文档级实现。

但在**统计验证完备性**上为 weak/blocker:三类策略共用主线的**统计验证层**缺失,具体表现为四个横切共性 blocker——多重检验门缺失(C1)、walk-forward/OOS 非一等阶段(C2)、市场冲击模型仅 commission/tax(C3)、容量/流动性感知 sizing 缺失(C4)。这四个 blocker 是框架从"研究编排框架"升级为"生产级量化框架"前必须补齐的硬门槛。

建议:CR150 内先以 deferred contract 字段位登记缺口(不扩边界),随后以 CR151(多因子统计门,已关闭)、CR152(ML 基础,已关闭)、CR153(事件驱动基础,下一推荐项)、横切生产 CR 分批补齐。ResearchDatasetSpec/BacktestRunSpec 两个核心契约在当前源码中已存在,后续应扩展而非重建平行契约。
