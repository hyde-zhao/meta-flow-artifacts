---
handoff_id: "CR091-FU-02-START-HANDOFF-2026-06-19"
status: "ready-for-context-reset"
created_at: "2026-06-19T17:32:50+08:00"
created_by: "host-orchestrator"
source_candidate: "CR091-FU-02"
recommended_formal_cr: "allocate-next-cr-id"
recommended_title: "NAS Package Exchange Offline Readiness Gate"
semantic: "context-reset-startup"
dispatch:
  required: false
  semantic: "handoff-created"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "host-orchestrator"
  canonical_role: "host-orchestrator"
  codex_agent_name: ""
  reasoning_profile: "default"
  dispatch_trigger: "user-context-reset"
  tool_name: ""
  agent_id: ""
  thread_id: ""
  evidence: "User agreed to prepare context for starting CR091-FU-02 after clearing context."
question_permission:
  can_ask_user: true
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "CR091-FU-02 scope, offline readiness boundaries, CP2/CP3/CP5/CP8 decisions"
  forbidden_question_scope: "credential values, NAS private path originals, runtime authorization, submit/cancel, simulation/live, publish execution without a separate explicit gate"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CR091-FU-02-START-HANDOFF-2026-06-19.md"
  read_profile: "compact"
  max_source_files: 10
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
---

# CR091-FU-02 Start Handoff

## Resume Prompt

清除上下文后，可以把下面这段作为新会话开头：

```text
恢复 quant-lab，必须用简体中文。

第一步必须运行：
uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab

然后读取：
process/context/CR091-FU-02-START-HANDOFF-2026-06-19.md
process/STATE.md
process/changes/CR-INDEX.yaml
process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md
process/context/CR091-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-18.md
process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml
process/context/CP5-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml

用户已同意按建议准备 CR091-FU-02。当前目标是启动 CR091-FU-02 NAS Package Exchange Offline Readiness Gate。

注意：
- 先做冲突预检，再分配下一个正式 CR ID；不要假定一定是 CR100。
- 当前 NAS 无法连接，所以本轮默认做 offline readiness，不做真实 NAS publish / pull / copy / 校验验收。
- 不授权访问、列取、读取、复制、写入、发布或删除 NAS。
- 不授权读取 Windows/Linux .env、NAS 凭据、账号、账户、资金、持仓、委托、成交或原始日志。
- 不授权 QMT/MiniQMT/XtQuant/gateway/runner runtime。
- 不授权 submit/cancel、buy/sell、simulation/live、provider/lake/catalog publish。

推荐 CR 范围：
CR091-FU-02 NAS Package Exchange Offline Readiness Gate，只做离线设计、本地 fake exchange fixture、package manifest/schema、publish/pull/checker 脚本和 NAS 恢复后的真实验证 runbook。

先输出 CP2 scope/risk/runtime-authorization decision brief，不要直接实现或连接 NAS。
```

## Current State

| 字段 | 内容 |
|---|---|
| 当前阶段 | `delivered` |
| active formal CR | none |
| 最近关闭 | `CR099` closed-current-delivery / READY_WITH_RISK |
| 当前候选 | `CR091-FU-02 NAS package exchange gate` |
| 用户意图 | 清除上下文后启动 CR091-FU-02 |
| 当前 NAS 状态 | 用户说明当前无法连上 NAS |
| 推荐策略 | 先做 `offline readiness gate`，不等待 NAS 恢复 |

## Must Read

| 路径 | 原因 |
|---|---|
| `process/context/CR091-FU-02-START-HANDOFF-2026-06-19.md` | 本启动交接入口 |
| `process/STATE.md` | 当前 active_change、cr_tracking 和不授权边界 |
| `process/changes/CR-INDEX.yaml` | 分配下一个正式 CR ID 前检查冲突 |
| `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md` | `CR091-FU-02` 候选来源 |
| `process/context/CR091-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-18.md` | CR091 关闭事实和后续候选说明 |
| `process/changes/CR-091-QMT-STRATEGY-RUNNER-RESEARCH-DESIGN-IMPLEMENTATION-PLAN-2026-06-18.md` | CR091 对 NAS package exchange 的后续定义 |
| `process/context/CP3-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml` | NAS package exchange 架构事实 |
| `process/context/CP5-CR089-QMT-INTERFACE-VALIDATION-CONTEXT.yaml` | package readiness / smoke contract 边界 |

## Read If Needed

| 路径 | 触发条件 |
|---|---|
| `docs/qmt/CR089-STRATEGY-PACKAGE-NAS-DELIVERY-PLAN.md` | CP3/CP5 capsule 字段不足、冲突或需要完整架构表时读取 |
| `docs/qmt/CR046-DUAL-TARGET-STRATEGY-FRAMEWORK.md` | 需要核对策略包结构或 dual target package contract 时读取 |
| `docs/research/HOST-WORKFLOW.md` | 需要核对 research_pc / NAS / trading_pc 边界时读取 |
| `docs/research/ARCHIVE-GOVERNANCE.md` | 需要核对 package exchange 禁止项或归档策略时读取 |

## Do Not Read By Default

| 路径 / 对象 | 原因 |
|---|---|
| NAS 实际挂载内容 | 当前 NAS 不可达，且未授权访问 / list / read / copy / write / publish / delete |
| `.env*`、凭据、token、NAS secret | 不属于 CR091-FU-02 offline readiness 输入 |
| QMT / MiniQMT / XtQuant / gateway / runner runtime | 本候选只做 package exchange readiness，不启动 runtime |
| 账户、资金、持仓、委托、成交、原始日志 | 不属于 package exchange gate |
| submit/cancel、buy/sell、simulation/live | 属于 `CR091-FU-03` / order-write gate，不属于本候选 |
| 完整历史 transcript、无关 CR 全文 | 使用 capsule-first；仅冲突或审计时扩展 |

## Recommended CR Scope

建议把 `CR091-FU-02` 启动为：

```text
NAS Package Exchange Offline Readiness Gate
```

目标：

- 定义 research_pc -> NAS exchange -> trading_pc immutable cache 的交付合同。
- 定义 package manifest / index / approval / quarantine / active pointer / rollback 规则。
- 定义研发机 publish tool 和交易机 pull tool 的权限边界。
- 用本地 fake exchange root 做离线 fixture，不连接真实 NAS。
- 生成 NAS 恢复后的真实 publish / pull / copy / 校验验收 runbook。
- 生成 redacted evidence schema，后续真实 NAS 验证只记录摘要、状态、hash pass/fail 和计数桶。

本轮不证明：

- 真实 NAS 可连。
- 真实 NAS publish / pull / copy 成功。
- trading_pc 已从 NAS 拉取真实包。
- QMT / MiniQMT / runner 能运行策略包。
- submit/cancel、成交或交易链路可用。

## Recommended Stages

| 阶段 | 目标 | 主要产物 |
|---|---|---|
| CP2 | Scope / non-authorization baseline | CP2 precheck + manual checkpoint，确认 offline readiness，不授权 NAS 真实访问 |
| CP3 | Package exchange architecture | HLD / architecture decisions：research_pc publish、NAS exchange、trading_pc readonly pull、本地 immutable cache |
| CP5 | Readiness design | DESIGN / TEST-PLAN / TASKS / LLD：manifest、index、approval、quarantine、active pointer、fake exchange fixture |
| CP6 | Offline implementation | package skeleton、checker、publish/pull dry-run scripts、fixture tests |
| CP7 | Verification | local fake exchange tests、hash mismatch fail closed、missing manifest fail closed、unapproved package fail closed |
| CP8 | Delivery readiness | READY_WITH_RISK；风险为真实 NAS 链路未验证 |

## Recommended Package Contract

最小策略包结构建议：

```text
qmt_interface_smoke-<version>/
  manifest.yaml
  strategy_core/
    __init__.py
    strategy.py
  targets/
    qmt_terminal/
      entry.py
    miniqmt_runner/
      entry.py
  validation/
    expected_capabilities.yaml
    smoke_checklist.md
  docs/
    README.md
```

`manifest.yaml` 最小字段建议：

```yaml
package_id: qmt_interface_smoke
package_version: 0.1.0
created_at: "..."
target_platforms:
  - qmt_terminal
  - miniqmt_runner
entrypoints:
  qmt_terminal: targets/qmt_terminal/entry.py
  miniqmt_runner: targets/miniqmt_runner/entry.py
approval:
  status: approved
  approved_by: user
  approved_at: "..."
hashes:
  strategy_core/strategy.py: "sha256:..."
permissions:
  runtime: false
  submit_cancel: false
  simulation_live: false
  credential_read: false
```

## Recommended Tool Boundaries

| 工具 | 运行位置 | NAS 权限 | 本轮状态 |
|---|---|---|---|
| package builder | research_pc / local dev | none | 可离线实现 |
| package validator | research_pc / trading_pc / local fixture | none | 可离线实现 |
| fake publish fixture | local fake exchange root | local temp only | 可离线测试 |
| fake pull fixture | local fake exchange root -> local cache | local temp only | 可离线测试 |
| real NAS publish | research_pc -> NAS | write | 本轮不授权 |
| real NAS pull | trading_pc <- NAS | readonly | 本轮不授权 |
| trading_pc active pointer switch | local cache | local write | 只能在 fake fixture 中验证 |

## Fail Closed Rules

| 条件 | 行为 |
|---|---|
| NAS 不可达 | 不改变 active pointer；真实 NAS 验证保持 blocked / deferred |
| approved index 缺失 | fail closed |
| package 缺失 | fail closed |
| manifest 缺失 | fail closed |
| approval 不是 `approved` | fail closed |
| sha256 不匹配 | quarantine，不启用 |
| entrypoint 越界 | fail closed |
| package 含 `.env` / secret marker | fail closed |
| cache copy 不完整 | 不切换 active |
| rollback 未显式允许 | 不允许降级覆盖 active |

## Initial Decision Items To Create

启动正式 CR 后，CP2 至少应形成这些 DQ：

| 决策 ID 建议 | 类型 | 推荐方案 |
|---|---|---|
| `DQ-CP2-<CR>-01` | scope | 启动 NAS package exchange offline readiness，不做真实 NAS 操作 |
| `DQ-CP2-<CR>-02` | runtime_authorization | 不授权 NAS 访问 / list / read / copy / write / publish / delete |
| `DQ-CP2-<CR>-03` | implementation | 允许本地 fake exchange fixture、package manifest/schema/checker、dry-run publish/pull scripts |
| `DQ-CP2-<CR>-04` | security | 凭据、`.env`、账号、账户、交易和 runtime 全部 out of scope |
| `DQ-CP2-<CR>-05` | risk_acceptance | 接受真实 NAS 链路不可验证，CP8 可按 READY_WITH_RISK 收口 |

## Non-Authorization Boundary

本 handoff 和后续 CP2/CP3/CP5 设计确认都不授权：

- NAS access / list / read / copy / pull / write / publish / delete。
- 读取 `.env`、NAS 凭据、账号、账户、资金、持仓、委托、成交或原始日志。
- QMT / MiniQMT / XtQuant / gateway / runner runtime。
- submit/cancel、buy/sell、simulation/live。
- provider fetch、lake write、catalog publish。
- 从 NAS 原地执行策略。
- trading_pc 写回 NAS receipt，除非未来独立授权。

## Startup Checklist

新上下文启动后先做：

1. 运行 `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab`。
2. 运行 `uv run --python 3.11 meta-flow check cr-tracking --project-root . --strict-warnings`。
3. 确认 active formal CR 仍为 none；若不是 none，先处理冲突。
4. 读取本 handoff 和 must-read 文件。
5. 为 `CR091-FU-02` 分配下一个正式 CR ID，不要复用已关闭 CR。
6. 创建正式 CR 和 CP2 scope/risk/runtime authorization baseline。
7. 发起 CP2 人工门禁，等待用户 approve / 修改 / reject。

## Expected First Answer In New Context

新会话读取本 handoff 后，应先回复：

```text
我已读取 CR091-FU-02 启动 handoff。当前 active formal CR 应为 none；CR091-FU-02 建议启动为 NAS Package Exchange Offline Readiness Gate。下一步我会先做 cr-tracking 冲突预检，再创建正式 CR 和 CP2 scope/risk/runtime authorization baseline；本轮不连接 NAS、不读取凭据、不启动 runtime。
```
