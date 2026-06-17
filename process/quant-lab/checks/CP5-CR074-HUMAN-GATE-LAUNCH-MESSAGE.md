---
check_id: "CP5-CR074-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-16T22:14:44+08:00"
checkpoint: "process/checkpoints/CP5-CR074-CUTOVER-READINESS.md"
---

# CP5 CR074 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR074-CUTOVER-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP5-CR074-CUTOVER-READINESS-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、自动预检、CR074 formal CR、CR073 readiness evidence、CR-INDEX follow-up candidate 和用户“继续”语句；候选问题 11 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log、LLD clarification queue 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR074-01 | implementation | CP5 approved 后是否允许执行 logical cutover marker？ | 允许，需三门 approved 且 preflight PASS。 | 只保留 gate；只发人工说明。 | 推荐方案完成可审计切换。 | STATE / CR-INDEX 更新。 |
| DQ-CP5-CR074-02 | implementation | 执行前 preflight 如何处理？ | 重跑 target CR tracking、CR073 evidence、forbidden paths checks。 | 只看文件存在；跳过 target check。 | 推荐方案证据充分。 | 防止切到不完整 target。 |
| DQ-CP5-CR074-03 | security | 执行是否仍禁止 env/data/report/credential/runtime/remote？ | 继续禁止。 | 合并 CR075/CR076/CR078。 | 推荐方案范围清晰。 | target 仍非 runtime-ready。 |
| DQ-CP5-CR074-04 | implementation | 执行后验证范围是什么？ | 验证 marker、old root retained、CR tracking PASS、forbidden paths empty。 | 只看 marker；运行 full tests。 | 推荐方案贴合 cutover。 | full tests 属于 CR075。 |
| DQ-CP5-CR074-05 | risk_acceptance | partial marker 或状态分叉是否自动清理？ | 不自动清理，记录并等待授权。 | 自动回滚覆盖；自动删除冲突。 | 推荐方案避免破坏。 | 可能留下待修复分叉。 |

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
