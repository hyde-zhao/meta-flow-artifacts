请审查人工门禁 `CP5-CR169-ALL-STORIES-LLD-BATCH`。

checklist 路径: `process/checkpoints/CP5-CR169-ALL-STORIES-LLD-BATCH.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP5-CR169-ALL-STORIES-LLD-BATCH.md` 的结果为准。

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
blocking / high-risk 决策摘要: DQ-CP5-CR169-001, DQ-CP5-CR169-002, DQ-CP5-CR169-003, DQ-CP5-CR169-004

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR169-001 | implementation | 是否批准五份 full LLD 作为一个批次？ | 批准 S01–S05。 | 指定 Story 修改并保持门禁。 | 批量批准保持跨 Story public contract 一致；逐 Story 修改更细但会阻塞整个依赖链。 | 解锁 CP6；未审查的公共合同变化会造成安全/返工风险。 |
| DQ-CP5-CR169-002 | implementation | 是否批准 5-Wave 串行 DAG 与 file owner？ | S01→S02→S03→S04→S05，一 Wave 一 Story。 | 调整依赖/owner 后重跑 CP4。 | 串行降低 shared evidence/catalog/security files 冲突；代价是吞吐低。 | 决定 merge 顺序和单写边界。 |
| DQ-CP5-CR169-003 | security | 是否授权本地源码/测试/合成 fixture/checker script 编辑与 `uv run` 验证？ | 仅批准文件与本地验证；所有真实/外部/远端/Stage3 禁止保持。 | 保持 design-only；或额外权限另立人工门禁。 | 推荐是交付 fixture foundation 的最小权限；design-only 无法完成，扩权不必要。 | 解锁 CP6/CP7，但不产生真实 capacity 或运行能力。 |
| DQ-CP5-CR169-004 | risk_acceptance | 是否接受 no-subagent 下 CP7 可 inline，且 CP8 必须披露 verifier independence？ | 接受 fixture 低风险例外；仍按分层验证执行。 | 保持 design-only，等待独立 verifier/FU006。 | 推荐遵从用户 no-subagent 指令并透明披露；备选独立性更强但停止交付。 | 影响验证独立性，不影响自动断言严格度。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
