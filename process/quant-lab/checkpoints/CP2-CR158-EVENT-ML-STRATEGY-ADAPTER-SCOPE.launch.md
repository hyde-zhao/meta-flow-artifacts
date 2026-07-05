请审查：`process/checkpoints/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.md`

自动预检结论：PASS
- CP0 request intake：PASS，CR158 已把 `DF-CR157-001` 与 `DF-CR157-002` 合并为一个正式 CR。
- CP1 use-case completeness：PASS，产品基线、场景和覆盖矩阵已更新。
- CP2 scope precheck：PASS，当前无自动预检 blocker。

Context Capsule Summary：
- capsule 路径：`process/context/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-CONTEXT.yaml`
- capsule 状态：ready
- read_profile：compact
- 默认读取策略：capsule-first；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。

审批者摘要：
- 本次确认服务的整体目标：确认 CR158 是否以一个统一 event + ML strategy adapter CR 继续，先进入 HLD / ADR 和 Story 设计批次，再进入本地 fixture/static 实现与验证。
- 推荐动作：`approve`，接受统一 event + ML adapter scope、no-runtime/no-publish 授权边界、CP2 -> CP3 -> CP5 -> CP6/CP7 的门禁顺序。
- approve 后会发生什么：CR158 进入 `solution-design`；准备 HLD / ADR；后续 Story decomposition 和实现仍受 CP3 / CP5 约束。
- approve 不授权什么：不授权真实 feed、真实模型训练、外部模型服务、registry promotion、real lake/NAS/provider/credential access、catalog/store/registry/prediction write、QMT/gateway/runtime、simulation/paper/live/trading/broker、external framework run、Git remote write、publish 或 production deployment。
- 不确认会阻塞什么：阻塞 CR158 HLD、Story split、LLD、实现和验证；`DF-CR157-001/002` 保持 deferred/promoted-but-unapproved。

决策收集覆盖：已扫描 11 类来源，发现候选问题 25 个，纳入待决策 3 个；N/A / 缺失来源 3 类，原因见 checkpoint 的 `Decision Collection Coverage`。

决策分层：
- 必须用户决策：2
- 高风险策略确认：1
- agent 默认处理：16
- 仅审计记录：6

本轮待人工决策项：3

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| `DQ-CP2-CR158-UNIFIED-SCOPE` | scope | 是否把 event adapter 与 ML adapter 合并为一个统一 CR？ | 合并为一个 CR，shared core + type-specific extensions。 | 拆为两个 CR；只做 event；只做 ML。 | 推荐方案减少重复 HLD/CP5 和 schema 分叉；备选降低单次范围但增加门禁成本和残留债务。 | 风险是 shared core 过度泛化；CP3 必须明确 extension 边界。 |
| `DQ-CP2-CR158-NO-RUNTIME` | security | CP2 approve 是否授权真实 feed、训练、runtime、data access 或 publish？ | 不授权，只允许 local/static/fixture 设计、实现和验证。 | 授权只读真实数据；授权真实 feed/training/runtime。 | 推荐方案与 CR157 边界一致且风险最低；备选触发 runtime-high-risk 和额外授权。 | 误写 runtime-ready 会造成 release overclaim 和安全边界破坏。 |
| `DQ-CP2-CR158-GATE-SEQUENCE` | implementation | CP2 approve 后是否允许直接 Story/LLD/实现？ | 不允许；CP2 后进入 CP3 HLD/ADR，CP5 批准后才实现。 | 直接 Story split；直接实现。 | 推荐方案较慢但保留架构和设计证据；备选会绕过关键门禁。 | 直接实现会破坏 adapter contract、file ownership 和 no-runtime 验证设计。 |

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
回复 `reject` 表示拒绝当前 CP2 推荐方案并回退范围处理。
