---
cr_id: "CR-033"
story_id: "STORY-EX-05"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-05-LLD.md"
status: "implemented"
implemented_at: "2026-07-29T17:20:00+08:00"
author: "meta-dev"
---

# ST-EX-05 实现执行证据：dry-run 默认门 + --execute 授权门 + runtime_authorization 审计

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 |
|---|---|---|---|
| authorize(args, addresses) | skills/case-execution/scripts/case_runner.py#authorize | ST-EX-05#authorize | 新建 |
| execute_steps(...) | skills/case-execution/scripts/case_runner.py#execute_steps | ST-EX-05#execute_steps | 新建 |
| _build_dry_run_envelope | skills/case-execution/scripts/case_runner.py#_build_dry_run_envelope | ST-EX-05 辅助 | 新建 |
| _build_step_error | skills/case-execution/scripts/case_runner.py#_build_step_error | ST-EX-05 辅助 | 新建 |
| _map_build_err | skills/case-execution/scripts/case_runner.py#_map_build_err | ST-EX-05 辅助 | 新建 |
| main 接入点 | skills/case-execution/scripts/case_runner.py#main | ST-EX-05 接入 | 修改（authorize + execute_steps 调用） |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §3.1 authorize() --execute 无 --authorized -> sys.exit(2) | authorize(): not args.execute -> None; not args.authorized -> sys.exit(2) | ✓ 一致 |
| §3.1 run 级 runtime_authorization（who/scope/authorized_at/reason/target_devices） | authorize() 返回 dict 含 5 字段 | ✓ 一致 |
| §3.2 execute_steps dry-run 门（不连设备，build_command 打印） | dry_run=True: resolve_env_refs + resolve_step_refs + build_command，不调 subprocess | ✓ 一致 |
| §3.2 --execute 分支调 execute_op(dry_run=False, authorized=True) | dry_run=False + runtime_auth 非 None: execute_op(..., dry_run=False, authorized=True) | ✓ 一致 |
| §3.3 dry-run 门绕过 subprocess（LCQ-ST-EX-05-01 方案 A） | dry_run=True 分支直接 build_command，不调 execute_op | ✓ 一致（方案 A） |
| §4.1 authorize() 流程 | 三分支：无 --execute -> None；无 --authorized -> exit(2)；--execute --authorized -> dict | ✓ 一致 |
| §4.2 execute_steps [1] resolve_env_refs（dry-run 也执行） | dry_run 分支前先调 resolve_env_refs（Gotcha #12） | ✓ 一致 |
| §4.2 execute_steps [2] resolve_step_refs | step_refs_dir 非空时调 resolve_step_refs | ✓ 一致 |
| §4.2 execute_steps [3] dry-run build_command | build_command(dry_run=True) -> _build_dry_run_envelope | ✓ 一致 |
| §4.2 execute_steps [4] --execute execute_op | execute_op(dry_run=False, authorized=True, ...) | ✓ 一致 |
| §4.3 dry-run step-refs 不落盘 | dry-run 门绕过 execute_op（_write_step_ref 不触发） | ✓ 一致 |
| §4.4 runtime_authorization 双层审计 | run 级 authorize() + op 级 execute_op 内部（op_mapper L1322-1329） | ✓ 一致 |
| §5.1 异常分类 | OpNotFoundError->OP_NOT_FOUND; ValidationError->PARAM_INVALID; ValueError->VALIDATION_FAILED | ✓ 一致 |
| fw_login SHARED_SESSION（Gotcha #8） | --execute 模式 fw_login_web_management step 标 skipped | ✓ 一致 |
| timeout 分层（Gotcha #9） | step.target=="tg" -> tg_timeout(90); else timeout(45) | ✓ 一致 |

## 3. 测试 Fixture

同 ST-EX-04（共用 /tmp/cr033-test/ fixtures）

## 4. 最小实现切片

- authorize()：纯函数，解析 argparse Namespace，返回 None 或 run 级授权 dict
- execute_steps()：dry-run 门 + --execute 分支，返回 List[envelope]
- dry-run envelope: status="dry_run", data={"command": command}
- --execute envelope: execute_op 返回值（含 op 级 runtime_authorization）

## 5. 本地验证

| 验证项 | 结果 |
|---|---|
| dry-run 默认门不连设备（TC-EX-05-01） | 所有 step status=dry_run，command 无 --execute，0 次 subprocess |
| --execute 无 --authorized exit(2)（TC-EX-05-02） | exit_code=2，stderr 含授权缺失 |
| dry-run resolve_env_refs 执行（TC-EX-05-04） | ${ENV.dut.next_hop} -> 192.168.102.1，${ENV.tg.ports[...]} -> ["1/1/1","1/1/2"] |
| dry-run fw_login 只构建命令（TC-EX-05-05） | step status=dry_run，command 含 "auth login"，无 --execute |
| dry-run ${STEP-N.id} 解析失败（TC-EX-05-06） | dry-run 不落盘 step-refs，${STEP-N.id} 引用会 VALIDATION_FAILED（LCQ 方案 A） |
| runtime_auth None（dry-run） | result.json runtime_authorization=null |

## 6. 平台差异

- dry-run 门纯 Python（无 subprocess/网络），WSL2/Linux 跨平台一致
- --execute 模式 subprocess 由 op_mapper 处理，NO_PROXY 已含设备 IP

## 7. 交接摘要

- ST-EX-05 实现完成，dry-run 默认门 + --execute 授权门 + runtime_authorization 双层审计就绪
- dry-run 门验证：0 次 subprocess，resolve_env_refs 仍执行（Gotcha #12）
- --execute 授权门验证：无 --authorized 时 exit(2)
- 下游 ST-EX-06/10/11 消费 execute_steps 返回的 envelopes
