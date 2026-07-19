---
handoff_id: "CR172-TRIAL-RETURN-DEPLOYMENT-META-PM-CORRECTION-R1"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
change_id: "CR-172"
phase: "requirement-clarification"
checkpoint: "CP2"
status: "completed"
created_at: "2026-07-17T15:10:00+08:00"
context_ref: "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
return_summary_path: "process/handoffs/CR172-TRIAL-RETURN-DEPLOYMENT-META-PM-CORRECTION-R1-RETURN-SUMMARY.md"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "user-corrected-storage-authority-and-added-signal-flow"
  agent_path: ".codex/agents/meta-pm.toml"
  tool_name: "followup_task"
  agent_id: "/root/cr172_trial_return_cp2_pm"
  agent_name: "cr172_trial_return_cp2_pm"
  thread_id: "/root/cr172_trial_return_cp2_pm"
  spawned_at: ""
  resumed_at: "2026-07-17T15:10:00+08:00"
  completed_at: "2026-07-17T15:35:06+08:00"
  evidence: "REQ/DQ-009~015=7/7; CR172 scenarios=27/27; CP1/CP2 result-check OK; real sync/pull/signal/runtime=0"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "仅不可从本次用户纠正推导的产品灰区"
  forbidden_question_scope: "正式 CP2、真实 sync/runtime/signal/trading/Git remote 授权"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
  context_ref: "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "field_conflict"
  allowed_reads:
    - "process/changes/summaries/CR-172-TRIAL-RETURN-DEPLOYMENT-DELTA.summary.json"
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "docs/product/RELEASE-SLICES.md"
    - "docs/product/BACKLOG.md"
    - "process/checkpoints/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE.md"
  must_read:
    - "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
    - "process/changes/summaries/CR-172-TRIAL-RETURN-DEPLOYMENT-DELTA.summary.json"
  read_if_needed:
    - "process/changes/CR-172.md"
    - "README.md"
    - "docs/USER-MANUAL.md"
  do_not_read_by_default:
    - "docs/design/**"
    - "process/stories/**"
    - "process/archive/**"
    - "完整会话 transcript"
---

# 用户纠正与新增范围

1. 撤回 `NAS canonical + research-machine staging`。研究机本地 data lake、实验/回测报告和 trial returns 才是运行目录与 active canonical；NAS 是 versioned/hash-verified 同步副本、备份与执行机分发源。
2. 执行机需要离线数据时，从 NAS 拉取到本地 temporary staging，校验 release/manifest/hash 后原子 materialize 为 immutable cache；执行 runtime 不直接读 NAS。
3. DQ/REQ-010～012、相关场景/矩阵/规划/CP1/CP2 证据必须按上述语义修订；`local/NAS dual canonical` 场景改为 `replica stale/partial/hash mismatch/direct-runtime-read`。
4. 新增 DQ/REQ-015：默认交易信号由执行机基于批准策略包与本地行情生成，不在研究机和执行机间传递；日频/EOD 可候选 NAS immutable `SignalBatch/TargetPositionBatch` mailbox；intraday/低延迟传输必须独立实时 transport CR。
5. 低频 SignalBatch 必须覆盖 strategy/package hash、sequence、validity、content hash/signature、source refs、幂等和 ack；不得携带 credential/account secret/broker order command。
6. 修订 8 个产品文档、CP1/CP2 result/checkpoint/context 引用与 return summary；原 DQ-001～008 历史不变，本轮待人工变为 DQ-009～015。真实 sync/pull/signal/runtime 操作仍为 0。
