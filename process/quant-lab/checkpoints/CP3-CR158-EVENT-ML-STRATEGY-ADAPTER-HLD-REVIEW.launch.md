请审查：`process/checkpoints/CP3-CR158-EVENT-ML-STRATEGY-ADAPTER-HLD-REVIEW.md`

自动预检结论：PASS
- CP3 HLD consistency：PASS，Blueprint、HLD、ADR、traceability、scenario simulation 和 discussion checkpoint 均已就绪。

Context Capsule Summary：
- capsule 路径：`process/context/CP3-CR158-EVENT-ML-STRATEGY-ADAPTER-DESIGN-CONTEXT.yaml`
- capsule 状态：ready
- read_profile：compact
- 默认读取策略：capsule-first；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。

审批者摘要：
- 本次确认服务的整体目标：确认 CR158 CP3 架构方案是否可作为 CP4 Story 拆解和 CP5 设计证据输入。
- 推荐动作：`approve`，接受 thin shared core + typed event/ML extensions、refs-only evidence、adapter counter report + FEAT-07 no-runtime guard。
- approve 后会发生什么：CR158 进入 `story-planning`；允许准备 Story split、FEATURE-DESIGN-MATRIX、CP4/CP5 设计队列。
- approve 不授权什么：不授权 source/test implementation、LLD 自动通过、真实 event feed、真实 ML training、registry write、provider/lake/NAS/credential、runtime、trading、publish、deployment 或 Git remote write。
- 不确认会阻塞什么：阻塞 CR158 Story decomposition、FEATURE-DESIGN-MATRIX、LLD、implementation 和 verification。

决策收集覆盖：已扫描 8 类来源，发现候选问题 21 个，纳入待决策 2 个；N/A / 缺失来源 1 类，原因见 checkpoint 的 `Decision Collection Coverage`。

决策分层：
- 必须用户决策：1
- 高风险策略确认：1
- agent 默认处理：10
- 仅审计记录：8

本轮待人工决策项：2

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| `DQ-CP3-CR158-001` | architecture | 是否确认 thin shared core + typed event/ML extensions 作为 CP4/CP5 输入？ | Confirm thin shared core + typed extensions。 | Fat common schema；two independent adapters。 | 推荐方案统一消费且避免字段污染；备选要么污染字段，要么重复门禁。 | 影响 Story split、schema owner、CP5 LLD 和 CP7 tests。 |
| `DQ-CP3-CR158-002` | security | CP3 approval 是否继续不授权真实 runtime/data/registry/publish？ | Confirm fixture/static only and no-runtime guard。 | 授权只读真实数据；另起 runtime authorization CR。 | 推荐方案与 CP2 一致且风险最低；备选需要更重授权。 | 防止 HLD 被误读为 runtime-ready 或 registry-ready。 |

不授权项：
- source/test implementation before CP5 approval
- real event feed or live listener
- real ML model training, external model service call, model registry write or promotion
- real lake/NAS/provider access or credential/env/session read
- catalog/store/registry/model/prediction write
- QMT/MiniQMT/xtquant/gateway runtime
- simulation/paper/live/trading runtime or broker operation
- external framework clone/install/run
- Git remote write
- true release execution, publish, production deployment or runtime readiness claim

如果你回复 approve，表示接受上表全部推荐方案；不表示授权真实运行、凭据读取、外部写入、publish、production deployment、实现或交易相关操作。
回复 `修改: <具体修改点>` 表示要求按指定 DQ 或内容修改后重新发起确认。
回复 `reject` 表示拒绝当前 CP3 推荐方案并回退架构处理。
