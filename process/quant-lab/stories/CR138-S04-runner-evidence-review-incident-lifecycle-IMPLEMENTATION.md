---
story_id: "CR138-S04-runner-evidence-review-incident-lifecycle"
status: "implemented"
owner: "host-orchestrator"
implemented_at: "2026-06-24T16:17:40+08:00"
cp6: "PASS"
---

# CR138-S04 Implementation

## 实现对象

| 对象 | 路径 | 说明 |
|---|---|---|
| Evidence / review / incident | `trading/runner_control_plane.py` | `query_run_evidence`、`build_review_summary`、`record_incident`、`propose_strategy_change`。 |
| Tests | `tests/test_cr138_runner_evidence_review_incident_lifecycle.py` | raw refs blocked、incident follow-up、rollback target required。 |

## 设计契约映射

| LLD 契约 | 实现位置 | 验证 |
|---|---|---|
| raw evidence ref blocked | `query_run_evidence()` | `test_run_evidence_blocks_raw_refs_and_allows_redacted_refs_only` |
| unresolved incident 形成 follow-up candidate | `build_review_summary()` | `test_review_summary_links_unresolved_incidents_to_follow_up_candidates` |
| StrategyChangePlan rollback required | `propose_strategy_change()` | `test_strategy_change_plan_requires_rollback_target_and_never_applies` |

## 验证结果

CR138 定向测试 PASS：48 passed。

## 不授权边界

证据只保存 redacted refs，不复制 raw log、账户原文或 broker payload；策略变更只 dry-run，不 apply。
