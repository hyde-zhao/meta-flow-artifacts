---
check_id: "CP5-CR073-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-16T10:07:45+08:00"
checkpoint: "process/checkpoints/CP5-CR073-SCOPED-SYNC-READINESS.md"
---

# CP5 CR073 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR073-SCOPED-SYNC-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP5-CR073-SCOPED-SYNC-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、自动预检、CR073 formal CR、diff check、CR071/CR072 上游证据和用户显式启动语句；候选问题 11 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log、LLD clarification queue 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR073-01 | implementation | CP5 approved 后是否允许执行 scoped local sync？ | 允许，但必须三门 approved 且 preflight PASS。 | 只保留 dry-run；只同步 missing。 | 推荐方案完成收敛；备选会留过期证据或无改动。 | target evidence 更新。 |
| DQ-CP5-CR073-02 | implementation | 是否冻结命令形态？ | 使用 `rsync -ai --relative --files-from=...`，保留排除项，不用 `--delete`。 | manual patch；tracked-only archive。 | 推荐方案可复现；备选易漏。 | 同步可审计性。 |
| DQ-CP5-CR073-03 | security | 执行前 preflight 如何处理？ | 重跑 dry-run、root find、git status、git ls-files；异常即停止。 | 直接执行；只看 exit code。 | 推荐方案降低越界风险。 | 防止复制敏感/运行态路径。 |
| DQ-CP5-CR073-04 | implementation | 执行后验证范围是什么？ | 验证 target CR071 CP8、CR072、CR073、STATE、CR-INDEX 和 forbidden paths。 | 只看 rsync exit code；运行 full pytest。 | 推荐方案贴合证据收敛；full pytest 超出且需 env。 | target readiness 可审计。 |
| DQ-CP5-CR073-05 | risk_acceptance | partial sync 或冲突是否自动清理？ | 不自动清理，不用 `--delete`；记录并等待授权。 | 自动删除 target；自动覆盖重试。 | 推荐方案避免破坏；备选需 destructive 授权。 | 可能留下 partial evidence。 |

不授权项：

- 不授权 root cutover 或切换日常工作根。
- 不授权删除、重命名或移动旧根。
- 不授权复制、读取或比对 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env`。
- 不授权读取、打印或迁移凭据。
- 不授权 `--delete`、target cleanup、remote Git write、NAS 新写入、data lake、QMT/MiniQMT runtime 或 CR046 recovery。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
