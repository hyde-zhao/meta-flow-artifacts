---
discussion_id: "CP2-CR173-SCENARIO-DISCUSSION"
cr_id: "CR-173"
stage: "requirement-clarification"
status: "completed-for-cp2-submission"
owner: "meta-pm"
updated_at: "2026-07-16T12:05:00+08:00"
---

# CR173 CP2 场景讨论日志

## 讨论来源

- CR172 CP2 已批准 PATH-B 与 `separate_methodology_cr`。
- 用户明确要求完成整改后自动推进到下一个人工门禁。
- CR173 继承的是“离线 estimator 前置”的用户选择，不继承任何真实数据、策略身份或 runtime 授权。

## Scenario Gray Areas

| ID | 问题 | 推荐 | 备选 | 状态 |
|---|---|---|---|---|
| SGA-CR173-001 | CP2 是否冻结具体算法？ | 只冻结 estimand/可测行为；算法留 CP3。 | 先转方法 Spike。 | OPEN-CP2 |
| SGA-CR173-002 | 失败能否回退 raw count？ | 不允许；typed_unavailable/blocked。 | 保持全量 unavailable。 | OPEN-CP2 |
| SGA-CR173-003 | 是否要求具体策略身份？ | 不要求、不推断，strategy-agnostic。 | 仅 synthetic case label。 | RESOLVED-BY-PATH-B |
| SGA-CR173-004 | 完成后是否自动恢复 CR172？ | 不自动；三项恢复前置后重开 CR172 CP2。 | 等用户另行发起。 | OPEN-CP2-CONFIRMATION |

## 用户可见确认交互

| Question ID | 问题 | 选项 | 推荐 | 用户回答 | 复述确认 | 影响面 | 状态 |
|---|---|---|---|---|---|---|---|
| SGQ-CR173-001 | 五字段不可冻结且方法学 owner 独立时，是否先完成离线 estimator？ | A. 独立 CR；B. 合并 CR172；C. 暂停。 | A | 用户批准 CR172 PATH-B，并要求继续到下一人工门禁。 | CR173 是 strategy-agnostic offline predecessor，不是 activation，也不授权真实操作。 | user value / scope / authorization / sequencing | confirmed-for-CP2-submission |

## Decision Brief 输入

- 用户真实意图：避免 raw trial count 把相关试验伪装成独立试验，同时用可审计离线方法解除长期 typed-unavailable。
- 认知盲区：算法选择不能在 CP2 偷跑；有效 count 可能与 raw 数值相等但仍必须有独立方法 provenance；offline ready 不是 real evidence。
- 推荐范围：8 项 REQ、8 场景、七字段 evidence、六类 golden vectors、单一 C1 projection、零真实授权。
- Deferred：具体策略身份、真实 producer/activation、C2-C4、FU-006、OI-005、aggregate/FU-009。
- 待用户决策：`DQ-CR173-001..008`。

## 讨论结论

- 场景充分性：`PASS_FOR_CP2_SUBMISSION`。
- BLOCKING_FOR_CP1：`0`。
- OPEN_FOR_CP2：`8`。
- `ready_for_design=false`。

