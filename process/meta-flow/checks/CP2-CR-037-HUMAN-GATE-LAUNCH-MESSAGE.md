CR-037 CP2 产品 / 场景 / 范围基线准备完成。

checklist 路径: `process/checkpoints/CP2-CR-037-REQUIREMENTS-BASELINE.md`

自动预检结论: `process/checks/CP2-CR-037-REQUIREMENTS-BASELINE.result.json` 为 PASS；CP0 result PASS；CP2 context capsule ready；human-gate 发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR-037-REQUIREMENTS-BASELINE.md --launch-message-file process/checks/CP2-CR-037-HUMAN-GATE-LAUNCH-MESSAGE.md` 的结果为准。

Context Capsule: `process/context/CP2-CR-037-CONTEXT.yaml`，read_profile=`compact`，默认读取 CR-037、产品基线、场景矩阵、Story Map、MVP Scope 和 CP2 discussion 证据；不默认读取 quant-lab、完整归档或完整会话 transcript。

审批者摘要:
- 本次确认服务的整体目标: 确认 CR-037 的产品 / 场景 / 范围基线，允许进入 CP3 蓝图 / HLD 人工确认路径。
- 推荐动作: approve 当前基线；自动预检无 blocker。
- approve 后会发生什么: CR-037 可进入 CP3；后续仍需 CP3 和 CP5 才能进入实现。
- approve 不授权什么: 不授权实现、CP5、runtime、生产写入、publish、live、真实交易、读取凭据、修改 quant-lab 发布库，也不关闭、恢复或完成 CR-036。
- 不确认会阻塞什么: 阻塞 CR-037 进入 CP3，P0/P1/P2 均不得实现。

决策分层:
- 必须用户决策: 2
- 高风险策略确认: 5
- agent 默认处理: 2
- 仅审计记录: 4

决策收集覆盖: 已扫描 STATE / dispatch / 自动预检 / discussion log / 产品基线 / 当前对话与 CR；候选问题数 18，纳入待决策 7，N/A / 缺失来源原因见 checkpoint 的 `### Decision Collection Coverage`。

本轮待人工决策项: 7

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP2-CR037-DQ-01 | `scope` | 是否批准 CR-037 产品 / 场景 / 范围基线，并允许进入 CP3？ | 批准当前基线，进入 CP3；实现仍需 CP3 / CP5。 | 要求修改指定基线后重发 CP2；或 reject 暂停 CR-037。 | 推荐方案保持八轮评审收敛结果并推进设计确认。 | 若基线错误会传导到 HLD 和 Story；可回退修订。 |
| CP2-CR037-DQ-02 | `risk_acceptance` | 是否接受 CR-036 仍 blocked / unfinished，且冲突检查仍会报告 CR-037 与 CR-036 重叠？ | 接受该风险边界；CR-036 保持 blocked，不视为完成。 | 先完成 / 关闭 CR-036；或显式 supersede 后再继续。 | 推荐方案符合已授权暂停策略，可继续处理更紧急的状态治理。 | 冲突检查预期失败需继续在后续门禁说明。 |
| CP2-CR037-DQ-03 | `security` | 是否确认 CP2 approve 不授权实现、CP5、runtime、生产写入、publish、live、真实交易、读取凭据或 quant-lab 发布库修改？ | 确认不授权边界；高风险动作需后续门禁或独立授权。 | 在 CP2 额外授权有限实现预研；或新增 runtime / publish 授权门禁后再继续。 | 推荐方案最小权限且符合 meta-flow 门禁。 | 防止 CP2 被误解为实现或生产授权。 |
| CP2-CR037-DQ-04 | `implementation` | 是否确认 P0 state enforcement 采用 allowlist + field budget + audit/enforce，并阻断第二套机制？ | 接受 allowlist + budget + audit/enforce；不采用黑名单 strip 或新增平行机制。 | 先 audit-only；或只做检查报告不阻断写入。 | 推荐方案能根治未知字段和重型状态污染。 | allowlist 可能误伤存量字段，需灰度。 |
| CP2-CR037-DQ-05 | `runtime_authorization` | 是否确认 roadmap refresh 只自动写过程归档库机器状态？ | 只自动写过程归档库；发布库影响进入 stale / follow-up。 | 完全禁用 cascade；或未来独立 CR 授权发布库写入。 | 推荐方案兼顾追踪和最小授权。 | 避免把 doc impacts 误解为发布库写入授权。 |
| CP2-CR037-DQ-06 | `implementation` | 是否确认 feature_refs / capability_refs 必须引用标准 registry？ | 只引用真实已注册 ID；缺失时 blocked 或 FU-RF。 | 生成 proposed entries 后人工确认；或短期 warning。 | 推荐方案避免第二套 capability 命名空间。 | 过严会增加迁移阻力，过松会延续漂移。 |
| CP2-CR037-DQ-07 | `scope` | 是否确认 P2 quant-lab migration 是验证切片，不授权发布库修改？ | 只做过程侧迁移、stale report 和 FU-RF；发布库改动另行授权。 | 暂缓 P2；或将 quant-lab 迁移拆成独立 CR。 | 推荐方案用真实项目验证机制且保持边界。 | P2 过早绑定真实项目可能扩大范围。 |

如果你回复 approve，表示接受上表 7 项推荐方案；不表示授权实现、CP5、runtime、production write、publish、live、真实交易、凭据读取、quant-lab 发布库修改，也不表示关闭、恢复或完成 CR-036。

不授权项:
- 不将 CR-036 暂停视为完成或关闭。
- 不自动关闭、恢复或完成 CR-036。
- 不自动修改 quant-lab 发布库代码、测试、正式设计文档或发布文档。
- 不读取凭据、账户、token、cookie 或私钥。
- 不执行真实交易、live、publish、production write 或外部服务真实调用。
- 不进入实现或 CP5。

请只回复以下三个 exact 选项之一:

- approve
- 修改: <具体修改点>
- reject
