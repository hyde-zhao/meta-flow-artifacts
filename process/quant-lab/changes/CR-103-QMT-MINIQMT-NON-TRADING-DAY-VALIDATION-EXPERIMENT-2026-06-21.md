---
cr_id: "CR-103"
cr_kind: "runtime-authorization"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "runtime"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 QMT/MiniQMT/gateway/runtime 授权边界，必须保持 standard/runtime gate；本 CR 只允许非交易日、本地 fixture/static/dry-run 验证。"
rollback_to: "CR101 closed-current-delivery / READY_WITH_RISK; RA-CR101-001 and RA-CR101-002 candidate-not-started"
approval_result: "user-requested-non-trading-day-validation-experiment"
created_at: "2026-06-21T13:31:01+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-21T13:31:01+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-101"
source_checkpoint: "process/checkpoints/CP8-CR101-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR101-04"
follow_up_type: "runtime-validation-non-trading-day"
risk_class: "qmt-miniqmt-runtime-boundary"
owner: "host-orchestrator"
revisit_condition: "用户要求进入真实 QMT/MiniQMT/gateway runtime、读取 env/凭据/账户、query_positions、submit/cancel、simulation/live 或交易日验证。"
acceptance_criteria: "不启动 QMT/MiniQMT/XtQuant/gateway，不读取 env/凭据/账户，不访问真实账户/持仓/委托/成交，不执行 submit/cancel/simulation/live；完成本地 package/checker/manifest/adapter/gateway allowlist/HMAC/session/blocked gate/证据脱敏验证，且记录交易日剩余验证项。"
close_condition: "CP8 收口确认通过；用户于 2026-06-22 同意关闭 CR103。"
cp8_status: "approved"
cp8_auto_check: "process/checks/CP8-CR103-DELIVERY-READINESS.md"
cp8_manual_review: "process/checkpoints/CP8-CR103-DELIVERY-READINESS.md"
cp8_context: "process/context/CP8-CR103-DELIVERY-CONTEXT.yaml"
cp8_release_context: "process/release/RELEASE-CONTEXT-CR103.yaml"
cp8_approved_by: "user"
cp8_approved_at: "2026-06-22T11:55:02+08:00"
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-103 QMT/MiniQMT Non-Trading-Day Validation Experiment

## 变更描述

用户要求先启动一个验证实验，将除必须等交易日才能验证的 QMT/MiniQMT 内容尽可能完成验证；同时把交易日需要验证的内容另起 CR，计划在 2026-06-22 用户配合完成。

本 CR 只覆盖 **非交易日可验证** 的本地合同、fixture、checker、dry-run、fail-closed、allowlist、证据脱敏和安全边界。它不授权真实 QMT/MiniQMT/XtQuant/gateway runtime。

## CR 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 当前 active CR | PASS_WITH_RISK | CR102 仍 `cp8_pending`，但影响面是 NAS package exchange 收口；CR103 影响面是 QMT/MiniQMT 非 runtime 验证。 |
| RA-CR101-001 | CONVERTED_SCOPE | QMT direct-run candidate 的非 runtime 部分纳入 CR103；真实 QMT terminal 加载保留给 CR104。 |
| RA-CR101-002 | CONVERTED_SCOPE | MiniQMT readonly gateway adapter candidate 的非 runtime 部分纳入 CR103；真实 gateway/positions 保留给 CR104。 |
| FU-CR101-001 | NOT_STARTED | order-write / simulation / live 仍不启动；只在 CR104 中列为明日是否进入设计门禁的后续项。 |
| 权限边界 | PASS | 本轮不启动 runtime、不读 env/凭据/账户、不交易、不 provider/lake/catalog publish。 |

## 结构化权限策略

```yaml
authorization_policy:
  nas:
    access: false
    list: false
    read: false
    write: false
    publish: false
    delete: false
  credentials:
    env_read: false
    secret_read: false
    account_read: false
  runtime:
    qmt: false
    miniqmt: false
    xtquant: false
    gateway: false
  trading:
    submit: false
    cancel: false
    simulation: false
    live: false
  allowed_validation:
    static_contract_review: true
    local_fixture_tests: true
    local_checker_tests: true
    dry_run_only: true
    fake_transport_only: true
```

## 五维度影响分析

| 维度 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|
| 需求层 | CR101 后续验证路线 | true | 将 QMT/MiniQMT 非交易日验证从候选项中切出为 CR103。 |
| 场景层 | QMT direct-run / MiniQMT readonly adapter / gateway allowlist / evidence redaction | true | 输出 CR103 验证报告和 CP7 证据。 |
| 计划层 | CR102 cp8 pending、CR103 验证、CR104 明日交易日 gate | true | 允许 CR102 收口与 CR103 非 runtime 验证并行；CR104 不执行，等待 CP2/交易日。 |
| 安全层 | QMT/MiniQMT/gateway/runtime/credential/trading boundary | true | 明确全部真实 runtime 和交易动作 false。 |
| 交付层 | tests、docs/features、process/checks、CR tracking | true | 跑本地验证集，修正一个测试口径，记录报告。 |

## 本轮验证结果摘要

| 项 | 结论 | 证据 |
|---|---|---|
| qmt_interface_smoke package checker | PASS | `scripts/check_cr089_qmt_interface_smoke_package.py --package-root packages/qmt_interface_smoke/0.1.0` |
| QMT/MiniQMT offline regression suite | PASS | `125 passed in 0.69s` |
| 发现并修正的测试口径 | PASS_WITH_RISK | `tests/test_cr019_qmt_endpoint_matrix.py` 将 `positions` 的无 transport fail-closed 口径与 ADR-092 对齐。 |
| runtime / credential / trading safety | PASS | 本轮未启动 QMT/MiniQMT/XtQuant/gateway，未读 env/凭据/账户，未交易。 |

## 运行时风险复核

| 检查项 | 结论 | 说明 |
|---|---|---|
| DryRun / Fixture 模式 | PASS | 全部验证使用 local checker、fixture、fake/offline contracts。 |
| 执行隔离 | PASS | 只在当前项目本地 `uv run` 执行测试。 |
| 权限最小化 | PASS | 未使用 runtime、网络、凭据、真实账户、NAS 或交易权限。 |
| 超时保护 | PASS_WITH_RISK | 命令由当前会话控制，单次测试耗时 < 1s；未另设外层 timeout。 |
| 失败保护 | PASS | 首轮失败后未跳过，定位并修正测试口径后全量重跑。 |

## 后续事项

| 后续项 | 归属 | 状态 |
|---|---|---|
| 2026-06-22 交易日 QMT direct-run / MiniQMT readonly runtime gate | CR104 | created-blocked-cp2-pending |
| order-write / simulation / live 设计门禁 | FU-CR101-001 / CR104 后续 | not-started |

## 处理结论

CR103 非交易日验证实验已完成，结论为 `PASS_WITH_RISK`：非交易日可验证的 QMT/MiniQMT 合同、fixture、checker、allowlist、gateway gate、session gate、证据脱敏全部通过；真实 QMT/MiniQMT/gateway runtime 和交易日 query_positions / direct-run 加载仍需 CR104。
