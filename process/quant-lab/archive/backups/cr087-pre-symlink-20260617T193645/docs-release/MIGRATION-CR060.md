---
status: "ready-with-risk"
version: "1.0"
change_id: "CR-060"
title: "CR060 Migration"
created_by: "host-orchestrator"
created_at: "2026-06-15T06:43:41+08:00"
---

# CR060 Migration

CR060 是 repo-local docs-only identity migration 的第一个真实执行切片。

| 迁移项 | 状态 | 说明 |
|---|---|---|
| README identity | done | `quant-lab` canonical + `local_backtest` legacy alias。 |
| USER-MANUAL identity | done | 同步 README。 |
| package/import | deferred | CR061。 |
| Git remote | deferred | CR062。 |
| NAS archive | deferred | CR063。 |
| Data lake root/catalog | deferred | CR064。 |
| Trading runtime | deferred | CR065 / CR046。 |
