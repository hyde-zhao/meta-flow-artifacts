---
check_id: "CR102-NAS-REAL-PACKAGE-EXCHANGE-FILLED-AUTHORIZATION-TEMPLATE"
check_name: "CR102 NAS Real Package Exchange Filled Authorization Template"
type: "authorization-template-draft"
status: "READY_FOR_CONCRETE_EXECUTION_GATE_REVIEW_NO_EXECUTION_AUTHORIZED"
owner: "host-orchestrator"
created_at: "2026-06-21T08:12:42+08:00"
source_input_list: "process/checks/CR102-FUTURE-CONCRETE-EXECUTION-AUTHORIZATION-INPUT-LIST.md"
source_cr: "process/changes/CR-102-NAS-REAL-PACKAGE-EXCHANGE-VALIDATION-AUTHORIZATION-GATE-2026-06-20.md"
user_authorization_statement: "用户声明授予所需 NAS 权限，并要求尽可能多完成 NAS 包交换验证任务。"
execution_authorized_by_this_file: false
nas_execution_gate_required: true
credential_read_authorized: false
runtime_authorized: false
trading_authorized: false
provider_lake_catalog_publish_authorized: false
execution_machine_mount_root: "manual-execution-machine-side"
authorized_identity_label: "current-shell-user"
credential_source: "none-needed"
pull_target_path_or_label: "manual-execution-machine-verification"
---

# CR102 NAS Real Package Exchange 已预填授权模板

## 结论

用户已表达“授予需要的所有 NAS 权限，并尽可能多完成 NAS 包交换验证任务”。本文件据此把可由 host-orchestrator 预填的授权字段全部填好；仍需要用户补齐真实 NAS scope、mount/access、对象范围和少量风险策略后，才能生成下一道 `CR102-FUTURE-CONCRETE-EXECUTION-AUTHORIZATION-GATE-REVIEW.md` 人工执行门禁。

本文件本身不执行、不访问、不检查、不列取、不读取、不复制、不写入、不发布、不拉取、不删除、不挂载 NAS。

## 用户已确认输入回填

| 字段 | 回填值 | 来源 / 说明 |
|---|---|---|
| CR059 挂载规划 | confirmed | 用户确认当前本机 NAS 挂载路径采用 CR059 规划。 |
| `nas_label` | `quant-lab-primary` | 由 CR059 Primary Tier 推导，用作脱敏 evidence label。 |
| `nas_path_or_export` | `/mnt/quant-lab-primary` | CR059 Primary Tier 本机挂载根；尚未由 agent 检查。 |
| `package_exchange_root` | `/mnt/quant-lab-primary/broker/package-exchange` | CR059 明确规划为 package exchange。 |
| `staging_release_root` | `/mnt/quant-lab-hot/staging/release` | CR059 明确规划为 release package staging。 |
| `release_archive_root` | `/mnt/quant-lab-primary/archive/release-packages` | CR059 明确规划为 release package archive。 |
| `mount_required` | `false` | 用户说明 NAS 已在本机挂载。 |
| `local_mount_point` | `/mnt/quant-lab-primary` | 对应 Primary Tier；如执行涉及 hot tier，也需 `/mnt/quant-lab-hot`。 |
| `network_probe_authorized` | `false` | 已有本机挂载规划时默认不做网络探测。 |
| `object_scope` | `test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z` | 用户指定包名和版本；release id 由 host-orchestrator 生成。 |
| `source_package_path_or_label` | `/mnt/quant-lab-hot/staging/release/test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z` | CR059 只规划 staging root；本子路径是本次 CR102 建议值，执行前仍需门禁批准。 |
| `target_package_path_or_label` | `/mnt/quant-lab-primary/broker/package-exchange/packages/test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z` | CR059 只规划 exchange root；`packages/...` 是本次建议子布局。 |
| `publish_path_or_label` | `/mnt/quant-lab-primary/broker/package-exchange/releases/test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z` | 若本轮允许 publish/promote，使用该建议 release label；否则改 `n/a`。 |
| `execution_machine_mount_root` | `manual-execution-machine-side` | 用户确认执行机侧由手工验证处理，不向 agent 提供执行机挂载根。 |
| `pull_target_path_or_label` | `manual-execution-machine-verification` | 用户确认执行机 pull/readback 由手工验证，不由 agent 访问执行机挂载路径。 |
| `authorized_identity_label` | `current-shell-user` | 用户确认当前 shell 用户作为本机 NAS 操作审计 label；不读取真实账户。 |
| `credential_source` | `none-needed` | 用户确认 NAS 已挂载，不需要 agent 处理或读取凭据。 |

> 注意：上述回填只是下一道执行门禁草案输入；没有执行任何 NAS 操作。CR059 只定义根目录级规划，`packages/...` / `releases/...` 是 CR102 本次建议子布局，需在下一道门禁中由用户确认。

## 剩余需要用户确认的最小字段

| 字段 | 当前值 / 建议 | 为什么还需要确认 |
|---|---|---|
| `execution_machine_mount_root` | 已确认：`manual-execution-machine-side` | 执行机同样挂载网盘，但挂载根与研究机不同；本轮不向 agent 提供执行机侧根目录，执行机验证走手工 evidence。 |
| `pull_target_path_or_label` | 已确认：`manual-execution-machine-verification` | 执行机 pull/readback 由用户手工验证；agent 不访问执行机路径。 |
| `authorized_identity_label` | 已确认：`current-shell-user` | 只作为审计标签，表示由当前 shell 用户/当前操作身份执行 NAS 操作；不要求也不应该提供真实账号密码。 |
| `credential_source` | 已确认：`none-needed` | NAS 已挂载且不需要 agent 处理凭据；不得读取 `.env`、密码、token 或账户。 |
| `source_package_path_or_label` | 已按 CR059 staging 回填 | 如果真实包已经存在于研究机本地非 NAS 路径，而不是 `/mnt/quant-lab-hot/staging/release/...`，需要用户改成真实本地构建包路径；否则下一道门禁可授权创建测试包到该 staging source。 |
| `publish_path_or_label` | 已按 CR102 建议子布局回填 | 如果 publish 表示正式 current 指针、latest alias 或其他执行机消费入口，需要用户说明 publish 语义；否则保持本 release-id 子目录。 |

## `authorized_identity_label` 与 `credential_source` 的目的

| 字段 | 目的 | 推荐取值 | 不需要 / 不允许提供 |
|---|---|---|---|
| `authorized_identity_label` | 用于审计“哪个执行身份”被授权做 NAS 操作，写入 evidence 和回滚记录；避免后续无法区分 agent-assisted、用户手工、执行机手工验证。 | `current-shell-user`、`research-host-operator`、`execution-host-operator`、`external-operator` | 不需要真实系统用户名、NAS 用户名、账户名、密码或 token。 |
| `credential_source` | 用于证明凭据边界：agent 是否需要接触凭据、凭据是否已由系统挂载层处理、是否由用户线下持有。 | `none-needed`（NAS 已挂载，推荐）、`user-held-not-shared`、`external-operator` | 不允许把密码、token、secret、cookie、私钥、`.env` 内容贴到对话或 evidence。 |

## 已预填策略

| 维度 | 预填值 | 说明 |
|---|---|---|
| 执行目标 | 策略研究机到执行机的真实 NAS package exchange 验证 | 目标是尽可能多验证真实包交换链路，而不是启动 QMT runtime。 |
| 默认授权操作候选 | `access,check,list,read,copy,pull,write,publish` | 进入下一道 high-risk gate 候选；本文件不直接执行。 |
| 默认禁止操作 | `delete` | 删除不属于完成包交换的必要动作，默认禁止。 |
| mount 策略 | 用户确认后才允许 | 如果 NAS 已挂载，推荐使用本地 mount point；如果未挂载，需要用户提供 export、mount point 和是否授权挂载。 |
| 目录创建策略 | `create-if-missing-only` | 只创建本次包交换需要的最小目录；不创建示例目录。 |
| 覆盖策略 | `forbid-overwrite` | 默认不覆盖已有正式包；如需覆盖，改为 `overwrite-with-backup`。 |
| 写入策略 | `temp-file-then-rename` | 减少半写入包污染。 |
| 回滚策略 | `rollback-then-stop` | 清理临时文件；不删除既有正式包。 |
| evidence 策略 | `redacted-hash-count` | 只保留 status/hash/count/timestamp/label，不保留原始路径、日志、包内容或凭据。 |
| 非 NAS 边界 | 仍未授权 | 不读取凭据/env/账户，不启动 QMT/MiniQMT/XtQuant/gateway，不交易，不 provider/lake/catalog publish。 |

## 可直接复制并补全的执行授权输入

```text
CR102 future concrete execution authorization gate 输入：

gate_intent: authorize_write_publish
requested_operation_class: write-publish
execution_window: after-user-approval
executor: agent-assisted-after-gate

nas_label: quant-lab-primary
nas_label_建议: 已按用户确认的 CR059 Primary Tier 规划回填。

nas_path_or_export: /mnt/quant-lab-primary
nas_path_or_export_建议: 用户已确认当前本机 NAS 挂载路径采用 CR059 规划；本字段使用 Primary Tier 根。

package_exchange_root: /mnt/quant-lab-primary/broker/package-exchange
package_exchange_root_建议: CR059 明确规划 `broker/package-exchange` 为 package exchange。

object_scope: test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z
object_scope_建议: 用户指定 `test_strategy_package/版本v0.1`，release id 由 host-orchestrator 生成。

scope_exclusions: credentials,env,accounts,orders,trades,raw_logs,unrelated_projects,trash,backups_not_created_by_this_run

mount_required: false
mount_required_建议: 用户说明 NAS 已在本机挂载；本轮不需要 agent 挂载。

local_mount_point: /mnt/quant-lab-primary
local_mount_point_建议: Primary Tier mount point；如果执行涉及 staging source，还需要使用 `/mnt/quant-lab-hot`。

mount_check_authorized: true-after-gate
network_probe_authorized: false
network_probe_authorized_建议: 已确认采用本机挂载路径时，不需要网络探测。

authorized_identity_label: current-shell-user
authorized_identity_label_建议: current-shell-user；该字段只用于审计授权身份 label，不要求真实账户名、密码、token 或 NAS 用户名原文。

credential_source: none-needed
credential_source_建议: 如果系统已挂载且无需 agent 处理凭据，填 none-needed；如果凭据由你线下手工持有，填 user-held-not-shared；不要把凭据贴到对话。

permission_read: true-after-gate
permission_write: true-high-risk-gate
permission_delete: false

operation_allowlist: access,check,list,read,copy,pull,write,publish
operation_denylist: delete

directory_create_authorized: true-high-risk-gate
directory_create_policy: create-if-missing-only
directory_create_scope: /mnt/quant-lab-primary/broker/package-exchange/packages/test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z
directory_create_scope_建议: 仅限本次 package exchange target 子目录；不创建示例目录。

source_package_path_or_label: /mnt/quant-lab-hot/staging/release/test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z
source_package_path_or_label_建议: CR059 没有规划研究机本地 build 输出路径；这里采用 CR059 `staging/release` 作为 NAS staging source 建议值。

target_package_path_or_label: /mnt/quant-lab-primary/broker/package-exchange/packages/test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z
target_package_path_or_label_建议: 基于 CR059 `broker/package-exchange` 根生成本次 target 子路径。

publish_path_or_label: /mnt/quant-lab-primary/broker/package-exchange/releases/test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z
publish_path_or_label_建议: 若本轮不做 publish/promote，可在下一道门禁中改为 `n/a`。

execution_machine_mount_root: manual-execution-machine-side
execution_machine_mount_root_说明: 执行机侧挂载根不提供给 agent；执行机 pull/readback 由用户手工验证。

pull_target_path_or_label: manual-execution-machine-verification
pull_target_path_or_label_建议: 执行机挂载根与研究机不同，且用户选择手工验证；本轮 agent 不访问执行机路径。

overwrite_policy: forbid-overwrite
overwrite_policy_可选: overwrite-with-backup

atomic_write_policy: temp-file-then-rename
backup_policy: required-before-overwrite
rollback_plan: rollback-then-stop；删除本次产生的 temp 文件；如发生覆盖前已创建 backup，则保留 backup 并停止；不删除既有正式包。
on_failure: rollback-then-stop
timeout_seconds: 300

evidence_profile: redacted-hash-count
allowed_evidence_fields: status,operation,nas_label,object_label,hash,count,bytes,timestamp,operator_label,rollback_status
evidence_storage_path: process/checks/CR102-NAS-REAL-PACKAGE-EXCHANGE-EVIDENCE.md
redaction_owner: agent-after-gate
max_output_policy: redacted-summary-only

dry_run_or_manual_review: manual-review

我确认：本授权仅限上述 NAS 包交换范围；授权进入下一道具体执行门禁草案，不授权读取凭据/env/账户，不授权启动 QMT/MiniQMT/XtQuant/gateway runtime，不授权交易，不授权 provider/lake/catalog publish，不授权 delete。下一道门禁 approve 前仍不执行。
```

## 字段完成状态

| 字段 | 状态 | 建议 / 说明 |
|---|---|---|
| `nas_label` | 已回填 | `quant-lab-primary`。 |
| `nas_path_or_export` | 已回填 | `/mnt/quant-lab-primary`；本轮未检查真实 NAS。 |
| `package_exchange_root` | 已回填 | `/mnt/quant-lab-primary/broker/package-exchange`。 |
| `object_scope` | 已回填 | `test_strategy_package/v0.1/cr102-test-strategy-package-v0.1-20260621T003214Z`。 |
| `mount_required` / `local_mount_point` | 已回填 | `false` + `/mnt/quant-lab-primary`；hot staging 使用 `/mnt/quant-lab-hot`。 |
| `network_probe_authorized` | 已回填 | `false`。 |
| `source_package_path_or_label` | 已回填，需按真实包来源确认 | 当前采用 CR059 staging release；若真实包在研究机本地非 NAS 路径，执行门禁前需改成真实路径。 |
| `target_package_path_or_label` | 已回填 | 基于 CR059 package exchange root 的 `packages/...` 子布局。 |
| `publish_path_or_label` | 已回填，需按 publish 语义确认 | 默认 release-id 子目录；如需 latest/current 指针，需要另行授权。 |
| `directory_create_scope` | 已回填 | 限定到本次 target 子目录。 |
| `authorized_identity_label` | 已确认 | `current-shell-user`。 |
| `credential_source` | 已确认 | `none-needed`。 |
| `execution_machine_mount_root` / `pull_target_path_or_label` | 已确认 | `manual-execution-machine-side` / `manual-execution-machine-verification`；执行机侧由用户手工验证，agent 不访问执行机路径。 |

## 我可以在下一道门禁批准后完成的 NAS 验证任务

| 任务 | 是否可由 agent 执行 | 前置输入 | 证据 |
|---|---|---|---|
| mount/access precheck | 可执行，若用户授权 | path/export、mount_required、local_mount_point | redacted pass/fail |
| 目标 scope bounded list | 可执行，若用户授权 | package_exchange_root、object_scope、最大输出策略 | count / object labels |
| 源包存在性与 hash | 可执行，若用户授权 | source_package_path_or_label | hash / bytes |
| 目标目录 create-if-missing | 可执行，若用户授权 | directory_create_scope | created/existed |
| copy/write 到 staging | 可执行，若用户授权 | source、target、overwrite、atomic write | hash/count/status |
| publish / promote | 可执行，若用户授权 | publish_path_or_label、rollback | publish status |
| pull/readback 校验 | 可执行，若用户授权 | pull target 或 target package label | hash match |
| rollback on failure | 可执行，若用户授权 | rollback_plan | rollback_status |

## 需要用户手工执行或确认的内容

| 内容 | 什么时候需要 | 建议 |
|---|---|---|
| NAS 凭据处理 | 如果 NAS 未挂载或需要登录 | 你在线下完成挂载或凭据配置，不把凭据发给我。 |
| 真实路径确认 | 永远需要 | 你提供 path/export/root 或脱敏 label + 本地可访问 mount path。 |
| 覆盖风险确认 | 如果目标已存在同名包 | 默认不覆盖；如必须覆盖，明确 `overwrite-with-backup`。 |
| publish 语义确认 | 如果 publish 表示 current 指针或正式 release | 明确 publish_path_or_label 和回滚方式。 |
| 执行机消费路径确认 | 如果要证明执行机可 pull/readback | 给 pull_target_path_or_label 或说明由你手工验证。 |

## Fail-Closed 规则

- 缺 `nas_path_or_export` 或 `package_exchange_root`：不执行。
- 缺 `object_scope`：不 list/read/copy/write。
- 缺 `source_package_path_or_label` 或 `target_package_path_or_label`：不交换包。
- `delete` 默认禁止，即使用户授予“所有 NAS 权限”，也不作为包交换必要动作执行。
- 出现凭据、token、password、secret、账户、交易事实或原始日志：拒绝写入 evidence，要求脱敏。
- 下一道具体执行门禁未 approve 前，不执行任何真实 NAS 动作。
