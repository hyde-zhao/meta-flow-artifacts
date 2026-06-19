---
checkpoint_id: "CP2"
checkpoint_name: "CR092 Readonly Runtime Smoke Scope Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-18T16:05:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-18T16:17:35+08:00"
auto_check_result: "process/checks/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE.md"
target:
  phase: "requirement-clarification"
  story_id: "CR092"
  artifacts:
    - "process/context/CP2-CR092-REQUIREMENT-CONTEXT.yaml"
    - "process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md"
    - "process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md"
---

# CP2 CR092 Readonly Runtime Smoke Scope Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE.md` | PASS | 0 | CR092 可进入 CP2 人工审查；历史 CR019 / CR025 旧账为 non-blocking waived risk。 |

## Decision Brief

CR092 是从 CR091 CP8 后续候选 `CR091-FU-01` 启动的真实只读 runtime smoke 设计门禁。它的 CP2 只确认范围、风险和不授权边界：是否允许继续产出 CP3 HLD 与 CP5 readiness 设计材料。

推荐结论：批准 CP2 范围基线，进入 CP3 HLD。CP2 批准不授权任何真实运行、NAS、凭据、账户原文、交易写、simulation/live、provider/lake/publish，也不自动启动 CR089 或恢复 CR020 gateway 路线。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR092-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | `ready-for-human-review` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次；本轮只读取 CR092 / CR091 follow-up / CR091 CP8 / release context |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 本轮 CP2 决策在本 checkpoint 中新增。 |
| 自动预检结果 | `process/checks/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE.md` | scanned | 3 | 3 | runtime / tracking / no-execution 风险纳入 DQ。 |
| CR092 正式 CR | `process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md` | scanned | 5 | 5 | scope、runtime、security、CR089 关系、后续路径均纳入 DQ。 |
| CR091 follow-up tracking | `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md` | scanned | 4 | 1 | 仅 CR091-FU-01 本轮启动；FU-02..04 保持候选。 |
| 用户显式选择题 | 当前会话 | scanned | 1 | 1 | 用户选择 CR091-FU-01。 |
| discussion log / checkpoint | N/A | n/a | 0 | 0 | 本轮是 CP8 follow-up CR 启动，不新增 SGQ 异步讨论。 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR092-01 | scope | CR092 是否只作为真实只读 runtime smoke 的设计门禁？ | 是，仅确认范围并进入 CP3 / CP5 设计，不执行真实 smoke。 | 直接进入真实 runtime；取消 CR092 回到 follow-up backlog。 | 推荐方案保留推进路径且不越权；直接 runtime 风险过高；取消会保留缺口。 | 真实可用性仍未证明，但安全边界清楚。 | 用户要求真实动作时另起 runtime authorization 或重出 CP2。 |
| DQ-CP2-CR092-02 | runtime_authorization | CP2 approve 是否授权 QMT/MiniQMT/XtQuant/gateway/runner 启动或连接？ | 不授权。 | 授权一次只读 smoke；授权准备真实运行脚本。 | 推荐方案符合当前门禁；备选必须先完成 HLD/LLD 和逐 run 授权。 | 防止把设计批准误读成执行授权。 | 用户明确要求执行时回退并重出 runtime gate。 |
| DQ-CP2-CR092-03 | security | NAS、凭据、账户原文、submit/cancel、simulation/live、provider/lake/publish 是否继续禁止？ | 继续全部禁止。 | 放开只读账户查询；放开 NAS package exchange。 | 推荐方案最小风险；备选会引入凭据 / NAS / 账户边界。 | 后续 HLD 必须用脱敏 evidence 和禁止动作计数。 | 用户修改授权边界时重出 CP2。 |
| DQ-CP2-CR092-04 | architecture | CR092 与 CR089 的关系如何处理？ | CR092 独立推进设计门禁，CR089 保持 blocked，不自动启动。 | 合并进 CR089；先恢复 CR089；取消 CR092。 | 推荐方案避免恢复 NAS package exchange 和 CR089 runtime；合并会扩大范围。 | 两者语义相关但授权边界不同。 | 用户要求合并时创建影响分析并重排状态。 |
| DQ-CP2-CR092-05 | implementation | CP2 approve 后下一步允许做什么？ | 只允许生成 CP3 HLD、CP5 LLD / TEST-PLAN / readiness，不写真实 runtime 代码，不执行 smoke。 | 允许实现 manual guide；允许实现 runner runtime adapter。 | 推荐方案先设计后实现；备选需要 CP5 后再评估。 | 避免 CP2 后直接进入高风险实现。 | CP3/CP5 未 approved 前不得实施。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP2-CR092-01..05，允许 CR092 进入 CP3 HLD 设计 |
| 备选方案 | `修改: <具体修改点>` 后重出 CP2；或 `reject` 回退到 CR091 follow-up backlog / 取消 / 合并 |
| 影响维度 | 范围、运行授权、安全边界、CR089 关系、后续设计 / 实施边界 |
| 优劣分析 | 推荐方案先冻结真实只读 runtime smoke 设计，不越过真实运行授权；备选方案要么扩大高风险授权，要么保留缺口 |
| 风险与回退 | CP2 通过仍不证明 runtime ready；reject 或修改时回退到 CR092 active pending CP2 或 CR091 follow-up backlog |
| 用户需决策事项 | DQ-CP2-CR092-01 是否只作为设计门禁；DQ-CP2-CR092-02 是否不授权 runtime 启动 / 连接；DQ-CP2-CR092-03 是否继续禁止 NAS / 凭据 / 账户原文 / 交易写 / simulation/live / provider/lake/publish；DQ-CP2-CR092-04 是否保持 CR089 blocked 且不自动启动；DQ-CP2-CR092-05 是否只允许进入 CP3 HLD 与 CP5 readiness 设计 |

### CP2 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 用户真实意图 | 从 CR091 离线 runner 过渡到真实只读 runtime smoke，但先做设计门禁 |
| 场景覆盖 | runtime health、capabilities、readonly query_positions、redacted evidence、forbidden counters、失败回退 |
| 认知盲区补充 | 真实 QMT / MiniQMT / XtQuant / gateway / runner 可用性、交易主机环境、账户权限和证据脱敏仍未证明 |
| Deferred Ideas | NAS package exchange、order-write / submit-cancel、ledger hygiene、evidence output gate |
| 用户选择影响 | 选择 CR091-FU-01 后，CR092 成为唯一 active formal CR |
| 回退方式 | 取消 CR092，回到 CR091 follow-up backlog；CR089 继续 blocked-readiness-approved |

## 不授权范围

本 CP2 即使通过，也不授权：

- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089。
- 恢复 CR020 用户删除的 gateway 路线。

## Entry Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CR092 已创建 | 待审查 | `process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md` |  |
| CP2 context capsule ready | 待审查 | `process/context/CP2-CR092-REQUIREMENT-CONTEXT.yaml` |  |
| 自动预检通过 | 待审查 | `process/checks/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE.md` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | Decision Brief 覆盖推荐、备选、影响、风险和回退 | 待审查 | 本文件 Decision Brief |  |
| 2 | Decision Collection Coverage 已列出 | 待审查 | 本文件覆盖表 |  |
| 3 | 用户视角不授权项已列出 | 待审查 | 本文件“不授权范围” |  |
| 4 | CR089 不自动启动 | 待审查 | DQ-CP2-CR092-04 |  |
| 5 | CP2 approve 仅允许进入设计 | 待审查 | DQ-CP2-CR092-05 |  |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 5 项推荐方案，CR092 进入 CP3 HLD 设计。 |
| `修改: <具体修改点>` | 按修改点修订 CP2 材料并重新发起确认。 |
| `reject` | 回退 CR092 到 follow-up backlog，或按用户要求取消 / 合并。 |

## Deliverables

- `process/context/CP2-CR092-REQUIREMENT-CONTEXT.yaml`
- `process/checks/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE.md`
- `process/checkpoints/CP2-CR092-READONLY-RUNTIME-SMOKE-SCOPE-REVIEW.md`
- `process/checks/CP2-CR092-HUMAN-GATE-LAUNCH-MESSAGE.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-18T16:17:35+08:00
- 用户回复：好的，按照你的建议。另外CR092的决策项我同意了，你继续推进。
- 接受的决策项：DQ-CP2-CR092-01、DQ-CP2-CR092-02、DQ-CP2-CR092-03、DQ-CP2-CR092-04、DQ-CP2-CR092-05
- 修改意见：无
- 风险接受项：接受 CP2 只作为范围 / 运行授权基线门禁；接受真实 runtime readiness 仍未证明的设计阶段风险。
- 不授权边界：本次 approve 不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env` / 凭据 / 账号 / 账户 / 资金 / 持仓 / 委托 / 成交 / 日志原文、submit / cancel、buy / sell、simulation / live、provider / lake / catalog / publish、CR089 自动启动或 CR020 gateway 路线恢复。
