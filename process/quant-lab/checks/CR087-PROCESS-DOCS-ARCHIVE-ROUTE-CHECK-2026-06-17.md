# CR087 Process Docs Archive Route Check

## Scope

本检查只覆盖过程文档归档路由与本地软链接。未执行 data/reports 内容读取、凭据读取、NAS 内容操作、远端 Git 写、provider/lake/catalog、QMT/MiniQMT runtime 或 cleanup。

## Result

`PASS`

## Checks

| Check | Result | Evidence |
|---|---|---|
| `process` symlink | PASS | `/home/hyde/workspace/quant-lab/process -> ../process/quant-lab` |
| `process/STATE.md` readable | PASS | `test -f process/STATE.md` |
| route metadata readable | PASS | `test -f process/.meta-flow-process.yaml` |
| `docs/design` symlink | PASS | `docs/design -> ../../process/quant-lab/docs/design` |
| `docs/features` symlink | PASS | `docs/features -> ../../process/quant-lab/docs/features` |
| `docs/quality` symlink | PASS | `docs/quality -> ../../process/quant-lab/docs/quality` |
| `checkpoints` symlink | PASS | `checkpoints -> ../process/quant-lab/checkpoints` |
| no broken symlink | PASS | `find docs/design docs/features docs/quality docs/release checkpoints -xtype l` returned no rows |
| docs archive file count | PASS | `find -L docs/design docs/features docs/quality docs/release -maxdepth 4 -type f` returned `252` |
| checkpoint file count | PASS | `find -L checkpoints -maxdepth 1 -type f` returned `195` |
| public release files stay local | PASS | `docs/release/RELEASE-NOTES.md`, `DEPLOY-CHECKLIST.md`, `ROLLBACK.md`, `MIGRATION.md`, `FEEDBACK.md` remain tracked candidates |
| internal release archive links | PASS | CR-specific release files resolve through `/home/hyde/workspace/process/quant-lab/docs/release/` |
| bootstrap idempotency | PASS | `scripts/link-engineering-ledger.sh` completed successfully |
| Git ignore coverage | PASS | `/process`, `/checkpoints`, `/docs/design`, `/docs/features`, `/docs/quality`, and internal `docs/release` patterns match |

## Git Index Impact

`git rm --cached` was run only for historical process documents now owned by the external process namespace:

- `checkpoints/**`
- `docs/design/**`
- `docs/features/**`
- `docs/quality/**`
- internal `docs/release/*-CR*` and related process release artifacts

The working files were not deleted; they remain accessible via the new symlinks and external archive. Public release entry files remain in the project:

- `docs/release/RELEASE-NOTES.md`
- `docs/release/DEPLOY-CHECKLIST.md`
- `docs/release/ROLLBACK.md`
- `docs/release/MIGRATION.md`
- `docs/release/FEEDBACK.md`

## Backup

Pre-symlink source directories were moved to:

```text
/home/hyde/workspace/process/quant-lab/backups/cr087-pre-symlink-20260617T193645
```
