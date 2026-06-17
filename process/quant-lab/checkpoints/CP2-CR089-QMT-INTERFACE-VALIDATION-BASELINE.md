---
checkpoint_id: "CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE"
checkpoint_name: "CR089 QMT Interface Validation Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T21:08:59+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T21:34:53+08:00"
auto_check_result: "process/checks/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md"
context_capsule: "process/context/CP2-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/context/CP2-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml"
    - "docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md"
---

# CP2 CR089 QMT Interface Validation Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md` | PASS | 0 | CR089 scope、CR046 active lock、CR020 deleted-by-user 和不授权边界已记录。 |

## Decision Brief

推荐决策：批准 CR089 CP2 baseline，把 CR089 定义为独立但仍 blocked 的 QMT 接口验证门禁；当前只确认策略包输出规范、NAS package exchange 交付位置、交易主机取包校验流程和 `query_positions` 只读 smoke 边界。不恢复 CR020，不推进 CR046 CP7，不授权 agent 访问 NAS、读取凭据或启动 QMT。

备选方案：合并回 CR046 恢复；等待 CR046 CP8 后再启动；仅保留无 runtime 的 package 规范；取消 CR089。推荐方案优先，因为它保留接口验证路线，同时不破坏当前 active formal CR 锁。

影响维度：QMT 接口验证范围、NAS 策略包交换、交易主机取包、凭据边界、CR046 active lock、CR020 deleted-by-user 追溯。

风险与回退：若 reject，CR089 保持 blocked，不进入 CP3/CP5；若 approve，只进入设计和 readiness 门禁，不授权真实运行。若后续要合并、等待或拆分，可通过 CR089 状态回退或新 CR 处理。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Scenario discussion log N/A：CR089 是 runtime gate，不是新产品发现流程。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 本轮由 CR089 formal CR 和用户显式选择建立。 |
| CR089 formal CR | `process/changes/CR-089-QMT-INTERFACE-VALIDATION-GATE-2026-06-17.md` | scanned | 8 | 5 | scope、CR046、CR020、runtime authorization 纳入。 |
| 启动冲突预检 | `process/checks/CR089-STARTUP-CONFLICT-PRECHECK-2026-06-17.md` | scanned | 5 | 3 | conflict decision 和 endpoint risk 纳入后续门禁。 |
| 策略包交付规划 | `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | scanned | 8 | 4 | package exchange、local cache、smoke 边界纳入。 |
| 用户显式选择题 | 当前对话“同意” | scanned | 4 | 4 | 已接受 DQ-CR089-01..04，作为 CP2 输入，不直接视为 CP2 approved。 |
| discussion log / checkpoint | `process/discussions/*` | n/a | 0 | 0 | runtime gate 不需要 CP2 场景讨论日志。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR089-01 | scope | 是否正式进入 CR089 QMT 接口验证门禁材料阶段？ | 进入 CP2/CP3/CP5 材料阶段，但 CR089 保持 blocked，不设 active。 | 合并到 CR046；等待 CR046 CP8；取消 CR089。 | 推荐保留路线且不破坏 active lock；备选更保守但会推迟验证。 | active lock 仍由 CR046 持有。 | 用户要求合并或等待时切换。 |
| DQ-CP2-CR089-02 | scope | 本轮范围是否仅覆盖策略包 + 只读 smoke？ | 是，仅覆盖 package exchange、manifest、trading_pc intake 和 `query_positions` smoke。 | 扩展 health/capabilities；扩展 submit/cancel；扩展 simulation。 | 推荐权限最小；扩展项风险更高。 | 不覆盖交易写入能力。 | 需要下单或模拟盘时另起 high-risk gate。 |
| DQ-CP2-CR089-03 | security | CR020 是否可以恢复为验证入口？ | 不恢复；CR020 deleted-by-user，仅审计复用只读接口代码和 runbook 边界。 | 恢复 CR020；复制旧根 STATE。 | 推荐符合用户删除历史；备选会污染状态。 | 旧根 STATE 不能覆盖当前状态。 | 用户明确撤销 deleted-by-user 时另起治理。 |
| DQ-CP2-CR089-04 | runtime_authorization | CP2 是否授权 NAS/QMT/凭据/账户动作？ | 不授权，仅确认范围。 | CP2 直接授权 NAS pull；授权 agent 代跑 QMT。 | 推荐最小权限；备选越过门禁。 | 无外部副作用。 | 需要真实动作时进入独立 runtime authorization。 |
| DQ-CP2-CR089-05 | follow_up_tracking | CR089 和 CR046 的关系如何登记？ | CR089 继续登记为 blocked formal CR；CR046 保持 active-cp6-pass-ready-for-verification。 | 将 CR089 改为 active；关闭或恢复 CR046。 | 推荐避免双 active；备选需要更大状态迁移。 | CR089 不会自动执行。 | CR046 CP8 或用户要求状态切换时重审。 |

### 用户需决策事项

用户需决策事项：是否接受 DQ-CP2-CR089-01、DQ-CP2-CR089-02、DQ-CP2-CR089-03、DQ-CP2-CR089-04、DQ-CP2-CR089-05 的推荐方案。回复 `approve` 表示接受这 5 项推荐方案；不表示授权“不授权项”中的任何操作。

### CP2 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 用户真实意图 | 用户要验证 QMT 接口，并确认 QMT 运行级可从 NAS 获取可运行策略。 |
| 场景覆盖 | 策略包从 research_pc 产出到 NAS package exchange；trading_pc 拉取、校验、本地缓存；用户手工只读 smoke。 |
| Scenario Gray Areas | N/A，本轮是 runtime gate；灰区已转为 DQ。 |
| Deferred Ideas | submit/cancel、simulation/live、agent 代跑、NAS 原地执行、CR020 恢复。 |
| 回退方式 | reject 时保持 CR089 blocked；修改时重写 CP2/CP3/CP5 材料。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权访问、列目录、读取、写入、复制或删除 NAS 内容。
- 不授权读取 `.env`、token、API key、账号、密码、HMAC secret、cookie、session、private key 或 QMT 凭据。
- 不授权启动 QMT、MiniQMT、XtQuant、gateway 或任何 broker runtime。
- 不授权账户原文查询、持仓原文输出、成交 / 委托原文输出。
- 不授权 submit_order、cancel_order、simulation、live_readonly、small_live 或 scale_up。
- 不授权恢复 CR020、从旧根 STATE 恢复状态、覆盖当前 CR046 active lock。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR089 formal CR 已存在 | approved | `process/changes/CR-089-QMT-INTERFACE-VALIDATION-GATE-2026-06-17.md` |  |
| CP2 context 已生成 | approved | `process/context/CP2-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml` |  |
| CP2 auto check 通过 | approved | `process/checks/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否接受 CR089 blocked gate baseline | approved | DQ-CP2-CR089-01 |  |
| 2 | 是否接受只读 smoke scope | approved | DQ-CP2-CR089-02 |  |
| 3 | 是否接受 CR020 不恢复 | approved | DQ-CP2-CR089-03 |  |
| 4 | 是否接受 CP2 不授权 runtime | approved | DQ-CP2-CR089-04 |  |
| 5 | 是否接受 CR089/CR046 登记关系 | approved | DQ-CP2-CR089-05 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 本文件人工审查结果 |  |
| approve 后仍不执行 NAS/QMT/凭据动作 | approved | 不授权项 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 context | `process/context/CP2-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml` | approved |  |
| CP2 auto check | `process/checks/CP2-CR089-QMT-INTERFACE-VALIDATION-BASELINE.md` | approved |  |
| CP2 launch message | `process/checks/CP2-CR089-HUMAN-GATE-LAUNCH-MESSAGE.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T21:34:53+08:00
- 修改意见：用户回复“同意”，按 CR089 CP2 approve 处理；接受 DQ-CP2-CR089-01..05 的推荐方案。CR089 继续作为独立 blocked formal CR 记录，CR046 保持 active-cp6-pass-ready-for-verification，CR020 保持 deleted-by-user 且不得恢复。
- 风险接受项：接受 CR089 当前只完成范围 / 安全边界确认，不进入 active、不恢复 CR020、不推进 CR046 CP7；本批准不授权 NAS 内容访问、凭据读取、QMT/MiniQMT/XtQuant/gateway 启动、账户原文查询、submit/cancel、simulation/live。
