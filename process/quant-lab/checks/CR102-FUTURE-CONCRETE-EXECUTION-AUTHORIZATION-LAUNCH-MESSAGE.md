# CR102 Future Concrete Execution Authorization Gate 发起消息

请审查：`process/checkpoints/CR102-FUTURE-CONCRETE-EXECUTION-AUTHORIZATION-GATE-REVIEW.md`

自动预检结论：PASS。当前没有执行任何真实 NAS/env/runtime/trading/provider 动作；真实执行仍等待本门禁 approve。

Context Capsule：`process/context/CR102-FUTURE-CONCRETE-EXECUTION-AUTHORIZATION-CONTEXT.yaml`，read_profile=compact。

决策收集覆盖：已扫描 8 个来源，发现候选问题 42 个，纳入待决策 6 个；N/A 来源为真实 NAS 内容、凭据/env/账户、QMT/MiniQMT/XtQuant/gateway runtime、执行机挂载路径，均未读取。

本轮待人工决策项：6。

如果你回复 approve，表示接受下表 6 项推荐方案，并授权下一步执行研究机侧 NAS 包交换验证；不表示授权访问执行机挂载路径、读取凭据/env/账户、启动 QMT/MiniQMT/XtQuant/gateway、交易、provider/lake/catalog publish、delete 或任何 CR102 scope 之外的 NAS 操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CONCRETE-CR102-01 | runtime_authorization | 是否授权研究机侧真实 NAS package exchange 验证？ | 授权 agent-assisted 研究机侧 staging -> package exchange -> release label；执行机侧手工验证。 | 只生成 runbook；只做 check/list/read；取消验证。 | 推荐方案能取得真实研究机侧交换证据；备选更保守但验证不足。 | 真实 NAS 写入 / publish 风险。 |
| DQ-CONCRETE-CR102-02 | security | 是否采用 CR059 研究机路径和身份 / 凭据边界？ | 使用 CR059 路径、`current-shell-user`、`none-needed`，不 mount、不 network probe。 | 用户另给 source；完全手工；授权 mount/probe。 | 推荐方案避免凭据和探测；备选会扩大权限或降低自动证据。 | 路径或权限异常时停止。 |
| DQ-CONCRETE-CR102-03 | runtime_authorization | operation allowlist / denylist 如何冻结？ | allow access/check/list/read/copy/write/publish；deny delete/mount/network_probe/credential/runtime/trading/provider。 | 去掉 publish；只做 check/list/read；另起 delete/mount gate。 | 推荐方案覆盖包交换；备选降低风险但验证不足。 | 写入/publish 改变 NAS 状态。 |
| DQ-CONCRETE-CR102-04 | implementation | 目录创建、覆盖、原子写和回滚策略？ | create-if-missing-only、forbid-overwrite、temp-file-then-rename、rollback-then-stop、timeout=300。 | overwrite-with-backup；手工预创建；source-only。 | 推荐方案能执行且限制污染；备选要么风险更高要么验证不足。 | 目标存在时停止，不覆盖。 |
| DQ-CONCRETE-CR102-05 | security | evidence 如何脱敏并保存？ | redacted-hash-count，只写本地 process evidence。 | 用户手工 evidence；pass/fail only；原始日志。 | 推荐方案可审计且不泄露内容；原始日志不可接受。 | 元数据仍需限定到本 release id。 |
| DQ-CONCRETE-CR102-06 | risk_acceptance | 是否接受执行机侧手工验证残余风险？ | 接受：agent 不访问执行机路径，执行机 evidence 由用户补充或后续 gate。 | 提供执行机 mount root 重出门禁；不声明执行机验证完成；取消 pull/readback。 | 推荐方案避免越权；备选可更完整但需新输入。 | 本轮不能由 agent 证明执行机 pull 成功。 |

不授权项：

- 不访问 CR102 scope 之外的任何 NAS 路径。
- 不访问执行机挂载路径。
- 不删除任何 NAS 正式包、备份、历史归档或非本次 temp 文件。
- 不覆盖已有目标包；目标已存在则停止。
- 不读取 `.env`、凭据、token、secret、账号、账户、资金、持仓、委托、成交或原始日志。
- 不启动 QMT/MiniQMT/XtQuant/gateway runtime。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不恢复 CR089 或 CR020。

回复 `approve` 表示接受上表 6 项推荐方案，并授权下一步执行研究机侧 NAS 包交换验证；如需调整，请回复 `修改: <具体修改点>`；如不同意，请回复 `reject`。

可回复：

approve

修改: <具体修改点>

reject
