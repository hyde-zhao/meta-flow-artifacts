---
discussion_id: "CP3-CR138-HLD-DISCUSSION"
status: "completed"
created_at: "2026-06-24T14:10:00+08:00"
created_by: "host-orchestrator"
cr_id: "CR-138"
source_context: "process/context/CP2-CR138-RUNNER-QMT-OPERATIONAL-USE-CASE-CONTEXT.yaml"
---

# CP3 HLD Discussion Log - CR138

## 讨论目标

基于已确认的 Runner / QMT Gateway operational use-case baseline，收敛 CP3 Architecture Gray Areas，形成 Runner Control Plane 与 QMT Gateway Service Layer 的功能边界、能力地图、领域对象、依赖方向和 HLD 推荐方案。

## 输入摘要

| 输入 | 结论 |
|---|---|
| `process/USE-CASES.md` v1.16 | UC-33..UC-43 描述 Runner 运营路径；UC-44..UC-50 描述 QMT Gateway 运营路径；UC-10/11/15/16/17/28..32 作为治理、接口和交付框架来源。 |
| CR138 CP2 Decision Brief | Runner 负责策略运行、观察、调整、复盘；Gateway 负责安全稳定触达 QMT / MiniQMT / XtQuant；CP3 不自动授权 runtime 或真实交易，后续验证可按需授权。 |
| Runner 研究报告 | 应吸收 BulletTrade 的运行时骨架、lite-qmt-executor 的 WAL / 恢复 / 订单状态经验、EasyXT 的多策略场景覆盖。 |
| Gateway 研究报告 | 应组合 quant-qmt-proxy 的协议分层与 qmt-bridge 的服务生命周期、xtdata 串行化、独立调度器、事件回报和故障恢复经验；P0 不必继承 WebSocket。 |

## Architecture Gray Areas

| 灰区 ID | 关键问题 | 为什么影响架构 | 影响面 | canonical refs | 结论状态 |
|---|---|---|---|---|---|
| AGA-138-01 | Runner Control Plane 是否独立于 offline `strategy-runner-core`？ | 若混入 offline runner core，会继续围绕 RunSpec / bundle / registry 拆 Story，无法承接盘前、盘中、异常、策略生命周期运营。 | Feature 边界、数据归属、Story 拆分、测试矩阵 | UC-33..UC-43、CR137 handoff、`process/docs/features/strategy-runner-core/DESIGN.md` | resolved |
| AGA-138-02 | QMT Gateway 是只读查询小接口、策略包 target，还是服务化运行层？ | 若只保留 CR019/CR020 只读准入或 CR046 target，无法覆盖启动、health、订阅、回报、恢复、审计和配置变更。 | 模块边界、协议面、授权、运行故障 | UC-44..UC-50、UC-16/17、quant-qmt-proxy / qmt-bridge 报告 | resolved |
| AGA-138-03 | Runner 与 Gateway 之间采用直接 xtquant 调用、文件交换，还是领域事件 / 命令契约？ | 直接调用会绕过 Gateway 能力探测、风控、幂等和审计；纯文件交换难以支持盘中订阅和回报。 | 接口契约、失败恢复、审计、测试 | UC-35..UC-49、HLD-QMT-TRADING、研究报告 | resolved |
| AGA-138-04 | CP3 approval 与后续按需 runtime 授权如何区分？ | 若在 HLD 阶段合并 runtime 授权，会破坏后续 per-run authorization 边界；但永久不授权又会阻断必要验证。 | 安全、门禁、发布风险、测试声明 | 用户 CP3 反馈、DQ-CP2-CR138-03、UC-47、CR138 authz policy | resolved |
| AGA-138-05 | 首轮落地应做单份 HLD 还是拆为 Runner / Gateway 两份 HLD？ | 两个产物可独立实现，但共享事件模型、授权边界、审计 ID 和场景模拟；过早拆分会重复 ADR。 | 文档结构、评审负担、Story 拆分 | HLD 拆分原则、UC-33..UC-50 | resolved-with-risk |
| AGA-138-06 | Gateway P0 是否应 REST-only，还是提前引入实时推送 / SDK / FIX 协议？ | 协议选择影响 Gateway 复杂度、测试策略、SDK 承诺和实时性；P0 多协议会扩大范围。 | Gateway service boundary、协议、测试、运维 | 用户 CP3 反馈、quant-qmt-proxy / qmt-bridge 报告 | resolved |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 独立 Runner Control Plane + 独立 QMT Gateway Service Layer，通过领域命令 / 事件 / 审计 ID 衔接 | Runner 不直连 xtquant；Gateway 不决定策略逻辑；可按授权层分开验证；符合两份研究报告的分层经验 | 需要定义更多领域对象和状态机；Story 数会多于离线小 slice | Feature、Domain、Dependency、HLD、ADR、CP5 Story 拆分 | 推荐 | 当前目标覆盖 UC-33..UC-50 全链路运营；若后续只做 offline runner，可降级为 existing `strategy-runner-core` 增量。 |
| B. 扩展现有 `strategy-runner-core`，让 Runner 直接持有 Gateway client / xtquant adapter | 改动表面最小；复用已有 RunResult / bundle / registry 概念 | 高风险混淆 offline artifact 与 online operation；Runner 容易直接承担账户/行情/订单语义 | 源码、测试、安全、文档 | 不推荐 | 仅当用户明确把 CR138 缩回 offline-only operational reporting 时可切换。 |
| C. 以 Gateway 为中心，Gateway 同时调度策略、管理组合和交易 | Gateway 运维面统一；对单机 PoC 直观 | Gateway 业务化，策略生命周期、组合再平衡和复盘难以测试；违反 CP2 边界 | 架构、安全、测试、维护 | 不推荐 | 仅当未来采用完整第三方交易平台替代当前研究/Runner 层时重评估。 |
| D. 先做只读 Gateway + Runner 观察面，延后订单回报 / submit / cancel HLD | 风险最低，能快速进入只读状态检查 | 会遗漏 UC-47 和恢复对未确认订单的关键建模，后续 HLD 返工 | 范围、测试、Story 计划 | 条件推荐为缩小版 | 若用户要求缩小 CP3 范围，可切换；本轮推荐完整建模，运行按需授权。 |

## Lane 汇总

| Lane | 输入 | 影响章节 | 处理 |
|---|---|---|---|
| lane-product | 用户已纠偏 use-case 必须描述真实运营，而不是 RunSpec / bundle / registry | HLD §1、§6、§7、§15 | adopted |
| lane-architecture | Runner / Gateway / OMS / FEAT-09 必须边界分明，Gateway service layer 不等于 terminal target | HLD §3、§4、§8、§9、ADR | adopted |
| lane-quality | CP3 approval 不自动授权 runtime；账户、行情、订单、submit/cancel 必须通过最小 action scope 授权；fixture / design pass 不得声明 runtime-ready | HLD §12、§13、§18、CP3 review | adopted |
| lane-docs | HLD 必须显式说明与 CR046、CR020、strategy-runner-core 和 legacy QMT HLD 的关系 | HLD §1、§9、§14 | adopted |

## HLD 拆分判定

| 信号 | 判断 | 处理 |
|---|---|---|
| 核心产物 > 1 | Runner Control Plane 与 Gateway Service Layer 两个核心产物 | 当前保留单份 umbrella HLD，因为共享事件契约、审计 ID、授权边界和场景模拟；Story planning 阶段再按 Feature 拆分。 |
| Story 数量超阈 | 预计 8 个 Story，按 Runner/Gateway/Shared 分组明显 | 记录 `resolved-with-risk`；CP3 人工确认时列为风险接受项。 |
| ADR 分簇 | ADR 可按 Runner、Gateway、Shared Safety 分簇 | 当前 HLD 统一记录 ADR-CR138-001..007，后续 CP4/CP5 可拆 Feature 设计。 |
| 评审者差异 | 架构、安全、运营均需关注 | CP3 Decision Brief 将拆分风险列为 DQ-CP3-CR138-04。 |

结论：本轮保留单份 Runner / QMT Gateway Operational Control Plane umbrella HLD；后续 Story planning 必须按 `FEAT-11 Runner Control Plane`、`FEAT-12 QMT Gateway Service Layer`、`FEAT-07 安全授权`、`FEAT-06 OMS / 风控` 拆分。

## Deferred Architecture Ideas

| ID | 想法 / 风险 / 扩展方向 | 延后原因 | 触发切换或重启条件 |
|---|---|---|---|
| DAI-138-01 | SSE/WebSocket 实时推送、gRPC 正式 SDK、FIX 北向桥接 | 当前 CP3 只冻结 REST-only 服务边界和事件模型；实时推送与机构级多协议后置 | Gateway P0 REST 模型和安全授权通过 CP7 后另起协议 CR。 |
| DAI-138-02 | 多账户 / 多券商 / 多 QMT 实例聚合 | 需要真实账户、实例和授权事实；当前不能推断 | 用户明确提供账户 / 实例边界并授权 runtime Spike。 |
| DAI-138-03 | 完整回测 / 实盘同构 runtime | 当前已有研究和 Backtrader optional reference，不应在 Gateway HLD 中重做平台 | Runner P0 事件模型稳定后另起 research-to-runtime consistency CR。 |
| DAI-138-04 | 外部项目源码级 clone / run Spike | 两份研究报告已足够支撑 CP3；当前无 clone/run 授权 | HLD 评审发现关键接口事实不足，且用户批准临时目录 clone。 |

## 结论

Architecture Gray Areas 已收敛。推荐方案为：独立 Runner Control Plane + 独立 QMT Gateway Service Layer，通过领域命令、事件、授权记录和审计 ID 解耦；Gateway P0 使用 REST-only；CP3 approval 不自动授权 runtime、QMT / MiniQMT / XtQuant、凭据、账户、行情、订单读取、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入，后续必要验证可按需走 `runtime_authorization` gate。
