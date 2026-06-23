---
doc_id: "HUMAN-GATE-LAUNCH-MESSAGE-CHECKER-IMPLEMENTATION-CANDIDATE-CR119"
cr_id: "CR-119"
status: "ready-for-cp8"
version: "0.1"
created_at: "2026-06-22T19:14:46+08:00"
updated_at: "2026-06-22T19:30:58+08:00"
owner: "host-orchestrator"
---

# Human Gate Launch Message Checker Implementation Candidate - CR119

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-22 | host-orchestrator | 创建 CR119 implementation candidate notes；只定义 CP2 范围，不授权源码/tests/checker 修改。 |
| 0.2 | 2026-06-22 | host-orchestrator | CP2 approved 后进入 CP5 implementation authorization；记录最小 owner discovery 和 CP5 授权边界。 |
| 0.3 | 2026-06-22 | host-orchestrator | CP5 approved 后完成最小实现和验证，进入 CP8 delivery readiness。 |

## 背景

CR115 已定义 human gate launch message 的 checkable draft contract，要求人工门禁发起消息具备 checklist 路径、自动预检结论、Context Capsule Summary、Decision Collection Coverage、待决策项数量、待决策表、不授权项和三个 exact replies。

CR118 已完成 path alias checker enforcement，强化 `process/docs/design/*` 与 `docs/design/*` 别名混用的检查。CR119 的候选目标是评估是否把 CR115 的 launch message contract 纳入 checker implementation，使后续 CP2 / CP3 / CP5 / CP8 的人工门禁发起消息可以被本地静态 / fixture 检查覆盖。

## 推荐路线

| 阶段 | 推荐动作 | 授权边界 |
|---|---|---|
| CP2 | 确认 CR119 范围、分阶段实施路线和 runtime 禁止边界 | 只写 process artifact，不改源码/tests/checker。 |
| CP5 | 若 CP2 approved，定义最小 owner、实现切片、fixture 范围和验证命令 | CP5 approved 后才允许源码/tests/checker implementation 修改。 |
| CP6 / CP7 | 实施并验证 checker 对 launch message contract 的覆盖 | 仅允许本地静态 / fixture 验证。 |
| CP8 | 关闭 CR119 或转 follow-up | 不自动授权 runtime/NAS/credentials/trading/publish。 |

## 候选检查范围

后续 CP5 可考虑的 checker target definition：

- 发起消息必须包含 `process/checkpoints/CP*.md` checklist 路径。
- 发起消息必须包含自动预检结论。
- 发起消息必须包含 Context Capsule Summary 或等价摘要。
- 发起消息必须包含 Decision Collection Coverage 摘要。
- 待决策项数量大于 0 时，发起消息必须包含待决策表格。
- 发起消息必须展示 `approve`、`修改: <具体修改点>`、`reject` 三个 exact replies。
- 发起消息必须说明 `approve` 不代表授权的不授权项。

## 非目标

- CP2 不修改 `scripts/check_human_gate_decision_brief.py`。
- CP2 不新增或修改 tests。
- CP2 不实现 checker behavior。
- 不启动 runtime、不访问 NAS、不读取凭据、不触发交易写、不执行 provider/lake/catalog publish。
- 不批量重写历史 checkpoint 或 STATE 长表。

## 风险与回退

| 风险 | 等级 | 缓解 |
|---|---|---|
| CP2 与 CP5 授权混淆，导致提前改 checker | medium | CP2 checkpoint 明确 `source/tests/checker` 未授权，CP5 前不得实施。 |
| checker 目标过宽导致误报历史文件 | medium | CP5 只选择最小 fixture / active gate 文件，历史批量治理另起 CR。 |
| path alias 与 launch message contract 两类规则混在一起 | low | 保留 CR118 path alias 规则，CR119 仅追加 launch message 发起消息规则。 |

## 当前结论

CR119 当前 ready for CP8。CP5 已批准并实施最小切片：`scripts/check_human_gate_decision_brief.py` 新增 context / coverage 摘要检查，`tests/test_cr119_human_gate_launch_message_checker.py` 覆盖通过和缺失场景。后续扩大历史扫描或自动生成 launch message 需另起 CR。
