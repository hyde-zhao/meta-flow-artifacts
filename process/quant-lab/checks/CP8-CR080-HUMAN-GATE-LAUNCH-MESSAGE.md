---
check_id: "CP8-CR080-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T07:14:53+08:00"
checkpoint: "process/checkpoints/CP8-CR080-DELIVERY-READINESS.md"
---

# CP8 CR080 Human Gate Launch Message

请审查：`process/checkpoints/CP8-CR080-DELIVERY-READINESS.md`

自动预检结论：PASS_WITH_RISK，阻断项 0。

Context Capsule：`process/context/CP8-CR080-DELIVERY-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CP8 自动预检、CP6 execution、CP7 verification、Release Context、release docs、子 agent 复核和当前用户语句；候选问题 33 项，纳入待决策 5 项；discussion log 和委托 Agent 交还摘要为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR080-01 | risk_acceptance | 是否接受 CR080 READY_WITH_RISK 并关闭当前 restore/copy 交付？ | 接受并关闭 CR080 为 `closed-current-delivery / READY_WITH_RISK`。 | 保持 active；reject。 | 推荐方案收敛已完成受控 copy。 | 接受 R-CR080-001..03。 |
| DQ-CP8-CR080-02 | scope | 关闭范围是否只覆盖 local restore/copy 与 metadata-only verification？ | 限定为 legacy -> target copy、metadata quick-check、账本同步和 release readiness。 | 扩大为内容级完整性证明；扩大到 NAS/provider rebuild。 | 推荐方案符合已批准边界。 | 不证明内容 hash、freshness 或语义质量。 |
| DQ-CP8-CR080-03 | security | CP8 approve 是否继续不授权内容读取、凭据、NAS/provider/lake/catalog、remote/runtime 和 cleanup？ | 继续不授权，相关事项均需独立 CR。 | 同时授权 checksum/content read；同时授权 cleanup 或旧根退役。 | 推荐方案保持最小安全面。 | 防止数据、凭据、外部写入和交易风险。 |
| DQ-CP8-CR080-04 | implementation | 若需要回滚或清理 target restored assets，是否必须另起 destructive cleanup / rollback CR？ | 是；默认不删除、不移动、不覆盖 target assets，只允许 process ledger 修订。 | 在 CP8 approve 中预授权删除；自动移动到隔离目录。 | 推荐方案避免破坏性操作。 | target `reports/` / `data/` 已写入，删除需额外授权。 |
| DQ-CP8-CR080-05 | follow_up_tracking | CR080 关闭后后续事项如何分流？ | content validation、NAS compare、provider rebuild、CR077、CR078、CR046 recovery 均保留且不自动启动。 | 立即启动其中一项；取消全部后续项。 | 推荐方案保留追踪且不扩大当前范围。 | 后续仍需人工决策。 |

不授权项：

- 不授权读取文件内容、打印样本、解析 parquet/csv/json 或计算内容 hash。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS compare/promote、provider fetch、lake write、catalog publish 或 MARKET_DATA_LAKE_ROOT replacement。
- 不授权远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 QMT/MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live 或 CR046 recovery。
- 不授权 old root retirement、删除 / 移动 target restored assets、optional dependency groups、full tests、删除 `.venv` 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
