---
handoff_id: "CR172-CP4-STORY-PLANNING-20260718"
from_agent: "host-orchestrator"
to_agent: "meta-se-critical"
change_id: "CR-172"
phase: "story-planning"
checkpoint: "CP4"
status: "completed-cp4-pass"
created_at: "2026-07-18T10:15:54+08:00"
context_ref: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
return_summary_path: "process/handoffs/CR172-CP4-STORY-PLANNING-META-SE-RETURN-SUMMARY.md"
reuse_key: "meta-se-critical+CR-172+story-planning"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent-reuse"
  platform: "codex"
  agent_role: "meta-se"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "cp3-approved+public-contract+runtime-high-risk-story-decomposition"
  agent_path: ".codex/agents/meta-se-critical.toml"
  tool_name: "followup_task"
  agent_id: "/root/cr172_cp3_review_correction_r1"
  agent_name: "cr172_cp3_review_correction_r1"
  thread_id: "/root/cr172_cp3_review_correction_r1"
  resumed_at: "2026-07-18T10:15:54+08:00"
  completed_at: "2026-07-18T10:37:15+08:00"
  evidence: "process/checks/CP4-CR172-PATH-I-STORY-DAG-PARALLEL-SAFETY.result.json"
  fallback_reason: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "只能提交会改变已批准 HLD/ADR、Story 总边界、公共 contract、文件 owner 或 CP5 阻断性的设计问题。"
  forbidden_question_scope: "不得请求实现、真实 lake/NAS/runtime、trial-return/R、SignalBatch exchange、交易、发布、部署或 Git remote 授权。"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
  context_ref: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
  read_profile: "compact"
  max_source_files: 18
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "CP3-to-CP4 stage transition; approved HLD/ADR and repository source inventory are required to close DO-CR172-CP5-001"
  must_read:
    - "process/state/STATE.current.json"
    - "process/current/CURRENT.json"
    - "process/checkpoints/CP3-CR172-PATH-I-HLD-REVIEW.md"
    - "docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
    - "docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
    - "process/checks/CP3-CR172-PATH-I-HLD-REVIEW-CORRECTION-R1.result.json"
  allowed_reads:
    - "docs/design/BLUEPRINT.md"
    - "docs/design/DOMAIN-MAP.md"
    - "docs/design/DEPENDENCY-MAP.md"
    - "docs/design/FEATURE-DESIGN-MATRIX.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "engine/mature_multifactor_research.py"
    - "engine/experiment_family_lineage.py"
    - "engine/effective_trial_evidence.py"
    - "tests/research/**"
  read_if_needed:
    - "process/changes/CR-172.md（仅 summary/checkpoint 冲突或审计完整范围时）"
    - "docs/components/MULTIFACTOR-RESEARCH.md（核对现有 producer/run path 契约时）"
  do_not_read_by_default:
    - "process/archive/**"
    - "其他 CR 的完整 Story/LLD/quality/release 文档"
    - "完整会话 transcript"
---

# CR-172 PATH-I CP4 Story Planning 与 Feature Implementation Design 交接

## 已批准输入

CP3 已在评审整改 `3/3 PASS` 后获用户批准。原 `CP3-DQ-CR172-I-01～04`、ARCH-A 目标态、严格 seal/replica/materialization、六动作独立授权与执行资格 DAG、empirical/signal ceiling 均已冻结。批准只允许 Feature/Story/LLD 设计准备与 CP4 自动预检；不授权实现或真实操作。

## 任务

1. 完整读取 `story-planning`、`implementation-design`、`story-manager`、`wave-planner`、`dag-validator`、`coverage-checker` 和 `checkpoint-manager` 的 SKILL.md，再开始写入。
2. 增量更新 `docs/design/FEATURE-DESIGN-MATRIX.md`，为 PATH-I 全部 Feature 作 required/waived/n/a 判定；高风险/数据/权限/跨机/迁移相关 Story 一律 `full-lld`，不得 batch-lld 或 technical-note 降级。
3. 为 required Feature 生成独立 `docs/features/<cr172-feature>/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md`，必须消费 HLD v1.1/ADR v1.1，并把 `DO-CR172-CP5-001` 闭环为可检查合同。
4. 对 repository 做只读 source-touch inventory，确定唯一 native producer integration point 或明确路由到 `NEEDS_DESIGN_CLARIFICATION`/Spike/ARCH-B。必须列出 source/file owner、调用方向和时机、输入输出、授权顺序、失败/回退、测试入口、merge order；不能用“CP6 再决定”。
5. 增量更新 `process/DEVELOPMENT-PLAN.yaml` 并生成 `process/stories/STORY-CR172-*.md`；每个 Story 有稳定 ID/slug、三件套、量化 AC、feature_design_refs、full-lld policy、depends_on/type、文件所有权、merge owner、forbidden path、dev/verification gate。
6. Story 总数以最小可独立验证切片为准。Signal detailed exchange/intraday、FU-CR173-001 方法 v2、external import activation、真实 migration/真实 runtime 均不得创建 Story。
7. 执行覆盖、DAG、文件冲突和并行安全检查，生成：
   - `process/checks/CP4-CR172-PATH-I-STORY-DAG-PARALLEL-SAFETY.result.json`
   - 对应 summary（或 Markdown 摘要）
   - `process/handoffs/CR172-CP4-STORY-PLANNING-META-SE-RETURN-SUMMARY.md`
8. 只完成 CP4；不生成 Story LLD，不发起 CP5 人工门禁。Host Orchestrator 将在 CP4 PASS 后按全量 Story 设计批次调度 meta-dev。

## CP4 必须证明

- CR172 适用 P0 scenarios/requirements/AC 的 Story 计划覆盖率 `100%`；未覆盖=`0`。
- Story DAG cycle=`0`、invalid refs=`0`、未处理 file conflict=`0`。
- `DO-CR172-CP5-001` 的唯一插桩点与 `7/7` integration contract、`5/5` failure path、`3/3` test/merge/rollback 证据可进入 LLD。
- 六动作 approval independence 与 execution eligibility DAG 不被合并；runtime-without-read negative path 保留。
- Story/Feature/LLD 设计不产生任何真实授权：六动作 authorized/executed=`0/6`、`0/6`。

## 允许写入

- `docs/design/FEATURE-DESIGN-MATRIX.md`（仅 CR172 增量）
- 新建的 `docs/features/<cr172-feature>/{DESIGN,TEST-PLAN,TASKS}.md`
- `process/DEVELOPMENT-PLAN.yaml`（仅 CR172 增量）
- `process/stories/STORY-CR172-*.md`
- CP4 result/summary、return summary，以及本 handoff completion 字段

不得编辑 CR、STATE/CURRENT、CP3/CP5 人工 checkpoint、任何 ledger、HLD/ADR/产品基线、代码或测试。

## 不授权边界

代码/测试/fixture 实现、目录创建、runner/default 变更、真实 lake/NAS、runtime、trial-return/R、signal exchange、migration、trading、publish/deploy、Git remote write全部禁止。CP4 只做设计与只读 source inventory。

## 退出条件

- CP4 result=`PASS`，blocker/waiver=`0/0`；否则明确 `BLOCKED` / `NEEDS_DESIGN_CLARIFICATION` 并停止。
- 全部目标 Story 已达到 `lld-ready`，但 LLD 文件数量仍为 `0`。
- 无新增未分类人工决策；非阻断 OPEN/Spike 有 owner、触发条件和 CP5 暴露方式。
