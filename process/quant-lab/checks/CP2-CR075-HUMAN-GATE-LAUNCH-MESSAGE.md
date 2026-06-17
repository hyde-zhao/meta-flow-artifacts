---
check_id: "CP2-CR075-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-16T23:31:26+08:00"
checkpoint: "process/checkpoints/CP2-CR075-ENV-REBUILD-BASELINE.md"
---

# CP2 CR075 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR075-ENV-REBUILD-BASELINE.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP2-CR075-ENV-REBUILD-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、自动预检、CR075 formal CR、CR073/CR074 follow-up、CR-INDEX 和用户“启动CR075”语句；候选问题 11 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log、LLD clarification queue 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR075-01 | scope | 是否确认正式启动 CR075？ | 启动 CR075，进入 CP3/CP5 门禁。 | 保持候选；取消 CR075。 | 推荐方案响应用户启动请求。 | 新增正式 CR 和状态索引。 |
| DQ-CP2-CR075-02 | scope | CR075 是否只覆盖 target env rebuild 和最小 smoke？ | 是，只覆盖基础依赖、import smoke、package layout 小回归。 | 扩大到 data/reports；扩大到 optional groups。 | 推荐方案最小可审计。 | target 最小运行可被验证。 |
| DQ-CP2-CR075-03 | security | 是否继续禁止敏感和外部动作？ | 继续禁止。 | 合并 CR076/CR078 或 runtime。 | 推荐方案边界清晰。 | 避免凭据、数据和外部写入风险。 |
| DQ-CP2-CR075-04 | implementation | CP2 approve 是否授权立即执行 `uv sync`？ | 不授权；需三门 approved。 | CP2 后立即执行。 | 推荐方案符合门禁。 | 避免过早写入 `.venv`。 |
| DQ-CP2-CR075-05 | risk_acceptance | 是否接受 target 当前仍非 runtime-ready？ | 接受，等待 CR075 smoke 判定。 | 先不启动；直接 full tests。 | 推荐方案分阶段。 | target 仍可能无法运行。 |

不授权项：

- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权读取、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权 provider fetch、lake write、catalog publish、远端 Git 写入、QMT/MiniQMT runtime 或 CR046 recovery。
- 不授权 CP2 后立即执行 `uv sync` 或 smoke。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
