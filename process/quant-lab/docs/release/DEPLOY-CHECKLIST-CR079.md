# CR079 Deploy Checklist

## 适用性

CR079 无代码、无安装、无服务部署、无真实发布动作。本 checklist 只覆盖过程状态与证据交付。

## 检查表

| 项 | 状态 | 证据 |
|---|---|---|
| CP2/CP3/CP5 已批准 | PASS | `process/checkpoints/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md`; `process/checkpoints/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-REVIEW.md`; `process/checkpoints/CP5-CR079-DATA-REPORTS-ACCESS-READINESS.md` |
| CP6 inventory evidence 已生成 | PASS_WITH_RISK | `process/checks/CR079-DATA-REPORTS-ACCESS-INVENTORY-2026-06-17.md` |
| CP7 verification evidence 已生成 | PASS_WITH_RISK | `process/checks/CR079-DATA-REPORTS-ACCESS-VERIFICATION-2026-06-17.md` |
| Release Context 已生成 | PASS | `process/release/RELEASE-CONTEXT-CR079.yaml` |
| CP8 人工审查稿已生成 | PASS | `process/checkpoints/CP8-CR079-DELIVERY-READINESS.md` |
| 真实恢复 / 复制是否执行 | N/A | 当前 CR 不授权。 |

## 不执行项

- 不启动服务。
- 不安装依赖。
- 不运行 optional groups 或 full tests。
- 不访问 provider、lake、catalog、remote Git 或 QMT/MiniQMT runtime。
- 不复制、恢复、移动、重命名或删除 `reports/`、`data/`。
