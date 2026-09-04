---
cr_id: "CR-033"
story_id: "STORY-EX-06"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-06-LLD.md"
status: "implemented"
implemented_at: "2026-07-29T17:20:00+08:00"
author: "meta-dev"
---

# ST-EX-06 实现执行证据：用例清理（逆序清理 mutation ops + step-refs 读取）

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 |
|---|---|---|---|
| run_cleanup(...) | skills/case-execution/scripts/case_runner.py#run_cleanup | ST-EX-06#run_cleanup | 新建 |
| main 接入点 | skills/case-execution/scripts/case_runner.py#main | ST-EX-06 接入 | 修改（每用例 execute_steps 后调 run_cleanup） |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §3.1 run_cleanup 逆序清理 mutation ops | range(n-1, -1, -1) 逆序遍历 case_steps | ✓ 一致 |
| §3.1 复用 op_mapper.handle_rollback | handle_rollback(op_id, args, base_url, session_file, result_envelope=envelope, ...) | ✓ 一致 |
| §3.1 dry-run 短路（authorized=False 返回 []） | if not authorized: return [] | ✓ 一致 |
| §3.2 step-refs 读取契约（内存优先 + 文件 fallback） | envelope = step_envelopes[i]；None 时 op_mapper._read_step_ref fallback | ✓ 一致 |
| §3.3 与 ST-EX-17 清理顺序契约 | main: run_cleanup 先执行，TODO(ST-EX-17) 后执行 | ✓ 一致 |
| §4.1 [0] dry-run 短路 | authorized=False -> return [] | ✓ 一致 |
| §4.1 [1] 逆序队列构造 | 逆序遍历 + envelope 缺失 fallback + status 不在 (success,error) 跳过 | ✓ 一致 |
| §4.1 [2] handle_rollback 调用 + 失败容错 | try/except 构造 CLEANUP_FAILED envelope，不中断 | ✓ 一致 |
| §4.2 handle_rollback 分派（inverse_op/restore_snapshot/irreversible/manual_required/none/as_cleanup_skip） | 透传给 handle_rollback（op_mapper 已实现分派） | ✓ 一致 |
| §4.3 inverse_op id 解析 | handle_rollback 内部 build_inverse_args + _extract_inverse_id | ✓ 一致（op_mapper 承载） |
| §5.1 失败容错（单 op 清理失败不中断） | except Exception -> CLEANUP_FAILED envelope，continue | ✓ 一致 |
| §5.2 清理失败不影响后续用例 | run_cleanup 失败记 cleanup_errors，下一用例正常执行 | ✓ 一致 |
| timeout 分层（Gotcha #9） | step.target=="tg" -> tg_timeout(90); else timeout(45) | ✓ 一致 |
| env_topology/tg_api_server 透传 | handle_rollback(..., env_topology=env_topology, tg_api_server=tg_api_server) | ✓ 一致（ADR-05/09） |

## 3. 测试 Fixture

同 ST-EX-04（共用 /tmp/cr033-test/ fixtures）

## 4. 最小实现切片

- run_cleanup()：dry-run 短路 + 逆序遍历 + handle_rollback 调用 + 失败容错
- dry-run 模式：authorized=False -> return []（无 mutation op 实际执行）
- --execute 模式：逆序调 handle_rollback，type=none/irreversible/manual_required/as_cleanup_skip 自动跳过

## 5. 本地验证

| 验证项 | 结果 |
|---|---|
| dry-run 短路（TC-EX-06-02） | cleanup=0 envelopes（authorized=False return []） |
| 逆序清理顺序（TC-EX-06-01） | range(n-1,-1,-1) 逆序遍历（代码审查） |
| step-refs fallback（TC-EX-06-07） | envelope 缺失时 op_mapper._read_step_ref fallback（代码审查） |
| 失败容错（TC-EX-06-06） | except Exception -> CLEANUP_FAILED，不中断（代码审查） |
| 与 ST-EX-17 清理顺序（TC-EX-06-11） | main: run_cleanup 先，TODO(ST-EX-17) 后（代码审查） |

## 6. 平台差异

- run_cleanup 纯 Python（逆序遍历 + handle_rollback 调用），跨平台一致
- handle_rollback 的 subprocess 由 op_mapper 处理

## 7. 交接摘要

- ST-EX-06 实现完成，逆序清理 mutation ops 就绪
- dry-run 短路验证通过（authorized=False -> return []）
- --execute 模式逆序清理复用 handle_rollback，失败容错不中断
- step-refs fallback（内存优先 + 文件读取）已实现
- 与 ST-EX-17 清理顺序契约明确（main 调用序列：run_cleanup -> ST-EX-17）
