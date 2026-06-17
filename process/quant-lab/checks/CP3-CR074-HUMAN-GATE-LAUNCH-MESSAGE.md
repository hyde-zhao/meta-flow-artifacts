---
check_id: "CP3-CR074-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-16T22:14:44+08:00"
checkpoint: "process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md"
---

# CP3 CR074 Human Gate Launch Message

请审查：`process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、自动预检、CR074 formal CR、CR073 readiness evidence、CR-INDEX follow-up candidate 和用户“继续”语句；候选问题 11 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log、LLD clarification queue 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR074-01 | architecture | 是否采用 logical cutover marker？ | 采用。 | manual instruction；symlink/shell config；stay old root。 | 推荐方案可审计且可回退。 | root authority 语义变化。 |
| DQ-CP3-CR074-02 | implementation | 写入面是否只限 STATE / CR-INDEX / CR074 evidence？ | 是。 | 同步更多 docs/process；修改 shell/IDE。 | 推荐方案最小；备选过宽。 | 减少状态分叉。 |
| DQ-CP3-CR074-03 | security | 是否禁止 symlink、目录移动和旧根 cleanup？ | 禁止。 | 允许 symlink；允许目录移动。 | 推荐方案保留回退路径。 | old root retirement 走 CR077。 |
| DQ-CP3-CR074-04 | risk_acceptance | 是否接受 target 尚未 env/data/runtime ready？ | 接受。 | 先 CR075/CR076，再 CR074。 | 推荐方案先完成入口治理。 | 后续运行需验证。 |
| DQ-CP3-CR074-05 | follow_up_tracking | 后续 CR075-CR078 如何处理？ | 保持候选，不自动启动。 | 立即创建；取消。 | 推荐方案聚焦且可追溯。 | 后续治理不丢失。 |

不授权项：

- 不授权当前会话自动切换 cwd、shell / IDE / cron / service 修改。
- 不授权删除、重命名或移动旧根。
- 不授权复制、读取或比对 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env`。
- 不授权读取、打印或迁移凭据。
- 不授权 target env rebuild、远端 Git 写入、NAS 新写入、data lake、QMT/MiniQMT runtime 或 CR046 recovery。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
