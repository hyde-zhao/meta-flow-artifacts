---
cr_id: "CR-033"
artifact_type: "feature-tasks"
feature_id: "FE-EX-02"
feature_name: "case-execution 执行引擎"
version: "1.0"
created_at: "2026-07-28T16:10:00+08:00"
author: "meta-se"
status: "draft"
source_design: "docs/features/case-execution/DESIGN.md"
source_dev_plan: "process/DEVELOPMENT-PLAN.yaml"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md"
---

# FE-EX-02 case-execution 执行引擎 Feature 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CP3 approved / CP4 PASS 后生成：Feature 级任务清单 TASK-EX-FE02-*；与 13 Story 关联；与 DEVELOPMENT-PLAN file_ownership 对齐 |

## 任务清单

> 任务 ID 格式 `TASK-EX-FE02-<seq>`。每项任务关联 Story 与 file_ownership（DEVELOPMENT-PLAN.yaml 真相源）。
> 实现顺序遵循 Story DAG（DEVELOPMENT-PLAN depends_on）：Wave2 ST-EX-04 -> 05/06/07 -> Wave3 08/09/10/11/12/15/16/17 -> Wave4 14。

### TASK-EX-FE02-01: case_runner.py 骨架与三入口（ST-EX-04，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 实现 case_runner.py main(argparse) + discover_cases + parse_case_file + resolve_addresses |
| 关联 Story | ST-EX-04 |
| file_ownership | skills/case-execution/scripts/case_runner.py#main, #discover_cases, #parse_case_file, #resolve_addresses |
| 依赖 | ST-EX-03（op_mapper resolve_env_refs + _build_exec_env 签名扩展） |
| 完成准则 | 三入口(--case-file/--cases-dir)可用；devices.yaml 取址 firewall.host+tg.api_server；case_steps YAML 解析；import op_mapper 路径正确（Gotcha #2） |
| 验证 | dry-run 24 用例发现 + 解析 |

### TASK-EX-FE02-02: case-execution 安装验证（ST-EX-04，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | install.py PTM_TE_SKILLS 新增 "case-execution" |
| 关联 Story | ST-EX-04 |
| file_ownership | script/ptm_team/install.py#PTM_TE_SKILLS |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | PTM_TE_SKILLS 含 case-execution；install/uninstall/install 循环验证 skill 不丢失（Gotcha #10） |
| 验证 | 重装一致性（TEST-PLAN TP-FE02-06） |

### TASK-EX-FE02-03: load_env_file 加载 --env-file（ST-EX-04，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 实现 case_runner.load_env_file（加载 --env-file -> env_topology，None 时返回 None 向后兼容） |
| 关联 Story | ST-EX-04 |
| file_ownership | skills/case-execution/scripts/case_runner.py#load_env_file |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | env-file 加载为 env_topology；env_file=None 返回 None（Gotcha #12） |
| 验证 | dry-run env-link3.yaml 加载 |

### TASK-EX-FE02-04: dry-run 默认门 + --execute 授权门（ST-EX-05，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 实现 execute_steps + authorize（dry-run 默认门 / --execute --authorized / runtime_authorization 审计） |
| 关联 Story | ST-EX-05 |
| file_ownership | skills/case-execution/scripts/case_runner.py#execute_steps, #authorize |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | dry-run 默认不执行写操作；mutation op 只构建命令；--execute 需 authorized=True；runtime_authorization 审计字段(who/scope/authorized_at/reason)；fw_login step dry-run 跳过实际登录（Gotcha #5） |
| 验证 | dry-run + review（TEST-PLAN TP-FE02-04） |

### TASK-EX-FE02-05: 逆序清理 mutation ops（ST-EX-06，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 实现 run_cleanup（按 config 顺序逆序执行 inverse_op / step-refs 读取 / irreversible 不回滚） |
| 关联 Story | ST-EX-06 |
| file_ownership | skills/case-execution/scripts/case_runner.py#run_cleanup |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | mutation ops 逆序清理；step-refs ${STEP-N.id} 正确引用；irreversible(reset-hitcount) 不回滚 |
| 验证 | dry-run + review |

### TASK-EX-FE02-06: fw_logout op 映射 + 降级（ST-EX-07，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | op_mapper 新增 fw_logout 映射(auth,logout) + ARGS_TO_FLAGS + ROLLBACK_STRATEGY + EXPECTED_OP_COUNT 21->22；case_runner cleanup_session（登出 + 降级 session 文件清理） |
| 关联 Story | ST-EX-07 |
| file_ownership | skills/policy-route-execution/scripts/op_mapper.py#OP_ID_TO_SUBCOMMAND.fw_logout, #ARGS_TO_FLAGS.fw_logout, #ROLLBACK_STRATEGY.fw_logout, #EXPECTED_OP_COUNT; skills/case-execution/scripts/case_runner.py#cleanup_session |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | OP_ID_TO_SUBCOMMAND 含 fw_logout; EXPECTED_OP_COUNT=22; validate_mapping_consistency 通过（Gotcha #1）；安装前 `ptm-atomic show fw_logout` 验证；未暴露降级 os.remove(session_file)（Gotcha #4, ADR-04） |
| 验证 | static + review（TEST-PLAN TP-FE02-08） |

### TASK-EX-FE02-07: ARP 预热引擎强制（ST-EX-08，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 实现 apply_warming_up（warming_up:true 强制 post_op tg_stop_traffic_stream / 参数继承 / auto_post_op 标记） |
| 关联 Story | ST-EX-08 |
| file_ownership | skills/case-execution/scripts/case_runner.py#apply_warming_up |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | warming_up=true 时主 op 后强制 post_op；参数从主 op 继承(ports/txport/rxport/name)；result 标记 auto_post_op=true（ADR-06 引擎侧） |
| 验证 | dry-run（TEST-PLAN SCN-EX-05） |

### TASK-EX-FE02-08: retry 轮询（ST-EX-09，technical-note）

| 字段 | 值 |
|---|---|
| 任务 | 实现 apply_retry（retry 字段 interval/count/until 解析 + 轮询执行） |
| 关联 Story | ST-EX-09 |
| file_ownership | skills/case-execution/scripts/case_runner.py#apply_retry |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | retry 配置正确解析；按 interval/count 轮询；until 条件判定 |
| 验证 | dry-run + review |

### TASK-EX-FE02-09: known_issue 四态分级（ST-EX-10，technical-note）

| 字段 | 值 |
|---|---|
| 任务 | 实现 classify_result（PASS/FAIL/KNOWN_FAIL/ERROR 四态判定，优先级 ERROR>KNOWN_FAIL>FAIL>PASS） |
| 关联 Story | ST-EX-10 |
| file_ownership | skills/case-execution/scripts/case_runner.py#classify_result |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | known_issue=true 且 fail -> KNOWN_FAIL；四态优先级正确 |
| 验证 | dry-run + review（TEST-PLAN SCN-EX-09） |

### TASK-EX-FE02-10: 失败诊断 + 结构化报告（ST-EX-11，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 实现 diagnose_failure + generate_report（失败字段提取 + report.md 四态统计 + 幂等容错记录） |
| 关联 Story | ST-EX-11 |
| file_ownership | skills/case-execution/scripts/case_runner.py#generate_report, #diagnose_failure |
| 依赖 | TASK-EX-FE02-01, TASK-EX-FE02-09（ST-EX-10） |
| 完成准则 | 失败诊断提取 error_type/error_code/reason/details/command；report.md 模板完整（DESIGN §6.3）；幂等容错记录（对象已存在/流不存在） |
| 验证 | review（TEST-PLAN SCN-EX-17） |

### TASK-EX-FE02-11: extract_payload 统一解析（ST-EX-12，technical-note）

| 字段 | 值 |
|---|---|
| 任务 | 实现 extract_payload(op_id, envelope)（按 op_id 字段提取表，DESIGN §6.1） |
| 关联 Story | ST-EX-12 |
| file_ownership | skills/case-execution/scripts/case_runner.py#extract_payload |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | 22 op_id 字段提取表覆盖；policy_route_id/object_id/interface_id/template_id/stream_handle/tx/rx/loss_ratio 提取正确 |
| 验证 | review（TEST-PLAN TP-FE02-03） |

### TASK-EX-FE02-12: verify_loss 消费侧（ST-EX-14，technical-note）

| 字段 | 值 |
|---|---|
| 任务 | extract_payload 扩展 tg_verify_traffic_loss 提取 tx/rx/loss_ratio |
| 关联 Story | ST-EX-14 |
| file_ownership | skills/case-execution/scripts/case_runner.py#extract_payload.tg_verify_traffic_loss |
| 依赖 | TASK-EX-FE02-11（ST-EX-12） |
| 完成准则 | tg_verify_traffic_loss -> data.tx_count/data.rx_count/data.loss_ratio 提取 |
| 验证 | review（TEST-PLAN SCN-EX-25） |

### TASK-EX-FE02-13: 用例结构化约定（ST-EX-15，technical-note）

| 字段 | 值 |
|---|---|
| 任务 | SKILL.md 用例结构化约定段 + parse_frontmatter（目录三级 + 命名正则 + frontmatter 16 列） |
| 关联 Story | ST-EX-15 |
| file_ownership | skills/case-execution/SKILL.md#用例结构化约定; skills/case-execution/scripts/case_runner.py#parse_frontmatter |
| 依赖 | TASK-EX-FE02-01 |
| 完成准则 | 目录 cases/特性/配置管理/策略配置/；命名正则 ^PC-[A-Z0-9]+-\d+-\d+-\d+（Gotcha #6, ADR-07）；frontmatter 8 必填+8 可选，缺失填 N/A；忽略测试步骤/预期结果列（Gotcha #7, AGA-03=C） |
| 验证 | review（TEST-PLAN SCN-EX-21） |

### TASK-EX-FE02-14: 标签/关键字执行（ST-EX-16，technical-note）

| 字段 | 值 |
|---|---|
| 任务 | 实现 filter_by_tag + filter_by_keyword（--tag 精确 / --keyword 模糊） |
| 关联 Story | ST-EX-16 |
| file_ownership | skills/case-execution/scripts/case_runner.py#filter_by_tag, #filter_by_keyword |
| 依赖 | TASK-EX-FE02-01, TASK-EX-FE02-13（ST-EX-15） |
| 完成准则 | --tag 按 frontmatter tags 列精确过滤；--keyword 模糊匹配标题/描述 |
| 验证 | dry-run（TEST-PLAN SCN-EX-22） |

### TASK-EX-FE02-15: 环境解析层 + DUT 接口预配置（ST-EX-17，full-lld）

| 字段 | 值 |
|---|---|
| 任务 | 实现 build_env_topology + preconfigure_dut_interfaces（env_topology 契约 + ${ENV.*} 9 类解析集成 + DUT 接口预配置 + 用例后清理 + TREX_API_URL 环境文件优先） |
| 关联 Story | ST-EX-17 |
| file_ownership | skills/case-execution/scripts/case_runner.py#build_env_topology, #preconfigure_dut_interfaces |
| 依赖 | ST-EX-03（op_mapper resolve_env_refs）, TASK-EX-FE02-01（ST-EX-04） |
| 完成准则 | env_topology(port_mapping/nodes/links) 校验（DA-006/007）；${ENV.*} 9 类占位符解析（SM-EX-12）；向后兼容无 ${ENV.*} 原样透传（Gotcha #9）；YAML 1.1 整数陷阱规避端口名加引号（Gotcha #8）；DUT 接口预配置 fw_update_interface；清理顺序先 mutation ops 逆序再预配置接口逆序（Gotcha #11）；TREX_API_URL ${ENV.tg.url} 优先 devices.yaml fallback（ADR-05）；无直接 TG REST 调用（Gotcha #10） |
| 验证 | dry-run + review（TEST-PLAN SCN-EX-26/27/28/29） |

## 任务依赖图

```
ST-EX-03 (FE-EX-01, op_mapper resolve_env_refs + _build_exec_env)
   │
   ▼
TASK-EX-FE02-01 (ST-EX-04 骨架)
   ├─ TASK-EX-FE02-02 (安装验证)
   ├─ TASK-EX-FE02-03 (load_env_file)
   ├─ TASK-EX-FE02-04 (ST-EX-05 授权门)
   ├─ TASK-EX-FE02-05 (ST-EX-06 清理)
   ├─ TASK-EX-FE02-06 (ST-EX-07 fw_logout)
   ├─ TASK-EX-FE02-07 (ST-EX-08 ARP 预热)
   ├─ TASK-EX-FE02-08 (ST-EX-09 retry)
   ├─ TASK-EX-FE02-09 (ST-EX-10 四态) ── TASK-EX-FE02-10 (ST-EX-11 报告)
   ├─ TASK-EX-FE02-11 (ST-EX-12 extract_payload) ── TASK-EX-FE02-12 (ST-EX-14 verify_loss)
   ├─ TASK-EX-FE02-13 (ST-EX-15 结构化) ── TASK-EX-FE02-14 (ST-EX-16 标签)
   └─ TASK-EX-FE02-15 (ST-EX-17 环境解析层, 依赖 ST-EX-03 + ST-EX-04)
```

## 工作量汇总

| Story | Wave | 工作量 | 任务 |
|---|---|---|---|
| ST-EX-04 | 2 | L | TASK-EX-FE02-01/02/03 |
| ST-EX-05 | 2 | M | TASK-EX-FE02-04 |
| ST-EX-06 | 2 | M | TASK-EX-FE02-05 |
| ST-EX-07 | 2 | M | TASK-EX-FE02-06 |
| ST-EX-08 | 3 | M | TASK-EX-FE02-07 |
| ST-EX-09 | 3 | S | TASK-EX-FE02-08 |
| ST-EX-10 | 3 | S | TASK-EX-FE02-09 |
| ST-EX-11 | 3 | M | TASK-EX-FE02-10 |
| ST-EX-12 | 3 | S | TASK-EX-FE02-11 |
| ST-EX-14 | 4 | S | TASK-EX-FE02-12 |
| ST-EX-15 | 3 | S | TASK-EX-FE02-13 |
| ST-EX-16 | 3 | S | TASK-EX-FE02-14 |
| ST-EX-17 | 3 | M | TASK-EX-FE02-15 |

**总计**：13 Story / 15 任务 / 4 Wave，工作量 6S + 6M + 1L（与 HLD §19 对齐：ST-EX-04=L, 05/06/07/08/11/17=M, 09/10/12/14/15/16=S）。
