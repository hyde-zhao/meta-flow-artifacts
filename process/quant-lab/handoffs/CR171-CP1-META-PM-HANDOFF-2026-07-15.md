---
handoff_id: "CR171-CP1-META-PM-2026-07-15"
from_role: "host-orchestrator"
to_role: "meta-pm"
canonical_role: "meta-pm"
workflow_id: "CR-171"
change_id: "CR-171"
stage: "requirement-clarification"
created_at: "2026-07-15T21:08:30+08:00"
status: "completed"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "phase-default"
  agent_path: ".agents/agents/meta-pm.md"
  tool_name: "spawn_agent"
  agent_id: "/root/pm_wu_cr171"
  agent_name: "pm_wu_cr171"
  thread_id: "/root/pm_wu_cr171"
  spawned_at: "2026-07-15T21:09:00+08:00"
  resumed_at: ""
  completed_at: "2026-07-15T21:35:00+08:00"
  evidence: "spawn_agent+agent return"
  fallback_reason: ""
  approved_by: "user"
  approved_at: "2026-07-15T21:00:00+08:00"
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Only unresolved CP1 scenario/requirement ambiguity; return any question as a concise host-relay item."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 formal gate, runtime authorization, credentials, security boundary, publish, live or trading authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP1-CR171.context.json"
  context_ref: "process/context/CP1-CR171.context.json"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 13
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "CP1 must incrementally update the listed product baseline documents; read-log each target before full read."
  allowed_reads:
    - "process/context/CP1-CR171.context.json"
    - "process/changes/summaries/CR-171.summary.json"
    - "process/plans/STAGE3-LAUNCH-DECISION-MEMO-2026-07-15.yaml"
    - "process/policies/RISK-ID-ALIASES.yaml"
    - "process/checks/CR171-STAGE3-LAUNCH-CONFLICT-PRECHECK.json"
  must_read:
    - "process/context/CP1-CR171.context.json"
    - "process/changes/summaries/CR-171.summary.json"
    - "process/plans/STAGE3-LAUNCH-DECISION-MEMO-2026-07-15.yaml"
  read_if_needed:
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "docs/product/RELEASE-SLICES.md"
    - "docs/product/BACKLOG.md"
    - "docs/components/MULTIFACTOR-RESEARCH.md"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/archive/**"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/changes/CR-*.md"
    - "process/stories/**"
---

# CR-171 CP1 Requirement Clarification Dispatch

## 任务

以 CR-171 的 CP1 需求澄清代理身份，生成 **增量** Stage 3 Launch / Real-Lake Entry Decision Gate 产品基线和 CP1 完整性结果。用户已经授权创建 CR-171 并自动推进到下一人工门禁；这不授权任何真实数据读取、真实 computation、runtime、provider/NAS 操作、凭据访问、写入、publish 或 trading。

## 已冻结事实与边界

- CP2 仍须由 Host Orchestrator 发起人工门禁；不得将路线、verifier 或 read scope 当成已批准。
- 推荐方案仅作为 CP2 recommendation：`C1-C4 real-producer + event-bounded FU-006 waiver`，由此为 2 个入口 CR；current runner 为 1 个入口 CR，FU-006 first 则 3 个。
- waiver 的机械失效点是：首个 real-evidence admission 决策可为 PASS/PASS_WITH_RISK 前，或 Stage 3 exit gate 可启动前。
- revalidation 只判定、标注、报告，允许 verdict 为 `reaffirmed_as_legacy_only`、`insufficient_for_current_entry`、`incompatible_rework_required`；任何修复另立 CR。
- `MULTIFACTOR-RESEARCH.md:126` 需从无条件 Stage 3 出口表述改为保留历史事实的 legacy/require-revalidation 表述，不能声称当前 entry-ready。
- CR-010/018 与数据湖主题强邻接，但 CR-171 不能重开其 provider/lake-write/publish/current-pointer/production-close 行为；CR-032 是状态冲突，只披露不修。
- 新风险消费者使用 `R-CR170-RUNNER-GAP`，历史 `R-CR170-STAGE3-OVERCLAIM` 只经 policy alias 解析，不能重写 CR-170。

## 允许写入

- `docs/product/USE-CASES.md`, `REQUIREMENTS.md`, `SCENARIOS.yaml`, `TEST-MATRIX.md`, `STORY-MAP.md`, `MVP-SCOPE.md`, `RELEASE-SLICES.md`, `BACKLOG.md`：只做增量更新并保留历史基线和修订记录。
- `docs/components/MULTIFACTOR-RESEARCH.md`：只增加 legacy/require-revalidation 叙事标记和修订记录。
- `process/discussions/CP2-CR171-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP1-CR171-USE-CASE-COMPLETENESS.result.json`、`process/checks/CR171-PRODUCT-BASELINE-EVIDENCE.json`、`process/handoffs/CR171-CP1-META-PM-RETURN-SUMMARY.md`。

## 禁止写入与完成标准

- 不得改源码、real lake/NAS、credentials、环境变量、provider、catalog、current pointer、runtime/trading、CR-010/018/032 的正式对象，也不得创建实现 Story/LLD。
- 对每个全文产品文档读取先写 `meta-flow context read-log`；把这些 event IDs 放进 CP1 result 的 `read_expansion_refs`。
- CP1 result 必须满足 `meta-flow cp result-check --check-consistency`，并清楚指出 CP2 的三项正式决策和“CP8 成功不等于 entry-ready”。
- 若没有真正阻断的歧义，不向用户提问；直接以冻结 memo 中的 recommended/alternative/trigger 形成 CP2 候选。若有阻断，返回 host-relay 的单个明确问题。
