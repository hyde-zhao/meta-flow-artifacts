---
check_id: "CP2-CR082-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T09:11:43+08:00"
checkpoint: "process/checkpoints/CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE.md"
---

# CP2 CR082 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP2-CR082-PROCESS-LEDGER-NAMESPACE-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描当前用户语句、CR081 关闭证据、当前目录结构、STATE pending queue；候选问题 11 项，纳入待决策 5 项；discussion log 为 N/A，原因见 checkpoint 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR082-01 | scope | 是否正式启动 CR082 project namespace normalization？ | 启动 CR082，范围限定为 external process ledger 项目命名空间和指针更新。 | 不启动；合并到 CR078。 | 推荐方案先修正结构边界。 | CR082 将占用当前下一步执行队列。 |
| DQ-CP2-CR082-02 | scope | 项目归档目标目录是否采用 `/home/hyde/workspace/process/quant-lab`？ | 采用 `process/quant-lab/`。 | 采用 `process/projects/quant-lab/`；继续 flat root。 | 推荐方案短路径、与用户示例一致。 | 影响 symlink、README 和恢复脚本路径。 |
| DQ-CP2-CR082-03 | implementation | 现有 flat root 如何处理？ | 仅搬入 `quant-lab/` 并保留必要 root README；不删除 CR081 backup。 | 保留 flat root 内容副本；立即清理旧层。 | 推荐方案减少重复且保留回退材料。 | 需要更新 root README 防止误读。 |
| DQ-CP2-CR082-04 | follow_up_tracking | CR082 与 CR078 的顺序如何安排？ | CR082 先于 CR078，完成 namespace 后再进入 remote Git governance。 | 先做 CR078；取消 CR078。 | 推荐方案避免远端固化 flat root。 | CR078 延后。 |
| DQ-CP2-CR082-05 | security | CP2 approve 是否继续不授权 data/reports、凭据、NAS 内容、remote write、runtime 和 cleanup？ | 继续不授权。 | 同时授权 Git commit/push；同时授权 NAS 管理。 | 推荐方案保持最小权限。 | 防止数据、凭据、远端和 runtime 风险。 |

不授权项：

- 不授权在 CP2/CP3/CP5 全部 approve 前移动 `/home/hyde/workspace/process` 内容或替换 `process` 软链接。
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
