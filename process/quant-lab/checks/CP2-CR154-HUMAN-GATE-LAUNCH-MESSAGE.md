# CP2 CR154 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE.md`

自动预检结论：PASS

上下文胶囊：`process/context/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE-CONTEXT.yaml`（read_profile=compact）

Context Capsule Summary:

- capsule 路径：`process/context/CP2-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-SCOPE-CONTEXT.yaml`
- capsule 状态：ready
- read_profile：compact

审批者摘要：

- 本次确认服务的整体目标：确认 CR154 Cross-Strategy Production Reliability Gates 的 CP2 范围基线。
- 推荐动作：`approve`，接受 7 项推荐方案。
- approve 后会发生什么：CR154 进入 CP3 设计，并携带 `CP3-DC-CR154-001` 统计可靠性工件约束；不会进入 Story、LLD、实现或运行时。
- approve 不授权什么：不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog/event-store/model-registry 写入；不授权 live event listener、真实事件 feed、真实下单、真实数据验证、真实 reconciliation、真实发布执行或源码实现。
- 不确认会阻塞什么：阻塞 CR154 的 CP3 设计、Story 拆解、LLD、实现、验证和 release readiness。

本轮待人工决策项：7

决策收集覆盖：已扫描 11 个来源，发现候选问题 49 个，纳入待决策 7 个；已补 CR154 专属 SGQ 证据 `process/discussions/CP2-CR154-SCENARIO-DISCUSSION-LOG.md` / `process/checks/CP2-CR154-DISCUSSION-CHECKPOINT.json`；N/A / 缺失来源 1 个：`docs/product/USE-CASES.md` 缺失，已使用 `process/USE-CASES.md` 中已确认 UC-58/59/60 作为 fallback。

决策分层：

- 必须用户决策：7
- 高风险策略确认：1
- agent 默认处理：3
- 仅审计记录：6

如果你回复 approve，表示你接受以下 7 项推荐方案。

不表示授权：真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog/event-store/model-registry 写入、live event listener、真实事件 feed、真实下单、真实数据验证、真实 reconciliation、真实发布执行或源码实现。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR154-BACKTEST-TRAP-GATE | architecture | 是否建立三类策略共享 backtest trap gate？ | 覆盖 lookahead、survivorship、data snooping、regime overfit、cost underestimation；CP3 强制拆出 multiple-testing / FDR-BH / WRC-SPA / PBO-DSR 等统计工件，默认 fail-closed / needs-review。 | 仅文档；各策略自建；延后到 runtime。 | 推荐方案可审计且统一；备选会不可验证、重复或继续保留 blocker。 | 影响 research admission、backtest validation、statistical reliability artifacts、release blocking。 |
| DQ-CP2-CR154-CV-GOVERNANCE | architecture | Walk-forward / OOS / purged-embargo 是否统一治理？ | 建立共享 CV governance，统一 rolling / OOS / purged / embargo status、refs、fail reasons 和 fixture semantics。 | 复用 ML CV 默认；各自实现；本轮不处理。 | 推荐方案避免误用 ML 假设和三套框架。 | 影响 CR151/152/153 admission 关系。 |
| DQ-CP2-CR154-SURVIVORSHIP-UNIVERSE-GATE | architecture | Survivorship-free / PIT universe 是否进入 first wave？ | 将 PIT universe / survivorship-free universe 扩展为共享 gate contract；真实 universe 数据仍不授权。 | 真实构建；只保留 slot；只写文档。 | 推荐方案机器可见且不触发真实数据。 | 影响样本有效性和 no-overclaim wording。 |
| DQ-CP2-CR154-CAPACITY-IMPACT-GATE | implementation | Capacity / market impact / liquidity sizing 如何进入 first wave？ | 定义 ADV participation、capacity dollars、impact model family、cost-underestimation status 和 liquidity sizing refs；只验证 contract semantics。 | 真实 TCA；只沿用 commission/tax；完全延后。 | 推荐方案补齐 blocker 且不触发真实交易数据。 | 影响 backtest credibility 和 AUM scaling。 |
| DQ-CP2-CR154-REGIME-ATTRIBUTION-RECONCILIATION-SLOTS | implementation | Regime / attribution / reconciliation 做到什么程度？ | 定义结构化 slots、status、refs 和 `n/a-with-reason`；不执行真实 runtime/broker reconciliation。 | 真实 reconciliation；不记录；自由文本。 | 推荐方案保留接口且可静态验证。 | 影响 future paper/live readiness，但本轮不声明 readiness。 |
| DQ-CP2-CR154-ADMISSION-GATE-DEFAULT-POLICY | architecture | CR151/CR152/CR153 gates 何时 default-required 或 release-blocking？ | 按 strategy class、release profile、risk level 和 evidence completeness 定义 default-required / release-blocking / opt-in exception。 | 立即全量强制；全部 opt-in；调用方自由选择。 | 推荐方案兼顾历史兼容和可靠性。 | 影响 completion map、admission package 和 release gate。 |
| DQ-CP2-CR154-NO-RUNTIME-NO-REAL-DATA-BOUNDARY | security | CP2 approve 是否授权真实 runtime、真实数据、broker、feed、reconciliation 或 publish？ | 不授权。CR154 first wave 保持 local/static/fixture-only，真实外部系统和真实交易/发布动作必须另开人工 gate。 | 同时做 lake readonly；同时做 paper/live reconciliation；同时连接 provider/feed。 | 推荐方案符合当前边界和最小风险。 | 防止 CR154 变 runtime / data / trading CR。 |

不授权项：

- real lake read/write、NAS、provider、credential / `.env` read
- QMT / MiniQMT / xtquant / gateway runtime、simulation、paper、live、trading
- live event listener、broker read/write/submit/cancel、真实账户查询
- external framework clone / install / run、Git remote write
- catalog pointer mutation、feature/label/event/model/prediction store write
- real event feed execution、real order flow、real data validation、real reconciliation
- true release execution、production deployment、publish

CP3 硬性设计约束：

- `CP3-DC-CR154-001`：`DQ-CP2-CR154-BACKTEST-TRAP-GATE` 不得只落成 trap name / status 枚举。CP3 HLD / ADR 必须定义可机器验证的 statistical reliability artifacts，至少包含 multiple-testing correction refs、FDR/BH refs、White Reality Check / Hansen SPA refs、PBO / CSCV refs、DSR / Sharpe 或 IC deflation refs、trial count / effective trials、OOS split refs、purge / embargo refs、survivorship audit refs、impact / capacity refs、blocked claims 和 release-blocking reason；不适用字段必须写 `n/a-with-reason`。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

可直接回复以下任一整行：

```text
approve
修改: <具体修改点>
reject
```
