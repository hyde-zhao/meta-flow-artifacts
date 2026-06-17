---
check_id: "CP5-CR075-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-16T23:31:26+08:00"
checkpoint: "process/checkpoints/CP5-CR075-ENV-SMOKE-READINESS.md"
---

# CP5 CR075 Human Gate Launch Message

请审查：`process/checkpoints/CP5-CR075-ENV-SMOKE-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP5-CR075-ENV-SMOKE-READINESS-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、自动预检、CR075 formal CR、smoke test file、CR-INDEX 和用户“启动CR075”语句；候选问题 11 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log、LLD clarification queue 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR075-01 | implementation | CP5 approved 后是否允许执行 CR075 env rebuild？ | 允许，但需三门 approved 且 preflight PASS。 | 只保留 gate；只做人工说明。 | 推荐方案完成 target runtime readiness 验证。 | 写入 `.venv` 并下载依赖。 |
| DQ-CP5-CR075-02 | implementation | 执行前 preflight 如何处理？ | 重跑 CR tracking，并确认不授权项未扩大。 | 直接执行。 | 推荐方案可审计。 | 防止状态分叉和越权。 |
| DQ-CP5-CR075-03 | implementation | smoke 命令是否固定为三条最小命令？ | 固定为 `uv sync`、import smoke、CR061 package layout pytest。 | 只 import；full tests。 | 推荐方案平衡覆盖和风险。 | 只证明最小 target runtime。 |
| DQ-CP5-CR075-04 | security | 执行时是否仍禁止敏感和外部动作？ | 继续禁止。 | 合并 CR076/CR078 或 runtime。 | 推荐方案边界清楚。 | 不证明数据链路或交易链路。 |
| DQ-CP5-CR075-05 | risk_acceptance | partial `.venv` 或 smoke failure 是否自动清理？ | 不自动清理，记录并等待授权。 | 自动删除 `.venv`；自动重试。 | 推荐方案避免破坏。 | 可能留下 partial env。 |

不授权项：

- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权读取、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权 provider fetch、lake write、catalog publish、远端 Git 写入、QMT/MiniQMT runtime 或 CR046 recovery。
- 不授权 optional groups、full tests、章节研究 smoke、删除 `.venv` 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
