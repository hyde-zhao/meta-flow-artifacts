---
status: "ready_with_risk"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "compact"
change_id: "CR-138"
created_at: "2026-06-24T18:58:00+08:00"
---

# CR138 Rollback

## 回滚范围

CR138 当前交付未执行真实发布、未启动 runtime、未写外部系统。回滚只涉及本地源码、测试、文档和过程产物。

| 对象 | 回滚方式 |
|---|---|
| Runner control plane files | 回退 `trading/runner_control_*.py`。 |
| Gateway service-layer changes | 回退 `trading/qmt_gateway_*` 中 CR138 增量。 |
| Tests | 回退 `tests/test_cr138_*.py` 和 hygiene regression 增量。 |
| Docs | 回退 CR138 authorization runbook、gateway install doc、release/quality/process 文档。 |
| Process state | 回退 CR138 CP6/CP7/CP8 和 CR index/status 增量。 |

## 不需要回滚的外部对象

| 对象 | 原因 |
|---|---|
| QMT / MiniQMT / XtQuant runtime | 未启动、未连接。 |
| Credentials / account / market / order data | 未读取。 |
| submit / cancel / simulation / live | 未执行。 |
| NAS / provider / lake / catalog | 未访问或写入。 |
| Git remote | 本轮未执行远端写入。 |

## 回滚验证

回滚后至少运行：

```bash
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr019_qmt_gateway_lifecycle.py tests/test_cr020_query_positions_readonly.py tests/test_cr019_qmt_gateway_run_gates.py tests/test_cr137_runner_run_registry.py
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root /home/hyde/workspace/quant-lab
git diff --check
```
