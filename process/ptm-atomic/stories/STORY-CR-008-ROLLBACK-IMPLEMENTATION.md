# STORY-CR008 Rollback Contract Implementation Evidence

| 字段 | 值 |
|---|---|
| story_scope | CR-008 all implementation stories |
| implemented_by | host-orchestrator-inline |
| implementation_date | 2026-06-25 |
| subagent_dispatch | N/A: user did not explicitly request parallel subagents; current implementation completed inline. |

## 实现对象清单

| 对象 | 文件 |
|---|---|
| Rollback 规则模块 | `src/atomic_ops/rollback.py` |
| CLI 接入 | `src/atomic_ops/commands/validate.py`, `src/atomic_ops/commands/rollback.py`, `src/atomic_ops/cli.py` |
| list JSON 摘要 | `src/atomic_ops/repository.py`, `src/atomic_ops/commands/list_ops.py` |
| run envelope metadata | `src/atomic_ops/commands/run.py` |
| schema | `schemas/atomic-op.schema.yaml` |
| 首批 atom 契约 | `atoms/fw/fw_config_policy_route.yaml`, `atoms/fw/fw_update_policy_route.yaml`, `atoms/fw/fw_delete_policy_route.yaml`, `atoms/fw/fw_verify_policy_route.yaml`, `atoms/fw/fw_update_interface.yaml`, `atoms/fw/fw_verify_interface.yaml`, `atoms/tg/tg_start_traffic_stream.yaml`, `atoms/tg/tg_stop_traffic_stream.yaml`, `atoms/tg/tg_capture_traffic_result.yaml`, `atoms/tg/tg_verify_traffic_loss.yaml` |
| 公开文档 | `README.md`, `docs/schema-field-reference.md` |
| 测试 | `tests/test_runner.py` |

## 设计契约映射

| 需求 | 实现位置 | 验证 |
|---|---|---|
| side_effect / rollback_strategy schema 字段 | `schemas/atomic-op.schema.yaml` | schema 全量 79 atom 通过。 |
| rollback 静态校验 | `atomic_ops.rollback.validate_rollback_contract` | 单测 `RollbackContractTests`。 |
| readiness JSON | `atomic_ops.rollback.readiness_report`, `commands/rollback.py` | 单测 `test_cli_rollback_check_blocks_irreversible_contract`。 |
| list JSON 暴露字段 | `OperationSummary` 新字段 | schema/单测间接覆盖，CLI 可读取缓存后输出。 |
| run rollback_context | `_to_run_envelope` + `run_rollback_context` | 单测 `test_wraps_rollback_context_when_contract_exists`。 |
| 策略路由回滚契约 | 4 个 policy route atom | 当前工作树 readiness 扫描，create/update/delete pass。 |
| 接口和流量契约 | interface/tg atom | 当前工作树 readiness 扫描，符合预期。 |

## 验证命令

```bash
uv run python -m unittest tests.test_runner
uv run python scripts/validate_schema.py --schema schemas/atomic-op.schema.yaml atoms
uv run python -c "<workspace Repository readiness scan for CR-008 targets>"
```

## 验证结果

| 命令 | 结果 |
|---|---|
| `uv run python -m unittest tests.test_runner` | PASS: 53 tests |
| `uv run python scripts/validate_schema.py --schema schemas/atomic-op.schema.yaml atoms` | PASS: files_checked=79 |
| workspace readiness scan | PASS for policy-route create/update/delete/verify, interface update/verify, traffic start/capture/verify; `tg_stop_traffic_stream` blocked as `manual_required` by design |

## 剩余风险

| 风险 | 状态 | 处理 |
|---|---|---|
| 默认 cache 未同步时 CLI 读取旧 atom | known | README 已说明 read commands 读本地 cache；发布后需 `ptm-atomic sync`。 |
| 全量 atom 尚未全部业务化 rollback | open | 后续按 package 补齐，ptm-te 真实执行前由 readiness 阻断。 |
| create 后真实对象 ID 解析不足 | open | 后续按 runner/device response 增强。 |
