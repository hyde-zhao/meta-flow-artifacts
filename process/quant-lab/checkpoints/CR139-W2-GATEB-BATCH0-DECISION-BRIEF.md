# CR139 W2 Gate B Batch 0 Decision Brief

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Lake root aligned to NAS final directory | PASS | `/home/hyde/data/quant-lab/data-lake` | 旧 `/home/hyde/data/quant-lab/lake` 已改名 |
| Root-aligned Gate A no-write inventory complete | PASS | `process/evidence/CR139-W2-GATEA-REAL-LAKE-INVENTORY-DATALAKE-2026-06-29.json` | 17 catalog datasets, physical_missing_count=0 |
| Gate B Batch 0 preflight complete | PASS | `process/evidence/CR139-W2-GATEB-BATCH0-PREFLIGHT-2026-06-29.json` | operation_counts 全 0 |
| Gate B Batch 0 duplicate profile complete | PASS | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE.index.json` | key+metadata profile 覆盖 6 个重复键数据集和 events |
| Evidence schema consistency | PASS | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE-events-2026-06-29.json` | events 已统一到 `dataset_profiles[]`，不再使用顶层 `events_profile` |

## 审批者摘要

CR139 W2 的 root alignment、Gate A no-write inventory、Gate B Batch 0 preflight 和 duplicate profile 已完成。当前可以确认的是：本机 lake root 已对齐为 `/home/hyde/data/quant-lab/data-lake`，inventory 覆盖 17 个 catalog dataset，且本轮 Batch 0 没有执行 lake object 写入、catalog 写入、provider-lake-catalog 写入、pointer advance、物理迁移、NAS 操作、runtime 操作或凭据读取。

本次确认服务的整体目标是：在任何真实写入前，冻结重复键、events、fence、legacy path、lineage checksum 和 published pointer 的决策边界。推荐动作是 `approve` 本 brief 的推荐方案。Approve 后只允许生成 Gate B per-dataset write plan，不授权真实 lake 写入。Approve 不授权 catalog 写入、published pointer advance、物理迁移、凭据读取、NAS/QMT/MiniQMT/gateway runtime、trading/small_live/live 或 Git remote write。不确认会阻塞 Gate B Batch 1/2、Gate C、Gate D 和 Gate E。

## Decision Collection Coverage

| Source | Scanned | Candidate Issues | Included Decisions | N/A / Notes |
|---|---:|---:|---:|---|
| Gate A aligned check | yes | 5 | D-GATEB-006, D-GATEB-007, D-GATEB-008, D-GATEB-009 | S30/S34/Gate D/fence 均纳入 |
| Gate B Batch 0 preflight check | yes | 3 | D-GATEB-001, D-GATEB-002, D-GATEB-003, D-GATEB-006 | duplicate/events/fence 纳入 |
| Duplicate profile index | yes | 3 | D-GATEB-001, D-GATEB-002, D-GATEB-003, D-GATEB-005 | full-row profile scope 纳入 |
| Reviewer corrections | yes | 5 | D-GATEB-002, D-GATEB-003, D-GATEB-005, D-GATEB-010, D-GATEB-011 | evidence schema、排序键、events readiness、full-row 前置门、quarantine 形态纳入 |
| Non-authorization boundary | yes | 1 | D-GATEB-004, D-GATEB-009 | 是否生成 write plan、是否推进 pointer 纳入 |

Coverage summary: scanned sources = 5, candidate issue groups = 17 across sources (gross count, includes repeated issues), deduplicated decision items = 11, decision items included = 11, duplicates merged = 6, N/A = 0. Blocking/high items included: fence/quiescence, published pointer, duplicate-key business decision, events duplicate primary key.

## 决策分层

| 层级 | 决策项 | 推荐 |
|---|---|---|
| 必须用户决策 | 6 个重复键数据集的 keep/drop/quarantine 原则 | 默认 quarantine metadata-conflicting duplicate keys；仅对无冲突且排序键明确的重复行 deterministic dedup |
| 必须用户决策 | `events` 重复主键处理 | 不修改 W2 events 合同；先 quarantine duplicate primary keys，保留当前主键 `(symbol,event_type,event_date,available_at)` |
| 必须用户决策 | fence / quiescence | Batch 1/2 写入前建立 fenced window 并重跑 Gate A；不能证明 fence 时必须显式接受漂移风险 |
| 高风险策略确认 | Gate E legacy path 范围 | 本轮 write plan 先不执行迁移；记录全量 17 dataset legacy path 决策，Gate C/D 前必须选全量迁移或 publish-target-only + waiver |
| 高风险策略确认 | lineage checksum 时机 | Gate C 前生成 checksum backfill plan；不得在 checksum plan 缺失时进入 Gate C/D |
| 高风险策略确认 | published pointer | 本 CR 当前阶段不推进任何 dataset 到 published；Gate D 留待独立人工授权 |
| Agent 默认处理 | 生成 per-dataset write plan | 只做 plan/evidence，不写 lake，直到明确授权 Batch 1/2 |
| Batch 1/2 前置门 | full-row exact duplicate profile | 每个将写 dataset 必须先有 per-dataset full-row profile 或 explicit N/A with reason |
| Batch 1/2 前置门 | quarantine physical shape | write plan 必须定义新 CR139 run-id 路径下的 `quarantine/` 子区，append-only，不碰 legacy |

## Batch 0 Evidence

| Evidence | Path |
|---|---|
| Gate A aligned inventory | `process/evidence/CR139-W2-GATEA-REAL-LAKE-INVENTORY-DATALAKE-2026-06-29.json` |
| Gate B Batch 0 preflight | `process/evidence/CR139-W2-GATEB-BATCH0-PREFLIGHT-2026-06-29.json` |
| Duplicate profile index | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE.index.json` |
| adj_factor profile | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE-adj_factor-2026-06-29.json` |
| liquidity_capacity profile | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE-liquidity_capacity-2026-06-29.json` |
| market_cap profile | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE-market_cap-2026-06-29.json` |
| prices profile | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE-prices-2026-06-29.json` |
| prices_limit profile | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE-prices_limit-2026-06-29.json` |
| trade_status profile | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE-trade_status-2026-06-29.json` |
| events profile | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE-events-2026-06-29.json` |

## Findings

| Dataset | Duplicate Key Groups | Duplicate Rows In Groups | Rows Over Unique | Metadata Conflict Signal |
|---|---:|---:|---:|---|
| adj_factor | 11,824,907 | 23,889,777 | 12,064,870 | source_run_id/lineage conflicts: 81,852; available_at conflicts: 74,960 |
| liquidity_capacity | 1,830 | 3,660 | 1,830 | source_run_id/lineage conflicts: 1,830 |
| market_cap | 106,776 | 213,552 | 106,776 | source_run_id/lineage conflicts: 106,776 |
| prices | 11,313,190 | 22,866,262 | 11,553,072 | source_run_id/lineage conflicts: 81,788 |
| prices_limit | 17,356,374 | 38,609,095 | 21,252,721 | source_run_id/lineage conflicts: 7,445,795; available_at conflicts: 74,956 |
| trade_status | 3,762,192 | 7,682,591 | 3,920,399 | source_run_id/lineage conflicts: 3,762,192; available_at conflicts: 74,956 |
| events | 11,357 | 22,717 | 11,360 | source_run_id/lineage conflicts: 11,357 |

Count note: duplicate key groups are unique duplicate primary-key groups; rows over unique are `row_count - unique_key_count`. For events, `11,357` groups and `11,360` rows over unique are consistent because some groups contain more than two rows. If duplicate primary keys are quarantined, events usable canonical rows are `362,887 - 11,360 = 351,527`; this number must enter readiness evaluation.

## 推荐决策

| Decision ID | Type | Recommendation | Rationale | Alternative | Risk / Rollback |
|---|---|---|---|---|---|
| D-GATEB-001 | implementation | 对 metadata-conflicting duplicate keys 默认 quarantine，不自动选择覆盖值 | 多个 dataset 的重复键存在 source_run_id/lineage 或 available_at 冲突，直接 keep-last 会改变数据真相 | 全部 deterministic keep-last | 高风险；如后续发现策略错误，需要重写 candidate run 并废弃旧 manifest |
| D-GATEB-002 | implementation | 对无 metadata 冲突的重复键，可在 per-dataset plan 中申请 deterministic dedup，但必须写明显式排序键 | 无排序键的 dedup 等价于隐式 keep-last；排序键必须进入 evidence | 全部 quarantine | 更保守但会降低可写覆盖率 |
| D-GATEB-003 | implementation | `events` 暂不改合同，按当前主键重复 quarantine；readiness 使用 351,527 可用行数 | W2 events 合同不要求 `trade_date`；当前 blocker 是 `(symbol,event_type,event_date,available_at)` 重复 | 设计 delta 扩展 events 主键或恢复 trade_date 字段 | 修改合同影响 S31/S39 审计链，需重新走设计 delta |
| D-GATEB-004 | runtime_authorization | 允许下一步只生成 Gate B Batch 1/2 per-dataset write plan，不执行写入 | 先把 candidate、quarantine、row/hash 预期和 rollback manifest 规划出来 | 直接授权 Batch 1 小表写入 | 可行但少一层 plan 审查；不推荐 |
| D-GATEB-005 | risk_acceptance | 接受本轮未做全局 full-row exact duplicate 统计，但每个将写 dataset 的 full-row profile 是 Batch 1/2 前置门 | 当前 key+metadata profile 足以阻断不安全写入；写入前必须补目标 dataset 级 full-row 证据 | 对所有 dataset 立即做 full-row profile | 更完整但成本高，可能造成不必要的全表扫描 |
| D-GATEB-006 | runtime_authorization | Batch 1/2 写入前建立 fenced window 并重跑 Gate A；若不能 fence，必须单独记录 drift risk acceptance | Gate A/Batch0 均未机械证明外部 quiescence | 直接接受漂移风险 | 高风险；pre-write snapshot 可能过期 |
| D-GATEB-007 | architecture | Gate E 范围先不执行；write plan 必须显式记录“全量 17 dataset 迁移”与“publish-target-only + waiver”两个选项 | 17/17 catalog paths 非 CR139 run-id 命名，会影响 Gate C/D path refs | 现在就全量迁移 | 当前未准备 checksum/rollback map，不建议 |
| D-GATEB-008 | implementation | lineage checksum backfill plan 必须在 Gate C 前生成；不得在 checksum 缺失时进入 Gate C/D | 17/17 缺 lineage checksum，Gate A 标记为 Gate C/D blocker | 推迟到 Gate D 后 | 不可接受；会让 current truth 缺 lineage |
| D-GATEB-009 | scope | 本 CR 当前阶段不推进任何 dataset 到 published；Gate D 留待独立人工授权 | 17/17 unpublished，pointer advance 是 current truth mutation | 在本 CR 内推进部分 dataset | 可行但必须另起 Gate D review |
| D-GATEB-010 | implementation | write plan 必须定义 quarantine 物理形态：新 CR139 run-id 路径下 `quarantine/` 子区，append-only，带 conflict metadata，不碰 legacy | legacy path 不应在 Gate B 被改写；quarantine 必须可审计可回滚 | 在 legacy 路径标记 quarantine | 会破坏 additive-only 边界 |
| D-GATEB-011 | implementation | duplicate profile evidence schema 统一为 `dataset_profiles[]`；write plan consumer 只能消费统一 schema | 避免 events 被下游解析器漏掉 | 继续保留 `events_profile` | 不可接受，审计链易漏项 |

## Write Plan Hard Requirements

下一步生成 write plan 时必须满足：

1. 只生成 plan/evidence，不执行真实 lake 写入。
2. 所有 profile consumer 使用统一 `dataset_profiles[]` schema。
3. deterministic dedup 必须写明显式排序键，例如 `ORDER BY source_run_id DESC, available_at DESC`，并说明为何该排序代表业务真相。
4. `events` quarantine 后可用行数 `351,527` 必须进入 readiness 评估。
5. 每个将写 dataset 必须先有 per-dataset full-row exact duplicate profile，或明确 N/A with reason。
6. Quarantine 物理形态必须是新 CR139 run-id 路径下 `quarantine/` 子区，append-only，带 conflict metadata，不碰 legacy path。
7. Batch 1/2 写入前必须建立 fence/quiescence 并重跑 Gate A，或单独记录 drift risk acceptance。
8. Gate E 范围、Gate C checksum backfill、Gate D publish pointer 均只做计划，不执行。

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| All blocking/high evidence findings represented as decisions | PASS | D-GATEB-001..011 | fence/publish/run-id/checksum/events/duplicates 已覆盖 |
| Exact user reply options provided | PASS | 本文件 `人工审查结果` | approve / 修改 / reject 三选一 |
| Next step remains non-mutating | PASS | D-GATEB-004 and Write Plan Hard Requirements | 只生成 write plan |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Decision Brief | `process/checkpoints/CR139-W2-GATEB-BATCH0-DECISION-BRIEF.md` | READY_FOR_REVIEW | 本文件 |
| Duplicate profile index | `process/evidence/CR139-W2-GATEB-BATCH0-DUPLICATE-PROFILE.index.json` | PASS | key+metadata profile 汇总 |
| Batch0 preflight check | `process/checks/CR139-W2-GATEB-BATCH0-PREFLIGHT-2026-06-29.md` | PASS_WITH_BLOCKERS | 阻断写入，要求决策 |

## 不授权范围

- 不授权 Gate B Batch 1/2 真实 lake 写入
- 不授权 catalog/provider-lake-catalog 写入
- 不授权 published pointer advance
- 不授权物理分区迁移
- 不授权凭据读取、NAS/QMT/MiniQMT/gateway runtime、trading/small_live/live 或 Git remote write

## 人工审查结果

请使用以下 exact 回复之一：

- `approve`
- `修改: <具体修改点>`
- `reject`

`approve` 表示接受 D-GATEB-001..011 的推荐方案，并仅授权下一步生成 Gate B per-dataset write plan；不授权任何真实写入、catalog 写入、pointer advance 或物理迁移。

Review result:

- Result: `approved`
- Approved at: `2026-06-29T13:54:41+08:00`
- Approver: `user`
- Approved scope: Generate Gate B per-dataset write plan and supporting evidence only.
- Still not authorized: real lake writes, catalog/provider-lake-catalog writes, published pointer advance, physical partition migration, credential reads, NAS/QMT/MiniQMT/gateway runtime, trading/small_live/live, Git remote write.
