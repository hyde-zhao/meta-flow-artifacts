请审查人工门禁 `CP2-CR047`。

checklist 路径: `process/checkpoints/CP2-CR047.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR047.md` 的结果为准。

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
blocking / high-risk 决策摘要: DQ-047-01, DQ-047-02

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-047-01 | scope | SKILL.md 是否补 post_delay step 级参数说明？ | **A：补一行**（case_steps step 可选字段 `post_delay`，支持 `"5s"` / `"500ms"` / 纯数字秒，叠加在 `OP_INTERVAL_SEC` 之上，仅 `--execute` 模式生效，None/非法值按 0.0）+ 修订记录追加一行 | B：不补，CP6/CP8 写 N/A 理由 | A 使参数对用例作者可发现，与既有惯例一致（转换规则表已文档化 retry/known_issue）；成本一行；B 零文档成本但形成隐性参数，且 N/A 理由与事实相悖 | 影响 / 风险：A 纯文档增量无代码风险；B 回源价值打折 |
| DQ-047-02 | implementation | workflow_mode 判定？ | **A：standard**（规则默认值；同 CR-039/CR-046 回源先例；CP3 预判 N/A、CP5 technical-note，全链路 CP0-CP8 轻量保留） | B：fast-lane（单文件+单测、无运行授权，形式上符合低风险轻量） | A 审计链完整，与并行 CR-036 基线注明衔接有据；B 省几个轻量门禁，但「fast-lane × 并行 CR 影响面重叠」无先例，且 fast-lane 仍不可跳过 CP6/CP7/CP8，节省有限 | 影响 / 风险：B 使 CR-036 收尾基线注明缺乏标准证据链支撑；A 代价是多走 CP2/CP5/CP8 三个轻量门禁 |

如果你回复 approve，表示你接受以上 2 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
