---
handoff_id: "HO-CR051-CP1-CP2-META-PM"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "requirement-clarification"
status: "completed"
created_at: "2026-07-17T07:47:17Z"
semantic: "delegated-user-interaction"
return_summary_path: "process/handoffs/CR051-CP1-CP2-META-PM-RETURN-SUMMARY.md"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "CR-051 product baseline refresh for project-first per-project worktree lifecycle"
  agent_path: "delivery/agents/meta-pm.md"
  tool_name: "spawn_agent"
  agent_id: "/root/cr051_meta_pm"
  agent_name: "cr051_meta_pm"
  thread_id: "/root/cr051_meta_pm"
  spawned_at: "2026-07-17T07:47:17Z"
  completed_at: "2026-07-17T08:12:28Z"
  evidence: "spawn_agent"
  fallback_reason: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Scenario Gray Areas, requirement ambiguity, scope trade-offs and acceptance semantics"
  forbidden_question_scope: "CP2 formal approval, runtime authorization, credentials, repository publication, real artifact migration or real Git mutation"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
  context_ref: "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  allowed_reads:
    - "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
    - "process/state/STATE.current.json"
    - "process/current/CURRENT.json"
    - "process/changes/summaries/CR-051.summary.json"
    - "process/checks/CP0-CR-051.route-plan.json"
    - "process/docs/product/*"
  must_read:
    - "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
    - "process/state/STATE.current.json"
    - "process/changes/summaries/CR-051.summary.json"
    - "process/checks/CP0-CR-051.route-plan.json"
  read_if_needed:
    - "process/changes/CR-051.md"
    - "meta_flow/workflow/git_branch_lifecycle.py"
    - "meta_flow/workspace/git_sync.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/archive/**"
    - "process/stories/**"
    - "process/quant-lab/**"
    - "../meta-flow-artifacts/docs/**"
---

# CR-051 CP1/CP2 Meta-PM Handoff

## Objective

在不执行真实 artifact 迁移或 Git mutation 的前提下，增量刷新 Meta Flow 产品基线，使其从 CR-050 的“共享 artifact 整仓配对”假设转向“project-first + 每项目独立 worktree”的用户能力，并生成 CP1/CP2 输入。

## Required outcome

- 保留既有 UC/REQ/TC/ST ID 和修订历史；通过 CR-051 新条目或显式 supersede 关系更新基线。
- 把用户已确认的三项方向作为真实 SGQ 证据：project-first、per-project worktree、真实迁移后续逐项目进行。
- 明确能力开发 In Scope 与真实文件/软链接迁移 Out of Scope。
- 把 worktree 常驻策略、main 前进后的分支刷新策略、控制仓/路径用户契约整理为 CP2 待决策项。
- 生成 CP1/CP2 自动结果、讨论证据和 return summary；不得发起 CP2 正式审批或进入设计。

## Reuse key and close condition

- Reuse key: `meta-pm + meta-flow-self-dev + CR-051 + no-story + no-wave`。
- 写完 return summary、CP1 result、CP2 precheck result 与 CP2 决策输入后停止；若遇到阻断性产品歧义，只向 host-orchestrator 返回一组批量问题。
