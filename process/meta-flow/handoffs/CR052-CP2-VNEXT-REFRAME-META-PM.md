---
handoff_id: "HO-CR052-CP2-VNEXT-REFRAME-META-PM"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
workflow_id: "meta-flow-self-dev"
change_id: "CR-052"
stage: "requirement-clarification"
status: "completed"
created_at: "2026-07-19T08:44:54Z"
semantic: "delegated-user-interaction"
return_summary_path: "process/handoffs/CR052-CP2-VNEXT-REFRAME-META-PM-RETURN-SUMMARY.md"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  canonical_role: "meta-pm"
  codex_agent_name: "pm-zheng"
  reasoning_profile: "default"
  dispatch_trigger: "用户重解释 CR-052 为 Meta Flow vNext 简化治理，旧 CP2 必须 changes_requested 并增量重建产品基线"
  agent_path: "delivery/agents/meta-pm.md"
  tool_name: "spawn_agent"
  agent_id: "/root/cr052_vnext_pm"
  agent_name: "cr052_vnext_pm"
  thread_id: "/root/cr052_vnext_pm"
  spawned_at: "2026-07-19T08:49:30Z"
  completed_at: "2026-07-19T09:15:23Z"
  evidence: "spawn_agent"
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  allowed_question_scope: "仓库边界、长期治理、G0/G1/G2、token budget、迁移兼容的产品灰区；如存在阻断，批量返回 host"
  forbidden_question_scope: "CP2 正式审批、远端 push、凭据、runtime、production、publish/trading"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml"
  read_profile: "minimal"
  max_source_files: 8
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  must_read:
    - "process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml"
    - "process/changes/summaries/CR-052.vnext-reframe.summary.json"
    - "process/REQUEST.md"
    - "process/state/STATE.current.json"
  allowed_reads:
    - "process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml"
    - "process/changes/summaries/CR-052.vnext-reframe.summary.json"
    - "process/REQUEST.md"
    - "process/state/STATE.current.json"
    - "process/docs/product/*"
  read_if_needed:
    - "process/changes/CR-052.md"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/archive/**"
    - "process/stories/**"
    - "旧 CP2 Migration Readiness checkpoint"
    - "完整会话 transcript"
---

# CR-052 vNext 产品基线重构交接

## Objective

在不修改源码、不执行仓库迁移、不 push 的前提下，把 CR-052 产品基线从共享 artifact worktree readiness 重构为每项目双库隔离、长期四层治理、风险分级流程和 Work-scoped 读写检查/token 控制，并重新生成 CP1/CP2 自动预检。

## Required outcomes

1. 更新八份 `process/docs/product/*` 正式对象，保留原修订记录和历史 UC/REQ/TC ID，用 `superseded/reframed/deferred` 明确其当前状态。
2. 至少形成 UC-VNEXT-001..006：项目双库隔离、长期 Roadmap/Phase、Work 生命周期、G0/G1/G2 路由、scoped read/write/check/token、快照迁移与只读历史。
3. 所有 P0/P1 需求必须量化，至少覆盖跨项目变化数=0、路由唯一数=1、两个试点×两个周期、G0/G1 token/read 上限、过程 main expected-OID CAS 和旧共享仓新增写入数=0。
4. 明确第一版不做历史拆分、旧 CP/CR/Story 无损转换、分布式多写者自动 merge、批量项目迁移和未授权远端 publication。
5. 形成 3-4 个 Scenario Gray Areas 和至少 1 条 SGQ，用户当前指令可作为“接受推荐方向并开始正式流程”的证据，但不得伪造成 CP2 全部 DQ 已批准。
6. 生成新的 CP1 result、CP2 precheck result、discussion log/checkpoint 和 return summary；文件名使用 `*-VNEXT-R2*`，不得覆盖旧 v1.1 result。

## Single-writer boundary

- 允许写：`process/docs/product/*`、`process/discussions/CP2-*VNEXT*`、`process/checks/CP1-CR052-*VNEXT*`、`process/checks/CP2-CR052-*VNEXT*`、`process/handoffs/CR052-CP2-VNEXT-REFRAME-META-PM-RETURN-SUMMARY.md`。
- 禁止写：源码、测试、`process/changes/*`、`process/state/*`、`process/checkpoints/*`、`process/context/*`、其他项目路径。
- 只使用 `apply_patch` 编辑文件；Python 运行使用 `uv run --python 3.11`。

## Exit

完成产品基线、自动预检和 return summary 后停止。若出现阻断性产品歧义，写入返回摘要交由 host-orchestrator 统一询问用户；不得自行进入 HLD、Story 或实现。

## Return closure

- 返回状态：`completed`
- 返回时间：`2026-07-19T09:15:23Z`
- 返回摘要：`process/handoffs/CR052-CP2-VNEXT-REFRAME-META-PM-RETURN-SUMMARY.md`
- CP1：`PASS`
- CP2 自动预检：`PASS`
- 产品阻断项：`0`
- 正式 CP2 人工门：仍为 `pending`
