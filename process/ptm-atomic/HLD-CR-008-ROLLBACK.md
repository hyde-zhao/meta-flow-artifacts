# CR-008 HLD: Rollback Contract Baseline

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.0 | 2026-06-25 | host-orchestrator-inline | 首版 HLD，覆盖 rollback schema、CLI readiness、首批 atom 契约和运行结果上下文。 |

## 1. 问题定义

ptm-te 真实执行物理用例时，ptm-atomic op 可能创建、更新、删除配置或启动运行态流量。当前 atom schema 缺少统一 side-effect 分类和 rollback strategy，ptm-te 无法在执行前判断某个 op 是否可安全进入真实执行。失败后如果没有明确 rollback 契约，环境可能残留策略、接口、路由、流量或不可恢复的清零动作。

## 2. 成功标准

| ID | 标准 | 度量 |
|---|---|---|
| SC-001 | 首批 P1/P2 readiness 可判定 | 10 个首批 atom 中 9 个 readiness 无 error；`tg_stop_traffic_stream` 以 `manual_required` 返回 blocked。 |
| SC-002 | schema 兼容 | `scripts/validate_schema.py --schema schemas/atomic-op.schema.yaml atoms` 检查 79 个 atom，violations=0。 |
| SC-003 | CLI 校验可用 | 单测覆盖 `validate --check rollback` 和 `rollback check --op`，总测试 53 个通过。 |
| SC-004 | ptm-te 可消费 | readiness JSON 至少包含 `status`、`op_id`、`side_effect`、`rollback_strategy`、`rollback_op_id`、`missing`、`warnings`。 |

## 3. Architecture Gray Areas

| Gray Area | Options | Impact | Recommendation |
|---|---|---|---|
| 强制方式 | A. schema required；B. rollback validator required | A 会让历史 atom 全量失效；B 可逐批治理并让 ptm-te 阻断缺口 | 选 B：schema 支持字段，`validate --check rollback` 强制规则。 |
| 自动回滚声明 | A. 未覆盖 op 自动填 `manual_required`；B. 只为已理解 op 写契约，缺口由 validate 暴露 | A 可能掩盖真实风险；B 更保守但首批覆盖有限 | 选 B：不伪造可回滚能力。 |
| update 回滚粒度 | A. 只记录变更字段；B. 捕获完整旧配置 | A 无法恢复系统名、默认策略等初始值；B 快照较重但可恢复 | 选 B：`restore_snapshot` 必须捕获 `full_config` 或等价完整配置。 |
| run 输出上下文 | A. runner 强依赖设备响应解析；B. envelope 从 atom contract 注入静态 rollback_context | A 首批落地慢；B 能先给 ptm-te 稳定结构但部分输入为 placeholder | 选 B，后续增强真实 ID 解析。 |

## 4. 方案比较

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 只写文档契约 | 改动小 | ptm-te 无法自动阻断 | docs only | 不推荐 | 仅适合纯讨论阶段。 |
| B. schema + validator + readiness CLI + 首批 atom | 可被 ptm-te 直接消费；保守暴露缺口 | 需要后续逐批补 atom | schema, CLI, atoms, docs, tests | 推荐 | 当前 CR 使用；后续按 package 扩展。 |
| C. ptm-atomic 内置完整 rollback executor | 自动化程度最高 | 需要设备快照持久化、rollback stack、权限审批，范围过大 | runner, state store, ptm-te protocol | 暂缓 | 当 ptm-te 需要 ptm-atomic 独立执行 rollback stack 时再切换。 |

## 5. 推荐架构

```text
atoms/*.yaml
  side_effect / rollback_strategy / pre_snapshot / rollback_apply / post_verify
       |
       v
atomic_ops.rollback
  validate_rollback_contract()
  readiness_report()
  run_rollback_context()
       |
       +--> list/show JSON metadata
       +--> validate --check rollback
       +--> rollback check --op/--plan
       +--> run envelope metadata.rollback_context
```

## 6. 模块职责

| 模块 | 职责 | 非职责 |
|---|---|---|
| `schemas/atomic-op.schema.yaml` | 定义字段形状、枚举和对象结构 | 不负责跨文件 op_id 存在性和 required_inputs 可得性。 |
| `src/atomic_ops/rollback.py` | 唯一 rollback 规则源，输出 findings/readiness/runtime context | 不访问设备，不采集快照。 |
| `validate --check rollback` | 仓库级或 package 级静态契约校验 | 不决定人工风险接受。 |
| `rollback check` | ptm-te 执行前 readiness 查询 | 不执行 rollback，不修改环境。 |
| atom YAML | 声明业务语义和回滚路径 | 不保存生产快照或敏感数据。 |
| ptm-te | 按契约采集快照、维护 rollback stack、执行人工门禁 | 不从 ptm-atomic 推断未声明回滚能力。 |

## 7. 关键流程

### 7.1 执行前 readiness

1. ptm-te 收集执行计划 op_id。
2. 调用 `ptm-atomic rollback check --plan plan.yaml --format json`。
3. 若任一结果 `status=blocked`，真实执行停止。
4. 若 strategy 为 `restore_snapshot`，ptm-te 按 `pre_snapshot.op_id` 在正向 op 前采集快照。
5. 正向 op 成功后，ptm-te 用 `rollback_strategy` 与 `metadata.rollback_context` 压栈。

### 7.2 失败回滚

1. ptm-te 逆序弹出 rollback stack。
2. `inverse_op` 直接调用 rollback op。
3. `restore_snapshot` 从 snapshot 输入调用 `rollback_apply.op_id`。
4. 执行 `post_verify.op_id` 判断恢复状态。
5. rollback 失败时同时记录 primary failure 与 rollback failure。

## 8. 失败路径

| 条件 | 处理 |
|---|---|
| op 无 `side_effect` | validate error；readiness blocked。 |
| 状态变更 op 无 `rollback_strategy` | validate error；readiness blocked。 |
| `inverse_op` 指向不存在 op | validate error；readiness blocked。 |
| `restore_snapshot` 缺 `pre_snapshot` 或 `rollback_apply` | validate error；readiness blocked。 |
| `required_inputs` 无法从参数、返回或 snapshot 获得 | validate error；readiness blocked。 |
| `manual_required` | readiness blocked，需人工确认后由 ptm-te 决策是否继续。 |
| `irreversible` | readiness blocked，默认禁止真实执行。 |

## 9. ADR 候选

| ADR | 决策 | 理由 |
|---|---|---|
| ADR-CR008-001 | rollback 强制规则放在 validator，不把字段设为 schema required | 保持历史 atom schema 兼容，同时让 ptm-te 能阻断缺口。 |
| ADR-CR008-002 | update/delete 使用完整快照恢复 | 支持系统名称、默认策略等“修改回初始值”场景，避免简单反向 op 误恢复。 |
| ADR-CR008-003 | readiness 对 `manual_required/irreversible` 返回 blocked | 与 ptm-te 默认阻断规则一致。 |

## 10. 分阶段落地

| 阶段 | 内容 | 状态 |
|---|---|---|
| P0 | schema、list/show、validate rollback、rollback check | completed |
| P1 | 策略路由 create/update/delete/verify rollback 契约 | completed |
| P2 | 接口 update/verify、流量 start/stop/capture/verify rollback 契约 | completed |
| P3 | clear/reset 不可逆类模板与现有 op 扫描 | partial：规则已支持，当前未发现专门 clear/reset atom |
| P4 | 全量 atom 按 package 逐批补齐自动或人工 rollback 契约 | backlog |

## 11. 自审

| 维度 | 结果 |
|---|---|
| 内部一致性 | PASS：schema、validator、readiness 和 atom 字段使用同一枚举。 |
| 目标量化 | PASS：成功标准含 atom 数、测试数、schema 文件数和 JSON 字段集。 |
| 集成契约 | PASS：ptm-te 调用 `rollback check`，ptm-atomic 只声明和校验，不维护 rollback stack。 |
| 相邻边界 | PASS：ptm-te 负责编排，ptm-atomic 负责原子契约。 |
| 失败路径 | PASS：缺字段、缺 op、缺 snapshot、manual、irreversible 均有行为。 |
