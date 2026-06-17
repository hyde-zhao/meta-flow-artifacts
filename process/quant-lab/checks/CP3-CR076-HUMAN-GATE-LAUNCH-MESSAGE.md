---
check_id: "CP3-CR076-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T00:18:01+08:00"
checkpoint: "process/checkpoints/CP3-CR076-DATA-REPORTS-POLICY-REVIEW.md"
---

# CP3 CR076 Human Gate Launch Message

请审查：`process/checkpoints/CP3-CR076-DATA-REPORTS-POLICY-REVIEW.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP3-CR076-DATA-REPORTS-POLICY-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 CP3 Context Capsule、CR076 formal CR、自动预检和不授权边界；候选问题 15 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log 和 Architecture Gray Areas 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR076-01 | architecture | 是否采用 policy-first / no-data-touch？ | 采用，不访问数据。 | 直接真实访问；取消 CR076。 | 推荐方案权限最小。 | data/reports 事实仍未知。 |
| DQ-CP3-CR076-02 | architecture | 后续恢复路径如何建模？ | local retained、NAS archive、provider rebuild 并列候选。 | 仅保留其中一条路径。 | 推荐方案保留可选性。 | 后续计划需要再决策。 |
| DQ-CP3-CR076-03 | runtime_authorization | 后续真实访问是否逐 run 授权？ | 必须逐 run 明确授权。 | 一次性授权后续所有访问。 | 推荐方案风险可控。 | 降低越权风险。 |
| DQ-CP3-CR076-04 | security | 样本 / 清单证据是否脱敏且不写入 memory？ | 必须脱敏，不保存敏感样本或未脱敏日志。 | 允许保存完整清单或样本。 | 推荐方案符合 memory policy。 | 限制证据表达方式。 |
| DQ-CP3-CR076-05 | implementation | 失败 / 回退是否只回滚 policy ledger？ | 是，不做 cleanup。 | 失败时删除 / 清理数据或恢复旧根。 | 推荐方案不触碰数据。 | 保留现状不变。 |

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
