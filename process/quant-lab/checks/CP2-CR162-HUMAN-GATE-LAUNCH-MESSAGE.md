请审查人工门禁 `CP2-CR162-PRODUCT-BASELINE-REFRESH`。

checklist 路径: `process/checkpoints/CP2-CR162-PRODUCT-BASELINE-REFRESH.md`
自动预检结论: PASS，0 个 blocker。

审批者摘要:
- 本次确认服务的整体目标: 修复 CR161 关闭后遗漏的产品基线追溯，不改写 CR161 历史、设计或 waiver。
- 推荐动作: `approve`，刷新 6 个产品文档与 3 个 feature 文档；以 CR162 reframe 当前基线；通用 CP8 checker 保持独立 candidate。
- approve 后会发生什么: 自动更新 9 文档、运行静态追溯验证、进入 CP7/CP8。
- approve 不授权什么: HLD/ADR、代码/测试/schema/checker、统计计算、研究引擎、真实数据、凭据、runtime/trading、external framework、Git remote、publish。
- 不确认会阻塞什么: CR161 evidence contract 继续不在当前产品基线中可追溯。

本轮待人工决策项: 3

Context Capsule:
- `process/context/CP2-CR162-PRODUCT-BASELINE-REFRESH-CONTEXT.yaml`
- read_profile: compact
- 全文档读取: `RE-20260710T122812Z0000-cfa7c029`

决策收集覆盖:
- 已扫描 CR161 post-closure finding、formal CR document commitments、CP1 completeness、CP8 checker gap 和 current CR tracking。

决策分层:
- 必须用户决策: DQ-CP2-CR162-001、DQ-CP2-CR162-002、DQ-CP2-CR162-003。
- 高风险策略确认: 无；本 CR 不授权 runtime 或数据。
- agent 默认处理: 仅增量更新、保留旧基线、CP3-CP6 N/A。
- 仅审计记录: CR161 independent-verifier waiver 与 legacy tracking errors。

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|
| `DQ-CP2-CR162-001` | scope | 刷新全部 9 个承诺文档。 | 只刷新 6 个产品文档；或继续 defer。 | 仅 6 个会留下 feature baseline 缺口。 |
| `DQ-CP2-CR162-002` | scope | CR162 更新当前基线，CR161 完成后标为 reframed。 | 重开 CR161；或不 reframe。 | 保留历史审计，避免篡改已关闭门禁。 |
| `DQ-CP2-CR162-003` | follow_up_tracking | 将通用 CP8 checker 记录为未来 candidate。 | 当前实现 checker；或不跟踪。 | 当前 correction 不扩展为工具实现。 |

如果你回复 approve，表示你接受 CR162 的 9 文档刷新、CR161 historical reframe 和 checker candidate 分拆。

不表示授权任何实现、计算、数据、runtime、trading、external framework、Git remote 或 publish。

不授权项: HLD/ADR 语义变更、代码/测试/schema/checker、research-engine instrumentation、FDR/PBO/DSR/OOS/TCA/capacity computation、real lake/NAS/provider/credential/broker、simulation/paper/live/trading、external framework、Git remote、publish。

请只回复以下三个选项之一：
- `approve`
- `修改: <具体修改点>`
- `reject`
