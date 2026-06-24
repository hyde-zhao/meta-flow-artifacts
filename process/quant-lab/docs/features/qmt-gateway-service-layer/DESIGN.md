---
feature_id: "FEAT-12"
feature_name: "QMT Gateway Service Layer"
status: "ready-for-cp5-review"
version: "1.0"
change: "CR-138"
source_hld: "process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md"
created_at: "2026-06-24T16:10:00+08:00"
created_by: "host-orchestrator"
---

# FEAT-12 QMT Gateway Service Layer Design

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-24 | host-orchestrator | 按 CR138 CP3 approved HLD 生成 QMT Gateway Service Layer Feature 级实现设计，供 CP5 Story LLD 消费 |

## 目标与边界

FEAT-12 负责 QMT / MiniQMT / XtQuant 触达的服务层合同：lifecycle、health、capabilities、session、REST-only P0、reference/account query、market subscription、order command gate、execution report、recovery、audit 和 change plan。

它不生成策略信号、不计算目标组合、不发布策略版本、不决定策略准入、不拥有 offline runner bundle / registry authority，也不在 CP4/CP5 默认授权真实 runtime。

## 领域对象

| 对象 | 关键字段 | 状态 | 说明 |
|---|---|---|---|
| GatewayHealth | gateway_id、status、capabilities_ref、session_state、degraded_reason、last_heartbeat | not_started / healthy / degraded / cooldown / unavailable / disabled | health pass 不代表业务授权 |
| CapabilitySnapshot | protocols、query_scopes、market_scopes、order_scopes、runtime_mode | discovered / stale / blocked | P0 protocols 只承诺 REST |
| TradingSession | session_id、account_label、run_mode、scope、started_at、expires_at、redaction_status | missing / opening / ready / expired / revoked / blocked | account label 必须脱敏 |
| TradingCalendar | market、date_range、trading_days、holidays、source、source_freshness | available / stale / unavailable | 本地参考优先 |
| CommissionSchedule | account_label、instrument_type、commission_rate、min_fee、taxes、source | configured / broker_confirmed / estimated / unavailable | 不伪造 broker confirmed |
| PnLSnapshot | account_label、run_id、period、realized_pnl、unrealized_pnl、source、redaction_status | available / estimated / blocked / stale / unavailable | 账户级必须授权 |
| MarketSubscription | subscription_id、symbols、period、cache_policy、recovery_state | requested / active / degraded / recovering / stopped / blocked | P0 通过 REST 管理和拉取事件 |
| GatewayCommand | command_id、command_type、order_intent_id、scope、idempotency_key、risk_ref | accepted / hard_rejected / sent_by_future_runtime / blocked | 未授权 submit/cancel 时 adapter_calls=0 |
| ExecutionReport | execution_report_id、order_intent_id、broker_order_ref、state、filled_qty、error_code、audit_id | ack / partial / filled / canceled / rejected / unknown / stale | 可由 fixture / mock 表达 |
| GatewayAuditRecord | audit_id、request_id、run_id、scope、redaction_status、event_refs | recorded / redacted / blocked | 贯通 Runner review |
| GatewayChangePlan | change_id、config_diff、protocol_diff、auth_diff、compatibility_result、rollback_target | draft / review_ready / approved_by_followup / rejected | 配置不得含明文凭据 |

## REST-only P0 接口组

| 接口组 | 动作 | 输出 | 默认授权 |
|---|---|---|---|
| lifecycle | health、capabilities、session status | GatewayHealth / CapabilitySnapshot | static / mock only |
| query | trading calendar、commission schedule、cost estimate、PnL / return summary | Reference / Account Query result | calendar local；account read blocked unless authorized |
| subscription | register、stop、status、pull events | MarketSubscription / QuoteEvent refs | blocked unless market scope authorized |
| order | validate command、hard reject、future runtime send boundary | GatewayCommand / ExecutionReport | submit/cancel blocked |
| recovery | degraded state、cooldown、manual takeover plan | RecoveryPlan | static / mock only |
| audit | audit search、redacted event refs | GatewayAuditRecord | redacted summary only |
| change | config compatibility、rollback check | GatewayChangePlan | no secret material |

## 文件影响范围

| 类别 | 路径 | 说明 |
|---|---|---|
| primary | `trading/qmt_gateway_contracts.py` | GatewayHealth / CapabilitySnapshot / Query / Event / Command 合同 |
| primary | `trading/qmt_gateway_service.py` | Gateway service layer 入口，后续按 Story 增量扩展 |
| primary | `trading/qmt_gateway_gates.py` | scope / authorization / fail-closed gates |
| primary | `trading/qmt_gateway_config.py` | non-secret config / compatibility / rollback |
| shared | `trading/qmt_auth.py`、`trading/qmt_redaction.py` | 授权与脱敏复用 |
| shared | `trading/oms.py`、`trading/pretrade_risk.py`、`trading/reconciliation.py` | OMS / risk / execution report 协调 |
| tests | `tests/test_cr138_qmt_gateway_service_layer.py` | Gateway lifecycle / query / subscription / order / recovery fixture |
| forbidden | QMT runtime start/connect、真实 account/quote/order、submit/cancel、`.env` | 未授权时全部 blocked |

## 授权分层

| 能力 | 默认状态 | 可按需授权的 action scope |
|---|---|---|
| health / capabilities schema | fixture / static allowed | 无需 runtime |
| trading calendar 本地参考 | allowed if local source exists | vendor / QMT calendar 需说明来源 |
| commission configured / estimated | allowed with non-secret config | broker confirmed 需账户 / 券商资料授权 |
| account PnL / funds / positions / orders / fills | blocked | account_readonly |
| market subscription / quote pull | blocked | market_readonly |
| order submit / cancel | blocked | order_write / simulation / live 分项 |

## 失败路径

| 场景 | 输出 | 说明 |
|---|---|---|
| no runtime authorization | blocked | 不启动 Gateway，不连接 QMT |
| QMT 不支持佣金 / PnL 查询 | unavailable_with_reason | 不伪造 broker facts |
| session expired | blocked / manual_reauth_required | 不自动读取凭据 |
| subscription degraded | degraded / recovering | P0 不承诺 WebSocket 推送 |
| order scope missing | hard_rejected / adapter_calls=0 | 不下单撤单 |
| config no rollback | GatewayChangePlan.rejected | 不允许配置变更 |

## Story 映射

| Story | 设计消费点 | lld_policy |
|---|---|---|
| CR138-S01 | shared domain / auth / audit contract | full-lld |
| CR138-S05 | lifecycle / health / capabilities / REST P0 | full-lld |
| CR138-S06 | trading calendar / commission / PnL query service | full-lld |
| CR138-S07 | subscription / order report / recovery | full-lld |
| CR138-S08 | docs / runbook / CP7 guardrails | full-lld |

## Gotchas

- 不要把 REST endpoint 可见性写成 runtime ready；未授权时 endpoint 必须返回 blocked / unavailable，而不是伪成功。
- 不要把 local trading calendar 与 broker account facts 混成一个来源；calendar 可本地参考，账户级 PnL / commission 必须授权。
- 不要默认引入 SSE / WebSocket / gRPC / FIX；P0 是 REST-only，实时推送另起协议 CR。
- 不要让 Gateway 计算策略目标组合或决定策略发布；这属于 FEAT-11 / FEAT-03 / FEAT-09 上游。

