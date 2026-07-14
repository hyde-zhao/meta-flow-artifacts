---
handoff_id: "CR168-S02-CP6-CP7-INLINE-VERIFICATION-HANDOFF-2026-07-14"
cr_id: "CR-168"
from: "host-orchestrator"
to: "host-orchestrator-inline-meta-qa"
stage: "CP7"
story_id: "CR168-S02-deterministic-cost-producer"
created_at: "2026-07-14T13:06:00+08:00"
context_ref: "process/context/stories/STORY-CR168-S02-deterministic-cost-producer.CP7.verify-packet.json"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "inline-fallback"
  platform: "codex"
  agent_role: "meta-qa"
  canonical_role: "meta-qa"
  codex_agent_name: "meta-qa"
  reasoning_profile: "default"
  dispatch_trigger: "S02-CP6-PASS"
  tool_name: "host-orchestrator-inline"
  agent_id: ""
  thread_id: ""
  spawned_at: ""
  completed_at: ""
  evidence: "user-approved-inline-fallback"
  fallback_reason: "用户明确要求不拉起子 Agent；CP5 DQ-004 已接受 inline CP7 verification，并要求 CP8 披露独立性限制。"
  approved_by: "user"
  approved_at: "2026-07-14T10:57:44+08:00"
question_permission:
  can_ask_user: false
  mode: "none"
  structured_choice_allowed: false
  allowed_question_scope: ""
  forbidden_question_scope: "CP2/CP3/CP5/CP8 门禁、运行授权、凭据、安全边界、真实数据、publish、live/trading"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  context_ref: "process/context/stories/STORY-CR168-S02-deterministic-cost-producer.CP7.verify-packet.json"
  story_packet_ref: "process/context/stories/STORY-CR168-S02-deterministic-cost-producer.CP7.verify-packet.json"
  read_profile: "compact"
  max_source_files: 5
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "S02 implementation evidence and numeric LLD must be read to verify exact arithmetic and boundary assertions."
  allowed_reads:
    - "process/context/stories/STORY-CR168-S02-deterministic-cost-producer.CP7.verify-packet.json"
    - "process/checks/CP6-CR168-S02-DETERMINISTIC-COST-PRODUCER.result.json"
    - "process/returns/STORY-CR168-S02-deterministic-cost-producer.CP6.return.json"
    - "process/stories/STORY-CR168-S02-deterministic-cost-producer-IMPLEMENTATION.md"
    - "process/stories/STORY-CR168-S02-deterministic-cost-producer-LLD.md"
  must_read:
    - "process/checks/CP6-CR168-S02-DETERMINISTIC-COST-PRODUCER.result.json"
    - "process/returns/STORY-CR168-S02-deterministic-cost-producer.CP6.return.json"
    - "process/stories/STORY-CR168-S02-deterministic-cost-producer-IMPLEMENTATION.md"
    - "process/stories/STORY-CR168-S02-deterministic-cost-producer-LLD.md"
  read_if_needed:
    - "engine/economic_cost_calculator.py"
    - "engine/economic_cost_evidence.py"
  do_not_read_by_default:
    - "process/archive/**"
    - "process/discussions/**"
    - "all S03-S05 implementation/verification artifacts"
    - "docs/quality/**"
---

# CR-168 S02 CP6→CP7 内联验证交接

S02 已完成 CP6。验证只检查其 static-only numeric/projection producer 合同与 C2 compatibility subset；不得运行真实数据、runtime、registry、C4、Gate4 或 aggregate 路径。

## 必检断言

- 五个分项使用 LLD §8 的 exact basis；gross-to-net 及 `net_return` 可重算。
- `proxy=0/1/>1` 均可算，`>1` 不产生 capacity claim；zero/negative reference fail-closed。
- issues 非空时 calculator invocation=0；public producer 不接收 caller-declared validated mapping。
- component hash 不含 family-1 identity，篡改不 false PASS。

## 回传

产出 CP7 return/evidence/result。inline verifier 风险维持为 non-blocking，并在全 CR 的 CP8 Decision Brief 统一披露。
