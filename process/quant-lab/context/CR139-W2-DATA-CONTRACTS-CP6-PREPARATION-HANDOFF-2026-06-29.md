---
handoff_id: "CR139-W2-DATA-CONTRACTS-CP6-PREPARATION-HANDOFF-2026-06-29"
created_at: "2026-06-29T09:09:40+08:00"
from: "host-orchestrator"
to: "next-host-orchestrator"
workflow_id: "CR139-STRATEGY-DATA-FOUNDATION"
source_change: "CR-139"
batch_id: "CR139-W2-DATA-CONTRACTS"
phase: "story-execution"
semantic: "context-reset"
status: "ready-for-cp6-preparation"
dispatch:
  required: false
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "next-host-orchestrator"
  canonical_role: "host-orchestrator"
  codex_agent_name: ""
  reasoning_profile: "default"
  dispatch_trigger: "context-reset-before-cp6-preparation"
  agent_path: ""
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: "handoff-only; no subagent dispatch and no code implementation executed"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Only if CP6 preparation finds an unrecoverable conflict in approved CP5 evidence, file owner boundaries, or work-packet authorization scope."
  forbidden_question_scope: "Runtime authorization, credential/secret access, real lake write, provider catalog/provider-lake-catalog write, published pointer advance execution, physical partition migration execution, NAS/QMT/MiniQMT/gateway runtime, trading/small_live/live, Git remote write, CP8 final approval."
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CR139-W2-DATA-CONTRACTS-CP6-PREPARATION-HANDOFF-2026-06-29.md"
  context_ref: "process/context/CR139-W2-DATA-CONTRACTS-CP6-PREPARATION-HANDOFF-2026-06-29.md"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 18
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/state/STATE.current.json"
    - "process/STATE.md"
    - "process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml"
    - "process/checkpoints/CP5-CR139-W2-DATA-CONTRACTS-FORMAL-LLD-BATCH-REVIEW.md"
    - "process/checks/CP5-CR139-W2-DATA-CONTRACTS-FORMAL-LLD-BATCH.result.json"
    - "process/STORY-STATUS-CR139.md"
    - "process/DEVELOPMENT-PLAN-CR139.yaml"
    - "process/STORY-BACKLOG-CR139.md"
    - "docs/design/FEATURE-DESIGN-MATRIX.md"
    - "process/stories/CR139-S09-schema-evolution-contract-freeze-LLD.md"
    - "process/stories/CR139-S14-incremental-append-pointer-advance.md"
    - "process/stories/CR139-S22-runid-lineage-penetration.md"
    - "process/stories/CR139-S30-runid-naming-convention.md"
    - "process/stories/CR139-S31-events-schema-repair.md"
    - "process/stories/CR139-S32-write-dedup-guarantee.md"
    - "process/stories/CR139-S33-catalog-manifest-source-of-truth.md"
    - "process/stories/CR139-S34-lineage-checksum-backfill.md"
    - "process/stories/CR139-S39-cr-data-audit-chain.md"
  must_read:
    - "process/context/CR139-W2-DATA-CONTRACTS-CP6-PREPARATION-HANDOFF-2026-06-29.md"
    - "process/state/STATE.current.json"
    - "process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml"
    - "process/checkpoints/CP5-CR139-W2-DATA-CONTRACTS-FORMAL-LLD-BATCH-REVIEW.md"
    - "process/checks/CP5-CR139-W2-DATA-CONTRACTS-FORMAL-LLD-BATCH.result.json"
    - "process/STORY-STATUS-CR139.md"
  read_if_needed:
    - "process/DEVELOPMENT-PLAN-CR139.yaml"
    - "process/STORY-BACKLOG-CR139.md"
    - "docs/design/FEATURE-DESIGN-MATRIX.md"
    - "process/checks/CP5-CR139-S09-schema-evolution-contract-freeze-LLD-IMPLEMENTABILITY.md"
    - "process/checks/CP5-CR139-S14-incremental-append-pointer-advance-LLD-IMPLEMENTABILITY.md"
    - "process/checks/CP5-CR139-S22-runid-lineage-penetration-LLD-IMPLEMENTABILITY.md"
    - "process/checks/CP5-CR139-S30-runid-naming-convention-LLD-IMPLEMENTABILITY.md"
    - "process/checks/CP5-CR139-S31-events-schema-repair-LLD-IMPLEMENTABILITY.md"
    - "process/checks/CP5-CR139-S32-write-dedup-guarantee-LLD-IMPLEMENTABILITY.md"
    - "process/checks/CP5-CR139-S33-catalog-manifest-source-of-truth-LLD-IMPLEMENTABILITY.md"
    - "process/checks/CP5-CR139-S34-lineage-checksum-backfill-LLD-IMPLEMENTABILITY.md"
    - "process/checks/CP5-CR139-S39-cr-data-audit-chain-LLD-IMPLEMENTABILITY.md"
  do_not_read_by_default:
    - "complete conversation transcript"
    - "complete git diff"
    - "process/changes/CR-*.md"
    - "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
    - "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
    - "process/stories/CR139-S10-*"
    - "process/stories/CR139-S11-*"
    - "process/stories/CR139-S12-*"
    - "process/stories/CR139-S13-*"
    - "process/stories/CR139-S15-*"
    - "process/stories/CR139-S16-*"
    - "process/stories/CR139-S17-*"
    - "process/stories/CR139-S18-*"
    - "process/stories/CR139-S19-*"
    - "process/stories/CR139-S20-*"
    - "process/stories/CR139-S21-*"
    - "process/stories/CR139-S23-*"
    - "process/stories/CR139-S24-*"
    - "process/stories/CR139-S25-*"
    - "process/stories/CR139-S26-*"
    - "process/stories/CR139-S27-*"
    - "process/stories/CR139-S28-*"
    - "process/stories/CR139-S29-*"
    - "process/stories/CR139-S35-*"
    - "process/stories/CR139-S36-*"
    - "process/stories/CR139-S37-*"
    - "process/stories/CR139-S38-*"
    - "process/stories/CR139-S40-*"
    - "data/**"
    - ".env"
    - ".env.*"
    - "credentials"
    - "runtime logs"
---

# CR139 W2 Data Contracts CP6 Preparation Handoff

本文件是清除上下文后的恢复入口。下一轮会话只推进 `CR139-W2-DATA-CONTRACTS` 的 **CP6 preparation**：生成 CP6 implementation context 和 9 个 Story work packets。不要直接开始改代码，不要运行真实数据 / runtime 操作。

## 1. 当前状态

| 项 | 当前事实 | 证据 |
|---|---|---|
| Active CR | `CR-139` / Strategy Data Foundation | `process/state/STATE.current.json` |
| 当前阶段 | `story-execution`，CP6 preparation pending | `process/state/STATE.current.json` |
| 批次 | `CR139-W2-DATA-CONTRACTS` | `process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml` |
| CP5 formal result | PASS | `process/checks/CP5-CR139-W2-DATA-CONTRACTS-FORMAL-LLD-BATCH.result.json` |
| CP5 人工门禁 | approved by user at `2026-06-29T07:41:03+08:00` | `process/checkpoints/CP5-CR139-W2-DATA-CONTRACTS-FORMAL-LLD-BATCH-REVIEW.md` |
| Story 状态 | S09/S14/S22/S30/S31/S32/S33/S34/S39 均为 `lld-approved` | `process/STORY-STATUS-CR139.md` |
| 当前限制 | CP6 preparation only；不得实现代码 | 用户最新指令 |

本 handoff 本身不构成 subagent 执行证据，也不代表 CP6 已开始。下一轮必须先生成 CP6 context / work packets，再决定是否进入实现。

## 2. 下一轮只做的事

按以下顺序执行，直到第 2 步完成为止；不要进入代码修改。

1. 生成 CP6 implementation context。
   - 目标路径：`process/context/CP6-CR139-W2-DATA-CONTRACTS-IMPLEMENTATION-CONTEXT.yaml`
   - 类型：`cp6-implementation`
   - 作用：为 9 个 Story 的 CP6 work packets 提供批次级事实、顺序、授权边界和读取策略。

2. 生成 9 个 Story work packets。
   - `process/context/stories/STORY-CR139-S09.CP6.work-packet.json`
   - `process/context/stories/STORY-CR139-S30.CP6.work-packet.json`
   - `process/context/stories/STORY-CR139-S31.CP6.work-packet.json`
   - `process/context/stories/STORY-CR139-S32.CP6.work-packet.json`
   - `process/context/stories/STORY-CR139-S33.CP6.work-packet.json`
   - `process/context/stories/STORY-CR139-S34.CP6.work-packet.json`
   - `process/context/stories/STORY-CR139-S14.CP6.work-packet.json`
   - `process/context/stories/STORY-CR139-S22.CP6.work-packet.json`
   - `process/context/stories/STORY-CR139-S39.CP6.work-packet.json`
   - 可选索引：`process/context/stories/CR139-W2-DATA-CONTRACTS.CP6.story-packets.json`

3. 每个 work packet 必须写清：
   - `story_id`、`slug`、`objective`
   - `design_evidence_ref`
   - `cp5_result_ref` 与 `cp5_checkpoint_ref`
   - `dependencies` 与 `dependency_type`
   - `merge_order`
   - `allowed_read_paths`
   - `allowed_write_paths`
   - `forbidden_write_paths`
   - `forbidden_actions`
   - `static_fixture_validation_scope`
   - `verification_commands_planned`
   - `expected_return_packet_ref`
   - `expected_evidence_index_ref`
   - `authz_policy_refs`

4. 生成 packet 后停下，等待用户下一步指令或明确授权再进入代码实现。

## 3. 推进顺序

用户已指定并确认以下顺序：

```text
S09 -> S30/S31/S32 -> S33 -> S34 -> S14 -> S22 -> S39
```

执行解释：

| 顺序 | Story | 理由 |
|---:|---|---|
| 1 | S09 | 冻结 schema evolution、reader 兼容回退、文件切片和 downstream contract。 |
| 2 | S30 | run_id 命名约定先定，避免 normalization/lake_layout 后续返工。 |
| 3 | S31 | events schema repair 消费基线门事实，并影响 normalization。 |
| 4 | S32 | write dedup guarantee 依赖 S07，和 S31 同属 normalization 关键路径。 |
| 5 | S33 | catalog/manifest source-of-truth，包含 catalog.py CR014/CR018 命名清理验收补充。 |
| 6 | S34 | lineage_checksum backfill 消费 S33 的 source-of-truth 决策。 |
| 7 | S14 | incremental append / pointer plan 消费 S30/S31/S32 和 S08，不执行 pointer advance。 |
| 8 | S22 | run_id lineage penetration 消费 S14，FEAT-02 写侧为 owner，下游只读消费。 |
| 9 | S39 | audit chain 消费 S33/S08，只负责 `triggered_by_cr` / `run_lineage`，不接管 CR014/CR018 命名清理。 |

S30/S31/S32 可以在 context/work-packet 生成时并列准备；进入代码实现时仍需按文件 owner 和 `normalization.py` 冲突串行化。

## 4. Story work packet 生成提示

下表是下一轮生成 work packets 时应落入每个 packet 的初始边界。实际 packet 可以更细，但不得更宽。

| Story | 设计证据 | 初始 allowed_write_paths | 初始 forbidden_write_paths / actions |
|---|---|---|---|
| S09 | `process/stories/CR139-S09-schema-evolution-contract-freeze-LLD.md` | `engine/contracts.py`; `market_data/readers.py`; focused tests for schema/reader compatibility; CP6 return/evidence/check files | real lake data; runtime; catalog/provider writes; physical migration; pointer advance; unrelated reader rewrites |
| S30 | `process/stories/CR139-S30-runid-naming-convention.md#技术说明` | `market_data/normalization.py`; `market_data/lake_layout.py`; focused run_id naming tests; CP6 return/evidence/check files | physical path migration; real lake rewrite; provider catalog writes; unrelated normalization behavior |
| S31 | `process/stories/CR139-S31-events-schema-repair.md#技术说明` | `market_data/normalization.py`; focused events schema tests; CP6 return/evidence/check files | real lake rewrite; runtime ingestion; catalog writes; unrelated schema families |
| S32 | `process/stories/CR139-S32-write-dedup-guarantee.md#技术说明` | `market_data/normalization.py`; `market_data/validation.py`; focused write-dedup tests; CP6 return/evidence/check files | physical migration; real lake rewrite; runtime ingestion; broad reader dedup changes outside S07 contract |
| S33 | `process/stories/CR139-S33-catalog-manifest-source-of-truth.md#技术说明` | `market_data/catalog.py`; `market_data/manifest.py`; focused catalog/manifest tests; CP6 return/evidence/check files | provider catalog / provider-lake-catalog writes; real catalog writes; runtime publish; removing CR014/CR018 compatibility aliases before tests/callers migrate |
| S34 | `process/stories/CR139-S34-lineage-checksum-backfill.md#技术说明` | `market_data/manifest.py`; `market_data/normalization.py`; focused lineage_checksum fixture tests; CP6 return/evidence/check files | real backfill; real lake write; provider catalog writes; physical migration |
| S14 | `process/stories/CR139-S14-incremental-append-pointer-advance.md#技术说明` | `market_data/incremental.py`; `market_data/publish.py`; `market_data/normalization.py`; `market_data/cli.py`; focused append/pointer-plan fixture tests; CP6 return/evidence/check files | published pointer advance execution; real lake write; provider catalog writes; runtime/NAS/QMT; physical partition migration |
| S22 | `process/stories/CR139-S22-runid-lineage-penetration.md#技术说明` | `market_data/publish.py`; `market_data/manifest.py`; `trading/strategy_runner/evidence_index.py`; focused lineage fixture tests; CP6 return/evidence/check files | runtime trading; gateway/QMT reads; provider catalog writes; reverse dependency from trading to market_data write path |
| S39 | `process/stories/CR139-S39-cr-data-audit-chain.md#技术说明` | `market_data/catalog.py`; focused audit-chain tests; CP6 return/evidence/check files | CR014/CR018 naming cleanup ownership; provider catalog writes; real catalog writes; real lake write; physical migration |

For all packets, process evidence write paths are limited to:

- `process/returns/STORY-CR139-Sxx.CP6.return.json`
- `process/evidence/STORY-CR139-Sxx.CP6.index.json`
- `process/checks/CP6-CR139-Sxx-*-CODING-DONE.md`
- `process/stories/CR139-Sxx-*-IMPLEMENTATION.md` only where the packet requires a full implementation summary

Packet generation itself may write only:

- `process/context/CP6-CR139-W2-DATA-CONTRACTS-IMPLEMENTATION-CONTEXT.yaml`
- `process/context/stories/STORY-CR139-Sxx.CP6.work-packet.json`
- optional `process/context/stories/CR139-W2-DATA-CONTRACTS.CP6.story-packets.json`
- optional status/source-ref updates in `process/state/STATE.current.json` and `process/STATE.md`

## 5. 不授权项

继续禁止，除非另走独立 `runtime_authorization`：

- runtime
- 真实 lake 写入
- provider catalog 写入
- provider-lake-catalog 写入
- published pointer advance execution
- physical partition migration execution
- credential / secret read
- NAS / QMT / MiniQMT / gateway runtime
- trading / small_live / live
- Git remote write

这些禁止项适用于 CP6 preparation、后续 CP6 代码实现和 static/fixture 验证阶段。任何 packet 不得把这些内容写成 allowed。

## 6. 下轮启动步骤

1. 读取本 handoff。
2. 运行或确认 process route health check：

```bash
uv run meta-flow workspace check --project-root .
```

3. 读取 `process/state/STATE.current.json`，确认：
   - `active_change=CR-139`
   - `current_phase=story-execution`
   - `pending_gate=null`
   - `parallel_execution.lld_design_batch.status=cp5-approved-cp6-preparation-ready`

4. 读取 CP5 capsule 和 approved checkpoint：
   - `process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml`
   - `process/checkpoints/CP5-CR139-W2-DATA-CONTRACTS-FORMAL-LLD-BATCH-REVIEW.md`
   - `process/checks/CP5-CR139-W2-DATA-CONTRACTS-FORMAL-LLD-BATCH.result.json`

5. 生成 CP6 implementation context。
6. 生成 9 个 Story work packets。
7. 校验 JSON/YAML/NDJSON。
8. 停下并汇报生成路径，不开始代码实现。

## 7. 需要避免的误读

- CP5 approved 只表示设计证据通过并允许准备 CP6，不表示 CP6 已开始或任何代码已实现。
- 本 handoff 不是 CP6 context，也不是 work packet；它只是清空上下文后的恢复入口。
- 生成 work packet 不等于授权执行 work packet。
- S33 负责 CR014/CR018 命名清理验收补充；S39 只负责 audit chain，不接管命名清理。
- S14 可以设计 / 实现 pointer plan，但不能执行 published pointer advance。
- S34 可以设计 / 实现 fixture/static lineage_checksum 逻辑，但不能做真实 backfill。

## 8. 下轮可复制启动语

```text
读取 process/context/CR139-W2-DATA-CONTRACTS-CP6-PREPARATION-HANDOFF-2026-06-29.md，进入 CR139-W2-DATA-CONTRACTS 的 CP6 preparation。先生成 process/context/CP6-CR139-W2-DATA-CONTRACTS-IMPLEMENTATION-CONTEXT.yaml 和 9 个 STORY-CR139-Sxx.CP6.work-packet.json；不要改业务代码，不要执行 runtime、真实 lake 写入、provider catalog/provider-lake-catalog 写入、published pointer advance execution、物理分区迁移、凭据读取、NAS/QMT/MiniQMT/gateway runtime、trading/small_live/live 或 Git remote write。
```

## 9. 本 handoff 生成时已确认

| 检查 | 结果 |
|---|---|
| process route health | `uv run meta-flow workspace check --project-root .` 返回 `process_link_health: ok` |
| CP5 formal result-check | 上一轮已通过 `CP Result Check: OK` |
| CP5 formal human gate checker | 上一轮已通过 `PASS: human gate decision brief valid; decision_count=3` |
| Gate ledger | 上一轮已通过 `Event Ledger Check: OK` |
| 代码改动 | 本 handoff 生成不修改业务代码 |
