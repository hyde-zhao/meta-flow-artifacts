---
status: "pending-cp5-approval"
version: "1.0"
change_id: "CR-060"
title: "Preserve-audit Decisions for CR060"
created_by: "host-orchestrator"
created_at: "2026-06-15T06:43:41+08:00"
---

# CR060 Preserve-audit Decisions

## 1. 默认策略

CR060 默认继承 CR058 的 preserve-audit allowlist：历史过程证据中的 `local_backtest` 是当时事实，不因为当前 canonical identity 变为 `quant-lab` 而批量改写。

## 2. 保留清单

| ID | path_pattern | 决策 | 原因 |
|---|---|---|---|
| PAD-CR060-001 | `process/**` | preserve-audit | 运行态过程证据不可重写历史语境。 |
| PAD-CR060-002 | `process/checks/**` | preserve-audit | 自动检查结果是门禁证据。 |
| PAD-CR060-003 | `process/checkpoints/**` | preserve-audit | 人工确认文本是审计事实。 |
| PAD-CR060-004 | `process/handoffs/**` | preserve-audit | 子 agent 调度证据需保留原上下文。 |
| PAD-CR060-005 | `process/stories/**` | preserve-audit | Story / LLD / implementation evidence 保留当时命名。 |
| PAD-CR060-006 | `process/context/**` | preserve-audit | Context capsule 是恢复点证据。 |
| PAD-CR060-007 | `process/changes/CR-*.md` historical | preserve-audit | 历史 CR 保留原事实，CR060 自身可使用 `quant-lab`。 |
| PAD-CR060-008 | `DEV-LOG.md` | preserve-audit | 时间线日志不做机械改写。 |
| PAD-CR060-009 | `checkpoints/**` legacy fallback | preserve-audit | legacy fallback 仅作读取兼容。 |

## 3. 允许改写清单

| ID | path | 决策 | 条件 |
|---|---|---|---|
| PAD-CR060-010 | `README.md` future-facing identity paragraphs | allow-after-cp5 | 只改用户入口和项目身份说明，保留 legacy alias 段。 |
| PAD-CR060-011 | `docs/USER-MANUAL.md` future-facing identity paragraphs | allow-after-cp5 | 与 README 同步，不改 data lake / runtime / historical CR 语义。 |

## 4. 单项豁免

当前豁免数量：0。

新增豁免必须包含 path、owner、reason、risk、rollback_ref、approval_decision_id，并重新进入 CP5 或后续等价人工门禁。

## 5. 敏感边界

CR060 不读取 `.env`、token、password、private key、cookie、session、账户、资金、持仓、委托、成交或未脱敏交易事实。任何需要打开敏感正文才能判断的项目一律转为 blocked。
