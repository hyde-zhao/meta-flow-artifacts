---
check_id: "CP5-CR079-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T05:56:16+08:00"
checkpoint: "process/checkpoints/CP5-CR079-DATA-REPORTS-ACCESS-READINESS.md"
---

# CP5 CR079 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR079-DATA-REPORTS-ACCESS-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP5-CR079-DATA-REPORTS-ACCESS-READINESS-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 CP5 Context Capsule、自动预检、CR079 formal CR、STATE / CR-INDEX 和用户启动语句；候选问题 19 项，纳入待决策 5 项；委托 Agent 交还摘要为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR079-01 | implementation | CP5 approve 后是否允许执行一次 metadata-only 只读清点？ | 允许，但必须三门全部 approved 且 preflight PASS。 | 只保留 gate；直接扩大到内容读取。 | 推荐方案完成最小 access。 | 触碰目录元数据。 |
| DQ-CP5-CR079-02 | implementation | 执行前 preflight 是否必须跑 CR tracking 和 human-gate 校验？ | 必须执行并记录 PASS。 | 跳过 preflight；只人工确认。 | 推荐方案保留一致性证据。 | 防止状态漂移。 |
| DQ-CP5-CR079-03 | security | 输出证据是否限制为脱敏摘要？ | 只写存在性、数量、规模、mtime 范围和风险标记。 | 写完整深层清单；写样本或内容摘要。 | 推荐方案降低泄露风险。 | 后续分析粒度有限。 |
| DQ-CP5-CR079-04 | security | 是否继续禁止 `.env`、凭据、provider/lake/catalog、remote Git、runtime、CR046、cleanup？ | 继续禁止全部。 | 放开其中一项；一次性授权全部。 | 推荐方案最小安全面。 | 防止敏感数据、外部写入和交易风险。 |
| DQ-CP5-CR079-05 | follow_up_tracking | 清点后后续 candidate 如何分流？ | restore/copy、NAS compare、provider rebuild、CR077、CR078、CR046 recovery 均不自动启动。 | 清点后自动恢复；清点后自动旧根退役。 | 推荐方案保留人工决策。 | 后续仍需用户决策。 |

不授权项：

- 不授权读取文件内容、打印样本、解析 parquet/csv/json、计算内容 hash、复制、恢复、比对内容、移动、重命名或删除。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 provider fetch、lake write、catalog publish、NAS promote、remote Git、QMT/MiniQMT runtime、CR046 recovery、旧根退役、optional groups、full tests 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
