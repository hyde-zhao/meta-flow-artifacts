请审查人工门禁 `CP2-CR045-RECOVERY-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR045-RECOVERY-BASELINE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR045-RECOVERY-BASELINE.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 1
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 1
blocking / high-risk 决策摘要: CP2-CR045-DQ-01

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP2-CR045-DQ-01 | scope | 是否接受 CR-045 当前工程实现作为既有事实，并从当前时间开始补齐 CP2/CP5 恢复确认、独立 CP7 和 CP8，而不伪造这些门禁早于 CP6？ | 接受透明恢复；保留历史偏差，继续验证和关闭。 | 回退并撤销现有实现后重新按 CP2→CP5→CP6 执行；或保持 CR active 暂不推进。 | 推荐方案避免无价值重做且保持审计真实性；重做方案流程最整齐但成本高并会丢失已验证实现；暂缓方案不改变仓库但继续占用 active CR。 | 推荐方案留下“实施先于门禁”的已披露过程 finding，但功能与最终验证仍可可信关闭；重做增加回归和合并风险。 |

如果你回复 approve，表示你接受以上 1 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
