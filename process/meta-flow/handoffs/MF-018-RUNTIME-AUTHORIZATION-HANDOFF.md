---
handoff_id: "H-MF-018-RUNTIME-AUTHORIZATION"
created_at: "2026-06-21T16:57:51+08:00"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
status: "handoff-created"
stage: "runtime-authorization-preparation"
story_id: "MF-018"
context_ref: "process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "meta-qa"
  canonical_role: "meta-qa"
  codex_agent_name: "meta-qa-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "risk-review"
  agent_path: "delivery/agents/meta-qa.md"
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: "Handoff package only; no runtime validation or subagent execution has started."
  fallback_reason: "Current request asked to create a runtime handoff package and next plan, not to execute external runtime validation."
  approved_by: "user"
  approved_at: "2026-06-21T16:57:51+08:00"
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Runtime authorization scope clarification must be relayed through host-orchestrator."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 正式人工门禁、凭据、安全边界、publish、live / 交易类授权的最终批准"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml"
  context_ref: "process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 8
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml"
    - "process/release/RELEASE-CONTEXT.yaml"
    - "process/checkpoints/CP8-MF-018-DELIVERY-READINESS.md"
  must_read:
    - "process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml"
    - "process/release/RELEASE-CONTEXT.yaml"
    - "process/checkpoints/CP8-MF-018-DELIVERY-READINESS.md"
  read_if_needed:
    - "docs/release/DEPLOY-CHECKLIST.md"
    - "docs/release/ROLLBACK.md"
    - "docs/release/FEEDBACK.md"
    - "process/evidence/MF-018.CP7.index.json"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/changes/CR-*.md"
    - "process/stories/*-LLD.md"
    - "完整会话 transcript"
---

# MF-018 Runtime Authorization Handoff

## 交接目标

为 MF-018 后续外部 runtime 验证准备最小上下文。当前文件只表示交接包已创建，不表示已授权或已执行任何外部操作。

## 边界

| 类别 | 结论 | 说明 |
|---|---|---|
| MF-018 CP8 | approved | 用户已批准 `READY_WITH_RISK` 交付就绪 |
| runtime 操作 | not-authorized | CP8 approve 不授权真实发布、外部 SaaS、凭据、trace upload、publish/live、production 写入或删除 artifact 仓库 |
| 推荐下一步 | runtime scope gate | 先确认 runtime scope，再决定是否创建正式 runtime_authorization CR |

## 必读输入

| 路径 | 用途 |
|---|---|
| `process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml` | 本 handoff 的最小上下文 |
| `process/release/RELEASE-CONTEXT.yaml` | MF-018 发布范围和不授权项 |
| `process/checkpoints/CP8-MF-018-DELIVERY-READINESS.md` | CP8 人工批准和风险接受 |

## 推荐执行计划

| 步骤 | 名称 | 产物 | 进入条件 |
|---|---|---|---|
| RTA-01 | Runtime scope selection | `process/checkpoints/MF-018-RUNTIME-AUTHORIZATION-SCOPE.md` | 用户选择只读 dry-run / credentialed dry-run / true publish-production write |
| RTA-02 | Authorization precheck | `process/checks/MF-018-RUNTIME-AUTHORIZATION-PRECHECK.md` | 明确目标系统、凭据边界、网络/写权限、日志脱敏和停止条件 |
| RTA-03 | Dry-run or sandbox validation | `process/evidence/MF-018.RUNTIME.index.json` | RTA-02 approved 后才允许执行 |
| RTA-04 | Runtime result gate | `process/checkpoints/MF-018-RUNTIME-AUTHORIZATION-RESULT.md` | 验证证据生成后由 host-orchestrator 发起人工确认 |

## 推荐方案

建议下一步选择 `option_a_minimal_readonly`：只做本地 / 只读 / dry-run 能力和边界验证，不读取凭据、不访问外部 SaaS、不写 production。这样能先验证 runtime 包装、证据结构和失败停止策略，再决定是否升级到 credentialed dry-run 或 true publish。

## 不得执行

- 不得读取或记录任何 secret、token、cookie、私钥或凭据。
- 不得发起外部 SaaS 请求、trace upload、publish/live 或 production write。
- 不得删除或替换用户提供的 artifact 仓库。
- 不得把本 handoff 当作 runtime authorization approval。
