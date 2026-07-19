---
handoff_id: "HO-CR051-CP3-CP4-META-SE"
from_agent: "host-orchestrator"
to_agent: "meta-se"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "story-planning"
status: "interrupted-split-dispatch"
created_at: "2026-07-18T05:56:00Z"
context_ref: "process/context/CP4-CR051-STORY-PLANNING-CONTEXT.yaml"
upstream_gate_ref: "process/checkpoints/CP3-CR051-ARTIFACT-WORKTREE-HLD-REVIEW.md"
dispatch:
  mode: "subagent"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "post-CP3 Story/Feature planning for cross-module Git state machine and CP5 design queue"
  tool_name: "followup_task"
  agent_id: "/root/cr051_meta_se_cp3_finish"
  thread_id: "/root/cr051_meta_se_cp3_finish"
  resumed_at: "2026-07-18T06:01:20Z"
  completed_at: "2026-07-18T06:11:18Z"
---

# CR-051 CP3 → CP4 Meta-SE 交接

## 目标

基于已确认的 project-first artifact worktree 架构，完成五个 Feature 的实现设计适用性判定、所有 required Feature 的 DESIGN/TEST-PLAN/TASKS、ST-AW-001..005 正式 Story 卡片、稳定 DAG/Wave/文件所有权，以及 CP4 自动预检。只做设计规划，不进入 LLD 或实现。

## 必须使用的 Skill

- `implementation-design`
- `story-manager`
- `dependency-mapper`
- `dag-validator`
- `wave-planner`
- `checkpoint-manager`（仅 CP4 自动预检）

## 固定约束

1. Story ID/数量固定为 `ST-AW-001..005` 共 5 个，不新增、不重编号；Feature ID 固定为 `FEAT-AW-01..05`。
2. 五个 Feature 均按 HLD 判定为 `required` 并生成三件套；如代码事实要求豁免，必须作为 blocking finding 交还，不得自行降级。
3. 高风险 Story 命中状态机、并发、持久化、远端 ref、跨 Story 契约，默认 `full-lld`。ST-AW-005 只有在纯只读迁移 handoff、无 schema/迁移 mutation/跨 Story contract 时才可 `technical-note`；否则升级 `full-lld`。
4. O-AW-01/02 必须进入 FEAT-AW-02 DESIGN/TEST-PLAN/TASKS 和相关 Story lld_policy/AC；O-AW-03 进入 FEAT-AW-05 的运维指标与 follow-up 路由，不允许自动同步。
5. `process/DEVELOPMENT-PLAN.yaml` 是当前 active CR 的 Story 真相源，可从 CR-050 delivered 计划切换为 CR-051，但不得删除 CR-050 的历史 Story 文件/检查结果。
6. 明确 primary/shared/merge_owner/forbidden 文件；共享 lifecycle/schema/CLI 文件存在冲突时开发必须串行或拆出明确 contract owner，LLD 写作可按冻结契约并行。
7. CP4 只自动检查，不创建人工 checkpoint；通过后交还 Host，由 Host 计算全量 LLD 队列。
8. 禁止修改源码、测试、人工 checkpoint、CR、STATE 或任何真实 Git/worktree/link/ref/remote；禁止进入 CP5 设计证据写作。

## 期望交还

- 变更/新增文件清单。
- 5 Feature / 5 Story 的矩阵、DAG、Wave、file ownership 和 lld_policy 摘要。
- CP4 result/summary 及 `meta-flow cp result-check`、`meta-flow story plan-check`、DAG/结构校验结果。
- blocking clarification、non-blocking OPEN/Spike 和 CP5 设计证据队列建议。
- `process/handoffs/CR051-CP4-META-SE-RETURN-SUMMARY.md`。

## 调度收敛记录

- 原复用线程已完成 Skill/基线读取，但在规定窗口内未产生首批文件，Host 于 `2026-07-18T06:11:18Z` 中断。
- 为遵守单写与工作流健康阈值，任务拆为 core planning、FEAT-AW-01/02 packs、FEAT-AW-03/04/05 packs 三个互斥写入 lane；本 handoff 不再作为完成证据。
