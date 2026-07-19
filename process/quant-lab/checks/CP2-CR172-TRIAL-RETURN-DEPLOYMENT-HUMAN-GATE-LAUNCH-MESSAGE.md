请审查人工门禁 `CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR172-TRIAL-RETURN-DEPLOYMENT-REQUIREMENTS-BASELINE.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 DQ-009～015 推荐方案与三个强制边界，只解锁 CP3 solution design。
- approve 不授权什么: 不授权实现、目录创建、真实 lake/NAS 访问、sync/pull/materialize、multi-trial runtime、trial-return/R 生成、SignalBatch 传输、迁移、publish、Git remote write、live 或交易操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 7
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 7
blocking / high-risk 决策摘要: DQ-CR172-009, DQ-CR172-010, DQ-CR172-011, DQ-CR172-012, DQ-CR172-013, DQ-CR172-014, DQ-CR172-015

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CR172-009 | scope | trial-return source 与对象 | future native multi-trial per-trial portfolio return；source 缺失/错型 fail-closed | 外部预计算 import contract；或保持 absent | native 方案 lineage 最完整，但需要后续 instrumentation/runtime gate；import 更快但 provenance 较弱；absent 最保守。 | 影响 trial-return schema、seal/hash/URI 和 CR-163 lineage 引用边界；source absent 会阻断 C1 正向结果。 |
| DQ-CR172-010 | architecture | 单一 canonical、NAS replica 与 stable identity 如何落地 | 研究机本地 active canonical；NAS 仅 verified replica/backup/distribution；执行机按需 pull 到临时目录并校验 release/manifest/hash 后原子物化本地 immutable cache；logical URI + content hash | NAS 不可用时保留研究机 canonical 并阻塞分发；或 storage blocked | 推荐方案保持研究主权、运行时隔离和可追溯性；NAS 故障只影响分发。备选更保守但不可跨机消费。 | replica 陈旧、partial 或 hash 不符时可能产生错误消费；运行时直读 NAS 会扩大故障域。 |
| DQ-CR172-011 | runtime_authorization | multi-trial 与跨机数据动作如何授权 | `data_lake_read`、`multi_trial_runtime_and_workspace_write`、`trial_return_generation`、`empirical_R_computation`、`nas_replica_sync`、`execution_pull_verify_materialize` 六类动作 6/6 分离 | 合并 runtime+generation；或全不授权 | 分离方案 blast radius 最小且可逐项撤销，但门禁较多；合并较简洁却削弱审计；全不授权只能 fixture。 | 影响研究机→NAS→执行机完整链路；任何授权合并都可能把数据读、runtime 与写入风险混为一体。 |
| DQ-CR172-012 | security | 四组件 ownership/data flow | 研究机生产并持有 canonical、NAS 仅分区 replica/backup/distribution、GitHub metadata-only、执行机仅拉取 approved package 并使用本地 immutable cache | 取消 NAS 分发；或执行机离线预置已验证 package | 推荐方案兼顾最小泄漏与可分发性；离线预置更安全但运维效率更低。 | 扩大 GitHub 或执行机数据面会引入数据泄漏和 runtime 依赖；执行机直读研究机目录被禁止。 |
| DQ-CR172-013 | scope | 新旧运行路径 | 新 run 使用 `multifactor-strategy-research`；legacy 只读不迁移 | 继续旧路径；或一次性 migration/rewrite | 新路径语义清晰且保留历史；旧路径继续误导；迁移可统一命名但身份与回滚成本高。 | 影响 runner 默认值、artifact identity、历史可读性和回滚；CP3 提前切换会形成双默认路径。 |
| DQ-CR172-014 | risk_acceptance | empirical R 缺证据/授权时处理 | declared-exact/empirical 分类；重开 PATH-C/A 时显式三选一：完成 `FU-CR173-001`、拆 future activation CR、或 DQ-003 typed_unavailable 降级。v2 不阻止降级设计，但阻断 positive empirical effective count / `c1_computable=true` | 永久 declared-exact-only；或保持所有 empirical 输入 unavailable | 推荐方案保留方法演进与诚实降级；硬阻断所有 PATH-C/A 会违反 DQ-003，静默复用 v1 则越过有效域。 | 错把 empirical 重标为 declared_exact 会产生不可审计正向结果；未完成 v2 时不得声明 effective count available。 |
| DQ-CR172-015 | architecture | 信号在哪里生成、PATH-I 冻结到什么粒度 | 默认执行机本地生成；低频/EOD 只冻结 optional immutable batch 与精确 8 字段：schema_version、batch_id、strategy_id、strategy_package_hash、content_sha256、signature/key_id、validity window、sequence_no；intraday 独立 CR。物理路径、七级状态机、ack/idempotency/replay 全部 deferred | 全部执行机本地生成且不声明 batch；或把低频 detailed exchange 提升为独立后续 CR | 推荐方案保留最小扩展边界且防止 PATH-I 跨入 trading owner/Stage 4/5；本地-only 最安全；独立 CR 可完整设计但增加审批。 | detailed transport、ack/replay 或 intraday 混入会扩大 owner、运行授权和交易风险；PATH-I signal Story 必须为 0。 |

三个强制边界：CP3 只冻结设计；DQ-015 只冻结默认本地生成与精确 8 字段边界，详细 exchange/intraday 均 deferred；`FU-CR173-001` 只阻断 positive empirical result / `c1_computable=true`，不阻断 DQ-003 typed-unavailable 降级设计。

如果你回复 approve，表示你接受以上 7 项推荐方案和三个强制边界，不表示授权任何实现或真实操作。
不授权项: 实现、目录创建、真实 lake/NAS 访问、sync/pull/materialize、multi-trial runtime、trial-return/R 生成、SignalBatch 传输、迁移、publish、Git remote write、live / 交易类操作。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
