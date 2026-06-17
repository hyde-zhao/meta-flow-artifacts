---
check_id: "CP5-CR080-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T06:42:48+08:00"
checkpoint: "process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md"
---

# CP5 CR080 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CR080 formal CR、CP5 自动预检、CP2/CP3 context 和当前用户语句；候选问题 27 项，纳入本门待决策 5 项；Story / LLD artifacts 和委托 Agent 交还摘要为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR080-01 | runtime_authorization | 是否允许 post-approval preflight？ | 允许，只做元数据和状态检查。 | 不允许执行；只保留文档。 | 推荐方案为安全执行做准备。 | preflight 会触碰目录元数据。 |
| DQ-CP5-CR080-02 | runtime_authorization | preflight PASS 后是否允许受控 copy？ | 允许本地 legacy -> target copy。 | 只批准 preflight；改 NAS compare。 | 推荐方案能补齐 target。 | 会向 target root 写入。 |
| DQ-CP5-CR080-03 | security | 验证证据是否限制为元数据摘要？ | 是，只写存在性、数量、规模、mtime、exclude 命中和禁止项计数。 | 完整清单；内容 hash。 | 推荐方案防泄露。 | 不证明内容质量。 |
| DQ-CP5-CR080-04 | implementation | partial 状态是否禁止自动 cleanup？ | 禁止，记录后人工决策。 | 自动删除；自动隔离。 | 推荐方案避免破坏性操作。 | 失败可能留下 partial。 |
| DQ-CP5-CR080-05 | follow_up_tracking | 执行后后续候选如何分流？ | 成功后再评审 CR077；NAS/provider/CR078 不自动启动。 | 立即旧根退役；立即远端治理。 | 推荐方案保持最小步进。 | 旧根/远端仍需独立门禁。 |

不授权项：

- 不授权跳过 post-approval preflight。
- 不授权内容读取、样本打印、文件解析、内容 hash、checksum 校验或内容比对。
- 不授权复制 credential-like path；不授权读取或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 `rsync --delete`、覆盖已有非空 target、自动 merge、自动 cleanup、旧根删除/移动。
- 不授权 NAS/provider/lake/catalog、remote Git、QMT/MiniQMT runtime、CR046 recovery、optional groups 或 full tests。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
