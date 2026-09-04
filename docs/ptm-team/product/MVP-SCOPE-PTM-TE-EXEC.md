---
status: confirmed
version: "1.0"
---

# ptm-te 执行引擎 - MVP 范围

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-pm | CR-033 初始 MVP 范围 |
| 1.1 | 2026-07-28 | meta-pm | CP2 范围修改：In Scope 补充用例结构化整改（目录+命名+frontmatter+tags）；--tag/--keyword 替代 --case-files |
| 1.2 | 2026-07-28 | meta-pm | CP3 评审范围扩展（环境文件驱动）：In Scope 补充环境文件驱动（用例与环境解耦是核心价值，非 Deferred） |

## 本轮目标

| 目标 ID | 用户 outcome | 度量方式 | 目标值 | 来源 |
|---|---|---|---|---|
| GOAL-EX-01 | 测试平台开发者能建模 TG 设备并固化规则 | 6 组合 + ≥4 条规则 + 重装不丢失 | 6 组合 + ≥4 规则 + 0 丢失 | UC-EX-01,08 |
| GOAL-EX-02 | 测试执行工程师能用一个命令跑用例并得到四态结果 | 3 入口 + 四态 + 硬编码 0 | 3 入口 + 4 态 + 0 硬编码 | UC-EX-02..06 |
| GOAL-EX-03 | 安全合规负责人能阻止未授权写操作 | dry-run 默认 + --execute 授权 + 审计 | 0 未授权写操作 | UC-EX-04,05 |

## In Scope（MVP = P0）

| 范围项 | 需求 | 交付物 | 验收 |
|--------|------|--------|------|
| TG 设备建模 | R-F-001,002,003 | devices.yaml tg 块 + SKILL.md + device-reference.md 6 组合 | 6 组合全覆盖 |
| case_runner 三入口 | R-F-004 | case_runner.py run --cases-dir/--case-files/--case-file | 3 入口零代码新增用例 |
| devices.yaml 取址 | R-F-005 | DUT/TG 地址从 devices.yaml 读取 | 0 硬编码 IP |
| dry-run 默认门 | R-F-006 | 默认 dry_run=True | 不连接设备 |
| --execute 授权门 | R-F-007 | --execute --authorized 才执行写操作 | 未授权返回 EXEC_FAILED |
| 逆序清理 | R-F-015 | 每用例结束逆序清理 mutation ops | 清理结果记入 result.json |
| fw_logout op | R-F-016,017 | op_mapper 新增 fw_logout + case_runner cleanup 登出 | session 文件清理 |
| TREX_API_URL 注入 | R-F-018 | _build_exec_env 对 tg_* op 注入 TREX_API_URL | tg op 连接 devices.yaml 配置的 api_server |
| install.py 规则块 | R-F-019 | ≥4 条新规则（TG路由/max_loss/ARP预热/session） | 安装后 CLAUDE.md 含规则块 |
| warming_up/post_op 引擎强制 | R-F-008 | case_runner 识别 warming_up:true 自动补 post_op | 引擎强制不依赖 md |
| 用例目录结构约定 | R-F-022 | cases/三级/四级/五级/<编号>-<名称>.md | 24 用例迁移到新结构 |
| 用例命名约定 | R-F-023 | <用例编号>-<用例名称>.md | 编号保留组网前缀 |
| frontmatter 16 列 | R-F-024 | 16 列字段定义 | 必填/可选见 DQ-05 |
| tags/关键词列 | R-F-025 | tags 精确过滤 + 关键词模糊匹配 | --tag/--keyword 替代 --case-files |
| --tag/--keyword 执行 | R-F-026 | case_runner --tag 精确过滤 + --keyword 模糊匹配 | 可组合使用 |
| 环境文件驱动（用例与环境解耦） | R-F-027 | 用例 ${ENV.*} 占位符 + resolve_env_refs 自动解析 | 换环境只换 --env-file，不改用例 |
| DUT 接口自动预配置 | R-F-028 | login 后自动 fw_update_interface 配 IP，用例后清理 | 用例不写预配置步骤 |
| 设备 URL 自动解析 | R-F-029 | TREX_API_URL 从环境文件注入，devices.yaml fallback | 无硬编码 IP |

## Out of Scope

| 排除项 | 原因 |
|--------|------|
| 改 ptm-atomic CLI 本体 | 只改消费侧 |
| 引入 pytest/robot | 借鉴理念不引入框架 |
| pydantic devices.yaml | 保持轻量 |
| HTML 报告 | result.json + report.md 足够 |
| 改 traffic-skill / ngfw-install | 不在范围 |
| TG 系统快照采集 | TG 纳入清单不纳入快照 |
| 凭据读取 | NO_CREDENTIAL_READ |
| 真实设备自动授权 | NO_PRODUCTION_WRITE |

## Deferred（后续迭代）

| Deferred 项 | 延后原因 | 重启条件 | 来源 |
|-------------|---------|---------|------|
| HTML 报告生成 | result.json + report.md 足够 | 用户明确要求 | DEF-EX-01 |
| --validate 中间授权模式 | dry-run + --execute 二级已足够 | 需要更细粒度验证 | DEF-EX-02 |
| 24 用例 md 迁入 ptm-team | 留 workspace 更合适 | 需要版本控制 | DEF-EX-03 |
| expected_result 语义推断 | 显式标记更可审计 | NLP 成熟后 | DEF-EX-04 |
| ptm-atomic 本体扩展 | 只改消费侧 | ptm-atomic 升级 CR | DEF-EX-05 |
| pydantic schema 校验 | 保持轻量 | 设备模型复杂度增加 | DEF-EX-06 |
| retry 轮询（P1） | P1 优先级，MVP 后迭代 | M2 完成后 | R-F-009 |
| known_issue 四态（P1） | P1 优先级 | M2 完成后 | R-F-010,011 |
| 失败诊断 + 报告（P1） | P1 优先级 | M2 完成后 | R-F-012,013 |
| extract_payload（P1） | P1 优先级 | M2 完成后 | R-F-014 |
| 24 用例 ARP 整改（P2） | P2 优先级 | M3 完成后 | R-F-021 |
| verify_loss 消费侧（P3） | P3 优先级 | M3 完成后 | R-F-020 |
