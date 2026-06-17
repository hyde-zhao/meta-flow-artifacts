---
story_id: "CR058-S03"
status: "verified-with-risk-static-only"
change_id: "CR-058"
title: "Preserve-audit Allowlist and Sensitive Filter Gate"
lld_policy:
  required_level: "full-lld"
feature_design_refs:
  - "FEAT-CR058-03"
dev_gate: "Preserve-audit allowlist approved; sensitive inputs excluded."
---

# CR058-S03 Preserve-audit Allowlist and Sensitive Filter Gate

## 技术说明

本 Story 定义默认 preserve-audit 和敏感输入过滤规则。

默认 preserve-audit allowlist：

- `process/**`
- `process/checks/**`
- `process/checkpoints/**`
- `process/handoffs/**`
- `process/stories/**` historical non-CR058
- `DEV-LOG.md` historical entries
- legacy `checkpoints/**`

敏感输入默认排除：

- `.env`、`*.env`
- token、password、cookie、session、private key、pem
- 未脱敏账户、资金、持仓、委托、成交原始事实
- 外置 data lake 正文、NAS archive 正文、untracked data

验收条件：任何 preserve-audit 路径进入 mechanical rewrite 时必须 fail-closed。
