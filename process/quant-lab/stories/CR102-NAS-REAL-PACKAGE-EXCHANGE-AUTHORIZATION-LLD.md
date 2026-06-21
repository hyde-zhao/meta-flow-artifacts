---
story_id: "CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION"
title: "CR102 NAS Real Package Exchange Authorization LLD"
status: "ready-for-cp5-review"
source_cr: "CR-102"
checkpoint: "CP5"
lld_policy:
  required_level: "full-lld"
  reason: "真实 NAS、权限、安全、证据和运行授权边界均为高风险对象。"
created_at: "2026-06-21T06:32:36+08:00"
owner: "host-orchestrator"
---

# CR102 NAS Real Package Exchange Authorization LLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-21 | host-orchestrator | 初版，定义 authorization matrix、execution plan shell、evidence schema、失败路径和 CP5 不授权边界。 |

## 1. 设计输入

| 输入 | 路径 | 状态 |
|---|---|---|
| CP2 scope review | `process/checkpoints/CP2-CR102-NAS-REAL-PACKAGE-EXCHANGE-SCOPE-REVIEW.md` | approved |
| CP3 HLD review | `process/checkpoints/CP3-CR102-NAS-REAL-PACKAGE-EXCHANGE-HLD-REVIEW.md` | approved |
| HLD | `docs/qmt/CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-HLD.md` | approved by CP3 |
| CR formal ledger | `process/changes/CR-102-NAS-REAL-PACKAGE-EXCHANGE-VALIDATION-AUTHORIZATION-GATE-2026-06-20.md` | active / cp5_pending |

## 2. 文件影响范围

| 类型 | 路径 | 操作 |
|---|---|---|
| 新增 | `docs/features/cr102-nas-real-package-exchange-authorization/DESIGN.md` | CP5 design-only |
| 新增 | `docs/features/cr102-nas-real-package-exchange-authorization/TEST-PLAN.md` | CP5 design-only |
| 新增 | `docs/features/cr102-nas-real-package-exchange-authorization/TASKS.md` | CP5 design-only |
| 新增 | `process/stories/CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-LLD.md` | CP5 design-only |
| 新增 | `process/context/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-CONTEXT.yaml` | CP5 human gate input |
| 新增 | `process/checks/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-READINESS.md` | auto precheck |
| 新增 | `process/checkpoints/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-READINESS.md` | manual gate |
| 更新 | `process/STATE.md` / `process/changes/CR-INDEX.yaml` / CR102 ledger | gate status |

## 3. 数据模型

```yaml
authorization_matrix:
  nas:
    path: "UNSET_BY_USER"
    export: "UNSET_BY_USER"
    package_exchange_root: "UNSET_BY_USER"
    mount:
      authorized: false
      local_mount_point: "UNSET_BY_USER"
      mount_command: "not-authorized"
      mount_mode: "not-authorized"
    permission:
      authorized_identity: "UNSET_BY_USER"
      read: false
      write: false
      copy: false
      publish: false
      pull: false
      delete: false
      check: false
      list: false
    operations:
      access: false
      list: false
      read: false
      write: false
      copy: false
      publish: false
      pull: false
      delete: false
      mount: false
      check: false
  credentials:
    env_read: false
    secret_read: false
    account_read: false
  runtime:
    qmt: false
    miniqmt: false
    xtquant: false
    gateway: false
  trading:
    submit: false
    cancel: false
    simulation: false
    live: false
  provider_lake_catalog:
    provider_fetch: false
    lake_write: false
    catalog_publish: false
```

## 4. 控制流

```text
CP5 approve
  -> mark design package approved
  -> keep authorization_matrix false
  -> keep CR102 blocked-for-execution
  -> wait for future explicit runtime_authorization gate

future runtime gate, only if user starts it
  -> collect path/mount/identity/operation/evidence/rollback
  -> run human gate
  -> if any required field missing: blocked
  -> if approved: user-defined execution route, outside current CP5
```

## 5. 异常与回退

| 异常 | 处理 |
|---|---|
| CP5 被误解为执行授权 | 停止推进，回滚到 CP5 pending 并重发不授权边界 |
| 用户要求当前直接 check | 回退到 CP2/CP3 修改 runtime_authorization 决策，不执行 |
| 用户提供真实路径 | 只记录为未来输入候选；当前会话仍不得访问或验证 |
| 用户要求 publish/delete | 拆 high-risk gate，不在当前 CP5 放行 |
| 证据 schema 被要求保存原文 | reject，保留 redacted-only schema |

## 6. 测试设计

| 测试 | 验收 |
|---|---|
| matrix false check | 10 类 NAS operation 均为 false |
| unset field check | path/export/root/mount/identity 均为 `UNSET_BY_USER` |
| forbidden domain check | env/secret/account/runtime/trading/publish flags 均为 false |
| evidence schema check | forbidden fields 不在 allowed schema |
| gate wording check | CP5 approve 明确不授权真实执行 |

## 7. 发布与回滚

当前 CP5 不发布、不执行、不部署。回滚方式是将 CR102 保持在 `cp3-approved-no-execution-authorized`，删除或废弃 CP5 design-only 产物，并重新发起 CP5。

## 8. 不授权边界

本 LLD 不授权访问、列取、读取、复制、写入、发布、拉取、删除、检查或挂载真实 NAS；不读取凭据/env/账户；不启动 QMT/MiniQMT/XtQuant/gateway runtime；不执行交易或 provider/lake/catalog publish。
