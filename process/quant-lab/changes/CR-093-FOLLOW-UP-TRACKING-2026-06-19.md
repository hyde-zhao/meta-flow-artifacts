---
source_cr: "CR-093"
status: "open"
created_at: "2026-06-19T08:52:56+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-19T09:39:15+08:00"
checkpoint_source: "CP8"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR093 后续事项跟踪台账

## 目的

本台账记录 CR093 CP8 后续候选及 FU-01 strict-warnings readiness 的历史 `cp8-follow-up` 账本补行。它只覆盖静态账本和 checker 语义，不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env` / 凭据 / 账户 / 资金 / 持仓 / 委托 / 成交 / 原始日志、submit / cancel / buy / sell、simulation / live、provider fetch、lake write、catalog publish 或真实 release execution / publish。

## 后续 CR / 历史补行索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR-053 | Quant-lab migration inventory and dry-run | closed | CR | 0 | process_ledger; cp8-follow-up-backfill | `process/changes/CR-053-QUANT-LAB-MIGRATION-INVENTORY-AND-DRY-RUN-2026-06-14.md` | closed-current-delivery | closed | 历史 `source=cp8-follow-up` 正式 CR 缺 tracking row，CR094 补行用于 strict-warnings readiness | 已关闭，仅补机器可读索引 | CR093-FU-01 |
| CR-058 | Repo-local mechanical migration relayout gate | closed | CR | 0 | process_ledger; cp8-follow-up-backfill | `process/changes/CR-058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE-2026-06-14.md` | closed-current-delivery | closed | 历史 `source=cp8-follow-up` 正式 CR 缺 tracking row，CR094 补行用于 strict-warnings readiness | 已关闭，仅补机器可读索引 | CR093-FU-01 |
| CR-074 | Root cutover / daily workflow switch | closed | CR | 0 | root_cutover; cp8-follow-up-backfill | `process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md` | closed-current-delivery | closed | 历史 `source=cp8-follow-up` 正式 CR 缺 tracking row，CR094 补行用于 strict-warnings readiness | 已关闭，仅补机器可读索引 | CR093-FU-01 |
| CR-075 | Target env rebuild and smoke | closed | CR | 0 | target_env_rebuild; cp8-follow-up-backfill | `process/changes/CR-075-TARGET-ENV-REBUILD-AND-SMOKE-2026-06-16.md` | closed-current-delivery | closed | 历史 `source=cp8-follow-up` 正式 CR 缺 tracking row，CR094 补行用于 strict-warnings readiness | 已关闭，仅补机器可读索引 | CR093-FU-01 |
| CR-078 | Remote Git governance publication gate | closed | CR | 0 | remote_git_governance; cp8-follow-up-backfill | `process/changes/CR-078-REMOTE-GIT-GOVERNANCE-PUBLICATION-GATE-2026-06-17.md` | closed-current-delivery | closed | 历史 `source=cp8-follow-up` 正式 CR 缺 tracking row，CR094 补行用于 strict-warnings readiness | 已关闭，仅补机器可读索引 | CR093-FU-01 |
| CR-080 | Data/reports restore or copy gate | closed | CR | 0 | reports_restore; cp8-follow-up-backfill | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | closed-current-delivery | closed | 历史 `source=cp8-follow-up` 正式 CR 缺 tracking row，CR094 补行用于 strict-warnings readiness | 已关闭，仅补机器可读索引 | CR093-FU-01 |
| CR-092 | Real QMT readonly runtime smoke design gate | closed | CR | 0 | readonly_runtime_smoke_design; cp8-follow-up-backfill | `process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md` | closed-current-delivery | closed | 历史 `source=cp8-follow-up` 正式 CR 缺 tracking row，CR094 补行用于 strict-warnings readiness | 已关闭，仅补机器可读索引 | CR093-FU-01 |
| CR-093 | Ledger hygiene for CR019 / CR025 tracking cleanup | closed | CR | 0 | cr_tracking_consistency; cp8-follow-up-backfill | `process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md` | closed-current-delivery | closed | CR093 自身为 `source=cp8-follow-up`，需要在 CR093 follow-up 台账里建立关闭行 | 已关闭，仅补机器可读索引 | CR093-FU-01 |
| CR-094 | Warning cleanup / strict-warnings readiness | closed | CR | 1 | cr_tracking_consistency; strict_warnings; cp8-follow-up-backfill | `process/changes/CR-094-WARNING-CLEANUP-STRICT-WARNINGS-READINESS-2026-06-19.md` | closed-current-delivery | closed | 用户已同意 CP8 READY；CR094 当前 strict-warnings readiness 交付关闭 | 当前交付已关闭；CR093-FU-02 仍需用户另选候选后新建 CR | DQ-CP8-CR093-02 / DQ-CP8-CR093-04 |
| CR-095 | Standalone checker 与主 CLI 输出收敛 | closed | CR | 2 | checker_output_convergence | `process/changes/CR-095-STANDALONE-CHECKER-CLI-OUTPUT-CONVERGENCE-2026-06-19.md` | closed-current-delivery | closed | 用户已同意 CP8 READY；CR095 当前交付关闭 | 已关闭；stdout 兼容性问题如出现再新建后续 CR | DQ-CP8-CR094-03 |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 | 来源 |
|---|---|---|---|---|
| NA-CR093-FU01-01 | QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env`、凭据、账户、资金、持仓、委托、成交、原始日志、submit / cancel、simulation / live、provider fetch、lake write、catalog publish、真实 release execution / publish | CR094 只做静态账本和 checker readiness | 必须新建独立 CR 并经人工门禁授权 | DQ-CP8-CR093-05 |

## 验收标准

| AC ID | 内容 | 验证方式 |
|---|---|---|
| AC-CR094-01 | `meta-flow check cr-tracking --project-root . --strict-warnings` exit 0 | 本地命令 |
| AC-CR094-02 | 常规 `meta-flow check cr-tracking --project-root .` 仍 exit 0 且 active formal CR 可解释 | 本地命令 |
| AC-CR094-03 | 历史 `source=cp8-follow-up` 正式 CR 的 tracking row 可机器读取 | 本台账 + checker |
| AC-CR094-04 | 历史嵌套 `active_change` 不再导致 strict warning 失败 | 单元 / CLI 验证 |
