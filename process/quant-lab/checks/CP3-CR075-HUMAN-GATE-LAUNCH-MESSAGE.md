---
check_id: "CP3-CR075-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-16T23:31:26+08:00"
checkpoint: "process/checkpoints/CP3-CR075-ENV-SMOKE-DESIGN-REVIEW.md"
---

# CP3 CR075 Human Gate Launch Message

请审查：`process/checkpoints/CP3-CR075-ENV-SMOKE-DESIGN-REVIEW.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP3-CR075-ENV-SMOKE-DESIGN-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、自动预检、CR075 formal CR、pyproject.toml、CR-INDEX 和用户“启动CR075”语句；候选问题 11 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log、LLD clarification queue 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR075-01 | architecture | 是否采用基础依赖 + 最小 smoke 方案？ | 采用。 | no-op；optional groups；full tests。 | 推荐方案可证明最小可运行。 | 写入 `.venv`，运行本地 smoke。 |
| DQ-CP3-CR075-02 | implementation | dependency sync 命令是否固定为基础 `uv sync --python 3.11`？ | 固定基础 sync。 | 加 dev 到 sync；安装所有 groups。 | 推荐方案最小。 | 避免非必要 provider/ML/notebook 依赖。 |
| DQ-CP3-CR075-03 | implementation | smoke 子集是否只包含 import 和 CR061 package layout？ | 是。 | full pytest；章节研究 smoke。 | 推荐方案快且不碰数据。 | 只证明最小 runtime。 |
| DQ-CP3-CR075-04 | security | 是否禁止敏感和外部动作？ | 禁止。 | 允许读取 `.env` 或 data；允许 provider fetch。 | 推荐方案避免外部副作用。 | 不证明数据链路可用。 |
| DQ-CP3-CR075-05 | risk_acceptance | 失败后是否自动 cleanup / 重试？ | 不自动，记录 BLOCKED。 | 删除 `.venv` 后重试；切 full reinstall。 | 推荐方案避免破坏。 | 可能留下 partial `.venv`。 |

不授权项：

- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权读取、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权 provider fetch、lake write、catalog publish、远端 Git 写入、QMT/MiniQMT runtime 或 CR046 recovery。
- 不授权 optional groups、full tests、章节研究 smoke 或 `.venv` cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
