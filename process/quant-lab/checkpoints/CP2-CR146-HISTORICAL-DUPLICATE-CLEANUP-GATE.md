---
checkpoint_id: "CP2-CR146-HISTORICAL-DUPLICATE-CLEANUP-GATE"
checkpoint_name: "CR146 Historical Duplicate Cleanup Gate"
type: "auto_then_manual"
status: "pending_user_decision"
owner: "host-orchestrator"
created_at: "2026-07-01T10:34:00+08:00"
source_cr: "CR-146"
parent_cr: "CR-139"
auto_check_result: "process/checks/CP2-CR146-HISTORICAL-DUPLICATE-CLEANUP-GATE.result.json"
context_ref: "process/context/CP2-CR146-HISTORICAL-DUPLICATE-CLEANUP-GATE-CONTEXT.yaml"
decision_brief_required: true
auto_final_authorization: false
---

# CP2 CR146 Historical Duplicate Cleanup Gate

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR146-HISTORICAL-DUPLICATE-CLEANUP-GATE.result.json` | PASS | 0 | 自动预检通过，下一步必须人工决策；未授权任何 cleanup write/delete。 |
| `process/evidence/CR146-PRODUCTION-LAKE-HISTORICAL-DUPLICATE-CLEANUP-PLAN-POST-N1.index.json` | PASS_WITH_RISK | 0 | 只读 split planning 完成；5 个 dataset 需要策略决策。 |
| `process/evidence/CR146-PRODUCTION-LAKE-CURRENT-TRUTH-PROFILE-FULL-POST-N1-2026-07-01.json` | PASS | 0 | catalog current truth 仍是 17 published datasets / 103,264,532 rows / duplicate_key_total=0。 |
| `process/evidence/CR146-PRODUCTION-LAKE-INVENTORY-FULL-POST-N1-2026-07-01.json` | PASS_WITH_RISK | 0 | physical_missing_count=0；full-root duplicate_key_total 仍属于历史根 cleanup 范围。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-146 是否要在 N1 完成后继续执行 historical-root duplicate cleanup/rewrite，还是把该风险保留为后续分数据集门禁。 |
| 推荐动作 | `approve` 默认方案：不执行全量 historical cleanup，不做 business-conflict semantic selection，不删除/archive legacy；只把 FU-CR139-001 保持为 open，并允许后续先为 `adj_factor` 起草低风险 dataset-level cleanup 门禁。 |
| approve 后会发生什么 | 我会关闭本次“全量 cleanup 是否立即执行”的决策为“不立即执行”，把 FU-CR139-001 细分为 dataset-level gates；若继续推进，只先准备 `adj_factor` exact-copy/metadata-only cleanup 的独立门禁，不自动写 lake。 |
| approve 不授权什么 | `approve` 不授权历史 parquet rewrite、business-conflict semantic selection、full-group quarantine 写入、catalog pointer mutation、legacy delete/archive、NAS sync/write/restore drill、provider fetch、credential read、trading/runtime/Git remote write。 |
| 不确认会阻塞什么 | 不确认会阻塞 FU-CR139-001 的分流关闭和任何 historical-root cleanup 执行；不阻塞已完成的 N1 current layout 和 catalog current truth 只读使用。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR146-HISTORICAL-DUPLICATE-CLEANUP-GATE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；只在审计冲突、字段不足、人工深度评审或用户要求时读取完整 plan detail。 |
| 全文档读取扩展 | 已读取 post-N1 current truth、inventory、duplicate split plan index、N1 execution summary；未读取 `.env`、NAS 凭据或完整 parquet。 |
| 缺失 / waived 理由 | N/A；本门禁已有 context capsule、CP result 和 post-N1 split planning evidence。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| FU-CR139-001 tracking | `process/changes/CR-139-FOLLOW-UP-TRACKING-2026-06-30.md#FU-CR139-001` | scanned | 1 | 1 | historical-root cleanup/rewrite 仍 open，必须进入本门禁。 |
| post-N1 duplicate split plan | `process/evidence/CR146-PRODUCTION-LAKE-HISTORICAL-DUPLICATE-CLEANUP-PLAN-POST-N1.index.json` | scanned | 5 | 3 | 5 个 dataset 合并为 defer/adj_factor canary/business-conflict policy 三类决策。 |
| post-N1 current truth | `process/evidence/CR146-PRODUCTION-LAKE-CURRENT-TRUTH-PROFILE-FULL-POST-N1-2026-07-01.json` | scanned | 1 | 0 | current truth duplicate_key_total=0；不需要为读层可用性立即 cleanup。 |
| post-N1 inventory | `process/evidence/CR146-PRODUCTION-LAKE-INVENTORY-FULL-POST-N1-2026-07-01.json` | scanned | 1 | 1 | full-root duplicate surface 仍存在，但不影响 catalog current truth；纳入风险接受决策。 |
| CR146 authorization boundary | `process/changes/CR-146-CR139-POST-CLOSE-LAKE-COMPLETION-2026-06-30.md` | scanned | 1 | 1 | business-conflict, delete/archive, NAS/provider/credential 均需要独立授权。 |
| N1 execution evidence | `process/checks/CP7-CR146-PRODUCTION-LAKE-N1-MIGRATION.result.json` | scanned | 1 | 0 | N1 已完成，不再纳入本门禁执行范围。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 2 | DQ-CP2-CR146-HIST-01、DQ-CP2-CR146-HIST-02。 |
| 高风险策略确认 | 1 | DQ-CP2-CR146-HIST-03。 |
| agent 默认处理 | 1 | 若 approve，记录“全量 cleanup 不立即执行”，并仅准备 `adj_factor` 独立门禁草案；不写 lake。 |
| 仅审计记录 | 1 | post-N1 split planning 是 read-only；operation_counts 均为 0。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR146-HIST-01 | risk_acceptance | 是否立即执行全量 historical-root duplicate cleanup/rewrite？ | 不立即执行；保留 FU-CR139-001 open，并按 dataset-level gate 分流。 | A. 直接全量 cleanup；B. 只执行 exact-copy drop；C. 只写 quarantine。 | 推荐方案避免把 258,388,545 行历史根数据、72,710,457 个 duplicate key group 和 4,272,624 个 business-conflict group 混入一次高风险写入；A 风险最大；B/C 仍需重写和 catalog 边界。 | full-root 扫描仍会看到重复；但 catalog current truth 已干净，读层不受影响。 | 若后续必须降低存储/扫描成本或全根工具必须无重复，再启动 dataset-level cleanup gate。 |
| DQ-CP2-CR146-HIST-02 | runtime_authorization | 是否允许先准备 `adj_factor` 低风险 cleanup 门禁？ | 允许准备门禁草案，但不授权执行；`adj_factor` 无 business-conflict groups，适合作为 cleanup canary 候选。 | A. 不准备任何 cleanup 门禁；B. 同时准备 5 个 dataset 门禁。 | 推荐方案最小化下一步认知负担；A 会让 FU-CR139-001 停滞；B 会一次性暴露过多语义决策。 | 只写 process/checkpoints，不写 lake；不会改变数据。 | 若用户要求暂停 cleanup，保持 FU-CR139-001 candidate/open。 |
| DQ-CP2-CR146-HIST-03 | risk_acceptance | 对 prices/prices_limit/events/trade_status 的 business-conflict groups 应采用什么默认策略？ | 默认不做语义择优；后续 dataset-level gate 推荐 full-group quarantine 优先于 semantic selection，除非先定义并评审领域规则。 | A. 直接按 source_run_id precedence 选主；B. 人工抽样后制定 semantic rule；C. 全部保留不写。 | 推荐方案避免污染历史事实；A 会把 metadata precedence 错用于业务冲突；B 更准确但成本高；C 风险低但不减重复。 | business-conflict groups 共 4,272,624，错误选择会污染价格、涨跌停、事件、交易状态历史事实。 | 若用户提供领域规则和抽样验收标准，可为单 dataset 启动 semantic selection gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`，接受 DQ-CP2-CR146-HIST-01..03 的推荐方案：不立即执行全量 cleanup，只准备后续 `adj_factor` dataset-level 门禁。 |
| 备选方案 | 修改任一 DQ；要求直接起 5 个 dataset 门禁；或要求暂停 FU-CR139-001。 |
| 影响维度 | 历史根数据质量、全根扫描工具、存储成本、业务冲突语义风险、回滚复杂度。 |
| 优劣分析 | 推荐方案保留当前生产可用 current truth，同时不把 business-conflict 语义选择和删除/archive 放进自动执行。 |
| 风险与回退 | 风险等级 high；本门禁不执行写入，因此回退是状态回写和门禁关闭；任何真实 cleanup 都必须另起 dataset-level gate。 |
| 用户需决策事项 | DQ-CP2-CR146-HIST-01、DQ-CP2-CR146-HIST-02、DQ-CP2-CR146-HIST-03。 |
| 不授权项 | historical parquet rewrite、business-conflict semantic selection、quarantine 写入、catalog pointer mutation、legacy delete/archive、NAS/provider/credential/trading/runtime/Git remote write。 |
| 自动终验授权 | `auto_final_authorization: false`；本门禁不允许 agent 在无用户回复时自动 approve。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR tracking valid | 待审查 | `meta-flow check cr-tracking` | CR-146 only active formal CR. |
| N1 completed | 待审查 | `process/checks/CP7-CR146-PRODUCTION-LAKE-N1-MIGRATION.result.json` | N1 current layout migration passed. |
| Duplicate split plan exists | 待审查 | `process/evidence/CR146-PRODUCTION-LAKE-HISTORICAL-DUPLICATE-CLEANUP-PLAN-POST-N1.index.json` | Plan-only; no lake write. |
| Current truth is clean | 待审查 | `process/evidence/CR146-PRODUCTION-LAKE-CURRENT-TRUTH-PROFILE-FULL-POST-N1-2026-07-01.json` | duplicate_key_total=0. |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | Historical cleanup is separated from N1 | 待审查 | Decision Brief | N1 complete; cleanup is separate high-risk gate. |
| 2 | Business-conflict groups are not auto-selected | 待审查 | DQ-CP2-CR146-HIST-03 | Default no semantic selection. |
| 3 | Delete/archive remains not authorized | 待审查 | 不授权项 | No legacy delete/archive in this gate. |
| 4 | Dataset-level gate path is explicit | 待审查 | DQ-CP2-CR146-HIST-02 | Only prepares `adj_factor` gate; no execution. |
| 5 | Process evidence is sufficient | 待审查 | CP2 result + context capsule | Context capsule ready; result JSON exists. |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| User decision recorded | 待审查 | 用户回复 / 本文件人工审查结果 | Awaiting user response. |
| Cleanup execution boundary decided | 待审查 | DQ-CP2-CR146-HIST-01 | Recommended no immediate execution. |
| No unauthorized execution implied | 待审查 | 不授权项 / auto_final_authorization | approve does not authorize cleanup writes/deletes. |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 context capsule | `process/context/CP2-CR146-HISTORICAL-DUPLICATE-CLEANUP-GATE-CONTEXT.yaml` | 待审查 | compact context. |
| CP2 result JSON | `process/checks/CP2-CR146-HISTORICAL-DUPLICATE-CLEANUP-GATE.result.json` | 待审查 | machine-readable precheck. |
| Post-N1 duplicate cleanup plan | `process/evidence/CR146-PRODUCTION-LAKE-HISTORICAL-DUPLICATE-CLEANUP-PLAN-POST-N1.index.json` | 待审查 | read-only split planning. |
| Current truth profile | `process/evidence/CR146-PRODUCTION-LAKE-CURRENT-TRUTH-PROFILE-FULL-POST-N1-2026-07-01.json` | 待审查 | clean current truth. |

## 人工审查结果

- 结论：`approved | changes_requested | rejected`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：

回复 `approve` 表示接受推荐方案：不立即执行全量 historical-root cleanup，不授权任何 rewrite/delete/archive/business-conflict semantic selection；只允许继续准备 `adj_factor` dataset-level cleanup 门禁草案。
