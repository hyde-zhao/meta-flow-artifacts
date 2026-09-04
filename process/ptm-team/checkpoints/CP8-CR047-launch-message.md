请审查人工门禁 `CP8-CR047`。

checklist 路径: `process/checkpoints/CP8-CR047.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP8-CR047.md` 的结果为准。

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
blocking / high-risk 决策摘要: CP8-DQ-047-RISK, CP8-DQ-047-RUNTIME, CP8-DQ-047-039FU, CP8-DQ-047-DOC

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP8-DQ-047-RISK | risk_acceptance | 接受 R-1（负数 post_delay 透传固化语义：极端负值使 sleep 总和为负抛 ValueError，clamp 未实现）与 R-2（post_delay 真机 tear-down 效果未实测）？ | **接受**：R-1 -> deferred（clamp 记 follow-up 台账 CR-047-FU-NEG）；R-2 -> 并入 BL-039-02 / CR-039-FU-RUNTIME 真机回归候选 | ①立即立项 clamp（劣：打破回源逐块一致原则、与工作区基线分歧）；②要求真机验证后再交付（劣：阻断交付，违反 static-only 已决策口径） | 推荐：正常用例不传负值（文档口径 "5s"/"500ms"/纯数字秒），触发概率低；透传语义已有单测锁定 | 影响 / 风险：负值用例极端场景抛 ValueError（单测已锁定行为）；真机未验与 CR-039 同口径 |
| CP8-DQ-047-RUNTIME | runtime_authorization | 真机三不授权（trex 发流 / SSH 重启 / sw3 telnet）+ post_delay 真机实测是否维持不授权？ | **维持不授权**（DQ-039-04 沿用；本次 CP8 仅确认交付就绪） | ①一并授权 BL-039-02 真机回归（劣：扩大授权边界）；②仅授权 post_delay 单项实测（劣：真机回归碎片化） | 推荐：独立授权边界清晰，沿用 CR-038/039 模式 | 影响 / 风险：不授权则 R-2 与 CR-039 真机风险继续挂台账 |
| CP8-DQ-047-039FU | follow_up_tracking | BL-039-01（BACKLOG，open）/ CR-039-FU-047（CR-039 台账，active）是否随 CR-047 回源完成关闭？ | **关闭（closed）**：回源已完成并通过 CP7，台账项使命完成；关闭动作由 host-orchestrator 执行台账侧 status 更新 | ①保持 active 至真机回归完成（劣：台账语义失真）；②置 superseded（劣：无实质替代对象，不推荐） | 推荐：回源完成即关闭，状态一致 | 影响 / 风险：无；纯台账状态一致性 |
| CP8-DQ-047-DOC | implementation | README / USER-MANUAL 是否需要 CR-047 增量补记？ | **N/A**：未新增安装脚本与用户流程；post_delay 行为契约已落 SKILL.md 转换规则表 + 修订记录 v1.9（DQ-047-01=A），发布说明承载交付内容 | ①转 meta-doc 小任务补记 USER-MANUAL（劣：非必要文档面扩大）；②仅 RELEASE-NOTES 扩充（已做） | 推荐：无交付缺口；备选文档更完整但非必需 | 影响 / 风险：无；低风险文档判定 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
