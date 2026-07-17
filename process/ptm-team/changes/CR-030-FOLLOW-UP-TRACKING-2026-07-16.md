---
source_cr: "CR-030"
status: "candidate-tracking"
created_at: "2026-07-16T17:15:00+08:00"
---

# CR-030 后续跟踪台账

本文件只记录候选后续工作，不创建新的 active CR，也不授权任何运行时操作。

| ID | 类型 | 候选事项 | 触发条件 | 状态 |
|---|---|---|---|---|
| CR-030-FU-01 | follow_up_tracking | 数据存储容量阈值与受控清除策略 | 首批受控运行出现容量规划需求 | candidate |
| CR-030-FU-02 | follow_up_tracking | pri/severity 真实 ITR fixture 扩展 | 映射异常或质量门控显示覆盖不足 | candidate |
| CR-030-FU-03 | runtime_authorization | S1/S2、SQLite 权限、真实 ITR schema 和 reviewer 交互 smoke test | 用户单独授权受控运行环境 | candidate |

## 转换规则

任一候选仅在用户确认范围、授权和交付目标后转为独立 CR；在此之前不得执行网络请求、写入或生产操作。
