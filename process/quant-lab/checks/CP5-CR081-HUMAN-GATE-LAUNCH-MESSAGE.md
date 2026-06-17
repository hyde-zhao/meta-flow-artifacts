---
check_id: "CP5-CR081-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T08:03:08+08:00"
checkpoint: "process/checkpoints/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md"
---

# CP5 CR081 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CR081 formal CR、CP5 自动预检、`.gitignore` / Git state、运行 / 外部接口和 discussion log；候选问题 17 项，纳入本门待决策 5 项；N/A 来源原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR081-01 | implementation | 三门通过后是否先执行 preflight？ | 是，检查 process、external root、tracked count、断链和 dirty worktree。 | 直接执行 cutover；只做手工检查。 | 推荐方案可阻断错误路径。 | 可能需用户补路径。 |
| DQ-CP5-CR081-02 | implementation | 是否允许 mirror `process/` 到外部 ledger？ | preflight PASS 后用 `rsync -a`，禁止 `--delete`。 | 手工复制；`rsync --delete`。 | 推荐方案保守且可验证。 | 目标非空可能冲突。 |
| DQ-CP5-CR081-03 | implementation | 是否允许 Git index 停止跟踪 `process/**`？ | 允许 `git rm --cached -r process`，但不 commit/push。 | 不改 Git index；track symlink。 | 推荐方案让 owner 单一。 | 产生 staged deletion，需单独提交确认。 |
| DQ-CP5-CR081-04 | implementation | 是否允许本地 `process` 替换为 symlink？ | 允许在备份和 mirror 验证后替换。 | 只建 `process_external` symlink。 | 推荐方案实现目标。 | symlink 断链会影响 Meta Flow。 |
| DQ-CP5-CR081-05 | risk_acceptance | 是否接受 cutover 后剩余风险？ | 接受 bootstrap、external ledger 在线和禁止 destructive symlink 操作要求。 | 先转 Spike；mirror-only。 | 推荐方案可落地。 | clone 后需重建 symlink。 |

不授权项：

- 不授权 Git commit、Git push、remote write、branch/default branch governance、history rewrite、force push 或 tag。
- 不授权 `rsync --delete`、`find -L`、跟随 symlink 的批量删除或 destructive cleanup。
- 不授权访问 NAS 内容、mount、promote、删除或写入 NAS 数据文件。
- 不授权读取 `.env`、凭据、data/reports 内容、样本解析、hash、provider/lake/catalog 或 runtime。
- 不授权迁移 `docs/design`、`docs/features`、`docs/quality`、`docs/release`。
- 不授权 CR046 recovery、old root retirement 或目标根删除/移动。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
