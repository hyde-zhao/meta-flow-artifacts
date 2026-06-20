---
story_id: CR101-S04-docs-validation-and-follow-up-gates
change_id: CR-101
title: Docs, validation matrix, and follow-up authorization gates LLD
status: implemented-and-verified
created_at: "2026-06-20T09:20:00+08:00"
owner: host-orchestrator
source_hld: docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md
cp6_check: process/checks/CP6-CR101-S04-docs-validation-and-follow-up-gates-CODING-DONE.md
cp7_check: process/checks/CP7-CR101-S04-docs-validation-and-follow-up-gates-VERIFICATION-DONE.md
---

# CR101-S04 Docs, validation matrix, and follow-up authorization gates LLD

## 1. 目标

把 CR101 的实现后文档、验证矩阵、CP7/CP8 预期和后续 gate 统一收口，防止离线证据被误读为真实 QMT / MiniQMT / NAS / trading ready。

## 2. 文件影响范围

| 文件 | 操作 | 说明 |
|---|---|---|
| `docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md` | 小幅同步 | 若实现命名或风险处理偏离 HLD，追加修订记录。 |
| `docs/qmt/*CR101*` | 新增 / 修改 | 可新增实现说明、验证说明或 follow-up gate 摘要。 |
| `process/checks/CP7-*CR101*` | 后续生成 | 验证报告必须明确 offline-only。 |
| `process/checkpoints/CP8-*CR101*` | 后续生成 | 终验必须区分 READY / READY_WITH_RISK。 |
| `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md` | 按需更新 | 只更新候选状态和后续 gate，不创建未授权正式 CR。 |

禁止：真实 runtime logs、账户数据、凭据、NAS 内容、publish 证据原文。

## 3. 文档契约

CR101 文档必须包含：

- 当前支持：离线 QMT direct-run package contract 与 MiniQMT readonly adapter contract。
- 当前不支持：真实 QMT direct-run proof、真实 MiniQMT gateway proof、真实 NAS exchange、order-write。
- 后续 gate：
  - QMT-DIRECT-RUN-VALIDATION-FU。
  - MINIQMT-GATEWAY-ADAPTER-VALIDATION-FU。
  - NAS-REAL-EXCHANGE-FU。
  - ORDER-WRITE-SIMULATION-LIVE-FU。
- 不授权清单。
- READY_WITH_RISK 条件。

## 4. 验证设计

| 验证项 | 方式 |
|---|---|
| 不授权项存在 | grep / human gate checklist。 |
| 后续 gate 存在 | docs / follow-up tracking 检查。 |
| 离线与真实 ready 区分 | CP7 / CP8 结论必须包含 residual risk。 |
| human gate 合规 | `scripts/check_human_gate_decision_brief.py`。 |

## 5. 实施步骤

1. 汇总 S01 / S02 / S03 的实际实现命名。
2. 更新 CR101 docs / validation matrix。
3. 更新 follow-up tracking 中 CR101 后续候选，不自动启动。
4. 生成 CP7 / CP8 evidence 时保留 READY_WITH_RISK 判断。
5. 运行 human gate / cr-tracking / workspace check。

## 6. 风险与回退

| 风险 | 缓解 |
|---|---|
| 文档声称真实 ready | 固定不授权清单和 READY_WITH_RISK 语句。 |
| 后续 gate 被自动启动 | 明确候选状态为 candidate-not-started。 |
| CP8 关闭范围不清 | CP8 Decision Brief 单独列 not-authorized 和 residual risk。 |

回退：若用户要求真实验证，将 CR101 停在 CP8 前，另起 runtime_authorization CR，不在本 CR 内追加真实执行。

## 7. 不授权项

本 LLD 不授权 NAS、凭据、账户 / 持仓 / 委托 / 成交 / 原始日志、QMT/MiniQMT/XtQuant/gateway runtime、simulation/live、交易或 publish。
