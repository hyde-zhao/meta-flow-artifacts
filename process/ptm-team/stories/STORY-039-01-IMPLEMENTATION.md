---
story_id: STORY-039-01
story_slug: g1-static-route-rollback-verify
cr_id: CR-039
stage: implementation
status: dev-complete
implemented_at: 2026-08-19
implementer: meta-dev
design_evidence_ref: process/stories/STORY-039-01.md#技术说明
design_evidence_type: technical-note
design_evidence_confirmed: true
---

# STORY-039-01 实现执行证据：G1 静态路由回滚 verify 兜底

## 实现前置检查

| 检查项 | 结果 |
|---|---|
| Story status（实现前） | planned -> in-development -> dev-complete |
| 设计证据已确认 | 是（CP5 approve，technical-note = Story 卡片 `## 技术说明`，DQ-039-01=方案 A） |
| `dev_gate` | cp5-approved，满足 |
| `depends_on` | 空（W1 起点，无前置门控） |
| 文件所有权 | primary：`skills/policy-route-execution/scripts/op_mapper.py` + `tests/test_op_mapper_rollback.py`；无 `dev_running` 冲突（W1 并行仅 01+04，文件互斥） |
| 回源方式 | 以 canonical HEAD 为基底逐块合并（非整文件覆盖）；工作区 G1 区块逐行 diff 核对 |
| 验证方式 | static-only 单测（mock `execute_op`），无真机，符合 CR-039 `security_constraints`（仅 static-only 验证） |

## 实现对象清单

| 对象 | 路径 | 改动类型 | 行号（改后） |
|---|---|---|---|
| ROLLBACK_STRATEGY 注释（语义声明） | `skills/policy-route-execution/scripts/op_mapper.py` | 修改 | L705-707 |
| `_query_static_route_id()` | `skills/policy-route-execution/scripts/op_mapper.py` | 新增 | L2340-2395 |
| `handle_rollback()` 静态路由分支 | `skills/policy-route-execution/scripts/op_mapper.py` | 修改（inverse_op 分支头部插入） | L2448-2478 |
| `HandleRollbackStaticRouteVerifyTests`（四分支 + 扩展） | `tests/test_op_mapper_rollback.py` | 新增测试类 | 文件尾部 |

**未触碰清单（硬约束核对）**：

- `EXPECTED_OP_COUNT`（L1060，仍为 44；44->45 归 STORY-039-03）
- G4 区块 `resolve_env_refs`（L1445 起，无 diff；归 STORY-039-02）
- G5 sw3 族区块（`_execute_sw3_op` / `_exec_sw3_telnet` / `_parse_sw3_pppoe_route` 等，无 diff；归 STORY-039-03）
- `INTENT_TO_OP` / `ARGS_TO_FLAGS` 等 dict（无 diff）
- `case_runner.py` / `SKILL.md` / `op-coverage-matrix.md` / CR 档案（未修改）
- 工作区参考文件（`/home/hyde/projects/ptm-te-manaul/.claude/skills/...`，只读）

## 设计契约映射

| technical-note 契约 | 实现位置 | 落实方式 |
|---|---|---|
| `_query_static_route_id(args, base_url, session_file, *, authorized, timeout) -> Optional[Any]` | op_mapper.py L2340 | 签名与工作区一致 |
| dst_network 必填，缺失返回 None | L2633-2635 对应逻辑 | `(args.get("dst_network") or "").strip()` 空即 None |
| 经 `fw_verify_static_route`（route_type + page=1/size=100）查询 | execute_op 调用，`step_name="rollback-fw_config_static_route-verify"`, `dry_run=False` | 与工作区一致 |
| vals 取自 `data.response.vals` 或 `data.vals` | 双形态 fallback，非 list 返回 None | 与工作区一致 |
| next_hop_addr 含 `${` 占位符时跳过匹配 | `if next_hop_addr and "${" in next_hop_addr: next_hop_addr = ""` | 不展开占位符，按 dst_network 继续 |
| 多路由匹配取首个非空 id | for 循环首个 `id not in (None, "", 0)` 返回 | 与工作区一致 |
| `handle_rollback` 静态路由分支无条件优先 verify | L2453 `if op_id == "fw_config_static_route":` 先于声明路径 / required_inputs / 兜底路径 | DQ-039-01 方案 A：无条件 verify，放弃 step-refs fallback |
| 无匹配返回 EXEC_FAILED envelope | reason 含「静态路由回滚：verify 查询未匹配到 {dst_network} 的 id，无法清理」 | 文案与工作区逐字一致 |
| 匹配则 `execute_op("fw_delete_static_route", {"id": rid}, ...)` | 透传 `env_topology` / `tg_api_server` | 与工作区一致 |
| L705-707 注释语义更新 | 「无条件 verify 查询获取；匹配失败即 EXEC_FAILED，无 step-refs fallback（DQ-039-01 方案 A）」 | 契约要求的语义声明 |
| 命名规范 | `_query_static_route_id` / `dst_network` / `next_hop_addr` / `rid` | 与工作区一致 |

**逐块对拍证据**：`diff` 校验 canonical `handle_rollback` 静态路由分支（L2453-2478）与工作区 L2729-2754 逐行一致（IDENTICAL）；`_query_static_route_id` 剔除注释行后与工作区 L2618-2671 逐行一致。仅 handle_rollback 分支前导注释比工作区多 2 行（补充 DQ-039-01 方案 A 与 CR-039 归属标注，属注释级增强，不影响逻辑）。

## 单元测试与 Fixture 计划

- 测试文件：`tests/test_op_mapper_rollback.py`，新增 `HandleRollbackStaticRouteVerifyTests`（5 个用例，mock `execute_op`，无真机）。
- Fixture：mock verify envelope 构造 `data.response.vals` / `data.vals` 两种形态；mock delete envelope 返回 success。

| 用例 | 分支 | 断言要点 |
|---|---|---|
| test_matched_returns_route_id | 有匹配 | delete 调用 `{"id": 11}`；verify 参数 route_type/page/size、`dry_run=False`；envelope success |
| test_multi_route_next_hop_disambiguates | 多路由 + next_hop_addr 消歧 | 同 dst_network 双路由（id 21/22），仅命中 next_hop 匹配项 id=22 |
| test_no_match_returns_exec_failed | 无匹配 | `error_type=EXEC_FAILED`；reason 含目标 dst_network 与「无法清理」；仅 verify 一次不触发 delete |
| test_placeholder_next_hop_skips_match | 占位符跳过 | next_hop_addr=`${ENV.router_ip}` 时仍按 dst_network 命中 id=41，不误判不匹配 |
| test_vals_direct_data_shape（扩展） | data.vals 形态 | verify envelope 无 response 包装时同样匹配（SM-039-02 之外的实现内部双形态覆盖） |

## 最小实现切片

| Slice | 内容 | 验证 |
|---|---|---|
| S1 | L705-707 注释语义更新 | 人工核对 + 单测（注释无行为影响） |
| S2 | `_query_static_route_id` 新增（与 S3 一起验证） | 单测四分支 |
| S3 | `handle_rollback` 静态路由无条件 verify 分支 | 单测四分支 + EXEC_FAILED envelope 断言 |
| S4 | 单测扩展 `HandleRollbackStaticRouteVerifyTests` | `uv run pytest` 32 passed |

## 本地对拍（工作区一致性）

- 构造与工作区相同形态的 verify envelope（`data.response.vals` 列表 + dst_network/next_hop_addr/id 字段）与 rollback args，断言 canonical 回源行为：无条件 verify -> 匹配取首个非空 id -> `fw_delete_static_route --id`；未匹配 -> EXEC_FAILED。四分支单测即对拍载体，行为与工作区实测基线（26/28 step PASS）一致。
- 逐行 diff：handle_rollback 静态路由分支与工作区 L2729-2754 完全一致；`_query_static_route_id` 逻辑（剔注释）与工作区 L2618-2671 完全一致。

## 验证结果

| 验证项 | 命令 | 结果 |
|---|---|---|
| Story 单测 | `uv run --python 3.11 pytest tests/test_op_mapper_rollback.py -x` | **32 passed**（原 27 + 新增 5） |
| mapping consistency 回归 | `uv run --python 3.11 pytest tests/test_tg_op_mapping.py tests/test_op_mapper_pppoe_client.py` | **55 passed**（含 `validate_mapping_consistency`，EXPECTED_OP_COUNT=44 不回归） |
| 语法检查 | `python -m py_compile op_mapper.py` | 通过 |
| 未触碰区块核对 | `git diff` 逐块审计 | 仅 G1 三处区块；G4/G5/EXPECTED_OP_COUNT/dict 无 diff |

## 未覆盖项

- 真机验证（verify 查询超时、大路由表分页 >100 条）：CR-039 为 static-only 验证，DQ-039-04 不授权真机；风险 RA-039-01 已记录回退条件（verify 超时则按 DQ-039-01 备选 B 重评，不静默改）。
- verify 返回超过 size=100 的路由分页遍历：与工作区实现一致（单页查询），工作区实测基线未出现该场景；如真机出现，属 RA-039-01 范畴。

## 设计缺口反馈

无。technical-note 契约完整可执行，无灰区遗留（DQ-039-01 方案 A 已 approve）。

## G6 M9 归属补记（T-3 台账项留痕）

按 `process/DEVELOPMENT-PLAN-CR-039.yaml` security_constraints 要求，G6 M9 归属为 T-3 台账项，在此补记留痕，不阻塞本 CR 门禁：

- **G6 M9**：指工作区改动集中「G6」分组（文档/追溯类）中的 M9 项，归属 STORY-039-04（文档与追溯）承载，非本 Story（G1）范围。本 Story 仅补记归属事实，M9 实体内容随 STORY-039-04 Wave 1 并行交付。

## sw3 用例样例来源说明

sw3 用例样例来源为工作区 CR-046 验证用例（node3_dut1_tg1_sw1_pppoe_link3），归 STORY-039-03（G5 sw3 族回源）与 STORY-039-04（env-file 样例）消费。本 Story（G1 静态路由）不涉及 sw3 用例，此处仅按 CR 约束留痕来源口径，避免后续 Story 重复溯源。

## 平台差异处理

N/A。本 Story 仅改动 canonical 源仓库 Python 脚本（ptm-team），回源后经 ptm-atomic 安装链生效，无平台目录/安装结构差异（canonical 源即交付源，非工作区 `.claude/skills` 旁路安装）。

## 后续交接（-> meta-qa / CP6）

- 验证入口：`uv run --python 3.11 pytest tests/test_op_mapper_rollback.py`（32 passed）；回归 `tests/test_tg_op_mapping.py` + `tests/test_op_mapper_pppoe_client.py`（55 passed，EXPECTED_OP_COUNT=44）。
- 风险提示：RA-039-01（大路由表 verify 超时 / dst_network 歧义）已由 next_hop_addr 消歧 + 单测缓解；真机超时回退按 DQ-039-01 备选 B 重评。
- 交接对象：CP6 编码完成检查（`process/checks/CP6-STORY-039-01-g1-static-route-rollback-verify-CODING-DONE.md`，由 checkpoint 流程生成）；CP7 验证消费本文件 + return packet。
- 下游依赖：STORY-039-02（G4）与 STORY-039-03（G5，merge_owner）串行复用 `op_mapper.py`；本 Story 已完成首个写入块，其 G1 区块（L705-707 注释 / L2340-2395 / L2448-2478）为后续 Story 的不可回退基线。
