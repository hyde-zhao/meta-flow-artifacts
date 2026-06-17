---
check_id: "CR-STATUS-CONSISTENCY-AUDIT-2026-06-17"
status: "PASS_WITH_NOTES"
checked_at: "2026-06-17T20:10:23+08:00"
owner: "host-orchestrator"
current_root: "/home/hyde/workspace/quant-lab"
legacy_root: "/home/hyde/workspace/local_backtest"
authoritative_process_root: "/home/hyde/workspace/meta-flow-artifacts/process/quant-lab"
---

# CR 状态一致性审计

## 结论

当前权威根 `/home/hyde/workspace/quant-lab` 的 CR tracking 机器一致性检查通过；旧根 `/home/hyde/workspace/local_backtest` 的 CR tracking 机器一致性检查也通过。

非迁移类正式 CR 在当前根与旧根之间未发现状态差异。迁移 / 发布 / 过程路由相关 CR 在当前根均为关闭态或已关闭的替代态。

需要注意：旧根 `process/STATE.md` 正文尾部仍保留历史摘要“当前 active formal CR 为 CR-020”，这是旧根遗留文本；当前权威根已经纠正为 `CR-046`。不得用旧根 `STATE.md` 覆盖当前权威状态。

## 权威状态

| 项 | 当前权威结论 |
|---|---|
| 当前 active CR | `CR-046` |
| CR046 状态 | `active-cp6-pass-ready-for-verification`，用户暂停在 CP7 前 |
| CR020 状态 | `deleted-by-user` |
| CR021..CR024 状态 | `cancelled-user-deleted` |
| QMT / MiniQMT runtime | 未授权 |
| 凭据 / 账户读取 | 未授权 |

## 迁移相关 CR 关闭核对

以下迁移 / 发布 / 过程路由相关 CR 在当前根均已关闭或等价关闭：

| CR | 状态 |
|---|---|
| `CR-053` | `closed-current-delivery` |
| `CR-058` | `closed-current-delivery` |
| `CR-059` | `closed-current-delivery` |
| `CR-060` | `closed-current-delivery` |
| `CR-061` | `closed-current-delivery` |
| `CR-062` | `closed-blocked-superseded` |
| `CR-066` | `closed-current-delivery` |
| `CR-067` | `closed-current-delivery` |
| `CR-068` | `closed-current-delivery` |
| `CR-069` | `closed-current-delivery` |
| `CR-070` | `closed-current-delivery` |
| `CR-071` | `closed-current-delivery` |
| `CR-072` | `closed-current-delivery` |
| `CR-073` | `closed-current-delivery` |
| `CR-074` | `closed-current-delivery` |
| `CR-075` | `closed-current-delivery` |
| `CR-076` | `closed-current-delivery` |
| `CR-078` | `closed-current-delivery` |
| `CR-079` | `closed-current-delivery` |
| `CR-080` | `closed-current-delivery` |
| `CR-081` | `closed-current-delivery` |
| `CR-082` | `closed-current-delivery` |
| `CR-083` | `closed-current-delivery` |
| `CR-084` | `closed-superseded-by-cr085` |
| `CR-085` | `closed-current-delivery` |
| `CR-086` | `closed-current-delivery` |
| `CR-087` | `closed-local-remediation` |
| `CR-088` | `closed-local-remediation` |

## 非迁移 CR 差异核对

当前根与旧根的非迁移类正式 CR frontmatter 状态比对结果：`NONE`。

也就是说，除迁移 / 发布 / 过程路由 CR 外，旧根没有发现需要搬入当前根的状态更新。

## 留意项

以下是当前根正式 CR 文件里仍不是标准 `closed*` / `cancelled*` / `deleted-by-user` 的状态：

| CR | 状态 | 处理意见 |
|---|---|---|
| `CR-010` | `verified-limited-window-pass-pending-close-decision` | 历史数据湖能力状态，当前根与旧根一致；若要标准化关闭，需要单独决策。 |
| `CR-018` | `verified-current-truth-cr029-accepted-s09-later-gated-pending-close` | 历史 production current truth / QMT 后置状态，当前根与旧根一致；若要标准化关闭，需要单独决策。 |
| `CR-031` | `doc-consolidation` | 历史文档归一化状态，当前根与旧根一致；若要标准化关闭，需要单独决策。 |
| `CR-032` | `completed-offline`（正文状态，缺少 YAML frontmatter） | 历史离线整改状态，当前根与旧根一致；若要标准化关闭，需要单独决策。 |
| `CR-033` | `implemented` | 历史因子库边界整改状态，当前根与旧根一致；若要标准化关闭，需要单独决策。 |
| `CR-046` | `active-cp6-pass-ready-for-verification` | 当前唯一业务 active CR，保持不变。 |

## 执行过的检查

```bash
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root /home/hyde/workspace/quant-lab
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root /home/hyde/workspace/local_backtest
```

两条命令均为 `CR tracking consistency: PASS`。

## 不执行项

- 未移动 `/home/hyde/workspace/local_backtest` 中的文件。
- 未用旧根 `STATE.md` 覆盖当前权威 `STATE.md`。
- 未改变迁移相关 CR 的关闭状态。
- 未恢复 CR046。
- 未连接 QMT / MiniQMT。
- 未读取凭据、账户、资金、持仓、委托或成交信息。
