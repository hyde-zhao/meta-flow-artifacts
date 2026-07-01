# CP2 CR146 Production Lake N1 Migration Launch Message

请审查 `process/checkpoints/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION.md`。

## 审批者摘要

本次确认服务的整体目标：确认是否授权 CR-146 执行生产数据湖 N1 current-truth 物理分区迁移，把 17 个 catalog current parquet 对象 copy-first 到 `canonical/<dataset>/1.0/current/` 并更新 catalog canonical_path。

推荐动作：回复 `approve` 接受推荐方案，授权执行 N1 copy-first 迁移 + catalog canonical_path 更新。

approve 后会发生什么：我会按同一授权分两段执行稳定入口：先对 `bse_code_mapping` 运行 `scripts/data_lake/execute_physical_partition_migration.py --datasets bse_code_mapping --execute-copy --update-catalog --approval-id CP2-CR146-N1-APPROVED-20260701 --created-at <审批时间>`，完成 copy、size/sha256/parquet-readable 校验、catalog switch、current/ reader smoke 与 golden 归因对比；通过后再对剩余 16 个 dataset 执行同样流程。全部完成后重跑 current-truth profile、inventory、golden baseline、reader smoke 和相关 pytest。

approve 不授权什么：不授权 legacy 删除/archive、NAS sync/write/restore drill、provider fetch、credential read、business-conflict semantic selection、historical-root cleanup/rewrite、trading/runtime/Git remote write。

不确认会阻塞什么：阻塞 N1 物理分区迁移和 catalog current layout 切换；不阻塞现有 catalog current truth 的只读使用。

如果你回复 approve，表示接受本轮 4 项推荐方案；不表示授权 legacy 删除/archive、NAS sync/write/restore drill、provider fetch、credential read、business-conflict semantic selection、historical-root cleanup/rewrite、trading/runtime/Git remote write。

不授权项：legacy delete/archive、NAS sync/write/restore drill、provider fetch、credential read、business-conflict semantic selection、historical-root cleanup/rewrite、trading/runtime/Git remote write。

## 自动预检结论

- 结论：PASS
- 阻断项：0
- 证据：`process/checks/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION.result.json`
- N1 计划：17/17 copy-first candidates，0 source missing。
- dry-run：PASS，operation counts 全为 0。
- current truth：17 published datasets，103264532 rows，duplicate_key_total=0。
- full data-lake regression：`uv run --python 3.11 pytest tests/data_lake -q` PASS，342 passed；覆盖 dedup fail-closed 与 reader fail-fast 合同。

## Context Capsule

- Context Capsule：`process/context/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION-CONTEXT.yaml`
- read_profile：compact
- must_read：N1 plan、N1 dry-run、current truth profile、readiness matrix。
- do_not_read_by_default：`.env`、NAS credentials、`scripts/legacy/cr/*`。

## 决策收集覆盖

- 已扫描来源：CR146 active CR、N1 physical plan、N1 dry-run、current truth profile、historical duplicate split plan、用户 NAS 备份说明、STATE pending queue、script entrypoint contract。
- 候选问题数：10
- 本轮待人工决策项：4
- N/A 原因：current truth duplicate_key_total=0，因此 business-conflict 当前读层门禁 N/A；historical cleanup/rewrite 后续另起 gate。
- data-lake regression：旧 deterministic fallback 测试已改为 fail-closed 契约；`catalog_run_id_missing_for_duplicate_dedup` 与 `catalog_run_id_not_found_for_duplicate_dedup` 均覆盖。

## 决策分层

- 必须用户决策：2
- 高风险策略确认：2
- agent 默认处理：1
- 仅审计记录：1

## 本轮待人工决策项

本轮待人工决策项：4

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR146-N1-01 | runtime_authorization | 是否授权执行 N1 current-truth copy-first physical migration 和 catalog canonical_path 更新？ | 授权执行 17 个 catalog current parquet copy-first 到 `current/`，但按 `bse_code_mapping` canary + remaining 16 分批执行；每个 target 做 size/sha256/parquet-readable 校验，全部通过后更新对应 catalog canonical_path。 | A. 只 copy 不更新 catalog；B. 继续只保留 legacy run_id 布局。 | 推荐方案完成 N1 结构目标且保留 legacy 回滚，canary 先行降低一次性切换半径；A 产生双布局但 current truth 不切换；B 保持现状不能完成生产级物理布局。 | 会写 17 个 current parquet 对象并更新 catalog；若任一 copy 校验失败，catalog 不切换。 | Canary 或 remaining 阶段任一 copy/post-copy verify/current-path smoke/golden attribution 失败时停止，保留或回滚 legacy 指针；若 catalog update 后 post-validation 失败，按 rollback_pointer 恢复 catalog canonical_path。 |
| DQ-CP2-CR146-N1-02 | risk_acceptance | N1 执行前是否需要再做完整二次备份？ | 不做完整二次备份；复用用户确认的 NAS 既有备份，并以 current catalog/manifest/inventory/golden/current-truth profile 作为本次回滚包。 | A. 先做全量本地二次备份；B. 先做 NAS restore drill。 | 推荐方案避免制造第三份全量数据状态，且 copy-first 不删除 legacy；A 成本高；B 需要 NAS/credential scope。 | 若 NAS 备份事实不成立，灾备能力下降；但本次不删除 legacy，主要回滚是 catalog pointer。 | 若用户要求 restore drill 或发现 legacy path 不完整，则暂停 N1 执行。 |
| DQ-CP2-CR146-N1-03 | risk_acceptance | N1 后是否立即删除/archive legacy `run_id=` 路径？ | 不删除、不 archive，legacy `run_id=` 路径保留为回滚源。 | A. archive movement；B. delete cleanup。 | 推荐方案风险最低；archive/delete 可释放空间但会把回滚从 pointer revert 变成 restore。 | 不释放历史空间；但避免不可逆数据损失。 | N1 稳定验证通过后，另起 cleanup gate。 |
| DQ-CP2-CR146-N1-04 | risk_acceptance | 是否在本门禁内处理 historical-root business conflicts / cleanup rewrite？ | 不处理；本门禁只迁移 catalog current truth。历史根 4,272,624 business-conflict groups 留给后续 dataset-level cleanup gate。 | A. 同时执行 historical cleanup；B. 按 dataset semantic rule 重写。 | 推荐方案边界清晰，current truth duplicate_key_total=0；A/B 会扩大到 189,950,712 行历史重写风险。 | 历史根仍有重复/冲突，影响全根扫描类工具；不会影响 catalog current truth reader。 | 若 N1 后仍需清理存储或历史扫描面，启动 dataset-level cleanup/rewrite gate。 |

## 回复格式

- approve：接受 4 项推荐方案，授权按 canary `bse_code_mapping` + remaining 16 执行 N1 current-truth copy-first migration + post-copy verify + catalog canonical_path 更新。
- 修改: <具体修改点>：例如“只 copy 不更新 catalog”或“先做 NAS restore drill”。
- reject：不授权本轮 N1 执行。
