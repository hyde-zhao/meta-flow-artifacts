请审查：process/checkpoints/CP2-CR071-REQUIREMENTS-BASELINE.md

自动预检结论：PASS
上下文胶囊：process/context/CP2-CR071-REQUIREMENT-CONTEXT.yaml
本轮待人工决策项：5

决策收集覆盖：已扫描 6 个来源，发现候选问题 15 个，纳入待决策 5 个；N/A / 缺失来源 2 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权不授权项中的操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR071-01 | scope | 是否正式启动 CR071 / MIG-A？ | 启动为正式 active gate，CR046 仍 paused。 | 保持 candidate；取消。 | 推荐方案可审计推进。 | 影响 CR tracking。 |
| DQ-CP2-CR071-02 | scope | 是否接受 copy-first shadow root 范围？ | local_backtest -> sibling quant-lab。 | rename/move；只写计划。 | 推荐方案保留旧 root。 | 影响本机目录布局。 |
| DQ-CP2-CR071-03 | security | 是否确认排除清单？ | 排除 reports/data/.venv/node_modules/.env。 | 额外排除；允许部分复制。 | 推荐方案匹配用户请求。 | target 初始无数据/报告。 |
| DQ-CP2-CR071-04 | runtime_authorization | CP2 approve 是否授权外部/runtime？ | 不授权 NAS、remote Git、QMT runtime、data lake。 | 合并外部动作。 | 推荐方案分层清晰。 | 防止外部副作用。 |
| DQ-CP2-CR071-05 | risk_acceptance | 是否接受 target 初始不是完整 artifact/data mirror？ | 接受排除项导致的缺口。 | 完整镜像；tracked-only。 | 推荐方案风险较低。 | 后续可能需重建 venv/data/reports。 |

不授权项：

- 不授权删除、重命名或移动 `/home/hyde/workspace/local_backtest`。
- 不授权复制 `reports/`、`data/`、`.venv/`、`node_modules/` 或 `.env`。
- 不授权读取、打印或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS、远端 Git 写动作、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 data lake 切换、provider fetch、lake write、catalog publish、QMT / MiniQMT runtime 或 CR046 recovery。

approve
修改: <具体修改点>
reject
