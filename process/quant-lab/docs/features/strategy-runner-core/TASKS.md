---
status: "active"
version: "1.0"
feature_id: "strategy-runner-core"
source_cr: "CR-128"
---

# Feature Tasks: Strategy Runner Core

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-23 | host-orchestrator | 将 CR128 runner core MVP 收敛为单 CR 内部任务。 |

| TASK-ID | 顺序 | 任务 | 输出文件 | 验证入口 | 状态 |
|---|---:|---|---|---|---|
| CR128-S1 | 1 | 定义 `RunSpec` 和 `RunResult` 合同。 | `trading/strategy_runner/run_spec.py`, `trading/strategy_runner/result.py` | CR128 tests | planned |
| CR128-S2 | 2 | 编排 package -> adapter -> fake readonly -> evidence -> result。 | `trading/strategy_runner/runner.py` | CR128 tests | planned |
| CR128-S3 | 3 | 暴露 public API，并让旧 CR091 checker 复用正式 runner API。 | `trading/strategy_runner/__init__.py`, `scripts/check_cr091_strategy_runner_package.py` | CR091 checker smoke | planned |
| CR128-S4 | 4 | 增加 runner core MVP 自动化测试。 | `tests/test_cr128_runner_core_mvp.py` | pytest | planned |
| CR128-S5 | 5 | 治理 feature 目录，删除 CR103/104 runtime 碎片。 | `process/docs/features/strategy-runner-core/*` | file review | planned |
| CR128-S6 | 6 | 写入 CP6/CP7/CR evidence。 | `process/checks/*CR128*` | meta-flow checks | planned |

## 阻塞项

| Blocker ID | 问题 | 处理 |
|---|---|---|
| BLK-CR128-01 | 真实 runtime / NAS / provider / trading 不在授权范围内。 | 不实现；如需要另起 CR129+。 |
| BLK-CR128-02 | CR102/103/104 feature 碎片不能直接删除历史证据。 | 只从默认 feature 面删除；历史保留在 CR / checks / release context。 |
