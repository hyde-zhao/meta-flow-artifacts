---
check_id: "CP8-CR079-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T06:25:00+08:00"
checkpoint: "process/checkpoints/CP8-CR079-DELIVERY-READINESS.md"
---

# CP8 CR079 Human Gate Launch Message

请审查：`process/checkpoints/CP8-CR079-DELIVERY-READINESS.md`

自动预检结论：PASS_WITH_RISK，阻断项 0。

Context Capsule：`process/context/CP8-CR079-DELIVERY-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CP8 自动预检、CP6 inventory、CP7 verification、Release Context、release docs 和用户“好的，先完成CR079的迁移”语句；候选问题 30 项，纳入待决策 5 项；discussion log 和委托 Agent 交还摘要为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR079-01 | risk_acceptance | 是否接受 CR079 READY_WITH_RISK 并关闭当前 metadata-only 交付？ | 接受并关闭 CR079 为 `closed-current-delivery / READY_WITH_RISK`。 | 保持 active；reject。 | 推荐方案收敛已授权清点。 | 接受 R-CR079-01..03。 |
| DQ-CP8-CR079-02 | scope | 关闭范围是否只覆盖 metadata-only inventory 和 CP7 verification？ | 限定为四路径存在性、聚合统计、脱敏摘要、禁止项计数和后续分流。 | 扩大为真实迁移完成；扩大到内容校验。 | 推荐方案符合已批准边界。 | 不证明 target root 已恢复，也不证明内容质量。 |
| DQ-CP8-CR079-03 | security | CP8 approve 是否继续不授权内容读取、复制恢复、凭据、provider、remote、runtime 和 cleanup？ | 继续不授权，相关事项均需独立 CR。 | 同时授权 restore/copy 或内容读取。 | 推荐方案保持最小安全面。 | 防止数据、凭据、外部写入和交易风险。 |
| DQ-CP8-CR079-04 | runtime_authorization | 后续真实 data/reports 迁移 / 恢复如何授权？ | 通过 `CR079-restore-candidate` 或等价新 CR 单独批准来源、目标、命令族、验证和回滚。 | 在 CP8 approve 中一并授权恢复。 | 推荐方案可审计且权限最小。 | target root 仍缺 data/reports。 |
| DQ-CP8-CR079-05 | follow_up_tracking | CR079 关闭后后续事项如何分流？ | restore/copy、NAS compare、provider rebuild、CR077、CR078、CR046 recovery 均保留且不自动启动。 | 立即启动其中一项；取消全部后续项。 | 推荐方案保留追踪且不扩大当前范围。 | 后续仍需人工决策。 |

不授权项：

- 不授权读取文件内容、打印样本、解析 parquet/csv/json 或计算内容 hash。
- 不授权复制、恢复、比对内容、移动、重命名或删除 `reports/`、`data/`。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 provider fetch、lake write、catalog publish、NAS promote、远端 Git 写入、QMT/MiniQMT runtime 或 CR046 recovery。
- 不授权 old root retirement、optional dependency groups、full tests、删除 `.venv` 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
