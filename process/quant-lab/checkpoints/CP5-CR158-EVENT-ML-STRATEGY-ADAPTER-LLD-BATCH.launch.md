请审查：`process/checkpoints/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-BATCH.md`

自动预检结论：PASS
- CP5 LLD implementability：PASS，S01-S05 full LLD、S06 technical-note、clarification queue、merge order 和 authorization boundary 均已就绪。

Context Capsule Summary：
- capsule 路径：`process/context/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-CONTEXT.yaml`
- capsule 状态：ready
- read_profile：compact
- 默认读取策略：capsule-first；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。

审批者摘要：
- 本次确认服务的整体目标：确认 CR158 CP5 Story 级设计证据是否可作为 CP6 local/static/fixture 实现依据。
- 推荐动作：`approve`，接受 S01-S05 full LLD、S06 technical-note、受控 merge order 和 no-runtime 边界。
- approve 后会发生什么：CR158 可进入 `story-execution`；Story 可推进为 `lld-approved` / `dev-ready`；后续只允许 local/static/fixture source/test implementation。
- approve 不授权什么：不授权真实 feed、真实训练、registry、provider/lake/NAS/credential、QMT/runtime、simulation/paper/live/trading/broker、publish/deployment、external framework 或 Git remote write。
- 不确认会阻塞什么：阻塞 CR158 CP6 实现、CP7 验证和 CP8 发布就绪。

决策收集覆盖：已扫描 8 类来源，发现候选问题 17 个，纳入待决策 4 个；N/A / 缺失来源 1 类，原因见 checkpoint 的 `Decision Collection Coverage`。

决策分层：
- 必须用户决策：2
- 高风险策略确认：1
- agent 默认处理：12
- 仅审计记录：9

本轮待人工决策项：4

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| `DQ-CP5-CR158-001` | implementation | 是否批准 S01-S05 full LLD + S06 technical-note 作为 CP6 实现依据？ | Approve current CR158 CP5 design batch. | 修改指定 Story LLD；reject batch。 | 推荐方案可进入 CP6；修改提升局部准确性但延迟；reject 停止实现。 | 设计成为 CP6 契约；实现偏离需 design delta。 |
| `DQ-CP5-CR158-002` | implementation | Shared 文件如何合并？ | 串行/受控实现：S01 -> S02/S03 -> S04/S05 -> S06。 | 并行拆 section；或拆模块后并行。 | 推荐方案冲突最低；备选更快但共享合同风险更高。 | 影响 CP6 排期、文件 owner 和回修成本。 |
| `DQ-CP5-CR158-003` | runtime_authorization | CP5 approve 是否授权真实 feed/training/runtime/data/registry/publish/trading？ | 不授权；仅 local/static/fixture source/test implementation。 | 单独 runtime authorization CR；或回退 CP2/CP3 扩 scope。 | 推荐方案安全且符合 CP2/CP3；备选治理成本高。 | 防止 runtime/model-registry/trading readiness overclaim。 |
| `DQ-CP5-CR158-004` | architecture | 是否继续采用 thin shared core + typed extensions？ | 继续当前 shared core + event/ML typed extensions。 | Two independent adapters；fat common schema。 | 推荐方案保留统一证据和 guard；备选要么重复治理要么污染字段。 | 影响 schema owner、Story 依赖和未来维护。 |

不授权项：
- real event feed or live listener
- real ML model training, external model service call, model registry write or promotion
- real lake/NAS/provider access or credential/env/session read
- catalog/store/registry/model/prediction write
- QMT/MiniQMT/xtquant/gateway runtime
- simulation/paper/live/trading runtime or broker operation
- external framework clone/install/run
- Git remote write
- true release execution, publish, production deployment or runtime readiness claim

如果你回复 approve，表示接受上表全部推荐方案；不表示授权真实运行、凭据读取、外部写入、publish、production deployment 或交易相关操作。
回复 `修改: <具体修改点>` 表示要求按指定 DQ 或内容修改后重新发起确认。
回复 `reject` 表示拒绝当前 CP5 推荐方案并回退设计处理。
