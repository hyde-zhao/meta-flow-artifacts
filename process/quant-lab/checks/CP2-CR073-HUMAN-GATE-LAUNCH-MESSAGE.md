---
check_id: "CP2-CR073-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-16T10:07:45+08:00"
checkpoint: "process/checkpoints/CP2-CR073-TARGET-EVIDENCE-BASELINE.md"
---

# CP2 CR073 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR073-TARGET-EVIDENCE-BASELINE.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP2-CR073-REQUIREMENT-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、自动预检、CR073 formal CR、diff check、CR071/CR072 上游证据和用户显式启动语句；候选问题 16 项，纳入待决策 5 项；委托 Agent 交还摘要、discussion log、LLD clarification queue 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR073-01 | scope | 是否正式启动 CR073 / MIG-B？ | 启动为正式 active gate，CR046 仍 paused。 | 保持候选；取消 CR073。 | 推荐方案补齐 target 证据；取消保留审计缺口。 | 更新 STATE / CR-INDEX，不触发 runtime。 |
| DQ-CP2-CR073-02 | scope | 本轮范围是否限定为 target evidence convergence / cutover readiness？ | 限定为证据收敛，不做 root cutover。 | 直接 cutover；只报告不收敛。 | 推荐方案先消除证据分叉；直接 cutover 风险过大。 | target 只成为候选根。 |
| DQ-CP2-CR073-03 | security | 是否确认 forbidden paths 和凭据边界？ | 排除 `.git`、`reports/`、`data/`、`.venv/`、`node_modules/`、`.env`，不读取凭据。 | 允许同步部分报告/数据；允许读取 `.env` 校验。 | 推荐方案安全面最小；备选扩大敏感面。 | target 不 runtime-ready。 |
| DQ-CP2-CR073-04 | runtime_authorization | CP2 approve 是否授权外部动作或 CR046 recovery？ | 不授权。 | 合并某类外部动作；恢复 CR046。 | 推荐方案保持本机静态范围；备选引入外部副作用。 | 防止越权。 |
| DQ-CP2-CR073-05 | risk_acceptance | 是否接受 target 只达到 cutover readiness？ | 接受。 | 同时重建 env；同时恢复 data/reports。 | 推荐方案证据先行；备选扩大范围。 | 后续运行 target 仍需单独门禁。 |

不授权项：

- 不授权 root cutover 或切换日常工作根。
- 不授权删除、重命名或移动旧根。
- 不授权复制、读取或比对 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env`。
- 不授权读取、打印或迁移凭据。
- 不授权远端 Git 写入、NAS 新写入、data lake、QMT/MiniQMT runtime 或 CR046 recovery。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
