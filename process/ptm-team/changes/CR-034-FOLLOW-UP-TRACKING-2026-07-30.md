---
source_cr: "CR-034"
status: "candidate-tracking"
created_at: "2026-07-30T16:40:00+08:00"
---

# CR-034 后续跟踪台账

本文件只记录候选后续工作，不创建新的 active CR，也不授权任何运行时操作。

| ID | 类型 | 候选事项 | 触发条件 | 状态 |
|---|---|---|---|---|
| CR-034-FU-01 | follow_up_tracking | test_install_mapping 期望更新（test_ptm_te_skills_consistent 4->5 skills 含 case-execution） | 测试维护需求 | candidate |
| CR-033-FU-01 | runtime_authorization | T-01 runtime 端到端验证（24 用例 --execute 真机执行） | 用户单独授权运行环境 + 设备 + 凭据 | candidate |
| CR-033-FU-02 | follow_up_tracking | PF-04 tg_config_interface 聚合占位符 `${ENV.tg.ports[port1,port2]}` | 需 TG 聚合端口场景的用例 | candidate |
| CR-033-FU-03 | follow_up_tracking | case_runner lint 子命令（用例规范校验工具） | 用例格式批量校验需求 | candidate |

## 转换规则

任一候选仅在用户确认范围、授权和交付目标后转为独立 CR；在此之前不得执行网络请求、写入或生产操作。

## 详细条目

### CR-034-FU-01 test_install_mapping 期望更新
- **背景**：CR-034 CP7 发现 `tests/test_install_mapping.py::test_ptm_te_skills_consistent` pre-existing 失败。CR-033 给 `PTM_TE_SKILLS` 加入 `case-execution`（第 5 skill），但测试期望仍为 4 skills（`["device-management", "device-connection", "policy-route-execution", "trex-traffic"]`），未更新。
- **真相源**：`script/ptm_team/install.py` PTM_TE_SKILLS（5 skills，含 case-execution）；`tests/test_install_mapping.py:126` 期望（4 skills）。
- **范围**：测试期望更新 4->5（加 `case-execution`），非 CR-034 范围（CR-034 未碰 PTM_TE_SKILLS）。
- **关闭条件**：测试期望更新 + `uvx pytest tests/test_install_mapping.py` PASS。

### CR-033-FU-01 T-01 runtime 端到端验证
- **背景**：CR-033 validation_mode=static-only + dry-run-only，runtime 端到端未验证（CR-034 继承，仍 static-only + dry-run-only）。
- **范围**：24 用例 --execute 真机执行（DUT + TG）。
- **前置**：设备可达 + 凭据环境变量 + `--execute --authorized` 运行授权。
- **关闭条件**：24 用例 runtime PASS 或 PASS_WITH_RISK。

### CR-033-FU-02 PF-04 tg_config_interface 聚合占位符
- **背景**：tg_config_interface 需聚合端口（`${ENV.tg.ports[port1,port2]}`），当前 resolve_env_refs 聚合分支未覆盖。
- **范围**：op_mapper resolve_env_refs 聚合占位符解析（需改实现）。
- **关闭条件**：聚合占位符解析正确 + dry-run 验证。

### CR-033-FU-03 case_runner lint 子命令
- **背景**：case_runner 仅有 run/validate，无用例规范校验工具。
- **范围**：新增 lint 子命令，复用 parse_case_file/parse_frontmatter 批量校验。
- **关闭条件**：lint 子命令实现 + 当前 cases 目录校验通过。
