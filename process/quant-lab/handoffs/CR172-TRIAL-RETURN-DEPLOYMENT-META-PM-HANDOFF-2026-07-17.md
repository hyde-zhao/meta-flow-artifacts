---
handoff_id: "CR172-TRIAL-RETURN-DEPLOYMENT-META-PM"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
change_id: "CR-172"
phase: "requirement-clarification"
checkpoint: "CP2"
status: "completed"
created_at: "2026-07-17T14:17:39+08:00"
context_ref: "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
return_summary_path: "process/handoffs/CR172-TRIAL-RETURN-DEPLOYMENT-META-PM-RETURN-SUMMARY.md"
reuse_key: "meta-pm+CR-172+trial-return-deployment+no-story+no-wave"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-pm"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "phase-default"
  agent_path: ".codex/agents/meta-pm.toml"
  tool_name: "spawn_agent"
  agent_id: "/root/cr172_trial_return_cp2_pm"
  agent_name: "cr172_trial_return_cp2_pm"
  thread_id: "/root/cr172_trial_return_cp2_pm"
  spawned_at: "2026-07-17T14:21:31+08:00"
  resumed_at: ""
  completed_at: "2026-07-17T14:40:12+08:00"
  evidence: "process/handoffs/CR172-TRIAL-RETURN-DEPLOYMENT-META-PM-RETURN-SUMMARY.md"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "只允许把不可从用户当前明确部署意图和仓库事实推导的产品灰区提交给 host-orchestrator。"
  forbidden_question_scope: "CP2 正式人工门禁、真实 runtime/NAS/GitHub remote-write 授权、凭据、安全例外、trading/publish 授权。"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
  context_ref: "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/state/STATE.current.json"
    - "process/current/CURRENT.json"
    - "process/changes/summaries/CR-172-TRIAL-RETURN-DEPLOYMENT-DELTA.summary.json"
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "docs/product/RELEASE-SLICES.md"
    - "docs/product/BACKLOG.md"
  must_read:
    - "process/context/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-CONTEXT.yaml"
    - "process/state/STATE.current.json"
    - "process/changes/summaries/CR-172-TRIAL-RETURN-DEPLOYMENT-DELTA.summary.json"
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/MVP-SCOPE.md"
  read_if_needed:
    - "process/changes/CR-172.md"
    - "docs/components/MULTIFACTOR-RESEARCH.md"
    - "engine/mature_multifactor_research.py"
    - "engine/effective_trial_evidence.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/archive/**"
    - "process/discussions/**"
    - "process/stories/**"
    - "docs/design/**"
    - "完整会话 transcript"
---

# CR-172 trial-return / deployment requirement clarification handoff

## 目标

以现有 CR-172 为同一 parent CR，增量更新产品基线，把以下新事实转为可人工确认、可验收、可追溯的 CP2 scope delta：

1. 当前没有独立 trial-return series 存储对象，CR-163 只存 metadata/ref，CR-173 只消费已计算 R。
2. 在 PATH-C/A 前增加 PATH-I instrumentation-first 前置，fixture implementation 与真实 runtime 授权分离。
3. 新 multi-trial artifact 使用 family/run/trial 逻辑 URI 与 NAS canonical path；物理挂载不进入 lineage identity。
4. 当前单次 runner 的新运行目录从 `stage3_mature_multifactor/{run_id}` 改为 `multifactor-strategy-research/{run_id}`；历史目录原地只读保留。
5. 研究机、交易机、NAS、GitHub 四组件的职责、数据主权、允许/禁止数据流和 GitHub data ceiling 必须进入产品范围。
6. README 现有 local-canonical + NAS-rsync 与用户目标 NAS-canonical 冲突，必须作为 DQ 暴露并推荐 local staging + sealed publish；不得静默保留两个默认。

## 输出要求

- 保留 UC-58、REQ-001～008、原 8 个场景和 PATH-B 历史，增量追加而非整体重写。
- 新增 DQ/REQ-009～014；每项必须有推荐、备选、影响、风险、切换/回退条件和可量化成功标准。
- 覆盖正向、负向、边界、权限、失败恢复和 precheck；重点含错误 returns 对象、hash mismatch、unsealed publish、跨 trial 对齐失败、交易机越权、GitHub 大数据泄漏、legacy/new dual truth。
- 更新 STORY-MAP/MVP-SCOPE/RELEASE-SLICES/BACKLOG，使 PATH-I 完成不自动恢复 PATH-C/A。
- 生成 CP1/CP2 自动检查机器结果、人工 CP2 Decision Brief 草案和 return summary；正式 CP2 由 host-orchestrator 发起。
- 不修改蓝图、代码、README/USER-MANUAL，不执行真实数据、NAS、runtime、迁移或 Git remote 操作。

## 退出条件

- 产品文档修订记录完整，旧基线无删除。
- 新增 6 项 DQ 与需求/场景/矩阵 100% 可追溯。
- PATH-I、NAS canonicality、stable URI 和 legacy path 兼容策略可由用户在单一 CP2 门禁中确认。
- `stage3_started=false`、`c1_computable=false`、真实操作计数全部为 0。
