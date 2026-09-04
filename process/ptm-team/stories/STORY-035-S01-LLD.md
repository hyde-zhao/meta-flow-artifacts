---
doc_type: LLD
cr_id: CR-035
story_id: STORY-035-S01
evidence_type: full-lld
lld_policy_required_level: full-lld
tier: S
risk_level: Medium
file_ownership: skills/policy-route-execution/scripts/op_mapper.py（独占）
dependencies: []
blocks: [STORY-035-S03, STORY-035-S05]
hld_ref: process/changes/CR-035-HLD.md
author: meta-dev
created_at: 2026-07-31
---

# LLD：STORY-035-S01 op_mapper 注册 acquire/release/verify-ownership + tg 族 user_id 映射

> 本 LLD 不重复 HLD 已有内容，引用 HLD 章节并补充 Story 级实现细节（精确行号、函数签名、测试用例名）。
> HLD ref：`process/changes/CR-035-HLD.md` §9（op_mapper 新增映射）/ §8.1（集成契约）/ §13（Gotchas）。

## 0. 概述

本 LLD 对应 STORY-035-S01（op_mapper 注册 acquire/release/verify-ownership + tg 族 user_id 映射），基于 HLD v0.2（`process/changes/CR-035-HLD.md`）细化 Story 级实现设计。CP3 approved（DQ-035-06 方案 A dry-run 不 skip + AGA-1/3/4 agent 默认 + P0 acquire_ok 初始 True 整改）。

## 1. 工程依据与背景

op_mapper.py 是 op_id -> CLI 子命令映射的模块级真相源（单点维护）。当前 `EXPECTED_OP_COUNT = 23`（L562），tg 族 6 个 op（L88-94）。CR-035 需新增 3 个 tg 族 op（acquire/release/verify-ownership）并为 tg 族补 `user_id` flag 映射以支持 owned 占用模式。完整背景见 HLD §1。

## 2. 目标与需求（量化）

1. `len(OP_ID_TO_SUBCOMMAND) == 26`（23 基线 + 3 新 op）。
2. 3 个新 op 的 `build_command` 生成正确三层命令 `tg trex <action>`。
3. 6 个现有 tg op 的 `ARGS_TO_FLAGS` 含 `"user_id": "--user-id"` 映射。
4. `REQUIRED_FLAGS` 对 `tg_acquire_ports` 校验 `["--user-id", "--ports"]`，`tg_release_ports` 校验 `["--user-id"]`，`tg_verify_ownership` 校验 `[]`。
5. `OP_METADATA` 含 3 个新 op 的 side_effect/rollback/idempotent。

## 3. 模块拆分与文件影响范围

| 文件 | 区段 | 改动 |
|---|---|---|
| `skills/policy-route-execution/scripts/op_mapper.py` | L64-95 `OP_ID_TO_SUBCOMMAND` | 追加 3 个 op |
| 同上 | L228-271 `ARGS_TO_FLAGS` tg 族 | 追加 3 个新 op dict + 6 个现有 tg op 加 `user_id` flag |
| 同上 | L276- `REQUIRED_FLAGS` | 追加 3 个新 op |
| 同上 | L529-558 `OP_METADATA` tg 族 | 追加 3 个新 op |
| 同上 | L562 `EXPECTED_OP_COUNT` | 23 -> 26 |
| 同上 | L58-63 注释 | 更新 op 总数注释 + acquire/release 说明 |

不改 `build_command`（L784-786 对 `family == "tg"` 自动插入 `"trex"` 生成三层命令，HLD §8.1 已确认无需改动）。

## 4. API 与接口设计

本 Story 无新增函数，仅扩展模块级常量。`build_command(op_id, args, ...)` 签名不变，通过查表自动覆盖新 op。

新 op 的 CLI 命令形态（HLD §8.1）：
- `ptm-atomic run ... tg trex acquire-ports --user-id <id> --ports <p1,p2>`
- `ptm-atomic run ... tg trex release-ports --user-id <id>`（不传 --ports）
- `ptm-atomic run ... tg trex verify-ownership`（无 required flag，查询全量占用）

## 5. 流程（核心）

`build_command` 既有流程（不改动）：查 `OP_ID_TO_SUBCOMMAND` 得 `(family, action)` -> family=="tg" 插入 "trex" -> 查 `ARGS_TO_FLAGS` 翻译 args -> 查 `REQUIRED_FLAGS` 校验必填。新 op 自动走该流程，无需分支。

## 6. 代码结构与数据模型

### 6.1 OP_ID_TO_SUBCOMMAND 追加（L94 后）
```python
"tg_acquire_ports": ("tg", "acquire-ports"),       # required: --user-id, --ports
"tg_release_ports": ("tg", "release-ports"),       # required: --user-id
"tg_verify_ownership": ("tg", "verify-ownership"), # 无 required flag
```
注意：action 名用连字符（`acquire-ports`），op_id 用下划线（HLD Gotcha#7）。

### 6.2 ARGS_TO_FLAGS 追加（L271 后）
```python
"tg_acquire_ports": {"user_id": "--user-id", "ports": "--ports"},
"tg_release_ports": {"user_id": "--user-id", "ports": "--ports"},
"tg_verify_ownership": {},
```
6 个现有 tg op（`tg_config_interface` / `tg_apply_traffic_template` / `tg_delete_traffic_template` / `tg_start_traffic_stream` / `tg_stop_traffic_stream` / `tg_verify_traffic_loss`）的 dict 内追加 `"user_id": "--user-id"`。`ports` 在 `tg_acquire_ports` 是 list -> 逗号串序列化（与现有 `tg_start_traffic_stream` 的 `ports` 一致，build_command 已处理）。

### 6.3 REQUIRED_FLAGS 追加
```python
"tg_acquire_ports": ["--user-id", "--ports"],
"tg_release_ports": ["--user-id"],
"tg_verify_ownership": [],
```

### 6.4 OP_METADATA 追加（L558 后）
```python
"tg_acquire_ports": {"side_effect": "state_mutation", "rollback": "tg_release_ports", "idempotent": False},
"tg_release_ports": {"side_effect": "state_mutation", "rollback": "", "idempotent": True},
"tg_verify_ownership": {"side_effect": "observation", "rollback": "", "idempotent": True},
```

### 6.5 EXPECTED_OP_COUNT（L562）
```python
EXPECTED_OP_COUNT = 26  # 23 基线 + 3 CR-035（acquire/release/verify-ownership）
```
同步更新 L58 注释 `# 共 23 个 op_id` -> `# 共 26 个 op_id`。

## 7. 技术细节、安全与异常处理

- `build_command` 对新 op 缺 required flag 时抛 `ValueError`（既有机制，REQUIRED_FLAGS 校验，无需新增分支）。
- `tg_acquire_ports` 缺 `--user-id` 或 `--ports` -> ValueError，由 S02 main 的 acquire 块捕获 envelope status 非 success -> `acquire_ok=False`。
- `tg_verify_ownership` 无 required flag，不会因缺 flag 抛错；查询全量占用返回 envelope。

## 8. 测试

测试在 STORY-035-S05 实现，本 Story 定义用例名与断言（HLD §14）：

| 用例名 | 断言 |
|---|---|
| `test_expected_op_count_is_26` | `assert len(OP_ID_TO_SUBCOMMAND) == 26 and EXPECTED_OP_COUNT == 26` |
| `test_tg_acquire_ports_subcommand` | `assert map_op_id_to_subcommand("tg_acquire_ports") == ("tg", "acquire-ports")` |
| `test_tg_release_ports_subcommand` | `== ("tg", "release-ports")` |
| `test_tg_verify_ownership_subcommand` | `== ("tg", "verify-ownership")` |
| `test_tg_acquire_ports_build_command` | build_command 输出含 `tg trex acquire-ports --user-id dev1-a3f2 --ports 2/1,2/2` |
| `test_tg_release_ports_build_command` | 输出含 `tg trex release-ports --user-id dev1-a3f2`，不含 `--ports` |
| `test_tg_verify_ownership_build_command` | 输出含 `tg trex verify-ownership`，无 `--user-id` |
| `test_tg_acquire_ports_required_flags` | 缺 user_id/ports 各抛 ValueError |
| `test_tg_release_ports_required_flags` | 缺 user_id 抛 ValueError；不传 ports 不抛 |
| `test_existing_tg_ops_have_user_id_flag` | 6 个现有 tg op 的 `ARGS_TO_FLAGS[op]` 含 `"user_id": "--user-id"` |
| `test_tg_acquire_ports_metadata` | side_effect=state_mutation, rollback=tg_release_ports, idempotent=False |
| `test_tg_release_ports_metadata` | idempotent=True |
| `test_tg_verify_ownership_metadata` | side_effect=observation |

## 9. 实施

1. 在 `OP_ID_TO_SUBCOMMAND`（L94 `tg_verify_traffic_loss` 行后）追加 3 个 op（§6.1）。
2. 在 `ARGS_TO_FLAGS`（L271 `tg_delete_traffic_template` dict 后）追加 3 个新 op dict；在 6 个现有 tg op dict 内各追加 `"user_id": "--user-id"`（§6.2）。
3. 在 `REQUIRED_FLAGS` tg 族区段追加 3 个新 op（§6.3）。
4. 在 `OP_METADATA`（L558 `tg_delete_traffic_template` 后）追加 3 个新 op（§6.4）。
5. 修改 `EXPECTED_OP_COUNT = 26`（L562）+ 更新 L58 注释（§6.5）。
6. 运行 `test_tg_op_mapping.py` 全部 PASS。

## 10. 回滚策略

纯常量追加，回滚 = 删除新增行 + 还原 `EXPECTED_OP_COUNT = 23` + 删除 6 个现有 tg op 的 `user_id` flag。无数据迁移、无外部状态。

## 11. DoD（验收准则）

- [ ] `len(OP_ID_TO_SUBCOMMAND) == 26` 且 `EXPECTED_OP_COUNT == 26`。
- [ ] 3 个新 op 的 build_command 断言通过（§8 表）。
- [ ] 6 个现有 tg op 含 `user_id` flag 映射。
- [ ] REQUIRED_FLAGS 校验行为符合 §6.3。
- [ ] OP_METADATA 3 个新 op 字段正确（§6.4）。
- [ ] 现有 op_mapper 测试无回归（非 tg 族断言不变）。

## 12. 依赖

- 无上游 Story 依赖。HLD §9 为唯一设计真相源。
- 下游：S03（execute_steps 的 `TG_OPS_NEED_USER_ID` 依赖 user_id flag 注册）、S05（测试）。

## 13. 风险

| 风险 | 等级 | 缓解 |
|---|---|---|
| 6 个现有 tg op 加 user_id flag 后，未注入 user_id 时 build_command 不传该 flag（args 无 key 不生成 flag），向后兼容 | Low | build_command 仅对 args 中存在的 key 生成 flag；不注入走 legacy force 模式（HLD Gotcha#5） |
| `tg_release_ports` 的 ARGS_TO_FLAGS 含 `ports` 但 REQUIRED_FLAGS 不含，build_command 传 ports 时生成 flag 但 main 不传 | Low | main 仅传 `{user_id}`，ports key 不出现；ARGS_TO_FLAGS 保留 ports 仅为对称声明 |
| EXPECTED_OP_COUNT 与实际 len 不一致 | Low | S05 测试 `test_expected_op_count_is_26` 同时断言两者相等 |

## 14. open_items

- 无 Story 级 open item。HLD O-035-02（runner_ip 来源）属 S02 范围；O-035-03（Wave 编排）属 CP4 预检范围。


## 语义维度覆盖

> lld-check required token 覆盖声明（指向对应章节）：

- **工程依据**：本 LLD 已覆盖（见对应章节）
- **目标**：本 LLD 已覆盖（见对应章节）
- **需求**：本 LLD 已覆盖（见对应章节）
- **模块拆分**：本 LLD 已覆盖（见对应章节）
- **代码结构**：本 LLD 已覆盖（见对应章节）
- **数据模型**：本 LLD 已覆盖（见对应章节）
- **API**：本 LLD 已覆盖（见对应章节）
- **流程**：本 LLD 已覆盖（见对应章节）
- **技术细节**：本 LLD 已覆盖（见对应章节）
- **安全**：本 LLD 已覆盖（见对应章节）
- **测试**：本 LLD 已覆盖（见对应章节）
- **实施**：本 LLD 已覆盖（见对应章节）
- **风险**：本 LLD 已覆盖（见对应章节）
- **DoD**：本 LLD 已覆盖（见对应章节）
