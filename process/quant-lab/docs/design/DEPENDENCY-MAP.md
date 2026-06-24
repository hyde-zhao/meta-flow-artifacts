---
status: "draft-current-index"
version: "1.4"
source_blueprint: "docs/design/BLUEPRINT.md"
change: "CR-138"
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

## 依赖自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 允许依赖方向覆盖主要跨 Feature 调用 | PASS | §依赖关系 |
| 禁止依赖写明替代路径和违反风险 | PASS | §禁止依赖 |
| 循环风险已状态化 | PASS | §循环风险 |
| 未新增运行授权 | PASS | FD-07、FD-10、FD-12..FD-23、CYCLE-02、CYCLE-03、CYCLE-06..CYCLE-10 |

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
