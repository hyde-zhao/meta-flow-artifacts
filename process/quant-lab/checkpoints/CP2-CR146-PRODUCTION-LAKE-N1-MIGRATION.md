---
checkpoint_id: "CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION"
checkpoint_name: "CR146 Production Lake N1 Physical Migration Gate"
type: "auto_then_manual"
status: "pending_user_decision"
owner: "host-orchestrator"
created_at: "2026-07-01T08:45:00+08:00"
source_cr: "CR-146"
parent_cr: "CR-139"
auto_check_result: "process/checks/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION.result.json"
context_ref: "process/context/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION-CONTEXT.yaml"
decision_brief_required: true
auto_final_authorization: false
---

# CP2 CR146 Production Lake N1 Physical Migration Gate

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION.result.json` | PASS | 0 | N1 copy-first migration gate machine precheck passed. |
| `process/evidence/CR146-PRODUCTION-LAKE-PHYSICAL-MIGRATION-PLAN-2026-07-01.json` | PASS | 0 | 17/17 published datasets require copy-first current layout migration; 0 source missing. |
| `process/evidence/CR146-PRODUCTION-LAKE-PHYSICAL-MIGRATION-DRY-RUN-2026-07-01.json` | PASS | 0 | Dry-run lists 17 copy candidates and executes 0 lake/catalog/pointer/NAS/provider operations. |
| `process/evidence/CR146-PRODUCTION-LAKE-CURRENT-TRUTH-PROFILE-2026-07-01.json` | PASS | 0 | 17 published current datasets, 103264532 rows, duplicate_key_total=0. |
| `process/evidence/CR146-PRODUCTION-LAKE-READINESS-MATRIX-2026-07-01.json` | PASS_WITH_HUMAN_GATES_REQUIRED | 0 | 17/17 research_ready; N1 physical migration is the current structural blocker. |
| `uv run --python 3.11 pytest tests/data_lake -q` | PASS | 0 | 342 passed; includes dedup fail-closed contracts and reader fail-fast contracts. |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认是否授权 CR-146 执行生产数据湖 N1 current-truth 物理分区迁移，把 17 个 catalog current parquet 对象 copy-first 到 `canonical/<dataset>/1.0/current/` 并更新 catalog canonical_path。 |
| 推荐动作 | `approve` 默认方案：执行 N1 copy-first 迁移 + catalog canonical_path 更新；不删除 legacy `run_id=` 路径；不执行 NAS sync；不执行历史冲突 cleanup/rewrite。 |
| approve 后会发生什么 | 我会按同一授权分两段执行稳定入口：先对 `bse_code_mapping` 运行 `scripts/data_lake/execute_physical_partition_migration.py --datasets bse_code_mapping --execute-copy --update-catalog --approval-id CP2-CR146-N1-APPROVED-20260701 --created-at <审批时间>`，完成 copy、size/sha256/parquet-readable 校验、catalog switch、current/ reader smoke 与 golden 归因对比；通过后再对剩余 16 个 dataset 执行同样流程。全部完成后重跑 current-truth profile、inventory、golden baseline、reader smoke 和相关 pytest。 |
| approve 不授权什么 | `approve` 不授权 legacy 删除/archive、NAS sync/write/restore drill、provider fetch、credential read、business-conflict semantic selection、historical-root cleanup/rewrite、trading/runtime/Git remote write。 |
| 不确认会阻塞什么 | 不确认会阻塞 N1 物理分区迁移和 catalog current layout 切换；不阻塞现有 catalog current truth 的只读使用。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在审计冲突、字段不足、人工深度评审或用户要求时读取完整 evidence。 |
| 全文档读取扩展 | 已读取 CR146 scope、N1 plan、current-truth profile、readiness matrix 和 dry-run result；未读取 `.env` 或 NAS 凭据。 |
| 缺失 / waived 理由 | N/A；本门禁已有 context capsule、机器 result、readiness matrix 和 dry-run evidence。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CR146 active CR | `process/changes/CR-146-CR139-POST-CLOSE-LAKE-COMPLETION-2026-06-30.md` | scanned | 3 | 3 | N1 physical migration、legacy cleanup、business-conflict policy 均为高风险项；本门禁只纳入 N1 current-truth migration。 |
| N1 physical plan | `process/evidence/CR146-PRODUCTION-LAKE-PHYSICAL-MIGRATION-PLAN-2026-07-01.json` | scanned | 17 | 1 | 17 个 dataset 共享同一 copy-first + catalog update 决策，合并为 DQ-CP2-CR146-N1-01。 |
| N1 dry-run | `process/evidence/CR146-PRODUCTION-LAKE-PHYSICAL-MIGRATION-DRY-RUN-2026-07-01.json` | scanned | 1 | 1 | Dry-run 通过，但真实 copy/catalog update 需 DQ-CP2-CR146-N1-01 授权。 |
| current truth profile | `process/evidence/CR146-PRODUCTION-LAKE-CURRENT-TRUTH-PROFILE-2026-07-01.json` | scanned | 1 | 0 | current truth duplicate_key_total=0，不触发 business-conflict 当前读层门禁。 |
| historical duplicate split plan | `process/evidence/CR146-PRODUCTION-LAKE-DUPLICATE-SPLIT-PLAN.index.json` | scanned | 5 | 1 | 历史根 cleanup/rewrite 风险纳入 DQ-CP2-CR146-N1-04 的不授权边界，后续另起门禁。 |
| backup assumption | 用户说明 NAS 已有备份，当前 lake 从备份路径迁移而来 | scanned | 1 | 1 | 是否再做完整二次备份纳入 DQ-CP2-CR146-N1-02。 |
| STATE pending queue | `process/state/STATE.current.json` | scanned | 1 | 1 | active_change=CR-146；旧禁止项由本门禁显式覆盖 N1 current-truth migration execution only。 |
| script entrypoint contract | `scripts/data_lake/*` / `docs/components/SCRIPT-ENTRYPOINTS.md` | scanned | 1 | 0 | 本门禁执行入口是 stable script；本轮新增/使用入口无 legacy import。 |
| data-lake regression | `tests/data_lake/` | scanned | 2 | 0 | 旧 deterministic fallback 测试已改为 fail-closed 契约；`catalog_run_id_missing_for_duplicate_dedup` 与 `catalog_run_id_not_found_for_duplicate_dedup` 均覆盖。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 2 | DQ-CP2-CR146-N1-01、DQ-CP2-CR146-N1-02。 |
| 高风险策略确认 | 2 | DQ-CP2-CR146-N1-03、DQ-CP2-CR146-N1-04。 |
| agent 默认处理 | 1 | approve 后按 canary `bse_code_mapping` + remaining 16 分批执行 copy-first、catalog update、post validation；失败则停止并生成 rollback evidence。 |
| 仅审计记录 | 1 | stable script no-legacy contract and operation count boundary. |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR146-N1-01 | runtime_authorization | 是否授权执行 N1 current-truth copy-first physical migration 和 catalog canonical_path 更新？ | 授权执行 17 个 catalog current parquet copy-first 到 `current/`，但按 `bse_code_mapping` canary + remaining 16 分批执行；每个 target 做 size/sha256/parquet-readable 校验，全部通过后更新对应 catalog canonical_path。 | A. 只 copy 不更新 catalog；B. 继续只保留 legacy run_id 布局。 | 推荐方案完成 N1 结构目标且保留 legacy 回滚，canary 先行降低一次性切换半径；A 产生双布局但 current truth 不切换；B 保持现状不能完成生产级物理布局。 | 会写 17 个 current parquet 对象并更新 catalog；若任一 copy 校验失败，catalog 不切换。 | Canary 或 remaining 阶段任一 copy/post-copy verify/current-path smoke/golden attribution 失败时停止，保留或回滚 legacy 指针；若 catalog update 后 post-validation 失败，按 rollback_pointer 恢复 catalog canonical_path。 |
| DQ-CP2-CR146-N1-02 | risk_acceptance | N1 执行前是否需要再做完整二次备份？ | 不做完整二次备份；复用用户确认的 NAS 既有备份，并以 current catalog/manifest/inventory/golden/current-truth profile 作为本次回滚包。 | A. 先做全量本地二次备份；B. 先做 NAS restore drill。 | 推荐方案避免制造第三份全量数据状态，且 copy-first 不删除 legacy；A 成本高；B 需要 NAS/credential scope。 | 若 NAS 备份事实不成立，灾备能力下降；但本次不删除 legacy，主要回滚是 catalog pointer。 | 若用户要求 restore drill 或发现 legacy path 不完整，则暂停 N1 执行。 |
| DQ-CP2-CR146-N1-03 | risk_acceptance | N1 后是否立即删除/archive legacy `run_id=` 路径？ | 不删除、不 archive，legacy `run_id=` 路径保留为回滚源。 | A. archive movement；B. delete cleanup。 | 推荐方案风险最低；archive/delete 可释放空间但会把回滚从 pointer revert 变成 restore。 | 不释放历史空间；但避免不可逆数据损失。 | N1 稳定验证通过后，另起 cleanup gate。 |
| DQ-CP2-CR146-N1-04 | risk_acceptance | 是否在本门禁内处理 historical-root business conflicts / cleanup rewrite？ | 不处理；本门禁只迁移 catalog current truth。历史根 4,272,624 business-conflict groups 留给后续 dataset-level cleanup gate。 | A. 同时执行 historical cleanup；B. 按 dataset semantic rule 重写。 | 推荐方案边界清晰，current truth duplicate_key_total=0；A/B 会扩大到 189,950,712 行历史重写风险。 | 历史根仍有重复/冲突，影响全根扫描类工具；不会影响 catalog current truth reader。 | 若 N1 后仍需清理存储或历史扫描面，启动 dataset-level cleanup/rewrite gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`，接受 DQ-CP2-CR146-N1-01..04 的推荐方案。 |
| 备选方案 | 修改任一 DQ；只授权 dry-run；只授权 copy 不授权 catalog update；或要求先 NAS restore drill。 |
| 影响维度 | 数据湖物理布局、catalog current truth、回滚复杂度、历史 cleanup 边界、NAS/备份策略。 |
| 优劣分析 | 推荐方案完成 N1 current layout 且保留 legacy 回滚；不把 cleanup/NAS/语义冲突选择混入同一执行窗口。 |
| 风险与回退 | 风险等级 high；copy-first + legacy retained + catalog rollback map 是主要回退手段；执行策略为 canary first，先验证 `current/` 读链路和 golden 归因后再扩大到剩余 16 个 dataset。 |
| 用户需决策事项 | DQ-CP2-CR146-N1-01、DQ-CP2-CR146-N1-02、DQ-CP2-CR146-N1-03、DQ-CP2-CR146-N1-04。 |
| 不授权项 | legacy delete/archive、NAS sync/write/restore drill、provider fetch、credential read、business-conflict semantic selection、historical cleanup/rewrite、trading/runtime/Git remote write。 |
| 自动终验授权 | `auto_final_authorization: false`；本门禁不允许 agent 在无用户回复时自动 approve。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| Process route health checked | 待审查 | `meta-flow workspace check` | process symlink healthy; artifact repo dirty but writable. |
| CR tracking valid | 待审查 | `meta-flow check cr-tracking` | CR-146 only active formal CR. |
| N1 plan exists | 待审查 | `process/evidence/CR146-PRODUCTION-LAKE-PHYSICAL-MIGRATION-PLAN-2026-07-01.json` | 17/17 copy-first candidates; 0 source missing. |
| Dry-run exists | 待审查 | `process/evidence/CR146-PRODUCTION-LAKE-PHYSICAL-MIGRATION-DRY-RUN-2026-07-01.json` | PASS; zero side effects. |
| Current truth profile exists | 待审查 | `process/evidence/CR146-PRODUCTION-LAKE-CURRENT-TRUTH-PROFILE-2026-07-01.json` | duplicate_key_total=0. |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | Human gate separates high-risk copy/catalog updates | 待审查 | Decision table | No execution without user approve. |
| 2 | Stable script entrypoint used | 待审查 | `scripts/data_lake/execute_physical_partition_migration.py` | No legacy import in the execution path. |
| 3 | Rollback path is explicit | 待审查 | N1 plan rollback_pointer | Legacy run_id paths retained; catalog canonical_path can be reverted. |
| 4 | Post-copy verification is mandatory before catalog switch | 待审查 | `scripts/data_lake/execute_physical_partition_migration.py` | size/sha256/parquet-readable verification blocks catalog switch on mismatch. |
| 5 | Canary execution strategy explicit | 待审查 | `scripts/data_lake/execute_physical_partition_migration.py --datasets` / DQ-CP2-CR146-N1-01 | `bse_code_mapping` runs first, validates current/ reader path and golden attribution before remaining 16. |
| 6 | Full data-lake regression covers reader contract changes | 待审查 | `uv run --python 3.11 pytest tests/data_lake -q` | 342 passed; includes `test_dedup_contracts.py` and `test_reader_fail_fast_contracts.py`. |
| 7 | Non-authorized actions explicit | 待审查 | 审批者摘要 / 不授权项 | NAS, delete/archive, provider, credential, trading/runtime, historical cleanup not included. |
| 8 | Backup policy explicit | 待审查 | DQ-CP2-CR146-N1-02 | User stated NAS backup exists and current lake was migrated from backup path; no full second backup or NAS restore drill in this gate. |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| User decision recorded | 待审查 | 用户回复 / 本文件人工审查结果 | Awaiting user response. |
| N1 execution authorization decided | 待审查 | DQ-CP2-CR146-N1-01 | approve required for real copy/catalog update. |
| No unauthorized execution implied | 待审查 | 不授权项 / auto_final_authorization | approve does not authorize cleanup/NAS/provider/credential/trading/runtime. |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 context capsule | `process/context/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION-CONTEXT.yaml` | 待审查 | compact context. |
| CP2 result JSON | `process/checks/CP2-CR146-PRODUCTION-LAKE-N1-MIGRATION.result.json` | 待审查 | machine-readable precheck. |
| N1 migration plan | `process/evidence/CR146-PRODUCTION-LAKE-PHYSICAL-MIGRATION-PLAN-2026-07-01.json` | 待审查 | 17 source/target/rollback entries. |
| N1 dry-run | `process/evidence/CR146-PRODUCTION-LAKE-PHYSICAL-MIGRATION-DRY-RUN-2026-07-01.json` | 待审查 | zero side effects. |
| Readiness matrix | `process/evidence/CR146-PRODUCTION-LAKE-READINESS-MATRIX-2026-07-01.json` | 待审查 | 17 research_ready; N1 pending. |
| Follow-up tracking sync | `process/changes/CR-139-FOLLOW-UP-TRACKING-2026-06-30.md#FU-CR139-002` | 待审查 | FU-CR139-002 is active/ready_with_risk with related_active_cr=CR-146. |
| Full data-lake regression | `tests/data_lake/` | 待审查 | 342 passed; dedup fallback conflict resolved in favor of fail-closed. |

## 人工审查结果

- 结论：`approved | changes_requested | rejected`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：

回复 `approve` 表示接受推荐方案：按 canary `bse_code_mapping` + remaining 16 执行 N1 copy-first migration + post-copy verify + catalog canonical_path update；不授权 legacy delete/archive、NAS sync/write/restore drill、provider fetch、credential read、business-conflict semantic selection、historical cleanup/rewrite、trading/runtime/Git remote write。
