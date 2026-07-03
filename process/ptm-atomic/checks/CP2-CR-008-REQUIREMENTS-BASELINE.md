# CP2 CR-008 Requirements Baseline

## Entry Criteria

- CP0 PASS。
- 用户原始需求包含 side_effect、rollback_strategy、schema、CLI、执行结果和 ptm-te 阻断规则。

## Checklist

| 项 | 结果 | 说明 |
|---|---|---|
| 需求可编号 | PASS | `REQ-RB-001` 至 `REQ-RB-014` 已保留并结构化。 |
| 验收条件可检验 | PASS | 每条需求包含 CLI、schema、atom 或 runner 输出验收入口。 |
| 范围边界明确 | PASS | 全量 atom 自动回滚、ptm-te rollback stack、设备快照持久化不在本 CR。 |
| 开放项状态化 | PASS | `RB-FU-001` 至 `RB-FU-003` 已标记 OPEN。 |

## Exit Criteria

CP2 PASS。允许进入 HLD。

## Deliverables

- `process/REQUIREMENTS-CR-008-ROLLBACK.md`
