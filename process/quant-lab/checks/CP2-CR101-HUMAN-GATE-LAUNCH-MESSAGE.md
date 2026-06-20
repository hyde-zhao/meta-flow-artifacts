请审查：/home/hyde/workspace/quant-lab/process/checkpoints/CP2-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-SCOPE-REVIEW.md

自动预检结论：PASS

Context Capsule 摘要：使用 `process/context/CR091-FU-05-START-HANDOFF-2026-06-19.md`，read_profile=minimal；启动时只必读 handoff、STATE、当前需求基线、CR-INDEX、CR091 follow-up tracking；CR098 / CR100 / HLD / strategy_runner 代码仅按需读取。

决策收集覆盖摘要：已扫描 `process/STATE.md`、`process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml`、`process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md`、`process/changes/CR-INDEX.yaml`、启动 handoff 和当前用户不授权约束；候选问题 24 项，纳入待决策 5 项，其余归类为历史证据、非阻断开放项或后续独立 gate。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 7 项禁止操作。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR101-01 | scope | 是否启动 CR101 并限定为跨平台策略交付与 adapter layer 重对齐？ | 启动 standard CR101，仅做设计 / schema / fixture / checker / package layout 重对齐。 | 暂缓启动；只做文档勘误。 | 推荐可先修正历史 `miniqmt_runner` 混淆；备选会继续阻塞 CR089 / order-write。 | 不证明真实 runtime ready。 |
| DQ-CP2-CR101-02 | architecture | runner 与 MiniQMT 的关系如何冻结？ | runner 固定为 quant-lab-side；MiniQMT 只是当前 adapter target。 | MiniQMT runner-hosted；双 runner 模型。 | 推荐职责清晰并可扩展；备选耦合平台或增加验证矩阵。 | 影响 HLD、adapter protocol、manifest 和 checker。 |
| DQ-CP2-CR101-03 | implementation | 当前策略交付 target 支持哪些平台？ | 当前 implemented 仅 QMT direct-run；保留 future target 扩展位。 | 同时实现 MiniQMT target；只做抽象 schema。 | 推荐控制范围且匹配当前支持；备选扩大风险或无法解除阻塞。 | 影响 package generation、entrypoint、fixture、checker。 |
| DQ-CP2-CR101-04 | runtime_authorization | 本 CR 是否授权真实 NAS / QMT / MiniQMT / XtQuant / gateway / simulation/live / 交易 / publish？ | 不授权；只允许离线文件、schema、fixture、checker 和本地包结构验证。 | 授权用户手工真实验证；授权 agent 代跑 runtime / NAS / publish。 | 推荐权限最小；备选需要独立 evidence 和运行授权。 | 真实验证必须另起逐 run gate。 |
| DQ-CP2-CR101-05 | risk_acceptance | 是否接受先完成离线重对齐、真实运行后置的 READY_WITH_RISK 路线？ | 接受离线优先；CP8 可按证据关闭为 READY_WITH_RISK 并保留后续 gate。 | 等真实环境授权后再做；关闭为 blocked。 | 推荐先修正架构和交付契约；备选延迟解除混淆。 | 残余风险是真实 runtime、NAS 权限或交易安全未验证。 |

不授权项：
- 不访问、列取、读取、复制、写入、挂载、发布或删除真实 NAS。
- 不读取 Windows/Linux `.env`、NAS 凭据、token、secret、账号、账户、资金、持仓、委托、成交或原始日志。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不由 agent 代跑 QMT direct-run 或 MiniQMT gateway runtime 验证。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不把 CP2 设计通过解释为运行授权；真实运行和 publish 类动作必须另起逐 run 授权。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

approve
修改: <具体修改点>
reject
