# CR-047 CP2 Scenario Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-13 | host-orchestrator-inline-fallback | 记录 CR-047 用户自由表达确认、灰区与 CP2 决策候选。 |

## 用户可见确认

| ID | 用户表达 | 确认理解 | 状态 | 影响面 |
|---|---|---|---|---|
| SGQ-CR047-001 | “其他问题需要解决”与“开始推进这个CR” | 用户确认把已复现的 workflow truth、Doctor、guardrail、Ruff、安装与 CR-046 状态收敛作为一个 CR-047 目标包推进。 | confirmed-freeform | scope, validation, delivery gate |
| SGQ-CR047-002 | “不使用子agent，继续推进” | CR-047 后续阶段禁用子 Agent；使用逐阶段、逐角色记录的 user-approved inline fallback。 | confirmed-freeform | execution mode, evidence, verification risk |

## Scenario Gray Areas

| Gray Area | 为什么重要 | 状态 | CP2 决策 |
|---|---|---|---|
| SGA-WT-01 根规则 canonical source | 决定 clean clone 与个人 wrapper 的边界 | open | CP2-DQ-01 |
| SGA-WT-02 Doctor warning 与退出码 | 决定何时可宣称质量门通过 | open | CP2-DQ-02 |
| SGA-WT-03 历史 budget 收敛 | 决定是否改写历史或隔离冷数据 | open | CP2-DQ-03 |
| SGA-WT-04 ignored cache 策略 | 决定测试后本机 guardrail 是否永久变红 | open | CP2-DQ-04 |

## Deferred Ideas

- repository-verifiable platform receipt producer：等待平台能力。
- 独立 runtime/SaaS/pilot：等待独立授权。
- prelink backup：用户明确排除，不进入整改。
