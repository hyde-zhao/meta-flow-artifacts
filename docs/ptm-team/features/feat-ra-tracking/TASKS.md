---
status: draft
version: "1.0"
source_cr: "CR-030"
feature_id: "FEAT-RA-TRACKING"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-TRACKING — 任务清单

### ST-RA-04: 闭环跟踪与有效性决策

**TASK-TRK-01**: 在 `skills/improvement-tracker/SKILL.md` 追加跟踪侧定义
**TASK-TRK-02**: 实现 Action Item 状态机（not-started→in-progress→done/overdue）
**TASK-TRK-03**: 实现 Effectiveness Check 模型（planned/passed/failed/inconclusive）
**TASK-TRK-04**: 实现 Closure Decision 模型（四个关闭条件）
**TASK-TRK-05**: 实现关闭条件不可绕过逻辑
**TASK-TRK-06**: 创建 模板文件

### ST-RA-06.3-TRACK: 措施基线管理与刷新提示

**TASK-TRK-07**: 实现 MeasureBaseline 模型与管理
**TASK-TRK-08**: 实现措施刷新规则（保持/完成/需复核/失效/needs-baseline）
**TASK-TRK-09**: 实现不自动改变批准状态的门控
**TASK-TRK-10**: 创建 Fixture
