# CR059 Backup Point Plan

## Purpose

本计划定义真实迁移前必须准备的备份点。CR059 已创建备份目标目录，并完成一次 cold pre-migration backup copy；该 copy 是备份，不是迁移式路径切换。

## Backup Targets

| Source class | Planned target | Current status |
|---|---|---|
| repo/data working tree snapshot | `/mnt/quant-lab-cold/backups/pre-migration/cr059/repo-data-snapshot-20260614` | copied, about 4.4G |
| git bundle | `/mnt/quant-lab-cold/git-bundles/pre-cr059/local_backtest-pre-cr059-e7a3a0d.bundle` | created and verified, about 7.8M |
| future data lake backup | `/mnt/quant-lab-cold/backups/data-lake` | directory ready; full lake backup remains future execution gate |
| future research backup | `/mnt/quant-lab-cold/backups/research` | directory ready; CR059 snapshot already includes current `reports` |
| future process backup | `/mnt/quant-lab-cold/backups/process` | directory ready; CR059 snapshot already includes current `process` |

## Required Before Real Migration

| Gate | Requirement | Evidence |
|---|---|---|
| repo-local migration | Git bundle or equivalent rollback ref | CR059 bundle exists; refresh if HEAD changes |
| NAS archive migration | source/target manifest and backup copy evidence | CR059 snapshot exists; refresh if source changes |
| data lake migration | backup + restore drill or waived decision | `restore-drills/cr063` |
| Git remote cutover | remote identity gate and push authorization | future CR |

## Current Decision

- `execute_allowed=false`
- `backup_copy_status=completed-for-cr059`
- `delete_allowed=false`
- `restore_drill_required_before_data_lake_cutover=true`
