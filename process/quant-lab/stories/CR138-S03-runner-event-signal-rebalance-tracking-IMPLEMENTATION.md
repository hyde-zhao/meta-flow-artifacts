---
story_id: "CR138-S03-runner-event-signal-rebalance-tracking"
status: "implemented"
owner: "host-orchestrator"
implemented_at: "2026-06-24T16:17:40+08:00"
cp6: "PASS"
---

# CR138-S03 Implementation

## 实现对象

| 对象 | 路径 | 说明 |
|---|---|---|
| Event / state / rebalance | `trading/runner_control_plane.py` | `ingest_signal_event`、`build_rebalance_intent`、`update_run_state`、`build_ops_summary`、`build_batch_ops_summary`。 |
| CLI summary renderers | `trading/runner_control_cli.py` | `render_ops_summary`、`render_batch_ops_summary`。 |
| Tests | `tests/test_cr138_runner_event_signal_rebalance_tracking.py` | 幂等、draft-only、manual takeover、batch summary。 |

## 设计契约映射

| LLD 契约 | 实现位置 | 验证 |
|---|---|---|
| SignalEvent 幂等 | `ingest_signal_event()` | `test_signal_event_idempotency_and_run_state_tracking` |
| 风险失败只产生 manual_review | `build_rebalance_intent()` | `test_rebalance_risk_fail_produces_manual_review_without_order_submit` |
| 通过风险只生成 OrderIntentDraft | `build_rebalance_intent()` | `test_rebalance_pass_creates_order_intent_draft_only` |
| stale report -> manual_takeover | `update_run_state()` | `test_stale_report_enters_manual_takeover_and_batch_summary_stays_local` |

## 验证结果

CR138 定向测试 PASS：48 passed；`git diff --check` PASS。

## 不授权边界

OrderIntentDraft 不包含 broker order ref，`submit_allowed=false`，`cancel_allowed=false`；CLI summary 只渲染本地状态。
