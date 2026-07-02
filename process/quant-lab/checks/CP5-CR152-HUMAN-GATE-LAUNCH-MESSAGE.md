请审查：`process/checkpoints/CP5-CR152-ML-STRATEGY-E2E-LLD-BATCH.md`

自动预检结论：PASS

Context Capsule：`process/context/CP5-CR152-ML-STRATEGY-E2E-CONTEXT.yaml`（read_profile=compact）

审批者摘要：
- 本次确认服务的整体目标：确认 CR152 的 5 个 Story 设计证据可作为后续本地/static/fixture 实现输入。
- 推荐动作：`approve`，批准 S01-S04 full LLD、S05 technical-note、实现顺序、active `triple_barrier` / `meta_label` first-wave `BLOCKED` enforcement 和 no-real-op 安全边界。
- approve 后会发生什么：CR152 可进入受控 story-execution，后续仍只允许本地/static/fixture 源码实现和测试。
- approve 不授权什么：不授权真实训练、真实数据验证、model registry/store/catalog 写入、lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote 操作。
- 不确认会阻塞什么：阻塞 CR152 进入实现；ML Strategy E2E first-wave foundation 无法进入 CP6。

本轮待人工决策项：4
本轮待人工决策项: 4

决策收集覆盖：已扫描 5 个来源，发现候选问题 8 个，纳入待决策 4 个；N/A / 缺失来源 0 个。

决策分层：
- 必须用户决策：4
- 高风险策略确认：1
- agent 默认处理：2
- 仅审计记录：3

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR152-001 | implementation | 是否确认 S01-S04 full LLD + S05 technical-note 作为后续实现输入？ | approve 全批次设计证据，进入本地/static/fixture 实现。 | A. 修改指定 Story LLD；B. 拆成两个 CP5 子批次。 | 推荐方案保持 first-wave 链路一致；修改可降低局部风险但延迟；拆批增加跨契约漂移。 | 影响 dev-ready 队列、Wave 调度和 CP6/CP7 验证入口。 |
| DQ-CP5-CR152-002 | implementation | 是否接受当前实现顺序和文件 owner？ | 接受 S01 -> S02 -> S03 -> S04 -> S05；共享文件按 merge owner 串行。 | A. 合并 S01/S02；B. S03-S05 后置。 | 推荐方案降低共享文件冲突并保持 E2E 闭环。 | 影响 implementation wave、merge owner 和回归范围。 |
| DQ-CP5-CR152-003 | implementation | 是否接受 active `triple_barrier` / `meta_label` first-wave exact enforcement？ | 选中 `triple_barrier` 或 `meta_label` 时返回 `BLOCKED`，因为算法仅预留 slot，未实现。 | A. 返回 `NEEDS_REVIEW`；B. 本轮实现算法；C. 移除扩展 slots。 | 推荐方案消除 HLD 模糊且不扩大范围。 | 影响 S01 validator、S04 aggregate status 和 CP7 release wording。 |
| DQ-CP5-CR152-004 | security | CP5 approve 是否授权任何真实训练、真实数据、registry/store/catalog 写入或 runtime 操作？ | 不授权；仅授权本地/static/fixture 源码实现和测试。 | A. 另起 runtime/storage gate；B. 暂停 CR152 等真实数据验证。 | 推荐方案与 CP2/CP3 一致，能推进框架补齐。 | 影响 CP6/CP7 验证模式和 release wording。 |

不授权项：
- credential or `.env` read
- real model training or real data validation
- model registry write / publish / promote / upload / set_current
- feature store / label store / model store / prediction store write
- catalog pointer mutation or Git remote write
- real lake / NAS / provider access
- QMT / MiniQMT / xtquant runtime
- simulation / paper / live / trading runtime
- broker/account/market/order/fill operations
- external framework clone/install/run

用户需决策事项 summary：

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 4 |
| 必须用户决策 | `DQ-CP5-CR152-001`、`DQ-CP5-CR152-002`、`DQ-CP5-CR152-003`、`DQ-CP5-CR152-004` |
| 推荐回复 | `approve` |
| 如果你回复 approve | 表示接受上表 4 项推荐方案，并允许进入本地/static/fixture CP6 implementation。 |
| 不表示授权 | 真实训练、真实数据验证、model registry/store/catalog 写入、runtime、lake/NAS/provider/QMT/broker/credential/external framework/Git remote 操作。 |

CP6 执行关注项：
- S03 默认只读检查 `engine/research_production_contracts.py`；优先在 `engine/training_snapshot_contract.py` / `engine/research_manifest.py` 放置 metadata companion。若确需修改共享 contract 文件，先停止并记录 CP5 / merge-owner 确认。
- S04 修改 `engine/strategy_admission_package.py` 前必须源码检查 `StrategyAdmissionPackage` 能否安全扩展 gate linkage 字段；若安全扩展不明显，停止并 reopen design 或 dispatch meta-se。
- S05 的 factor-research-loop Feature refs 只作为 CP7/CP8 文案对齐参考；若与 CR152 fixture-only / no-real-op 边界冲突，以 CR152 边界为准回退文案。

如果你回复 approve，表示接受上表全部推荐方案，并允许进入本地/static/fixture CP6 implementation。

不表示授权：真实训练、真实数据验证、model registry/store/catalog 写入、runtime、lake/NAS/provider/QMT/broker/credential/external framework/Git remote 操作。

可回复：

approve

修改: <具体修改点>

reject
