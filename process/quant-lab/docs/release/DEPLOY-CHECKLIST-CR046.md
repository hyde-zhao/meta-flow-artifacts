# CR046 Deploy Checklist

> 结论：`READY_WITH_RISK`，CP8 已 approved。
> 本次无软件部署、无服务启动、无交易终端操作；仅发布过程文档、契约和检查点证据。

## Entry Criteria

| 项 | 状态 | 证据 |
|---|---|---|
| CP6 implementation 已完成 | PASS | `process/checks/CP6-CR046-DUAL-TARGET-FRAMEWORK-BATCH-A-CODING-DONE.md` |
| CP7 verification 已完成 | PASS_WITH_RISK | `process/checks/CP7-CR046-DUAL-TARGET-FRAMEWORK-BATCH-A-VERIFICATION-DONE.md` |
| 质量报告存在 | PASS | `process/docs/quality/VERIFICATION-REPORT-CR046.md`、`process/docs/quality/TEST-REPORT-CR046.md`、`process/docs/quality/REVIEW-CR046.md`、`process/docs/quality/FIXES-CR046.md` |
| Release Context 已生成 | PASS | `process/release/RELEASE-CONTEXT-CR046.yaml` |

## Checklist

| 检查项 | 状态 | 说明 |
|---|---|---|
| 发布范围限定为 framework-first 文档 / 契约 | PASS | 不包含具体策略、runner 实现或 runtime。 |
| release notes 已生成 | PASS | `process/docs/release/RELEASE-NOTES-CR046.md` |
| rollback 方案已生成 | PASS | `process/docs/release/ROLLBACK-CR046.md` |
| migration 说明已生成 | PASS | `process/docs/release/MIGRATION-CR046.md` |
| feedback 入口已生成 | PASS | `process/docs/release/FEEDBACK-CR046.md` |
| 不授权项已显式列出 | PASS | CP8 checkpoint 和 launch message 均列出。 |
| 后续路线未自动启动 | PASS | CR089 / CR091 保持 blocked；CR047..CR050 仅为候选。 |

## 不执行项

| 项 | 状态 |
|---|---|
| QMT/MiniQMT/XtQuant/gateway 启动或连接 | not-authorized |
| NAS 访问 | not-authorized |
| `.env` / 凭据 / 账号 / 账户 / 资金 / 持仓 / 委托 / 成交 / 日志原文读取 | not-authorized |
| submit/cancel | not-authorized |
| simulation/live | not-authorized |
| provider fetch / lake write / catalog publish | not-authorized |

## Exit Criteria

CP8 已人工确认通过，CR046 当前交付关闭为 `closed-current-delivery / READY_WITH_RISK`。若未来发现范围或风险接受误判，只能通过新的修订 CR 或回滚流程处理。
