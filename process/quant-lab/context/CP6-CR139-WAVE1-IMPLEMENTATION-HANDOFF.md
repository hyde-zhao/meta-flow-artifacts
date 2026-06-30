---
title: CR-139 Wave1 实现 (CP6/CP7) 启动 Handoff
status: active
cr_id: CR-139
phase: story-execution
wave: CR139-W1
created_at: 2026-06-28T19:25:00+08:00
created_by: host-orchestrator
purpose: 清除上下文后的恢复入口；新会话从本文件 + STATE.current.json 进入 Wave1 实现
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "inline-fallback"
  platform: "codex"
  agent_role: "meta-dev"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev"
  reasoning_profile: "default"
  dispatch_trigger: "phase-default"
  agent_path: ""
  tool_name: "inline-host-orchestrator"
  agent_id: ""
  agent_name: "host-orchestrator-inline"
  thread_id: ""
  spawned_at: ""
  resumed_at: "2026-06-28T20:00:18+08:00"
  completed_at: ""
  evidence: "user-approved-inline-fallback"
  fallback_reason: "Current Codex tool surface does not expose spawn_agent/resume_agent/send_input; user explicitly instructed host-orchestrator to converge Wave1 context and advance CR139 Wave1."
  approved_by: "user"
  approved_at: "2026-06-28T20:00:18+08:00"
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Only implementation clarification items that block CR139 Wave1 static/fixture delivery may be relayed by host-orchestrator."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 正式人工门禁、运行授权、凭据、安全边界、publish、live / 交易类授权"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP6-CR139-WAVE1-IMPLEMENTATION-CONTEXT.yaml"
  context_ref: "process/context/CP6-CR139-WAVE1-IMPLEMENTATION-CONTEXT.yaml"
  story_packet_ref: "process/context/stories/CR139-W1.CP6.story-packets.json"
  read_profile: "compact"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "Initial convergence audit read CP5 result, CR, S01 LLD, development plan, story status and ledgers because the handoff lacked dispatch/context_policy metadata and DEVELOPMENT-PLAN/STORY-STATUS conflicted with CP5 approval."
  allowed_reads:
    - "process/context/CP6-CR139-WAVE1-IMPLEMENTATION-CONTEXT.yaml"
    - "process/context/stories/CR139-W1.CP6.story-packets.json"
    - "process/state/STATE.current.json"
    - "process/checks/CP5-CR139-WAVE1-LLD-BATCH.result.json"
    - "process/checkpoints/CP5-CR139-WAVE1-LLD-BATCH-REVIEW.md"
    - "process/stories/CR139-S01-remediation-object-inventory-LLD.md"
    - "process/stories/CR139-S02-remediation-baseline-golden-values-LLD.md"
    - "process/stories/CR139-S03-duplicate-fingerprint-profiling-LLD.md"
    - "process/stories/CR139-S04-published-pointer-read-selector.md"
    - "process/stories/CR139-S05-pit-as-of-reader.md"
    - "process/stories/CR139-S06-unified-panel-reader-LLD.md"
    - "process/stories/CR139-S07-read-layer-dedup-LLD.md"
    - "process/stories/CR139-S08-runid-partition-key-governance-LLD.md"
  must_read:
    - "process/context/CP6-CR139-WAVE1-IMPLEMENTATION-CONTEXT.yaml"
    - "process/context/stories/CR139-W1.CP6.story-packets.json"
    - "process/state/STATE.current.json"
  read_if_needed:
    - "process/changes/CR-139-STRATEGY-DATA-FOUNDATION-2026-06-28.md"
    - "process/DEVELOPMENT-PLAN-CR139.yaml"
    - "process/STORY-STATUS-CR139.md"
    - "process/STORY-BACKLOG-CR139.md"
    - "process/context/DATA-LAKE-REMEDIATION-HANDOFF.md"
    - "process/REQUIREMENTS.md"
    - "process/TEST-MATRIX.md"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md"
    - "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md"
    - "process/stories/CR139-S09-*"
    - "process/stories/CR139-S1*"
    - "process/stories/CR139-S2*"
    - "process/stories/CR139-S3*"
    - "process/stories/CR139-S40-*"
    - "full conversation transcript"
---

# CR-139 Wave1 实现 (CP6/CP7) 启动 Handoff

> 本文件是上下文清除后的恢复入口。CR-139「Strategy Data Foundation」已通过 CP2/CP3/CP4/CP5-W1，进入 Wave1 实现（CP6 编码 / CP7 验证）。新会话读本文件 + `process/state/STATE.current.json` 即可恢复全部上下文。

## 1. 当前状态快照

| 项 | 值 |
|---|---|
| Active CR | CR-139（Strategy Data Foundation / 数据湖整改 parent CR） |
| Phase | story-execution（Wave1 CP6） |
| Pending gate | CP6（S01 编码完成门） |
| Active Story | **CR139-S01**（T8 整改对象自动化清册，基线门第 1 步，无前置依赖，可立即实现） |
| CP2 | approved 2026-06-28T16:05（D8a 6 项纳入 d1 完整 LLD；D8b 14 项纳入 d2 既有合同闭环不重复设计；Wave1 基线门） |
| CP3 | approved 2026-06-28T17:30（AGA-1=A1 / AGA-3=C1 / AGA-5=E1；设计产物 confirmed-cp3） |
| CP4 | PASS_WITH_RISK 2026-06-28T17:50（40 Story DAG 无环，文件所有权 merge_order 落定，4 LCQ resolved） |
| CP5-W1 | approved 2026-06-28T19:20（首轮 changes_requested 6 项返修闭环 + 二轮 approve；Wave1 8 Story 设计证据完成） |

机器状态真相源：`process/state/STATE.current.json`。人类摘要：`process/STATE.md`。

## 2. Wave1 实现顺序（基线门 REQ-247 硬依赖）

```
S01 T8 清册 (verified) → S02 T7 黄金值 (verified) → S03 C2a 画像 (verified)
                                                          ↓ 基线冻结门（freeze gate）
                          S04 V1 pointer / S05 C1 PIT reader / S06 R1 panel reader / S07 C2b 去重
                                                          ↓
                                            S08 N1 分区治理（deferred-until-baseline-verified，物理迁移后置）
```

**硬约束**：S04-S08 结构性变更必须在 S01/S02/S03 verified 后才实现。S08 物理迁移执行后置到基线冻结之后且需单独 runtime_authorization。

## 3. Wave1 Story 实现清单（设计证据路径 + 关键约束）

| Story | 标题 | lld_policy | 设计证据 | 基线门角色 | 实现关键约束 |
|---|---|---|---|---|---|
| **S01** | T8 整改对象自动化清册 | full-lld (d1) | `process/stories/CR139-S01-remediation-object-inventory-LLD.md` | 冻结第 1 步 | 新建 `scripts/lake_inventory.py` + `market_data/remediation_inventory.py`；只读 catalog/manifest/readers；产出 InventoryReport（17 dataset 行数/覆盖/重复键/pit/published/lineage）；4 Slice T8-1..T8-4 |
| S02 | T7 整改回归基线+黄金值 | full-lld (d1) | `process/stories/CR139-S02-remediation-baseline-golden-values-LLD.md` | 冻结第 2 步 | 新建 `market_data/remediation_baseline.py` + `scripts/lake_golden_baseline.py`；消费 S01 `build_inventory()`；**最小 MetricSpec 7 类已冻结（§3.3）**，实现必须全覆盖不得选择性取值；freeze/compare + 归因报告 |
| S03 | C2a 重复画像 | full-lld (d1) | `process/stories/CR139-S03-duplicate-fingerprint-profiling-LLD.md` | 冻结第 3 步 | **独立只读分析函数，不改读路径**（LCQ-01）；调 `_canonical_paths`(:375) 只读枚举 |
| S04 | V1 published pointer/read selector | technical-note (a) | `process/stories/CR139-S04-published-pointer-read-selector.md` §技术说明 | 结构性变更 | a 类消费既有 HLD-DATA-LAKE §5/§17.4；既有 `publish.py:605 publish_current_pointer`（dry-run only）；**pointer 不自动前移** |
| S05 | C1 PIT as-of reader | technical-note (a) | `process/stories/CR139-S05-pit-as-of-reader.md` §技术说明 | 结构性变更 | a 类消费 HLD-DATA-LAKE §14/§17.7.1；新增 `read_panel_as_of`；**CP6 硬门禁：S05 严禁写 catalog.py，写须拆 S33/独立切片**（catalog 写=0 校验） |
| S06 | R1 统一 panel reader | full-lld (c) | `process/stories/CR139-S06-unified-panel-reader-LLD.md` | 结构性变更 | 新增 `read_panel(datasets, as_of)` 复用 `read_dataset` published 门禁(:2728/2778)；**异粒度 as-of/forward-fill 物化算法 7 条已冻结（§3.3）**；**CP6 硬门禁：S06 严禁写 catalog.py**（与 S05 一致） |
| S07 | C2b 读层去重 | full-lld (d1) | `process/stories/CR139-S07-read-layer-dedup-LLD.md` | 结构性变更 | **嵌入 read_dataset 出口去重**（LCQ-01），按 source_run_id 取最新；readers.py merge_order S05→S06→S07 串行 |
| S08 | N1 run_id 分区治理 | full-lld (c) | `process/stories/CR139-S08-runid-partition-key-governance-LLD.md` | deferred-execution | **dev_gate=deferred-until-baseline-verified**：LLD 已写，实现延迟到 S01/S02/S03 verified 后；**CP6/CP7 不授权物理分区迁移执行**；run-id 在 publish.py 生成（LCQ-03），manifest.py 记录 |

## 4. 文件所有权 merge_order（CP6 实现串行依据）

readers.py 是冲突最密集文件（10 Story 触及），按 merge_order 串行合并：
- **readers.py**: S05→S06→S07→S09→S25→S27→S28→S13→S16→S36（Wave1 部分：S05→S06→S07）
- **publish.py**: S04→S14→S22（Wave1: S04）
- **catalog.py**: S04→S33→S39（Wave1: S04；**S05/S06 只读不写**，CP6 硬门禁）
- **normalization.py**: S08→S31→S32→S34→S14→S23→S22（Wave1: S08）
- **lake_layout.py / cli.py / manifest.py**: 见 `process/DEVELOPMENT-PLAN-CR139.yaml`

完整 merge_order + 文件所有权矩阵：`process/DEVELOPMENT-PLAN-CR139.yaml`。

## 5. LCQ resolved 答案（实现必须遵循）

- **LCQ-01**: C2a（S03）独立只读分析函数不改读路径；C2b（S07）嵌入 read_dataset 出口按 source_run_id 取最新
- **LCQ-02**: readers.py 函数切片 + merge_order 串行（S05→S06→S07...）
- **LCQ-03**: run-id 在 publish.py 生成，manifest.py 记录
- **LCQ-04**: N1（S08）LLD 在 CP5 写，dev_gate=deferred-until-baseline-verified

## 6. CP6/CP7 验证策略

- **validation_mode = static-only**（CP7 默认）：fixture + 静态验证 + `uv run --python 3.11 pytest -q tests/test_cr139_*.py`
- runtime 验证需独立 `runtime_authorization`（当前未授权）
- 每个 Story CP6 完成后写 `process/checks/CP6-CR139-Snn-*.md` + return packet `process/returns/STORY-*.return.json`
- CP7 用 `verification-execution` 消费 SCENARIOS/TEST-MATRIX + 设计证据，输出 `docs/quality/VERIFICATION-REPORT.md`，结论 PASS/PASS_WITH_RISK/BLOCKED/NEEDS_REWORK

## 7. 不授权范围（CP5-W1 通过后仍持续）

CP5-W1 授权 Wave1 Story **代码实现 + 静态/fixture 验证**。**不授权**：
- runtime / NAS / provider / catalog 运行时连接与凭据读取
- QMT / MiniQMT / gateway 运行时、下单撤单/模拟盘/实盘
- 真实数据湖写入 / published pointer 前移执行
- **物理分区迁移执行**（S08 N1 deferred-until-baseline-verified，后置到基线冻结之后，且需单独 runtime_authorization）
- 实盘读审计写 / broker facts 实盘写
- Git remote write / commit / push / 分支治理
- 任何 publish / live / production 写入

## 8. 必读文件（新会话恢复顺序）

1. 本文件（`process/context/CP6-CR139-WAVE1-IMPLEMENTATION-HANDOFF.md`）
2. `process/state/STATE.current.json`（机器状态真相源）
3. `process/changes/CR-139-STRATEGY-DATA-FOUNDATION-2026-06-28.md`（parent CR 审计边界 + §5 Wave 路线 + §9 不授权范围）
4. Active Story LLD：`process/stories/CR139-S01-remediation-object-inventory-LLD.md`（当前 S01）
5. `process/DEVELOPMENT-PLAN-CR139.yaml`（Wave 编排 + merge_order + 文件所有权 + LCQ）
6. `process/context/DATA-LAKE-REMEDIATION-HANDOFF.md` v0.7 §3（各整改项代码触点）

## 9. 下一步（新会话首个动作）

**实现 S01 T8 清册**（基线门第 1 步，无前置依赖）：
- 按 `process/stories/CR139-S01-remediation-object-inventory-LLD.md` §8 实施步骤，4 Slice（T8-1..T8-4）推进
- 新建 `scripts/lake_inventory.py` + `market_data/remediation_inventory.py`（d1 纯新建）
- 只读 catalog/manifest/readers，不写 lake/catalog
- 每 Slice 跑 `uv run --python 3.11 pytest -q tests/test_cr139_s01_inventory.py`
- S01 verified 后推进 S02 T7 黄金值（消费 S01 `build_inventory()`）

精确提示词：`推进阶段: story-execution (CR-139 Wave1 CP6, 实现 S01 T8 清册)`
