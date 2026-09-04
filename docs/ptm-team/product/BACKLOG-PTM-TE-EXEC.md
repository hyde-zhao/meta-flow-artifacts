---
status: confirmed
version: "1.0"
---

# ptm-te 执行引擎 - 用户价值 Backlog

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-pm | CR-033 初始 Backlog |

| Item ID | 类型 | 受益用户 | 来源 | 内容 | 当前状态 | 延后原因 | 重启条件 |
|---|---|---|---|---|---|---|---|
| BL-EX-01 | enhancement | 测试经理、测试执行工程师 | DEF-EX-01 | HTML 报告生成（含图表和交互式筛选） | candidate | result.json + report.md 足够 | 用户明确要求或 CP8 后 |
| BL-EX-02 | enhancement | 测试执行工程师 | DEF-EX-02 | --validate 中间授权模式（只读 op 实际执行，写 op 跳过） | candidate | dry-run + --execute 二级已足够 | 需要更细粒度验证时 |
| BL-EX-03 | enhancement | 测试平台开发者 | DEF-EX-03 | 24 用例 md 迁入 ptm-team 统一管理（版本控制） | candidate | 24 用例是运行时数据，留 workspace 更合适 | 需要版本控制用例 md 时 |
| BL-EX-04 | experiment | 测试平台开发者 | DEF-EX-04 | expected_result 文本语义自动推断 KNOWN_FAIL（NLP 辅助） | candidate | 文本推断不可靠，显式标记更可审计 | NLP 能力成熟后 |
| BL-EX-05 | enhancement | 测试平台开发者 | DEF-EX-05 | ptm-atomic CLI 本体扩展（fw_delete_object 暴露等） | candidate | 只改消费侧，不改 ptm-atomic 本体 | ptm-atomic 升级 CR |
| BL-EX-06 | enhancement | 测试平台开发者 | DEF-EX-06 | pydantic devices.yaml schema 校验 | candidate | 不引入 pydantic，保持轻量 | 设备模型复杂度显著增加时 |
| BL-EX-07 | enhancement | 测试执行工程师 | SCN-EX-13,14 | STATE_INVALID/ConnectTimeout 故障注入测试 | candidate | 需要真实故障环境或 mock 框架 | mock 框架就绪或 runtime 环境可用 |
| BL-EX-08 | enhancement | 测试经理 | CR-033 #8 | TG dry-run 真实路由验证（dry-run 模式下 tg op 实际连接 trex-api 校验路由） | candidate | P3 优先级，dry-run 已校验命令构建 | P3 迭代时 |

## 台账边界

本 Backlog 是产品规划输入，不等同 CP8 follow-up tracking。只有后续用户决定启动某项时，才新建正式 CR。
