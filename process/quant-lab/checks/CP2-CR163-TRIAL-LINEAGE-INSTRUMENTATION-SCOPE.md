---
checkpoint_id: "CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE"
checkpoint_name: "CR163 Requirement, Scenario and Scope Baseline"
type: "auto_precheck"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-07-11T10:45:00+08:00"
checked_at: "2026-07-11T10:45:00+08:00"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md"]
manual_checkpoint: "process/checkpoints/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.md"
---

# CP2 CR163 Requirement, Scenario and Scope Baseline

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP1 已通过 | PASS | `process/checks/CP1-CR163-TRIAL-LINEAGE-INSTRUMENTATION-USE-CASE-COMPLETENESS.result.json` | 机器校验通过。 |
| 产品基线齐备 | PASS | `docs/product/*` 八份 CR163 增量文档 | 保留旧基线与修订记录。 |
| SGQ 用户确认完成 | PASS | `process/checks/CP2-CR163-DISCUSSION-CHECKPOINT.json` | 4/4 confirmed A，remaining=0。 |
| Context capsule 可用 | PASS | `process/context/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-CONTEXT.yaml` | compact capsule-first。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 2 条 producer chains / 4 mappings 分母明确 | PASS | `USE-CASES.md` / `REQ-CR163-005` | CP3 必须保持 4/4 覆盖。 |
| 2 | trial/attempt/seed/failure/exclusion 计数语义明确 | PASS | `REQ-CR163-003` | 不得后验缩小 family。 |
| 3 | immutable seal + supersession 明确 | PASS | `REQ-CR163-004` | 禁止 sealed 原地改写。 |
| 4 | availability 与 C1 claim ceiling 明确 | PASS | `REQ-CR163-006` | 仅 raw-input-ready，C1 不可计算。 |
| 5 | CR155 negative regression 明确 | PASS | `SC-CR163-G01` | 不补造 lineage，不提升 admission。 |
| 6 | 场景与测试追溯完整 | PASS | `SCENARIOS.yaml` / `TEST-MATRIX.md` | 12/12 P0。 |
| 7 | 五 Story 目标无范围漂移 | PASS | `STORY-MAP.md` | S03 覆盖两条 chains，Story 数仍为 5。 |
| 8 | 非授权与 deferred 隔离 | PASS | `MVP-SCOPE.md` / `BACKLOG.md` | statistical/runtime/backfill 不进入 CR163。 |
| 9 | 真实 meta-pm 调度证据存在 | PASS | dispatch/handoff ledgers | 无 inline fallback。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 致命/阻塞问题为 0 | PASS | CP2 result JSON | blockers=0。 |
| 可发起正式 CP2 人工门 | PASS | `process/checkpoints/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.md` | 人工结论尚待用户填写。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| CP2 result | `process/checks/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.result.json` | PASS | 机器真相源。 |
| CP2 capsule | `process/context/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-CONTEXT.yaml` | PASS | compact。 |
| Discussion checkpoint | `process/checks/CP2-CR163-DISCUSSION-CHECKPOINT.json` | PASS | 4/4 SGQ confirmed。 |
| Manual checkpoint | `process/checkpoints/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.md` | pending | 待正式人工确认。 |

## 结论

- 结论：`PASS`
- 阻断项：0
- 豁免项：0
- 下一步：打开正式 CP2 human gate；未 approved 前不得进入 CP3。
