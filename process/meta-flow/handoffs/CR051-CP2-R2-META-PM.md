---
handoff_id: "HO-CR051-CP2-R2-META-PM"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "requirement-clarification"
status: "completed"
created_at: "2026-07-17T08:44:20Z"
semantic: "changes-requested-revision"
return_summary_path: "process/handoffs/CR051-CP2-R2-META-PM-RETURN-SUMMARY.md"
dispatch:
  required: true
  semantic: "changes-requested-revision"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "CP2 changes_requested: adopt durable per-project integration branch and namespaced short-lived CR branches"
  agent_path: "delivery/agents/meta-pm.md"
  tool_name: "followup_task"
  agent_id: "/root/cr051_meta_pm"
  agent_name: "cr051_meta_pm"
  thread_id: "/root/cr051_meta_pm"
  attempt_id: "ATTEMPT-CR051-CP2-R2-PM-02"
  spawned_at: ""
  resumed_at: "2026-07-17T08:56:21Z"
  completed_at: "2026-07-17T09:02:59Z"
  evidence: "followup_task"
  fallback_reason: ""
  prior_attempt: "ATTEMPT-CR051-CP2-R2-PM-01 interrupted by host to constrain excessive pre-write analysis; no product file write had occurred"
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "only blocking ambiguity in applying the explicit CP2 branch-model decision"
  forbidden_question_scope: "CP2 formal approval, CP3 architecture approval, runtime authorization, credentials, repository publication, real artifact migration or real Git mutation"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
  context_ref: "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
  read_profile: "compact"
  max_source_files: 14
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  allowed_reads:
    - "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
    - "process/state/STATE.current.json"
    - "process/changes/summaries/CR-051.summary.json"
    - "process/checks/CP0-CR-051.route-plan.json"
    - "process/handoffs/CR051-CP2-R2-META-PM.md"
    - "process/checkpoints/CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE.md"
    - "process/docs/product/*"
    - "process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md"
    - "process/checks/CP2-CR051-DISCUSSION-CHECKPOINT.json"
  must_read:
    - "process/context/CP2-CR051-REQUIREMENT-CONTEXT.yaml"
    - "process/handoffs/CR051-CP2-R2-META-PM.md"
    - "process/changes/summaries/CR-051.summary.json"
  read_if_needed:
    - "process/changes/CR-051.md"
    - "meta_flow/workflow/git_branch_lifecycle.py"
    - "meta_flow/workspace/git_sync.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/archive/**"
    - "process/stories/**"
    - "../meta-flow-artifacts/process/quant-lab/**"
    - "../meta-flow-artifacts/docs/**"
---

# CR-051 CP2 R2 Meta-PM Handoff

## 用户已明确的修订

- CP2-DQ-01：采用每项目长期 `projects/<project-name>/integration` 分支；项目 worktree 空闲时驻留该分支；每个 CR 使用 `projects/<project-name>/cr/<cr-id>-<slug>` 短期分支；保留 `main` 作为共享集成基线。
- CP2-DQ-02：接受推荐方案——显式 merge fresh `origin/main`，冲突 fail closed，禁止静默 rebase/force/自动解冲突。
- CP2-DQ-03：接受推荐方案——保留现有 control checkout，项目 worktree 位于可配置 sibling root，采用 project namespace/sparse policy 与 owned-path gate。

## Objective

在不改变既有 ID 和历史正文的前提下，增量修订 CR-051 产品基线，使分支模型从“persistent worktree + idle detached”变为“persistent worktree + durable project integration branch + namespaced short-lived CR branch + shared main integration baseline”。

## Required outcome

- 更新八份 `process/docs/product/**` 中与 branch lifecycle、idle state、finish、abort、refresh、branch naming 和 main 职责相关的 CR-051 条目；不整体替换文档。
- 更新 CP2 discussion log/checkpoint，记录本次用户回答、三项 DQ 的 resolved-by-user 状态与影响面。
- 生成 `process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R2.result.json` 和对应 summary；使用 `check_attempt=2`、`supersedes_result_ref` 指向 R1 result。
- 生成 `process/handoffs/CR051-CP2-R2-META-PM-RETURN-SUMMARY.md`，列出精确修改文件、需求/场景/Story ID 变化、剩余开放项和验证结果。
- CP1 场景数量若未变化，不重写 CP1 result；说明其继续有效。
- 不创建 CP2 人工 checkpoint；由 Host Orchestrator 生成 R2。
- 不启动 CP3，不拆 Story，不修改源码，不执行任何真实 Git/worktree/link/remote mutation。

## Exit criteria

- 产品基线一致表达长期项目 integration 分支、短期 CR 分支和共享 `main`。
- DQ-01..03 均有 resolved-by-user 证据，但 CP2 总体门仍待用户对 R2 基线回复 `approve`。
- R2 CP2 result 可通过 `meta-flow cp result-check`。
