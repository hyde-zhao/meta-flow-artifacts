---
check_id: "CP2-CR074-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-16T22:14:44+08:00"
checkpoint: "process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md"
---

# CP2 CR074 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、自动预检、CR074 formal CR、CR073 readiness evidence、CR-INDEX follow-up candidate 和用户“继续”语句；候选问题 12 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log、LLD clarification queue 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR074-01 | scope | 是否正式启动 CR074 root cutover / daily workflow switch？ | 启动正式 active gate。 | 保持 candidate；取消 CR074。 | 推荐方案承接 CR073 readiness；备选继续停留旧根。 | 更新状态，不触发切换。 |
| DQ-CP2-CR074-02 | scope | 本轮 cutover 是否限定为 logical root authority？ | 仅做 logical root authority。 | 物理移动/重命名；系统级 symlink。 | 推荐方案可回退；备选破坏性更高。 | 后续会话入口语义变化。 |
| DQ-CP2-CR074-03 | security | 是否确认旧根保留且 forbidden paths 不触碰？ | 保留旧根，不复制/读取 forbidden paths。 | 迁移部分 forbidden paths；删除旧根。 | 推荐方案保留回退路径；备选扩大风险。 | target 仍可能非 runtime-ready。 |
| DQ-CP2-CR074-04 | runtime_authorization | CP2 approve 是否授权 env rebuild、远端 Git、data lake、runtime 或 CR046？ | 不授权。 | 合并外部动作；恢复 CR046。 | 推荐方案聚焦 root authority。 | 防止越权。 |
| DQ-CP2-CR074-05 | risk_acceptance | 是否接受 target authoritative 但仍非 runtime-ready？ | 接受；CR075/CR076 后续处理。 | 等 CR075/CR076 后再 cutover。 | 推荐方案更快完成入口治理；备选更保守。 | 后续运行仍需验证。 |

不授权项：

- 不授权当前会话自动切换 cwd、shell / IDE / cron / service 修改。
- 不授权删除、重命名或移动旧根。
- 不授权复制、读取或比对 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env`。
- 不授权读取、打印或迁移凭据。
- 不授权 target env rebuild、远端 Git 写入、NAS 新写入、data lake、QMT/MiniQMT runtime 或 CR046 recovery。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
