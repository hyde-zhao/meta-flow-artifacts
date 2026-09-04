---
doc_type: STORY
cr_id: CR-035
story_id: STORY-035-S05
evidence_type: technical-note
lld_policy_required_level: technical-note
risk_level: Medium
file_ownership: tests/（test_tg_op_mapping.py 扩展 + 新增 test_config_once_skip.py + test_tg_dry_run.py）
dependencies: [STORY-035-S01, STORY-035-S02, STORY-035-S03]
hld_ref: process/changes/CR-035-HLD.md
author: meta-dev
created_at: 2026-07-31
---

# STORY-035-S05 技术说明：测试（FU-02 关闭 + op_mapping + config-once + dry-run）

> HLD ref：§14（验证策略 static-only + dry-run-only）+ Story S05。
> 本技术说明汇总 S01/S02/S03 测试设计并补充 FU-02 关闭用例。validation_mode=static-only + dry-run-only；真实 --execute 行为 N/A，CP7 写明。

## 文件影响与设计依据

| 文件 | 改动 |
|---|---|
| `tests/test_tg_op_mapping.py`（扩展） | +13 用例（S01 §8） |
| `tests/test_config_once_skip.py`（新增） | +13 用例（S03 §8，含 skip_count 聚合） |
| `tests/test_tg_dry_run.py`（新增） | +9 用例（S02 §8）+ FU-02 用例 |

## 接口、数据与测试分组用例

### 1. FU-02 聚合占位符测试关闭（HLD §1）

| 用例名 | 断言 |
|---|---|
| `test_fu02_aggregate_placeholder_tg_ports` | `${ENV.tg.ports[port1,port2]}` 聚合占位符 resolve_env_refs 正确展开为逗号串（如 `2/1,2/2`）；FU-02 标记关闭 |

FU-02 原为聚合占位符测试缺口，CR-035 acquire 路径依赖 `tg_ports` 逗号串（S02 §5.2 `",".join(tg_ports)`），需验证占位符展开与 `_resolve_tg_ports` 输出一致。

### 2. op_mapping（S01 实现，HLD §14 static-only）

| 用例名 | 断言 |
|---|---|
| `test_expected_op_count_is_26` | `len(OP_ID_TO_SUBCOMMAND) == 26 and EXPECTED_OP_COUNT == 26` |
| `test_tg_acquire_ports_subcommand` | `== ("tg", "acquire-ports")` |
| `test_tg_release_ports_subcommand` | `== ("tg", "release-ports")` |
| `test_tg_verify_ownership_subcommand` | `== ("tg", "verify-ownership")` |
| `test_tg_acquire_ports_build_command` | 输出含 `tg trex acquire-ports --user-id dev1-a3f2 --ports 2/1,2/2` |
| `test_tg_release_ports_build_command` | 输出含 `tg trex release-ports --user-id dev1-a3f2`，不含 `--ports` |
| `test_tg_verify_ownership_build_command` | 输出含 `tg trex verify-ownership`，无 `--user-id` |
| `test_tg_acquire_ports_required_flags` | 缺 user_id/ports 各抛 ValueError |
| `test_tg_release_ports_required_flags` | 缺 user_id 抛 ValueError；不传 ports 不抛 |
| `test_existing_tg_ops_have_user_id_flag` | 6 个现有 tg op ARGS_TO_FLAGS 含 `"user_id": "--user-id"` |
| `test_tg_acquire_ports_metadata` | side_effect=state_mutation, rollback=tg_release_ports, idempotent=False |
| `test_tg_release_ports_metadata` | idempotent=True |
| `test_tg_verify_ownership_metadata` | side_effect=observation |

### 3. config-once skip（S03 实现，mock tg_run_ctx，HLD §14 static-only）

| 用例名 | 断言 |
|---|---|
| `test_config_once_first_executes_sets_flag` | tg_run_ctx{configured:False}，--execute，execute_op mock success -> 真实执行，configured 置 True |
| `test_config_once_subsequent_skipped` | tg_run_ctx{configured:True}，--execute，force False -> status==skipped，execute_op 不调用 |
| `test_config_once_force_config_resets_and_executes` | tg_run_ctx{configured:True}，force True -> 真实执行，执行前 configured=False，成功后 True |
| `test_config_once_fail_no_flag_set` | execute_op mock fail，--execute -> configured 保持原值（AGA-4） |
| `test_config_once_dry_run_not_skip` | dry-run，configured True -> 不 skip，build_command 展示命令（DQ-035-06） |
| `test_config_once_dry_run_no_flag_set` | dry-run -> configured 保持 False（不置标志） |
| `test_tg_user_id_auto_inject` | tg_run_ctx{tg_user_id:"dev1-a3f2"}，tg_start_traffic_stream -> args 含 user_id |
| `test_tg_user_id_not_injected_when_no_ctx` | tg_run_ctx=None -> args 不含 user_id（向后兼容） |
| `test_tg_user_id_not_overwrite_existing` | args 已含 user_id -> 不覆盖 |
| `test_tg_acquire_ports_not_in_tg_ops_need_user_id` | tg_acquire_ports 不在集合 |
| `test_tg_verify_ownership_not_in_tg_ops_need_user_id` | tg_verify_ownership 不在集合 |
| `test_skip_record_envelope_status_skipped` | skip 记录 envelope.status=="skipped"（供 S02 聚合） |
| `test_config_once_skip_count_aggregation` | 2 用例各 1 个 tg_config_interface skip -> main 汇总 count==2（P3：从 dry-run 组移入，测试 S02 main 聚合逻辑，语义属 config-once） |

### 4. dry-run 端到端（S02 实现，HLD §14 dry-run-only）

| 用例名 | 断言 |
|---|---|
| `test_build_tg_user_id_format` | 返回匹配 `dev\d+-[0-9a-f]{4}`；同 run_id 幂等；不同 run_id 不同 |
| `test_resolve_tg_ports_from_mapping` | port_mapping 含 2 端口 -> 返回 2 元素 list |
| `test_resolve_tg_ports_missing` | env_topology 无 port_mapping -> 返回 [] |
| `test_dry_run_acquire_ok_stays_true` | dry-run result.json `tg_port_ownership.acquire_ok == True`，`acquire_record is None`（P0 守卫） |
| `test_dry_run_release_not_called` | dry-run `release_record is None` |
| `test_dry_run_tg_user_id_format` | dry-run result.json `tg_user_id` 匹配 `dev\d+-[0-9a-f]{4}` |
| `test_acquire_fail_skips_cases` | acquire mock 失败 -> [5] 跳过，results 为空，release 不调用（mock 验证） |
| `test_port_mapping_missing_acquire_ok_false` | 无 port_mapping -> acquire_ok False，[5] 跳过 |
| `test_result_json_contains_tg_port_ownership` | result.json 含 tg_port_ownership 6 字段 |

## 实施

1. 扩展 `tests/test_tg_op_mapping.py`（+13 用例，组 2）。
2. 新增 `tests/test_config_once_skip.py`（+13 用例，组 3，mock execute_op / build_command，含 skip_count 聚合）。
3. 新增 `tests/test_tg_dry_run.py`（+9 用例 + FU-02，组 4 + 组 1，dry-run 端到端或 main mock）。
4. 运行全部测试 PASS；标记 FU-02 关闭。
5. 确认覆盖 HLD §6.1 判断表全部分支 + §6.2 dry-run vs --execute 表。

## 验收准则（含权限与失败处理）

- [ ] 全部测试 PASS（组 1-4 共 36 用例）。
- [ ] FU-02 标记关闭（`test_fu02_aggregate_placeholder_tg_ports` PASS）。
- [ ] `test_expected_op_count_is_26` PASS。
- [ ] 覆盖 HLD §6.1 判断表 5 分支（首个/后续 skip/force/失败/其他 tg op）。
- [ ] 覆盖 HLD §6.2 dry-run vs --execute 2 模式（不 skip / skip）。
- [ ] P0 守卫：`test_dry_run_acquire_ok_stays_true` PASS。
- [ ] DQ-035-06 守卫：`test_config_once_dry_run_not_skip` PASS。
- [ ] 现有测试无回归。

## 依赖

- S01 + S02 + S03（被测对象）。
- HLD §6 + §14 为验证真相源。

## 风险

| 风险 | 等级 | 缓解 |
|---|---|---|
| mock execute_op 与真实行为偏差 | Medium | mock 严格按 envelope schema 返回 status/data；真实 --execute 行为 CP7 N/A + 风险登记 |
| dry-run 端到端依赖 env 文件 | Low | 使用 fixture env_topology（含 port_mapping + runner_ip）或 mock |
| FU-02 占位符展开逻辑与 _resolve_tg_ports 不一致 | Low | 两者均从 port_mapping 取 tg 字段；测试断言输出一致 |


## 语义维度覆盖

> lld-check required token 覆盖声明（指向对应章节）：

- **设计依据**：本 LLD 已覆盖（见对应章节）
- **文件影响**：本 LLD 已覆盖（见对应章节）
- **接口**：本 LLD 已覆盖（见对应章节）
- **数据**：本 LLD 已覆盖（见对应章节）
- **权限**：本 LLD 已覆盖（见对应章节）
- **失败**：本 LLD 已覆盖（见对应章节）
- **测试**：本 LLD 已覆盖（见对应章节）
- **风险**：本 LLD 已覆盖（见对应章节）
