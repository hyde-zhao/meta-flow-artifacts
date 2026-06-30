---
handoff_id: "CR139-FOLLOWUP-PHASES-IMPLEMENTATION-HANDOFF"
status: "ready-for-followup-cp5"
created_at: "2026-06-28T23:03:51+08:00"
created_by: "host-orchestrator"
from_agent: "host-orchestrator"
to_agent: "meta-dev"
source_change: "CR-139"
workflow_id: "CR139-STRATEGY-DATA-FOUNDATION"
wave_id: "CR139-followup"
stage: "story-execution"
semantic: "stage-dispatch"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "meta-dev"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev"
  reasoning_profile: "default"
  dispatch_trigger: "phase-default"
  agent_path: ""
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: ""
  fallback_reason: "Current tool surface does not expose spawn_agent/resume_agent/send_input; this file is a context handoff only and does not claim subagent execution."
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "queue-only"
  structured_choice_allowed: false
  allowed_question_scope: "meta-dev may write clarification items for CP5 follow-up batch; host-orchestrator asks the user if required."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 正式人工门禁、运行授权、凭据、安全边界、publish、live / 交易类授权"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CR139-FOLLOWUP-CP5-IMPLEMENTATION-CONTEXT.yaml"
  context_ref: "process/context/CR139-FOLLOWUP-CP5-IMPLEMENTATION-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 8
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/state/STATE.current.json"
    - "process/context/CR139-WAVE1-COMPLETE-FOLLOWUP-CONTEXT.yaml"
    - "process/STORY-STATUS-CR139.md"
    - "process/DEVELOPMENT-PLAN-CR139.yaml"
    - "process/STORY-BACKLOG-CR139.md"
    - "docs/design/FEATURE-DESIGN-MATRIX.md"
  must_read:
    - "process/state/STATE.current.json"
    - "process/context/CR139-FOLLOWUP-CP5-IMPLEMENTATION-CONTEXT.yaml"
    - "process/context/CR139-WAVE1-COMPLETE-FOLLOWUP-CONTEXT.yaml"
    - "process/STORY-STATUS-CR139.md"
  read_if_needed:
    - "process/DEVELOPMENT-PLAN-CR139.yaml"
    - "process/STORY-BACKLOG-CR139.md"
    - "docs/design/FEATURE-DESIGN-MATRIX.md"
    - "market_data/catalog.py"
    - "docs/quality/VERIFICATION-REPORT-CR139-W1.md"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/changes/CR-*.md"
    - "process/stories/*-LLD.md"
    - "docs/design/HLD.md"
    - "docs/quality/VERIFICATION-REPORT-CR139-W1.md"
    - "完整 git diff"
    - "完整会话 transcript"
---

# CR139 Follow-up Phases And Implementation Handoff

## 目标

本交接把 CR139 Wave1 完成后的后续推进上下文收口成一个可恢复入口。下一轮默认先读：

1. `process/context/CR139-FOLLOWUP-CP5-IMPLEMENTATION-CONTEXT.yaml`
2. `process/context/CR139-WAVE1-COMPLETE-FOLLOWUP-CONTEXT.yaml`
3. `process/STORY-STATUS-CR139.md`

不要直接读取完整 HLD、全部 Story LLD、完整 Wave1 验证报告或完整 git diff，除非出现缺字段、冲突、人工审计或深度评审。

## 当前事实

Wave1 已完成，但 CR139 未完成。

| 项目 | 状态 | 证据 |
|---|---|---|
| Active CR | `CR-139` | `process/state/STATE.current.json` |
| 当前阶段 | `story-execution`，Wave1 verified 后 follow-up context convergence | `process/STATE.md` / `STATE.current.json` |
| Wave1 | S01-S08 全部 `verified` | `process/STORY-STATUS-CR139.md` |
| Wave2/W3 | 全部仍 `lld-pending` / `pending-cp5` | `process/STORY-STATUS-CR139.md` |
| CR tracking | active formal CR 只有 `CR-139`，blocked formal CR 为 none | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` |
| subagent tools | 当前工具面无 `spawn_agent` / `resume_agent` / `send_input` | 本交接仅 handoff-only，不声明子 agent 已执行 |

Wave1 验证范围是静态 / fixture：S08 只交付路径治理和 fixture 级兼容。物理分区迁移、真实 lake 写入、provider-lake-catalog 写入、published pointer advance execution、runtime/NAS/QMT/trading/Git remote write 均未授权。

## 后续阶段

| 顺序 | 阶段 | 目标 | 进入条件 | 退出条件 |
|---:|---|---|---|---|
| 1 | Follow-up CP5 批次选择 | 选择 W2/W3 的下一批设计证据范围 | Wave1 verified，follow-up context ready | 用户 approve 推荐批次或明确改批次 |
| 2 | CP5 设计证据收敛 | meta-dev 生成 / 刷新 batch Story LLD 或 technical-note | 批次已选，依赖和文件 owner 可解释 | 全部 batch Story ready-for-review，阻断 clarification 为 0 |
| 3 | CP5 人工门禁 | host-orchestrator 汇总 Decision Brief、待决策项、不授权边界 | CP5 自动预检通过 | CP5 approved 才允许进入实现 |
| 4 | CP6 实现 | 按 DAG 和 merge_order 串行 / 并行实现已批准 Story | 当前 Story `dev_ready`，无文件冲突 | CP6 PASS，return packet 和 evidence index 完整 |
| 5 | CP7 验证 | meta-qa 做静态 / fixture / dry-run / 审计验证 | CP6 PASS | PASS / PASS_WITH_RISK / NEEDS_REWORK / BLOCKED 合法路由 |
| 6 | 下一批循环 | 继续 ML/read/consistency、config/audit、W3 ops | 前序 batch verified 或风险接受 | 所有目标 Story verified 后再考虑 CP8 / release readiness |

## 推荐首批

推荐先启动 `CR139-W2-DATA-CONTRACTS`，因为它关闭 Wave1 之后最容易扩散的底层数据契约，避免 ML/read-gate 后续返工。

| Story | 主题 | 依赖 / 说明 |
|---|---|---|
| S09 | schema evolution contract freeze | 基线门 + S07 |
| S14 | incremental append + pointer advance | S04 + S08；不得授权 pointer advance execution |
| S22 | run-id lineage penetration | S04 + S14；T6 owner=FEAT-02 写侧 |
| S30 | run_id naming convention | S08 |
| S31 | events schema repair | 基线门 |
| S32 | write dedup guarantee | S07 |
| S33 | catalog/manifest source of truth | S04；catalog.py merge_order 第 2 位 |
| S34 | lineage_checksum backfill | S33 |
| S39 | CR -> data audit chain | S08 + S33；catalog.py merge_order 第 3 位 |

S33/S39 因为都触碰 `market_data/catalog.py`，不能并行合并；顺序必须是 S04 已完成 -> S33 -> S39。

## catalog.py 命名清理补充项

本轮检查确认 `market_data/catalog.py` 仍有历史 CR 命名 API：

- `CR014_CATALOG_POINTER_REQUIRED_FIELDS`
- `CR018_*` 常量
- `build_cr018_release_contract_metadata()`
- `build_cr018_release_rollback_contract()`
- `build_cr018_current_pointer_update_plan()`
- `build_cr018_publish_evidence_record()`
- `_cr018_*` helper

现有 CR139 计划已包含 `catalog.py` 后续整改：

- S33：catalog/manifest 定主。
- S39：`triggered_by_cr` / `run_lineage` 审计链。

但原计划没有把“CR014/CR018 命名清理”写成显式验收项。后续 CP5 应把它作为 S33 的设计和验收补充：

| 决策 | 推荐处理 |
|---|---|
| 归属 | 放入 S33，不单独另起 CR，除非评估为 public API 破坏面过大 |
| 方式 | 新增领域命名 API，保留旧 CR 命名 alias |
| 测试 | 保留现有 CR014/CR018 测试，同时新增领域命名测试 |
| S39 边界 | S39 只做 `triggered_by_cr`、`run_lineage`，不承担旧 API 改名主体 |
| 禁止 | 不得直接删除旧 API；当前 tests 仍直接 import 部分 CR018 符号 |

## 文件所有权

| 文件 | 后续 Story | 顺序 |
|---|---|---|
| `market_data/readers.py` | S09, S25, S27, S28, S13, S16, S36 | S05 -> S06 -> S07 已完成；后续按 S09 -> S25 -> S27 -> S28 -> S13 -> S16 -> S36 |
| `market_data/publish.py` | S14, S22 | S04 已完成；后续 S14 -> S22 |
| `market_data/catalog.py` | S33, S39 | S04 已完成；后续 S33 -> S39 |
| `market_data/normalization.py` | S31, S32, S34, S14, S23, S22 | S08 已完成；后续 S31 -> S32 -> S34 -> S14 -> S23 -> S22 |

同一文件同一时刻只能有一个 merge owner。CP5 设计证据必须把 shared / forbidden write boundary 写清楚。

## 不授权边界

后续任何 `approve` 都不自动授权以下动作：

- runtime / NAS / QMT / MiniQMT / gateway runtime
- trading / small_live / live
- credential / secret read
- provider-lake-catalog write
- real lake write
- published pointer advance execution
- physical partition migration execution
- Git remote write

如需真实运行或真实写入，必须另建 `runtime_authorization` 决策项，写明 action scope、运行窗口、脱敏、回滚、审计范围和停止条件。

## CP5 待决策项

| ID | 类型 | 问题 | 推荐方案 |
|---|---|---|---|
| CR139-FU-DQ-01 | implementation | 下一批 CP5 范围 | 先启动 `CR139-W2-DATA-CONTRACTS` |
| CR139-FU-DQ-02 | implementation | `catalog.py` CR014/CR018 命名清理 | 纳入 S33，领域命名新 API + 旧 alias 兼容 |
| CR139-FU-DQ-03 | runtime_authorization | 是否授权物理迁移 / 真实写入 / pointer advance | 本轮不授权，需要时另走 runtime gate |

这些决策项应进入后续 CP5 Decision Brief。用户回复 `approve` 只能接受推荐方案，不代表授权真实运行或写入。

## 下一步准确提示

```text
启动 CR139 后续 CP5 批次准备：默认选择 CR139-W2-DATA-CONTRACTS，生成 S09/S14/S22/S30/S31/S32/S33/S34/S39 的设计证据收敛计划和 CP5 Decision Brief；把 catalog.py CR014/CR018 命名清理作为 S33 验收补充；禁止 runtime、真实 lake 写入、provider catalog 写入、published pointer advance execution 和物理分区迁移执行。
```

## 完成判定

本交接完成后，下一轮可以不读取完整历史对话，直接从 `process/context/CR139-FOLLOWUP-CP5-IMPLEMENTATION-CONTEXT.yaml` 恢复。真正进入实现前必须满足：

1. Follow-up CP5 batch 已选定。
2. batch Story 的设计证据已输出。
3. CP5 自动预检无阻断项。
4. CP5 人工 checkpoint approved。
5. 每个 Story 的 dev_gate、依赖和 file ownership 均可计算。
