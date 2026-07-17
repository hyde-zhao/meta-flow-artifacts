---
status: draft
version: "1.2"
---

# ptm-tse 逆向分析 — 用户价值 Backlog

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator | 初始延后项。 |
| 1.1 | 2026-07-15 | host-orchestrator | 补充每项的真实受益用户，保持在本 CR 范围外。 |
| 1.2 | 2026-07-15 | host-orchestrator | 将非 ITR 外部来源与更高自动化级别保留为独立后续项。 |

| Item ID | 类型 | 受益用户 | 来源 | 内容 | 当前状态 | 延后原因 | 重启条件 |
|---|---|---|---|---|---|---|---|
| BL-RA-01 | enhancement | 测试经理、质量负责人 | DEF-RA-01 | 到期提醒与复发告警，减少人工追踪观察窗 | candidate | 需要通知和运行授权 | MVP 稳定且批准 runtime CR |
| BL-RA-02 | enhancement | 测试设计/执行/自动化负责人 | DEF-RA-02 | 下游 Agent 自动消费已批准改进输入 | candidate | 首版需先验证低耦合契约 | 至少 5 个报告完成闭环 |
| BL-RA-03 | spike_candidate | 测试架构师、测试设计负责人 | DEF-RA-03 | 内部问题阶段前移分析模型 | candidate | 与现网流出模型不同 | 现网流程完成 2–3 迭代 |
| BL-RA-04 | experiment | 质量负责人、测试经理 | SCN-RA-05 | 用真实样本校准指标阈值和 30 天观察窗 | candidate | 需要真实历史样本 | 首批 3 份 RA 报告可用 |
| BL-RA-05 | enhancement | 测试架构师、质量负责人 | UC-RA-07 | 接入 ITR 以外的工单、日志、TAC 或知识库来源 | candidate | 每个来源都有独立认证、数据分类、保留与运行风险 | ITR 单一来源稳定且独立 CR 获批准 |
| BL-RA-06 | enhancement | 测试设计/执行/自动化负责人 | UC-RA-04 | 根据已批准改进候选自动创建下游任务 | candidate | 会引入跨 Agent 写入、权限和回滚风险 | 至少 5 个报告人工闭环并通过独立授权 |

## 台账边界

本 Backlog 是产品规划输入，不等同 CP8 follow-up tracking。只有后续用户决定启动某项时，才新建正式 CR。
