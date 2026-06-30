---
status: "approved-cp3"
version: "1.1"
change: "CR-138"
source_hld: "process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md"
created_at: "2026-06-24T14:10:00+08:00"
created_by: "host-orchestrator"
---

# 架构决策记录：Runner / QMT Gateway Operational Control Plane

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-24 | host-orchestrator | 初版 ADR 候选，覆盖 Runner Control Plane、QMT Gateway Service Layer、交互契约、协议优先级、runtime 授权和 HLD 拆分策略 |
| 1.1 | 2026-06-24 | host-orchestrator | 根据用户 CP3 反馈修订：长期 ADR 文件采用功能域命名；Gateway P0 改为 REST-only；runtime policy 改为按需授权；新增交易日历、佣金和收益查询服务决策 |

## ADR-CR138-001：Runner Control Plane 独立于 offline strategy-runner-core

**状态**：Accepted by CP3

**决策**：新增 FEAT-11 Runner Control Plane。它只消费 offline runner core 的 bundle / evidence / registry 摘要、FEAT-09 策略包合同、FEAT-03 admission package 和 FEAT-04 order intent draft，不覆盖 `process/docs/features/strategy-runner-core/DESIGN.md` 的 implementation authority。

**理由**：UC-33..UC-43 描述的是日常运营路径，不是 RunSpec / bundle / registry 小能力。独立 FEAT-11 可避免 offline pass 被误读为 runtime-ready。

## ADR-CR138-002：QMT Gateway Service Layer 独立于只读准入和策略 target

**状态**：Accepted by CP3

**决策**：新增 FEAT-12 QMT Gateway Service Layer。它覆盖 lifecycle、health、capabilities、session、readonly query、market subscription、order command gate、execution report、recovery、audit 和 change plan。CR020 readonly gateway 和 CR046 QMT terminal / MiniQMT target 作为历史输入，不替代 FEAT-12。

**理由**：UC-44..UC-50 超出单一只读 endpoint 或策略包 target；两份 Gateway 研究报告均显示协议层和服务生命周期需要独立建模。

## ADR-CR138-003：Runner / Gateway 使用领域命令和事件契约衔接

**状态**：Accepted by CP3

**决策**：Runner 与 Gateway 之间只能通过 RunnerCommand、OrderIntent、GatewayCommand、GatewayEvent、ExecutionReport、GatewayHealth、AuditRecord、authorization_ref 和 idempotency_key 衔接；Runner 禁止直接调用 xtquant / QMT API。

**理由**：直接调用会绕过 Gateway capabilities、OMS/risk、授权和审计。事件契约可以支持盘中观察、回报乱序、恢复和问题定位。

## ADR-CR138-004：Gateway P0 协议采用 REST-only，实时推送和 SDK 协议后置

**状态**：Accepted by CP3

**决策**：P0 只承诺 REST 接口，覆盖同步查询、命令提交、订阅管理、状态拉取、事件拉取、审计查询和人工控制台操作。SSE / WebSocket 只作为实时推送增强候选；gRPC 和 FIX 后置到独立协议扩展 CR。

**理由**：当前 operational baseline 的 P0 目标是可控、可审计、可验证，而不是亚秒级事件推送或外部 SDK 流式消费。REST-only 可以解决盘前查询、运行控制、订单意图提交、订阅状态管理、事件轮询和审计查询；WebSocket / gRPC / FIX 会扩大运行态复杂度，应等明确实时性或外部系统接入需求后再打开。

## ADR-CR138-005：CP3 不自动产生 runtime authorization，但后续验证可以按需授权

**状态**：Accepted by CP3

**决策**：CP3 approval 只确认 HLD 和架构方向，不自动授权 runtime、QMT / MiniQMT / XtQuant、凭据、账户、行情、订单读取、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入。后续 Story 设计或验证如确有必要，可以按 action scope 走独立 `runtime_authorization` gate，最小化授权只读、交易日历、佣金、收益、行情订阅、订单仿真或真实交易等具体动作。

**理由**：用户已明确要求不要把不授权作为长期默认结论。正确边界是“CP3 不自动授权”而不是“后续永不授权”。真实运行或账户相关读取仍必须给出运行窗口、账户脱敏、回滚、审计和 per-run 授权。

## ADR-CR138-006：交易日历、佣金和收益查询归入 Gateway Query Service

**状态**：Accepted by CP3

**决策**：FEAT-12 新增 Reference / Account Query Service 能力，显式覆盖 TradingCalendar / TradingWindow、CommissionSchedule / CostEstimate、PnLSnapshot / ReturnSummary。交易日历优先使用本地参考数据；佣金优先以配置化 FeeSchedule 或券商确认资料为事实源；账户级收益 / PnL、账户佣金、资金、持仓、委托、成交等必须经过账户只读授权并脱敏。

**理由**：这些能力是 Runner 盘前 preflight、费用估算、盘后复盘和收益归因的基础，不能只笼统归到 readonly query。QMT / XtQuant 对佣金和收益的可查询性不一定稳定，因此设计必须支持 `unavailable_with_reason` 和本地估算结果，禁止伪造 broker 事实。

## ADR-CR138-007：当前保留单份 umbrella HLD，CP4 按 Feature 拆 Story

**状态**：Accepted by CP3

**决策**：本轮保留单份 `HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md`，因为 Runner 和 Gateway 共享事件契约、授权边界、审计 ID 和场景模拟；CP4 Story planning 必须按 FEAT-11 Runner、FEAT-12 Gateway、FEAT-07 Safety、FEAT-06 OMS/Risk 拆分。

**理由**：过早拆成两份 HLD 会重复 ADR 和上下游契约；但 Story 层不拆会造成文件 owner 冲突和 CP5 设计证据过大。
