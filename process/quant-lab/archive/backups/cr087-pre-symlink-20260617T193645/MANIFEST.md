# CR087 Pre-Symlink Backup — Archive Manifest

- **source_path**: `process/backups/cr087-pre-symlink-20260617T193645/`
- **target_path**: `process/archive/backups/cr087-pre-symlink-20260617T193645/`
- **moved_at**: 2026-07-04
- **reason**: CR087 symlink 迁移前备份，CR087 已关闭，不再需要作为活跃恢复源
- **source_cr**: CR-087
- **current_discovery**: excluded
- **read_policy**: deny-default (process/archive/**)
- **restore_condition**: 仅在 CR087 migration 历史审计或争议时需要
- **file_count**: 326
- **size**: ~2.4MB
- **content**: checkpoints, docs-design, docs-features (qmt-gateway, factor-research, market-data-lake, strategy-lifecycle 等), docs-quality, docs-release
