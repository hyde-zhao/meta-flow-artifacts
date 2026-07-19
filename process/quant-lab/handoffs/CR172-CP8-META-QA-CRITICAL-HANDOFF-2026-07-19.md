---
handoff_id: "HO-CR172-CP8-META-QA-CRITICAL-20260719T013559+0800"
workflow_id: "CR-172"
cr_id: "CR-172"
stage: "CP8_PREPARATION"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
codex_agent_name: "meta-qa-critical"
reasoning_profile: "critical"
dispatch_mode: "subagent-reuse"
status: "returned"
context_ref: "process/context/CP8-CR172-PATH-I-PREPARATION-CONTEXT.json"
created_at: "2026-07-19T01:35:59+08:00"
completed_at: "2026-07-19T02:04:51+08:00"
dispatch_ref: "AD-CR172-CP8-META-QA-CRITICAL-COMPLETED-20260719T020451+0800"
result_ref: "process/checks/CP8-CR172-PATH-I-DELIVERY-READINESS.result.json"
---

# CR-172 PATH-I CP8 Release Readiness Handoff

## 目标

基于 S01～S05 最终独立 CP7 PASS 证据，生成 CR-172 专属质量/发布文档、Release Context、CP8 自动预检结果和待用户审查的人工门禁稿。

## 强制语义

- 推荐结论只能是 `READY_WITH_RISK`，最高正向声明是 `path_i_repository_contract_ready=true`。
- `manual_gate_status=pending`；不得代替用户批准、不得写 `approved_at/approved_by`。
- 不关闭 CR-172 activation，不自动恢复 PATH-C/A；真实六动作授权/执行继续 `0/6`、`0/6`。
- `stage3_started`、`stage3_entry_ready`、`c1_computable`、`real_data_authorized`、`multi_trial_runtime_authorized`、`signal_transport_authorized`、`path_c_or_a_resumed`、`public_c1_projection_ready`、`empirical_r_effective_count_ready` 必须全部为 `false`。
- 只写 context packet 明列的 14 个路径；实现、测试、产品/设计基线、CR、状态、ledger 全部只读或禁止。

## 交回条件

14 个允许产物齐备、结构可解析、CP8 result 可通过基本一致性检查，并明确列出三项待用户决策。若证据不足或发现 blocker，返回 `BLOCKED`/`NEEDS_REWORK`，不得自行放宽 claim ceiling。
