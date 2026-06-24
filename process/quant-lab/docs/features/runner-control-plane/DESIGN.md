---
feature_id: "FEAT-11"
feature_name: "Runner Control Plane"
status: "ready-for-cp5-review"
version: "1.0"
change: "CR-138"
source_hld: "process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md"
created_at: "2026-06-24T16:10:00+08:00"
created_by: "host-orchestrator"
---

# FEAT-11 Runner Control Plane Design

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-24 | host-orchestrator | 按 CR138 CP3 approved HLD 生成 Runner Control Plane Feature 级实现设计，供 CP5 Story LLD 消费 |

## 目标与边界

FEAT-11 负责 Runner 运营控制面：运行计划、盘前确认、运行命令、事件 / 信号接入、再平衡计划、运行状态、证据查询、盘后复盘、异常恢复和策略生命周期变更。

它不负责 QMT / MiniQMT / XtQuant native 调用，不读取凭据、账户、行情或订单原始数据，不直接下单撤单，不拥有 Gateway lifecycle，也不覆盖 `process/docs/features/strategy-runner-core/DESIGN.md` 的 offline runner core implementation authority。

## 领域对象

| 对象 | Owner | 关键字段 | 状态 | 说明 |
|---|---|---|---|---|
| RunPlan | FEAT-11 | run_id、strategy_id、strategy_version、data_release、target_date、mode_request、authorization_ref | draft / preflight_ready / blocked / approved_for_manual_review | 从策略准入包和数据 release 生成，不触发 runtime |
| PreflightResult | FEAT-11 / FEAT-07 | run_id、admission_status、data_status、gateway_health_ref、auth_status、blocked_reasons | pass / warn / blocked / manual_review | health 不等于授权 |
| RunnerCommand | FEAT-11 | command_id、run_id、command_type、idempotency_key、authorization_ref、audit_id | accepted / duplicate / rejected / blocked | 只能转化为 GatewayCommand 或 OrderIntent，不直连 xtquant |
| RunState | FEAT-11 | run_id、state、active_orders、last_event_at、gateway_status、incident_refs | planned / preflight / running / degraded / paused / completed / blocked / manual_takeover | 消费 GatewayEvent / ExecutionReport |
| RunEvidence | FEAT-11 / FEAT-08 | run_id、evidence_refs、audit_ids、redaction_status、summary | indexed / redacted / unavailable / blocked | 只输出摘要和引用 |
| ReviewSummary | FEAT-11 / FEAT-08 | run_id、trade_date、metrics_summary、incidents、follow_up_candidates | draft / reviewed / follow_up_created | 盘后复盘入口 |
| StrategyChangePlan | FEAT-11 | strategy_id、change_type、parameter_diff、version_diff、rollback_target、approval_ref | draft / approved / applied_by_followup / blocked | 不直接发布策略到交易节点 |

## 接口契约

| 接口 | 调用方 | 输入 | 输出 | 失败行为 |
|---|---|---|---|---|
| build_run_plan | operator / CLI / future API | strategy_id、version、target_date、data_release_ref | RunPlan | 缺 strategy package 或 admission package 时 blocked |
| run_preflight | operator / scheduler | RunPlan、GatewayHealth、AuthorizationRecord | PreflightResult | 缺授权、数据或 gateway health 时 blocked / manual_review |
| submit_runner_command | operator / event router | RunnerCommand draft | RunnerCommand result | duplicate idempotency key 返回 duplicate；无授权返回 blocked |
| ingest_signal_event | event / signal source | event_id、schema_version、payload_ref | RunnerCommand 或 rejected | schema mismatch 或重复事件 reject / skip |
| build_rebalance_intent | portfolio context | target/current summary、risk assumptions | OrderIntentDraft / manual_review | risk missing 或 query unavailable 时 manual_review |
| query_run_evidence | operator / review | run_id、request_id | redacted RunEvidence | 触及 raw sensitive data 时 blocked |
| build_review_summary | operator / scheduler | run_id、period、evidence refs | ReviewSummary | unresolved incident 时 follow_up_candidate |
| propose_strategy_change | operator | parameter/version diff、rollback target | StrategyChangePlan | rollback target missing 时 blocked |

## 文件影响范围

| 类别 | 路径 | 说明 |
|---|---|---|
| primary | `trading/runner_control_plane.py` | 新 Runner 控制面服务与状态机，后续 Story 新增 |
| primary | `trading/runner_control_contracts.py` | RunPlan / PreflightResult / RunnerCommand / RunState 等合同，后续 Story 新增 |
| primary | `trading/runner_control_cli.py` | 运营控制 CLI，后续 Story 新增 |
| shared | `trading/strategy_runner/*` | 只读消费 artifact bundle、run registry、evidence index；不得覆盖 offline runner authority |
| shared | `trading/qmt_gateway_contracts.py`、`trading/qmt_gateway_service.py` | 只消费 GatewayHealth / GatewayEvent / ExecutionReport 合同 |
| shared | `trading/oms.py`、`trading/pretrade_risk.py`、`trading/kill_switch.py` | 通过 FEAT-06 进行 risk / order intent / kill switch 协调 |
| tests | `tests/test_cr138_runner_control_plane.py` | Runner contract / preflight / command / review fixture |
| forbidden | `.env`、QMT SDK、`xtquant`、真实账户 / 行情 / 订单 / 原始日志 | CP5/CP6/CP7 默认禁止 |

## 权限与安全

| 规则 | 处理 |
|---|---|
| Runner 不直连 QMT / XtQuant | 静态 guardrail：FEAT-11 文件 forbidden import count 必须为 0 |
| CP3 / CP4 不授权 runtime | 所有 Story 的 authorization_boundary 保持 false |
| 后续必要验证可按需授权 | 仅通过 runtime_authorization gate 打开具体 action scope |
| 账户 / 行情 / 订单数据 | Runner 只消费 Gateway redacted summary 或 evidence ref |
| 审计 | RunnerCommand、OrderIntent、ExecutionReport 必须保留 audit_id / request_id |

## 失败路径

| 场景 | 失败输出 | 回退 |
|---|---|---|
| 数据 release 缺失 | PreflightResult.blocked | 回到 FEAT-02 / FEAT-03 补数据或降级研究模式 |
| gateway health stale | PreflightResult.manual_review | 暂停运行计划，等待 Gateway 恢复 |
| authorization_ref 缺失 | RunnerCommand.blocked | 发起 runtime_authorization gate |
| signal 重复 | RunnerCommand.duplicate | 幂等跳过 |
| order state unknown | RunState.manual_takeover | kill switch 保持，人工接管 |
| evidence 包含敏感字段 | RunEvidence.blocked | redaction policy 修复后重试 |

## Story 映射

| Story | 设计消费点 | lld_policy |
|---|---|---|
| CR138-S01 | shared domain contract、authorization、audit id | full-lld |
| CR138-S02 | RunPlan / PreflightResult / RunnerCommand | full-lld |
| CR138-S03 | Event / signal / rebalance / run tracker | full-lld |
| CR138-S04 | Evidence / review / incident / change plan | full-lld |
| CR138-S08 | docs / runbook / CP7 guardrails | full-lld |

## Gotchas

- 不要把 `RunState.running` 写成真实交易运行证明；没有 runtime_authorization 时只能是 fixture / static / mock 级状态。
- 不要让 Runner 读取账户级 PnL、持仓、订单或行情原始数据；这些必须经过 FEAT-12 Query Service 和 FEAT-07 授权。
- 不要把 `strategy-runner-core` 的 offline run registry 当成 online session registry。
- 不要把 GatewayHealth pass 当作 account / quote / order / trading 授权。

