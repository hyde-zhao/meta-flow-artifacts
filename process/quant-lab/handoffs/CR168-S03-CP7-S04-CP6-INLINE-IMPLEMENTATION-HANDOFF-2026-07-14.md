---
handoff_id: "CR168-S03-CP7-S04-CP6-INLINE-IMPLEMENTATION-HANDOFF-2026-07-14"
cr_id: "CR-168"
from: "host-orchestrator"
to: "host-orchestrator-inline-meta-dev"
stage: "CP6"
story_id: "CR168-S04-gate4-projection-containment"
created_at: "2026-07-14T13:58:00+08:00"
context_ref: "process/context/stories/STORY-CR168-S04-gate4-projection-containment.CP6.work-packet.json"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "inline-fallback"
  platform: "codex"
  agent_role: "meta-dev"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev"
  reasoning_profile: "medium"
  dispatch_trigger: "S03-CP7-pass-with-accepted-inline-risk"
  tool_name: "host-orchestrator-inline"
  evidence: "user-approved-inline-fallback"
  fallback_reason: "用户明确要求不拉起子 Agent；S03 CP7 已通过，在 CP5 已批准的 S04 文件所有权内执行。"
  approved_by: "user"
question_permission:
  can_ask_user: false
  mode: "none"
  forbidden_question_scope: "CP2/CP3/CP5/CP8 门禁、运行授权、凭据、安全边界、真实数据、publish、live/trading"
  broker_agent: "host-orchestrator"
---

# CR-168 S03 CP7 → S04 CP6 内联实施交接

S04 只能创建局部 adapter 与测试。它必须只投影四个 C3 字段；C4 `typed_unavailable` 时 ref/reason key 都不得存在。固定使用 `candidate-release` 调 public Gate4 validator，并只接受精确的 `adv_participation_missing`、`capacity_dollars_missing`、`liquidity_sizing_missing` 三项 BLOCKED claims。

不得修改 canonical Gate4 或 aggregate orchestration；不得运行时依赖 private helper、使用 monkeypatch，或在任何路径外传 PASS。任何需要 C4 present、全局修复或 aggregate 的请求都停止并路由 FU-005/FU-007。
