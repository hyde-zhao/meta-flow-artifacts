---
status: draft
version: "1.0"
source_cr: "CR-030"
---

# ptm-tse 逆向分析依赖地图

## 依赖关系

| From | To | 依赖类型 | 允许方向 | 原因 | 验证 / 监控 |
|---|---|---|---|---|---|
| FEAT-RA-ANALYSIS | 脱敏事件摘要 | read | allowed | 分析只消费用户/上游提供的摘要和索引 | input fixture |
| FEAT-RA-IMPROVEMENT | FEAT-RA-ANALYSIS | read | allowed | CA/PA 只能基于人工确认分析 | approval-state fixture |
| FEAT-RA-TRACKING | FEAT-RA-IMPROVEMENT | read | allowed | 跟踪只消费已批准改进输入 | provenance fixture |
| ptm-tm | ptm-tse | file / handoff | allowed | 提供合格事件触发与协调信息 | contract review |
| ptm-tse | ptm-tde/te/tae/qa | file / handoff | allowed, approval-gated | 只输出已批准改进输入 | contract fixture |
| ptm-qa | FEAT-RA-TRACKING | read | allowed | 消费聚合度量，不拥有 RA 行动状态 | metric review |

## 禁止依赖

| Forbidden ID | From | To | 禁止原因 | 替代路径 | 违反风险 |
|---|---|---|---|---|---|
| FD-RA-01 | 任一 RA Feature | 设备/日志/TAC/工单外部系统 | 本 CR 未授权外部读取或凭据 | 人工提供脱敏摘要/索引 | 数据泄露、越权运行 |
| FD-RA-02 | FEAT-RA-ANALYSIS | ptm-tde/te/tae/qa 交付文件 | 分析与下游实现必须解耦 | 生成 Approved Improvement Input | 覆盖其他 Agent owner |
| FD-RA-03 | FEAT-RA-TRACKING | RA 根因字段 | 跟踪不能修改已确认分析 | 新建 clarification / re-analysis 记录 | 审计链失真 |
| FD-RA-04 | 下游消费者 | 未批准 CA/PA | 未确认建议不可执行 | 只读已批准输入 | 误修复、不可追溯 |

## 循环风险

| Cycle ID | 涉及对象 | 风险 | 当前处理 |
|---|---|---|---|
| CYCLE-RA-01 | ANALYSIS → IMPROVEMENT → TRACKING → ANALYSIS | 跟踪失败可能需要重新分析 | 通过新 revision/clarification 记录回到 ANALYSIS，不回写既有 confirmed 事实 | eliminated |
| CYCLE-RA-02 | ptm-tse ↔ 下游 Agent | 双向改文件会产生所有权冲突 | 单向 Approved Improvement Input；下游结果通过独立 evidence ref 回链 | eliminated |
