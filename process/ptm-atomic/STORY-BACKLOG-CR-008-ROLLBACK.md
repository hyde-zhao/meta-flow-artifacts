# CR-008 Story Backlog: Rollback Contract Baseline

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.0 | 2026-06-25 | host-orchestrator-inline | 首批 rollback contract Story 拆解与完成状态。 |

## Stories

| Story ID | 用户结果 | 范围 | 状态 | 验收 |
|---|---|---|---|---|
| STORY-CR008-001 | 作为 ptm-te，我能读取每个 op 的 side-effect 与 rollback 摘要。 | schema 字段、repository summary、list/show JSON | done | list JSON 暴露 `side_effect` 和 `rollback_strategy`。 |
| STORY-CR008-002 | 作为 ptm-te，我能在执行前校验 rollback 契约缺口。 | `validate --check rollback`、规则模块、单测 | done | 缺 side_effect、缺 strategy、缺 snapshot、引用不存在均可产生 finding。 |
| STORY-CR008-003 | 作为 ptm-te，我能对单 op 或 plan 查询 readiness。 | `rollback check --op/--plan`、JSON/table 输出 | done | readiness 输出 `pass/blocked`、missing、warnings。 |
| STORY-CR008-004 | 作为测试工程师，我能对策略路由 create/update/delete 执行回滚准备检查。 | `fw_config_policy_route`、`fw_update_policy_route`、`fw_delete_policy_route`、`fw_verify_policy_route` | done | create/update/delete readiness pass；verify 为 observation。 |
| STORY-CR008-005 | 作为测试工程师，我能对接口变更和流量运行态做回滚准备检查。 | `fw_update_interface`、`fw_verify_interface`、`tg_start/stop/capture/verify` | done | update/start readiness pass；stop manual_required blocked；query类 observation。 |
| STORY-CR008-006 | 作为调用方，我能从执行结果拿到 rollback_context。 | run envelope metadata 注入 | done | `_to_run_envelope` 单测覆盖 `metadata.rollback_context`。 |

## 后续 Backlog

| ID | 内容 | 触发条件 |
|---|---|---|
| CR008-BL-001 | 全量 atom 按 package 补齐 rollback contract。 | ptm-te 准备真实执行对应 package 前。 |
| CR008-BL-002 | runner live response 解析 create 后真实对象 ID。 | 设备响应字段稳定或 ptm-te 要求自动压栈无 placeholder。 |
| CR008-BL-003 | clear/reset 类 atom 模板和不可逆样例。 | 新增清零、清日志、清会话或 factory reset op 时。 |
