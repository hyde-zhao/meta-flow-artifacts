请审查：/home/hyde/workspace/quant-lab/process/checkpoints/CP3-CR102-NAS-REAL-PACKAGE-EXCHANGE-HLD-REVIEW.md
自动预检结论：PASS，阻断项 0 for CP3 gate / 1 for execution。预检文件：process/checks/CP3-CR102-NAS-REAL-PACKAGE-EXCHANGE-HLD-PRECHECK.md。

Context Capsule 摘要：process/context/CP3-CR102-NAS-REAL-PACKAGE-EXCHANGE-CONTEXT.yaml 已生成；推荐架构为 two-plane authorization design：authorization ledger + future execution plan shell + evidence schema。真实 NAS path/export/root/mount/identity 仍为 UNSET_BY_USER，read/write/copy/publish/pull/delete/check/mount/list/access 仍全部 false。

决策收集覆盖（Decision Collection Coverage）摘要：已扫描 CP2 review、CR102 formal CR、CR102 HLD、CP3 discussion log/checkpoint、CR100/CR101 本地 HLD 示例、CR index/STATE 和用户显式约束；候选问题 26 项，纳入本轮待人工决策 5 项；BLUEPRINT / DOMAIN-MAP / DEPENDENCY-MAP 对单一 runtime authorization gate 记为 N/A，原因见 checklist。

本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 6 类禁止操作。

待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR102-01 | architecture | 是否批准 CR102 采用 two-plane authorization design？ | 批准 authorization ledger + future execution plan shell + evidence schema 分离架构。 | check-only direct design；merge into CR089；cancel real NAS gate。 | 推荐方案权限最小且可审计；备选要么需要真实授权，要么扩大 runtime 风险，要么保留真实 NAS 缺口。 | 影响 CP5 设计对象、后续 gate、CR089 边界和验证路径。 |
| DQ-CP3-CR102-02 | security | path/export/root/mount/identity 与 permission 是否继续 user-explicit-only？ | 保持全部 UNSET_BY_USER / not-authorized；agent 不发现、不探测、不读取 env。 | 用户在修改中提供路径和身份；授权 agent 探测；暂停等人工线下确认。 | 推荐方案最安全；用户提供值可推进但需重审；agent 探测越权；线下确认会延后。 | 防止 NAS 探测、凭据读取和路径泄漏。 |
| DQ-CP3-CR102-03 | implementation | CP5 是否只产出设计包，不执行 NAS？ | CP5 只产出 authorization matrix、execution plan shell、evidence schema、rollback checklist。 | CP5 直接实现 check 脚本；CP5 直接执行只读 check；只写文档关闭。 | 推荐方案符合门禁；脚本实现仍可能被误执行；直接 check 越权；文档-only 不能提高 readiness。 | 影响后续 Story/LLD/TASKS 和验证方式。 |
| DQ-CP3-CR102-04 | runtime_authorization | CP3 approve 是否授权 NAS/env/runtime/trading/publish 动作？ | 不授权；所有真实动作继续 false。 | 用户另起 per-run NAS check gate；授权 agent 代跑真实 NAS；授权 runtime/QMT 联动。 | 推荐方案权限最小；per-run gate 可控但需输入；agent 代跑和 runtime 联动风险最高。 | 防止 CP3 被误读为外部操作许可。 |
| DQ-CP3-CR102-05 | risk_acceptance | 是否接受 CR089 overlap 与 CR102 blocked-for-execution 状态？ | 接受 PASS_WITH_RISK：CR102 只处理 NAS gate 架构，不恢复 CR089；CP3 后仍 blocked-for-execution。 | 等 CR089 重评审；合并 CR102 到 CR089；取消 CR102。 | 推荐方案边界最窄；等待或合并会阻塞/扩大风险；取消会关闭真实 NAS 路线。 | 影响 CR tracking、follow-up backlog 和后续真实验证路线。 |

不授权项：
- 不访问、列取、读取、复制、写入、发布、拉取、删除、检查或挂载真实 NAS。
- 不读取 .env、token、secret、账号、账户、资金、持仓、委托、成交或原始日志。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 submit / cancel / buy / sell / simulation / live。
- 不执行 provider fetch、lake write 或 catalog publish。
- 不恢复 CR089 或 CR020，不自动启动 QMT direct-run、MiniQMT gateway 或 order-write gate。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。
回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。
审查后请在“人工审查结果”中填写结论，也可以直接回复以下任一整行：
approve
修改: <具体修改点>
reject
