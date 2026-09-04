请审查人工门禁 `CP8-CR039`。

checklist 路径: `process/checkpoints/CP8-CR039.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP8-CR039.md` 的结果为准。

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
blocking / high-risk 决策摘要: CP8-DQ-039-RISK, CP8-DQ-039-RUNTIME, CP8-DQ-039-047, CP8-DQ-039-DOC

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP8-DQ-039-RISK | risk_acceptance | 接受 5 项 static-only 遗留风险（R-1 真机大路由表 verify 未覆盖 / R-3 telnetlib py>=3.13 / R-4 sw3 全链路 dry-run / R-5 2 KNOWN_FAIL 追验 / P-1 存量测试失败）？ | **接受，全部转 follow-up candidate 跟踪** | 备选 1：本轮即授权真机回归（需 runtime_authorization）；备选 2：补齐 R-4 单测后再交付 | 推荐不扩大本轮范围、风险均有台账跟踪；真机回归可独立发起 | 接受后真机行为仍未验，重装部署后首跑需关注 |
| CP8-DQ-039-RUNTIME | runtime_authorization | 是否现在发起真机回归授权（trex 发流 / SSH 重启 trex-stl-te / sw3 telnet 实连）？ | **不发起**（本 CR 不授权；后续真机回归单独走 runtime_authorization 决策） | 现在一并授权 | 推荐：独立授权边界清晰（沿用 CR-038 模式）；备选：省一次门禁但风险集中 | 不授权：真机验证延后；授权：影响测试环境 |
| CP8-DQ-039-047 | follow_up_tracking | CR-047 post_delay 独立回源候选（CR-039-FU-047）维持 candidate？ | **维持 candidate**（DQ-039-05=B 已决策；用户决定推进时才建正式 CR） | 现在启动 CR-047 | 推荐：符合 DQ-039-05=B 决策与后续 CR 启动协议 | 维持：post_delay 回源延后，工作区与 canonical 差异保留（BACKLOG 已记） |
| CP8-DQ-039-DOC | scope | README/USER-MANUAL 增量判定 N/A 确认？ | **N/A 确认**（未新增安装脚本/用户流程；行为契约已落 SKILL.md 与 op-coverage-matrix 并经 CP7 核验） | 可选：补记 meta-doc 小任务（回源内容进 USER-MANUAL） | 推荐：无交付缺口；备选：文档更完整但非必需 | 影响 / 风险：无新增交付缺口，风险极低 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
