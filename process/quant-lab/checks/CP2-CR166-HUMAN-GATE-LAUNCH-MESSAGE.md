请审查人工门禁 `CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-SCOPE`。

checklist 路径: `process/checkpoints/CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-SCOPE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-SCOPE.md` 的结果为准。

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
blocking / high-risk 决策摘要: DQ-CP2-CR166-001, DQ-CP2-CR166-002, DQ-CP2-CR166-003, DQ-CP2-CR166-004

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR166-001 | scope | 是否批准 CR166 为 fixture/static C2 foundation，并冻结 Stage2 complete / Stage3 not-started / real-evidence unavailable ceiling？ | 批准；只接收显式 fixture/static folds，不连接生产数据，不把 producer foundation 解释为真实 evidence availability。 | A. 暂停 C2，保持 typed_unavailable；B. 扩为真实数据 Stage3（需拒绝本 gate 并另起授权 CR）。 | 推荐方案先补齐真实架构缺口且不扩大数据风险；暂停无当前价值；混入 Stage3 会改变权限、数据、验证和回滚边界。 | 决定 HLD 的输入/输出、CP8 声明与是否存在过度准入 claim。 |
| DQ-CP2-CR166-002 | risk_acceptance | 是否批准 8 类 P0 fail-closed、daily+ML 2 个 P0 fixture 族和 12 项 QAC？ | 批准；缺 fold、时间逆序、purge、embargo、metric、lineage、unauthorized ref、hash 8/8；10 reruns→1 hash；consumer=3/3。 | A. 把 unauthorized ref/hash 降 P1；B. 缩为 daily-only；C. 增加 event 为 P0。 | 推荐方案把零解引用与确定性保留为 foundation 基础，同时不要求深度 runtime resolver；降级会削弱审计性，扩大 event 会引入未冻结语义。 | 决定 CP3 test contract、CP7 required evidence 和完成定义。 |
| DQ-CP2-CR166-003 | architecture | 是否批准 versioned typed component 扩展与 event P1 applicability 边界？ | 批准 stable header + registered components；C3/C4 仅预留、calculator=0；event 未冻结时显式 N/A。 | A. C3+C4 同 CR 实现；B. 不预留扩展；C. 交付 event 空壳 producer。 | 推荐方案避免 C3/C4 未来破坏兼容，又不制造虚假 event 覆盖；扩大实现重复数据合同，取消扩展会导致后续迁移。 | 决定 envelope 演进、unknown-component fail-closed 和未来 CR 耦合。 |
| DQ-CP2-CR166-004 | security | 是否批准 architecture-major route 与 deny-default：CP2 后只进入 CP3，CP5 前不实现，继续 no-subagent inline？ | 批准 CP3→CP4→CP5→CP6→CP7→CP8；当前只授权本地设计，所有外部/生产数据/runtime/write 操作为 0。 | A. 暂停在 CP2；B. 允许 CP3 后直接实现；C. 启用子 agent。 | 推荐方案保持关键门禁和用户既有 no-subagent 要求；绕过 CP5 会缺少全量设计证据；暂停最保守但阻断价值。 | 决定下一步能否生成 HLD/ADR；不授权代码实现或任何高风险操作。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
