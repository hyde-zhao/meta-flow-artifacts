---
story_id: "CR155-S02-readonly-data-provenance-adapter"
title: "Readonly data provenance adapter and guardrails"
story_slug: "readonly-data-provenance-adapter"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "meta-dev"
created_at: "2026-07-04T18:37:08+08:00"
confirmed_by: ""
confirmed_at: ""
shared_fragments: []
feature_design_refs:
  - "docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md"
  - "docs/features/runtime-authorization-safety/DESIGN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["readonly governed lake boundary", "security guardrails", "provenance evidence"]
  rationale: "Readonly local lake approval must be enforceable and auditable before validation can claim real local data input."
open_items: 0
---

# LLD: CR155-S02 — Readonly Data Provenance Adapter

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | Isolated readonly validation pipeline. |
| ADR | `docs/design/ARCHITECTURE-DECISION-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | ADR-CR155-002 and ADR-CR155-005. |
| Feature DESIGN | `docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md` | Readonly provenance and forbidden counters. |
| Safety Feature | `docs/features/runtime-authorization-safety/DESIGN.md` | Deny-default authorization boundary. |

## 1. Goal

Define a CR155-scoped readonly provenance adapter contract for local governed lake/current truth inputs without authorizing writes or runtime operations.

## 2. Requirements

### 2.1 Functional

- Record dataset refs, current truth/snapshot refs and read scope.
- Emit provenance evidence with operation counters.
- Downgrade to fixture/static if readonly provenance cannot be proven.

### 2.2 Non-Functional

- Counters for lake_write, catalog_pointer_mutation, credential_read, NAS, provider_fetch, runtime and trading must remain zero.
- No credential or `.env` read is allowed.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/daily_multifactor_baseline_artifact.py` | Candidate readonly provenance object | Keeps refs and counters. |
| `tests/research/test_daily_multifactor_baseline_artifact.py` | Candidate guardrail tests | Verifies forbidden counters and downgrade behavior. |
| `process/evidence/CR155-*` | Later evidence | Only created in CP6/CP7 if authorized by CP5. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add `ReadonlyDataProvenance` and guardrail validator after CP5 approval. |
| 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add forbidden operation and fallback tests. |
| 不修改 | lake/catalog/NAS/provider/runtime files | This Story must not write or mutate external state. |

## 5. 数据模型与持久化设计

No persistent lake/catalog object is created.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `ReadonlyDataProvenance.input_refs` | list[str] | required | Dataset/snapshot/current truth refs. |
| `read_scope` | object | required | CR155-only purpose and date range. |
| `operation_counts` | object | required | Forbidden counters must be zero. |
| `fallback_mode` | enum | optional | `fixture_static` when provenance cannot be proven. |
| `claim_allowed` | bool | derived | false if fallback mode used. |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `validate_readonly_provenance` | provenance | status/reasons | S03 | Blocks validation claims on unsafe counters. |
| `build_provenance_summary` | refs/counters | summary dict | S01/S03/S05 | Deterministic evidence summary. |
| `downgrade_to_fixture_static` | reason | fallback provenance | S03 | Preserves audit trail without real-data claim. |

## 7. 核心处理流程

1. Receive readonly input refs from later validation setup.
2. Build provenance summary with operation counters.
3. Validate forbidden counters are zero.
4. Return PASS or fallback/blocking status.

## 8. 技术设计细节

- Readonly approval does not imply catalog pointer mutation.
- Provenance is refs-only at design time; actual read is not executed at CP5.
- If CP6/CP7 cannot prove readonly source, S03 must label outputs as fixture/static.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | Deny-default counters and no credential/env access. | Guardrail tests. |
| 性能 | Provenance object stores refs, not data frames. | Contract tests. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| All counters zero | readonly refs present | validate | PASS | unit test |
| Lake write counter nonzero | simulated counter | validate | BLOCKED | unit test |
| Missing provenance refs | no refs | validate | fallback/static or BLOCKED | unit test |
| Credential attempt counter nonzero | simulated counter | validate | BLOCKED | unit test |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR155-S02-01 | 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add provenance object and status enum. | provenance tests |
| TASK-CR155-S02-02 | 创建 | same | Add deny-default validator and fallback builder. | guardrail tests |
| TASK-CR155-S02-03 | 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add counter/fallback tests. | all S02 tests |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-CR155-S02-01 | CP5 是否授权实际 lake read | 推荐不授权；CP5 只批准设计，实际 readonly read 留到 CP6/CP7 在 CP2 边界内执行 | agent default | 安全 | CP2/CP3 approvals | 若用户要求 CP5 运行数据读取，需独立 runtime authorization |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| Readonly 被误读为 write/publish | 高 | Explicit counters and release wording. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | N/A | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

- 发布方式：contract-only until CP6/CP7 evidence.
- 回滚触发条件：provenance cannot distinguish readonly from write/runtime.
- 回滚动作：force fixture/static fallback and remove real-data claim.

## 14. Definition of Done

- [ ] Provenance validator blocks nonzero forbidden counters.
- [ ] Fallback/static path is explicit.
- [ ] No CP5-time data read is executed.
- [ ] CP5 confirms this LLD before implementation.

## 人工确认区

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
