# CR-008 Requirements: ptm-atomic Rollback Contract Baseline

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.0 | 2026-06-25 | host-orchestrator-inline | 首版，从用户输入提取 rollback contract 基线需求。 |

## 来源上下文

ptm-te 会在真实物理用例和问题单回归中调用 ptm-atomic 修改环境。ptm-te 执行前会做 rollback-readiness 检查；状态变更 op 若无可执行回滚策略，默认不得真实执行。

## 结构化需求

| ID | 优先级 | 类型 | 需求 | 验收条件 | 来源 |
|---|---|---|---|---|---|
| REQ-RB-001 | HIGH | schema | 每个 op 必须声明 `side_effect`。 | Given 任一 atom, When 执行 `ptm-atomic validate --check rollback`, Then 缺少 `side_effect` 的 op 产生 error finding。 | 用户 §1 |
| REQ-RB-002 | HIGH | schema | `traffic_runtime`、`state_mutation`、`destructive`、`irreversible` 必须声明 `rollback_strategy`。 | 状态变更 op 缺 rollback 时 validate 失败，非变更 `none/observation` 不要求 rollback。 | 用户 §1 |
| REQ-RB-003 | HIGH | cli | `list/show --format json` 暴露 rollback 元数据。 | `list --format json` 含 `side_effect` 与 rollback 摘要；`show` 返回完整 atom 字段。 | 用户 §1 |
| REQ-RB-004 | HIGH | operation | create 类 op 必须有 delete inverse op 或等价策略，并返回可删除对象标识和是否本次创建。 | `fw_config_policy_route` 声明 `inverse_op=fw_delete_policy_route`，`required_inputs=[policy_route_id]`，返回 `policy_route_id`、`created`。 | 用户 §2 |
| REQ-RB-005 | HIGH | operation | update 类 op 必须支持旧值快照恢复。 | `fw_update_policy_route` 和 `fw_update_interface` 声明 `restore_snapshot`、`pre_snapshot`、`rollback_apply` 和 `post_verify`。 | 用户 §2 |
| REQ-RB-006 | HIGH | operation | delete 类 op 必须支持删除前快照和重建；不可恢复时显式 irreversible。 | `fw_delete_policy_route` 通过 `fw_verify_policy_route` 捕获 `full_config`，用 `fw_config_policy_route` 重建。 | 用户 §2 |
| REQ-RB-007 | MEDIUM | operation | enable/disable 类 op 必须恢复原状态，不简单反向切换。 | `fw_update_interface` 的 rollback 输入为 `pre_snapshot.full_config`，包含 `admin_status` 与 `link_config`。 | 用户补充 |
| REQ-RB-008 | HIGH | operation | traffic runtime start 类 op 必须提供 stop/cleanup。 | `tg_start_traffic_stream` 声明 `inverse_op=tg_stop_traffic_stream` 并返回 `stream_id`。 | 用户 §2 |
| REQ-RB-009 | HIGH | operation | clear/reset 类不可逆动作必须显式声明 irreversible 并建议提供 pre_snapshot。 | rollback 规则要求 irreversible strategy 写 `notes`，readiness 返回 `blocked`。 | 用户 §2 |
| REQ-RB-010 | HIGH | schema | schema 支持 rollback 字段。 | `scripts/validate_schema.py --schema schemas/atomic-op.schema.yaml atoms` 通过。 | 用户 §3 |
| REQ-RB-011 | HIGH | cli | 新增 `validate --check rollback`。 | 能检查 side_effect 缺失、rollback 缺失、inverse 引用不存在、snapshot 缺失、required_inputs 不可得、irreversible 无 notes。 | 用户 §3 |
| REQ-RB-012 | HIGH | cli | 新增 rollback readiness 查询能力。 | `rollback check --op` 和 `rollback check --plan` 返回 `status`、strategy、missing、warnings。 | 用户 §3 |
| REQ-RB-013 | MEDIUM | run-output | 执行结果包含可回滚上下文。 | 成功 envelope 在可读取 atom 契约时包含 `metadata.rollback_context`。 | 用户 §4 |
| REQ-RB-014 | MEDIUM | error | 错误类型应能区分执行失败、快照失败、回滚失败和不可逆阻断。 | 本 CR 保留 schema/validation 层的不可逆阻断；runner 具体错误扩展作为后续 backlog。 | 用户 §4 |

## 未覆盖 / 后续项

| ID | 状态 | 说明 |
|---|---|---|
| RB-FU-001 | OPEN | 全量 79 个 atom 仍需逐批补齐业务级自动回滚或 `manual_required/irreversible` 契约。 |
| RB-FU-002 | OPEN | clear hitcount / clear log / clear session / factory reset 类 atom 当前仓库未发现专门定义；新增时必须按 irreversible 规则建模。 |
| RB-FU-003 | OPEN | runner live response 目前不能保证从真实设备响应解析 create 后对象 ID；需要后续按设备响应结构增强。 |
