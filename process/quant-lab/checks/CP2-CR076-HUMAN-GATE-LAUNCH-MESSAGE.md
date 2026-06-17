---
check_id: "CP2-CR076-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T00:18:01+08:00"
checkpoint: "process/checkpoints/CP2-CR076-DATA-REPORTS-BASELINE.md"
---

# CP2 CR076 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR076-DATA-REPORTS-BASELINE.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP2-CR076-DATA-REPORTS-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CR076 formal CR、CR075 CP8 approval、CR-INDEX follow-up、自动预检和用户“处理完成CR075后继续处理076”语句；候选问题 10 项，纳入待决策 5 项；委托 Agent 交还摘要和 discussion log 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR076-01 | scope | 是否正式启动 CR076？ | 启动 CR076，进入 CP3/CP5 策略门禁。 | 保持 candidate；取消 CR076。 | 推荐方案响应用户继续请求。 | 新增正式 CR 和待审门禁。 |
| DQ-CP2-CR076-02 | scope | 范围是否限定为 policy-only？ | 限定为策略、边界、候选路径和后续门禁。 | 立即纳入真实读取；立即恢复或删除。 | 推荐方案最小触碰。 | data/reports readiness 仍未知。 |
| DQ-CP2-CR076-03 | security | 是否继续禁止 `.env`、凭据、账户和交易事实？ | 继续禁止。 | 同时授权凭据读取或交易事实查询。 | 推荐方案最小权限。 | 防止凭据和交易事实泄露。 |
| DQ-CP2-CR076-04 | architecture | 三条恢复路径是否仅为候选？ | local retained、NAS archive、provider rebuild 均只作为候选。 | 立即选定并执行其中一条。 | 推荐方案先定策略。 | 后续恢复路径暂未验证。 |
| DQ-CP2-CR076-05 | risk_acceptance | 是否接受 readiness 仍未知？ | 接受未知状态，仅记录为后续风险。 | 现在验证 readiness；取消后续处理。 | 推荐方案不触碰敏感路径。 | 数据可用性仍未证明。 |

不授权项：

- 不授权读取、列出、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 provider fetch、lake write、catalog publish、远端 Git 写入、QMT/MiniQMT runtime 或 CR046 recovery。
- 不授权删除旧根、optional dependency groups、full tests、删除 `.venv` 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
