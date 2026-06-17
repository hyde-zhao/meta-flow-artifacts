# CP5-CR061 Package / Import / Layout Readiness 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。CR061 的 candidate list、layout plan、preserve-audit decisions、rollback gate 和 manifest 已冻结。用户回复 `approve` 后，只表示接受 CR061 package/import/layout 设计门禁与后续实现准备路线；不授权立即修改 `pyproject.toml`、`uv.lock`、源码目录、import 路径或任何真实迁移动作。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR061-LLD-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；实现前再读具体源码 |
| 全文档读取扩展 | 已读取 CR061 manifest、candidate list、layout plan、rollback ref、preserve-audit |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前无既有 pending DQ。 |
| 自动预检结果 | `process/checks/CP5-CR061-PACKAGE-IMPORT-LAYOUT-READINESS.md` | scanned | 5 | 5 | readiness 风险转 DQ。 |
| execution manifest | `docs/release/MIGRATION-EXECUTION-MANIFEST-CR061.yaml` | scanned | 8 | 5 | 执行授权、blocked items 和顺序转 DQ。 |
| rollback ref | `docs/release/ROLLBACK-REF-CR061.md` | scanned | 7 | 2 | dirty worktree / stale bundle 转 DQ。 |
| preserve-audit | `docs/release/PRESERVE-AUDIT-DECISIONS-CR061.md` | scanned | 8 | 1 | 历史审计策略合并。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR061-01 | runtime_authorization | 是否批准 CR061 当前 CP5 门禁包？ | 批准门禁包，但不授权立即执行真实改动。 | 备选 A：要求补充更多 scan 后再审；备选 B：reject 并暂停 CR061。 | 推荐方案可进入实现准备；补 scan 更保守；reject 停止迁移。 | 仍需后续实现授权。 | 用户需要更细清单时返工 candidate list。 |
| DQ-CP5-CR061-02 | implementation | 后续第一实现切片是否限定为 metadata + compatibility namespace，而非 bulk move？ | 限定 Batch B：package metadata 和 optional compatibility namespace；不 bulk move。 | 备选 A：直接 bulk move；备选 B：只改 metadata。 | 推荐方案平衡价值和风险；bulk move 风险高；只改 metadata 价值低。 | legacy roots 会保留。 | 新 namespace 稳定后再申请 bulk move。 |
| DQ-CP5-CR061-03 | risk_acceptance | 是否接受 CR059 bundle 仅作为 planning rollback，真实实现前刷新或 waiver？ | 接受；实现前必须刷新 rollback_ref 或显式风险接受。 | 备选 A：现在创建新 bundle；备选 B：无 bundle 直接做。 | 推荐方案避免本门禁执行 Git 操作；新 bundle 更强但扩大动作；无 bundle 不可接受。 | 后续实现前存在阻断项。 | 实现开始前未刷新则 BLOCKED。 |
| DQ-CP5-CR061-04 | security | 是否确认 preserve-audit 和 sensitive filter 对 CR061 继续有效？ | 确认，不改历史过程证据，不读敏感正文。 | 备选 A：单项豁免；备选 B：批量改历史引用。 | 推荐方案保护审计；单项豁免需字段齐全；批量历史改写风险高。 | 历史 `local_backtest` 继续存在。 | 外发文档有具体需求时单项豁免。 |
| DQ-CP5-CR061-05 | runtime_authorization | 是否确认 CP5 approve 不授权 Git remote、NAS、data lake、provider、QMT/MiniQMT、凭据或 CR046 recovery？ | 确认不授权；这些分别归 CR062-CR065 或明确 CR046 recovery。 | 备选 A：合并 Git remote；备选 B：合并 NAS/lake/runtime。 | 推荐方案风险分层；备选会混合多个回滚面。 | 完整迁移需要后续 CR。 | 用户要求合并时重新做 CR 影响分析。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP5-CR061-001 | 当前不修改 `pyproject.toml`、`uv.lock` 或源码目录。 |
| NA-CP5-CR061-002 | file move / rename / delete。 |
| NA-CP5-CR061-003 | bulk import rewrite。 |
| NA-CP5-CR061-004 | 历史 `process/**`、checks、checkpoints、handoffs、Story、LLD、DEV-LOG 或历史 CR 改写。 |
| NA-CP5-CR061-005 | `MARKET_DATA_LAKE_ROOT` replacement、lake write、catalog publish、provider fetch。 |
| NA-CP5-CR061-006 | NAS 数据 copy / move / delete / archive promote。 |
| NA-CP5-CR061-007 | Git remote add / push / tag / branch rename / history rewrite。 |
| NA-CP5-CR061-008 | `.env`、token、password、cookie、session、private key、账户或交易事实读取。 |
| NA-CP5-CR061-009 | QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。 |
| NA-CP5-CR061-010 | 恢复或推进 CR046 CP7。 |

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR061 正式 CR 已创建 | PASS |
| execution manifest 已冻结 | PASS |
| candidate list 已冻结 | PASS |
| rollback refs 已记录 | PASS_WITH_RISK |
| preserve-audit 决策已写明 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 执行范围最小且明确 | PASS |
| 决策项完整 | PASS |
| 不授权项完整 | PASS |
| 回滚策略存在 | PASS_WITH_RISK |
| 用户授权 | PENDING |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户回复 `approve` 后只批准门禁包和实现准备路线，不直接执行真实迁移 | PASS |
| 用户回复 `修改: <具体修改点>` 后返工 | PENDING |
| 用户回复 `reject` 后停止 CR061 | PENDING |

## Deliverables

- `docs/release/MIGRATION-EXECUTION-MANIFEST-CR061.yaml`
- `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md`
- `docs/release/REPO-LOCAL-PACKAGE-LAYOUT-PLAN-CR061.md`
- `docs/release/PRESERVE-AUDIT-DECISIONS-CR061.md`
- `docs/release/ROLLBACK-REF-CR061.md`
- `process/checks/CP5-CR061-PACKAGE-IMPORT-LAYOUT-READINESS.md`
- `process/checks/CP5-CR061-HUMAN-GATE-LAUNCH-MESSAGE.md`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T07:52:05+08:00
- 修改意见：用户回复“同意”，按 `approve` 解析，接受 DQ-CP5-CR061-01..05 推荐方案。
- 风险接受项：接受 CR061 当前 CP5 门禁包和后续实现准备路线；`approve` 不授权立即修改 `pyproject.toml`、`uv.lock`、源码目录、import 路径、Git remote、NAS/data lake、runtime、凭据或 CR046 recovery。
