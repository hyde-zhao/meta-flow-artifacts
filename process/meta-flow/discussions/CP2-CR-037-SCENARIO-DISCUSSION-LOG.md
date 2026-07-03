# CP2 CR-037 Scenario Discussion Log

## CR-037：项目治理与状态强制整改场景基线

| 字段 | 内容 |
|---|---|
| 日期 | 2026-07-02 |
| 来源 | 用户多轮评审、整改方案收敛和当前 CP2 推进授权 |
| 关联 CR | `process/changes/CR-037.md` |
| 场景基线 | `process/docs/product/USE-CASES.md` v1.1 |
| 当前状态 | `CR-037` 已 active；`CR-036` 仍 blocked / unfinished |

## 用户反馈摘要

用户要求 meta-flow 针对 quant-lab 暴露出的长期项目治理问题进行整改：命名管理、STATE.current.json 污染、token 读取膨胀、roadmap / milestone / project phase 缺失、项目规模治理不足、impact_surface 语义漂移和 capability registry 归一。多轮评审后，用户确认方案应避免新建第二套机制，优先复用既有 state、context、result、ledger、gate profile、CR index、capability governance 和 CP result 体系。

## 候选理解

| 候选 | 说明 | 处理结论 |
|---|---|---|
| A | 将 CR-037 作为一个目标导向整改包，按 P0 state enforcement、P0.5 ledger hygiene、P1 project governance、P2 quant-lab migration 分阶段推进。 | 用户接受，作为本轮主选 |
| B | 将每个机制拆成独立 CR，分别推进 state、roadmap、scale、registry、migration。 | 放弃；会割裂八轮评审收敛出的统一治理边界，并增加审批负担 |
| C | 仅修补 quant-lab 当前文档命名问题，不改 meta-flow 治理机制。 | 放弃；无法解决 STATE 污染、长期项目治理和 token 膨胀根因 |

## Scenario Gray Areas

| 灰区 ID | 问题 | 处理结论 |
|---|---|---|
| SGA-CR037-01 | 是否新增 hot/warm/cold、roadmap_impact、PROJECT-LEDGER、自由 capability_refs 等并行机制？ | 否。CR-037 必须复用既有 state/context/result/ledger/registry 体系，新增平行机制应被设计审查阻断。 |
| SGA-CR037-02 | roadmap refresh 是否自动修改 quant-lab 发布库正式文档？ | 否。自动 cascade 只写过程归档库机器状态；发布库只输出 must_check / stale_items / follow-up 候选。 |
| SGA-CR037-03 | CR-036 暂停是否等于完成或关闭？ | 否。CR-036 仍 blocked / unfinished；CP2 approve CR-037 不关闭、恢复或完成 CR-036。 |
| SGA-CR037-04 | capability_refs 能否使用方案中臆造的自由字符串？ | 否。必须引用标准 registry 中真实 ID；缺失注册时 blocked 或生成 follow-up。 |
| SGA-CR037-05 | CR-037 CP2 是否授权实现、CP5、runtime、production write 或 quant-lab 发布库修改？ | 否。CP2 只确认产品 / 场景 / 范围基线；实现、CP5、runtime、生产写入和发布库修改均需后续门禁或独立授权。 |

## 用户可见确认记录

| Question ID | 问题 | 用户回答 | 复述确认 | 状态 |
|---|---|---|---|---|
| SGQ-CR037-001 | 是否接受八轮收敛后的 CR-037 实施方案先建档并按 meta-flow 流程推进？ | “好的，你的实施方案和计划通过，先形成文档，方便后续跟踪。” | 用户接受实施方案与计划，要求形成长期可跟踪文档。 | confirmed |
| SGQ-CR037-002 | 是否先完成或暂停 CR-036，然后激活 CR-037 并生成 CP0？ | “同意先完成或暂停 CR-036，然后激活 CR-037 并生成 CP0。” | 用户授权暂停未完成的 CR-036，并激活 CR-037；不将 CR-036 视为完成或关闭。 | confirmed |
| SGQ-CR037-003 | CP2 输入状态语义修正后，是否可按计划推进下一步？ | “审核通过，你可以按照计划推进下一步了。” | 用户确认状态语义同步整改通过，授权生成 CP2 自动预检和人工审查稿。 | confirmed |

## 下游衔接

- 正式场景基线：`process/docs/product/USE-CASES.md`。
- 正式需求基线：`process/docs/product/REQUIREMENTS.md`。
- 工程验证场景：`process/docs/product/SCENARIOS.yaml`。
- 测试覆盖矩阵：`process/docs/product/TEST-MATRIX.md`。
- 规划基线：`process/docs/product/STORY-MAP.md`、`process/docs/product/MVP-SCOPE.md`。
- CP2 approve 后只允许进入 CP3 蓝图 / HLD 人工确认路径；不授权实现、CP5、runtime、生产写入或 quant-lab 发布库修改。
