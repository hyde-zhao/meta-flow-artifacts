---
story_id: "CR138-S02-runner-plan-preflight-control"
cr_id: "CR-138"
title: "Runner run plan, preflight, and command control"
status: "lld-ready-for-review"
priority: "P0"
wave: "CR138-W2-CONTROL-SHELL"
depends_on:
  - "CR138-S01-shared-contracts-authorization-audit"
dependency_type:
  - "contract"
feature_design_refs:
  - "process/docs/features/runner-control-plane/DESIGN.md"
  - "process/docs/features/runner-control-plane/TEST-PLAN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "state machine"
    - "authorization boundary"
    - "cross-feature input"
created_at: "2026-06-24T16:20:00+08:00"
created_by: "host-orchestrator"
---

# CR138-S02 Runner Run Plan, Preflight, and Command Control

## 用户价值

用户可以在盘前形成可审计的 RunPlan，并通过 PreflightResult 明确知道是否可以进入人工确认或后续授权，而不是把 offline run 成功误读成 runtime ready。

## 范围

- RunPlan、PreflightResult、RunnerCommand。
- RunPlanBatch / batch preflight，用于吸收 CR137 后续 offline batch run 计划。
- 盘前 data/admission/auth/gateway health 检查。
- Runner CLI / service shell 的命令合同。

## 非范围

- 不执行真实运行。
- 不查询真实账户、行情或订单。
- 不启动 Gateway 或 QMT。

## 验收标准

| AC | 标准 |
|---|---|
| AC-01 | RunPlan 包含 strategy_id、strategy_version、target_date、data_release、authorization_ref。 |
| AC-02 | RunPlanBatch 包含 batch_id、多个 RunPlan ref、batch policy 和 aggregate status，且不自动运行。 |
| AC-03 | 缺 admission、data release、gateway health 或 authorization_ref 时 PreflightResult / BatchPreflightResult 不为 pass。 |
| AC-04 | RunnerCommand 支持 accepted / duplicate / rejected / blocked。 |
| AC-05 | adapter_calls 在未授权路径为 0。 |

## 技术说明

需要 full LLD，重点说明 preflight gate、batch preflight、状态迁移、错误码、fixture 和未来 runtime_authorization 接入点。
