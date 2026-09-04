---
cr_id: "CR-033"
story_id: "STORY-EX-07"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-07-LLD.md"
status: "implemented"
implemented_at: "2026-07-29T17:20:00+08:00"
author: "meta-dev"
---

# ST-EX-07 实现执行证据：fw_logout 会话清理（op_mapper 映射 + case_runner cleanup 登出 + 降级）

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 |
|---|---|---|---|
| OP_ID_TO_SUBCOMMAND.fw_logout | skills/policy-route-execution/scripts/op_mapper.py#OP_ID_TO_SUBCOMMAND | ST-EX-07 | 修改（新增 "fw_logout": ("auth", "logout")） |
| ARGS_TO_FLAGS.fw_logout | skills/policy-route-execution/scripts/op_mapper.py#ARGS_TO_FLAGS | ST-EX-07 | 修改（新增 "fw_logout": {}） |
| REQUIRED_FLAGS.fw_logout | skills/policy-route-execution/scripts/op_mapper.py#REQUIRED_FLAGS | ST-EX-07 | 修改（新增 "fw_logout": []） |
| ROLLBACK_STRATEGY.fw_logout | skills/policy-route-execution/scripts/op_mapper.py#ROLLBACK_STRATEGY | ST-EX-07 | 修改（新增 type=none） |
| OP_METADATA.fw_logout | skills/policy-route-execution/scripts/op_mapper.py#OP_METADATA | ST-EX-07 | 修改（新增 side_effect=observation） |
| EXPECTED_OP_COUNT | skills/policy-route-execution/scripts/op_mapper.py#EXPECTED_OP_COUNT | ST-EX-07 | 修改（21 -> 22，Gotcha #1） |
| validate_mapping_consistency auth 族校验 | skills/policy-route-execution/scripts/op_mapper.py#validate_mapping_consistency | ST-EX-07 | 修改（新增 fw_logout -> auth logout 校验） |
| check_fw_logout_available() | skills/case-execution/scripts/case_runner.py#check_fw_logout_available | ST-EX-07 | 新建 |
| cleanup_session(...) | skills/case-execution/scripts/case_runner.py#cleanup_session | ST-EX-07 | 新建 |
| _fallback_session_cleanup(...) | skills/case-execution/scripts/case_runner.py#_fallback_session_cleanup | ST-EX-07 | 新建 |
| main 接入点 | skills/case-execution/scripts/case_runner.py#main | ST-EX-07 接入 | 修改（启动校验 check + 全部用例后 cleanup_session） |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §3.1 fw_logout 映射 (auth, logout) | OP_ID_TO_SUBCOMMAND["fw_logout"] = ("auth", "logout") | ✓ 一致 |
| §3.1 ARGS_TO_FLAGS["fw_logout"] = {} | 新增空 dict（logout 无 args，session 由 --session-file 传递） | ✓ 一致 |
| §3.1 REQUIRED_FLAGS["fw_logout"] = [] | 新增空 list | ✓ 一致 |
| §3.1 ROLLBACK_STRATEGY type=none | "fw_logout": {"type": "none", "reason": "会话清理 op，本身是清理动作，不回滚"} | ✓ 一致 |
| §3.1 OP_METADATA side_effect=observation | "fw_logout": {"side_effect": "observation", "rollback": "", "idempotent": True} | ✓ 一致 |
| §3.1 EXPECTED_OP_COUNT=22（Gotcha #1） | EXPECTED_OP_COUNT = 22 | ✓ 一致 |
| §3.1 validate_mapping_consistency PASS | validate 通过（22 op_id 全覆盖，三表一致） | ✓ 一致 |
| §3.2 check_fw_logout_available（DQ-CP3-02） | subprocess ptm-atomic show fw_logout；returncode==0 且 stdout 含 fw_logout -> True | ✓ 一致 |
| §3.3 cleanup_session fw_logout 登出 + 降级 | 三分支：fw_logout_available=False -> 降级；True/None -> execute_op；OP_NOT_FOUND/EXEC_FAILED -> 降级 | ✓ 一致 |
| §3.3 dry-run 短路（authorized=False -> skipped） | if not authorized: return {"logout": "skipped", ...} | ✓ 一致 |
| §4.1 check_fw_logout_available 流程 | subprocess + TimeoutExpired/FileNotFoundError -> False | ✓ 一致 |
| §4.2 cleanup_session 主流程 | [0]dry-run 短路 [1]安装前验证降级 [2]execute_op [3]结果判定 | ✓ 一致 |
| §4.2 _fallback_session_cleanup | os.remove(session_file)；OSError -> failed | ✓ 一致 |
| §4.3 build_command fw_logout 复用 | build_command("fw_logout", {}, ...) -> ["ptm-atomic", "run", ..., "auth", "logout", "--execute"] | ✓ 一致 |
| §4.4 main 调用顺序 | [1]启动校验 check_fw_logout_available [6]全部用例后 cleanup_session | ✓ 一致 |
| §5.1 降级链路（ADR-04） | fw_logout_available=False -> 降级；OP_NOT_FOUND/EXEC_FAILED -> 降级 | ✓ 一致 |
| §12.1 NO_CREDENTIAL_READ | cleanup_session 不读 session 内容，仅 os.remove | ✓ 一致 |
| §12.1 session 路径 | ~/.local/state/ptm-atomic/session-<run-id>.json | ✓ 一致 |

## 3. 测试 Fixture

同 ST-EX-04（共用 /tmp/cr033-test/ fixtures）

## 4. 最小实现切片

- op_mapper fw_logout 映射：五表新增 + EXPECTED_OP_COUNT=22 + validate auth 族校验
- check_fw_logout_available()：subprocess ptm-atomic show fw_logout，超时/不存在 -> False
- cleanup_session()：dry-run 短路 + fw_logout 登出 + 降级 os.remove + 结果判定
- _fallback_session_cleanup()：os.remove + OSError 容错

## 5. 本地验证

| 验证项 | 结果 |
|---|---|
| op_mapper validate（TC-EX-07-03/16） | PASS（22 op_id 全覆盖，三表一致） |
| EXPECTED_OP_COUNT=22（TC-EX-07-03） | validate 输出 "22 op_id 全覆盖" |
| fw_logout 四表映射（TC-EX-07-01） | OP_ID_TO_SUBCOMMAND/ARGS_TO_FLAGS/REQUIRED_FLAGS/ROLLBACK_STRATEGY 均含 fw_logout |
| OP_METADATA 一致（TC-EX-07-02） | side_effect=observation/rollback=""/idempotent=True |
| check_fw_logout_available（TC-EX-07-06/07） | result.json fw_logout_available=False（ptm-atomic 安装版未暴露） |
| cleanup_session dry-run 短路（TC-EX-07-12） | result.json logout=skipped（authorized=False） |
| 降级路径就绪（TC-EX-07-10/11） | fw_logout_available=False -> _fallback_session_cleanup（代码审查） |
| handle_rollback fw_logout 跳过（TC-EX-07-15） | ROLLBACK_STRATEGY type=none -> not_required（代码审查） |
| build_command fw_logout（TC-EX-07-04/05） | family="auth" -> ["auth", "logout"]；dry_run 不加 --execute（代码审查） |

## 6. 平台差异

- fw_logout 映射是数据常量（无平台差异）
- cleanup_session 逻辑纯 Python（subprocess + os.remove），跨平台一致
- check_fw_logout_available 调用 ptm-atomic CLI，依赖 ptm-atomic 安装

## 7. 交接摘要

- ST-EX-07 实现完成，fw_logout 映射 + cleanup_session + 降级就绪
- op_mapper EXPECTED_OP_COUNT=22，validate PASS（含 fw_logout）
- check_fw_logout_available 预验证：ptm-atomic 安装版未暴露 fw_logout（fw_logout_available=False）
- 降级路径就绪：fw_logout_available=False -> os.remove(session_file)（ADR-04）
- dry-run 短路验证通过（logout=skipped）
- session 安全：NO_CREDENTIAL_READ（仅 os.remove，不读 session 内容）
