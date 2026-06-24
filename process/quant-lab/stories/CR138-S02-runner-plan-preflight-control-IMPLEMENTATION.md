---
story_id: "CR138-S02-runner-plan-preflight-control"
status: "implemented"
owner: "host-orchestrator"
implemented_at: "2026-06-24T16:17:40+08:00"
cp6: "PASS"
---

# CR138-S02 Implementation

## 实现对象

| 对象 | 路径 | 说明 |
|---|---|---|
| Runner control plane | `trading/runner_control_plane.py` | `build_run_plan`、`build_run_plan_batch`、`run_preflight`、`run_batch_preflight`、`submit_runner_command`。 |
| Runner CLI renderer | `trading/runner_control_cli.py` | 本地 preflight / summary 渲染，不触发 runtime。 |
| Tests | `tests/test_cr138_runner_plan_preflight_control.py` | RunPlan、batch preflight、缺授权、gateway degraded、command idempotency。 |

## 设计契约映射

| LLD 契约 | 实现位置 | 验证 |
|---|---|---|
| RunPlan / RunPlanBatch 本地构建 | `RunnerControlPlane.build_run_plan*` | `test_build_run_plan_and_batch_preflight` |
| Preflight 缺授权 / 缺数据 fail-closed | `run_preflight()` | `test_preflight_blocks_missing_authorization_and_missing_data_without_adapter_call` |
| Gateway degraded 进入 manual_review | `run_preflight()` | `test_gateway_degraded_preflight_enters_manual_review` |
| RunnerCommand 幂等 | `submit_runner_command()` | `test_runner_command_is_idempotent_and_scope_gated` |

## 验证结果

CR138 定向测试 PASS：48 passed。相关 runner / gateway 回归 PASS：35 passed。

## 不授权边界

Preflight pass 只表示本地 review / fixture ready，不表示 runtime ready，不授权真实 QMT 或交易操作。
