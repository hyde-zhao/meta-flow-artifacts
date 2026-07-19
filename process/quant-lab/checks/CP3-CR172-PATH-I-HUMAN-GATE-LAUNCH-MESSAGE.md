请审查人工门禁 `CP3-CR172-PATH-I-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR172-PATH-I-HLD-REVIEW.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR172-PATH-I-HLD-REVIEW.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: approve 4 项推荐，确认 ARCH-A、严格 seal/replica/materialization、六动作六判定点以及 empirical/SignalBatch 边界。
- approve 后会发生什么: 自动进入 story-planning，完成 Feature/Story/DAG 与 CP4 自动预检后，在 CP5 全量设计证据人工门禁停下。
- approve 不授权什么: 不授权代码实现、目录创建、runner 默认值修改、真实 lake/NAS、multi-trial、trial-return/R、信号传输、迁移、交易、publish/deploy 或 Git remote write。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 4
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 4
blocking / high-risk 决策摘要: CP3-DQ-CR172-I-01, CP3-DQ-CR172-I-02, CP3-DQ-CR172-I-03, CP3-DQ-CR172-I-04

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP3-DQ-CR172-I-01 | architecture | 是否确认 native sealed artifact pipeline 为 PATH-I 目标态，并保留 import/absent 两级回退？ | `ARCH-A`：future native multi-trial producer 输出 per-trial payload，经 research-local validate/manifest/seal 成为 active canonical。 | `ARCH-B`：external import 先进入 quarantine，需独立 provenance contract；`ARCH-C`：source absent，保持 `typed_unavailable`。 | ARCH-A lineage 最完整、owner/identity 最清晰，但后续插桩和 fixture 设计量最大；B 可绕过 native producer 缺口但增加外部 provenance/security 面；C 风险最低但没有 positive empirical 价值。 | 影响 source owner、future Feature/Story、schema、lineage 与 C1 上游。主要风险是把合同误写成 producer 已实现，或让错误对象冒充 trial-return。 |
| CP3-DQ-CR172-I-02 | architecture | 是否确认 logical URI+hash、严格 payload→manifest→seal→local selection→replica verify→materialize verify 顺序和 pointer-only rollback？ | stable logical URI + payload SHA-256；research-local 唯一 canonical；NAS verified replica；execution local immutable cache；所有 pointer 仅在本层验证完成后原子推进。 | research-local only、distribution/materialization blocked。NAS runtime canonical 或 direct-NAS read 不是备选。 | 推荐方案提供跨主机稳定身份、故障隔离和可回滚性，代价是 manifest/seal/receipt/staging 合同较多；保守备选减少跨机价值但不扩大共享盘故障域。 | 主要风险是 stale/partial/hash mismatch 被消费、绝对路径进入 identity、NAS 被提升为 canonical 或回滚改写 sealed bytes。 |
| CP3-DQ-CR172-I-03 | security | 是否确认六类真实动作必须使用六个独立 authorization envelope/判定点，partial approval 不产生权限并集，mid-operation revoke 不推进 pointer？ | `data_lake_read`、`multi_trial_runtime_and_workspace_write`、`trial_return_generation`、`empirical_R_computation`、`nas_replica_sync`、`execution_pull_verify_materialize` 各自 owner/scope/hash/path/expiry/revoke/evidence；当前全 deny。 | 永久 fixture-only；不采用 runtime+generation 或 sync+pull 粗粒度合并。 | 推荐 blast radius、撤销和事故归因最小，但门禁与测试数量更多；全 deny 最安全但无法进入真实链路。 | 主要风险是一次批准隐含其他动作、撤销后继续 commit pointer、partial staging 被分发/运行。 |
| CP3-DQ-CR172-I-04 | risk_acceptance | 是否确认 empirical 四态/FU-CR173-001 条件式前置，同时把 SignalBatch 严格限制为 8-slot boundary？ | R 显式为 `declared_exact/empirical/typed_unavailable/BLOCKED`；DQ-003 降级可设计，positive available count/C1 true 必须先完成 FU-CR173-001；signal 默认 execution-local，只冻结 exact `8/8` slots。 | declared-exact-only / permanent typed_unavailable；或完全 local-signal-only、不保留 batch boundary。 | 推荐保留演进路线且防 sampling-error overclaim，代价是 claim guard/provenance 字段多；备选更保守但降低后续价值。 | 主要风险是 declared-exact 静默重标 empirical、完整性冲突被吞成 unavailable、8-slot boundary 膨胀为 mailbox/ack/replay/intraday 实现。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权代码实现、目录创建、runner 默认值修改、真实 lake/NAS、multi-trial、trial-return/R、signal generation/transport、migration、trading、publish/deploy 或 Git remote write；六类真实动作继续 `0/6`。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
