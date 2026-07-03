---
handoff_id: H-CR037-S08-NEXT-SESSION
cr_id: CR-037
story_id: CR037-S08
from_role: host-orchestrator
to_role: meta-dev
stage: story-execution
status: handoff-created
created_at: 2026-07-03T17:53:58+08:00
context_ref: process/context/stories/CR037-S08.CP6.work-packet.json
canonical_role: meta-dev
capsule_first: true
read_profile: compact
dispatch:
  required: false
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "meta-dev"
  canonical_role: "meta-dev"
  codex_agent_name: ""
  reasoning_profile: "default"
  dispatch_trigger: "next-session startup after CR037-S07 CP7 PASS_WITH_RISK"
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  completed_at: ""
  evidence: "handoff-only; no subagent execution claimed"
question_permission:
  can_ask_user: false
  mode: "queue-only"
  structured_choice_allowed: false
  allowed_question_scope: "Only implementation blockers should be queued by the next CP6 execution agent."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 gates, runtime authorization, credentials, security boundary changes, publish, live/trading authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  story_packet_ref: "process/context/stories/CR037-S08.CP6.work-packet.json"
  read_profile: "compact"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/context/stories/CR037-S08.CP6.work-packet.json"
  must_read:
    - "process/context/stories/CR037-S08.CP6.work-packet.json"
  read_if_needed:
    - "process/stories/STORY-CR037-S08-impact-surface-field-split-and-migration-report.md"
    - "process/stories/STORY-CR037-S08-impact-surface-field-split-and-migration-report-LLD.md"
    - "process/docs/features/impact-surface-normalization/DESIGN.md"
    - "process/docs/features/impact-surface-normalization/TEST-PLAN.md"
    - "process/docs/features/impact-surface-normalization/TASKS.md"
    - "meta_flow/design/feature_registry.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/STORY-STATUS.md"
    - "process/changes/*.md"
    - "process/quant-lab/**"
    - "docs/**"
    - "delivery/**"
    - "process/returns/**"
    - "process/evidence/**"
    - "process/checks/**"
---

# CR037-S08 Next Session Handoff

## 当前状态

- Active CR: `CR-037`
- Active Story: `CR037-S08`
- Current phase: `story-execution`
- Pending gate: `CP6`
- Active context: `process/context/stories/CR037-S08.CP6.work-packet.json`
- 上一阶段结论: `CR037-S07` 已通过 CP7，结论为 `PASS_WITH_RISK`，并解封 `CR037-S08`。

本文件是下一会话启动交接，不是子 agent 执行证据。下一会话若启动 `meta-dev`，必须记录 `spawn_agent` / `resume_agent` / `send_input` 或用户批准的 inline-fallback 证据。

## 已完成内容摘要

- `CR037-S00` 到 `CR037-S07` 已完成 CP6/CP7，均进入 `verified-with-risk` 或等价 verified 状态。
- `S07` 完成 feature / capability registry resolver，实现文件主要为 `meta_flow/design/feature_registry.py`，测试为 `tests/test_feature_registry.py`。
- 当前采用 S07+ 执行策略：dev agent 只负责源码和测试；Host Orchestrator 负责运行验证、生成 CP6/CP7 process artifacts，并记录 inline-fallback 或 subagent 证据。
- `STATE.current.json` 已切换到 `CR037-S08`，且 `project_state_ref`、`routing_ref`、`active_context_ref` 均通过 state enforce 检查。

## 下一步目标

启动 `CR037-S08 impact surface field split and migration report` 的 CP6 实现。

S08 目标是把历史被误用为路径 / 模块 / 文件名的 `impact_surface` 拆分为：

- `impact_surfaces`: 有限治理面枚举；
- `affected_paths`: 仅表示影响路径，不代表写权限；
- `feature_refs`: 引用 S07 registry resolver；
- `capability_refs`: 引用 S07 registry resolver；
- migration report: 对 legacy drift 生成报告，不重写历史 CR 文件。

## 必须先读

1. `process/context/stories/CR037-S08.CP6.work-packet.json`

除非 packet 缺字段、冲突、审计或深度评审需要，不要默认全文读取 HLD、全部 Story、全部测试矩阵或历史讨论记录。

## S08 允许写入

以 work packet 为准，当前允许写入：

- `meta_flow/workflow/cr_lifecycle.py`
- `meta_flow/checks/cr_tracking.py`
- `meta_flow/cli.py`
- `tests/test_cr037_impact_surface_normalization.py`
- `tests/test_cr_lifecycle.py`
- `process/checks/CR037-S08-IMPACT-MIGRATION.report.json`

如果继续沿用 S07+ 策略，dev agent 不写 CP6 process artifacts；Host Orchestrator 负责：

- `process/stories/STORY-CR037-S08-impact-surface-field-split-and-migration-report-IMPLEMENTATION.md`
- `process/returns/CR037-S08.CP6.return.json`
- `process/evidence/CR037-S08.CP6.index.json`
- `process/checks/CP6-CR037-S08-CODING-DONE.result.json`
- `process/checks/CP6-CR037-S08-CODING-DONE.result.summary.md`

## 明确禁止

- 不读取或写入 `process/quant-lab/**`。
- 不修改 `docs/**`、`delivery/**`、凭据、runtime、production write、publish、live/trading 相关路径或行为。
- 不把 `affected_paths` 当作写权限。
- 不自动重写历史 CR 文件。
- 不实现 roadmap refresh、stale-check、FU-RF 或 quant-lab migration；这些属于 S09-S13。
- 不新增第二套 feature / capability resolver，必须复用 S07 resolver。

## S08 验证重点

- 新 CR schema 支持 `impact_surfaces`、`affected_paths`、`feature_refs`、`capability_refs`。
- legacy `impact_surface` drift 在 audit 模式给 WARN，在 enforce 模式给 ERROR 或阻断 finding。
- `feature_refs` / `capability_refs` 必须经 S07 resolver 校验，不能接受自由字符串作为 canonical refs。
- migration report 包含 `source_cr`、`legacy_value`、建议字段、finding severity/code、recovery hint 和 source hash。
- migration report 不修改源 CR 文件。
- `process/quant-lab/**` 与 `docs/**` diff 必须为空。

## 已知风险

- `CR-036` 仍是 blocked / unfinished，`cr conflicts --id CR-037` 继续预期失败；不要把 CR-036 伪装成完成。
- `cr-tracking` 仍可能提示历史 warning: `CR-033 missing from CR-INDEX.yaml items`。
- `meta-flow cr check` 仍可能失败于历史 CR index 旧数据，不应误判为 S08 新增问题。
- QA 子 agent 曾多次出现 no-output / interrupted；S08 后续 CP7 可继续按用户已认可策略由 Host inline-fallback 生成验证产物，但质量命令必须照跑。

## 建议启动提示词

```text
继续推进 CR-037，从 CR037-S08 开始执行。请先读取 process/context/stories/CR037-S08.CP6.work-packet.json 和 process/handoffs/CR037-S08-NEXT-SESSION-HANDOFF.md，按 CP6 implementation-execution 流程启动 CR037-S08 impact surface field split and migration report。

保持 S07+ 执行策略：dev agent 只写源码和测试，Host Orchestrator 负责运行验证并生成 CP6/CP7 process artifacts。S08 必须复用 S07 feature/capability registry resolver，不得新增第二套 resolver；不得读取或写入 process/quant-lab/**、docs/**、delivery/**、凭据、runtime、publish、live/trading 或 production write。

完成后运行 S08 work packet 要求的测试、state enforce、event ledger 检查、write boundary diff，并生成 CP6 return/evidence/result/summary。
```
