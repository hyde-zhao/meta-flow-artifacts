---
status: draft
version: "1.0"
confirmed: false
confirmed_by: ""
confirmed_at: ""
source_cr: "CR-030"
---

# CR-030 架构决策记录

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator | CP3 前 ADR 候选；未获人工确认。 |

| ADR ID | 决策问题 | 推荐决定 | 备选 | 理由 | 状态 |
|---|---|---|---|---|---|
| ADR-RA-01 | 分析与跟踪的 Skill 边界 | 分离 `reverse-analysis` 和 `improvement-tracker` | 单 Skill 或模板-only | 分离结论生成和行动状态，降低权限/状态耦合并利于独立验证 | pending CP3 |
| ADR-RA-02 | 改进输入的交接形态 | 不可变、带 approval/provenance 的文件化 Approved Improvement Input | 直接改下游文件；外部 API | 最小授权、单写和追溯明确；不需要 runtime | pending CP3 |
| ADR-RA-03 | 自动化等级 | 首版 Level 1–2，所有结论和分发人工确认 | Level 3 自动分发；模板-only | 先验证安全和准确性，避免建议被当作生产动作 | pending CP3 |
| ADR-RA-04 | 观察期与验证模式 | 30 天业务观察作为人工证据；自动化使用 fixture/dry-run 验证规则 | 仅自动时间模拟；仅人工审查 | 区分产品现实与 CI 可验证性，避免虚假 runtime 声明 | pending CP3 |
