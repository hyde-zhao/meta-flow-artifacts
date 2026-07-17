---
status: draft
version: "1.0"
source_cr: "CR-030"
feature_id: "FEAT-RA-IMPROVEMENT"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-IMPROVEMENT — 任务清单

### ST-RA-03: 改进输入治理

**TASK-IMP-01**: 在 `skills/improvement-tracker/SKILL.md` 写入 CA/PA 侧定义
**TASK-IMP-02**: 实现 CA/PA Proposal 草案生成（依据 + 目标 + Owner + 验收）
**TASK-IMP-03**: 实现批准状态机（draft→approved/rejected）
**TASK-IMP-04**: 实现 Approved Improvement Input 生成（批准的 CA/PA → 不可变输入）
**TASK-IMP-05**: 实现消费者映射逻辑（target_agent → consumer_status）
**TASK-IMP-06**: 实现未批准门控（未确认 RA / 未批准 CA/PA → 拒绝生成）
**TASK-IMP-07**: 创建 CA/PA Proposal 和 Approved Input 模板
**TASK-IMP-08**: 创建 Fixture（已确认 RA、CA/PA 草案、批准/拒绝场景）
