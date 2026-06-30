# CP5-W2-DATA-CONTRACTS 批次准备发起消息

请审查：`process/checkpoints/CP5-CR139-W2-DATA-CONTRACTS-LLD-BATCH-REVIEW.md`

自动预检结论：`BLOCKED` for formal CP5 approval。这里的 BLOCKED 是预期状态：批次选择和设计证据收敛计划已准备完成，但 S09 full LLD 与八个 technical-note 尚未实际产出，因此不能发起正式 CP5 approval。

上下文胶囊：`process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml`（read_profile=compact）。

审批者摘要：
- 本次确认服务的整体目标：启动 CR139 后续 CP5 批次准备，默认选择 `CR139-W2-DATA-CONTRACTS`，冻结 S09/S14/S22/S30/S31/S32/S33/S34/S39 的设计证据收敛范围、顺序、S33 验收补充和不授权边界。
- 推荐动作：`approve` 本批次准备方案。
- approve 后会发生什么：进入设计证据写作，S09 产出 full LLD，S14/S22/S30/S31/S32/S33/S34/S39 产出 technical-note；完成后重新生成正式 CP5 自动预检和 Decision Brief。
- approve 不授权什么：不授权实现、runtime、真实 lake 写入、provider catalog 写入、provider-lake-catalog 写入、published pointer advance execution、物理分区迁移执行、凭据读取、NAS/QMT/trading 或 Git remote write。
- 不确认会阻塞什么：阻塞 CR139 Wave2/W3 的设计证据收敛；任何 W2/W3 Story 都不得 dev-ready 或实现。

本轮待人工决策项：3

决策收集覆盖：已扫描 STATE.current、Wave1 follow-up context、follow-up CP5 context、Story status、Development plan、Story cards、`catalog.py` symbol scan 和用户显式约束；候选问题 10，纳入待决策 3。

决策分层：
- 必须用户决策：2
- 高风险策略确认：1
- agent 默认处理：3
- 仅审计记录：2

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR139-W2-DATA-01 | implementation | 是否按默认启动 `CR139-W2-DATA-CONTRACTS` 作为后续首个 CP5 设计证据批次？ | 启动该批次，覆盖 S09/S14/S22/S30/S31/S32/S33/S34/S39。 | 先启动 ML/read consistency；或只启动 S33/S39 catalog 子批。 | 推荐方案先冻结共享数据契约，降低后续返工；备选可能更快但返工或延期风险更高。 | 影响后续 CP5 顺序、文件 merge_order 和 Story 可实现性。 |
| DQ-CP5-CR139-W2-DATA-02 | implementation | 是否把 `catalog.py` CR014/CR018 命名清理作为 S33 验收补充？ | 纳入 S33，新增领域命名 API，保留旧 alias 和兼容测试；S39 只做审计链。 | 保持历史名称不动；或破坏性删除旧 API。 | 推荐方案改善语义且保留兼容；保持不动会继续扩散 CR 命名；破坏性删除风险高。 | 影响 `catalog.py` public API、测试和历史合同兼容。 |
| DQ-CP5-CR139-W2-DATA-03 | runtime_authorization | 本轮是否授权 runtime、真实 lake 写入、provider catalog 写入、pointer advance 或物理分区迁移？ | 本轮不授权任何上述操作，仅允许设计证据收敛。 | 后续只授权 fixture/dry-run；或另走 scoped real operation runtime gate。 | 推荐方案风险最低并符合 CP5 准备门；真实执行必须独立授权。 | 安全、数据一致性和审计风险高；误授权会影响真实 lake、catalog pointer 和分区结构。 |

不授权项：
- runtime
- 真实 lake 写入
- provider catalog 写入 / provider-lake-catalog 写入
- published pointer advance execution
- physical partition migration execution
- credential / secret read
- NAS / QMT / MiniQMT / gateway runtime
- trading / small_live / live
- Git remote write

如果你回复 approve，表示接受上表 3 项推荐方案，并允许进入本批次设计证据写作；不表示正式 CP5 通过，不授权实现，也不授权任何不授权项。

回复 `修改: <具体修改点>` 表示调整批次、S33 验收补充或禁止项。

回复 `reject` 表示不启动该批次准备，回到 follow-up batch selection。

approve
修改: <具体修改点>
reject
