---
handoff_id: "HO-CR051-CP8-META-DOC"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "documentation"
from_role: "host-orchestrator"
to_role: "meta-doc"
codex_agent_name: "doc-cao"
reasoning_profile: "default"
status: "completed-with-risk"
context_ref: "process/context/CR051-DOCUMENTATION-CONTEXT.yaml"
created_at: "2026-07-18T16:45:43Z"
dispatch:
  canonical_role: "meta-doc"
  codex_agent_name: "doc-cao"
  reasoning_profile: "default"
  dispatch_trigger: "all-five-stories-cp7-complete"
  tool_name: "spawn_agent"
  agent_id: "/root/cr051_cp8_meta_doc"
  thread_id: "/root/cr051_cp8_meta_doc"
completed_at: "2026-07-18T16:53:00Z"
return_ref: "process/returns/CR051-DOCUMENTATION.return.json"
evidence_ref: "process/evidence/CR051-DOCUMENTATION.index.json"
---

# CR-051 Documentation Handoff

按 capsule-first 读取 `process/context/CR051-DOCUMENTATION-CONTEXT.yaml`，只做 CR-051 的增量用户文档更新。核心任务是清楚区分源码/交付仓库与共享产物仓库的 Git 生命周期，写明项目优先的 `main → projects/<project>/integration → projects/<project>/cr/<cr-id>-<slug>` 拓扑、artifact worktree 驻留/切换边界、双 leg 聚合规则以及人工 main↔project 同步边界。

不得声称已完成真实 remote、branch protection、Windows native、worktree/ref、迁移、软链接或发布验证；不得发明 migration/worktree CLI。仅允许写 context 中列出的两份用户文档与 documentation return/evidence。完成后运行规定的 guardrail/capability claims 检查并返回证据；不得修改实现、测试、State、ledger、checkpoint 或 release 文档。
