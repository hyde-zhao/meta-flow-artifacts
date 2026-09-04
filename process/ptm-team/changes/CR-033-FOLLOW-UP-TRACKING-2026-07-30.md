---
source_cr: "CR-033"
status: "candidate-tracking"
created_at: "2026-07-30T14:30:00+08:00"
---

# CR-033 后续跟踪台账

本文件只记录候选后续工作，不创建新的 active CR，也不授权任何运行时操作。

| ID | 类型 | 候选事项 | 触发条件 | 状态 |
|---|---|---|---|---|
| CR-033-FU-01 | runtime_authorization | T-01 runtime 端到端验证（24 用例 --execute 真机执行） | 用户单独授权运行环境 + 设备 + 凭据 | candidate |
| CR-033-FU-02 | follow_up_tracking | PF-04 tg_config_interface 聚合占位符 `${ENV.tg.ports[port1,port2]}` | 需 TG 聚合端口场景的用例 | candidate |
| CR-033-FU-03 | follow_up_tracking | case_runner lint 子命令（用例规范校验工具，C 改进） | 用例格式批量校验需求 | candidate |
| CR-033-FU-04 | follow_up_tracking | devices.yaml schema 脱节（device_groups vs 顶层 firewall/tg） | 规范统一需求 | candidate |

## 转换规则

任一候选仅在用户确认范围、授权和交付目标后转为独立 CR；在此之前不得执行网络请求、写入或生产操作。

## 详细条目

### CR-033-FU-01 T-01 runtime 端到端验证
- **背景**：CR-033 validation_mode=static-only + dry-run-only，runtime 端到端未验证
- **范围**：24 用例 --execute 真机执行（DUT + TG）
- **前置**：设备可达 + 凭据环境变量 + `--execute --authorized` 运行授权
- **关闭风险**：R-CP7-01 runtime 业务正确性未验证；R-CP7-04 性能 24 用例 <5min
- **关闭条件**：24 用例 runtime PASS 或 PASS_WITH_RISK

### CR-033-FU-02 PF-04 tg_config_interface 聚合占位符
- **背景**：tg_config_interface 需聚合端口（`${ENV.tg.ports[port1,port2]}`），当前 resolve_env_refs 聚合分支未覆盖
- **范围**：op_mapper resolve_env_refs 聚合占位符解析
- **触发**：需 TG 聚合端口场景的用例
- **关闭条件**：聚合占位符解析正确 + dry-run 验证

### CR-033-FU-03 case_runner lint 子命令
- **背景**：case_runner 仅有 run/validate，无用例规范校验工具（CP8 交付审查 C 改进）
- **范围**：新增 lint 子命令，复用 parse_case_file/parse_frontmatter 批量校验（frontmatter 16 列 / case_steps 顶层 list / 顶层 op_id+args / target 小写 / `${ENV.*}` / warming_up+post_op / max_loss）
- **触发**：用例格式批量校验需求（避免临时写 /tmp 脚本）
- **关闭条件**：lint 子命令实现 + 当前 cases 目录校验通过

### CR-033-FU-04 devices.yaml schema 脱节
- **背景**：devices.yaml.example 用 device_groups 嵌套结构，case_runner.resolve_addresses 期望顶层 firewall.host/tg.api_server 结构，二者脱节（dry-run 验证时发现）
- **范围**：统一 devices.yaml schema（devices.yaml.example vs resolve_addresses 实现）
- **触发**：规范统一需求
- **关闭条件**：devices.yaml.example 与 resolve_addresses 实现一致 + 文档同步
