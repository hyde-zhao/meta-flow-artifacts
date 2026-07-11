请审查人工门禁 `CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-HLD-REVIEW.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-HLD-REVIEW.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 4
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 4
blocking / high-risk 决策摘要: DQ-CP3-CR163-001, DQ-CP3-CR163-002, DQ-CP3-CR163-003, DQ-CP3-CR163-004

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR163-001 | architecture | 是否批准 `FamilyLineageSession` lifecycle façade，内部使用 typed append-only event commands？ | 批准 session + commands；producer 易接入，底层事件仍完整可审计。 | 纯 command/fold API；或 close-time snapshot session。 | 推荐兼顾 ergonomics 与 audit；纯 commands 更显式但增加 producer 复杂度；snapshot 简单但隐藏中途失败并丢事件事实。 | 决定 public producer API、S01/S02 contract 与两条 chain 接入方式。 |
| DQ-CP3-CR163-002 | architecture | 是否批准 repo-local canonical JSON/JSONL、SHA-256 immutable version seal 和 append-only full supersession chain？ | 批准 create-only spec + append-only events + immutable manifest/validation；full chain 为真相。 | 单 snapshot JSON；SQLite；外部 registry/MLflow。 | 推荐最小、确定、易 fixture/审计；snapshot 丢历史；DB/registry 并发更强但引入迁移/服务/依赖。 | 决定 storage、hash、tamper、recovery、rollback 和 S02 范围。 |
| DQ-CP3-CR163-003 | architecture | 是否批准 manual legacy count 仅作 reconciliation，并只复用现有 CR151/CR154/admission consumers？ | 批准：无 native sealed ref=`typed_unavailable`；mismatch/invalid=`blocked`；consumer 只取 validator projection；不建新 gate。 | 接受手填 count 产生 present；完全忽略 manual count；创建 CR163 平行 gate。 | 推荐消除后验 bypass 并保持现有 policy 单一；接受 manual present 违背核心目标；忽略会失去迁移诊断；新 gate 重复政策。 | 决定 S04 接口、legacy compatibility、fail-closed 和 admission 状态合并。 |
| DQ-CP3-CR163-004 | security | 是否批准冻结的六对象、五 Story CP4 输入与 design-only 授权边界？ | 批准 HLD/ADR 并进入 CP4；S01-S05 不变，S03 4/4 mappings；实现仍等 CP5。 | 请求架构修改；暂停 CR163。 | 推荐形成完整 CP4/CP5 输入且不扩大权限；修改可降低后续返工但需重跑 CP3；暂停最安全但阻塞价值。 | 决定是否正式拆 Story/DAG/owner/LLD policy；不会授权源码实现。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
