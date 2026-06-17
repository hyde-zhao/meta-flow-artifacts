---
story_id: "CR058-S05"
status: "verified-with-risk-static-only"
change_id: "CR-058"
title: "Mechanical Migration Dry-run and Authorization Boundary"
lld_policy:
  required_level: "technical-note"
feature_design_refs:
  - "FEAT-CR058-05"
dev_gate: "No-op authorization boundary approved; real execution separately authorized."
---

# CR058-S05 Mechanical Migration Dry-run and Authorization Boundary

## 技术说明

本 Story 定义后续 dry-run / no-op 输出边界。CP5 通过只允许进入“设计确认后的文档或 no-op 报告实现”，不授权真实 `mv`、`rename`、`delete` 或批量替换。

不授权项：

1. 真实文件 move / rename / delete。
2. 批量路径替换或 rewrite。
3. NAS mount / scan / mkdir / copy / delete / migration。
4. `MARKET_DATA_LAKE_ROOT` replacement 或数据湖迁移。
5. git push / tag / remote rename / history rewrite。
6. `.env`、token、password、cookie、session、private key 读取。
7. QMT / MiniQMT runtime、账户查询或交易动作。
8. provider fetch / lake write / catalog publish。
9. Windows full archive / cold / full lake mount。
10. 自动启动 CR060+ / data lake migration / trading runtime CR。

验收条件：任何真实动作必须作为新的 `runtime_authorization` 或 implementation 决策项进入人工门禁。
