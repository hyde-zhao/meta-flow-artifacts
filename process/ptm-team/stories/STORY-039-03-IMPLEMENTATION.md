---
story_id: STORY-039-03
story_slug: g5-sw3-family-backport
cr_id: CR-039
wave: 3
taker: meta-dev
implemented_at: 2026-08-19
status: dev-complete
evidence_type: implementation-execution
---

# STORY-039-03 实现执行证据：G5 sw3 族回源

## 实现前置检查

| 检查项 | 结果 |
|---|---|
| Story 状态与设计证据 | CP5 已 approve（process/checkpoints/CP5-CR039.md，STORY-039-01~04 全量批次，2026-08-19；DQ-039-02=A / DQ-039-04 / DQ-039-05=B 均 approve）；technical-note 已确认为实现基线 |
| 依赖门控 | STORY-039-02（file-ownership 串行，op_mapper.py 第二写入块）已 dev-complete；本 Story 为第三个且最后写入块（merge_owner），负责 `validate_mapping_consistency()` 最终一致性 |
| 文件所有权 | primary: skills/case-execution/scripts/case_runner.py + tests/test_tg_op_mapping.py；shared: skills/policy-route-execution/scripts/op_mapper.py（G5 区块写入）；无 dev_running 冲突 |
| 基底 | canonical 当前工作区（含 W1 G1 区块 `_query_static_route_id` L2340 + rollback 分支、W2 G4 区块 next_hop_a 分支 + 10 类文案），逐块合并，非整文件覆盖，G1/G4 零回退 |
| 验证环境 | Python 3.11 + pytest（uv run）；static-only + dry-run，无网络/真机 SW3（CR-039 security_constraints；真机 telnet 为独立 runtime_authorization，DQ-039-04 不授权） |
| 回源真相源 | /home/hyde/projects/ptm-te-manaul/.claude/skills/policy-route-execution/scripts/op_mapper.py 与 .../case-execution/scripts/case_runner.py（工作区已实测通过：PPPoE 环回 2 用例 26/28 step PASS，CR-046） |

## 实现对象清单

1. `skills/policy-route-execution/scripts/op_mapper.py`（G5 区块，9 处改动）：
   - **B1 OP_ID_TO_SUBCOMMAND**：新增 `sw3_sync_pppoe_route: ("sw3", "sync-pppoe-route")`（表尾，tg_verify_ownership 后）。
   - **B2 ARGS_TO_FLAGS**：新增 `vpn_instance -> --vpn-instance`（形式 flag，实际不走 ptm-atomic，仅为过四表一致性校验）。
   - **B3 REQUIRED_FLAGS**：新增 `sw3_sync_pppoe_route: []`（vpn_instance 有默认值 hyde，非必填）。
   - **B4 ROLLBACK_STRATEGY**：新增 `type=irreversible`，reason「修复性同步 SW3 默认路由下一跳到当前 client IP，回滚即回退到过期下一跳，无意义」。
   - **B5 OP_METADATA**：新增 `side_effect=state_mutation / rollback=irreversible / idempotent=True`。
   - **B6 EXPECTED_OP_COUNT**：44 -> **45**，注释「42（CR-043 基准）+ 2（CR-044 service-pool config/delete）+ 1（CR-046 sw3_sync_pppoe_route）= 45」。
   - **B7 execute_op sw3 旁路**：resolve_env_refs 后新增 `family == "sw3"` 分派 `_execute_sw3_op`（不经 resolve_step_refs / build_command / subprocess）。
   - **B8 sw3 执行段三函数**（execute_op 之后、op 声明读取之前整段插入，239 行）：`_execute_sw3_op`（定位 SW 节点 -> dry-run 计划 -> 授权校验 -> 密码环境变量解析 -> telnet 同步默认路由下一跳）、`_exec_sw3_telnet`（telnetlib 惰性导入 py<3.13，H3C `ogin:`/`assword:` 大小写不敏感匹配，`screen-length 0` 关分页，不打印密码不落日志）、`_parse_sw3_pppoe_route`（/32 Direct VA host 路由 -> client IP；0.0.0.0/0 行 -> default_next_hop）。
   - **B9 validate [5j]**：`expected_sw3_actions = {"sync-pppoe-route"}` 子命令校验（[5i] tg 族之后、[7] 之前）。
2. `skills/case-execution/scripts/case_runner.py`（sw3 dry-run envelope，3 处改动，**CR-047 post_delay 精细排除**）：
   - **C1 import**：`from op_mapper import` 块新增 `map_op_id_to_subcommand`（handle_rollback 之后，字母序）。
   - **C2 `_build_sw3_dry_run_envelope(step, sw3_env)`**（`_build_dry_run_envelope` 之后新增）：error 透传（status=error + error_type 原样）；success -> `status=dry_run, data.command=计划 dict`（非 ptm-atomic 命令 list）。
   - **C3 dry-run sw3 分支**（`_dry_run_fn` 内，build_command 之前）：`family == "sw3"` -> `execute_op(dry_run=True, authorized=False)` 获取命令计划 -> envelope 包装。
3. `tests/test_tg_op_mapping.py`（primary 所有权）：
   - L159 类 docstring 44 -> 45（14 族，含 tg 9 op + sw3 1 op）；`test_pass_44_ops` -> `test_pass_45_ops`（断言 45）。
   - `test_expected_op_count_is_44` -> `test_expected_op_count_is_45`（len + 常量双断言 45）。
   - 新增 `Cr046Sw3FamilyOpMappingTests`（8 用例：六处映射 / 族 action / dry-run 计划 / 无 SW 节点 PARAM_INVALID / 未授权 EXEC_FAILED / SW3_PASSWORD 未设置 PARAM_INVALID / 路由表解析）与 `Cr046Sw3DryRunEnvelopeTests`（3 用例：envelope 成功包装 / 错误透传 / post_delay 零残留源码断言）。
4. `tests/test_op_mapper_pppoe_client.py`（**最小计数载体同步，见偏离记录**）：`BackwardCompatTests::test_expected_op_count` 断言 44 -> 45（docstring 同步 + 变更注释），否则该文件因 EXPECTED_OP_COUNT 变更必然回归。

## 设计契约映射

| 技术说明契约 | 实现位置 | 验证入口 |
|---|---|---|
| 六处映射表 sw3 条目（OP_ID_TO_SUBCOMMAND / ARGS_TO_FLAGS / REQUIRED_FLAGS / ROLLBACK_STRATEGY / OP_METADATA / EXPECTED_OP_COUNT） | B1-B6 | test_sw3_six_mappings / test_expected_op_count_is_45 / test_pass_45_ops |
| `EXPECTED_OP_COUNT=45` 四处载体一致（SM-039-01） | B6 代码常量 + validate 三表 + 单测断言（tg_op_mapping + pppoe_client）+ SKILL.md Gotcha#8（W1 已落 45） | validate_mapping_consistency PASS + 单测 |
| execute_op `family=="sw3"` 旁路，不经 build_command/subprocess | B7 | test_execute_op_sw3_dry_run_returns_plan |
| 定位 SW 节点：优先 `role=="pppoe_server"`，fallback 任一 `node_type=="SW"`；未找到 -> PARAM_INVALID | B8 `_execute_sw3_op` [1] | test_execute_op_sw3_without_sw_node_param_invalid |
| dry_run=True 返回命令计划（不连设备，ADR-02） | B8 [2] | test_execute_op_sw3_dry_run_returns_plan |
| dry_run=False 且 authorized=False -> EXEC_FAILED（DQ-039-04 不授权真机） | B8 [3] | test_execute_op_sw3_execute_unauthorized_exec_failed |
| 密码仅从 `SW3_PASSWORD`（`password_env`）环境变量读取，不打印不落盘（ADR-02）；未设置 -> PARAM_INVALID | B8 [4] + `_exec_sw3_telnet` docstring | test_execute_op_sw3_password_env_missing_param_invalid |
| telnetlib 惰性导入（py < 3.13）；H3C 大小写不敏感登录匹配；screen-length 0 关分页 | B8 `_exec_sw3_telnet` | py_compile + 工作区逐行对拍（实测已验证） |
| `_parse_sw3_pppoe_route` 解析 (client_ip, default_next_hop) | B8 | test_parse_sw3_pppoe_route |
| validate `[5j]` sw3 子命令校验 | B9 | test_pass_45_ops（validate PASS）+ test_sw3_family_single_action |
| case_runner import `map_op_id_to_subcommand` | C1 | case_runner py_compile + envelope 单测（模块加载即验证 import） |
| `_build_sw3_dry_run_envelope`：error 透传 / success -> status=dry_run, data.command=计划 | C2 | test_envelope_success_wraps_plan / test_envelope_error_passthrough |
| dry-run sw3 分支 -> execute_op(dry_run=True) -> envelope 包装 | C3 | test_envelope_success_wraps_plan（消费同一 execute_op 输出形态） |
| **CR-047 post_delay 精细排除（DQ-039-05=B）**：不引入 `_parse_post_delay` 与 post_delay 睡眠改动 | 仅回源 C1-C3 三块，工作区 L114-133 / L2509-2511 零带入 | test_case_runner_no_post_delay_backport + grep 零命中（见验证结果） |

## 单元测试与 Fixture 计划

- Fixture：`SW3_TOPOLOGY` 构造 env_topology.nodes.sw1（node_type=SW / role=pppoe_server / host=10.113.55.179 / username=admin / password_env=SW3_PASSWORD），对齐工作区 CR-046 验证用例 `node3_dut1_tg1_sw1_pppoe_link3` 的 env-file SW 节点契约（样例 yaml 由 STORY-039-04/W1 落地：skills/case-execution/templates/env-file.pppoe-sw3.next_hop_a.example.yaml）。
- 加载方式：op_mapper / case_runner 均 importlib 按路径直载（与既有 tg 测试同风格）；case_runner 自带 sys.path 注入 import op_mapper（Gotcha #2）。
- 新增 11 用例（8 + 3，见实现对象清单第 3 条），mock：仅 `mock.patch.dict(os.environ)` 移除 SW3_PASSWORD（环境变量行为）；无 subprocess mock（sw3 dry-run 不触发 subprocess）。
- 既有断言更新：44 -> 45 共 3 处（test_tg_op_mapping.py 2 处 + test_op_mapper_pppoe_client.py 1 处）。

## 最小实现切片

- **Slice-1（op_mapper G5 区）**：B1-B9 落码 -> `uv run pytest tests/test_tg_op_mapping.py`（含既有 44->45 断言更新）+ validate standalone。
- **Slice-2（case_runner sw3 envelope）**：C1-C3 落码 -> py_compile + envelope 单测 + post_delay grep 零命中。
- **Slice-3（回归收口）**：指定三件套（tg_op_mapping + rollback + next_hop_a）+ env_unsupported + pppoe_client + 全量 tests/。

## 验证结果

| 命令 | 结果 |
|---|---|
| `uv run --python 3.11 pytest tests/test_tg_op_mapping.py tests/test_op_mapper_rollback.py tests/test_op_mapper_next_hop_a.py -x` | **85 passed**（tg_op_mapping 47 含新增 11；rollback + next_hop_a 38 不回归） |
| `uv run --python 3.11 pytest tests/test_env_unsupported.py tests/test_op_mapper_pppoe_client.py` | 26 passed（含 pppoe_client 计数载体 45 断言） |
| `uv run --python 3.11 pytest tests/ -q` | 244 passed + 2 failed（**存量失败**：test_cr018_p2 GATE-4 脚本问题；经 `git stash` 对拍确认改动前同样失败，与本 Story 无关，不属本写入范围） |
| `validate_mapping_consistency()`（standalone） | **passed=True, mismatches=[]**；EXPECTED_OP_COUNT=45，len(OP_ID_TO_SUBCOMMAND)=45，sw3 map=('sw3','sync-pppoe-route') |
| `uv run --python 3.11 python -m py_compile` op_mapper.py + case_runner.py | 通过 |
| `grep -n "post_delay\|_parse_post_delay" skills/case-execution/scripts/case_runner.py` | **零命中**（exit=1；CR-047 精细排除验证，另有单测 test_case_runner_no_post_delay_backport 源码级守护） |
| 工作区逐块对拍 | sw3 执行段 239 行 `diff` **逐行一致**（SW3-BLOCK-IDENTICAL）；六处映射条目 / execute_op 旁路 / [5j] / case_runner import / `_build_sw3_dry_run_envelope` / dry-run sw3 分支 grep-A4 `diff` 全部 IDENTICAL |

### EXPECTED_OP_COUNT=45 四处最终一致性确认（SM-039-01，DQ-039-02=A）

| 载体 | 位置 | 值 |
|---|---|---|
| 1. 代码常量 | op_mapper.py `EXPECTED_OP_COUNT = 45` | 45 |
| 2. validate 三表 | `validate_mapping_consistency()` PASS（四表 op_id 集合一致 + count 校验 + [5j] sw3 action 校验） | 45 |
| 3. 单测断言 | test_tg_op_mapping.py（test_pass_45_ops / test_expected_op_count_is_45 / test_sw3_six_mappings）+ test_op_mapper_pppoe_client.py（test_expected_op_count） | 45 |
| 4. SKILL.md Gotcha#8 | skills/case-execution/SKILL.md Gotcha#8（STORY-039-04/W1 已落）= CR-041 定格 40 + CR-043 +2 + CR-044 +2 + CR-046 +1 | 45 |

## 偏离记录

1. **tests/test_op_mapper_pppoe_client.py 计数载体同步（不在 Story 文件清单内）**：该文件 `BackwardCompatTests::test_expected_op_count` 为 EXPECTED_OP_COUNT 断言载体，44->45 变更后必然回归；按 SM-039-01「单测断言一致」与 merge_owner 最终一致性职责做最小同步（docstring + 1 行断言 + 变更注释），未触碰该文件其他 STORY-038-05 断言。属计数口径必然后果，非范围扩大。
2. **dry-run 成功 envelope 的 error_type 断言**：工作区 `_execute_sw3_op` dry-run 成功走 `_build_envelope(..., "", ...)`，error_type 为空串而非 "NONE"；单测按工作区实际行为断言 `""`（真相源优先，未改动实现）。
3. 工作区行号漂移：Story 卡片引用的 case_runner 行号（L925-950 / L2317-2338）在工作区实际为 L925-950 / L2362-2375（工作区含 CR-047 post_delay 改动致偏移）；回源以内容块定位，非行号定位。

## 未覆盖项

- 真机 SW3 telnet 同步端到端（H3C 登录、路由表实查、undo/ip route-static 下发）：CR-039 static-only，不授权真机（DQ-039-04）；运行时风险 RA-039-02 由台账承接，CP7 以 static-only + dry-run 口径验证。
- telnetlib py>=3.13 不可用分支：仅惰性导入注释声明（py<3.13），无单测覆盖（本仓 uv 锁 3.11，无 3.13 环境）。
- sw3 dry-run 经 case_runner `execute_steps` 全链路（retry 包装 / step-refs mock 落盘）：单测覆盖 envelope 构建函数与 execute_op(dry_run=True) 两侧；全链路 dry-run 属 CP7 dry-run 场景。

## 设计缺口反馈

无。technical-note 全部契约逐条落实；G6 M9 归属：sw3 族真机验证属 G6 M9（运行授权类），本 CR 不授权，已由 DQ-039-04 决策承接，无新增设计分叉。

## 平台差异处理

N/A。改动对象为 canonical 源 Skill 私有脚本（op_mapper.py / case_runner.py）与仓库级单测，不涉及平台安装目录、发现路径或 PLATFORM-CONTRACTS 平台分支；sw3 族走 telnet 旁路（不经 ptm-atomic），重装 ptm-atomic 后随 Skill 脚本自然生效（telnetlib 为 Python 3.11 标准库）。sw3 用例样例来源：工作区 CR-046 验证用例 `node3_dut1(DUT2_10.113.55.163)_tg1(trex_10.113.55.175)_sw1(SW3_10.113.55.179)_link3(PPPoE).yml`。

## 后续交接（-> meta-qa / CP6）

- Return Packet：`process/returns/STORY-039-03.return.json`；Evidence 摘要见本文件。
- 验证入口：`uv run --python 3.11 pytest tests/test_tg_op_mapping.py`（主，47 用例含 sw3 11 新增）；`tests/test_op_mapper_rollback.py` / `tests/test_op_mapper_next_hop_a.py` / `tests/test_env_unsupported.py` / `tests/test_op_mapper_pppoe_client.py`（回归）；`grep -n "post_delay\|_parse_post_delay" skills/case-execution/scripts/case_runner.py`（应零命中）。
- 风险提示：test_cr018_p2 两个失败为存量问题（GATE-4 脚本），与本 Story 无关（stash 对拍已证）；CP7 勿计入本 Story 缺陷。
- 风险提示：STORY-039-02 已提示本 Story 的 44->45 为预期变更；至此 CR-039 三波写入全部完成，op_mapper.py 最终态 = G1 + G4 + G5 叠加。
