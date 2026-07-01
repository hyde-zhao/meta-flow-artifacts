# CR149 NAS Current-Truth Sync Gate

CR-149 Phase 1 数据湖整改当前只剩 1 个阻断项：NAS/shared-node stale relative to local N1 current truth。

自动预检结论：PASS；见 `process/checks/CP2-CR149-NAS-CURRENT-TRUTH-SYNC.result.json`。阻断项 0；前置 CP7 结论为 BLOCKED，原因是 read-only NAS consistency 发现 catalog mismatch + 17 个 NAS current canonical paths missing。

Context Capsule Summary：见 checkpoint 的 `### Context Capsule Summary`，capsule 为 `process/context/CP2-CR149-NAS-CURRENT-TRUTH-SYNC-CONTEXT.yaml`，read_profile=`compact`，默认读取策略为 capsule-first，全文档读取只用于审计 NAS mismatch 和 sync 授权边界。

审批者摘要：
- 本次确认服务的整体目标：决定是否授权一次 scoped local-to-NAS current-truth sync，以修复 NAS/shared-node stale 状态并关闭 CR149 Phase 1 multi-node consistency exit。
- 推荐动作：approve 授权 scoped sync；先 dry-run，只允许 `catalog/catalog.json` 和 17 个 catalog current canonical parquet 对象，确认无 delete/restore/pull 后再 execute。
- approve 后会发生什么：我会运行 `scripts/data_lake/sync_nas_current_truth.py` dry-run，检查计划对象和 operation counters；通过后运行同脚本 `--execute --approval-id`，随后重跑 `scripts/data_lake/check_nas_multinode_consistency.py`；只有 18/18 comparisons match 才更新 Phase 1 exit 为 PASS。
- approve 不授权什么：不授权 delete/archive、NAS-to-local restore/pull、full lake cleanup/rewrite、provider fetch、local catalog pointer mutation、historical business-conflict cleanup、simulation/live/trading、broker write 或 Git remote write。
- 不确认会阻塞什么：会阻塞 CR-149 Phase 1 fully closed；不阻塞当前本地 production lake current truth 和 governed-lake readiness 使用。

决策收集覆盖：已扫描 5 个来源，发现候选问题 29 个，纳入待决策 3 个；N/A / 缺失来源 0 个，原因见 checkpoint 的 Decision Collection Coverage。
Decision Collection Coverage：见 checkpoint 的 `### Decision Collection Coverage`。

决策分层：
- 必须用户决策：2 项，DQ-CP2-CR149-NAS-SYNC-01、DQ-CP2-CR149-NAS-SYNC-02。
- 高风险策略确认：1 项，DQ-CP2-CR149-NAS-SYNC-03。
- agent 默认处理：2 项，approve 后先 dry-run 再 execute；post-sync mismatch fail-closed。
- 仅审计记录：1 项，credential access remains label-only in evidence and gate ledger。

本轮待人工决策项：3

待人工决策清单：

| 决策 ID | 决策类型 | 推荐方案 |
|---|---|---|
| DQ-CP2-CR149-NAS-SYNC-01 | runtime_authorization | 授权一次 scoped local-to-NAS current-truth sync，只同步 catalog 和 17 个 current canonical 对象。 |
| DQ-CP2-CR149-NAS-SYNC-02 | risk_acceptance | 采用本地 N1 current truth 作为 source of truth，NAS 作为 stale replica 补齐。 |
| DQ-CP2-CR149-NAS-SYNC-03 | risk_acceptance | post-sync 仍不一致时不自动继续修复，只记录 BLOCKED 并另起门禁。 |

请审查：`process/checkpoints/CP2-CR149-NAS-CURRENT-TRUTH-SYNC.md`。

如果你回复 approve：表示接受上述 3 个 DQ 的推荐方案，我将执行一次 scoped NAS current-truth sync；执行顺序为 dry-run first、execute only after scoped dry-run pass、post-sync consistency verification required。

approve

如果你回复 reject：我会保持 CR-149 为 local Phase 1 ready but NAS multi-node blocked，不执行 NAS sync/write。

reject

修改: <具体修改点>：如需修改，请指出要改的 DQ ID 和边界，例如“只 dry-run 不 execute”、“先人工核验对象清单”或“改用整湖 sync”。

修改: <具体修改点>

不表示授权：approve 不表示授权 delete/archive、NAS-to-local restore/pull、full lake cleanup/rewrite、provider fetch、local catalog pointer mutation、historical business-conflict cleanup、simulation/live/trading、broker write 或 Git remote write。

不授权项：NAS delete/archive、NAS-to-local restore or pull、full lake cleanup/rewrite、provider fetch、local catalog pointer mutation、historical business-conflict cleanup、simulation/live/trading、broker write、Git remote write。
