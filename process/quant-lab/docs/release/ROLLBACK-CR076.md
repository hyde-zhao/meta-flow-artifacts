# Rollback CR076

## 回滚范围

CR076 只写入 process / release 台账。回滚也只允许回滚这些台账文件，不允许借回滚名义访问或清理 `reports/`、`data/`。

## 回滚条件

- CP8 被用户 `reject`。
- 用户要求修改 policy-first / no-data-touch 决策。
- 后续发现 CR076 台账与 STATE / CR-INDEX 不一致。

## 回滚动作

1. 将 CR076 状态从 `active-cp8-review-pending` 回退到 `active-approved-policy-ready-for-cp8` 或 `active-gate-pending`。
2. 回滚或修订 `process/changes/CR-076-DATA-REPORTS-ACCESS-POLICY-2026-06-17.md`、`process/STATE.md`、`process/changes/CR-INDEX.yaml`。
3. 重新运行 CR tracking consistency 和 human-gate 校验。

## 禁止动作

- 禁止读取、列出、复制、恢复、比对或删除 `reports/`、`data/`。
- 禁止读取 `.env` 或任何凭据。
- 禁止 provider/lake/catalog、remote Git、runtime、CR046 recovery 或 cleanup。
