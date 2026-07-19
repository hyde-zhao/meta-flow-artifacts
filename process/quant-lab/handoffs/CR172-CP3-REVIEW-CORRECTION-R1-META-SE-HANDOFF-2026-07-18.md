---
handoff_id: "CR172-CP3-REVIEW-CORRECTION-R1-20260718"
from_agent: "host-orchestrator"
to_agent: "meta-se-critical"
change_id: "CR-172"
phase: "solution-design"
checkpoint: "CP3"
status: "completed"
created_at: "2026-07-18T09:58:50+08:00"
context_ref: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
return_summary_path: "process/handoffs/CR172-CP3-REVIEW-CORRECTION-R1-RETURN-SUMMARY.md"
reuse_key: "meta-se-critical+CR-172+PATH-I+solution-design"
dispatch:
  required: true
  semantic: "cp3-review-correction"
  mode: "subagent-recreated"
  platform: "codex"
  agent_role: "meta-se"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "user-review-correction+public-contract+runtime-authorization-dependency"
  agent_path: ".codex/agents/meta-se-critical.toml"
  tool_name: "spawn_agent"
  agent_id: "/root/cr172_cp3_review_correction_r1"
  agent_name: "cr172_cp3_review_correction_r1"
  thread_id: "/root/cr172_cp3_review_correction_r1"
  resumed_at: ""
  spawned_at: "2026-07-18T10:00:37+08:00"
  completed_at: "2026-07-18T10:05:36+08:00"
  evidence: "process/checks/CP3-CR172-PATH-I-HLD-REVIEW-CORRECTION-R1.result.json"
  fallback_reason: "旧线程 /root/cr172_path_i_cp3_meta_se_critical 已被平台回收，followup_task 返回 not found；按同一 canonical role/reasoning profile 重建。"
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "只能把无法由本次三项明确评审意见或既有 CP2 合同收敛的问题交还 host-orchestrator。"
  forbidden_question_scope: "不得请求真实 lake/NAS、runtime、trial-return 生成、R 计算、SignalBatch、交易、发布、部署或 Git remote 授权。"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
  context_ref: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
  read_profile: "compact"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "user-requested-cp3-deep-review-correction"
  must_read:
    - "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
    - "process/state/STATE.current.json"
    - "process/current/CURRENT.json"
    - "docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
    - "docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md"
  allowed_reads: "由 capsule.allowed_reads 限定，并允许读取本 handoff 明列的 CP3 设计与检查文件"
  read_if_needed: "由 capsule.read_if_needed 限定"
  do_not_read_by_default: "由 capsule.do_not_read_by_default 限定"
---

# CR-172 PATH-I CP3 评审整改 R1 交接

## 目标

只修订三项 CP3 设计合同缺口，不扩大 PATH-I 范围，不下沉到实现，并生成可由 Host Orchestrator 回填 CP3 批准的整改证据。

## 必须完成的三项整改

1. **冻结 return-definition ADR 归属**
   - 在 HLD 与 ADR 中明确：CP3 architecture owner 对 `ReturnDefinitionV1` 的业务语义、schema identity、演进规则负责。
   - v1 canonical payload 固定为恰好两个必填列：`timestamp`、`simple_return`；不得在 CP5 静默增加 `net_return`、`gross_return` 或 `nav`。
   - CP5 只允许决定校验/序列化实现细节；增加或重定义 canonical return 字段必须通过 versioned return-definition ADR + schema version bump。若改变 owner、真实运行授权、跨 trial 可比性或 empirical-R 统计语义，则拆独立后续 CR。
   - 将 HLD §1.8 中该问题标成 RESOLVED，不得继续以缺失信息存在。
2. **把 native producer 插桩位置变成 CP5 可验证设计义务**
   - 不在 CP3 猜测具体源码文件；定义 `DO-CR172-CP5-001`（或等价稳定 ID）。
   - CP5 必须给出唯一插桩 integration point、source-touch/file-owner inventory、调用方向/时机、输入输出、授权判定顺序、失败/回退、测试入口和 merge order。
   - 若 CP5 无法证明单一安全插桩点，结论必须为 `NEEDS_DESIGN_CLARIFICATION` 或转 Spike/ARCH-B，不得带着 OPTIONAL 进入实现。
3. **显式冻结六类授权的执行依赖 DAG**
   - 区分“授权记录可独立审批/撤销”与“动作具备执行资格所需前置”。partial authorization 不产生权限并集。
   - 至少冻结：`data_lake_read -> multi_trial_runtime_and_workspace_write -> trial_return_generation -> empirical_R_computation`；以及 `trial_return_generation -> nas_replica_sync -> execution_pull_verify_materialize`。
   - `multi_trial_runtime_and_workspace_write` 即使自身已获批，在相同 scope revision/hash、release/run/family context 下没有有效 `data_lake_read` 时也必须 `eligible_to_execute=false`。
   - 对后续动作说明其在操作时需校验的直接前置、sealed provenance 与本动作独立授权；不得把 DAG 写成六授权合并审批。
   - 增加至少一个负向设计模拟/检查，证明“runtime 获批但 data_lake_read 未获批”仍不启动 runner、不创建 workspace、不推进 pointer。

## 允许写入

- `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`
- `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`
- `process/discussions/CP3-CR172-PATH-I-HLD-DISCUSSION-LOG.md`
- `process/checks/CP3-CR172-PATH-I-DISCUSSION-CHECKPOINT.json`
- `process/checks/CP3-CR172-PATH-I-HLD-REVIEW-CORRECTION-R1.result.json`
- `process/handoffs/CR172-CP3-REVIEW-CORRECTION-R1-RETURN-SUMMARY.md`
- 本 handoff 的状态/dispatch completion 字段

如为内部一致性必须同步 `docs/design/BLUEPRINT.md`、`DOMAIN-MAP.md` 或 `DEPENDENCY-MAP.md`，仅允许做对应三项合同的增量指针/约束更新并追加修订记录。

## 禁止写入与不授权边界

- 不编辑 CR-172、STATE/CURRENT、人工 checkpoint、任何 ledger、Story/LLD/Feature 设计或代码/测试。
- 不创建运行目录，不读写真实 lake/NAS，不启动 multi-trial，不生成真实 trial-return，不计算真实 R，不传 SignalBatch，不交易、不发布、不部署、不执行 Git remote write。
- 当前六类真实动作授权/执行继续为 `0/6`，`stage3_entry_ready=false`、`c1_computable=false`。

## 退出条件

- 三项整改逐项 `3/3 PASS`，有独立 result JSON 和 return summary。
- HLD/ADR 内部一致，无新 OPEN/阻断；如发现新关键人工决策，停止并明确交还，不得隐式代选。
- PATH-I 范围、4 个原 CP3 DQ 与 claim ceiling 不改变。
