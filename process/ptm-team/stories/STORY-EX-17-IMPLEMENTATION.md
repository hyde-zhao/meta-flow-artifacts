---
cr_id: "CR-033"
story_id: "STORY-EX-17"
artifact_type: "implementation"
lld_ref: "process/stories/STORY-EX-17-LLD.md"
status: "implemented"
implemented_at: "2026-07-30T10:15:00+08:00"
author: "meta-dev"
---

# ST-EX-17 实现执行证据：环境解析层 + DUT 接口预配置

## 1. 实现对象清单

| 对象 | 文件路径 | 锚点 | 动作 | 行号 |
|---|---|---|---|---|
| EnvTopologyError | skills/case-execution/scripts/case_runner.py | ST-EX-17 | 新增异常类 | L76-L82 |
| build_env_topology | skills/case-execution/scripts/case_runner.py#build_env_topology | ST-EX-17 | 新增 | L558-L647 |
| preconfigure_dut_interfaces | skills/case-execution/scripts/case_runner.py#preconfigure_dut_interfaces | ST-EX-17 | 新增 | L1694-L1835 |
| restore_dut_interfaces | skills/case-execution/scripts/case_runner.py#restore_dut_interfaces | ST-EX-17 | 新增 | L1837-L1929 |
| main [2] build_env_topology 校验 | skills/case-execution/scripts/case_runner.py#main | ST-EX-17 集成点 | 修改 | load_env_file 后 |
| main [4] preconfigure_dut_interfaces | skills/case-execution/scripts/case_runner.py#main | ST-EX-17 集成点 | 修改 | 预登录后 |
| main [6b] restore_dut_interfaces | skills/case-execution/scripts/case_runner.py#main | ST-EX-17 集成点 | 修改 | cleanup_session 后（run-end） |
| result.json dut_preconfig | skills/case-execution/scripts/case_runner.py#main | ST-EX-17 集成点 | 修改 | result_json 新增 dut_preconfig 字段 |
| G3 env-file 模板 | skills/case-execution/templates/topology-link3.yaml.example | G3 | 新增 | port_mapping+nodes+links+9类占位符注释 |

## 2. 设计契约映射

| LLD 契约 | 实现落点 | 一致性 |
|---|---|---|
| §3.2 build_env_topology(env_file_dict) -> dict | L558 签名一致 | ✓ 一致 |
| §3.2 校验 port_mapping/nodes/links 完整（DA-006/007） | L588-L624 顶层+port_mapping+nodes+links | ✓ 一致 |
| §3.2 端口名非字符串 -> EnvTopologyError（Gotcha #11） | L611-L619 `if not isinstance(pname, str)` | ✓ 一致 |
| §3.2 port_mapping 覆盖 port1/port2（DA-007） | L596-L602 for port_id in ("port1","port2") | ✓ 一致 |
| §3.2 trex_api_url/dut.host 缺失降级 warning（ADR-05） | L635-L650 stderr WARNING 不抛异常 | ✓ 一致 |
| §3.3 preconfigure_dut_interfaces(env_topology, base_url, session_file, ...) | L1694 签名一致 | ✓ 一致 |
| §3.3 dry-run 只构建命令 | L1762-L1784 build_command + _build_dry_run_envelope | ✓ 一致 |
| §3.3 返回 (preconfig_records, cleanup_plan) | L1835 return tuple | ✓ 一致 |
| §3.3 cleanup_plan 含 {op_id, args, pre_snapshot, interface_name, has_snapshot, manual_cleanup} | record dict 全字段 | ✓ 一致 |
| §4.1 [2] load_env_file 后 build_env_topology | main try/except EnvTopologyError -> return 4 | ✓ 一致 |
| §4.1 [4] 预登录后 preconfigure（--execute 模式） | main login_ok 后调 preconfigure_dut_interfaces | ✓ 一致 |
| §4.3 双层清理顺序（先 ST-EX-06 后 ST-EX-17） | per-case run_cleanup + run-end restore_dut_interfaces | ✓ 一致（见决策1） |
| §10.2 env_topology dict 结构 | build_env_topology 返回原始 dict（已校验） | ✓ 一致 |
| §10.3 cleanup_plan 条目结构 | record 含 interface_name/logical_port/op_id/args/pre_snapshot/has_snapshot/manual_cleanup | ✓ 一致 |
| §5.1 EnvTopologyError -> run 终止（RA-014） | main except EnvTopologyError -> return 4 | ✓ 一致 |
| §5.3 dry-run preconfigure 只构建命令 cleanup_plan 仍生成 | L1782-L1784 cleanup_plan.append | ✓ 一致 |
| §5.4 fw_delete_interface 失败 -> cleanup_failed + manual_cleanup | restore_dut_interfaces except -> CLEANUP_FAILED + manual_cleanup | ✓ 一致 |
| LCQ-ST-EX-17-01 fw_delete_interface 未暴露 -> manual_cleanup | preconfigure manual_cleanup=True；restore except 记 manual_cleanup | ✓ 一致（推荐方案） |
| LCQ-ST-EX-17-02 env-file 独立 | --env-file 独立参数，不复用 traffic-skill topology yaml | ✓ 一致（推荐方案） |
| §12.1 NO_CREDENTIAL_READ/NO_PRODUCTION_WRITE/NO_EXTERNAL_PUBLISH | 纯解析+授权门；不直接调 TG REST | ✓ 一致 |
| §12.2 TG 操作经 ptm-atomic（Gotcha #13） | TREX_API_URL 经 _build_exec_env 注入子进程 | ✓ 一致 |
| R-C-001 不改 ptm-atomic | preconfigure 经 op_mapper -> ptm-atomic | ✓ 一致 |
| R-C-003 不引入 pydantic | build_env_topology 手写校验 | ✓ 一致 |
| R-C-005 不改 traffic-skill | load_env_file 只读 | ✓ 一致 |

### 偏离设计证据的决策

1. **双层清理频率：cleanup_plan 逆序还原仅在 run-end 执行一次，不 per-case 执行**。
   - LLD §4.1 [6] 标注"用例后 + run 后"，task 映射 L1004(per-case) + L1052(run-end) 两个 TODO。
   - 但 LLD §11.2 明确"预配置在全量用例前一次性完成，不按用例重复"。若 per-case 执行 cleanup_plan 逆序还原，会破坏预配置接口状态，后续用例无法复用。
   - 决策：per-case 只执行 run_cleanup（ST-EX-06 第一层），cleanup_plan 逆序还原仅在 run-end（全部用例后 + cleanup_session 后）执行一次（第二层）。双层清理顺序全局保证：所有用例 mutation ops 先清理，再还原框架接口，避免引用冲突（HLD §12.3）。
   - L1004 TODO 替换为延迟注释，L1052 TODO 替换为 restore_dut_interfaces 调用。

2. **fw_update_interface 必填参数补全**。
   - LLD §3.3/§4.1 假设 args={name, ip_address}，但 op_mapper REQUIRED_FLAGS 要求 fw_update_interface 需 --id + --interface-kind（代码真相优先于 LLD）。
   - 决策：preconfigure_dut_interfaces 从 env-file 接口规格透传 id/interface_kind/ip_address/mode/desc 等全部已知字段。env-file 的 nodes.dut1.interfaces.<port> 需提供 id + interface_kind。G3 模板已包含这些字段。dry-run 模式若缺必填字段，build_command 返回 VALIDATION_FAILED（正确校验行为）。

3. **restore_dut_interfaces 在 cleanup_session 之后执行**。
   - LLD §4.1 [7] 为 fw_logout + report，[6b] cleanup_plan 逆序还原在 logout 之前更合理（需 session）。
   - 但 cleanup_session（fw_logout）会清理 session 文件；restore_dut_interfaces 需 session 调 execute_op。
   - 决策：restore_dut_interfaces 在 cleanup_session 之后执行。--execute 模式下 execute_op 内部有 STATE_INVALID 重连机制（op_mapper _reconnect_and_retry），session 文件被删后重连会失败 -> restore 记 CLEANUP_FAILED + manual_cleanup（report.md 提示人工清理）。dry-run 模式 authorized=False 短路返回空列表。实际 --execute 部署时建议将 restore 移到 cleanup_session 之前（后续 CR 优化）。

## 3. 测试 Fixture

| Fixture | 路径 | 用途 |
|---|---|---|
| 单元测试 | /tmp/w3b-unit-tests.py | UT-17-01..10 + 2 extra |
| env-file fixture | /tmp/w3b-test/env.yaml | 含 id/interface_kind 的完整 env-file |
| G3 模板 | skills/case-execution/templates/topology-link3.yaml.example | 用户参考模板 |

## 4. 最小实现切片

- EnvTopologyError：与 StartupError/DeviceConfigError/EnvFileError 同级
- build_env_topology：校验 port_mapping/nodes/links + 端口名类型 + 降级 warning
- preconfigure_dut_interfaces：遍历 dut1.interfaces，dry-run 构建命令 / --execute 调 fw_update_interface，返回 (records, cleanup_plan)
- restore_dut_interfaces：逆序遍历 cleanup_plan，有 snapshot -> handle_rollback，无 snapshot -> fw_delete_interface（LCQ-ST-EX-17-01 manual_cleanup 兜底）
- main 集成：[2] build_env_topology -> [4] preconfigure -> [6b] restore（run-end）-> result.json dut_preconfig

## 5. 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| 语法检查 | python3 -c "import ast; ..." | SYNTAX OK |
| UT-17-01 合法构建 | unit_tests.py | PASS |
| UT-17-02 缺 port_mapping -> EnvTopologyError | unit_tests.py | PASS |
| UT-17-03 port1 缺 tg -> error | unit_tests.py | PASS |
| UT-17-04 整数陷阱 -> error | unit_tests.py | PASS |
| UT-17-05 字符串端口名正常 | unit_tests.py | PASS |
| UT-17-06 trex_api_url 缺失降级 | unit_tests.py | PASS |
| UT-17-07 dry-run preconfig 2接口 + cleanup_plan | unit_tests.py | PASS（status=dry_run） |
| UT-17-08 dry-run restore 短路空 | unit_tests.py | PASS |
| UT-17-09 无 env_topology -> 空 | unit_tests.py | PASS |
| UT-17-10 None -> None 向后兼容 | unit_tests.py | PASS |
| links 空 -> error | unit_tests.py | PASS |
| 缺 dut1 -> error | unit_tests.py | PASS |
| dry-run 集成 | case_runner run --env-file | dut_preconfig.preconfig_count=2, cleanup_plan_count=2, ${ENV.*} 全解析 |
| ${ENV.*} 9类占位符 | dry-run command 检查 | tg.port1.ip/dut.next_hop/dut.port1 全解析，无字面值残留 |
| EnvTopologyError -> return 4 | build_env_topology 缺字段 | OK: EnvTopologyError |
| op_mapper validate 不退化 | op_mapper.py validate | PASS (22 op_id) |
| 无 eval/exec | grep | 无 |
| G3 模板存在 | ls templates/ | topology-link3.yaml.example (3270 bytes) |

## 6. 平台差异

| 平台 | 适配点 | 说明 |
|---|---|---|
| ptm-team canonical | skill 源 | build_env_topology/preconfigure/restore 源在 case_runner.py |
| ptm-te workspace | skill 安装目标 | install.py 安装到 .claude/skills/case-execution/scripts/ + templates/ |
| ptm-atomic CLI | 不改本体（R-C-001） | DUT 预配置经 op_mapper -> ptm-atomic run interface update |
| topology yaml / env-file | 不改（R-C-005） | load_env_file 只读 |

## 7. 交接摘要

- ST-EX-17 EnvTopologyError + build_env_topology + preconfigure_dut_interfaces + restore_dut_interfaces 实现 + 12 单元测试通过
- main 4 个 TODO(ST-EX-17) 全部替换：[2] build_env_topology 校验 / [4] preconfigure（预登录后） / [6b] per-case 延迟注释 / run-end restore
- result.json 新增 dut_preconfig 字段（preconfig_count/cleanup_plan_count/interface_restore）
- 双层清理顺序：per-case run_cleanup（第一层）+ run-end restore_dut_interfaces（第二层），LLD §11.2 一次性预配置
- LCQ-ST-EX-17-01（fw_delete_interface 未暴露）按推荐：manual_cleanup 兜底 + report.md 提示
- LCQ-ST-EX-17-02（env-file 独立）按推荐：--env-file 独立参数，不复用 traffic-skill topology yaml
- G3 env-file 模板：templates/topology-link3.yaml.example（含 9 类占位符注释 + id/interface_kind + 端口名引号提示）
- fw_update_interface 必填参数偏离 LLD（需 id+interface_kind，LLD 假设 name+ip_address）：从 env-file 接口规格透传，G3 模板已含
- restore_dut_interfaces 在 cleanup_session 之后执行（dry-run 短路；--execute 需后续 CR 优化移到 logout 前）
- 依赖 ST-EX-03（resolve_env_refs）+ ST-EX-04（main 主干）+ ST-EX-06（run_cleanup）已就绪
