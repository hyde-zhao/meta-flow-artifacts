请审查人工门禁 `CP8-CR034-DELIVERY-REVIEW`。

checklist 路径: `process/checkpoints/CP8-CR034-DELIVERY-REVIEW.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP8-CR034-DELIVERY-REVIEW.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 2
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 2
blocking / high-risk 决策摘要: DQ-034-04, DQ-034-05

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-034-04 | follow_up_tracking | CP7 发现的 pre-existing test_install_mapping::test_ptm_te_skills_consistent 失败（CR-033 遗留：PTM_TE_SKILLS 加 case-execution 第5 skill 但测试期望 4 skills 未更新）如何处理 | 转 follow-up 台账（CR-034-FOLLOW-UP-TRACKING），不在本 CR 修复（范围控制） | 本 CR 顺手修复（1 行改测试期望 4->5） | 推荐转 follow-up 保持 CR-034 范围纯文档；备选 1 行修复但扩大范围至测试 | 无阻塞；风险：测试失败持续存在但不影响 CR-034 交付 |
| DQ-034-05 | follow_up_tracking | CR-033 follow-up 台账 FU-01/02/03 是否保持独立 | 保持独立（FU-01 runtime 端到端 / FU-02 PF-04 聚合占位符 / FU-03 lint 子命令） | 并入本 CR | 推荐保持独立符合台账规则；备选并入超 CR-034 范围 | 无阻塞；风险：并入扩大范围至 runtime/实现 |

如果你回复 approve，表示你接受以上 2 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
