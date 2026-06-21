请审查：`process/checkpoints/CP8-MF-018-DELIVERY-READINESS.md`

自动预检结论：PASS，阻断项 0。机器结果见 `process/checks/CP8-MF-018-DELIVERY-READINESS.result.json`。

Context Capsule：`process/context/CP8-DELIVERY-CONTEXT.yaml`，read_profile=`compact`；默认先读 capsule，仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。

决策收集覆盖：已扫描 7 个来源，发现候选问题 6 个，纳入待决策 3 个；N/A / 缺失来源和分类原因见 checkpoint 的 `### Decision Collection Coverage`。

本轮待人工决策项：3

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP8-MF018-DQ-01 | `risk_acceptance` | 是否接受 MF-018 `READY_WITH_RISK`：质量验证 PASS，但 CP7 由 host-orchestrator inline fallback 执行。 | 接受 `READY_WITH_RISK`，将 inline fallback 作为已披露非阻断风险。 | A：降级 `NOT_READY` 并要求功能 subagent 复验；B：只接受代码与测试，暂缓 CP8。 | 推荐方案完成收尾；备选 A 证据更独立但延迟；备选 B 缩小确认范围但无法 delivered。 | 影响交付状态和风险接受记录；不授权真实发布。 |
| CP8-MF018-DQ-02 | `runtime_authorization` | 是否确认 CP8 `approve` 不授权真实发布、外部 SaaS、凭据、trace upload、publish/live、production 写入或删除 artifact 仓库。 | 明确不授权上述真实运行和外部操作。 | A：另开 runtime_authorization CR；B：转 Spike 评估真实运行。 | 推荐方案保持最小权限；备选 A/B 可补真实运行证据但需要额外授权。 | 降低安全和权限风险。 |
| CP8-MF018-DQ-03 | `follow_up_tracking` | 是否确认当前不立即创建 MF-018 follow-up CR，仅按观察阈值触发。 | 当前不创建后续 CR，通过 `docs/release/FEEDBACK.md` 观察。 | A：立即创建 subagent 复验 CR；B：立即创建 parser hardening CR。 | 推荐方案保持收尾范围清晰；备选 A/B 提前降风险但扩大范围。 | 影响后续 backlog 和 CR 台账。 |

不授权项：
- true publish 或 live release execution
- 外部 SaaS 执行
- credential / secret 读取
- trace upload
- production target project write
- 删除或替换用户提供的 artifact 仓库

如果你回复 approve，表示接受上表全部推荐方案，并接受 MF-018 `READY_WITH_RISK` 交付就绪；不表示授权上述不授权项。

请直接回复以下任一整行：

approve

修改: <具体修改点>

reject
