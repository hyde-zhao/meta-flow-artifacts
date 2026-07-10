请审查人工门禁 `CP3-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-HLD-REVIEW.md`
自动预检结论: PASS，0 个 blocker。

审批者摘要:
- 本次确认服务的整体目标: 确认 CR161 CP3 的 strategy admission evidence pipeline hardening HLD/ADR，让 admission 能清楚区分 evidence-computable、`typed_unavailable`、`not_applicable_with_reason` 和 `blocked`，并在强制证据缺失时 fail closed。
- 推荐动作: `approve`，接受 contract-first evidence availability overlay、7-object evidence coverage matrix、claim-tier fail-closed policy、CR151/CR154 integration、CR155 blocked negative regression、FU-CR161-001..005 follow-up split 和 no-runtime/no-data authorization boundary。
- approve 后会发生什么: Host Orchestrator 将回填 CP3 人工结果；按当前 CP2 决策，CR161 第一切片保持 design-only，CP4/CP5/CP6 预期为 N/A，进入 CP7 自动设计验证。
- approve 不授权什么: 不授权源代码或测试实现、schema/checker、research-engine trial-lineage instrumentation、FDR/PBO/DSR/OOS/TCA/capacity 计算实现、strategy remediation、simulation/paper/live/trading/runtime、QMT/MiniQMT/xtquant/gateway、new real lake read、real lake write、NAS read/write/sync、provider fetch、credential/env/secret read、broker/order write、catalog/store/registry/model/prediction write、external framework clone/install/run、Git remote write、release 或 publish。
- 不确认会阻塞什么: 阻塞 CR161 CP7 设计验证和 CP8 交付关闭；C1-C4 blocker 仍停留在 CP2 范围批准但 CP3 架构未确认状态。

Context Capsule:
- `process/context/CP3-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-DESIGN-CONTEXT.yaml`
- read_profile: compact
- 全文档读取扩展: `RE-20260709T142420Z0000-cr161cp3discussion`

决策收集覆盖:
- CP3 result、discussion checkpoint、HLD、ADR、handoff 和 dispatch ledger 已扫描。
- 本轮待人工决策项: 6。
- 7 个 evidence objects 已在 HLD §8 显式覆盖: `ExperimentFamilyManifest`、`MultipleTestingEvidence`、`DataSnoopingEvidence`、`OverfitRiskEvidence`、`WalkForwardEvidence`、`EconomicCostEvidence`、`CapacityLiquidityEvidence`。

决策分层:
- 必须用户决策: 6。
- 高风险策略确认: `DQ-CP3-CR161-002` typed_unavailable fail-closed、`DQ-CP3-CR161-004` CR155 negative regression only、`DQ-CP3-CR161-006` no new authorization。
- agent 默认处理: HLD 文档结构、ADR 编号、discussion checkpoint 格式、CP3 result item 划分。
- 仅审计记录: Blueprint standalone artifact waiver、CP4/CP5/CP6 expected N/A、read expansion ledger、status-sync summary 工具偏差修正。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| `DQ-CP3-CR161-001` | architecture | 是否批准 contract-first evidence availability overlay？ | 使用 typed evidence availability overlay，通过 CR151/CR154 refs 和 summaries 集成，不新增 CR161 gate status。 | compute-first implementation；documentation-only checklist；parallel CR161 gate family。 | 推荐方案符合 CP2 授权，最小化范围并让缺失证据可审计；compute-first 需要 CP4/CP5/CP6 和实现授权；parallel gate 会与 CR151/CR154 冲突。 | 决定 CR161 当前切片是否保持 CP3-only design-first。 |
| `DQ-CP3-CR161-002` | architecture | 是否批准 `typed_unavailable` 按 claim tier fail closed？ | 强制证据为 `typed_unavailable` 时阻断 statistical significance、robustness、`paper_candidate` 和 `production_like`；仅允许明确标注的 exploratory limitation。 | warning-only；block all exploratory；manual waiver。 | 推荐方案既真实表达缺失证据，也保留低等级探索性记录；warning-only 会静默放行高风险 claims。 | 高风险 no-overclaim 决策。 |
| `DQ-CP3-CR161-003` | architecture | 是否批准通过 CR151/CR154 集成且不创建 parallel CR161 gate family？ | 扩展 CR151/CR154 evidence refs 和 summaries，CR161 只提供 evidence availability envelope 和 claim-tier policy。 | parallel CR161 gate family；direct package-only fields。 | 推荐方案复用既有 statistical admission 和 reliability gate surface，降低维护成本。 | 影响模块边界和未来 adapter 设计。 |
| `DQ-CP3-CR161-004` | risk_acceptance | 是否批准 CR155 仅作为 blocked negative regression？ | 保持 CR155 `blocked_admission_failed`、`paper_candidate=false`；不重建历史 trial lineage、p-values、PBO/DSR 或 fold metrics。 | 要求历史 C1/C2 computable proof；omit CR155；treat CR155 as remediation target。 | 推荐方案最诚实且防止 rerun consistency 被误读为 admission proof。 | 高风险历史证据边界。 |
| `DQ-CP3-CR161-005` | follow_up_tracking | 是否批准 FU-CR161-001..005 作为 deferred follow-up candidates？ | 将 trial lineage instrumentation、computable statistical evidence、OOS folds、economic cost/impact、capacity/liquidity 分别登记为 follow-up。 | 当前全部实现；合并为一个大 follow-up。 | 推荐方案避免 CP3 scope 膨胀，并保留未来实现路径。 | 影响后续 CR 分解、实现授权和验证计划。 |
| `DQ-CP3-CR161-006` | security | 是否确认不新增 implementation/runtime/data/credential/external/publish authorization？ | 保持 CR161 CP3 design-only；不执行源码/测试实现、真实数据访问、运行时、凭据、外部框架、Git remote 或 publish。 | separate implementation route；separate data/runtime authorization CR。 | 推荐方案与 CP2 授权边界一致；任何实现或真实数据动作都需要新人工门禁。 | 高风险授权边界。 |

如果你回复 approve，表示你接受以上 6 项推荐方案，并允许 CR161 进入 CP7 自动设计验证。

不授权项:
- `approve` 不表示授权以下事项。
- 代码/测试/schema/checker 实现。
- trial-lineage instrumentation。
- FDR/PBO/DSR/OOS/TCA/capacity 计算。
- real lake/NAS/provider/credential/broker/trading/runtime/external framework/Git remote/release/publish。

请只回复以下三个 exact 选项之一：
- `approve`
- `修改: <具体修改点>`
- `reject`
