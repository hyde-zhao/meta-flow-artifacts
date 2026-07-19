---
checkpoint_id: "CP2-CR173-REQUIREMENTS-BASELINE"
checkpoint_name: "CR173 需求/场景/范围基线自动预检"
type: "auto_precheck"
status: "PASS"
owner: "meta-pm"
created_at: "2026-07-16T12:12:00+08:00"
checked_at: "2026-07-16T12:12:00+08:00"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "docs/product/RELEASE-SLICES.md"
    - "docs/product/BACKLOG.md"
manual_checkpoint: "process/checkpoints/CP2-CR173-REQUIREMENTS-BASELINE.md"
---

# CP2 CR173 需求/场景/范围基线自动预检

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP1 通过 | PASS | `process/checks/CP1-CR173-USE-CASE-COMPLETENESS.result.json` | blocker=0。 |
| 需求草案存在 | PASS | `docs/product/REQUIREMENTS.md` | CR173 8/8 P0。 |
| 场景/矩阵存在 | PASS | `SCENARIOS.yaml` / `TEST-MATRIX.md` | 8/8，trace 100%。 |
| 产品规划输入存在 | PASS | STORY-MAP / MVP / RELEASE / BACKLOG | outcome-only；正式 Story=0。 |
| 讨论证据存在 | PASS | CR173 discussion log/checkpoint | SGQ 1/1。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 功能需求完整 | PASS | REQ-CR173-001..008 | 8/8 P0。 |
| 2 | NFR 量化 | PASS | NFR-CR173-001..004 | alias、determinism、trace、security 可测。 |
| 3 | 范围清晰 | PASS | CR173 MVP Scope | In/Out/Deferred 完整。 |
| 4 | 验收标准明确 | PASS | CR173 requirements | 每项有精确目标。 |
| 5 | 约束条件记录 | PASS | REQ-005/008 | strategy-agnostic、fixture-only。 |
| 6 | 依赖和风险识别 | PASS | DQ-001..008 | 8 个 CP2 决策。 |
| 7 | 需求无隐含冲突 | PASS | CR173 conflict precheck + scope | CR172 overlap 已串行。 |
| 8 | 变更机制明确 | PASS | CR173 recovery contract | 不自动恢复 CR172。 |
| 9 | 追溯矩阵建立 | PASS | TEST-MATRIX CR173 rows | 8/8 rows，100%。 |
| 10 | Gray Areas 已处理 | PASS | discussion log | 4/4 有状态。 |
| 11 | Deferred 隔离 | PASS | UC/MVP/BACKLOG | activation 与相邻项未污染。 |
| 12 | 用户可见确认 | PASS | SGQ-CR173-001 | 1/1。 |
| 13 | 八维扫描 | PASS | UC CR173 八维表 | 8/8。 |
| 14 | 六类工程场景 | PASS | SCENARIOS | positive/negative/boundary/permission/recovery/precheck=6/6。 |
| 15 | MVP 可确认 | PASS | 四个 planning docs | 正式 Story/DAG/Wave/LLD=0。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| P0 基线无自动 blocker | PASS | 8 REQ / 8 SC / 8 matrix rows | blocker=0。 |
| 人工确认 | 待审查 | `process/checkpoints/CP2-CR173-REQUIREMENTS-BASELINE.md` | Host 生成并发起。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| 产品基线 8 文档 | `docs/product/*` | PASS | 全部增量更新。 |
| Discussion evidence | `process/discussions/CP2-CR173-SCENARIO-DISCUSSION-LOG.md` | PASS | completed。 |
| CP2 自动 result | `process/checks/CP2-CR173-REQUIREMENTS-BASELINE.result.json` | PASS | machine truth。 |

## 结论

- 结论：`PASS`
- 阻断项：`0`
- 人工门禁：`pending`
- 待决策：`DQ-CR173-001..008` 共 `8` 项
- `ready_for_design=false`
- 下一步：Host 生成 CP2 context/checkpoint/launch message 并等待用户人工决定。

