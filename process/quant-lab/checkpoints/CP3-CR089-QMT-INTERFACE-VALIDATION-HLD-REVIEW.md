---
checkpoint_id: "CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-REVIEW"
checkpoint_name: "CR089 QMT Interface Validation HLD Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T21:08:59+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T21:34:53+08:00"
auto_check_result: "process/checks/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml"
    - "docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md"
---

# CP3 CR089 QMT Interface Validation HLD Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-CONSISTENCY.md` | PASS | 0 | NAS package exchange、strategy package structure、trading_pc local cache 和 readonly smoke 边界已定义。 |

## Decision Brief

推荐决策：批准 CR089 CP3 轻量架构：NAS 是 package exchange，不是默认执行根；策略包按 `strategy-package-<strategy_id>-<version>` 输出 zip、sha256、manifest、docs 和 validation；交易主机只读拉取 approved package，校验后复制到 `${TRADING_PACKAGE_CACHE_ROOT}` 的不可变缓存，再通过 `active/<strategy_id>` 指向当前版本；首个验证策略为 `qmt_interface_smoke`。

备选方案：NAS 原地执行；人工 U 盘传输；Git release 作为策略包分发；直接在交易主机手工放脚本。推荐方案优先，因为它兼顾可审计、可回退、避免半写入和网络抖动风险。

影响维度：package manifest/index、NAS 目录、交易主机缓存、approval/quarantine、readonly smoke 证据、后续 CP5 执行边界。

风险与回退：如果 package 校验失败或 manifest 缺字段，进入 quarantine，不进入 active local；如果实际 NAS / trading_pc 目录与模板不符，停止并要求用户确认环境值。endpoint matrix 的 `query_positions` 默认阻断优先级冲突暂不改代码，作为 CP5 风险暴露。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Full HLD N/A：CR089 架构证据为策略包 NAS 交付规划 + 本 Decision Brief。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 5 | 0 | CP2 决策作为架构输入，不重复计入。 |
| CP2 checkpoint | `process/checkpoints/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md` | scanned | 5 | 0 | scope / non-auth 已承接。 |
| 策略包交付规划 | `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | scanned | 12 | 6 | NAS、package、cache、approval、smoke 和 endpoint risk 纳入。 |
| 自动预检结果 | `process/checks/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-CONSISTENCY.md` | scanned | 6 | 6 | 架构检查项转 DQ。 |
| discussion log / checkpoint | `process/discussions/*` | n/a | 0 | 0 | runtime gate 不需要 CP3 architecture gray area 讨论日志。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR089-01 | architecture | NAS 是否作为默认执行根？ | 不作为执行根，仅作为 package exchange。 | NAS 原地执行；人工 U 盘；Git release。 | 推荐减少网络抖动和半写入风险；NAS 原地执行简单但脆弱。 | 需要交易主机本地缓存。 | 用户要求临时 smoke 原地读取时另列例外。 |
| DQ-CP3-CR089-02 | architecture | 策略包结构如何确定？ | 使用 `zip + sha256 + manifest.yaml + docs + validation`，包含 `targets/qmt_terminal` 和 `targets/miniqmt_runner`。 | 只放单文件脚本；只放 wheel；仅 Git tag。 | 推荐可审计；单文件不可追溯。 | 需要 manifest 校验。 | 目标平台变化时扩展 manifest。 |
| DQ-CP3-CR089-03 | architecture | 交易主机如何落地 package？ | 校验后复制到 `${TRADING_PACKAGE_CACHE_ROOT}/packages/<id>/<version>`，active symlink 指向当前版本。 | 每次直接从 NAS 运行；覆盖同名目录。 | 推荐不可变缓存和可回滚；覆盖目录风险高。 | 需要本地磁盘空间。 | 缓存空间不足时停止。 |
| DQ-CP3-CR089-04 | implementation | index / approval / quarantine 如何设计？ | 使用 `index/package-index.yaml`、`approvals/*.yaml`、`quarantine/<package_id>/rejection.yaml`。 | 只靠文件名约定；人工口头审批。 | 推荐可审计；口头审批不可追踪。 | 多一个审批文件。 | 简化 smoke 可保留 approval stub。 |
| DQ-CP3-CR089-05 | security | qmt_interface_smoke 能否含交易写入能力？ | 不允许，只含 manifest/checksum/target check 和授权后的 `query_positions`。 | 包含 submit/cancel 测试；包含 simulation。 | 推荐权限最小；备选高风险。 | 不验证交易写接口。 | 需要写接口时另起 high-risk gate。 |
| DQ-CP3-CR089-06 | risk_acceptance | 如何处理 endpoint matrix 语义冲突？ | 接受为 CP5/实现前风险；不在本门禁改代码。 | 立即改 `query_positions` 默认阻断优先级；忽略冲突。 | 推荐先决策再改；立即改可能破坏 CR020 测试。 | 后续实现前必须重审。 | CP5/实现阶段确认语义后再修。 |

### 用户需决策事项

用户需决策事项：是否接受 DQ-CP3-CR089-01、DQ-CP3-CR089-02、DQ-CP3-CR089-03、DQ-CP3-CR089-04、DQ-CP3-CR089-05、DQ-CP3-CR089-06 的推荐方案。回复 `approve` 表示接受这 6 项推荐方案；不表示授权“不授权项”中的任何操作。

### CP3 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 候选架构适用条件 | 需要 QMT 运行级从 NAS 获取可运行策略，但希望最小化真实运行风险。 |
| 推荐方案 | NAS package exchange + manifest/checksum + trading_pc immutable cache + qmt_interface_smoke readonly endpoint。 |
| 优化项 | 可审计、可回滚、隔离 NAS 网络状态、明确不授权边界。 |
| 牺牲项 | 首次准备多一个 package / approval 流程。 |
| When to switch | 需要低延迟执行、NAS 不稳定、交易主机缓存空间不足或需要写接口时另起 gate。 |
| Use Case -> Architecture Traceability | UC: 从 NAS 获取策略并验证 QMT 接口；Design: package exchange + local cache + readonly smoke。 |

### 不授权项

如果你回复 approve，表示接受上述 6 项推荐方案；不表示授权以下操作：

- 不授权访问、列目录、读取、写入、复制或删除 NAS 内容。
- 不授权读取 `.env`、token、账号、密码、HMAC secret 或 QMT 凭据。
- 不授权启动 QMT / MiniQMT / XtQuant / gateway。
- 不授权账户原文查询或下单 / 撤单 / simulation / live。
- 不授权修改 `query_positions` 语义冲突；实现前需另行确认。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 context 已生成 | approved | `process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml` |  |
| CP3 auto check 通过 | approved | `process/checks/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-CONSISTENCY.md` |  |
| 策略包规划可读 | approved | `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | NAS package exchange 架构是否接受 | approved | DQ-CP3-CR089-01 |  |
| 2 | 策略包结构是否接受 | approved | DQ-CP3-CR089-02 |  |
| 3 | trading_pc 本地缓存是否接受 | approved | DQ-CP3-CR089-03 |  |
| 4 | index / approval / quarantine 是否接受 | approved | DQ-CP3-CR089-04 |  |
| 5 | smoke 不含交易写入是否接受 | approved | DQ-CP3-CR089-05 |  |
| 6 | endpoint 语义冲突后置是否接受 | approved | DQ-CP3-CR089-06 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 本文件人工审查结果 |  |
| approve 后仍需 CP5 readiness | approved | Decision Brief |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 context | `process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml` | approved |  |
| CP3 auto check | `process/checks/CP3-CR089-QMT-INTERFACE-VALIDATION-HLD-CONSISTENCY.md` | approved |  |
| CP3 launch message | `process/checks/CP3-CR089-HUMAN-GATE-LAUNCH-MESSAGE.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T21:34:53+08:00
- 修改意见：用户回复“同意”，按 CR089 CP3 approve 处理；接受 DQ-CP3-CR089-01..06 的推荐方案。NAS 仅作为 package exchange，不作为默认执行根；策略包采用 zip + sha256 + manifest.yaml + docs + validation；交易主机校验后复制到本地不可变缓存；endpoint matrix / query_positions 语义冲突保留为后续实现前风险，本门禁不改代码。
- 风险接受项：接受当前只确认架构与交付合同，不执行 NAS 原地运行、不执行 NAS publish/pull/list/copy/delete、不启动 QMT/MiniQMT、不验证交易写接口；后续需要真实接口或写接口时必须另起 runtime / high-risk gate。
