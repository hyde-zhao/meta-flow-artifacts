# CP2 CR102 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR102-NAS-REAL-PACKAGE-EXCHANGE-SCOPE-REVIEW.md`

自动预检结论：PASS。预检允许发起 CP2；真实 NAS execution 仍被阻断，因为 path/mount/permission 和 read/write/copy/publish/pull/delete/check 授权未被用户明确批准。

Context Capsule：`process/context/CP2-CR102-NAS-REAL-PACKAGE-EXCHANGE-CONTEXT.yaml`（read_profile=compact）。

决策收集覆盖：已扫描 6 个来源，发现候选问题 12 个，纳入待决策 5 个；N/A / 缺失来源 0 个。`CR-089` 与本 CR 在 `nas_package_exchange` 语义上重叠，已作为 `PASS_WITH_RISK` 纳入 DQ-CP2-CR102-02。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案：启动 CR102 作为正式门禁、接受 CR089 overlap 风险但不恢复 CR089、不授权真实 NAS path/mount/permission、不授权 read/write/copy/publish/pull/delete/check、不授权凭据/env/账户/runtime/交易/publish。approve 不表示授权以下不授权项。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR102-01 | scope | 是否启动 CR102 作为 `RA-CR101-003` 的正式 NAS real package exchange 授权门禁？ | 启动 CR102，但 CP2 当前只批准门禁范围与禁止边界，不授权任何 NAS 执行。 | 保持候选不启动；合并回 CR089；取消 NAS real exchange 验证。 | 推荐方案可建立可审计授权边界；备选会阻塞、扩大范围或关闭缺口。 | 影响 CR101 后续真实验证路线和 CR089 blocked 关系。 |
| DQ-CP2-CR102-02 | risk_acceptance | 是否接受与 `CR-089` 的语义重叠风险？ | 接受为 `PASS_WITH_RISK`：CR102 只处理 NAS exchange gate，不恢复 CR089/QMT runtime。 | 先解锁 CR089；等待 CR089 重评审；将 CR102 标记 blocked。 | 推荐方案边界最窄；解锁 CR089 会扩大运行验证语义。 | 误读可能导致 NAS/QMT runtime 越权。 |
| DQ-CP2-CR102-03 | runtime_authorization | 真实 NAS path / mount / permission 如何处理？ | 当前不授权，path/export/root/mount/identity 全部保持 `UNSET_BY_USER`；用户必须用 `修改:` 提供具体值后才能授权后续 CP。 | 提供真实 path/mount/permission 并限定 read/check；授权 publish/pull/copy；授权 mount。 | 推荐方案避免凭空推断；备选需要更细回退和证据策略。 | 没有明确 path/mount/permission 时，任何 NAS 操作都必须 BLOCKED。 |
| DQ-CP2-CR102-04 | runtime_authorization | read/write/copy/publish/pull/delete/check 授权范围是什么？ | 当前全部为 false；只允许本地文档和账本检查。 | 只授权 `check` 且不读取内容；授权 read/copy/pull；授权 write/publish/delete。 | 推荐方案最安全；publish/delete 风险最高。 | publish/delete 可能改变真实 NAS 状态；pull/copy 可能接触真实包内容。 |
| DQ-CP2-CR102-05 | security | 是否允许读取 env/凭据/账户或启动 QMT/MiniQMT/XtQuant/gateway？ | 不允许；NAS gate 不包含凭据/env/账户读取或 runtime 启动。 | 用户手工提供脱敏 evidence；独立 runtime gate 授权；agent 代跑 runtime。 | 推荐方案符合最小权限；备选进入高风险运行验证。 | 凭据泄露、账户原文落库、交易风险。 |

不授权项：

- 不访问、列取、读取、复制、写入、发布、拉取、删除或挂载真实 NAS。
- 不读取 Windows/Linux `.env`、NAS 凭据、token、secret、账号、账户、资金、持仓、委托、成交或原始日志。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不恢复 CR089 或 CR020，不自动启动 QMT direct-run、MiniQMT gateway 或 order-write gate。

请直接回复以下任一整行：

approve

修改: <具体修改点>

reject
