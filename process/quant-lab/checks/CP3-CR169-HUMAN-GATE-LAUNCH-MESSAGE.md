请审查人工门禁 `CP3-CR169-CAPACITY-LIQUIDITY-ADV-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR169-CAPACITY-LIQUIDITY-ADV-HLD-REVIEW.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR169-CAPACITY-LIQUIDITY-ADV-HLD-REVIEW.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 5
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 5
blocking / high-risk 决策摘要: DQ-CP3-CR169-METHOD, DQ-CP3-CR169-HEADER, DQ-CP3-CR169-JOINT, DQ-CP3-CR169-ALPHA, DQ-CP3-CR169-TRANSITION

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR169-METHOD | architecture | C4 v1 是可复算的 explicit static proxy，还是只冻结 schema？ | static proxy：显式 synthetic/static basis、模型、limitations，生成 `capacity_liquidity@v1`。 | schema-only；calculator / adapter 留 FU007。 | 推荐验证计算和 consumer contract；备选更小但 C4 evidence 缺少可消费证明。 | 决定 12 P0、hash 与 fixture 测试范围；两者均不授权真实数据。 |
| DQ-CP3-CR169-HEADER | architecture | 如何同时保证 C3/C4 同 context join 与 multi-strategy method-hash 语义？ | minimal header exact match；identity 仅入 envelope binding，component semantic hash 只覆盖计算 body。 | 将 identity 纳入 component hash；或延后 shared header。 | 推荐分离 subject binding 与方法语义，且强制 join 前比对；备选降低概念复杂度但削弱兼容 fixture 的真实意义。 | 决定 C4 hash 域、correlation mismatch 行为与未来 C4 互操作。 |
| DQ-CP3-CR169-JOINT | security | C4 完成后是否由 CR169 验证 Gate4 C3+C4 消费兼容？ | CR169 local strict adapter：exact 7-key、拒绝 N/A/extra key、public canonical read-only、local postcondition；仅输出 fixture outcome。 | C4 component-only，consumer 验证留 FU007。 | 推荐为 C4 交付留下 consumer proof，且不改 canonical/aggregate；备选最隔离但缺少消费侧证据。 | 最高风险：错误映射可能把 fixture 当 aggregate；推荐方案以 0 aggregate / 0 scalable claim 限制。 |
| DQ-CP3-CR169-ALPHA | scope | alpha-decay 是 C4 v1 的一部分还是独立问题？ | C4 v1 `alpha_decay_calculator=0`；保留独立 / C2-adjacent follow-up。 | 在 C4 v1 加 static alpha-decay calculator。 | 推荐不预占 C2/OOS 预测衰减语义，且 Gate4 不消费 alpha field；备选减少未来集成但增加方法/测试边界。 | 决定输入 contract 和 Story 范围；不影响 C4 三 refs。 |
| DQ-CP3-CR169-TRANSITION | follow_up_tracking | Stage2 contract、Stage3 entry 与 FU007 proposal 如何声明？ | stage3_entry_ready=false；CP8 做 7/7 exit result；FU007a/b 仅 future tracking，不创建 / 不启动。 | CR169 将 `stage2_complete` 当作 Stage3 ready，或立即启动 FU007a。 | 推荐阻止错误阶段升级，保留后续排程空间；备选简化叙事但违反现有授权与风险边界。 | 直接影响 claim ceiling、Stage3 authorization、canonical global remediation 排程。 |

如果你回复 approve，表示你接受以上 5 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
