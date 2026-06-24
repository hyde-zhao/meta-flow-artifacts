请审查：`process/checkpoints/CP5-CR138-RUNNER-QMT-OPERATIONAL-CONTROL-LLD-BATCH.md`

自动预检结论：PASS

上下文胶囊：`process/context/CP5-CR138-RUNNER-QMT-OPERATIONAL-CONTROL-CONTEXT.yaml`（Context Capsule Summary，read_profile=compact，完整来源见 checklist）

审批者摘要：
- 本次确认服务的整体目标：确认 CR138 Runner Control Plane 与 QMT Gateway Service Layer 的 8 份 full LLD 设计证据可作为后续受控实现输入。
- 推荐动作：approve。8 份 LLD、8 份 CP5 自动预检、CP5 context 和批次 result 均已齐，阻断项 0。
- approve 后会发生什么：8 个 Story 可从 `lld-ready-for-review` 推进为 `dev-ready` 候选；后续实现仍需按 Wave、依赖、merge owner、CP6/CP7 和 no-real-operation gate 执行。
- approve 不授权什么：不授权真实 runtime、QMT、MiniQMT、XtQuant、gateway 启动、端口绑定、凭据读取、账户 / 行情 / 订单查询、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入。
- 不确认会阻塞什么：阻塞 CR138 进入 story-execution；S01..S08 不能进入受控实现。

本轮待人工决策项：3

决策收集覆盖：已扫描 6 个来源，发现候选问题 8 个，纳入待决策 3 个；N/A / 缺失来源 0 个，原因见 checkpoint 的 Decision Collection Coverage。

用户反馈 / follow-up CR 覆盖处理：S02/S03 已吸收 CR137 offline batch run，分别补入 `RunPlanBatch / BatchPreflightResult` 与 `BatchOpsSummary`；S03 LLD 已补入 Runner P0 `OpsSummary / CLI Summary`；S05 LLD 已补入 `GatewayChangePlan` dry-run / compatibility / rollback target；CR126 / RA-CR097 / RA-CR101 的旧后续跟踪已收敛，真实 QMT adapter / xtquant runtime、QMT direct-run、order-write / simulation / live 仍只作为后续 scoped gate，不在本 CP5 授权中；WebSocket / gRPC / FIX 与多实例主备等本轮不支持。

决策分层：
- 必须用户决策：3
- 高风险策略确认：1
- agent 默认处理：0
- 仅审计记录：2

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR138-01 | implementation | 是否确认 S01..S08 八份 full LLD 作为后续实现输入 | approve 全批次 LLD，进入受控 story-execution 准备 | 修改指定 Story LLD；或拆成 Runner / Gateway 两个 CP5 子批次 | 推荐方案保持共享合同和授权边界一致；修改或拆批会延迟但可降低局部风险 | 影响后续 dev-ready 队列、Wave 调度和 CP6/CP7 验证入口 |
| DQ-CP5-CR138-02 | runtime_authorization | CP5 approve 是否授权真实 QMT / Gateway / 账户 / 行情 / 订单验证 | 不授权；真实验证仅能通过后续 scoped runtime_authorization gate | 同时授权 readonly 实机验证；或永久不允许真实验证 | 推荐方案与 CP3/CP4 一致并保留后续按需验证路径；直接授权风险高，永久不授权会阻断必要验收 | 影响 CP6/CP7 验证模式、安全责任和用户预期 |
| DQ-CP5-CR138-03 | implementation | 是否接受当前 Wave、file owner 和 merge order | 接受 S01 -> S02/S05 -> S03/S06/S07 -> S04/S08；`qmt_gateway_service.py` 由 S05 skeleton 后 S06/S07 串行扩展 | Gateway S06/S07 严格串行；或 Runner / Gateway 分支完全隔离到后续集成 Story | 推荐方案平衡并行和共享文件风险；严格串行更稳但慢；分支隔离降低冲突但增加集成 Story | 影响开发并行度、文件冲突和 CP6 调度 |

不授权项：
- 代码实现自动开始
- QMT / MiniQMT / XtQuant / gateway runtime 启动或连接
- gateway port bind / HTTP server start
- `.env`、token、secret、账号、资金、持仓、委托、成交、session、cookie、private key 读取
- 账户 / 行情 / 订单 / 成交真实查询或订阅
- submit / cancel / buy / sell / simulation / live
- NAS access / mount / list / read / write / publish / delete
- provider fetch / lake write / catalog publish / current pointer 修改
- Git remote write / push / tag

如果你回复 approve，表示接受上表全部推荐方案；不表示授权上方任何不授权项。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

请直接回复以下任一整行：

```text
approve
修改: <具体修改点>
reject
```
