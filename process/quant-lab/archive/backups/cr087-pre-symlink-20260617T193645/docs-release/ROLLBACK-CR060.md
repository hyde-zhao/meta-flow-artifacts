---
status: "ready-with-risk"
version: "1.0"
change_id: "CR-060"
title: "CR060 Rollback"
created_by: "host-orchestrator"
created_at: "2026-06-15T06:43:41+08:00"
---

# CR060 Rollback

## 回滚对象

| 对象 | 回滚方式 |
|---|---|
| `README.md` | 使用 Git diff 反向补丁，或从 CR059 bundle / cold backup 恢复指定文件。 |
| `docs/USER-MANUAL.md` | 使用 Git diff 反向补丁，或从 CR059 bundle / cold backup 恢复指定文件。 |

## 禁止回滚方式

不使用 `git reset --hard`，不覆盖未纳入 CR060 的用户改动，不删除 NAS 数据。
