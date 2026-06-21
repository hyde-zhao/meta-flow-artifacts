CP8 交付终验准备完成。

checklist 路径: `process/checkpoints/CP8-CR-034-DELIVERY-READINESS.md`

自动预检结论: CP6 result PASS；CP7 result `PASS_WITH_RISK`；release context 已生成；release docs 已更新；human-gate 发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP8-CR-034-DELIVERY-READINESS.md --launch-message-file process/checks/CP8-CR-034-HUMAN-GATE-LAUNCH-MESSAGE.md` 的结果为准。

Context Capsule: `process/release/RELEASE-CONTEXT.yaml`，read_profile=`compact`，默认读取 CP6/CP7 result、evidence index 和 release docs；不默认读取完整 CR、全量 LLD 或完整 diff。

决策收集覆盖: 已扫描 CP7 verification report、release context、release docs 和 authorization boundary；候选问题数 6，纳入待决策 2，其余列入不授权范围或 N/A。

本轮待人工决策项: 2

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| CP8-CR034-DQ-01 | `risk_acceptance` | 是否接受 CR-034 以 `READY_WITH_RISK` 收尾 | 接受 `READY_WITH_RISK` | 阻断直到 self-repo doctor adoption PASS；或回退 CP7 补完整 pytest | 推荐方案保持后续 quant-lab 前置能力推进 | 接受 self-repo readiness 和 full pytest 未闭环风险 |
| CP8-CR034-DQ-02 | `follow_up_tracking` | self-repo adoption completion 是否作为后续候选而非当前阻断 | 记录为 follow-up candidate，不立即启动新 CR | 立即创建后续 CR；或不记录候选 | 推荐方案保留追踪且不扩大 CR-034 | 后续若要求 self-repo doctor PASS，需启动候选项 |

如果你回复 approve，表示接受上表推荐方案，包括 `READY_WITH_RISK` 和 follow-up tracking；不表示授权 credentials、runtime、SaaS、production write、trading、publish、live release 或 CR-033 activation。

不授权项: credentials / secret / account read；runtime / SaaS trace upload；production write；trading / publish / live release；CR-033 activation。

请只回复以下三个 exact 选项之一:

- approve
- 修改: <具体修改点>
- reject
