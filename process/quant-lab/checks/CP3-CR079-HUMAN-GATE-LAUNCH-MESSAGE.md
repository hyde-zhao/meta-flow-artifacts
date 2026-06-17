---
check_id: "CP3-CR079-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T05:56:16+08:00"
checkpoint: "process/checkpoints/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-REVIEW.md"
---

# CP3 CR079 Human Gate Launch Message

请审查：`process/checkpoints/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-REVIEW.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 CP3 Context Capsule、CR079 formal CR、自动预检、CR076 policy 和用户启动语句；候选问题 16 项，纳入待决策 5 项；委托 Agent 交还摘要和 discussion log 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR079-01 | architecture | 是否采用 metadata-only 只读清点方案？ | 采用；只读目录元数据，不读内容。 | 内容读取；取消 CR079。 | 推荐方案最小权限。 | readiness 只证明结构和规模。 |
| DQ-CP3-CR079-02 | architecture | 清点路径是否限定为 target/legacy 四个目录？ | 限定为 target/legacy 的 `reports/`、`data/`。 | 同时访问 NAS；同时访问 provider/lake/catalog。 | 推荐方案范围清楚。 | NAS / provider 状态仍未知。 |
| DQ-CP3-CR079-03 | security | 证据是否只写脱敏摘要且不写 memory？ | 只写聚合摘要、顶层风险标记；不写样本或完整深层清单到 memory。 | 保存完整清单；保存样本。 | 推荐方案符合 memory policy。 | 后续排查细节较少。 |
| DQ-CP3-CR079-04 | runtime_authorization | approve 后允许的命令族是否只限本地只读元数据命令？ | 只允许存在性、find/stat/du 类只读元数据清点。 | 允许内容读取；允许 rsync/cp/rm。 | 推荐方案防止越权。 | 命令设计保守。 |
| DQ-CP3-CR079-05 | implementation | 失败 / 回退是否仅记录 BLOCKED，不自动 cleanup？ | 是，仅记录失败和下一步决策。 | 失败后删除/清理；失败后恢复旧根。 | 推荐方案不触碰数据。 | 可能保留未知状态。 |

不授权项：

- 不授权读取文件内容、打印样本、解析 parquet/csv/json、计算内容 hash、复制、恢复、比对内容、移动、重命名或删除。
- 不授权访问 NAS 归档内容、provider fetch、lake write、catalog publish 或 provider rebuild。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权远端 Git 写入、QMT/MiniQMT runtime、CR046 recovery、旧根退役、optional groups、full tests 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
