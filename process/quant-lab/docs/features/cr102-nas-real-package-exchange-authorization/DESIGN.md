---
feature_id: "CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION"
title: "CR102 NAS Real Package Exchange Authorization Design"
status: "ready-for-cp5-review"
source_cr: "CR-102"
checkpoint: "CP5"
created_at: "2026-06-21T06:32:36+08:00"
owner: "host-orchestrator"
---

# CR102 NAS Real Package Exchange Authorization Design

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-21 | host-orchestrator | 生成 CP5 design-only 设计，冻结 authorization matrix、execution plan shell、evidence schema 和 fail-closed 规则。 |

## 1. 设计目标

本设计承接已批准的 `CP3-CR102-NAS-REAL-PACKAGE-EXCHANGE-HLD-REVIEW`。CP5 只评审设计证据是否可作为后续 runtime authorization gate 的输入，不授权真实 NAS 动作。

目标：

- 冻结真实 NAS package exchange 的授权矩阵。
- 提供后续运行授权前的 execution plan shell，但不包含可直接执行的命令。
- 冻结 evidence schema，保证后续只允许脱敏摘要、hash、计数、状态和 pass/fail。
- 明确 CP5 approve 仍不授权 NAS access/list/read/write/copy/publish/pull/delete/check/mount。

## 2. 范围

| 类型 | 内容 |
|---|---|
| In Scope | authorization matrix、execution plan shell、evidence schema、rollback checklist、fail-closed preflight rules |
| Out of Scope | 真实 NAS 访问、路径发现、挂载、列取、读取、复制、写入、发布、拉取、删除、检查 |
| Out of Scope | 凭据/env/账户读取，QMT/MiniQMT/XtQuant/gateway runtime，交易，provider/lake/catalog publish |

## 3. Authorization Matrix

### 3.1 NAS Identity / Path State

| 字段 | 当前值 | 授权状态 | 规则 |
|---|---|---|---|
| `nas.path` | `UNSET_BY_USER` | not-authorized | 只能由用户在后续门禁显式提供 |
| `nas.export` | `UNSET_BY_USER` | not-authorized | 不由 agent 探测、推断或从 env 读取 |
| `nas.package_exchange_root` | `UNSET_BY_USER` | not-authorized | 不读取真实目录 |
| `nas.mount.local_mount_point` | `UNSET_BY_USER` | not-authorized | 不挂载、不检查 mount 状态 |
| `nas.permission.authorized_identity` | `UNSET_BY_USER` | not-authorized | 不读取账户、组、凭据或权限配置 |

### 3.2 Operation Matrix

| 操作 | 字段 | 当前授权 | CP5 是否可改变 | 后续改变条件 |
|---|---|---:|---:|---|
| access | `operations.access` | false | 否 | 用户提供 path / identity / scope 并通过 runtime authorization gate |
| list | `operations.list` | false | 否 | 用户明确授权 list，并定义脱敏 evidence |
| read | `operations.read` | false | 否 | 用户明确授权 read，并定义对象范围和敏感信息过滤 |
| write | `operations.write` | false | 否 | 用户明确授权 write，并定义 rollback |
| copy | `operations.copy` | false | 否 | 用户明确授权 copy，并定义源/目标/校验 |
| publish | `operations.publish` | false | 否 | 用户明确授权 publish，并接受高风险 gate |
| pull | `operations.pull` | false | 否 | 用户明确授权 pull，并定义本地隔离目标 |
| delete | `operations.delete` | false | 否 | 默认不建议；必须独立高风险门禁 |
| mount | `operations.mount` | false | 否 | 用户明确授权 mount 方式、主机和回退 |
| check | `operations.check` | false | 否 | 用户明确授权 check 且限定不读取内容 |

### 3.3 External Boundary

| 域 | 字段 | 当前授权 |
|---|---|---:|
| credentials | `env_read` / `secret_read` / `account_read` | false / false / false |
| runtime | `qmt` / `miniqmt` / `xtquant` / `gateway` | false / false / false / false |
| trading | `submit` / `cancel` / `simulation` / `live` | false / false / false / false |
| provider_lake_catalog | `provider_fetch` / `lake_write` / `catalog_publish` | false / false / false |

## 4. Execution Plan Shell

本 shell 是后续 runtime authorization gate 的结构模板，不是执行步骤，不包含命令。

| 阶段 | 必需输入 | Fail-closed 条件 | 输出 |
|---|---|---|---|
| AUTH-01 scope freeze | 用户明确 path/export/root、mount policy、identity、operation allowlist | 任一字段为 `UNSET_BY_USER` | 授权草案 |
| AUTH-02 risk review | 操作类型、对象范围、主机、身份、rollback、evidence schema | publish/delete/write/copy 未单独确认 | runtime Decision Brief |
| AUTH-03 preflight design | 最小权限、敏感字段过滤、hash/count/pass-fail 格式 | 需要读取 env/凭据/账户或启动 runtime | blocked |
| AUTH-04 manual approval | 用户逐项 approve | 用户未明确授权具体操作 | no execution |
| AUTH-05 future execution | 仅在后续 gate 明确授权后由用户指定执行者执行 | 当前 CP5 永远不进入此阶段 | future evidence |

CP5 approve 后允许进入的下一步仅是离线设计收敛或后续人工授权准备，不允许执行 AUTH-05。

## 5. Evidence Schema

### 5.1 Allowed Fields

```yaml
evidence_version: "cr102-nas-exchange-v1"
run_id: "USER_SUPPLIED_OR_GATE_ASSIGNED"
authorized_operation: "check | list | read | write | copy | publish | pull | delete | mount"
authorization_gate: "FUTURE_GATE_ID"
timestamp: "ISO-8601"
actor_role: "user | delegated-operator"
nas_path_ref: "REDACTED_REF_OR_USER_SUPPLIED_LABEL"
object_scope_ref: "REDACTED_REF_OR_USER_SUPPLIED_LABEL"
status: "PASS | FAIL | BLOCKED"
counts:
  files_seen: 0
  files_changed: 0
  bytes_changed: 0
hashes:
  manifest_hash: "sha256-or-N/A"
  package_hash: "sha256-or-N/A"
redaction:
  raw_path_stored: false
  credential_stored: false
  account_data_stored: false
  raw_log_stored: false
blocked_reason: ""
```

### 5.2 Forbidden Fields

- token、secret、password、private key、cookie、session。
- 账户、资金、持仓、委托、成交或原始日志。
- 未脱敏真实 NAS 完整路径或目录 listing。
- QMT/MiniQMT/XtQuant/gateway runtime 输出原文。
- provider/lake/catalog 原始 publish 日志。

## 6. Failure Paths

| 失败条件 | 处理 | 结果 |
|---|---|---|
| path/export/root 未提供 | fail closed | 不进入任何 execution |
| check/list/read 等操作仍为 false | fail closed | 输出 blocked reason |
| 需要读取 env/credential/account | fail closed | 切换到独立 credential/runtime gate |
| publish/delete/write 被请求 | high-risk gate required | 当前 CR102 CP5 不放行 |
| evidence 包含敏感字段 | blocked_redaction_failed | 不接受证据，回退 schema |
| CR089/QMT runtime 被引入 | route blocked | 新建或恢复独立 runtime_authorization CR |

## 7. File Owner 与后续可实现对象

| 对象 | 路径 | owner | CP5 结论 |
|---|---|---|---|
| Feature Design | `docs/features/cr102-nas-real-package-exchange-authorization/DESIGN.md` | host-orchestrator | design-only |
| TEST-PLAN | `docs/features/cr102-nas-real-package-exchange-authorization/TEST-PLAN.md` | host-orchestrator | design-only |
| TASKS | `docs/features/cr102-nas-real-package-exchange-authorization/TASKS.md` | host-orchestrator | design-only |
| LLD | `process/stories/CR102-NAS-REAL-PACKAGE-EXCHANGE-AUTHORIZATION-LLD.md` | host-orchestrator | design-only |
| Context Capsule | `process/context/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-CONTEXT.yaml` | host-orchestrator | human gate input |
| CP5 Precheck | `process/checks/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-READINESS.md` | host-orchestrator | auto precheck |
| CP5 Review | `process/checkpoints/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-READINESS.md` | host-orchestrator | human gate |

## 8. Gotchas

- CP5 approve 不是 NAS 授权。它只批准设计包可作为后续授权门禁输入。
- 不要把 `check` 视为安全的默认只读动作；没有 path/mount/identity 和 explicit check 授权时仍是 false。
- 不要把 CR100 fake exchange 的成功当作真实 NAS 权限证明。
- 不要把 CR101 READY_WITH_RISK 当作真实 QMT/NAS ready。
- 不要把 CR102 合并回 CR089，除非用户明确恢复 QMT/runtime/account 范围。
