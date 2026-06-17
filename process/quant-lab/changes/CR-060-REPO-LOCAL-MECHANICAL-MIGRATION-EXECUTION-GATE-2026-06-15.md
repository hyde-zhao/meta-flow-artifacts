---
cr_id: "CR-060"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 repo-local 真实文档改写、legacy alias 兼容、preserve-audit 边界、rollback_ref 与后续真实迁移分层，必须保持 standard。"
rollback_to: "pre-CR060 repo-local identity baseline"
approval_result: "cp8-approved-ready-with-risk"
created_at: "2026-06-15T06:43:41+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-15T07:14:03+08:00"
source: "USER-20260615-RESUME-MIGRATION-AFTER-DEVICE-INTERRUPTION"
parent_cr: "CR-059"
source_checkpoint: "process/checkpoints/CP8-CR059-DELIVERY-READINESS.md"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-060 Repo-local Mechanical Migration Execution Gate

## 变更描述

CR060 从 CR059 的准备基础设施继续推进，目标是在执行真实 repo-local 机械迁移前冻结执行清单、回滚证据和人工授权边界。

本 CR 的第一执行切片限定为 Git 工作区内的 future-facing 用户文档身份收敛：

- 将 `README.md` 的项目身份说明从 `local_backtest` canonical 调整为 `quant-lab` canonical，并保留 `local_backtest` 为 legacy alias。
- 将 `docs/USER-MANUAL.md` 的项目根说明同步调整为 `quant-lab` canonical，并保留旧目录 / 旧命令的兼容说明。
- 不改写历史 `process/**`、checkpoint、Story、LLD、handoff、DEV-LOG 或 CR 审计语境。
- 不改 `pyproject.toml` 包名、import path、环境变量名、`MARKET_DATA_LAKE_ROOT` 或 data lake root。

当前状态：用户已对 CP5 门禁回复“同意”，按 `approve` 处理；CR060 已执行 manifest 中 `CR060-ACT-001` / `CR060-ACT-002` 两项文档改写，并完成 CP6 / CP7。用户随后对 CP8 回复“批准”，按 `approve` 处理；CR060 当前 docs-only delivery 关闭为 `READY_WITH_RISK / closed-current-delivery`。

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 当前 active formal CR | `CR-046` paused | CR046 保持用户暂停的 CP6 PASS / ready-for-verification；CR060 不恢复 CR046。 |
| 与 CR046 影响面重叠 | false | CR060 不触碰 QMT / MiniQMT runtime、凭据、账户、策略导入或交易动作。 |
| 与 CR058 关系 | execution follow-up | CR058 关闭 static-only/no-op gate；CR060 消费其 candidate list、preserve-audit allowlist 和 rollback gate。 |
| 与 CR059 关系 | preparation consumer | CR059 已完成 NAS mount、backup、bundle 和目录骨架；CR060 不执行 NAS 数据迁移。 |
| 与 Git remote 关系 | out-of-scope | 用户已给出远端地址，但 remote add / branch rename / push 放入后续 CR062，不混入本 CR。 |

## 五维度影响分析

| 维度 | 影响对象 | 结论 | 处理动作 |
|---|---|---|---|
| 需求层 | 真实迁移拆分 | 新增 | 将 repo-local 文档身份收敛作为第一个真实迁移执行切片。 |
| 场景层 | 用户阅读 README / USER-MANUAL | 修改 | 面向未来的用户入口显示 `quant-lab` canonical，同时解释 `local_backtest` legacy alias。 |
| 计划层 | CR060 CP2/CP3/CP5/CP6/CP7/CP8 | 新增 | CP5 已批准并执行两项文档改写；CP8 已批准关闭。 |
| 安全层 | 历史审计、敏感配置、运行边界 | 高风险但可控 | 默认 preserve-audit；不读取 `.env`；不碰 runtime / provider / lake / Git remote。 |
| 交付层 | docs/release、process/checks、process/checkpoints、process/changes | 新增 | 固化 execution manifest、move plan、rollback ref 和人工门禁。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| `README.md` | 原文档更新完成 | Git diff + CR060 manifest + CR059 bundle | 只改 future-facing identity 段，不改历史 CR 记录。 |
| `docs/USER-MANUAL.md` | 原文档更新完成 | Git diff + CR060 manifest + CR059 bundle | 与 README 保持一致。 |
| `process/**` historical evidence | 不改 | preserve-audit | 历史 `local_backtest` 保留为当时事实。 |
| `pyproject.toml` / import path | 不改 | out-of-scope | 后续 package/import compatibility CR 处理。 |
| `MARKET_DATA_LAKE_ROOT` | 不改 | keep | 数据湖 root replacement 后续独立 CR。 |
| Git remote / branch / push | 不改 | out-of-scope | 后续 CR062 处理。 |

## 已冻结输入

| 输入 | 路径 | 状态 |
|---|---|---|
| CR058 candidate list | `docs/release/CR058-CANDIDATE-LIST.md` | consumed |
| CR058 preserve-audit allowlist | `docs/release/CR058-PRESERVE-AUDIT-ALLOWLIST.md` | consumed |
| CR058 rollback gate | `docs/release/CR058-ROLLBACK-GATE.md` | consumed |
| CR059 manifest seed | `docs/release/MIGRATION-MANIFEST-SEED-CR059.yaml` | consumed |
| CR059 backup snapshot | `/mnt/quant-lab-cold/backups/pre-migration/cr059/repo-data-snapshot-20260614` | observed |
| CR059 git bundle | `/mnt/quant-lab-cold/git-bundles/pre-cr059/local_backtest-pre-cr059-e7a3a0d.bundle` | observed |

## 不授权项

| Item ID | 不授权操作 |
|---|---|
| NA-CR060-001 | 未通过 CP5 前执行任何真实 file move / rename / delete / text rewrite |
| NA-CR060-002 | 改写历史 `process/**`、checks、checkpoints、handoffs、Story、LLD、DEV-LOG 或历史 CR |
| NA-CR060-003 | `pyproject.toml` 包名、import path、module rename 或 dependency lock 改动 |
| NA-CR060-004 | `MARKET_DATA_LAKE_ROOT` replacement、data lake root 切换、lake write、catalog publish |
| NA-CR060-005 | NAS 数据复制 / 移动 / 删除 / archive promote |
| NA-CR060-006 | Git remote add / push / tag / branch rename / history rewrite |
| NA-CR060-007 | `.env`、token、password、cookie、session、private key、账户或未脱敏交易事实读取 |
| NA-CR060-008 | QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live |
| NA-CR060-009 | 恢复或推进 CR046 CP7 |

## 后续 CR 切分

| 后续 CR | 目标 | 与 CR060 的关系 |
|---|---|---|
| CR061 | framework layout / package boundary convergence | 处理包名、import 兼容、目录布局和测试入口。 |
| CR062 | Git remote / release identity cutover | 配置 `git@github.com:hyde-zhao/quant-lab.git`、分支/推送/发布身份门禁。 |
| CR063 | NAS / archive real migration | 将研究产物、过程证据和数据按 hot / primary / cold 分层迁移。 |
| CR064 | Data lake root / catalog migration | 独立备份、root 切换、catalog publish gate 和回滚演练。 |
| CR065 | Trading runtime / QMT-MiniQMT rebind | 独立恢复 CR046 或新交易 CR；不从 CR060 继承授权。 |

## 当前处理结论

- 结论：`closed-current-delivery / READY_WITH_RISK`
- 当前已完成：正式 CR 创建、影响分析、执行 manifest、preserve-audit 决策、rollback_ref、CP2 / CP3 / CP5 门禁、README / USER-MANUAL docs-only 改写、CP6 实现证据、CP7 验证证据、CP8 发布准备。
- 当前未完成：Git commit、remote 初始化、NAS/data-lake migration、runtime rebind。
- 下一步：CR060 已关闭；后续若继续完整迁移，按 CR061 package/import/layout convergence、CR062 Git remote cutover、CR063 NAS/archive、CR064 data lake root/catalog、CR065 trading runtime 分层推进。
