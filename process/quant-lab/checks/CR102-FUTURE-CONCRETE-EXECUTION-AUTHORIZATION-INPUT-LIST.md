---
check_id: "CR102-FUTURE-CONCRETE-EXECUTION-AUTHORIZATION-INPUT-LIST"
check_name: "CR102 Future Concrete Execution Authorization Gate Input List"
type: "authorization-input-list"
status: "READY_FOR_CONCRETE_EXECUTION_GATE_REVIEW_NO_EXECUTION_AUTHORIZED"
owner: "host-orchestrator"
created_at: "2026-06-21T07:24:28+08:00"
updated_at: "2026-06-21T08:39:38+08:00"
source_gate: "CR102-FUTURE-RUNTIME-AUTHORIZATION"
source_checkpoint: "process/checkpoints/CR102-FUTURE-RUNTIME-AUTHORIZATION-GATE-REVIEW.md"
source_cr: "process/changes/CR-102-NAS-REAL-PACKAGE-EXCHANGE-VALIDATION-AUTHORIZATION-GATE-2026-06-20.md"
execution_authorized: false
nas_access_authorized: false
credential_read_authorized: false
runtime_authorized: false
trading_authorized: false
publish_authorized: false
execution_machine_mount_root: "manual-execution-machine-side"
authorized_identity_label: "current-shell-user"
credential_source: "none-needed"
pull_target_path_or_label: "manual-execution-machine-verification"
---

# CR102 Future Concrete Execution Authorization Gate 输入清单

## 结论

本文件只定义后续 `future concrete execution authorization gate` 发起前必须由用户显式提供或确认的输入字段。当前状态仍是 `preparation-only / no-execution`。

本文件不授权访问、列取、读取、复制、写入、发布、拉取、删除、检查或挂载真实 NAS；不授权读取凭据 / env / 账户；不授权启动 QMT / MiniQMT / XtQuant / gateway runtime；不授权交易；不授权 provider / lake / catalog publish。

## 来源与边界

| 来源 | 路径 | 状态 | 本清单采用方式 |
|---|---|---|---|
| Future runtime gate | `process/checkpoints/CR102-FUTURE-RUNTIME-AUTHORIZATION-GATE-REVIEW.md` | approved | 采用 DQ-RUNTIME-CR102-01..06 的 preparation-only 决策 |
| CR102 正式账本 | `process/changes/CR-102-NAS-REAL-PACKAGE-EXCHANGE-VALIDATION-AUTHORIZATION-GATE-2026-06-20.md` | active / not_ready | 采用当前 `UNSET_BY_USER` 和全部 operation=false 基线 |
| CP5 设计包 | `process/checkpoints/CP5-CR102-NAS-REAL-PACKAGE-EXCHANGE-READINESS.md` | approved-no-execution | 采用 authorization matrix / execution plan shell / evidence schema 的 design-only 约束 |
| 用户当前指令 | 当前对话 | no-execution | 明确禁止真实 NAS/env/runtime/trading/publish 访问或检查 |

## Gate 发起前置条件

| # | 前置条件 | 当前状态 | 缺失时处理 |
|---|---|---|---|
| 1 | 用户明确选择本次要申请的 operation class | 缺失 | 不发起执行门禁 |
| 2 | 用户显式提供 NAS 标识输入，且说明是否可写入门禁草案 | 缺失 | 不访问、不探测、不推断 |
| 3 | 用户显式提供 mount / access 现状输入，且说明 agent 是否仍不得检查 | 缺失 | 不检查 mount，不执行 `mount` / `showmount` / `ls` |
| 4 | 用户显式提供执行主体和权限边界 | 缺失 | 不读取账户，不读取 env，不推断 identity |
| 5 | 用户显式确认 operation allowlist / denylist | 缺失 | 全部 operation 保持 false |
| 6 | 用户显式确认 evidence 脱敏格式 | 缺失 | 不生成运行证据，不接受原始日志 |
| 7 | 用户显式确认 rollback / stop 条件 | 缺失 | 不发起真实执行门禁 |
| 8 | 用户显式确认本次是否只是草案门禁 | 缺失 | 默认草案，不执行 |

## 用户必须提供 / 确认的输入

### A. Gate Intent

| 字段 | 必填 | 允许值 / 格式 | 当前值 | 说明 |
|---|---|---|---|---|
| `gate_intent` | 是 | `draft_only` / `authorize_check_only` / `authorize_list_read` / `authorize_copy_pull` / `authorize_write_publish` / `authorize_delete` / `cancel` | `draft_only` | 推荐先选 `draft_only` 或 `authorize_check_only`。除 `draft_only` 外均需要后续人工门禁再次 approve。 |
| `requested_operation_class` | 是 | `check-only` / `list-read` / `copy-pull` / `write-publish` / `delete` | `UNSET_BY_USER` | `write-publish` 和 `delete` 必须拆 high-risk gate。 |
| `execution_window` | 是 | 文本时间窗，或 `manual-later` | `manual-later` | 不得由 agent 推断当前即可执行。 |
| `executor` | 是 | `user-manual` / `agent-assisted-after-gate` / `external-operator` | `UNSET_BY_USER` | 当前未授权 agent-assisted execution。 |

### B. NAS Scope

| 字段 | 必填 | 允许值 / 格式 | 当前值 | 说明 |
|---|---|---|---|---|
| `nas_label` | 是 | 脱敏标签，如 `nas-prod-package-exchange` | `UNSET_BY_USER` | 建议先提供脱敏 label，不要求在对话公开真实地址。 |
| `nas_path_or_export` | 条件必填 | 用户显式文本；可填 `withheld-offline` | `UNSET_BY_USER` | 若不提供真实值，则只能准备草案，不能授权执行。 |
| `package_exchange_root` | 条件必填 | 用户显式文本；可填 `withheld-offline` | `UNSET_BY_USER` | 不得由 agent 探测或列取。 |
| `object_scope` | 是 | 精确对象 / 包名 / 版本 / 子目录；可用脱敏 label | `UNSET_BY_USER` | 禁止笼统授权整个 NAS。 |
| `scope_exclusions` | 是 | 不得接触的目录 / 包 / 文件类别 | `UNSET_BY_USER` | 缺失时默认所有未列明对象禁止接触。 |

### C. Mount / Access State

| 字段 | 必填 | 允许值 / 格式 | 当前值 | 说明 |
|---|---|---|---|---|
| `mount_required` | 是 | `false` / `true-after-separate-gate` | `false` | 本清单不授权挂载。 |
| `local_mount_point` | 条件必填 | 用户显式文本；可填 `not-mounted` / `withheld-offline` | `UNSET_BY_USER` | agent 不检查本机 mount。 |
| `mount_check_authorized` | 是 | `false` / `true-after-gate` | `false` | 当前仍为 false；即使 check-only，也需后续门禁。 |
| `network_probe_authorized` | 是 | `false` / `true-after-gate` | `false` | 当前禁止 `showmount`、`rpcinfo`、`nc` 等真实探测。 |

### D. Identity / Permission Boundary

| 字段 | 必填 | 允许值 / 格式 | 当前值 | 说明 |
|---|---|---|---|---|
| `authorized_identity_label` | 是 | 脱敏身份标签 | `UNSET_BY_USER` | 不读取系统账户、env 或凭据。 |
| `credential_source` | 是 | `user-held-not-shared` / `external-operator` / `none-needed` | `user-held-not-shared` | 禁止提交 token、password、secret、private key。 |
| `permission_read` | 是 | `false` / `true-after-gate` | `false` | 默认 false。 |
| `permission_write` | 是 | `false` / `true-high-risk-gate` | `false` | 写权限必须 high-risk gate。 |
| `permission_delete` | 是 | `false` / `true-high-risk-gate` | `false` | 删除默认不建议授权。 |

### E. Operation Matrix

| 操作 | 当前授权 | 下一门禁可否申请 | 最低输入要求 |
|---|---|---|---|
| `access` | false | 仅随具体 operation 申请 | NAS scope、identity、evidence、stop 条件 |
| `check` | false | 可申请 check-only gate | NAS label/path、object scope、identity、只返回脱敏 pass/fail/hash/count |
| `list` | false | 需 list-read gate | 精确目录范围、最大条目数、禁止输出原始路径策略 |
| `read` | false | 需 list-read gate | 精确对象、禁止原文落盘、脱敏 evidence |
| `copy` | false | high-risk gate | 源/目标、覆盖策略、空间/校验、rollback |
| `pull` | false | high-risk gate | 源/目标、包版本、校验、禁止泄露内容 |
| `write` | false | high-risk gate | 目标、临时文件策略、原子性、rollback |
| `publish` | false | high-risk gate | 发布对象、版本、幂等、回滚、污染防护 |
| `delete` | false | high-risk gate | 默认不建议；必须单独 delete gate | 目标、备份、双人确认、rollback 可行性 |
| `mount` | false | separate mount gate | mount point、权限、卸载策略、失败回退 |

### F. Evidence Contract

| 字段 | 必填 | 允许值 / 格式 | 禁止内容 |
|---|---|---|---|
| `evidence_profile` | 是 | `redacted-status-only` / `redacted-hash-count` / `manual-summary` | raw path、raw log、secret、account、token |
| `allowed_evidence_fields` | 是 | status、operation、nas_label、object_label、hash、count、timestamp、operator_label | 原始文件内容、凭据、账户、持仓、委托、成交 |
| `evidence_storage_path` | 是 | 本地 process 路径或 `manual-only` | NAS 路径、env、runtime output |
| `redaction_owner` | 是 | `user` / `external-operator` / `agent-after-gate` | 当前不允许 agent 从真实输出做脱敏 |

### G. Runtime Safety

| 字段 | 必填 | 允许值 / 格式 | 当前要求 |
|---|---|---|---|
| `dry_run_or_manual_review` | 是 | `manual-review` / `dry-run-after-gate` / `n/a-with-reason` | 缺失则不发起门禁 |
| `timeout_seconds` | 是 | 正整数；草案可填 `TBD` | 真实执行前必须具体化 |
| `on_failure` | 是 | `stop-only` / `rollback-then-stop` / `manual-escalation` | 缺失则 fail closed |
| `rollback_plan` | 是 | 文本；写类必须具体 | check-only 可为 `no-state-change-stop-only` |
| `max_output_policy` | 是 | `no-raw-output` / `redacted-summary-only` | 必须禁止原始 NAS 输出 |

## 可复制的用户输入模板

```text
CR102 future concrete execution authorization gate 输入：
gate_intent: draft_only
requested_operation_class: check-only
execution_window: manual-later
executor: user-manual

nas_label: <脱敏 NAS label>
nas_path_or_export: <withheld-offline 或用户显式路径>
package_exchange_root: <withheld-offline 或用户显式 root>
object_scope: <精确对象 / 包名 / 版本 / 子目录 label>
scope_exclusions: <禁止接触范围>

mount_required: false
local_mount_point: not-mounted
mount_check_authorized: false
network_probe_authorized: false

authorized_identity_label: <脱敏身份 label>
credential_source: user-held-not-shared
permission_read: false
permission_write: false
permission_delete: false

operation_allowlist: check
operation_denylist: access,list,read,copy,pull,write,publish,delete,mount

evidence_profile: redacted-status-only
allowed_evidence_fields: status,operation,nas_label,object_label,hash,count,timestamp,operator_label
evidence_storage_path: manual-only
redaction_owner: user

dry_run_or_manual_review: manual-review
timeout_seconds: TBD
on_failure: stop-only
rollback_plan: no-state-change-stop-only
max_output_policy: redacted-summary-only
```

## Fail-Closed 规则

| 条件 | 处理 |
|---|---|
| 用户只回复 `approve` 但未提供上述输入 | 只能批准清单存在；不发起具体执行门禁 |
| path/export/root/identity/object scope 任一缺失 | 不授权 `check` 或任何真实 NAS 动作 |
| operation_allowlist 与 denylist 冲突 | 按 denylist 处理，并要求重提 |
| 出现凭据、token、password、secret、账户、交易事实或原始日志 | 拒绝纳入 evidence，要求用户脱敏重提 |
| 要求 agent 探测 NAS、读取 env、检查 mount 或启动 runtime | 停止当前清单路径，另起更高风险授权门禁 |
| 要求 write/copy/publish/pull/delete | 拆分 high-risk gate，不复用 check-only 输入 |

## 下一步

用户已补齐本清单中的最小身份 / 凭据边界 / 执行机验证字段：

```yaml
execution_machine_mount_root: manual-execution-machine-side
authorized_identity_label: current-shell-user
credential_source: none-needed
pull_target_path_or_label: manual-execution-machine-verification
```

因此允许准备 `CR102-FUTURE-CONCRETE-EXECUTION-AUTHORIZATION-GATE-REVIEW.md` 草案。即便草案生成，也仍需新的人工 `approve`；在该 approve 前，真实 NAS/env/runtime/trading/publish 全部保持未授权。执行机侧 pull/readback 本轮标记为用户手工验证，不由 agent 访问执行机挂载路径。
