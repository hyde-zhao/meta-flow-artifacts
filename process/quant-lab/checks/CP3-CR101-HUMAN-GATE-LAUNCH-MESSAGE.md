请审查：/home/hyde/workspace/quant-lab/process/checkpoints/CP3-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-HLD-REVIEW.md
自动预检结论：PASS，阻断项 0。预检文件：process/checks/CP3-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-HLD-PRECHECK.md。

Context Capsule 摘要：process/context/CP3-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-CONTEXT.yaml 已生成；推荐架构为 target + adapter 双边界，当前唯一 implemented target 为 qmt_terminal_direct，MiniQMT 仅为 gateway / broker adapter contract。

Decision Collection Coverage 摘要：已扫描 CP2 review、CR101 HLD、CP3 discussion log、当前需求基线、CR091/CR098/CR100 HLD 和 strategy_runner 接口；候选问题 21 项，纳入本轮待人工决策 5 项，NAS / 凭据 / runtime / simulation/live / 交易 / publish 均分类为不授权项。

本轮待人工决策项：5
如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权以下 7 项禁止操作。

待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR101-01 | architecture | 是否批准 target + adapter 双边界作为 CR101 推荐架构？ | 批准方案 A：delivery_targets 表达策略交付 target，execution_adapters 表达 runner 到 broker/gateway 的 adapter。 | QMT-only 固化；沿用 miniqmt_runner 历史字段。 | 推荐方案职责最清晰；备选要么扩展返工，要么继续混淆。 | 影响 manifest、checker、fixture、package layout、HLD/LLD 和测试。 |
| DQ-CP3-CR101-02 | architecture | 当前 implemented delivery target 是否固定为 qmt_terminal_direct？ | 固定为唯一 implemented target；future target 只保留 schema slot。 | 同时实现 MiniQMT target；只做抽象 schema。 | 推荐方案可控且可验证；备选会扩大 runtime 误读或不可验证。 | 不证明真实 QMT ready，只证明离线合同。 |
| DQ-CP3-CR101-03 | implementation | CP5 是否按 manifest / checker / fixture / evidence 重对齐推进？ | CP5 进入离线 LLD，覆盖 schema、checker、fixtures、negative tests、evidence redaction 和 package layout。 | 只写文档；跳过 LLD 直接改代码。 | 推荐方案符合门禁并控制 shared file 冲突；备选风险更高。 | 影响 trading/strategy_runner/*、tests、docs/qmt、package fixtures。 |
| DQ-CP3-CR101-04 | runtime_authorization | CP3 approve 是否授权真实 NAS、凭据、QMT/MiniQMT/XtQuant/gateway runtime、simulation/live、交易或 publish？ | 不授权；CP3 只确认架构。 | 用户手工真实验证另起 per-run gate；授权 agent 代跑真实 runtime。 | 推荐方案权限最小；备选必须另起授权证据。 | 不证明真实 runtime、NAS、账号或交易链路 ready。 |
| DQ-CP3-CR101-05 | risk_acceptance | 是否接受历史 miniqmt_runner 字段迁移风险和 READY_WITH_RISK 路线？ | 接受，要求 CP5/CP7 加负向验证，CP8 可在未真实运行时关闭为 READY_WITH_RISK。 | 等真实环境授权后再做；关闭 CR101 为 blocked。 | 推荐方案先消除架构漂移；备选会阻塞或无法修正混淆。 | 残余风险是离线证据不能代表真实 NAS/QMT/MiniQMT/gateway。 |

不授权项：
- 不访问、列取、读取、复制、写入、挂载、发布或删除真实 NAS。
- 不读取 .env、token、API key、password、HMAC secret、cookie、session、private key 或 QMT 凭据。
- 不读取账户、资金、持仓明细、委托、成交或原始日志。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 submit / cancel / buy / sell / simulation / live。
- 不执行 provider fetch、lake write 或 catalog publish。
- 不把 CP3 HLD approve 解释为自动终验或真实运行授权。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。
回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。
审查后请在“人工审查结果”中填写结论，也可以直接回复以下任一整行：
approve
修改: <具体修改点>
reject
