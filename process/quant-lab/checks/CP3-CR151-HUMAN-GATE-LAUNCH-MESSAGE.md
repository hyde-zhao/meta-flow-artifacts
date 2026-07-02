请审查：`process/checkpoints/CP3-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md`

自动预检结论：PASS

Context Capsule：`process/context/CP3-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-CONTEXT.yaml`（read_profile=compact，完整来源见 checklist）

审批者摘要：
- 本次确认服务的整体目标：确认 CR151 的多因子统计准入门架构，让后续 Story planning 和实现只围绕本地/static/fixture Wave A admission gate 展开。
- 推荐动作：`approve`，批准 dedicated metadata-only statistical gate module、Wave A only、四态 fail-closed model 和 static/fixture-only validation boundary。
- approve 后会发生什么：进入 CP4/Story planning；之后仍需 CP5 全量设计证据确认，才允许源码实现。
- approve 不授权什么：不授权源码实现、真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer 操作。
- 不确认会阻塞什么：阻塞 CR151 Story planning、LLD、实现、验证和 release readiness。

本轮待人工决策项：4

决策收集覆盖：已扫描 6 个来源，发现候选问题 20 个，纳入待决策 4 个；N/A / 缺失来源 0 个。

决策分层：
- 必须用户决策：4
- 高风险策略确认：0
- agent 默认处理：3
- 仅审计记录：2

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR151-001 | architecture | 是否批准独立 metadata-only statistical gate contract module？ | 新建 dedicated module，暂名 `engine.strategy_admission_statistical_gate`，由 mature multifactor/admission 只消费 refs 和 summary。 | A. 嵌入 mature runner；B. 放入 strategy admission package；C. 文档-only。 | 推荐方案复用性和测试隔离最好；备选会增加 runner 宽度、混淆 runtime admission 或无法完成 E2E。 | 影响 CP4 Story split、文件 owner、测试边界。 |
| DQ-CP3-CR151-002 | scope | 是否保持 CR151 Wave A only？ | 只做 multiple testing/FDR、minimum robust stats、walk-forward/OOS、PBO/DSR、aggregate statistical gate；完整评价统计、regime 分层、因子聚类 / 去重等 Wave B 后置。 | A. 纳入完整 Wave B；B. 只做 PBO/DSR；C. 回到数据湖。 | 推荐方案最小闭环；A 过大且重叠 CR154；B 不完整；C 背离路线 A。 | 控制 CP5/CP6 风险，并避免静默声称覆盖完整因子评价体系。 |
| DQ-CP3-CR151-003 | architecture | 是否批准四态 fail-closed gate model？ | 使用 `PASS / FAIL / NEEDS_REVIEW / BLOCKED`；mandatory evidence missing 为 BLOCKED。 | A. 二态 pass/fail；B. 全部 real-data 前 blocked；C. 不区分 missing/threshold。 | 推荐方案审计语义最好；备选会隐藏缺证据或阻塞 fixture 目标。 | 影响 tests、CP7、CP8 release wording。 |
| DQ-CP3-CR151-004 | security | 是否保持 static/fixture-only validation 和 no-runtime 边界？ | 保持全部不授权：真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer。 | A. 授权真实 lake read；B. 授权外部框架运行；C. 授权 NAS/report write。 | 推荐方案符合 CP2，风险最低；备选都需要独立 runtime_authorization。 | 避免把 CR151 设计扩展为运行时或真实收益验证。 |

不授权项：
- source implementation before CP3/CP5 approval
- real lake read or write
- NAS read/write/sync/chmod/chgrp/metadata normalization
- provider fetch
- QMT/MiniQMT/xtquant runtime
- simulation/paper/live/trading runtime
- broker read/write/submit/cancel/account query
- credential or `.env` read
- external framework clone/install/run
- Git remote write
- catalog pointer mutation

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

如果你回复 approve，表示接受上表全部推荐方案，并允许进入 CP4 Story planning。

不表示授权：源码实现、真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer 操作。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

可回复：

approve

修改: <具体修改点>

reject
