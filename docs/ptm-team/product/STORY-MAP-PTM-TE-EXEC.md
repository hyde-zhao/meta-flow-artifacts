---
status: confirmed
version: "1.0"
source_use_cases: ["docs/product/USE-CASES-PTM-TE-EXEC.md"]
source_scenarios: ["docs/product/SCENARIOS-PTM-TE-EXEC.yaml"]
source_requirements: ["docs/product/REQUIREMENTS-PTM-TE-EXEC.md"]
---

# ptm-te 执行引擎 - 用户故事地图

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 来源 |
|---|---|---|---|---|
| 1.0 | 2026-07-28 | meta-pm | CR-033 初始 Story Map | USE-CASES-PTM-TE-EXEC v1.0 |
| 1.1 | 2026-07-28 | meta-pm | CP2 范围修改：ST-EX-13 扩大为全量整改（目录迁移+重命名+frontmatter+tags+ARP）；新增 ST-EX-15 用例结构化约定 + ST-EX-16 标签/关键字执行 | USE-CASES-PTM-TE-EXEC v1.1 |
| 1.2 | 2026-07-28 | meta-pm | CP3 评审范围扩展（环境文件驱动）：新增 ST-EX-17 环境解析层 + DUT 接口预配置（R-F-027..029 / SCN-EX-26..29） | USE-CASES-PTM-TE-EXEC v1.2 |

## 用户活动

| Activity ID | 用户活动 | 目标用户 | 用户要达成的结果 | 来源 |
|---|---|---|---|---|
| ACT-EX-01 | 管理 TG 设备清单 | 测试平台开发者 | devices.yaml 新增 TG 设备，6 组合全覆盖 | UC-EX-01 |
| ACT-EX-02 | 执行用例并收集结果 | 测试执行工程师 | 一个命令跑全部或部分用例，得到四态结果 | UC-EX-02,03,04,05 |
| ACT-EX-03 | 诊断失败用例 | 测试执行工程师、测试经理 | 区分 DUT 行为差异和脚本 bug，得到失败诊断 | UC-EX-06 |
| ACT-EX-04 | 保障 ARP 预热一致性 | 测试执行工程师 | warming_up step 自动清理，重装不回退 | UC-EX-07 |
| ACT-EX-05 | 固化规则与重装一致性 | 测试平台开发者 | install.py 安装后规则和 skill 不丢失 | UC-EX-08 |
| ACT-EX-06 | 清理 DUT 会话 | 测试执行工程师 | 用例结束后登出 session，不残留 | UC-EX-09 |

## 用户任务与 Story 候选

| Task ID | 用户任务 | Activity | Story 候选 | 优先级 | 来源需求 |
|---|---|---|---|---|---|
| TASK-EX-01 | devices.yaml tg 块定义 + 6 组合对照 | ACT-EX-01 | ST-EX-01: TG 设备建模 | P0 | R-F-001,002,003 |
| TASK-EX-02 | install.py 规则块 ≥4 条 | ACT-EX-05 | ST-EX-02: 规则固化 | P0 | R-F-019 |
| TASK-EX-03 | _build_exec_env 注入 TREX_API_URL | ACT-EX-05 | ST-EX-03: TG 路由参数化 | P0 | R-F-018 |
| TASK-EX-04 | case_runner 三入口 + devices.yaml 取址 | ACT-EX-02 | ST-EX-04: case_runner 核心 | P0 | R-F-004,005 |
| TASK-EX-05 | dry-run 默认门 + --execute 授权门 | ACT-EX-02 | ST-EX-05: dry-run/授权门 | P0 | R-F-006,007 |
| TASK-EX-06 | 逆序清理 mutation ops | ACT-EX-02 | ST-EX-06: 用例清理 | P0 | R-F-015 |
| TASK-EX-07 | fw_logout op + cleanup 登出 | ACT-EX-06 | ST-EX-07: fw_logout 会话清理 | P0 | R-F-016,017 |
| TASK-EX-08 | warming_up/post_op 引擎强制 | ACT-EX-04 | ST-EX-08: ARP 预热引擎 | P1 | R-F-008 |
| TASK-EX-09 | retry 轮询 | ACT-EX-02 | ST-EX-09: retry 轮询 | P1 | R-F-009 |
| TASK-EX-10 | known_issue 四态分级 | ACT-EX-03 | ST-EX-10: 四态分级 | P1 | R-F-010,011 |
| TASK-EX-11 | 失败自动诊断 + 结构化报告 | ACT-EX-03 | ST-EX-11: 诊断与报告 | P1 | R-F-012,013 |
| TASK-EX-12 | extract_payload 统一解析 | ACT-EX-03 | ST-EX-12: extract_payload | P1 | R-F-014 |
| TASK-EX-13 | 24 用例全量整改（目录迁移+重命名+frontmatter+tags+ARP 预热） | ACT-EX-04 | ST-EX-13: 24 用例全量整改 | P2 | R-F-021,022,023,024,025 |
| TASK-EX-14 | verify_loss 消费侧提取 tx/rx | ACT-EX-03 | ST-EX-14: verify_loss 消费侧 | P3 | R-F-020 |
| TASK-EX-15 | 用例目录结构 + 命名约定 + frontmatter 16 列 | ACT-EX-04 | ST-EX-15: 用例结构化约定 | P1 | R-F-022,023,024,025 |
| TASK-EX-16 | --tag 精确过滤 + --keyword 模糊匹配 | ACT-EX-02 | ST-EX-16: 标签/关键字执行 | P1 | R-F-026 |
| TASK-EX-17 | 环境文件驱动（resolve_env_refs + DUT 接口预配置 + TREX_API_URL 注入） | ACT-EX-02 | ST-EX-17: 环境解析层 | P1 | R-F-027,028,029 |

## Story 与 12 条改进映射

| 改进 # | 改进描述 | Story 候选 | 优先级 |
|--------|---------|-----------|--------|
| #1 | TG 路由参数化（TREX_API_URL 注入） | ST-EX-03 | P0 |
| #2 | 规则固化（install.py 规则块） | ST-EX-02 | P0 |
| #3 | 24 用例 ARP 预热整改 | ST-EX-13 | P2 |
| #4 | fw_logout op + cleanup 登出 | ST-EX-07 | P0 |
| #5 | 用例文件驱动（三入口） | ST-EX-04 | P0/P1 |
| #6 | envelope 统一解析（extract_payload） | ST-EX-12 | P1 |
| #7 | 失败自动诊断 | ST-EX-11 | P1 |
| #8 | TG dry-run 真实路由 | ST-EX-05 | P3（dry-run 已 P0，真实路由 P3） |
| #9 | verify_loss 消费侧提取 tx/rx | ST-EX-14 | P3 |
| #10 | devices.yaml 取址 | ST-EX-04 | P0/P1 |
| #11 | 结构化报告 | ST-EX-11 | P1 |
| #12 | known_issue 四态 | ST-EX-10 | P1 |
