---
check_id: "CP5-CR082-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T09:11:43+08:00"
checkpoint: "process/checkpoints/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS.md"
---

# CP5 CR082 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 CP5 自动预检、CP2/CP3 materials、CR081 关闭风险、STATE pending queue；候选问题 18 项，纳入待决策 5 项；discussion log 为 N/A，原因见 checkpoint 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR082-01 | runtime_authorization | CP5 approve 后是否授权受控 namespace move？ | 授权 post-approval preflight 通过后执行 `process/* -> process/quant-lab/` 的受控移动。 | 只做 dry-run；不执行。 | 推荐方案完成用户目标。 | 文件路径变化，需严格验证。 |
| DQ-CP5-CR082-02 | implementation | 是否授权更新 quant-lab 项目 symlink？ | 授权将 `/home/hyde/workspace/quant-lab/process` 切换到 `../process/quant-lab`。 | 保持 `../process`；创建第二个 symlink。 | 推荐方案让项目入口直达项目目录。 | 影响日常查阅入口。 |
| DQ-CP5-CR082-03 | implementation | 是否授权更新 README 和指针文件？ | 更新 root README、project README、`LEDGER.md`、`ledger.yaml`、`scripts/link-engineering-ledger.sh`。 | 只更新 README；只更新 symlink。 | 推荐方案消除路径漂移。 | 影响后续 CR078 发布内容。 |
| DQ-CP5-CR082-04 | risk_acceptance | 是否接受 staged `process/**` removals 在 CR082 后继续不提交？ | 接受继续保留 staged removals，等待 CR078 remote governance 统一处理。 | 立即 commit；restore staged removals。 | 推荐方案避免未授权远端写入。 | Git status 仍会显示大量 staged deletions。 |
| DQ-CP5-CR082-05 | follow_up_tracking | CR082 完成后的下一步是否回到 CR078？ | CR082 CP8 关闭后优先启动 CR078 remote Git governance / publication gate。 | 启动 old root retirement；暂停全部后续 CR。 | 推荐方案处理当前最明显 Git 边界问题。 | 后续仍需人工门禁。 |

不授权项：

- 不授权跳过 post-approval preflight 直接移动目录。
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
