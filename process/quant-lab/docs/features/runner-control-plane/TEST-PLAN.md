---
feature_id: "FEAT-11"
feature_name: "Runner Control Plane"
status: "ready-for-cp5-review"
version: "1.0"
change: "CR-138"
created_at: "2026-06-24T16:10:00+08:00"
created_by: "host-orchestrator"
---

# FEAT-11 Runner Control Plane Test Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-24 | host-orchestrator | 初版测试计划，覆盖 Runner 控制面合同、授权、幂等、证据和异常恢复 |

## 测试策略

| 层级 | 范围 | 默认方式 | 禁止项 |
|---|---|---|---|
| 静态 guardrail | forbidden import、authorization false、敏感字段 | pytest / script | 不导入 xtquant、不读 `.env` |
| 单元测试 | RunPlan、PreflightResult、RunnerCommand、RunState | fixture | 不启动 Gateway |
| 合同测试 | RunnerCommand -> GatewayCommand / GatewayEvent refs | fake Gateway contract | 不触发真实 account / quote / order |
| 场景测试 | UC-33..UC-43 happy / failure path | fixture / event replay | 不下单撤单 |
| 文档审查 | runbook 不授权声明 | markdown guardrail | 不声明 runtime ready |

## 测试用例矩阵

| 测试 ID | Story | 场景 | 预期 |
|---|---|---|---|
| FEAT11-T01 | CR138-S02 | strategy admission + data release 完整 | RunPlan 进入 preflight_ready |
| FEAT11-T02 | CR138-S02 | authorization_ref 缺失 | PreflightResult.blocked |
| FEAT11-T03 | CR138-S02 | GatewayHealth healthy 但无 account auth | 不升级账户查询授权 |
| FEAT11-T04 | CR138-S03 | 重复 signal event | duplicate / idempotent skip |
| FEAT11-T05 | CR138-S03 | risk fail | adapter_calls=0，RunnerCommand.blocked |
| FEAT11-T06 | CR138-S04 | evidence query 请求 raw sensitive data | blocked / redacted |
| FEAT11-T07 | CR138-S04 | unknown order state | manual_takeover，不自动重发 |
| FEAT11-T08 | CR138-S04 | strategy rollback target missing | StrategyChangePlan.blocked |

## 验收命令候选

| 命令 | 触发 Story | 说明 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr138_runner_control_plane.py` | CR138-S02..S04 | 后续实现后运行 |
| `uv run --python 3.11 python -m py_compile trading/runner_control_contracts.py trading/runner_control_plane.py` | CR138-S02..S04 | 后续实现后运行 |
| `uv run --python 3.11 python scripts/check_process_artifact_hygiene.py --json` | CR138-S08 | 确认产物分类 |

## 手工 / 授权验证

本 Feature 默认不需要真实 QMT runtime。若 CP7 确认确需真实 readonly / market / account / order 验证，必须先生成 runtime_authorization gate，限定 action scope、运行窗口、账户脱敏、回滚方式和审计记录。

