---
handoff_id: "H-CR051-ST-AW-001-CP7-META-QA-CRITICAL"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
workflow_id: "meta-flow-self-dev"
cr_id: "CR-051"
story_id: "ST-AW-001"
wave_id: "QW1"
stage: "verification-execution"
status: "completed-needs-rework"
completed_at: "2026-07-18T12:02:00Z"
created_at: "2026-07-18T11:39:00Z"
context_ref: "process/context/CP7-CR051-VERIFICATION-CONTEXT.yaml"
story_packet_ref: "process/context/stories/STORY-ST-AW-001.CP7.verify-packet.json"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-qa"
  canonical_role: "meta-qa"
  codex_agent_name: "qa-lv"
  reasoning_profile: "critical"
  dispatch_trigger: "CP7-final-verification"
  agent_path: "delivery/agents/meta-qa.md"
  tool_name: "spawn_agent"
  agent_id: "/root/cr051_staw001_qa"
  agent_name: "cr051_staw001_qa"
  thread_id: "/root/cr051_staw001_qa"
  spawned_at: "2026-07-18T11:40:00Z"
  resumed_at: ""
  completed_at: "2026-07-18T11:48:00Z"
  evidence: "attempt 1 independently reproduced two findings and returned them to Host; protocol files were materialized without changing the QA verdict after two evidence-writer timeouts"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "verification blocker or design clarification only; return a structured item to Host"
  forbidden_question_scope: "CP2/CP3/CP5/CP8 正式人工门禁、运行授权、凭据、安全边界、publish、live / 交易类授权"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP7-CR051-VERIFICATION-CONTEXT.yaml"
  context_ref: "process/context/CP7-CR051-VERIFICATION-CONTEXT.yaml"
  story_packet_ref: "process/context/stories/STORY-ST-AW-001.CP7.verify-packet.json"
  read_profile: "minimal"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/context/stories/STORY-ST-AW-001.CP7.verify-packet.json"
    - "process/returns/ST-AW-001.CP6.return.json"
    - "process/evidence/ST-AW-001.CP6.index.json"
    - "process/checks/CP6-CR051-ST-AW-001-project-first-routing-CODING-DONE.result.json"
    - "process/stories/STORY-ST-AW-001-project-first-routing.md"
    - "process/docs/features/cr051-routing/TEST-PLAN.md"
    - "meta_flow/workspace/project_artifact_routing.py"
    - "tests/test_cr051_project_artifact_routing.py"
    - "process/state/STATE.current.json"
    - "process/policies/READ-POLICY.json"
  must_read:
    - "delivery/agents/meta-qa.md"
    - "process/context/stories/STORY-ST-AW-001.CP7.verify-packet.json"
    - "process/returns/ST-AW-001.CP6.return.json"
    - "process/evidence/ST-AW-001.CP6.index.json"
    - "process/checks/CP6-CR051-ST-AW-001-project-first-routing-CODING-DONE.result.json"
  read_if_needed:
    - "process/stories/STORY-ST-AW-001-project-first-routing-LLD.md (先写 deep_review read-log)"
    - "process/stories/STORY-ST-AW-001-project-first-routing-IMPLEMENTATION.md (先写 deep_review read-log)"
    - "meta_flow/workspace/routing.py"
    - "tests/test_workspace_routing.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/changes/CR-*.md"
    - "其他 Story / LLD / IMPLEMENTATION"
    - "process/archive/**"
---

# CR-051 / ST-AW-001 CP7 独立验证交接

## 目标

独立证明 project-first artifact routing 实现满足 Story、Feature、LLD 和 TC-AW-001/002/003/010/012，且所有歧义与不可信路径在 mutation 前 fail closed；给出 Story 级 CP7 结论。

## 执行边界

- 本 lane 只验证，不修改 `meta_flow/**`、`tests/**` 或交付实现；发现缺陷返回 `NEEDS_REWORK`，不得顺手修复。
- 不执行真实 artifact 文件迁移、软链接修改、Git/worktree/ref/remote mutation、commit、push、publish 或 main-sync；仅允许 fixture、临时目录和只读检查。
- 先执行 `meta-flow workspace check --project-root .`；不得修复或重建 process 路由。
- 完整 LLD/IMPLEMENTATION 是 deny-default。确需展开时，先分别执行 `meta-flow context read-log ... --reason deep_review --stage CP7 --agent meta-qa --context-ref process/context/stories/STORY-ST-AW-001.CP7.verify-packet.json --project-root .`。
- 仓库当前缺少 canonical `docs/design/MODULE-BOUNDARIES.yaml`。必须区分“检查不可适用”与实现缺陷，不得把缺失 manifest 伪报为通过；在结论中显式路由剩余风险。

## 必须交付

1. `process/docs/quality/CR051-ST-AW-001-VERIFICATION-REPORT.md`：验证范围、对象、REQ/TC/AC/设计契约追踪矩阵、分层验证证据、剩余风险和阶段决策。
2. `process/docs/quality/CR051-ST-AW-001-TEST-REPORT.md` 与 `process/docs/quality/CR051-ST-AW-001-REVIEW.md`；若无修复项，`FIXES.md` 明确 N/A。
3. `process/returns/ST-AW-001.CP7.return.json`，通过 `meta-flow story return-check`。
4. `process/evidence/ST-AW-001.CP7.index.json`，通过 evidence check。
5. 结论只能使用 `PASS`、`PASS_WITH_RISK`、`NEEDS_REWORK`、`NEEDS_DESIGN_CLARIFICATION`；真实跨设备 pilot 未授权时不得用推测代替证据。

## 退出条件

- 五项量化 AC 与适用 TC-AW 100% 建立证据引用；负向、边界、回归、portable digest、零 sibling/mutation 均有可复跑证据。
- 定向测试、ruff、py_compile、diff、return/evidence 校验通过；模块边界/import 检查按实际适用性记录。
- QA 输出未触碰实现源码、测试或任何真实 Git/worktree/link/remote 状态。
