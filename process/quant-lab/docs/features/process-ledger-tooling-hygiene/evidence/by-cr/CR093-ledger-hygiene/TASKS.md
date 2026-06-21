---
status: draft
version: "1.0"
feature_id: "F-CR093-01"
source_cr: "CR-093"
created_at: "2026-06-18T17:55:00+08:00"
owner: "host-orchestrator"
---

# CR093 Ledger Hygiene TASKS

| TASK-ID | 动作 | 文件范围 | 输出 | 验证入口 |
|---|---|---|---|---|
| TASK-CR093-01 | 新增测试 fixture / 单元测试 | `tests/test_cr093_cr_tracking_consistency.py` | 当前状态 / audit-history / 状态等价测试 | `pytest -q tests/test_cr093_cr_tracking_consistency.py` |
| TASK-CR093-02 | 重构 checker 当前状态读取 | `scripts/check_cr_tracking_consistency.py` | current / tracking-current / audit-history 分层 | TP-CR093-01/02/05 |
| TASK-CR093-03 | 引入状态等价表 | `scripts/check_cr_tracking_consistency.py` | closed / cancelled / active normalized status | TP-CR093-03/04 |
| TASK-CR093-04 | 最小规范化 CR019 tracking | `process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md` | 消除必要的状态歧义，保留历史 | diff review |
| TASK-CR093-05 | 回写状态和证据 | `process/STATE.md`、`process/checks/*CR093*`、`process/checkpoints/*CR093*` | CP6 / CP7 输入 | workspace / cr-tracking |

## 执行顺序

1. 先写失败测试，证明当前 CR019 / CR025 旧账问题。
2. 修改 checker 分层读取和状态等价。
3. 做 CR019 最小规范化。
4. 运行测试与 cr-tracking。
5. 写 CP6 / CP7 证据。

## 文件 Owner

| 文件 | owner | 并行限制 |
|---|---|---|
| `scripts/check_cr_tracking_consistency.py` | meta-dev | 独占 |
| `tests/test_cr093_cr_tracking_consistency.py` | meta-dev | 可与文档更新串行 |
| `process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md` | host-orchestrator / meta-dev | checker 测试通过后修改 |
