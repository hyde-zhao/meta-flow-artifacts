---
source_cr: "CR-120"
status: "open-candidate"
created_at: "2026-06-23T08:58:36+08:00"
created_by: "host-orchestrator"
checkpoint_source: "CP8"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR120 后续事项跟踪台账

## 目的

本台账记录用户在 CR120 CP8 approve 同时提出的目录路由 / merge 回归检测问题。CR120 row convention hardening 已关闭为 READY；本台账不表示已授权目录整改，只记录后续候选。

## 后续 CR / Spike 候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR-121 | Merge-Induced Directory Routing Regression Hardening | superseded | requirement-change | 1 | docs_design_symlink; docs_features_symlink; docs_quality_symlink; process_symlink; checkpoints_symlink; tracked_ignored_paths; merge_regression; docs_qmt; docs_goldminer; claude_artifacts | process/changes/CR-121-MERGE-INDUCED-DIRECTORY-ROUTING-REGRESSION-HARDENING-2026-06-23.md | source_candidate=FU-CR120-001; cp2_approved; cp5_approved; cp6_not_executed; superseded_by_cr125 | closed | CR125 CP5 已批准并接管剩余目录治理 stop-line；CR121 index-only remediation 不再恢复 | 不再恢复 CR121 CP6；如未来需要 Git index cleanup，另起独立 CR | USER-20260623-START-FU-CR120-001 |
| FU-CR120-001 | Merge-Induced Directory Routing Regression Hardening | closed | ledger-maintenance | 1 | docs_design_symlink; docs_features_symlink; docs_quality_symlink; process_symlink; checkpoints_symlink; tracked_ignored_paths; merge_regression; docs_qmt; docs_goldminer; claude_artifacts | process/changes/CR-121-MERGE-INDUCED-DIRECTORY-ROUTING-REGRESSION-HARDENING-2026-06-23.md | related_superseded_cr=CR-121; cp2_approved; cp5_approved; cp6_not_executed; superseded_by_cr125 | closed | CR125 CP5 已批准并接管剩余目录治理 stop-line；CR121 index-only remediation 不再恢复 | 不再恢复 CR121 CP6；如未来需要 Git index cleanup，另起独立 CR | USER-20260623-CP8-CR120-APPROVE-DIRECTORY-AUDIT |

## 当前审计结论

| 项目 | 结论 | 证据 |
|---|---|---|
| `docs/design` / `docs/features` 是否被 merge 还原成普通目录 | 否 | 当前为 symlink，且相对 merge 第一父提交无变化。 |
| 是否存在目录结论混乱 | 是 | `git ls-files -i -c --exclude-standard` 显示多个 ignore 规则覆盖路径仍被 Git 跟踪。 |
| 是否有其他目录因 merge 带回旧内容 | 是 | `.claude/**`、`docs/qmt/**`、`docs/goldminer/**`、部分 `docs/release/*.md` 从 master 侧进入当前树。 |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 |
|---|---|---|---|
| NA-FU-CR120-001 | `git rm --cached`、删除、移动、覆盖、checkout、reset、runtime、NAS、凭据、交易写、provider fetch、lake write、catalog publish | 当前只是候选和计划 | 必须启动正式 CR 并通过 CP5 实施授权 |
