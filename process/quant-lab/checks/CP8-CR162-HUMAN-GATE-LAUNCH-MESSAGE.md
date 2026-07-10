请审查人工门禁 `CP8-CR162-PRODUCT-BASELINE-RELEASE-READINESS`。

checklist 路径: `process/checkpoints/CP8-CR162-PRODUCT-BASELINE-RELEASE-READINESS.md`
自动预检结论: PASS / READY_WITH_RISK，0 个 blocker。

审批者摘要:
- 本次确认服务的整体目标: 关闭 CR162 产品基线纠错，确认 CR161 的九文档遗漏已补齐并已被静态验证。
- 推荐动作: `approve`，接受九文档刷新和 CR161 reframe，接受通用 checker 作为 `FU-CR162-001` 独立 candidate。
- approve 后会发生什么: Host 将关闭 CR162 为 `READY_WITH_RISK`，随后按用户已有请求提交并同步 artifacts 与 quant-lab 两个仓库。
- approve 不授权什么: 不授权代码/测试/schema/checker、trial-lineage、FDR/PBO/DSR/OOS/TCA/capacity 计算、真实数据/凭据/provider/NAS/broker/runtime/trading、部署或 publish。
- 不确认会阻塞什么: 阻塞 CR162 closure 和双仓库远端同步。

本轮待人工决策项: 3

Context Capsule:
- `process/context/CP8-CR162-PRODUCT-BASELINE-CONTEXT.yaml`
- read_profile: compact
- 全文档读取: `RE-20260710T130048Z0000-f28599a6`

决策收集覆盖:
- CP7 result、release context、follow-up tracking 和 release documents 已扫描。
- 3 个待决策项均已进入 checkpoint 的完整 Decision Brief。

决策分层:
- 必须用户决策: 3
- 高风险策略确认: READY_WITH_RISK 接受与 deny-by-default 边界。
- agent 默认处理: compact release docs、deploy/migration=N/A、候选不启动。
- 仅审计记录: CP7 dispatch-not-required 不声称 independent QA。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP8-CR162-001` | risk_acceptance | 是否接受 checker deferred 的 READY_WITH_RISK closure？ | 关闭纠错；generic checker 保持 FU-CR162-001。 | 实现 checker 后再关闭。 | 完成实际纠错而不扩大 process implementation。 | 中风险：未来可能重复遗漏。 | 同类遗漏重现时提升 FU-CR162-001。 |
| `DQ-CP8-CR162-002` | runtime_authorization | 是否确认 deny-by-default？ | 维持 deny-by-default。 | 单独授权 CR。 | 防止文档 closure 误读为计算或 runtime。 | 高风险：越权。 | 新 CR 的 CP0/CP2。 |
| `DQ-CP8-CR162-003` | follow_up_tracking | 是否保留所有 FU candidates？ | 只登记不启动。 | 提升指定 candidate。 | 保留路线且不扩大范围。 | 中风险：计算能力仍 deferred。 | 用户指定 candidate 后单独路由。 |

如果你回复 approve，表示接受上述 3 项推荐决策并允许关闭 CR162；按既有用户请求，随后将提交并同步 artifacts 与 quant-lab。approve 不表示授权任何统计计算、数据/运行时、交易、部署或 publish。

修改: <具体修改点>

不授权项:
- `approve` 不表示授权代码、checker、research engine、FDR/PBO/DSR/OOS/TCA/capacity、真实数据、凭据、provider、NAS、broker、runtime、trading、deployment 或 publish。

请只回复以下三个 exact 选项之一：
- `approve`
- `修改: <具体修改点>`
- `reject`
