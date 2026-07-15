---
title: "CR-169 Migration"
status: "not-applicable-runtime"
version: "1.0"
cr_id: "CR-169"
created_at: "2026-07-15T10:15:00+08:00"
owner: "host-orchestrator"
---

# CR-169 Migration

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator | 说明 additive schema 激活和无数据/runtime migration。 |

## 结论

无数据、数据库、lake、provider、runtime 或部署迁移。唯一合同演进是 neutral catalog 中 `capacity_liquidity@v1` 从保留位演进为 active descriptor，同时保留 `reserved` descriptor 兼容旧引用。

未来 C4 方法族、alpha-decay、真实 ADV/liquidity 或 schema v2 必须通过独立 CR；v1 不在本次交付后隐式扩展。
