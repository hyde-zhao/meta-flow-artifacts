---
cr_id: "CR-089"
status: "closed-superseded-by-successor-crs"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "runtime"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 QMT 运行级、NAS package exchange、策略包交付、接口验证、权限与凭据边界；不得使用 fast-lane。"
rollback_to: "CR046 CP6 framework-first recovery point"
approval_result: "closed-by-user-20260623 / absorbed-by-CR102-CR103-CR104"
created_at: "2026-06-17T21:20:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-17T21:34:53+08:00"
closed_by: "user"
closed_at: "2026-06-23T17:15:44+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-046"
source_checkpoint: "current conversation"
source_decision_id: "USER-20260617-QMT-NAS-STRATEGY-PACKAGE-INTERFACE-VALIDATION"
follow_up_type: "runtime-authorization-gate"
risk_class: "trading-runtime-boundary"
owner: "host-orchestrator"
revisit_condition: "QMT direct-run、order-write、simulation/live、账户/日志原文、额外 NAS 操作或 provider/lake/catalog publish 需要独立后续门禁。"
acceptance_criteria: "形成 QMT 接口验证的策略包输出规范、NAS package exchange 交付位置、交易主机取包校验流程、只读接口 smoke 边界和待人工决策清单；不执行真实 NAS / QMT / 凭据 / 账户动作。"
close_condition: "CR089 原始范围已由 CR102 NAS package exchange、CR103 qmt_interface_smoke package/checker 非 runtime 验证、CR104 MiniQMT/gateway readonly runtime smoke 吸收并关闭；用户确认关闭 CR089。"
cr_index_path: "process/changes/CR-INDEX.yaml"
summary_ref: "process/changes/summaries/CR-089.summary.json"
evidence_index_ref: "process/archive/CR-089/evidence-index.json"
---

# CR089 QMT Interface Validation Gate

## 变更描述

用户确认继续规划 QMT 接口验证，并补充“根据之前协议，QMT 运行级可以直接从 NAS 上获取可以运行的策略”。本 CR 草案将 QMT 接口验证从 CR046 framework-first 范围中拆出，作为独立运行授权门禁候选。

本轮规划目标：

1. 定义研究侧如何输出可运行策略包。
2. 定义策略包 manifest、目录、校验、状态机和授权边界。
3. 定义 NAS `STRATEGY_PACKAGE_EXCHANGE_ROOT` 的交付位置和只读消费规则。
4. 定义交易主机从 NAS 获取策略包、校验、复制到本地缓存再运行的推荐流程。
5. 定义首个 `qmt_interface_smoke` 策略包，只用于 package discovery、manifest/sha256 校验和 QMT 只读接口 smoke。

本 CR 已于 2026-06-23T17:15:44+08:00 按用户确认关闭为 `closed-superseded-by-successor-crs / ready_with_risk`。CR089 原始范围已由后续正式 CR 吸收：CR102 覆盖真实 NAS package exchange，CR103 覆盖 `qmt_interface_smoke` 本地包 / checker / 非 runtime 合同，CR104 覆盖 MiniQMT/gateway `query_positions_readonly` runtime smoke 和脱敏 evidence。剩余 QMT direct-run、order-write、simulation/live、账户 / 日志原文、额外 NAS 操作或 provider/lake/catalog publish 不属于 CR089 关闭范围，必须另起独立门禁。

## Closure 覆盖判定

| CR089 原始范围 | 后续覆盖 CR | 覆盖证据 | 关闭判断 |
|---|---|---|---|
| 策略包输出规范、manifest、checksum、target docs | CR103 | `process/checks/CP7-CR103-QMT-MINIQMT-NON-TRADING-DAY-VALIDATION.md`；`125 passed`；`qmt_interface_smoke` checker PASS | 覆盖 |
| NAS package exchange 与交易主机 readback | CR102 | `process/checks/CP8-CR102-DELIVERY-READINESS.md`；`runs/RUN-EXEC-20260621-003.md` 逐文件 sha256 / bytes PASS | 覆盖 |
| `query_positions` 只读 runtime smoke | CR104 | `process/checks/CP7-CR104-RUNTIME-READONLY-SMOKE-PASS-2026-06-21.md`；zero 与 one_to_ten 持仓 bucket 均 PASS | 覆盖 |
| 脱敏 evidence 与 forbidden counters | CR104 | redacted evidence checker PASS；forbidden counters 全 0；`raw_payload_emitted=false` | 覆盖 |
| QMT direct-run | 未覆盖 | CR104 CP8 明确 deferred | 后续独立 gate |
| order-write / simulation / live | 未覆盖 | CR104 CP8 明确转 proposed CR105 / FU-CR101-001 | 后续 high-risk gate |

Closure 不授权任何新的 NAS、QMT/MiniQMT/XtQuant/gateway、凭据、账户原文、submit/cancel、simulation/live 或 provider/lake/catalog 动作。

## 冲突预检结论

| 对象 | 当前状态 | 与 CR089 的关系 | 结论 |
|---|---|---|---|
| `CR-046` | `active-cp6-pass-ready-for-verification`，用户暂停在 CP7 前 | 定义双目标策略交付框架、策略包合同、QMT / MiniQMT target 和验证框架 | 重叠，必须人工选择合并 / 独立阻塞 / 拆分 |
| `CR-020` | `deleted-by-user` | 历史 QMT gateway 只读接口实现和 runbook 可作为审计与代码参考 | 不得恢复为当前验证入口 |
| `CR-051` | closed-current Git 内交付 | 定义研究 PC、NAS、交易 PC 和 package exchange 合同 | 可复用，不授权真实 transfer/import/runtime |
| `QMT-INTERFACE-VALIDATION-CR-CONTEXT` | ready | 已明确真实接口验证应新建独立 CR | 作为本 CR 输入 |

可选处理：

| 选项 | 含义 | 推荐度 |
|---|---|---:|
| A. 合并到 CR046 恢复 | 先恢复 CR046 CP7，扩展 CR046 CP8 后再启动 CR047/CR048/CR089 | 中 |
| B. CR089 独立阻塞启动 | 保持 CR046 active 不变，把 CR089 作为 blocked formal CR，等待用户确认可并行或后续解锁 | 高 |
| C. 拆分无冲突子集 | 只落策略包 / NAS 交付规范，不执行 QMT runtime；runtime smoke 后置 | 高 |
| D. supersede 部分 CR046 follow-up | 明确 CR089 替代 CR048/CR049 某些运行验证候选 | 低，需更大设计回退 |

当前文件采用 B + C 的保守形态：正式记录 CR089 草案和设计边界，但不设置 active，不执行 runtime。用户 2026-06-17T21:34:53+08:00 回复“同意”后，本 CR 的 CP2/CP3/CP5 formal review 已 approved；该同意只接受门禁推荐方案，不授权真实 NAS / QMT / 凭据 / 账户动作。

## Startup 冲突决策回填

| 字段 | 内容 |
|---|---|
| 决策时间 | 2026-06-17T21:08:59+08:00 |
| 用户输入 | `同意` |
| 解释 | 接受 DQ-CR089-01..04 的推荐方案：CR089 独立 blocked start；不从 NAS 原地执行策略；本次接口验证限定为 `query_positions` 只读 smoke；agent 不读取 `.env`、NAS 凭据或 QMT 账号。 |
| 明确不授权 | 不访问 NAS 内容，不读取凭据，不启动 QMT/MiniQMT/XtQuant/gateway，不查询账户原文，不 submit/cancel，不 simulation/live，不恢复 CR020，不推进 CR046。 |
| 后续门禁 | `process/checkpoints/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md`、`process/checkpoints/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-REVIEW.md`、`process/checkpoints/CP5-CR089-QMT-INTERFACE-VALIDATION-READINESS.md` |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/qmt/CR046-DUAL-TARGET-STRATEGY-FRAMEWORK.md` | 不变 | CR046 仍是 framework-first 基线 | 不适用 | approved |
| `docs/research/HOST-WORKFLOW.md` | 不变 | CR051 的 package exchange 合同保留 | 不适用 | approved |
| `docs/research/ARCHIVE-GOVERNANCE.md` | 不变 | CR051 的 storage domain 合同保留 | 不适用 | approved |
| `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | 新增 | 以 CR046/CR051 为上游基线，不替换 | 文档头部修订记录 | approved |
| `process/checks/CR089-STARTUP-CONFLICT-PRECHECK-2026-06-17.md` | 新增 | 记录 CR089 与 CR046/CR020/CR051 的冲突预检 | 文件头部 frontmatter | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR046 StrategyPackageContract | CR089 NAS package exchange runtime intake | 原文保留 | CR089 只扩展运行级取包与只读 smoke 入口，不修改 CR046 合同事实 |
| CR051 HOST-WORKFLOW | CR089 trading_pc package intake | 原文保留 | CR089 将 blocked 的 package exchange -> trading_pc 流程转为待授权验证设计 |
| CR020 readonly query_positions 实现 | CR089 qmt_interface_smoke 只读接口验证 | 审计复用 | CR020 不恢复，只复用代码和 runbook 的安全边界 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增或重定义需求 | QMT 接口验证、策略包取包方式 | true | 新增 CR089 需求草案：NAS package exchange + 只读 smoke |
| 场景层 | 是否改变测试矩阵覆盖范围 | package discovery、checksum、manifest、readonly query | true | 后续 CP2/CP3/CP5 需补场景和测试矩阵 |
| 计划层 | 是否改变 Story / 任务依赖 | CR046 / CR047 / CR048 / CR049 / CR089 | true | 先做冲突决策，再决定并行、等待或合并 |
| 安全层 | 是否引入高风险动作 | NAS、QMT runtime、凭据、账户只读接口 | true | 默认全部 blocked，逐 run 授权后仅允许脱敏只读 smoke |
| 交付层 | 是否需要新交付物 | 策略包规范、NAS 交付位置、交易主机取包流程 | true | 新增 CR089 交付规划文档和预检证据 |

## 回退决策

- 影响范围：局部但高风险，限定为 QMT 接口验证与策略包交付路径。
- 回退到阶段：`CR046 CP6 framework-first recovery point`。
- 回退条件：用户不接受 CR089 独立启动；或要求先完成 CR046 CP7/CP8；或 QMT/MiniQMT 权限不可用。
- 回退动作：保留本 CR 草案为审计，不启动 CP2/CP3/CP5，不执行 NAS/QMT/凭据/账户动作。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量文档 | false | 涉及运行级和交易接口验证 |
| 修改架构、权限、安全边界或平台路径 | true | NAS exchange、trading local cache、QMT readonly smoke |
| 修改外部接口契约或多 Story 依赖 | true | QMT runtime CLI、strategy package、交易主机取包 |
| 需要 HLD / LLD 才能解释影响 | true | 需要区分策略包、接口验证、运行授权 |
| 是否保持 fast-lane | false | 必须 standard |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR089-QMT-INTERFACE-VALIDATION-BATCH-A`
- 批次范围来源：CR089 影响分析
- 批次内 Story：
  - `CR089-S01-package-exchange-contract`
  - `CR089-S02-qmt-interface-smoke-strategy-package`
  - `CR089-S03-trading-pc-package-intake`
  - `CR089-S04-readonly-qmt-interface-smoke`
- 批次人工确认稿：`process/checkpoints/CP5-CR089-QMT-INTERFACE-VALIDATION-BATCH-A.md`
- 开发启动条件：
  - [x] 用户选择冲突处理方案
  - [x] CP2 / CP3 明确批准 CR089 独立或合并路线
  - [ ] 批次内全部 Story 设计证据输出
  - [ ] 批次 CP5 自动预检通过
  - [ ] 批次 CP5 人工确认 approved

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | host-orchestrator | 冲突决策 | CR046 / CR051 / CR020 / 本 CR | 决策项 | 用户确认 | CP2 intake 或合并回 CR046 |
| 2 | meta-se | 设计 CR089 HLD/Story | CR089 草案 | HLD / Story Plan | CP3 / CP4 | LLD 批次 |
| 3 | meta-dev | 实现策略包规范与离线校验 | CP5 approved | docs/schema/fixture/checks | CP6 | 交给 QA |
| 4 | meta-qa | 验证 package contract 和只读 smoke evidence 模板 | CP6 产物 | CP7 证据 | CP7 | CP8 readiness |
| 5 | 用户手工执行 | 在 Windows / Linux 手工执行 QMT 只读 smoke | per-run 授权 | 脱敏摘要 | runtime authorization | 回填证据 |
| 6 | host-orchestrator | 收敛终验 | QA 结果 / 用户脱敏摘要 | CP8 | 用户确认 | 关闭或回修 |

## 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CR089-01 | scope | CR089 与 CR046 如何处理重叠？ | CR089 先作为 blocked formal CR 记录，用户确认后独立启动；CR046 保持暂停 | 合并到 CR046 恢复；等待 CR046 CP8 后启动；取消 CR089 | 推荐方案能保留运行验证路线但不破坏 active lock | 若误设 active 会造成状态冲突 | 用户明确选择合并或等待时切换 |
| DQ-CR089-02 | architecture | QMT 运行级是否直接从 NAS 原地执行策略？ | 不原地执行；从 NAS 只读拉取 approved package，校验后复制到交易主机本地不可变缓存再运行 | NAS 原地执行；人工 U 盘拷贝；Git release | 推荐方案减少网络抖动和半写入风险 | 需要交易主机本地缓存规范 | 临时 smoke 可允许手工原地读取但不作为默认 |
| DQ-CR089-03 | runtime_authorization | 本次接口验证允许到哪一级？ | 只允许 `query_positions` 只读 smoke，scope=`qmt:positions:read`，仅用户手工执行并回填脱敏摘要 | 允许 health/capabilities；允许 submit/cancel；允许 simulation | 推荐方案权限最小；submit/cancel 风险高 | 账户、凭据、持仓原文泄露风险 | 需要下单或模拟盘时另起高风险 gate |
| DQ-CR089-04 | security | 是否允许 agent 读取 `.env`、NAS 凭据或 QMT 账号？ | 不允许，真实值仅用户本机保存 | 允许读取脱敏 `.env`；允许 agent 代跑命令 | 推荐方案符合最小权限 | 降低自动化程度 | 用户另行授权且指定脱敏策略时再评审 |

## 不授权项

- 不访问 NAS 内容、不挂载 NAS、不复制真实策略包。
- 不读取 `.env`、NAS 凭据、QMT 账号、HMAC secret、token、session、cookie、交易密码。
- 不启动 QMT / MiniQMT / XtQuant / gateway。
- 不查询真实账户、资金、持仓原文、委托、成交。
- 不 submit/cancel，不 simulation/live。
- 不把本 CR 草案解释为 QMT-ready、trade-ready 或 runtime_verified。

## 处理结论

- 审批结论：`cp2-cp3-cp5-approved / no-runtime-authorization`
- [ ] 自动批准（低风险）
- [ ] 待人工确认（中风险）
- [x] 待人工审批（高风险）- 已于 2026-06-17T21:34:53+08:00 approve；仅批准 CP2/CP3/CP5 readiness，不授权 runtime

## 本地离线骨架实现结果

| 字段 | 内容 |
|---|---|
| 实现时间 | 2026-06-17T22:03:03+08:00 |
| 实现状态 | `local-offline-package-ready` |
| package root | `packages/qmt_interface_smoke/0.1.0` |
| manifest | `packages/qmt_interface_smoke/0.1.0/manifest.yaml` |
| checksum | `packages/qmt_interface_smoke/0.1.0/checksums/SHA256SUMS` |
| 人工 intake checklist | `packages/qmt_interface_smoke/0.1.0/validation/offline-intake-checklist.md` |
| 脱敏证据模板 | `packages/qmt_interface_smoke/0.1.0/evidence/redacted-smoke-result-template.yaml` |
| 本地 checker | `scripts/check_cr089_qmt_interface_smoke_package.py` |
| checker tests | `tests/test_cr089_qmt_interface_smoke_package.py` |
| 实现证据 | `process/stories/CR089-QMT-INTERFACE-SMOKE-PACKAGE-LOCAL-IMPLEMENTATION.md` |
| 恢复上下文 | `process/context/CP6-CR089-QMT-INTERFACE-SMOKE-PACKAGE-CONTEXT.yaml` |

本实现只完成本地离线策略包骨架、manifest、checksum、目标说明、人工 intake checklist、脱敏证据模板和本地文件级 checker。它不表示 CR089 已 active，不表示 formal CP6 runtime PASS，不表示 QMT 接口已验证，也不授权 NAS publish/pull/list/copy/delete、凭据读取、QMT/MiniQMT/XtQuant/gateway 启动、账户原文查询、submit/cancel、simulation 或 live。

## Runtime Smoke 授权记录

| 字段 | 内容 |
|---|---|
| 授权时间 | 2026-06-17T22:40:30+08:00 |
| 授权记录 | `runs/RUN-EXEC-20260617-002.md` |
| 授权范围 | 仅允许用户本人在交易主机手工执行 `qmt_interface_smoke` 的 `query_positions` 只读验证 |
| required scope | `qmt:positions:read` |
| agent 权限 | 不授权 agent 读取 `.env`、账号、密码、token、session 或 QMT 凭据；不授权 agent 代跑 runtime |
| 输出限制 | 不授权输出账户、持仓、资金、委托、成交或日志原文；只允许回填脱敏摘要 |
| 交易写入 | 不授权 `submit_order`、`cancel_order`、simulation、live |
| CR 状态影响 | 不激活 CR089，不恢复 CR046 CP7，不恢复 CR020；仅登记本次手工 runtime smoke 许可 |

## Runtime Smoke server diagnostics 记录

| 字段 | 内容 |
|---|---|
| 执行记录 | `runs/RUN-EXEC-20260617-003.md` |
| 用户执行命令 | `uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli server-diagnostics --env-file .env --host 127.0.0.1 --port 18765 --runtime-authorization-ref cr089-runtime-smoke-20260617-query-positions` |
| 诊断结果 | PASS：`status=ok`、`schema_version=cr020-runtime-manual-validation-v1`、`secrets_redacted=true` |
| 配置摘要 | `host=127.0.0.1`、`port=18765`、`allowed_source=127.0.0.1/32`、`runtime_authorization_ref=cr089-runtime-smoke-20260617-query-positions` |
| 脱敏摘要 | `client_secret_ref=[REDACTED]`；`client_id_hash` 与 `account_ref` 具体哈希值未落库；`miniqmt_path_configured=true`；`xtquant_site_packages_configured=true` |
| 结论边界 | 仅证明 `.env` 配置加载与公开诊断脱敏通过；不证明 gateway 已启动，不证明 XtQuant / MiniQMT 已连接，不证明 `query_positions` 已通过 |
| CR 状态影响 | 不激活 CR089，不恢复 CR046 CP7，不恢复 CR020 |

## Runtime Smoke gateway serve health 记录

| 字段 | 内容 |
|---|---|
| 执行记录 | `runs/RUN-EXEC-20260617-004.md` |
| 用户执行命令 | `uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli serve --env-file .env --host 127.0.0.1 --port 18765 --runtime-authorization-ref cr089-runtime-smoke-20260617-query-positions` |
| 初始 health 结果 | PASS：`status=starting`，`health.status=ok`、`health.session_ready=true`、`health.session_state=ready`、`health.blocked_reason` 为空、`health.runtime_status=xtquant-ready` |
| 配置摘要 | `host=127.0.0.1`、`port=18765`、`allowed_source=127.0.0.1/32`、`runtime_authorization_ref=cr089-runtime-smoke-20260617-query-positions` |
| 脱敏摘要 | `client_secret_ref=[REDACTED]`；`client_id_hash` 与 `account_ref` 具体哈希值未落库；`miniqmt_path_configured=true`；`xtquant_site_packages_configured=true` |
| 结论边界 | 仅证明 gateway `serve` 初始 health 和 QMT / MiniQMT session ready gate 通过；不证明 `query_positions` 已通过 |
| CR 状态影响 | 不激活 CR089，不恢复 CR046 CP7，不恢复 CR020 |

## Runtime Smoke query_positions 记录

| 字段 | 内容 |
|---|---|
| 执行记录 | `runs/RUN-EXEC-20260617-005.md` |
| 用户执行命令 | `uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli client-diagnostics --env-file .env --base-url http://127.0.0.1:18765`；`uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli query-positions --env-file .env --base-url http://127.0.0.1:18765 --run-id cr089-query-positions-20260617-001 --request-id cr089-query-positions-20260617-001 --timeout-seconds 10` |
| Client diagnostics | PASS：`status=ok`、`base_url=http://127.0.0.1:18765`、`client_secret_ref=[REDACTED]`；`client_id_hash` 具体值未落库 |
| Query 结果 | PASS：client `status=ok`、`endpoint=positions`、`reason_code` 为空、`blocked_result=null`；gateway `allowed=true`、`blocked=false`、`transport_status=allowed`、`status_code=200` |
| 授权与 scope | `authorization_ref=cr089-runtime-smoke-20260617-query-positions`、`scope=qmt:positions:read`、`readonly_query_authorized=true`、`operation_authorized=false`、`real_operation=false` |
| 脱敏摘要 | `position_count_bucket=zero`、`positions_digest=positions:e025dfde5d22f9d1`、`redaction_status=pass`、`raw_payload_emitted=false` |
| 安全计数 | `query_positions_read_attempt=1`、`readonly_positions_adapter_call=1`、`raw_positions_emit=0`、`redaction_fallback_to_raw=0`、`real_order=0`、`real_cancel=0`、`account_write=0`、`provider_fetch=0`、`lake_write=0`、`publish=0`、`simulation_or_live_run=0` |
| 不落库字段 | 用户输出中的 `client_id_hash`、`client_id_ref`、`nonce_ref`、`signature_ref` 具体值不保存 |
| 结论边界 | 仅证明 `query_positions` 只读 smoke 在本次授权范围内通过；不授权账户 / 持仓 / 资金 / 委托 / 成交 / 日志原文输出，不授权 submit / cancel / simulation / live |
| CR 状态影响 | 不激活 CR089，不恢复 CR046 CP7，不恢复 CR020 |

## Runtime Smoke redacted collector 记录

| 字段 | 内容 |
|---|---|
| 执行记录 | `runs/RUN-EXEC-20260618-001.md` |
| 用户执行命令 | `uv run --python 3.11 python scripts/collect_cr089_qmt_runtime_smoke_summary.py --env-file .env --base-url http://127.0.0.1:18765 --runtime-authorization-ref cr089-runtime-smoke-20260617-query-positions --run-id cr089-query-positions-20260617-002 --request-id cr089-query-positions-20260617-002 --timeout-seconds 10 --output-json cr089-redacted-runtime-smoke-summary.json` |
| Collector 结果 | PASS：`status=pass`、schema `cr089-qmt-runtime-smoke-redacted-summary-v1` |
| Server health | PASS：`status=ok`、`session_ready=true`、`session_state=ready`、`blocked_reason` 为空、`runtime_status=xtquant-ready`、`redaction_status=redacted` |
| Local config summary | `client_id_configured=true`、`client_secret_configured=true`、`account_ref_configured=true`、`miniqmt_path_configured=true`、`xtquant_site_packages_configured=true`、`allowed_source=127.0.0.1/32`、`account_type=STOCK` |
| Query 结果 | PASS：`readonly_smoke.status=pass`、`transport_status=allowed`、`status_code=200`、`required_scope=qmt:positions:read`、`readonly_query_authorized=true`、`operation_authorized=false`、`real_operation=false` |
| 脱敏摘要 | `position_count_bucket=zero`、`positions_digest=positions:e025dfde5d22f9d1`、`redaction_status=pass`、`raw_payload_included=false`、`raw_account_output_included=false`、`items_redacted_count=0` |
| 安全计数 | NAS、凭据、原始账户、原始持仓、下单、撤单、simulation、live、provider、lake、publish、gateway/QMT 启动类 forbidden counters 均为 0 |
| Redaction assurance | `client_or_account_refs_included=false`、`nonce_or_signature_refs_included=false`、`qmt_logs_included=false`、`raw_account_output_included=false`、`raw_payload_included=false` |
| 结论边界 | collector 可作为本轮只读 smoke 的首选回填格式；仍不授权账户 / 持仓 / 资金 / 委托 / 成交 / 日志原文输出，不授权 submit / cancel / simulation / live |
| CR 状态影响 | 不激活 CR089，不恢复 CR046 CP7，不恢复 CR020 |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| 上游 CR | CR-046 | 双目标策略交付框架，当前 active/paused |
| 上游 CR | CR-051 | 研究主机 / NAS / 交易主机 package exchange 合同 |
| 历史 CR | CR-020 | deleted-by-user；只可作为审计和代码参考 |
| 上下文 | `process/context/QMT-INTERFACE-VALIDATION-CR-CONTEXT-2026-06-17.md` | 指定真实接口验证应新建独立 CR |
| 规划文档 | `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | 策略输出、规范与交付位置 |
| 本地 package | `packages/qmt_interface_smoke/0.1.0` | CR089 批准范围内的本地离线策略包骨架 |
| 实现证据 | `process/stories/CR089-QMT-INTERFACE-SMOKE-PACKAGE-LOCAL-IMPLEMENTATION.md` | 本地离线实现范围、验证和剩余风险 |
| 恢复上下文 | `process/context/CP6-CR089-QMT-INTERFACE-SMOKE-PACKAGE-CONTEXT.yaml` | 清上下文后的最小恢复入口 |
| 执行反馈 | `runs/RUN-EXEC-20260617-001.md` | 交易主机离线 intake PASS |
| Runtime 授权 | `runs/RUN-EXEC-20260617-002.md` | 用户手工 `query_positions` 只读 smoke 授权 |
| Runtime 诊断 | `runs/RUN-EXEC-20260617-003.md` | 交易主机 `server-diagnostics` PASS；尚未启动 gateway / query |
| Runtime Gateway | `runs/RUN-EXEC-20260617-004.md` | 交易主机 gateway `serve` 初始 health PASS；尚未执行 query |
| Runtime Query | `runs/RUN-EXEC-20260617-005.md` | 交易主机 `query_positions` 只读 smoke PASS；只保存脱敏摘要 |
| Runtime Collector | `runs/RUN-EXEC-20260618-001.md` | 交易主机 redacted collector PASS；脚本输出可作为后续回填首选格式 |
