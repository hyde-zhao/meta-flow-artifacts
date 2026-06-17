---
check_id: "CP8-CR081-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T08:40:59+08:00"
checkpoint: "process/checkpoints/CP8-CR081-DELIVERY-READINESS.md"
---

# CP8 CR081 Human Gate Launch Message

请审查：`process/checkpoints/CP8-CR081-DELIVERY-READINESS.md`

自动预检结论：PASS_WITH_RISK，阻断项 0。

Context Capsule：`process/context/CP8-CR081-DELIVERY-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CP8 自动预检、CP6 execution、CP7 verification、Release Context、release docs 和当前用户语句；候选问题 28 项，纳入待决策 5 项；discussion log 为 N/A，原因见 checkpoint 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP8-CR081-01 | risk_acceptance | 是否接受 CR081 READY_WITH_RISK 并关闭当前 process ledger externalization 交付？ | 接受并关闭 CR081 为 `closed-current-delivery / READY_WITH_RISK`。 | 保持 active；reject 并回退。 | 推荐方案收敛已完成迁移。 | 接受 R-CR081-001..03。 |
| DQ-CP8-CR081-02 | scope | 关闭范围是否只覆盖 `process/` 外部化和 symlink cutover？ | 限定为 `/home/hyde/workspace/process`、`process -> ../process`、README、bootstrap 和 Git index boundary。 | 同时迁移 `docs/*`；同时做 remote governance。 | 推荐方案符合第一阶段范围。 | docs 过程文件仍在当前项目，remote Git 未治理。 |
| DQ-CP8-CR081-03 | security | CP8 approve 是否继续不授权 data/reports、凭据、NAS 内容、provider/lake/catalog、runtime、remote write 和 cleanup？ | 继续不授权，相关事项均需独立 CR。 | 同时授权 remote commit/push；同时授权 cleanup。 | 推荐方案保持最小权限。 | 防止数据、凭据、外部写入和 runtime 风险。 |
| DQ-CP8-CR081-04 | implementation | `process/**` staged removals 如何处理？ | 保持 staged state，暂不 commit/push，等待后续 Git governance/用户提交决策。 | 立即 commit；restore staged removals。 | 推荐方案保留迁移事实且避免远端写入。 | Git status 会显示大量 staged deletions。 |
| DQ-CP8-CR081-05 | follow_up_tracking | CR081 后续事项如何分流？ | CR078 remote governance、CR077 old root retirement、可选 docs externalization 保留且不自动启动。 | 立即启动其中一项；取消全部后续项。 | 推荐方案保留追踪且不扩大当前范围。 | 后续仍需人工决策。 |

不授权项：

- 不授权 Git commit / push / remote write / branch governance。
- 不授权读取或修改 `data/`、`reports/` 内容。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS 内容访问、provider fetch、lake write、catalog publish。
- 不授权 QMT/MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live 或 CR046 recovery。
- 不授权 old root retirement、删除 external process backup、删除 `/home/hyde/workspace/process` 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
