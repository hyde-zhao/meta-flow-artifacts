---
cr_id: "CR-059"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 NAS 挂载持久化、外部存储目录初始化、迁移 manifest seed、backup point 和后续真实迁移门禁，必须保持 standard。"
rollback_to: "pre-CR059 systemd/NAS preparation baseline"
approval_result: "user-authorized-preparation-completed"
created_at: "2026-06-14T23:59:58+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-14T23:59:58+08:00"
source: "USER-20260614-COMPLETE-MIGRATION-PREPARATION"
parent_cr: "CR-058"
source_checkpoint: "process/checkpoints/CP8-CR058-DELIVERY-READINESS.md"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-059 Migration Preparation Foundation

## 变更描述

用户要求“将缺的准备工作全部执行完成”。本 CR 将迁移前置准备从口头计划落地为可审计基础设施：

- 三个 NAS NFS export 已临时挂载并验证读写。
- systemd mount unit 已安装并启用，支持重启后自动挂载。
- NAS 三层目录骨架已创建。
- 迁移 manifest seed、备份点计划、挂载计划和目录映射已固化。
- 已完成一次 cold pre-migration 备份快照和 Git bundle rollback ref。
- CR059 CP2 / CP3 / CP5 / CP8 证据已生成，标注本轮只完成准备，不执行真实迁移。

本 CR 不执行 repo-local move / rename / delete，不做批量路径替换，不执行迁移式数据搬家或路径切换，不替换 `MARKET_DATA_LAKE_ROOT`，不执行 Git remote 初始化 / push / tag / branch rename / history rewrite，不读取凭据，不启动 QMT / MiniQMT runtime，不做 provider fetch、lake write 或 catalog publish。CR059 只允许 cold backup copy 和 Git bundle rollback ref。

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 当前 active formal CR | `CR-046` paused | CR046 保持用户暂停的 CP6 PASS / ready-for-verification；本 CR 不恢复 CR046。 |
| 与 CR046 影响面重叠 | false | 本 CR 只处理 NAS / migration preparation foundation；不触碰 QMT / MiniQMT runtime、凭据、账户或交易动作。 |
| 与 CR058 关系 | follow-up / preparation execution | CR058 已关闭 static-only/no-op gate；CR059 执行迁移前置准备，但仍不执行真实迁移。 |
| 与后续真实迁移关系 | prerequisite | CR059 只关闭准备面；真实 repo-local migration、Git cutover、NAS/data-lake migration、runtime rebind 仍需后续 CR。 |

## 五维度影响分析

| 维度 | 影响对象 | 结论 | 处理动作 |
|---|---|---|---|
| 需求层 | 迁移准备范围 | 新增 | 明确“准备完成”不等于“真实迁移完成”。 |
| 场景层 | 挂载、目录骨架、备份点、manifest seed | 新增 | 增加验证场景：mount enabled、目录存在、写权限、容量、候选 manifest seed。 |
| 计划层 | CR059 prep gate | 新增 | 将真实迁移前置项收敛为独立 CR，后续 CR 只能消费其证据。 |
| 安全层 | NAS 写入、systemd root 写入 | 高风险但已授权 | 仅写空目录和 mount unit；不复制/删除源数据，不读取凭据。 |
| 交付层 | docs/release、process/checks、process/checkpoints、process/release | 新增 | 固化 CR059 evidence，便于后续 CR060+ 消费。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR046 paused 和 CR058 closed 记录 | 增加 CR059 closed-current-delivery。 |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused recovery point | 更新 last_action / next_action / cr_tracking。 |
| `process/changes/CR-053-FOLLOW-UP-TRACKING-2026-06-14.md` | 原文档更新 | 保留 FU-CR053-02..04 原台账 | 标注 NAS prep 已由 CR059 部分关闭，真实迁移仍未关闭。 |
| `ops/systemd/*.mount` | 新增 | N/A | 仓库内保留 systemd unit 源文件。 |
| `docs/release/NAS-MOUNT-PLAN-CR059.md` | 新增 | N/A | 记录 NFS exports、systemd unit、验证结果。 |
| `docs/release/NAS-DIRECTORY-MAP-CR059.md` | 新增 | N/A | 记录 hot / primary / cold 目录骨架。 |
| `docs/release/MIGRATION-MANIFEST-SEED-CR059.yaml` | 新增 | N/A | 后续真实迁移候选 seed，不授权执行。 |
| `docs/release/BACKUP-POINT-PLAN-CR059.md` | 新增 | N/A | 定义备份点，不执行备份复制。 |

## 已执行准备动作

| 动作 | 状态 | 证据 |
|---|---|---|
| 创建本地挂载点 | done | `/mnt/quant-lab-hot`、`/mnt/quant-lab-primary`、`/mnt/quant-lab-cold` |
| 临时挂载 NFS | done | `findmnt` 显示三个挂载均为 `nfs rw` |
| 验证容量 | done | hot 454G / primary 3.6T / cold 13T |
| 验证当前用户写权限 | done | `.quant-lab-mount-check` 和 `.cr059-write-check` 标记文件可创建 |
| 创建 NAS 目录骨架 | done | 88 个目录，见 `docs/release/NAS-DIRECTORY-MAP-CR059.md` |
| 安装 systemd mount units | done | `/etc/systemd/system/mnt-quant\\x2dlab\\x2d*.mount` |
| 启用开机挂载 | done | `systemctl is-enabled` 三项均为 `enabled` |
| 创建 manifest seed | done | `docs/release/MIGRATION-MANIFEST-SEED-CR059.yaml` |
| 创建 backup point plan | done | `docs/release/BACKUP-POINT-PLAN-CR059.md` |
| 执行 cold pre-migration backup | done | `/mnt/quant-lab-cold/backups/pre-migration/cr059/repo-data-snapshot-20260614`，约 4.4G |
| 创建 Git bundle rollback ref | done | `/mnt/quant-lab-cold/git-bundles/pre-cr059/local_backtest-pre-cr059-e7a3a0d.bundle`，`git bundle verify` PASS |

## 回退策略

| 回退对象 | 回退方式 | 注意事项 |
|---|---|---|
| 临时 NFS 挂载 | `sudo umount /mnt/quant-lab-*` | 只卸载，不删除 NAS 数据。 |
| systemd 持久化 | `sudo systemctl disable mnt-quant\\x2dlab\\x2d*.mount` 后删除 unit | 仅回退本 CR 安装的 unit。 |
| NAS 目录骨架 | 默认保留 | 空目录不影响现有数据；若要删除，需单独授权目录删除清单。 |
| 仓库证据文件 | Git revert / 后续 CR 修订 | 不使用 `git reset --hard` 或覆盖用户改动。 |

## 不授权项

| Item ID | 不授权操作 |
|---|---|
| NA-CR059-001 | 真实 repo-local file move / rename / delete |
| NA-CR059-002 | 批量路径替换 / reference rewrite |
| NA-CR059-003 | 复制、移动或删除历史数据、研究结果或 data lake 数据 |
| NA-CR059-004 | `MARKET_DATA_LAKE_ROOT` replacement |
| NA-CR059-005 | lake write / catalog publish / provider fetch |
| NA-CR059-006 | Git remote add / push / tag / branch rename / history rewrite |
| NA-CR059-007 | 凭据、`.env`、token、password、cookie、session、private key 读取或保存 |
| NA-CR059-008 | QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live |
| NA-CR059-009 | 恢复 CR046 CP7 或推进 CR046 runtime verification |

## 后续门禁

| 后续 CR | 目标 | 是否已由 CR059 完成 |
|---|---|---|
| CR060 | repo-local mechanical migration execution | 否；CR059 只准备 manifest seed 和 rollback facts。 |
| CR061 | framework layout / package boundary convergence | 否。 |
| CR062 | Git remote / release identity cutover | 否；远端仍未配置。 |
| CR063 | NAS / archive real migration | 否；目录骨架和 CR059 cold backup 已就绪，但未执行 archive 迁移式搬家。 |
| CR064 | Data lake root / catalog migration | 否；目标目录已就绪，但 root 未切换。 |
| CR065 | Trading runtime / QMT-MiniQMT rebind | 否；CR046 仍暂停。 |

## 处理结论

- 结论：`closed-current-delivery`
- 当前已完成：迁移准备基础设施。
- 当前未完成：真实迁移、数据备份复制、Git remote 初始化、data lake root 切换、runtime rebind。
- 下一步：进入真实迁移 CR 前，先冻结 candidate manifest，复用或刷新 CR059 backup / bundle evidence，并确认每个执行 CR 的 rollback_ref。
