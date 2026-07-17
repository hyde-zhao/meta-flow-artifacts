---
release: "CR-030 ptm-tse 现网问题逆向分析能力 v1.0"
feedback_plan_version: "v1.0"
release_artifact_profile: "compact"
---

# ptm-tse CR-030 发布后反馈计划

> 版本：v1.0 · 更新：2026-07-16

---

## 反馈目标

CR-030 是 ptm-tse 的首次逆向分析能力发布，重点收集以下方面的反馈：

### 1. 功能完整性

- ITR 摄取是否覆盖了实际的 ITR 响应格式
- 六维分析维度是否满足实际分析需求
- 改进措施模板是否适用于下游 Agent 消费

### 2. 安全边界

- allowlist 策略是否灵活但不过度宽
- deny-by-default 是否有误拒绝合法操作
- 权限边界是否有遗漏

### 3. 可用性

- Gotchas 缺失是否导致使用者踩坑（预计在补充 Gotchas 前收集）
- S1/S2 管线路由是否直观
- 状态机操作是否符合 reviewer 预期

### 4. 性能

- 大规模批次（>1000 条）的摄取性能
- 变更检测在大量 ticket 时的处理速度

## 反馈渠道

| 渠道 | 用途 |
|------|------|
| `docs/release/FEEDBACK.md`（本文件） | 结构化反馈记录 |
| CR 台账 | 转化为正式改进需求 |
| 直接对话 | 紧急问题即时反馈 |

## 已知待观察项

| 观察项 | 来源 | 优先级 | 重访条件 |
|--------|------|--------|---------|
| 4 Story 缺 CP6 检查点 | CP7 验证 | HIGH | 首次运行时验证后补写或豁免 |
| 2 Skill 缺 Gotchas | CP7 验证 | MEDIUM | 首次使用后收集 2 周，汇总补充 |
| 三线阈值 3 是否恰当 | HLD 设计 | MEDIUM | 首次 5 个分析任务后评估 |
| 观察窗 30 天是否足够 | HLD 设计 | LOW | 首个关闭决策后评估 |
| 显著变化阈值 20% 是否合理 | S2 设计 | LOW | 首个差异报告后评估 |
| HTTP 超时 30s 是否够用 | 摄取设计 | LOW | 首次大批次拉取后评估 |

## 结构化反馈模板

```yaml
feedback:
  id: "FB-CR030-YYYYMMDD-NNN"
  date: ""
  reporter: ""
  category: "功能 | 安全 | 可用性 | 性能 | 文档"
  severity: "HIGH | MEDIUM | LOW"
  description: ""
  steps_to_reproduce: ""
  expected: ""
  actual: ""
  workaround: ""
  suggested_fix: ""
```
