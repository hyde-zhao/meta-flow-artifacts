# ptm-te 步骤间 id 传递对齐 4 模式 — 实现计划

## 目标
ptm-te 经 `ptm-atomic show <op_id> --format json` 读 op 声明，按 `rollback_strategy.id_source` 四模式（response/args/query/placeholder）确定性解析「步骤间」与「回滚」id，替代当前 LLM 手动提取 + `_extract_inverse_id` 硬编码（line 1009-1028，仅支持模式 A）。

## 核心设计：一个 resolve_id，两个消费者

### 新增 `_load_op_decl(op_id) -> Optional[dict]`
- subprocess 调 `ptm-atomic show <op_id> --format json`，`json.loads` 解析
- 模块级 LRU `_OP_DECL_CACHE: Dict[str, dict]`（声明静态，安全缓存）
- 失败返回 None（op 不存在 / CLI 异常）

### 新增 `resolve_id(forward_op_id, forward_envelope, forward_args, *, base_url, session_file, authorized, timeout) -> Optional[Any]`
按 `decl["rollback_strategy"]["id_source"]` 分发：

| id_source | 动作 |
|---|---|
| response | `forward_envelope.data[id_field]`（跳过 None/""/0） |
| args | `forward_args[id_field]` |
| query | `match_val=forward_args[query_match]`；`_resolve_query_id(query_op, query_match, match_val, id_field, ...)`：execute_op 调 query_op，在结果 `data.full_config` 列表找 `item[query_match]==match_val`，返回 `item[id_field]` |
| placeholder | `forward_inputs.params["id"]`（如 "1"） |

### 消费者 1：回滚 `handle_rollback`（line 1031）
- inverse_op 分支先 `_load_op_decl(op_id)` 取 `rollback_strategy`
- 有声明：`id = resolve_id(...)`；`build_inverse_args()` 构造 inverse_args（mode D 互换 `old_name↔new_name`）
- 无声明（verify/login/update/delete-as-cleanup）：回退现有 `ROLLBACK_STRATEGY` 表（restore_snapshot/irreversible/none/as_cleanup_skip 不变）

### 消费者 2：步骤间传递 `${STEP-N.id}`
- `execute_op` 增参 `step_id: str=""`、`step_refs_dir: Optional[str]=None`
- 执行后写 `step_refs_dir/<step_id>.json` = `{step_id, op_id, args, envelope, resolved_id}`
- `build_command` 前调 `resolve_step_refs(args, step_refs_dir)`：扫描 args 值中的 `${STEP-N.id}`（及 `${STEP-N.<field>}`），读 `step_refs_dir/STEP-N.json` + 该步 op 声明，调 `resolve_id` 解析替换
- `step_refs_dir=None` 时原样返回（向后兼容，LLM 仍可手动传 id）

## 改动清单

### A. `skills/policy-route-execution/scripts/op_mapper.py`
1. 新增 `_OP_DECL_CACHE` + `_load_op_decl(op_id)`
2. 新增 `resolve_id(...)`（4 模式分发）+ `_resolve_query_id(...)`（query 模式）
3. 新增 `build_inverse_args(forward_op_id, forward_envelope, forward_args, decl, ...)`（4 模式 inverse_args；mode D 互换）
4. 改 `execute_op`：加 `step_id`/`step_refs_dir` 参数；前置 `resolve_step_refs`；后置 `_write_step_ref`
5. 新增 `resolve_step_refs(args, step_refs_dir)` + `_write_step_ref(...)`
6. 改 `handle_rollback` inverse_op 分支：声明优先，回退 ROLLBACK_STRATEGY
7. `ROLLBACK_STRATEGY` 表：`fw_config_object` 从 `none`（line 276，"待升级"）翻转为 inverse_op（ptm-atomic 已补 object delete）；新增 acl_policy / acl_policy_group 注释指向 `show` 声明；保留 update/delete/verify/login/reset 的非声明条目
8. `_extract_inverse_id` 降级为 `resolve_id(response 模式)` 内部实现或 deprecated wrapper（保留测试兼容）
9. 扩展 `validate_mapping_consistency`：config op 的 inverse_op 与 `OP_METADATA.rollback` 交叉校验；id_source 合法性校验

### B. `agents/ptm-te.md`
- [4] 步骤执行：step 间用 `${STEP-N.id}`，execute_op 传 `step_id` + `step_refs_dir`
- [6] exec-log：补 `step-refs/` 产物
- 产物表加 `runs/<run-id>/step-refs/<step_id>.json`
- PC `case_steps` 示例 args 改用 `${STEP-001.id}`

### C. `skills/policy-route-execution/SKILL.md`
- 回滚策略表补 4 模式（response/args/query/placeholder）
- 注：id 来源经 `ptm-atomic show` 声明解析

### D. `docs/ptm-te/执行指导.md`
- §4.4（line 244）：`<policy_route_id-from-config:STEP-N>` → `${STEP-N.id}`（声明驱动，自动按 id_source 取）
- §4.4 补 mode C（acl_policy：update/delete 需 verify 查询解析 id，resolver 自动处理）
- §4.4 补 mode D（policy_group rename：rollback 自动互换 old_name/new_name）
- Gotcha：acl_policy id 须 query 解析（设备 create 不返回 id）

### E. 测试
- 扩展 `tests/test_op_mapper_rollback.py`：
  - `resolve_id` 4 模式单测（A response / B args / C query mock / D placeholder）
  - `handle_rollback` 4 模式（C mock execute_op query；D 验证 arg 互换）
  - 回归：policy_route rollback / restore_snapshot / irreversible / none 不变
- 新增 `tests/test_step_refs.py`：
  - `resolve_step_refs`：`${STEP-001.id}` 插值（mock step_refs 文件）
  - step-refs 落盘测试

## 4 模式 inverse_args 规格
| id_source | inverse_args（rollback） |
|---|---|
| response | `{id, type?}` |
| args | `{id}` |
| query | `{id, policy_type}` |
| placeholder | `{id:"1", old_name=fwd.new_name, new_name=fwd.old_name, policy_type}` |

## 向后兼容
- `execute_op` 新参默认值不破坏现有调用
- `${STEP-N.id}` 仅在 args 含该模式时触发；LLM 仍可手动传 id（文档推荐占位符）
- `handle_rollback`：有 `show` 声明走新路径，无则回退 ROLLBACK_STRATEGY（老 op 行为不变）

## 风险与待验证
1. mode C `data.full_config` 实际形状（list vs JSON string）：实现时读 verify runner 实际输出确认，resolver 防御性解析（已确认 runner `acl_policy.py:325` 从 vals 项取 id；full_config 持原始记录）
2. `ptm-atomic show` 测试环境可用性：测试 mock subprocess；运行时复用既有 ptm-atomic CLI 依赖
3. mode D 自反 inverse_op 检测：用 `id_source=placeholder` 触发 arg-swap 分支（不靠 op_id 比对）
4. step_id 传递：agent 须在调 execute_op 时传 PC 的 step_id；文档明确

## 验证计划
- `pytest tests/test_op_mapper_rollback.py tests/test_step_refs.py` 全绿
- 4 模式手工 dry-run（policy_route/object/acl_policy/acl_policy_group 各一轮）
- `validate_mapping_consistency` 通过
- 现有 policy_route rollback 回归测试不变
