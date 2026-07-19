---
status: "draft-current-index"
version: "1.20"
source_blueprint: "docs/design/BLUEPRINT.md"
change: "CR-172"
companion_hld_cr172: "docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
companion_adr_cr172: "docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
companion_hld_cr139: "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
companion_dependency_map_cr168: "process/archive/design-cr-docs/DEPENDENCY-MAP-ECONOMIC-COST-IMPACT-EVIDENCE.md"
companion_dependency_map_cr169: "process/docs/design/DEPENDENCY-MAP-CAPACITY-LIQUIDITY-ADV-EVIDENCE-PRODUCER.md"
companion_dependency_map_cr170: "process/archive/design-cr-docs/DEPENDENCY-MAP-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md"
archived_previous: "process/archive/design-blueprints/DEPENDENCY-MAP-before-quant-lab-project-roadmap-2026-06-26.md"
---

# Dependency Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-07 | meta-po | 新增跨 Feature 允许依赖、禁止依赖和循环风险索引 |
| 1.1 | 2026-06-13 | meta-po | 按 CR-046 增补 FEAT-09 双目标策略交付框架的允许依赖、禁止依赖和循环风险 |
| 1.2 | 2026-06-14 | host-orchestrator | 按 CR-051 增补 FEAT-10 策略研究生命周期和项目迁移治理的依赖方向、禁止依赖和循环风险 |
| 1.3 | 2026-06-24 | host-orchestrator | 按 CR138 增补 FEAT-11 Runner Control Plane 与 FEAT-12 QMT Gateway Service Layer 的允许依赖、禁止依赖和循环风险 |
| 1.4 | 2026-06-24 | host-orchestrator | 根据用户 CP3 反馈刷新 CR138 依赖边界：Gateway P0 REST-only；交易日历、佣金 / 费用模型、收益 / PnL 查询进入 FEAT-12 Query Service；runtime policy 改为按需授权 |
| 1.5 | 2026-06-26 | host-orchestrator | 按项目级五阶段目标增补 FEAT-13 策略类型适配与 FEAT-14 成熟策略生产 / 模拟盘观察门禁依赖；明确 Stage 2 不连接数据湖、Stage 3 可在研究机连接数据湖。 |
| 1.6 | 2026-06-27 | codex | 增补统一因子目录依赖边界：mature runner 与 Chapter 3/5/6/7 只能消费登记因子；未登记因子不得隐式进入 FactorSpec 或成熟策略准入。 |
| 1.7 | 2026-06-27 | codex | 增补高级多因子模型评估依赖边界：FEAT-03 只读消费 factor panel、labels、universe、tradability 和 policy cycle config，FEAT-14 必须消费 FactorModelValidationReport 后才能 mature admission。 |
| 1.8 | 2026-06-27 | codex | 收紧 Stage 3 到 Stage 4 依赖：FEAT-14 只能消费评估通过的成熟策略；blocked / insufficient_data 候选不得进入模拟盘观察。 |
| 1.9 | 2026-06-27 | codex | 增补异象发现 / 研究依赖边界：Chapter5 异象候选只能在 AnomalyResearchReport 和 AnomalyAdmissionDecision 通过后进入因子目录或 Stage 3 候选。 |
| 1.10 | 2026-06-27 | codex | 增补自动异象发现依赖边界：候选生成只消费受控模板和研究特征面板，Stage 3 只能显式消费通过多重检验的动态目录候选。 |
| 1.11 | 2026-06-28 | codex | 增补研究引擎稳定模块依赖边界：FEAT-03 主实现只能使用领域名模块；旧 chapter/stage/root 脚本只允许兼容 wrapper 或 legacy archive。 |
| 1.12 | 2026-06-28 | meta-se | 按 CR-139「Strategy Data Foundation」CP3 增补策略生产数据底座依赖方向：reader → published current pointer（非 raw）、ML → panel reader（非 `--data-dir` 旁路）、broker facts/RunEvidenceIndex → 数据 run-id 贯通、配置类事实源 → release 闭环（复用 V1 pointer 语义）；明确禁止反向（engine 不写 lake、reader 不读未发布、DuckDB 不写持久事实源、ML 不旁路）；补禁止依赖 FD-46..52 与循环风险 CYCLE-20..22。AGA-1/3/5 推荐方案 CP3 已确认 A1/C1/E1。 |
| 1.13 | 2026-06-28 | host-orchestrator | CP3 approved，AGA-1/3/5 确认 A1/C1/E1，pending-cp3 → confirmed-cp3。 |
| 1.14 | 2026-07-14 | host-orchestrator inline meta-se-critical | 登记 CR168 companion dependency map 草案：C3→neutral public API、adapter→canonical public validator 单向依赖；禁止 neutral 反向依赖、私有 helper、mapping 透传、C4/aggregate/runtime；待 CP3。 |
| 1.15 | 2026-07-14 | host-orchestrator inline fallback | CP3 已批准后将 CR168 companion dependency map 正文归档到 `process/archive/design-cr-docs/`；长期 index 仅修正状态/路由，不改变依赖规则。 |
| 1.16 | 2026-07-14 | host-orchestrator inline meta-se-critical | 登记 CR169 companion dependency map 草案：C4 producer→neutral envelope、strict joint adapter→canonical public validator 的单向依赖；禁止 external data、private helper、canonical/aggregate 写入，待 CP3。 |
| 1.17 | 2026-07-14 | host-orchestrator inline meta-se-critical | CR169 companion dependency map 通过 CP3：public callable 固定 `validate_gate4_capacity_impact`，alpha 后置 FU008，Stage2 历史缺口只路由 CR157 / 新治理 CR。 |
| 1.18 | 2026-07-15 | host-orchestrator inline meta-se-critical | 登记 CR170 companion dependency map：Gate validators→internal policy、summaries→protected merge→admission resolver 单向依赖；禁止 global bool helper 充当 mandatory 充分条件、runner/aggregate 与 adapter/private-helper 耦合；待 CP3。 |
| 1.19 | 2026-07-15 | host-orchestrator inline meta-se-critical | 精确化 CR170 fixture/test caller→boundary/auth-ref→evaluator 单向依赖；禁止 evaluator 合成授权或重写 T3 existing early-return；待 CP3。 |
| 1.20 | 2026-07-17 | meta-se-critical | 按 CR-172 PATH-I 增补 trial-return→seal→replica→materialization→local runtime 单向依赖、六动作 authorization guard、empirical disposition 与 signal scope ceiling；新增禁止 direct-NAS、NAS canonical、GitHub payload、权限并集和 deferred 反向扩张。 |

## 依赖关系

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-01 本地研究核心 | FEAT-02 数据湖 catalog / readers | read | allowed | 研究消费只能读 published current truth 或 structured missing | consumer boundary tests |
| FEAT-03 多因子研究 | FEAT-02 数据湖 catalog / quality / claims | read | allowed | 因子研究依赖生产数据、benchmark、PIT、quality/readiness | CR030 + CR011 tests |
| FEAT-04 执行语义对齐 | FEAT-01 lightweight output | read / compare | allowed | optional backend 只做语义对照和 diff | CR025 semantic diff tests |
| FEAT-04 执行语义对齐 | FEAT-06 OMS / order intent | handoff | allowed | 只输出 `order_intent_draft_v1` 草稿供后续审查 | order intent draft tests |
| FEAT-05 QMT Gateway | FEAT-07 安全授权 | runtime guard | allowed | pairing / HMAC / scope / nonce / redaction / no-real-op 由安全治理约束 | CR019/CR020 auth tests |
| FEAT-06 交易治理 | FEAT-03 StrategyAdmissionPackage | read / gate | allowed | 交易阶段只能消费准入证据，不能反向生成研究结论 | 后续 CR021..024 |
| FEAT-06 交易治理 | FEAT-02 raw price / metadata | read | allowed | 交易执行只能使用 raw / broker price，不消费 qfq/hfq 执行价 | CR017 leakage tests |
| FEAT-08 文档 Runbook | 全部 Feature | read / summarize | allowed | 文档从设计、实现和验证证据汇总用户操作边界 | docs guardrail tests |
| FEAT-09 双目标策略交付框架 | FEAT-03 StrategyAdmissionPackage | read / contract input | allowed | 策略包合同消费研究准入包和策略元数据，但不反向生成研究结论 | CR046 / 后续 CR047 |
| FEAT-09 双目标策略交付框架 | FEAT-04 OrderIntentDraft | read / schema alignment | allowed | 策略核心合同需要继承 order intent draft 语义 | CR046 static design review |
| FEAT-09 双目标策略交付框架 | FEAT-07 安全授权 | runtime guard | allowed | 策略交付、runner 安装和验证证据必须受不授权边界约束 | CP3/CP5/CP7 |
| FEAT-08 文档 Runbook | FEAT-09 双目标策略交付框架 | read / summarize | allowed | 用户手册需要说明策略包结构、QMT terminal 目标、MiniQMT runner 安装设计和不授权项 | CP8 |
| FEAT-10 策略研究生命周期 | FEAT-02 数据湖 catalog / data release | read | allowed | 研究协议和 run manifest 只记录已发布数据 release 或 structured missing | CR051 manifest review |
| FEAT-10 策略研究生命周期 | FEAT-03 StrategyAdmissionPackage | read / handoff | allowed | delivery_candidate 需要消费研究准入证据和 blocked claim | CR051 / CR052 |
| FEAT-10 策略研究生命周期 | FEAT-09 StrategyCoreContract / StrategyValidationEvidence | read / contract alignment | allowed | 研究生命周期需要知道策略交付合同的最小证据字段 | CR051 CP5 |
| FEAT-08 文档 Runbook | FEAT-10 策略研究生命周期 | read / summarize | allowed | README / USER-MANUAL 需要说明 quant-lab、archive 和迁移边界 | CP8 |
| FEAT-07 安全授权 | FEAT-10 项目迁移治理 | guard / review | allowed | 迁移、archive、alias 和运行声明均必须受 no-real-operation 边界约束 | CP4/CP5/CP7 |
| FEAT-13 策略类型适配 | FEAT-03 多因子研究 | read / normalize | allowed | 多因子研究输出需要归一为 SignalSet / StrategyCandidate 后进入成熟策略门禁 | Stage 2 adapter tests |
| FEAT-13 策略类型适配 | FEAT-10 策略研究生命周期 | read / taxonomy | allowed | 策略类型、研究协议和运行证据需要继承研究生命周期分类和归档合同 | Stage 1 blueprint review |
| FEAT-13 策略类型适配 | FEAT-02 数据湖 current truth | read / reference | allowed-by-stage | Stage 1/2 只读取 schema / contract 或 typed unavailable；Stage 3 才允许在研究机读取真实 data release | Stage 2 no-lake guardrail / Stage 3 lineage gate |
| FEAT-14 成熟策略门禁 | FEAT-13 StrategyCandidate / ResearchEvidenceIndex | read / gate | allowed | mature admission 必须消费统一候选和证据索引，避免策略类型私有路径绕过门禁 | mature admission tests |
| FEAT-14 成熟策略门禁 | FEAT-11 Runner Control Plane | handoff | allowed-with-authorization | 成熟策略只通过 runner offline / plan-only / preflight 后，才可在逐次授权下进入 simulation runtime | Stage 4 runtime gate |
| FEAT-14 成熟策略门禁 | FEAT-07 安全授权 | runtime guard / risk acceptance | allowed | Stage 3/4/5 的 readiness 决策、风险接受和不授权项由安全治理约束 | readiness decision review |
| Mature runner / factor research consumers | FEAT-03 Unified Factor Catalog | read / metadata lookup | allowed | Mature runner、因子复刻、异象研究、稳健性和组合实践只能从统一目录读取 factor_id、状态、来源、计算能力和证据引用，不能各自隐式新增因子身份 | factor registry / CLI tests |
| FEAT-14 mature admission | FEAT-03 Unified Factor Catalog | read / gate | allowed | MatureStrategyDefinition 和 StrategyAdmissionPackage 必须能追溯 factor_refs 到已登记目录项 | mature runner factor spec source tests |
| FEAT-03 FactorModelValidationReport | FEAT-02 policy cycle dataset candidate / data release refs | read / reference | allowed | 政策周期覆盖只能引用配置或已发布 / 候选数据说明，FEAT-03 不生产政策周期事实源 | policy cycle contract tests |
| FEAT-14 mature admission | FEAT-03 FactorModelValidationReport | read / gate | allowed | Mature admission 必须消费高级评估报告状态、核心门禁和 risk_warnings；只有核心门禁无 blocked 且 admission PASS 的策略才能作为 Stage 3 输出，不能只看传统 IC / 分层收益 | mature validation integration tests |
| FEAT-03 AnomalyResearchReport | FEAT-03 factor panel / labels / model returns / factor catalog | read / evaluate | allowed | 异象研究需要消费已登记因子、标签和既有因子模型收益，执行排序、单调性、控制因子、时间切分和成本审查 | Chapter5 anomaly tests |
| FEAT-14 mature admission | FEAT-03 AnomalyAdmissionDecision | read / gate | allowed | 只有 alpha_feature_candidate / factor_catalog_candidate 异象才能作为 Stage 3 新因子候选输入，watch / rejected / blocked 不得进入成熟策略 | anomaly admission tests |
| FEAT-03 AnomalyDiscoveryRun | FEAT-03 ControlledAnomalyTemplate / feature panel / model returns | read / generate / evaluate | allowed | 自动异象发现只能从受控模板生成候选，并记录 candidate_count、搜索空间和多重检验结果 | anomaly discovery tests |
| FEAT-03 Unified Factor Catalog | FEAT-03 AnomalyAdmissionDecision | explicit dynamic read | allowed | 通过准入的自动异象候选可以作为动态 FactorCatalogEntry extras 显式传入查询 CLI 或 Stage 3 candidate spec 构建，不写静态 registry | factor registry dynamic admission tests |
| Stage 3 candidate search | FEAT-03 dynamic anomaly FactorCatalogEntry extras | read / candidate input | allowed | Stage 3 搜索只能消费 `stage3_candidate` 动态候选；默认成熟 runner 固定 5 因子不受影响 | Stage 3 candidate spec tests |
| FEAT-03 research engine stable modules | FEAT-03 shared contracts | import / reuse | allowed | `serialization`、`factor_research_matrices` 和 `admission_contracts` 是研究引擎共享工具，领域模块只消费公共 helper，不反向定义全局状态 | compileall / research tests / script entrypoint guardrail |

## 禁止依赖

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-01 | FEAT-01 / `engine` / experiments | `market_data.connectors` / `market_data.runtime` / provider SDK / credential env | 回测和研究主路径必须只读，不能隐式联网或补数 | 返回 `required_missing` / remediation spec | 污染研究可复现性，误触 provider / 凭据 |
| FD-02 | FEAT-01 / strategies | FEAT-05 / FEAT-06 QMT / OMS / adapter | 策略层不得直连交易系统 | 输出 target portfolio / order intent draft | 绕过风控和授权，可能误触交易 |
| FD-03 | FEAT-02 数据湖 | FEAT-06 broker lake / QMT gateway | 市场数据湖不拥有 broker facts 或交易运行状态 | 通过独立 broker lake / stage gate 管理 | 数据事实源和交易事实源混淆 |
| FD-04 | FEAT-03 多因子研究 | external project runtime / default dependency | CR-030 不授权运行外部框架或默认依赖 | 静态参考矩阵 / optional Spike | 依赖污染、license / provider / truth 边界失控 |
| FD-05 | FEAT-04 Backtrader optional reference | source copy / GPLv3 source migration | CR-025 不授权源码复制或迁移 | 模块级参考、接口适配、semantic diff | license、维护和回归风险 |
| FD-06 | FEAT-05 Linux C 侧 | xtquant / QMT native API | C 侧不得直接触达 QMT native API | REST client -> Windows gateway | 平台边界破坏，无法审计 |
| FD-07 | FEAT-05 gateway health/capabilities | simulation / live operation | endpoint 可见不等于真实操作授权 | stage gate + per-run authorization | 误把可达性当交易许可 |
| FD-08 | FEAT-06 OMS / adapter | qfq / hfq execution price | 复权价不得进入委托、成交、对账 | raw / broker price | 价格口径错误，真实交易风险 |
| FD-09 | FEAT-06 broker lake | repo `data/**` / `reports/**` | broker facts 必须外置或受控，不写研究数据目录 | `BROKER_LAKE_ROOT` 或 dry-run plan | 敏感交易事实污染仓库 |
| FD-10 | FEAT-08 docs / runbook / CP8 | runtime authorization | 文档和终验不自动授权真实操作 | 独立 authorization record | 用户误操作真实 provider / QMT |
| FD-11 | FEAT-09 StrategyCoreContract | QMT / XtQuant / MiniQMT native API | 策略核心必须平台无关，目标适配只能在 target adapter 层出现 | target adapter contract + static guardrail | 策略核心被平台绑定，无法双目标交付 |
| FD-12 | FEAT-09 验证框架 | QMT terminal runtime / MiniQMT connection | CR046 只设计验证框架，不执行真实终端或 runner | fixture/static/design evidence | CP3/CP5 被误读为运行验证 |
| FD-13 | FEAT-09 MiniQMT install design | 真实 Windows 安装目录 / 凭据 / 账户配置 | 本 CR 不真实安装、不读取凭据、不连接账号 | install dry-run spec + redacted placeholders | 泄露凭据或污染真实环境 |
| FD-14 | FEAT-09 策略包合同 | FEAT-06 OMS runtime submit/cancel | 策略包合同不得绕过 stage gate 触发订单 | FollowUpStrategyDeliveryGate + runtime authorization | 误触真实下单或撤单 |
| FD-15 | FEAT-03 研究框架 | FEAT-09 target adapter runtime | 研究框架完善只能消费合同，不得直接驱动 QMT/MiniQMT | StrategyCoreContract / StrategyValidationEvidence | 研究执行路径变成交易运行路径 |
| FD-16 | FEAT-09 交付文档 | runtime verified claim | 未执行 QMT / MiniQMT 真实验证时不得声明 runtime verified | 证据分级声明 | 用户误以为可直接运行 |
| FD-17 | FEAT-10 研究生命周期 | provider SDK / lake write / catalog publish | CR051 只设计研究生命周期和归档治理，不授权补数、写湖或发布 | 记录 data release ref 或 structured missing | 污染事实源并误触真实数据生产 |
| FD-18 | FEAT-10 迁移治理 | QMT / XtQuant / MiniQMT runtime | 项目迁移和研究 archive 不得触发交易终端或 runner | 只消费策略包合同和 redacted evidence | 误触 runtime 或账户环境 |
| FD-19 | FEAT-10 archive | broker lake write | research archive 不拥有 broker facts | broker lake 独立 CR / stage gate | 敏感交易事实进入研究归档 |
| FD-20 | FEAT-10 repository layout | Git 存 raw data / large artifact / credentials | Git 只存代码、schema、manifest、docs 和脱敏摘要 | 外部 archive / lake / ignored artifact roots | 仓库膨胀、凭据泄露、审计污染 |
| FD-21 | CR051 migration Story | NAS 真实扫描 / 挂载 / 搬迁 | CP4/CP5 只做设计，未获运行授权前不得碰真实 NAS | inventory spec + dry-run checklist | 误删、误搬迁或泄露本地数据 |
| FD-22 | ProjectIdentity rename | 批量重写历史 process / CR / handoff 文件 | legacy alias 是审计事实，历史证据不得机械改写 | 新文档使用 canonical name，旧文档保留 alias | 破坏审计链和历史可追溯性 |
| FD-23 | Trading PC workflow | full research archive mount | 交易主机只消费 package，不承担研究 archive 或开发职责 | strategy package / runner bundle + checksum | 交易环境膨胀并扩大运行风险面 |
| FD-32 | FEAT-13 策略类型适配 | runner / gateway / OMS runtime direct call | 策略适配层只做研究输出归一化，不执行 runtime | FEAT-14 mature admission -> FEAT-11 runner gate | 绕过风险准入、授权和 P0-P4 evidence |
| FD-33 | Stage 2 多因子框架升级 | market data lake runtime / provider / catalog publish | Stage 2 明确不连接数据湖，只升级框架合同和测试 | fixture / schema / typed unavailable / static checks | 数据环境耦合、伪造真实 lineage、扩大授权面 |
| FD-34 | 事件型 / ML 策略后续实现 | StrategyAdmissionPackage direct write without adapter | 不同策略类型不得绕过统一 SignalSet / StrategyCandidate 合同 | StrategyTypeAdapter -> ResearchEvidenceIndex -> mature admission | 测试交付分叉，runner 输入不可审计 |
| FD-35 | FEAT-14 mature admission | small_live / live runtime authorization | readiness decision 只能说明入口条件，不自动授权实盘 | independent live switch CR + human gate | 把 simulation 观察误读为实盘许可 |
| FD-36 | Stage 4 日常调整 | untracked parameter / universe / factor mutation | 模拟盘观察期间的调整必须可追溯到 StrategyChangePlan 或观察计划 | versioned StrategyChangePlan + evidence index | 模拟盘表现不可审计，无法判断是否达到实盘入口 |
| FD-37 | Mature runner / factor research consumers | implicit unregistered factor IDs | 因子身份、来源、状态和计算能力必须由统一目录登记，不能在 runner 或研究模块中私有新增 | `engine.factor_registry` + `scripts/list_factors.py` + registry tests | FactorSpec 无法追溯、成熟策略包和研究结果口径漂移 |
| FD-38 | FEAT-03 factor model validation | policy cycle / shortability / shell-value facts fabrication | 评估缺少政策周期、做空可行性或壳价值输入时必须返回 unavailable / risk warning，不得用推断替代事实 | structured insufficient_data / not_applicable + remediation note | Mature admission 证据失真，实盘可行性被高估 |
| FD-39 | FEAT-14 mature admission | skipping FactorModelValidationReport | 成熟策略不能只凭 IC、分层收益或组合回测进入 admission pass | FEAT-03 FactorModelValidationReport + gate summary | GRS / 样本外 / 经济显著性 / 偏差控制缺口被隐藏 |
| FD-40 | Stage 3 blocked / insufficient_data research artifact | Stage 4 simulation observation | Stage 3 的目标是输出评估通过的成熟多因子策略；blocked 候选只能作为诊断和下一轮研究输入 | 修复因子、组合、样本外、经济显著性或数据缺口后重新生成 FactorModelValidationReport 和 mature admission | 模拟盘观察消费失败策略，导致 Stage 4 观察和 small_live 候选判断失真 |
| FD-41 | Anomaly discovery | pure data-mined factor catalog insertion | 异象发现必须有先验逻辑和经济 / 行为 / 微观结构 / 财务扩展来源；不得把参数搜索胜者直接登记为可计算因子 | AnomalyCandidate -> AnomalyResearchReport -> AnomalyAdmissionDecision -> FactorCatalogEntry | 多重检验污染，Stage 3 使用统计幻觉 |
| FD-42 | AnomalyResearchReport | Stage 3 factor candidate without Harvey / monotonicity / alpha / split / cost gates | 异象候选必须通过严格研究或明确 watch / blocked；不得只凭样本内收益进入成熟策略 | 完成单变量排序、Harvey t>=3、五分组单调、控制因子 alpha、时间切分、成本和 A 股控制 | 因子模型过拟合，样本外失效 |
| FD-43 | FEAT-03 automatic anomaly discovery | black-box search / unrecorded candidate count | 自动发现必须使用受控模板并记录搜索空间、多重检验和候选总数；未记录者不得准入 | ControlledAnomalyTemplate + MultipleTestingControl | 多重检验偏差和过拟合被隐藏 |
| FD-44 | FEAT-03 dynamic anomaly catalog | static registry mutation / catalog publish from discovery run | 发现结果只能显式作为动态 extras 消费，不得自动写静态因子库或 publish 数据 catalog | anomaly_candidate_catalog_entries + no-real-op counters | 一次研究结果被永久化或误发布 |
| FD-45 | FEAT-03 research engine implementation | new `chapter*` / `stage*` / `cr*` top-level engine main module | 长期实现入口必须是领域名；旧名称必须集中在 `docs/legacy/archive/engine/`，不得留在 `engine/` 顶层 | `scripts/quality/check_script_entrypoints.py` + archived engine module check | 新实现继续扩散历史阶段 / 章节命名，文档、测试和 runner 消费入口再次漂移 |

## 循环风险

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-01 | FEAT-01 研究消费 <-> FEAT-02 数据湖生产 | 研究发现缺口后反向触发补数，破坏只读消费边界 | eliminated：consumer 只返回 structured missing；真实补数必须新授权 |
| CYCLE-02 | FEAT-03 StrategyAdmissionPackage <-> FEAT-06 StageGate | admission package 被解释为 simulation/live 授权 | eliminated：admission 只提供证据，不提供 runtime authorization |
| CYCLE-03 | FEAT-05 gateway capabilities <-> FEAT-07 authorization | endpoint 可见被误读成真实 QMT 可调用 | eliminated：run mode / scope / stage / per-run authorization 独立检查 |
| CYCLE-04 | FEAT-04 optional backend <-> FEAT-01 main path | Backtrader / Qlib 参考反向污染 lightweight 默认主路径 | accepted-with-guardrail：optional + lazy / unavailable + no default dependency |
| CYCLE-05 | FEAT-08 docs <-> FEAT-07 safety | 文档为了易用性弱化不授权项 | eliminated：docs guardrail 必须列不授权项和 no-real-op 计数 |
| CYCLE-06 | FEAT-09 策略包合同 <-> FEAT-06 交易治理 | 策略包合同被解释为可提交订单的 adapter 实现 | eliminated：CR046 只输出合同和验证计划；submit/cancel 必须后续 runtime authorization |
| CYCLE-07 | FEAT-09 MiniQMT runner target <-> FEAT-05 QMT gateway | runner 安装设计与 gateway runtime 边界混淆 | eliminated：runner target 只定义安装 / 进程 / 日志 / kill switch 合同，不启动 gateway 或连接 MiniQMT |
| CYCLE-08 | FEAT-10 研究生命周期 <-> FEAT-09 策略交付合同 | delivery_candidate 被反向解释为可运行策略包 | eliminated：delivery_candidate 只进入后续 CR；runtime_candidate 必须独立授权 |
| CYCLE-09 | ProjectIdentity alias <-> 历史审计证据 | 为了新名 `quant-lab` 批量改写 `local_backtest` 历史证据 | eliminated：新文档用 canonical，历史材料保留 legacy alias |
| CYCLE-10 | Research archive <-> Market data lake current truth | 研究归档 manifest 被误当作事实源或 publish gate | eliminated：archive 只存指针和脱敏摘要，current truth 仍由 FEAT-02 publish gate 管理 |
| CYCLE-16 | FEAT-13 策略类型适配 <-> FEAT-03 多因子框架 | 多因子实现需求反向把统一 adapter 写成多因子专用模型 | eliminated：多因子字段留在 adapter implementation，统一合同只承载 SignalSet / StrategyCandidate / evidence |
| CYCLE-17 | Stage 2 框架升级 <-> FEAT-02 数据湖 | 为了验证框架提前连接数据湖，导致研究框架改动依赖真实数据环境 | eliminated：Stage 2 no-lake；真实数据连接推迟到 Stage 3 |
| CYCLE-18 | Stage 4 模拟盘观察 <-> Stage 5 live gate | 模拟盘短期 pass 被反向解释为 small_live/live approval | eliminated：ReadinessDecision 必须列 non_authorized_items；Stage 5 独立 CR |
| CYCLE-19 | FEAT-03 FactorModelValidationReport <-> FEAT-02 policy cycle data | 研究评估为了补齐政策周期覆盖反向生产或修改数据湖事实 | eliminated：FEAT-03 只读配置 / 数据引用；真实政策周期数据集进入 FEAT-02 独立生产 |

## 依赖自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 允许依赖方向覆盖主要跨 Feature 调用 | PASS | §依赖关系 |
| 禁止依赖写明替代路径和违反风险 | PASS | §禁止依赖 |
| 循环风险已状态化 | PASS | §循环风险 |
| 未新增运行授权 | PASS | FD-07、FD-10、FD-12..FD-23、FD-32..FD-44、CYCLE-02、CYCLE-03、CYCLE-06..CYCLE-10、CYCLE-16..CYCLE-19 |

## CR138 增量：Runner / Gateway 依赖边界

### 依赖关系增量

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-11 Runner Control Plane | FEAT-09 StrategyPackageContract | read | allowed | Runner 需要读取策略包、target 和验证证据摘要生成 RunPlan | CR138 HLD traceability |
| FEAT-11 Runner Control Plane | FEAT-03 StrategyAdmissionPackage | read / gate | allowed | 多因子和策略运行必须先消费 admission 结果 | preflight fixture |
| FEAT-11 Runner Control Plane | FEAT-04 OrderIntentDraft | read / transform | allowed | Runner 只生成或消费 order intent draft，不直接提交订单 | order intent schema tests |
| FEAT-11 Runner Control Plane | FEAT-06 OMS / Risk | handoff | allowed | 事件 / 信号 / 再平衡结果必须经 OMS / Risk 才能进入 Gateway command gate | risk gate tests |
| FEAT-11 Runner Control Plane | FEAT-12 QMT Gateway Service Layer | event / query / command intent | allowed-with-authorization | Runner 可消费 GatewayHealth / ExecutionReport / AuditRecord；真实 query/order 需授权 | no-real-op counters |
| FEAT-12 QMT Gateway Service Layer | FEAT-07 Safety Authorization | runtime guard | allowed | Gateway 所有 account / quote / order / submit / cancel 动作必须先检查 authorization | auth tests |
| FEAT-12 QMT Gateway Service Layer | FEAT-06 OMS / Risk | execution report / order state | allowed | Gateway 标准化回报后回链 OMS 和 Runner | event replay tests |
| FEAT-12 QMT Gateway Service Layer | FEAT-06 OMS / Risk | fee / pnl / order-state read | allowed-with-authorization | 佣金估算、收益 / PnL 和账户状态查询必须引用 OMS / Risk / Order State 或授权账户快照，不能由 Gateway 单独解释策略收益 | query contract tests |
| FEAT-08 Docs / Runbook | FEAT-11 / FEAT-12 | read / summarize | allowed | CP8 和运行手册需要解释运营路径、不授权项和恢复流程 | docs guardrail |

### 禁止依赖增量

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-24 | FEAT-11 Runner Control Plane | xtquant / QMT / MiniQMT native API | Runner 只能表达策略运行意图，不能触达 broker 原语 | FEAT-12 GatewayCommand + FEAT-06 OMS / Risk + FEAT-07 Authorization | 绕过 Gateway、风控和审计 |
| FD-25 | FEAT-11 Runner Control Plane | credential env / `.env` / account secrets | Runner 运营面不读取凭据或账户敏感信息 | redacted AuthorizationRecord / GatewayHealth | 凭据泄露 |
| FD-26 | FEAT-12 QMT Gateway Service Layer | strategy signal generation / target portfolio calculation | Gateway 不决定策略逻辑或调仓目标 | FEAT-11 Runner + FEAT-03 / FEAT-04 inputs | Gateway 业务化且难以测试 |
| FD-27 | FEAT-12 GatewayHealth / CapabilitySnapshot | runtime authorization | 可用性和能力探测不等于真实查询、订阅、下单或撤单许可 | FEAT-07 AuthorizationRecord | 误把 health pass 当运行许可 |
| FD-28 | FEAT-11 / FEAT-12 CP3 HLD | source code / dependency / install script mutation | CR138 CP3 只做设计，不实现 | CP5 通过后按 Story 执行 | 未经设计证据确认直接实现 |
| FD-29 | FEAT-12 GatewayAuditRecord | raw QMT logs / account ids / credentials | 审计只保留脱敏摘要和引用，不复制敏感原文 | redaction policy + audit refs | 敏感数据进入 process / reports |
| FD-30 | FEAT-11 Runner Control Plane | account-level commission / pnl / position / order native query | Runner 只能消费脱敏查询摘要和引用，不能读取账户级事实 | FEAT-12 Reference / Account Query Service + FEAT-07 Authorization | Runner 越权读取账户或把敏感数据写入 evidence |
| FD-31 | FEAT-12 Query Service | fabricated broker commission / pnl facts | QMT 不支持或无授权时不得伪造 broker confirmed 查询结果 | unavailable_with_reason / estimated mode / configured FeeSchedule | 复盘和收益归因失真 |

### 循环风险增量

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-11 | FEAT-11 Runner <-> FEAT-12 Gateway | Runner 根据 Gateway 状态自动重发订单，Gateway 回报又触发 Runner 重试 | eliminated：重试必须经 OMS / Risk / Authorization；unknown 状态进入 manual_review |
| CYCLE-12 | FEAT-12 GatewayHealth <-> FEAT-07 Authorization | health pass 被反向解释为授权 active | eliminated：GatewayHealth 与 AuthorizationRecord 分离 |
| CYCLE-13 | FEAT-11 Runner <-> FEAT-09 StrategyPackageContract | Runner 运营需求反向修改策略包合同 | accepted-with-guardrail：合同变更必须走 StrategyChangePlan / follow-up CR |
| CYCLE-14 | FEAT-12 Gateway <-> FEAT-06 OMS | Gateway order report 与 OMS state 互相覆盖状态事实 | eliminated：OMS owns order state；Gateway owns raw standardized ExecutionReport |
| CYCLE-15 | FEAT-11 Runner <-> FEAT-12 Query Service <-> FEAT-06 OMS | Runner 复盘要求反向驱动账户查询，查询结果又覆盖 OMS / PnL 事实 | eliminated：FEAT-06 owns order / pnl state interpretation；FEAT-12 owns query transport and redaction；FEAT-11 consumes summaries only |

### CR138 依赖自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| FEAT-11 / FEAT-12 允许依赖方向明确 | PASS | CR138 依赖关系增量 |
| 禁止依赖写明替代路径和违反风险 | PASS | FD-24..FD-31 |
| 循环风险已状态化 | PASS | CYCLE-11..CYCLE-15 |
| 未新增自动运行授权 | PASS | FD-27、FD-28、HLD §12/§13 |

## CR-139 增量：策略生产数据底座依赖边界

> 来源：CR-139 companion HLD「Strategy Data Foundation」（写侧/读侧分离）+ REQ-201..249。明确依赖方向：reader 依赖 published current pointer（非 raw）、ML 依赖 panel reader（非旁路）、broker facts/RunEvidenceIndex 依赖数据 run-id 贯通、配置类事实源依赖 release 闭环。不得反向。

### 依赖关系增量

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-02 读侧（PIT/panel/dedup/selector/audit） | FEAT-02 写侧 published current pointer | read | allowed | reader 只消费已发布 current truth，不读 candidate/raw | V1/C1/R1 published-gate tests |
| FEAT-03 ML feature 层 | FEAT-02 读侧 panel reader | read | allowed | ML 接入 `read_panel_as_of`，废除 `--data-dir`/`load_local_frames` 旁路 | R2 no-bypass tests |
| FEAT-03 feature/label/artifact | FEAT-02 published dataset snapshot | read / reference | allowed | feature/artifact hash 引用 published dataset snapshot，可复现 | E1/E2 snapshot-ref tests |
| FEAT-06 broker event | FEAT-02 数据 run-id | read / lineage link | allowed | broker event 通过 run-id 与数据/研究审计链贯通 | T6 run-id tests |
| FEAT-11 RunEvidenceIndex | FEAT-02 读审计 log + 数据 run-id | read / lineage link | allowed | RunEvidenceIndex 闭环 lake run-id 与读审计 | L3/T6 run-id tests |
| FEAT-12 CommissionSchedule / FEAT-14 universe·risk policy / FEAT-03 policy_cycle | FEAT-02 config_facts release（复用 V1 pointer 语义） | read / release link | allowed | 配置类事实源版本化 + release 闭环，归因可复现 | F1-F4 version+release tests |
| FEAT-14 PIT universe 成分链 | FEAT-02 index_members snapshot | read | allowed | 按时点构建 PIT universe，消除固定快照偏差 | X4 PIT universe tests |
| FEAT-02 读侧 DuckDB adapter | FEAT-02 published parquet / catalog | read-only | allowed | DuckDB 只读查询，parquet 仍是存储（D4） | R3 readonly tests |
| FEAT-02 replay | FEAT-02 published as_of snapshot | read | allowed | replay 按 published as_of 重放单日快照，不触发 provider | L5 replay tests |

### 禁止依赖增量

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-46 | FEAT-01 / FEAT-03 / engine | FEAT-02 写侧 canonical 直写 / raw 直读 | 研究消费只读 published current truth，不得直写 lake 或直读 raw/candidate | 返回 required_missing / 走 published reader | 污染事实源、绕过 publish gate |
| FD-47 | FEAT-02 读侧 reader | 未发布 candidate / raw | reader 只消费 published current truth；读未发布破坏冻结快照防线 | published gate 阻断返回 unavailable | 调参偷看未来、版本漂移 |
| FD-48 | FEAT-03 ML 实验 | `--data-dir` / `load_local_frames` 旁路 | ML 必须接入 `read_panel_as_of`，旁路导致训练-实盘漂移 | read_panel_as_of | ML 旁路数据湖、不可复现 |
| FD-49 | FEAT-02 DuckDB adapter | 持久事实源写入 / `.duckdb` 替代 parquet lake | DuckDB 只读引擎，parquet 仍是存储（D4） | 只读 parquet/catalog | 事实源分叉、依赖膨胀 |
| FD-50 | FEAT-02 engine / reader | 反向写 canonical / publish pointer | engine 不写 lake；reader 不写 | 写侧 publish gate 唯一入口 | 破坏 publish gate 单一入口 |
| FD-51 | FEAT-06 broker lake / FEAT-11 run evidence | 伪造 run-id 贯通 / 断链静默成功 | run-id 贯通必须真实链接，断链标注断点 | 标注 broken + 补全 | 伪造审计链、复盘失真 |
| FD-52 | FEAT-03/12/14 配置类事实源 | 无 version / 无 release 闭环即声明可复现 | 配置类事实源必须带 version + release_id | 缺版本阻断复现声明 | 归因不可复现 |
| FD-53 | Wave1 结构性变更 | T8/T7/C2a 基线冻结未完成即执行 | Wave1 基线门（REQ-247）：结构性变更必须在清册→黄金值→重复画像之后 | 基线冻结门阻断 | 历史数据变化与结构修复混在一起无法归因 |

### 循环风险增量

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-20 | FEAT-02 写侧 publish ↔ FEAT-02 读侧 reader | reader 发现缺口反向触发写侧补数/publish，破坏只读消费边界 | eliminated：reader 只返回 structured missing；补数/publish 必须独立授权 |
| CYCLE-21 | FEAT-03 ML feature 层 ↔ FEAT-02 published snapshot | ML 为补齐 feature 反向修改 published dataset snapshot | eliminated：feature 层只读 published snapshot；feature 缺口返回 unavailable |
| CYCLE-22 | FEAT-06 broker event ↔ FEAT-02 数据 run-id | broker 侧为对账反向修改数据 run-id 或读审计 | eliminated：broker event 只读消费 run-id；run-id 由 FEAT-02 写侧生成，单向贯通 |

### CR-139 依赖自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 写侧/读侧依赖方向明确（reader→published、ML→panel、broker→run-id、config→release） | PASS | §依赖关系增量 |
| 禁止反向依赖写明替代路径与违反风险 | PASS | FD-46..53 |
| 循环风险已状态化 | PASS | CYCLE-20..22 |
| 未新增运行授权 / 物理分区迁移授权 | PASS | FD-53、REQ-247/248、Wave1 N1 后置 |

## CR158 增量：Event + ML Strategy Adapter 依赖边界

### 依赖关系增量

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-13 Strategy Type Adapter | FEAT-03 ML evidence refs | read / reference | allowed | ML extension 只引用 training snapshot、model artifact、validation report 和 prediction signal refs。 | SC-CR158-P01 / N01 |
| FEAT-13 Strategy Type Adapter | FEAT-07 Safety Authorization | guard | allowed | Adapter validation 必须消费 no-runtime policy 和 forbidden operation counters。 | SC-CR158-N02 |
| FEAT-13 Strategy Type Adapter | FEAT-11 Handoff Consumer | handoff refs | allowed | 只有 unified core 输出可进入 Stage 2/Stage 3 handoff refs。 | SC-CR158-P01 |
| FEAT-03 ML evidence refs | FEAT-13 Adapter Core | typed extension | allowed | ML-specific evidence 进入 type-specific extension，不反向修改 core 字段。 | CP5 design review |
| FEAT-08 Docs / Release Wording | FEAT-07 NotAuthorizedList | read / summarize | allowed | CP8 wording 必须复述 no-runtime / no-publish 边界。 | SC-CR158-A01 |

### 禁止依赖增量

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-CR158-01 | FEAT-13 Adapter Core | real event feed / live listener / provider / gateway | CR158 只允许 fixture/static event refs。 | EventAdapterExtension refs + no-runtime counters | 未授权 runtime / release overclaim |
| FD-CR158-02 | FEAT-13 / FEAT-03 ML Extension | real model training / external model service / model registry write | CR158 不授权训练、服务调用或 registry promotion。 | MLAdapterExtension refs + metadata only | 未授权模型发布或数据泄漏 |
| FD-CR158-03 | FEAT-11 Handoff Consumer | event-only / ML-only private fields | 下游只能消费 shared core 和 typed refs，不得依赖策略类型私有对象。 | StrategyTypeAdapterCore + typed extension refs | Handoff schema 分叉、Story 耦合 |
| FD-CR158-04 | CP3 HLD / ADR | source/test implementation | CP3 只做架构评审，implementation 等 CP5。 | CP4/CP5 Story design evidence | 绕过设计证据门禁 |
| FD-CR158-05 | Evidence index extension | report body / event payload body / model binary / transcript / full diff | Evidence index 必须 refs-only。 | typed refs + short metadata | process/evidence 膨胀、敏感内容泄露 |

### 循环风险增量

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-CR158-01 | FEAT-13 shared core <-> event / ML extensions | 为统一而把 event-only / ML-only 字段互相设为必填 | eliminated：thin core + typed extension；CP5 检查互不污染 |
| CYCLE-CR158-02 | FEAT-03 ML evidence <-> FEAT-13 adapter core | ML 证据需求反向扩张 adapter core，导致 event adapter 背负 ML 字段 | eliminated：ML 证据只进 ML extension |
| CYCLE-CR158-03 | CP3 architecture <-> implementation desire | 为验证架构而提前实现 adapter | eliminated：CP3 approval 不授权实现；CP5 前 source/test implementation blocked |

### CR158 依赖自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 允许依赖方向覆盖 adapter core、typed extension、safety guard 和 handoff | PASS | §依赖关系增量 |
| 禁止依赖写明替代路径和违反风险 | PASS | FD-CR158-01..05 |
| 循环风险已状态化 | PASS | CYCLE-CR158-01..03 |
| 未新增运行授权 | PASS | FD-CR158-01 / 02 / 04 |

## CR-172 增量：PATH-I Trial-Return 与部署依赖边界

### 依赖关系增量

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-CR172-I01 Trial Return Artifact | FEAT-CR172-I03 ActionAuthorizationGuard | policy input / guard | allowed | dereference/runtime-write/generation 前必须消费单 action deny-default decision | SC-A02/Q02、auth matrix |
| FEAT-CR172-I01 ManifestSeal | future producer/import candidate | validate / transform | allowed | candidate 只有通过 source/schema/identity/time/value/lineage 校验才可 manifest/seal | SC-I01/N02/N04 |
| FEAT-CR172-I02 ReplicaVerifier | FEAT-CR172-I01 sealed artifact | read / verify | allowed | 只读 immutable payload/manifest/seal 和 expected release；不改 canonical | SC-I02/N03/B03/F02 |
| FEAT-CR172-I02 ReplicaVerifier | FEAT-CR172-I03 ActionAuthorizationGuard | guard | allowed | NAS staging/write/pointer 前必须有 `nas_replica_sync` 单项允许 | SC-A02 |
| FEAT-CR172-I02 Materializer | FEAT-CR172-I02 verified replica receipt | read / verify | allowed | execution pull 只能从 verified expected release 开始 | SC-I02/F02 |
| FEAT-CR172-I02 Materializer | FEAT-CR172-I03 ActionAuthorizationGuard | guard | allowed | NAS read/pull、execution staging/pointer 前需 `execution_pull_verify_materialize` | SC-A02 |
| FEAT-CR172-I03 EmpiricalEligibilityGuard | FEAT-CR172-I01 sealed trial refs | read / classify | allowed | empirical disposition 必须引用 sealed same-family/run artifacts，不读 payload 以外对象冒充 | SC-N02/N04/C02 |
| FEAT-CR172-I03 EmpiricalEligibilityGuard | `FU-CR173-001` status / method evidence refs | read / gate | allowed | positive empirical claim 需 v2+hash/uncertainty/domain/bias/independent validation；降级设计不依赖其完成 | SC-Q02/C02 |
| GitHub metadata surface | FEAT-CR172-I01/I02 manifests/hashes/receipts refs | metadata reference | allowed | GitHub 只保存 code/schema/docs/manifest/hash/pointer/release metadata | SC-N05 |
| future execution runtime | FEAT-CR172-I02 local immutable cache selection | local read-only | allowed-after-future-authorization | runtime 只读本地 selected cache，不建立 NAS runtime dependency | SC-I02/A02 |

### 禁止依赖增量

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-CR172-I-01 | execution runtime | NAS replica / research-local canonical | shared/direct remote read 扩大故障域并绕过 materialization receipt | verified local immutable cache | stale/partial 被消费、runtime 不可用、权限越界 |
| FD-CR172-I-02 | NAS replica/distribution pointer | research canonical authority | NAS 只是副本，不能反向覆盖/提升 canonical | research-local selection 只由 sealed local artifact 推进 | 双 canonical、identity 漂移、错误恢复 |
| FD-CR172-I-03 | replica/materializer | 重新 seal / 修改 manifest/hash | 跨机复制必须保持原 identity，不产生新 canonical seal | verify original seal + typed receipt | 同一 artifact 跨机身份分叉 |
| FD-CR172-I-04 | GitHub metadata surface | real dataset/return payload、credential、account/order payload | 违反 GitHub data ceiling | refs/hash/manifest/release metadata only | 数据/凭据泄漏、仓库污染 |
| FD-CR172-I-05 | trial-return source resolver | CR-163 metadata/ref、`layered_returns.csv`、scalar metric、declared dependency R | 这些对象不是 per-trial portfolio return series | native producer；approved import contract；typed_unavailable | 伪 empirical R、C1 overclaim |
| FD-CR172-I-06 | authorization decision | 其他 action 的 approval / permission union | 六动作必须独立 owner/scope/hash/path/expiry/revoke | one action request→one decision | blast radius 扩大、无法归因/撤销 |
| FD-CR172-I-07 | EmpiricalEligibilityGuard | declared_exact→empirical relabel、silent repair、raw alias | 分类/provenance/repair/auth 不得推断 | typed_unavailable 或 BLOCKED | sampling-error/方法有效域 overclaim |
| FD-CR172-I-08 | new run path contract | legacy write/move/rename/recanonicalize/manifest rewrite | 破坏历史审计与 stable identity | semantic new root；migration 独立 CR | 历史链断裂、双默认 |
| FD-CR172-I-09 | SignalBatchBoundaryV1 | mailbox path/state/ack/idempotency/replay/exchange/consumer/intraday module | PATH-I 只批准 exact 8-slot boundary | deferred candidate / independent CR | scope inflation、交易/运行权限扩大 |
| FD-CR172-I-10 | CP3 HLD/ADR | source/test/fixture implementation、目录创建、runner/default 修改 | CP3 design-only | CP3 approved→CP4/CP5 | 绕过设计证据与人工门禁 |
| FD-CR172-I-11 | CP3/CP8 wording | Stage3/C1/real-data/multi-trial/signal authorization=true | PATH-I 最高 contract-ready，不是 activation | 保持五项 false；future runtime-high-risk gate | 发布 overclaim、未授权真实运行 |

### 循环风险增量

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-CR172-I-01 | research canonical ↔ NAS replica | stale NAS 为“恢复”反向覆盖 local canonical | eliminated：单向 local→replica；NAS receipt 无 canonical write authority |
| CYCLE-CR172-I-02 | NAS replica ↔ execution runtime | runtime 为便利直读 NAS，反向依赖共享盘 freshness | eliminated：authorized pull→local immutable cache；runtime local-only |
| CYCLE-CR172-I-03 | action authorization ↔ operation evidence | 某 action PASS 被用来隐含批准其他 action | eliminated：single action envelope/decision；no permission union |
| CYCLE-CR172-I-04 | empirical disposition ↔ declared-exact fixture | 为产生 positive result 将 fixture R 重标 empirical | eliminated：四态分类；pre-v2/缺 provenance unavailable，冲突 BLOCKED |
| CYCLE-CR172-I-05 | Signal boundary ↔ exchange implementation | 8-slot contract 反向驱动 mailbox/ack/replay/intraday Story | eliminated：Signal detailed module/Story/implementation=`0/0/0`，路由 deferred |

### 无环数据流

```text
future source candidate
  -> FEAT-I01 validate/manifest/seal/research selection
  -> FEAT-I02 NAS staging/verify/replica receipt
  -> FEAT-I02 execution staging/verify/materialization receipt/local cache
  -> future execution runtime (local read-only)

FEAT-I03 authorization guard -> 每个 action 的前置 decision（不接收下游反向授权）
FEAT-I03 empirical guard <- sealed refs + FU status（只输出 disposition，不写 trial artifact）
```

上述图不存在从 execution/NAS/GitHub/claim guard 回写 research canonical 的边；因此 DAG 无环。

### CR-172 依赖自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 允许方向覆盖 source→seal→replica→materialize→local runtime | PASS | 依赖关系增量 `10` 条 |
| 数据 owner 与写权限唯一 | PASS | FEAT-I01 canonical、FEAT-I02 receipts、FEAT-I03 governance |
| direct-NAS/NAS canonical/GitHub payload/权限并集明确禁止 | PASS | FD-CR172-I-01～06 |
| empirical/signal/CP3 scope 反向扩张被阻断 | PASS | FD-CR172-I-07～11 |
| 循环风险已消除 | PASS | CYCLE-CR172-I-01～05；无环数据流 |
| 真实动作与运行声明未放大 | PASS | real action auth/exec=`0/6`；五项 flag=false |
