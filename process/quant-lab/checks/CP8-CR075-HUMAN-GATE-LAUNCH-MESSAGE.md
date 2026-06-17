请审查：process/checkpoints/CP8-CR075-DELIVERY-READINESS.md
自动预检结论：PASS_WITH_RISK
上下文胶囊：process/context/CP8-CR075-DELIVERY-CONTEXT.yaml（read_profile=compact，完整来源见 checklist）
本轮待人工决策项：5
决策收集覆盖：已扫描 6 个来源，发现候选问题 27 个，纳入待决策 10 个，其中 5 个形成本轮 CP8 DQ；N/A / 缺失来源 1 个，原因见 checklist 的 Decision Collection Coverage。
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 7 类禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR075-01 | risk_acceptance | 是否接受 CR075 READY_WITH_RISK 并关闭当前交付？ | 接受并关闭 CR075 为 `closed-current-delivery / READY_WITH_RISK`。 | 保持 active 等待更广验证；reject。 | 推荐方案收敛已通过验证；备选会延长 active 或要求重定义关闭标准。 | 接受 minimal-smoke 风险。 |
| DQ-CP8-CR075-02 | scope | CR075 关闭范围是否只覆盖 target env rebuild 和最小 smoke？ | 限定为基础 dependency sync、import smoke、CR061 小回归和过程证据。 | 扩大到 data/reports；扩大到 optional groups/full tests。 | 推荐方案边界清晰；备选触碰未授权风险面。 | 不证明完整 runtime-ready。 |
| DQ-CP8-CR075-03 | security | CP8 approve 是否继续不授权 `.env`、reports/data、provider/lake/catalog、remote Git、runtime？ | 确认不授权，相关事项均需独立 CR。 | 同时授权 data/reports；同时授权 provider/lake/catalog 或 remote Git。 | 推荐方案保持最小安全面；备选改变敏感或外部状态。 | 防止凭据、数据、外部写入和交易风险。 |
| DQ-CP8-CR075-04 | runtime_authorization | CP8 approve 是否继续不授权 QMT/MiniQMT runtime、CR046 recovery、optional groups、full tests 和 destructive cleanup？ | 继续不授权。 | 同时恢复 CR046；删除 `.venv` 后重建；运行 full tests。 | 推荐方案符合已批准范围；备选需新运行授权或 destructive 授权。 | target `.venv` 保留，完整 runtime 状态未知。 |
| DQ-CP8-CR075-05 | follow_up_tracking | CR075 关闭后后续事项如何分流？ | 保留 CR076/CR077/CR078 和 CR046 recovery 为独立后续项，不自动启动。 | 立即启动 CR076；立即恢复 CR046；取消全部后续项。 | 推荐方案保留追踪且不扩大当前范围；备选会开启新门禁或丢失风险入口。 | 影响 data/reports、旧根退役、远端治理和 runtime 恢复路径。 |

不授权项：

- 不授权读取、打印、复制、迁移或恢复 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权读取、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权 provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement。
- 不授权远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 QMT / MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live。
- 不授权自动恢复或推进 CR046。
- 不授权 optional dependency groups、full test suite、删除 `.venv` 或 destructive cleanup。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。
回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。
审查后请直接回复以下任一整行：
approve
修改: <具体修改点>
reject
