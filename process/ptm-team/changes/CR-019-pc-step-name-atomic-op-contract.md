---
change_id: CR-019-pc-step-name-atomic-op-contract
title: 物理用例步骤名称与原子操作双层表达契约
status: closed
workflow_mode: fast-lane
created: 2026-06-11
approved: 2026-06-11
closed: 2026-06-11
impact_level: low
rollback_to: CR-013-ptm-tde-ppdcs-phase delivered baseline
related_active_change: CR-018-ptm-tde-workflow-compliance-and-workspace-isolation
---

# CR-019 - 物理用例步骤名称与原子操作双层表达契约

## 变更背景

用户确认：最终物理用例输出时，测试步骤不能只有原子操作，还必须包含面向测试人员的人类可读步骤名称。例如：

- 步骤名称：配置策略路由的匹配地址对象 xx
- 原子操作：`config-policy-route src-addr=xx`

现有 PPDCS/PC 产出规则主要要求 16 列物理用例表和 `action_source_refs`，但没有强制每一步同时表达“步骤名称 + 原子操作”。这会导致交付用例可读性不足，测试人员只能从原子操作猜测测试意图。

## 影响分析

| 维度 | 结论 |
|---|---|
| 需求层 | 增强既有 PC 输出契约，不新增功能范围。 |
| 场景层 | 不改变场景发现、LC、TD 生成逻辑，只要求 PC 步骤物化时保留场景动作意图。 |
| 计划层 | 影响 PPDCS 五方法 Skill、PPDCS 协调、覆盖验证和交付渲染规则。 |
| 安全层 | 不涉及权限、安全边界、外部接口、凭据或运行授权。 |
| 交付层 | 最终测试用例 16 列表结构不变，增强 `测试步骤*` 单元格内容。 |

## 文档处理决策

| 对象 | 处理方式 | 说明 |
|---|---|---|
| `.agents/skills/design-ppdcs-analyzer/SKILL.md` | 原文档更新 | 增加 `case_steps` 统一契约。 |
| `.agents/skills/process-design/SKILL.md` | 原文档更新 | PC 骨架增加步骤结构化字段。 |
| `.agents/skills/state-design/SKILL.md` | 原文档更新 | 与 process-design 保持一致。 |
| `.agents/skills/parameter-design/SKILL.md` | 原文档更新 | 16 列示例中的 `测试步骤*` 使用“步骤名称 + 原子操作”。 |
| `.agents/skills/data-design/SKILL.md` | 原文档更新 | 16 列示例中的 `测试步骤*` 使用“步骤名称 + 原子操作”。 |
| `.agents/skills/combination-design/SKILL.md` | 原文档更新 | 16 列示例中的 `测试步骤*` 使用“步骤名称 + 原子操作”。 |
| `.agents/skills/coverage-verifier/SKILL.md` | 原文档更新 | 覆盖验证要求 PC 保留 `case_steps`。 |
| `.agents/skills/deliverable-renderer/SKILL.md` | 原文档更新 | 最终交付渲染规范化 `测试步骤*` 表达。 |

## 决策

采用双层步骤表达：

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置策略路由的匹配源地址对象 OBJ_SRC_WEB
    target: DUT
    atomic_op:
      op_id: config-policy-route
      args:
        src-addr: OBJ_SRC_WEB
    expected_result: 策略路由规则成功引用源地址对象 OBJ_SRC_WEB
    trace_refs:
      - TP-001
      - TD-ADDR-001
```

最终 16 列表的 `测试步骤*` 列按以下规则渲染：

```text
1. 配置策略路由的匹配源地址对象 OBJ_SRC_WEB
   执行对象：DUT
   原子操作：config-policy-route src-addr=OBJ_SRC_WEB
```

Markdown 表格中使用 `<br>` 换行，不新增 16 列之外的新列。

## 回滚方案

如该格式不再适用，回滚到 CR-013 的 PC 16 列输出基线：移除 `case_steps` 强制要求，但保留原有 `action_source_refs` 和 `测试步骤*` 文本列。

## 验证结果

- 已更新 PPDCS 设计、协调、覆盖和交付渲染 Skill 契约。
- 已检查受影响 Skill 文档中包含 `case_steps`、`step_name`、`atomic_op` 和 `原子操作：` 规则。
