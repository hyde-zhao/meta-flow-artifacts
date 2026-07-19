---
handoff_id: "CR172-CP3-META-SE-CRITICAL-20260717"
from_agent: "host-orchestrator"
to_agent: "meta-se-critical"
change_id: "CR-172"
phase: "solution-design"
checkpoint: "CP3"
status: "completed"
created_at: "2026-07-17T16:58:00+08:00"
context_ref: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
return_summary_path: "process/handoffs/CR172-CP3-META-SE-RETURN-SUMMARY.md"
reuse_key: "meta-se-critical+CR-172+PATH-I+solution-design"
dispatch:
  required: true
  semantic: "delegated-user-interaction"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-se"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "architecture-freeze+public-contract+security-boundary+cross-machine-data-flow"
  agent_path: ".codex/agents/meta-se-critical.toml"
  tool_name: "spawn_agent"
  agent_id: "/root/cr172_path_i_cp3_meta_se_critical"
  agent_name: "cr172_path_i_cp3_meta_se_critical"
  thread_id: "/root/cr172_path_i_cp3_meta_se_critical"
  spawned_at: "2026-07-17T17:05:47+08:00"
  completed_at: "2026-07-17T17:30:37+08:00"
  evidence: "process/handoffs/CR172-CP3-META-SE-RETURN-SUMMARY.md"
  fallback_reason: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "只可把无法由已批准 CP2 合同或仓库事实收敛的 CP3 决策项提交给 host-orchestrator。"
  forbidden_question_scope: "不得向用户索取实现、真实数据、NAS、runtime、信号、交易、发布或 Git remote 授权。"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
  context_ref: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
  read_profile: "compact"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "deep_review"
  must_read:
    - "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
    - "process/state/STATE.current.json"
    - "process/current/CURRENT.json"
  allowed_reads: "由 capsule.allowed_reads 限定"
  read_if_needed: "由 capsule.read_if_needed 限定"
  do_not_read_by_default: "由 capsule.do_not_read_by_default 限定"
---

# CR-172 PATH-I CP3 高风险架构设计交接

## 已批准输入

CP2 已 7/7 批准 DQ-CR172-009～015。该批准只允许 PATH-I solution design：trial-return instrumentation 合同、logical URI/manifest/seal、研究机 local canonical → NAS verified replica → 执行机 local immutable cache、六类逐动作授权、empirical-R 分类与最小 SignalBatch 边界。

## 任务

1. 增量更新 `docs/design/BLUEPRINT.md`、`DOMAIN-MAP.md`、`DEPENDENCY-MAP.md`，保留既有基线和修订记录。
2. 新建 CR-specific HLD 与 ADR：
   - `docs/design/HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`
   - `docs/design/ARCHITECTURE-DECISION-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`
3. 先完成 Architecture Gray Areas、table-first advisor discussion、至少两套真实候选和 HLD 拆分判定；记录到 `process/discussions/CP3-CR172-PATH-I-HLD-DISCUSSION-LOG.md`。
4. 对 REQ-009～015 做 7/7 Use Case → Architecture Trace，并模拟至少 3 个关键场景：正常 seal/replicate/materialize、replica stale/hash mismatch、empirical R/FU-CR173-001 或 DQ-003 降级。
5. 生成 CP3 discussion checkpoint、HLD consistency result 和 return summary；人工 CP3 checkpoint、Gate Ledger 与 STATE 由 Host Orchestrator 维护。

## 强制边界

- PATH-I 当前只做设计，不做 Story/LLD/source/test/fixture 实现。
- 不创建运行目录，不修改 runner 默认值，不读写真实 lake/NAS，不运行 multi-trial，不计算真实 trial-return/R，不传 SignalBatch，不交易、不发布、不部署、不执行 Git remote write。
- CR-163 只拥有 lineage metadata/ref；trial-return 是独立 sealed artifact。现有 `layered_returns.csv` 不是 trial-return source。
- NAS 不得成为 runtime working canonical；执行机不得直读 NAS 或研究机目录运行。
- SignalBatch 只冻结精确 8 字段边界，不设计 mailbox 路径、状态机、ack、replay/idempotency、exchange 或 intraday。
- FU-CR173-001 只阻断 positive empirical effective count/c1_computable；DQ-003 typed_unavailable 降级必须保留。
- CP3 最高设计声明；`stage3_entry_ready=false`、`c1_computable=false`，六类真实动作授权/执行均为 0/6。

## 文件所有权

本任务只允许写入上列 3 份增量设计基线、2 份 CR-specific 设计文档、1 份 discussion log、2 份 CP3 自动检查结果和 1 份 return summary。不得编辑 CR、STATE、CURRENT、人工 checkpoint 或任何 ledger。

## 退出条件

- 设计内部一致性、官方/仓库事实、数据 owner 唯一性和依赖无环均通过。
- 7/7 requirement trace，3/3 simulation，真实操作 0，blocker 0。
- 所有开放设计决策均可枚举为 CP3 Decision Brief 项；不得以隐式默认替代人工确认。
