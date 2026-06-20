---
tracking_id: "CR098-FOLLOW-UP-TRACKING"
source_cr: "CR-098"
source_checkpoint: "process/checkpoints/CP8-CR098-DELIVERY-READINESS.md"
status: "active-follow-up-tracking"
created_at: "2026-06-19T12:31:24+08:00"
updated_at: "2026-06-20T10:45:00+08:00"
owner: "host-orchestrator"
---

# CR098 Follow-up Tracking

## 分类摘要

| 分类 | 数量 | 是否阻塞当前交付 | 说明 |
|---|---:|---|---|
| 关闭范围 | 1 | 否 | CR098 当前 runner readonly integration 离线交付已关闭为 `READY_WITH_RISK` |
| 不授权范围 | 1 | 否 | HMAC secret、Windows `.env`、gateway 启动、runner runtime、账户原文、NAS、交易写、simulation/live、provider/lake/publish 均不授权 |
| 风险接受项 | 3 | 否 | 真实 runner runtime 未执行、非空持仓 / 交易日路径未证明、CP6/CP7 inline fallback WAIVED 已由 CP8 接受 |
| 后续 CR 候选项 | 5 | 否 | `CR098-FU-01` 已转入正式 `CR-099` 并关闭；`CR091-FU-02` 已转入正式 `CR-100` 并关闭；`CR091-FU-05` 已转入正式 `CR-101` 并关闭；订单写入后续门禁迁移为 `FU-CR101-001` |

## 结构化候选项

```yaml
follow_up_items:
  - id: RA-CR098-001
    legacy_ids:
      - CR098-FU-01
      - CR-099
    title: Runner real readonly smoke per-run authorization
    kind: runtime-authorization
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: runtime
    formal_cr_path: process/changes/CR-099-QMT-RUNNER-REAL-READONLY-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: closed
  - id: RA-CR097-001
    legacy_ids:
      - CR097-FU-01
    title: Non-empty / trading-day readonly retest
    kind: runtime-authorization
    lifecycle_status: candidate
    readiness_status: n/a
    gate_status: not_started
    gate_profile: runtime
    formal_cr_path: ""
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: wait_for_user_selection
  - id: FU-CR091-005
    legacy_ids:
      - CR091-FU-05
    title: Cross-platform strategy delivery and adapter layer realignment
    kind: architecture-realignment
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: standard
    formal_cr_path: process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: reframed
    reframed_by: FU-CR091-005
    reframe_summary: Quant-lab owns the runner; MiniQMT is the current adapter target, while strategy delivery keeps a cross-platform target contract with QMT direct-run supported now.
    next_action: CR101 closed-current-delivery / READY_WITH_RISK; follow-up real validation gates require independent authorization.
  - id: FU-CR091-002
    legacy_ids:
      - CR091-FU-02
      - CR-100
    title: NAS package exchange gate
    kind: implementation-gate
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: compact
    formal_cr_path: process/changes/CR-100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-GATE-2026-06-19.md
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: real_nas_gate_requires_new_authorization
  - id: FU-CR091-003
    legacy_ids:
      - ORDER-WRITE-FU
      - CR091-FU-03
    title: Order-write / simulation / live design gate
    kind: runtime-authorization
    lifecycle_status: superseded
    readiness_status: n/a
    gate_status: not_started
    gate_profile: standard
    formal_cr_path: ""
    superseded_by: FU-CR101-001
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: use_FU-CR101-001_as_current_candidate_key
  - id: FU-CR101-001
    legacy_ids:
      - ORDER-WRITE-FU
      - ORDER-WRITE-SIMULATION-LIVE-FU
      - CR091-FU-03
    title: Order-write / simulation / live design authorization gate
    kind: runtime-authorization
    lifecycle_status: candidate
    readiness_status: n/a
    gate_status: not_started
    gate_profile: standard
    formal_cr_path: ""
    blocked_by:
      - RA-CR101-001
      - RA-CR101-002
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: not_recommended_until_readonly_validation_gates_are_settled
```

## 后续 CR / Spike 候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR-099 | Runner real readonly smoke per-run authorization | closed-current-delivery | CR | 1 | runtime_authorization; qmt_runner; readonly_gateway; hmac_client_env; redacted_evidence; credential_boundary | `process/changes/CR-099-QMT-RUNNER-REAL-READONLY-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md` | closed_by=CP8-CR099 | closed | CP2/CP3/CP5 已由用户回复“同意”批准，CP6 PASS；用户刷新 Windows gateway session 后，CR099 runtime rerun PASS，health/capabilities/query_positions_readonly 均通过，forbidden counters 全 0；用户接受 CP8 5 项推荐方案和风险 | 当前交付已关闭为 `READY_WITH_RISK`；非空持仓 / 交易日复测仍作为独立 follow-up 风险处理；不自动授权新增 runtime / 下单撤单 / Windows `.env` / NAS / publish / 交易写 | CR098-FU-01 / DQ-CP8-CR098-02 |
| CR097-FU-01 | Non-empty / trading-day readonly retest | candidate-not-started | CR | 2 | runtime_authorization; readonly_query_positions; evidence_coverage; non_empty_positions; trading_day_path |  |  | 未启动 | 需要非空持仓或交易日条件，且必须独立逐 run 授权 | 等待用户选择 | DQ-CP8-CR098-03 / DQ-CP8-CR097-02 |
| CR091-FU-05 | Cross-platform strategy delivery and adapter layer realignment | closed-current-delivery | CR | 1 | strategy_package_contract; cross_platform_delivery_target; qmt_direct_run_target; quant_lab_runner_adapter_layer; miniqmt_gateway_adapter; goldminer_future_adapter | `process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md` | closed_by=CR-101 | closed | 用户已批准 CR101 CP8；CR101 当前离线交付关闭为 READY_WITH_RISK。真实系统仍未授权或验证。 | 后续真实 QMT direct-run、MiniQMT gateway、NAS real exchange、order-write / simulation / live 均需独立 candidate gate，不自动启动。 | USER-20260619-RUNNER-ADAPTER-AND-CROSS-PLATFORM-DELIVERY / DQ-CP8-CR101-04 |
| CR091-FU-02 | NAS package exchange gate | closed-current-delivery | CR | 3 | nas_package_exchange; package_delivery; credential_boundary; no_runtime_connection | `process/changes/CR-100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-GATE-2026-06-19.md` | closed_by=CR100 | closed | CR100 已关闭为 READY_WITH_RISK；真实 NAS 仍未授权 | 如需真实 NAS publish / pull / copy / 校验，必须另起独立授权 gate | CR091 / CR098 not-authorized boundary |
| FU-CR101-001 | Order-write / simulation / live design authorization gate | candidate | CR | 4 | order_write; submit_cancel; simulation_live; trading_runtime_boundary; account_permission |  | legacy=ORDER-WRITE-FU / ORDER-WRITE-SIMULATION-LIVE-FU / CR091-FU-03 | 未启动 | 交易写风险最高，且应等待 adapter protocol、QMT direct-run 和 MiniQMT gateway readonly evidence 稳定 | 不建议立即启动；需要用户明确提出并接受更高风险门禁 | CR098 not-authorized boundary / DQ-CP8-CR101-04 |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 | 来源 |
|---|---|---|---|---|
| NA-CR098-01 | HMAC secret、Windows `.env`、gateway 启动、runner runtime、账户原文、NAS、交易写、simulation/live、provider/lake/publish、真实 release execution / publish、CR089 auto-start、CR020 gateway route restore | CR098 CP8 approve 只关闭当前离线 runner readonly integration 交付，不授权外部动作 | 另起独立 CR，并在该 CR 内获得明确逐 run 授权 | CP8-CR098 |

## 风险接受项

| 风险 ID | 状态 | 内容 | 后续处理 | 来源 |
|---|---|---|---|---|
| R-CR098-01 | accepted | 真实 runner runtime smoke 未执行 | 需要真实证明时启动 `CR098-FU-01` | CP8-CR098 |
| R-CR098-02 | accepted | 非空持仓 / 交易日路径未证明 | 需要更广覆盖时启动 `CR097-FU-01` 或在 `CR098-FU-01` 中纳入条件说明 | CP8-CR098 |
| R-CR098-03 | accepted | CP6/CP7 使用 host-orchestrator inline fallback，未通过独立子 agent lane | 后续高风险 runtime / order-write gate 建议补独立 review lane | CP8-CR098 |

## 推荐下一步

清理上下文后，推荐先读取 `process/context/CR098-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-19.md` 和 `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml`。如果用户明确要求继续推进，必须先确认候选目标：非空 / 交易日 readonly retest 仍是 `RA-CR097-001`；CR101 后续真实 QMT direct-run、MiniQMT gateway、NAS exchange 和 order-write gate 分别是 `RA-CR101-001`、`RA-CR101-002`、`RA-CR101-003`、`FU-CR101-001`。任何启动都不继承 CR098 / CR101 approve，也不自动授权读取 secret/env、启动 runtime、访问 NAS、查询账户原文或执行交易写。
