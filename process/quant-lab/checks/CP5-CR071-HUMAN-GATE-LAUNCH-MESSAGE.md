请审查：process/checkpoints/CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS.md

自动预检结论：PASS
上下文胶囊：process/context/CP5-CR071-EXECUTION-CONTEXT.yaml
本轮待人工决策项：6

决策收集覆盖：已扫描 6 个来源，发现候选问题 17 个，纳入待决策 6 个；N/A / 缺失来源 2 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 6 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR071-01 | implementation | CP5 approved 后是否允许执行本机 copy-first？ | CP2/CP3/CP5 approved 且 preflight PASS 后允许。 | 只计划；只 dry-run。 | 推荐方案达成目标。 | 新增 target 目录。 |
| DQ-CP5-CR071-02 | implementation | 是否冻结精确命令？ | `mkdir -p` + `rsync -a` + 5 个 exclude。 | `cp -a`；tracked-only archive。 | 推荐方案可控。 | 影响复制完整性。 |
| DQ-CP5-CR071-03 | security | 执行前 preflight 如何处理？ | 重新检查 target 不存在/为空、rsync 可用、source 存在。 | 忽略 target 状态。 | 推荐方案防覆盖。 | preflight FAIL 则停止。 |
| DQ-CP5-CR071-04 | implementation | 执行后验证范围是什么？ | target/old root/excluded paths/Git status。 | 只看 exit code；跑完整 pytest。 | 推荐方案聚焦迁移。 | 形成 CP6/CP7 证据。 |
| DQ-CP5-CR071-05 | risk_acceptance | partial target 是否自动清理？ | 不自动清理，等待用户授权。 | 自动删除；自动覆盖重试。 | 推荐方案避免二次破坏。 | 可能留下 partial 目录。 |
| DQ-CP5-CR071-06 | runtime_authorization | CP5 approve 是否授权外部动作或最终关闭？ | 不授权；复制后继续 CP6/CP7/CP8。 | 合并 remote/NAS/lake/runtime；直接关闭。 | 推荐方案保持门禁。 | 外部状态不变。 |

不授权项：

- 不授权复制 `reports/`、`data/`、`.venv/`、`node_modules/` 或 `.env`。
- 不授权删除、重命名或移动旧 root；不授权自动删除 partial target。
- 不授权读取、打印或迁移任何凭据或 `.env` 内容。
- 不授权 remote Git write、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 NAS/data lake/provider/catalog/runtime/QMT/MiniQMT/CR046 recovery。
- 不授权复制后自动切换 shell、IDE、systemd、cron、数据湖 root 或日常工作根。

approve
修改: <具体修改点>
reject
