# CR059 NAS Directory Map

## Hot Tier: `/mnt/quant-lab-hot`

| Path | Purpose |
|---|---|
| `staging/migration` | 迁移执行前 staging |
| `staging/data-lake-import` | data lake import staging |
| `staging/release` | release package staging |
| `working/latest-reports` | 最新报告工作副本 |
| `working/latest-research-runs` | 最新研究运行工作副本 |
| `working/cache` | 可重建 cache |
| `indexes/catalog-cache` | catalog cache |
| `indexes/manifest-cache` | manifest cache |
| `tmp/dry-run` | dry-run 临时目录 |
| `tmp/preflight` | preflight 临时目录 |
| `tmp/quarantine` | 隔离目录 |

## Primary Tier: `/mnt/quant-lab-primary`

| Path | Purpose |
|---|---|
| `project/docs` | 项目长期文档副本 |
| `project/release-evidence` | 发布证据 |
| `project/change-records` | CR 汇总副本 |
| `project/verification` | 验证报告 |
| `research/runs` | 研究运行产物 |
| `research/reports` | 研究报告 |
| `research/factor-research` | 因子研究过程产物 |
| `research/strategy-admission` | 策略准入过程产物 |
| `research/portfolio` | 组合研究产物 |
| `research/experiments` | 实验产物 |
| `research/notebooks` | notebook 归档目标 |
| `process/checkpoints` | 人工检查点归档目标 |
| `process/handoffs` | handoff 归档目标 |
| `process/execution-logs` | 执行日志归档目标 |
| `process/decisions` | 决策归档目标 |
| `process/cp-evidence` | CP 证据归档目标 |
| `data-lake/raw` | raw lake 目标 |
| `data-lake/manifest` | manifest 目标 |
| `data-lake/canonical` | canonical 目标 |
| `data-lake/quality` | quality 目标 |
| `data-lake/catalog` | catalog 目标 |
| `data-lake/gold` | gold 目标 |
| `data-lake/snapshots` | lake snapshots |
| `broker/package-exchange` | package exchange |
| `broker/broker-lake` | broker lake |
| `broker/qmt` | QMT 目标目录 |
| `broker/miniqmt` | MiniQMT 目标目录 |
| `archive/repo-snapshots` | repo snapshot |
| `archive/release-packages` | release package archive |
| `archive/migration-evidence` | migration evidence |

## Cold Tier: `/mnt/quant-lab-cold`

| Path | Purpose |
|---|---|
| `backups/pre-migration/cr059`..`cr065` | 各 CR 迁移前备份入口 |
| `backups/data-lake` | data lake 备份 |
| `backups/research` | research 备份 |
| `backups/process` | process 备份 |
| `backups/repo` | repo 备份 |
| `git-bundles/pre-cr059`..`pre-cr063` | Git bundle checkpoint |
| `snapshots/daily` | daily snapshot 入口 |
| `snapshots/weekly` | weekly snapshot 入口 |
| `snapshots/monthly` | monthly snapshot 入口 |
| `restore-drills/cr062` | NAS restore drill |
| `restore-drills/cr063` | data lake restore drill |
| `cold-archive/raw-history` | raw history cold archive |
| `cold-archive/old-reports` | old report archive |
| `cold-archive/old-process-evidence` | old process evidence archive |
| `failure-recovery` | failure recovery evidence |

## Verification

- 目录总数：88。
- 当前用户写权限：已通过 `.cr059-write-check` 验证。
- 本目录骨架为空目录准备，不表示任何数据已迁移。
