---
handoff_id: "CR050-CP2-CP3-META-SE"
workflow_id: "meta-flow-self-dev"
change_id: "CR-050"
from_agent: "host-orchestrator"
to_agent: "meta-se"
stage: "solution-design"
context_ref: "process/context/CP2-CR050-GIT-BRANCH-CONTEXT.yaml"
status: "returned"
returned_at: "2026-07-15T16:27:30Z"
return_context_ref: "process/context/CP3-CR050-GIT-BRANCH-DESIGN-CONTEXT.yaml"
return_result_ref: "process/checks/CP3-CR050-GIT-BRANCH-HLD-CONSISTENCY.result.json"
dispatch:
  mode: "inline-fallback"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "CP2 approved; freeze cross-repository Git ref lifecycle, destructive cleanup proof and security boundaries"
  tool_name: "host-orchestrator-inline"
  approved_by: "user"
  approved_at: "2026-07-13T03:40:39Z"
  fallback_reason: "User explicitly instructed: 不使用子agent，继续推进."
  started_at: "2026-07-15T16:11:32Z"
---

# CR-050 CP2 → CP3 meta-se Handoff

## 目标

把已批准的 Git CR 分支产品契约转化为可评审的 Blueprint、Domain Map、Dependency Map、HLD 和 ADR；生成 CP3 自动预检与人工 Decision Brief。不得进入 Story 拆解或代码实现。

## 已批准输入

- CP2-DQ-01：原生 Git subprocess，不安装 `gb`/Git Town。
- CP2-DQ-02：project 与 artifact 仓使用同名 CR 分支；逐仓报告，不声称原子事务。
- CP2-DQ-03：`finish` 不执行 merge，只验证外部 merge 后清理。
- CP2-DQ-04：无法以 ancestry 证明时 fail closed；未来 forge receipt adapter 才能扩展。
- CP2-DQ-05：`publish` 只推送调用前已经存在的 commits，不 stage/commit/amend。

## 必须形成的架构约束

1. 单一 CLI/service 边界复用 `meta_flow/workspace/git_sync.py`，不得引入第二套 Git 执行器。
2. `open`、`publish`、`finish` 使用显式状态和逐仓 step result；所有动作有 `--dry-run`。
3. destructive cleanup 必须绑定 exact repository identity、remote、branch、observed tip、merge target 与 proof strategy。
4. `finish` 必须先完成所有仓的非破坏性 preflight，再进入逐仓删除；网络故障仍可能导致 partial success，必须可重试且不 force。
5. protected/default branches、remote ref drift、dirty/uncommitted tree、diverged default branch、non-ancestry 和 unknown repository 必须 fail closed。
6. HLD 必须明确 CLI 与 Host Orchestrator/CR lifecycle 的集成时机，但本 CR 不自动改写用户 commit，不自动 merge，不接 forge API。

## 不授权

- source implementation、Story decomposition、LLD、repository commit/push、真实 branch create/delete。
- auto merge、force push/delete、rebase/history rewrite、自动冲突解决。
- credentials、runtime/SaaS、production write、trading、托管平台 API。
- `process/quant-lab/**` 与 prelink backup。

## Context Policy

- `read_profile=compact`，先读 CP2 capsule。
- 产品基线全文只因 `deep_review` 展开，并写入 READ-EXPANSION ledger。
- 默认禁止其他 CR 全文、archive、历史 discussion、全量 LLD、完整 transcript。

## 预期输出

- `process/docs/design/CR050-GIT-BRANCH-BLUEPRINT.md`
- `process/docs/design/CR050-GIT-BRANCH-DOMAIN-MAP.md`
- `process/docs/design/CR050-GIT-BRANCH-DEPENDENCY-MAP.md`
- `process/docs/design/CR050-GIT-BRANCH-HLD.md`
- `process/docs/design/CR050-GIT-BRANCH-ARCHITECTURE-DECISION.md`
- CP3 discussion log/checkpoint、context、result/summary、人工审查稿和 launch message。

## Return Summary

- 结论：`PASS`；五份设计、CP3 discussion/context/result/checkpoint/launch message 已生成并通过自动校验。
- 推荐架构：native Git lifecycle service + explicit CR Start Coordinator；旧 `cr bootstrap` 不产生隐式 remote writes。
- 关键恢复：preflight-all、确定顺序、append-only attempt、local-only recovery ref、observation-based resume；不做自动补偿或 force。
- HLD预算：17896 bytes `< 20480`。
- 待用户：CP3-DQ-01..04。
- 未执行：源码实现、Story/LLD、commit、真实branch create/push/delete、auto merge、forge API、quant-lab或backup修改。
