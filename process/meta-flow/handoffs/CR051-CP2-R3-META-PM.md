---
handoff_id: "HO-CR051-CP2-R3-META-PM"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "requirement-clarification"
status: "completed"
created_at: "2026-07-18T02:55:21Z"
completed_at: "2026-07-18T03:19:17Z"
semantic: "changes-requested-r3-product-baseline-revision"
return_summary_path: "process/handoffs/CR051-CP2-R3-META-PM-RETURN-SUMMARY.md"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "user-approved R3 heterogeneous dual-leg and integration-bootstrap product revision"
  tool_name: "spawn_agent"
  agent_id: "/root/cr051_meta_pm_r3"
  agent_name: "cr051_meta_pm_r3"
  thread_id: "/root/cr051_meta_pm_r3"
  attempt_id: "ATTEMPT-CR051-CP2-R3-PM-01"
  spawned_at: "2026-07-18T03:00:07Z"
  completed_at: "2026-07-18T03:19:17Z"
  terminal_result: "PASS"
  result_ref: "process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R3.result.json"
  return_ref: "process/handoffs/CR051-CP2-R3-META-PM-RETURN-SUMMARY.md"
  evidence: "spawn_agent"
context_policy:
  capsule_first: true
  context_ref: "process/context/CP2-CR051-R3-REVISION.context.json"
  read_profile: "compact"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
---

# CR-051 CP2 R3 Meta-PM Handoff

## Objective

在不新增 CR、不改写 R1/R2 历史正文、不执行任何真实 Git/worktree/link/remote mutation 的前提下，增量修订八份产品基线和 CP2 场景讨论证据，使当前真相明确表达“一个逻辑 CR、异构 source/artifact 双 leg、单一聚合门、project integration create-only 初始化、shared main 同步完全在 CR 外人工执行”。

## Canonical decision input

- `process/checks/CP2-CR051-R3-USER-DECISIONS.json`
- R2 的 `CP2-DQ-02` 必须保留历史并标记 `superseded-by-user`；当前语义由 `CP2-DQ-04..06` 承接。
- 用户已回复“按照这个方案，实施”，上述 R3 决策均为 `resolved-by-user`，不得重新包装成待人工决策。

## Allowed writes（single writer）

- `process/docs/product/USE-CASES.md`
- `process/docs/product/REQUIREMENTS.md`
- `process/docs/product/SCENARIOS.yaml`
- `process/docs/product/TEST-MATRIX.md`
- `process/docs/product/STORY-MAP.md`
- `process/docs/product/MVP-SCOPE.md`
- `process/docs/product/RELEASE-SLICES.md`
- `process/docs/product/BACKLOG.md`
- `process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md`
- `process/checks/CP2-CR051-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP1-CR051-USE-CASE-COMPLETENESS-R3.result.json` 及 summary
- `process/checks/CP2-CR051-REQUIREMENTS-BASELINE-R3.result.json` 及 summary
- `process/handoffs/CR051-CP2-R3-META-PM-RETURN-SUMMARY.md`

禁止修改 CR、STATE/current、任何 ledger、人工 checkpoint、context capsule、源码、测试和 sibling project 内容；这些由 Host Orchestrator 或后续阶段单写。

`process/discussions/**` 属于 deny-default。为保留旧日志而读取该文件前，必须使用 `meta-flow context read-log` 记录 `human_audit`；写入权限不等于默认读取权限。

## Required product semantics

1. source leg：从源码仓 fresh `main/master` 创建 CR branch，完成后进入该源码仓默认分支。
2. artifact leg：从 fresh `projects/<project>/integration` 创建短期 CR branch，只进入同一项目 integration；不 refresh、不 merge、不直接更新 artifact shared `main`。
3. shared `main` ↔ project integration 双向同步属于 CR 外人工维护；main/integration divergence 不是单个 CR blocker，integration expected OID drift 才阻断 artifact finish，禁止自动 merge/rebase/force。
4. 一个逻辑 CR 的必需 legs 独立出结果，由单一协调者聚合；优先级 `BLOCKED > FAIL > IN_PROGRESS > PASS`，仅全 PASS 才整体完成；`PARTIAL` 仅为 progress/effect；失败不自动关闭 CR，也不回滚已成功 leg。
5. project integration 仅在远端 ref 不存在时从 fresh `origin/main` exact OID create-only 初始化；存在时禁止 recreate/reset/orphan。人工 main→integration 同步默认要求该项目无活跃 artifact CR。
6. leg result 与 aggregate gate 需共享 CR ID/attempt correlation，但 ledger 单写、receipt/OID 防自引用的精确方案留 CP3。
7. shared-artifact 模式必须显式覆盖 CR-050 的 paired-default 适用前提；不得让旧 `REQ-GB-011..014` 在该模式下继续要求 artifact default/main merge。

## Revision constraints

- 保持 UC-AW=5、REQ-AW=27、TC-AW=15、ST-AW=5、SL-AW=4、SGA-AW=4；优先修订既有条目，不新增 ID。若机器校验表明无法保持，停止并在 return 中报告，不自行改变计数。
- 八份正式文档全部追加修订记录；保留旧基线追溯，但当前生效章节不得残留 per-CR merge-main refresh。
- CP1 R3 必须重新检查场景完备性；CP2 R3 result 使用新的 checkpoint_id，supersedes R2 result，blockers=0 才可交还。
- 所有 `Q/OPEN/权限/风险/运行授权/publish` 项必须分类；预计 pending decision items=0，真实 Git/remote/migration 均为 non-authorized。

## Exit criteria

- 八份产品文档、discussion log/checkpoint 对 R3 语义一致，YAML 与 Markdown 结构可解析。
- CP1 R3、CP2 R3 result 均通过 `meta-flow cp result-check`；相关 summary 可读。
- `git diff --check`、ID/计数、陈旧 refresh 扫描和“不越过 allowed writes”检查通过。
- return summary 列出修改文件、计数、resolved/superseded DQ、剩余 CP3 细化项和验证命令。
- 不创建人工 checkpoint、不进入 CP3、不拆 Story、不修改源码、不执行任何真实 Git/worktree/link/remote mutation。
