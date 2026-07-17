---
change_id: "CR-030-REV-01"
parent_cr: "CR-030"
type: "requirements-quality-correction"
status: "active"
rollback_to: "requirement-clarification"
created_at: "2026-07-15T08:30:00+00:00"
trigger: "用户反馈：用户场景不是基于用户来写"
---

# CR-030-REV-01：用户中心场景修订影响分析

## 修订请求

原 `USE-CASES.md` 以 RA 处理步骤为场景标题，虽列出了 persona，但没有从用户的目标、决策、痛点、协作和可观察结果出发。用户要求改为基于用户的用户场景。

## 为什么不新建 CR

修订不增加逆向分析能力范围、外部权限或交付目标；它校正同一 CR-030 的产品发现表达和下游追溯。因此在活跃 CR 内作为 REV-01 处理，同时撤回受影响门禁。

## 五维度影响分析

| 维度 | 是否受影响 | 对象 | 处理 |
|---|---|---|---|
| 需求层 | 是 | `USE-CASES.md`、`REQUIREMENTS.md` | 将系统步骤改写为用户旅程，需求来源回链改为用户目标。 |
| 场景层 | 是 | `SCENARIOS.yaml`、`TEST-MATRIX.md` | 以用户旅程结果和失败体验重建场景映射。 |
| 计划层 | 是 | `STORY-MAP.md`、`MVP-SCOPE.md`、release slices/backlog | Story 以人物收益而非 Agent 内部动作表达。 |
| 设计层 | 是 | BLUEPRINT/HLD/ADR/CP3 | 原 HLD 的 UC trace 和 CP3 设计门失效；在新的 CP2 后重建/重审。 |
| 安全/权限 | 否 | CR-030 不授权范围 | deny-by-default 不变。 |

## 文档处理决策与旧基线映射

| 原对象 | 处理方式 | 新对象 | 旧基线保留方式 |
|---|---|---|---|
| UC-RA-01..06（流程中心） | 原文档更新 | UC-RA-01..06（用户中心） | 本文映射表 + `USE-CASES.md` v1.1 修订记录；旧文本语义可由 Git/CR 追溯。 |
| REQ-RA-001..013 | 原文档更新 | 同 ID，更新 user-goal/source | 不删除 ID，仅追加 revision。 |
| SCN-RA-01..07 | 原文档更新 | 同 ID，更新 Persona/user-journey source | 不删除 ID。 |
| ST-RA-01..04 | 原文档更新 | 同 ID，更新为用户 outcome | 不删除 ID。 |
| HLD/BLUEPRINT/ADR | 原文档更新 | 新 CP2 后重建追溯和审查 | 旧 CP3 标为 changes-requested，不视为当前证据。 |

## 旧/新场景映射

| 旧 UC（流程标题） | 新 UC（用户目标标题） | 映射说明 |
|---|---|---|
| UC-RA-01 创建 RA | UC-RA-01 测试经理决定是否启动系统性预防 | 从“创建记录”改为管理者的风险决策。 |
| UC-RA-02 证据不足阻断 | UC-RA-02 测试架构师建立可被信任的事实基础 | 从系统阻断改为架构师避免错误结论的目标。 |
| UC-RA-03 人工复核分析 | UC-RA-03 测试架构师解释问题为何发生且未被发现 | 从工具步骤改为技术 Owner 的决策结果。 |
| UC-RA-04 生成 CA/PA | UC-RA-04 下游改进 Owner 将教训转为测试/质量改进 | 从输出动作改为消费者价值。 |
| UC-RA-05 跟踪关闭 | UC-RA-05 测试经理确认改进真正降低复发风险 | 从状态机动作改为管理结果。 |
| UC-RA-06 拒绝越权 | UC-RA-06 安全/质量 Owner 防止分析越过权限边界 | 从拒绝动作改为风险控制目标。 |

## 门禁影响

- CP1：失效，需基于 user-centered scenarios 重跑。
- CP2：`changes_requested`，原 approve 只适用于旧场景表达，不能自动复用。
- CP3：`changes_requested`，必须等待新 CP2 后刷新 Traceability、HLD 和 Decision Brief。
- CP4–CP8：尚未开始，无需回滚。

## 回退与验收

- 回退目标：`requirement-clarification`。
- 新 CP2 的验收：每个 UC 明确“谁、想达成何种结果、为什么、触发、协作、失败体验与可观察成功”；Agent/Skill 内部步骤仅作为支撑流程，不得作为场景标题。
