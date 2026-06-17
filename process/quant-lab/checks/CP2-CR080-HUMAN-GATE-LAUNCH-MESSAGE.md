---
check_id: "CP2-CR080-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T06:42:48+08:00"
checkpoint: "process/checkpoints/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md"
---

# CP2 CR080 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CR080 formal CR、CR079 CP8、CR079 inventory、CR-INDEX follow-up、自动预检和用户“启动CR079-restore-candidate”语句；候选问题 22 项，纳入本门待决策 5 项；委托 Agent 交还摘要和 discussion log 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR080-01 | scope | 是否正式启动 CR080？ | 启动 CR080，将候选转 active。 | 保持 candidate；取消恢复/复制。 | 推荐方案补齐 target 缺口。 | 新增高风险正式 CR。 |
| DQ-CP2-CR080-02 | scope | 来源和目标是否限定？ | legacy retained assets -> target root。 | 先 NAS compare；provider rebuild。 | 推荐方案外部依赖最少。 | legacy 不足时需后续 CR。 |
| DQ-CP2-CR080-03 | security | 是否继续禁止内容读取和凭据？ | 继续禁止。 | 授权内容验证；授权凭据。 | 推荐方案最小权限。 | 内容质量暂不证明。 |
| DQ-CP2-CR080-04 | runtime_authorization | CP2 approve 是否单独授权执行？ | 不授权，需三门 approved + preflight PASS。 | CP2 后立即执行；一次性授权。 | 推荐方案符合门控。 | 防止 CP2 被误读成执行许可。 |
| DQ-CP2-CR080-05 | follow_up_tracking | 后续候选是否独立？ | NAS/provider/CR077/CR078 均不自动启动。 | 同时启动 NAS 或远端/旧根治理。 | 推荐方案避免扩大影响面。 | 后续仍需逐项授权。 |

不授权项：

- 不授权 CP2 单独触发 `reports/` / `data/` restore/copy；必须 CP2 / CP3 / CP5 全部 approve 且 preflight PASS。
- 不授权读取文件内容、打印样本、解析 parquet/csv/json 或计算内容 hash。
- 不授权读取、打印、记录、复制或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS compare/promote、provider fetch、lake write、catalog publish、远端 Git、QMT/MiniQMT runtime、CR046 recovery、旧根删除/移动、optional groups、full tests、`--delete` 同步或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
