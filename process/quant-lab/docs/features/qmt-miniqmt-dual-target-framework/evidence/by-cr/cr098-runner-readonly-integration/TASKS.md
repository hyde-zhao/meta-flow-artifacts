---
task_plan_id: "cr098-runner-readonly-integration"
cr_id: "CR-098"
status: "implemented-pending-cp7"
owner: "host-orchestrator"
created_at: "2026-06-19T12:07:32+08:00"
---

# CR098 Runner Readonly Integration Tasks

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-19 | host-orchestrator | 初版任务计划，按 adapter、evidence、tests、optional runtime evidence 分组。 |
| 0.2 | 2026-06-19 | host-orchestrator | CP6 完成 T01-T06；T07 evidence schema 字段已落地，T08 runtime checklist 留待 CP7 授权。 |

## Wave 1: Offline Contract

| Task ID | 任务 | 文件 | 完成标准 |
|---|---|---|---|
| CR098-T01 | 扩展 readonly gateway result / config contract | `trading/strategy_runner/readonly_gateway.py` | fake default 不变；real config 不读取 env |
| CR098-T02 | 增加 real transport injection 路径 | `trading/strategy_runner/readonly_gateway.py` | 可注入 `QmtClient` / transport；默认不打开 socket |
| CR098-T03 | 扩展 evidence summary | `trading/strategy_runner/evidence.py` | health / capabilities / positions summary 脱敏落地 |

## Wave 2: Tests And Guardrails

| Task ID | 任务 | 文件 | 完成标准 |
|---|---|---|---|
| CR098-T04 | 新增 CR098 contract tests | `tests/test_cr098_runner_readonly_integration.py` | 覆盖 fake default、allowlist、fixture transport |
| CR098-T05 | 新增 redaction / forbidden counter tests | `tests/test_cr098_runner_readonly_integration.py` | sensitive / raw payload blocked |
| CR098-T06 | 回归 CR091 tests | `tests/test_cr091_strategy_runner_contracts.py` | 既有离线 runner 合同不回退 |

## Wave 3: Runtime Evidence Preparation

| Task ID | 任务 | 文件 | 完成标准 |
|---|---|---|---|
| CR098-T07 | 定义 CR098 evidence 输出路径 / schema | docs / process evidence | `/home/hyde/.quant-lab/evidence/qmt/cr098/redacted/` 单一脱敏路径 |
| CR098-T08 | 准备 CP7 optional runtime smoke checklist | `process/checkpoints/CP7-*` 后续生成 | 逐 run 授权前不执行 |

## 禁止任务

- 不读取 `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template`。
- 不读取 Windows `.env`。
- 不启动 gateway。
- 不执行 runner runtime。
- 不访问 NAS。
- 不提交 / 撤销 / 买卖 / simulation / live。
