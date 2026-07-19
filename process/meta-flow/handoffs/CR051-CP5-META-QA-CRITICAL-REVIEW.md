---
handoff_id: "HO-CR051-CP5-META-QA-CRITICAL-REVIEW"
from_agent: "host-orchestrator"
to_agent: "meta-qa"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "story-design-review"
status: "returned-with-findings"
created_at: "2026-07-18T07:13:30Z"
context_ref: "process/context/CP5-CR051-LLD-CONTEXT.yaml"
story_ids: ["ST-AW-001", "ST-AW-002", "ST-AW-003", "ST-AW-004", "ST-AW-005"]
dispatch:
  mode: "subagent"
  canonical_role: "meta-qa"
  codex_agent_name: "meta-qa-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "CP5 全量设计证据已生成；高风险 Git/worktree 状态机、权限边界与最终 CP5 独立复核"
  tool_name: "followup_task"
  agent_id: "/root/cr051_meta_se_cp3"
  thread_id: "/root/cr051_meta_se_cp3"
  resumed_at: "2026-07-18T07:14:15Z"
  completed_at: "2026-07-18T07:26:00Z"
---

# CR-051 CP5 Meta-QA Critical Independent Review

## 任务

对 CR-051 的 4 份 full LLD、1 份 technical-note、5 份单 Story CP5 result，以及 CP4 DAG/Feature contracts 做独立设计质量复核；这不是 CP6/CP7 实现验证。

## 必须读取

1. `process/context/CP5-CR051-LLD-CONTEXT.yaml`
2. `process/checks/CP4-CR051-STORY-DAG-PARALLEL-SAFETY.result.json`
3. `process/DEVELOPMENT-PLAN.yaml`
4. 5 张 Story 卡和各自 `feature_design_refs`
5. 4 份 `process/stories/STORY-ST-AW-00{1,2,3,4}-*-LLD.md`
6. ST-AW-005 卡内 `## 技术说明`
7. 5 份单 Story `CP5-CR051-ST-AW-00*-*IMPLEMENTABILITY.result.json`
8. `quality-review`、`review-artifact-protocol` 及 findings/summary 模板和 validator

读取 deny-default 的 LLD/plan 前，用 `meta-flow context read-log` 记录真实 read-expansion event；不得虚构 ID。

## 必查不变量

- ST-AW-001：portable project-first route、唯一 owned write target、legacy 显式模式。
- ST-AW-002 / O-AW-01：512MiB 只在 bounded profile 校准证明满足 `false-safe=0`、`underestimate=0` 时可用；证明失败自动 switch 关闭；CAP fixture 不可豁免。
- ST-AW-002 / O-AW-02：外置 durable store、file fsync→atomic replace→parent dir fsync→readback/seal；ENOSPC/EACCES/fsync/replace/torn/kill/cross-device 在 Git mutation 前 fail closed；DUR fixture 不可豁免。
- ST-AW-003：source leg 完成到 source default；artifact leg 完成到 project integration；artifact main/default/control/sibling mutation=0；typed authz/fresh OID/result 单写。
- ST-AW-004：matching source+artifact 2/2 PASS only；16 组合；PARTIAL 非 terminal；aggregate Git/worktree 调用=0；projection 单写。
- ST-AW-005 / O-AW-03：纯只读 manifest；三阈值仅生成去重后续 CR candidate；任何 mutation 都升级 full-lld 并重开设计。
- 跨 Story：DAG、file owner、LegResult/AggregateResult、CLI merge owner 和 clarification queue 一致；CP5 approve 不授权真实 Git/worktree/ref/remote/link/migration/main-sync、commit/push/publish。

## 只允许写入

- `process/docs/quality/CR051-CP5-INDEPENDENT-REVIEW-FINDINGS.md`
- `process/docs/quality/CR051-CP5-INDEPENDENT-REVIEW-SUMMARY.md`
- `process/handoffs/CR051-CP5-META-QA-CRITICAL-REVIEW-RETURN.md`

findings/summary 必须使用共享模板结构并运行 validator；findings 优先、严重度明确。若无问题也必须写“0 findings”和剩余风险/CP6-CP7 proof obligations。不得修改 LLD、Story、Feature、plan、STATE、ledger、checkpoint、源码、测试或任何真实 Git/worktree 状态。

## 退出条件

- 两个 review artifact validator 均 `OK`；
- 结论只能为 `proceed | revise | escalate`；
- blocker/required/optional、用户是否需新增决策、CP5 推荐动作和不授权边界明确；
- return 中列出 read-expansion refs、findings、剩余风险与下一步。
