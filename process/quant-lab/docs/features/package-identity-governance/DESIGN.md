---
status: "current"
version: "1.0"
change: "CR-109"
created_at: "2026-06-21T22:52:00+08:00"
created_by: "host-orchestrator"
---

# Feature Design: Package Identity Governance

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-21 | host-orchestrator | 为 package identity 相关历史质量证据建立功能域入口；当前 canonical identity 仍以 `docs/design/PACKAGE-IDENTITY.yaml` 为准。 |

## 目标

维护 quant-lab 的产品身份、包名、CLI 名称和 legacy alias 兼容策略，避免历史 `local-backtest` 文档被误读为当前主身份。

## 当前真相源

| 对象 | 路径 |
|---|---|
| Package identity | `docs/design/PACKAGE-IDENTITY.yaml` |
| Historical quality evidence | `docs/quality/by-feature/package-identity-governance/by-cr/` |

## 边界

本功能域不授权批量改业务导入路径、不授权 runtime、不授权凭据读取。任何 package rename 或 import path 变更必须单独走 CR。
