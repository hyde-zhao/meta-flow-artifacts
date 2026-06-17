---
check_id: "CP2-CR079-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T05:56:16+08:00"
checkpoint: "process/checkpoints/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md"
---

# CP2 CR079 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP2-CR079-DATA-REPORTS-ACCESS-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CR079 formal CR、CR076 CP8、CR-INDEX follow-up、自动预检和用户“启动真实 data/reports access CR”语句；候选问题 10 项，纳入待决策 5 项；委托 Agent 交还摘要和 discussion log 为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR079-01 | scope | 是否正式启动 CR079？ | 启动 CR079，进入 CP3/CP5 真实访问门禁。 | 保持 candidate；取消 CR079。 | 推荐方案响应用户请求。 | 新增正式 CR 和待审门禁。 |
| DQ-CP2-CR079-02 | scope | 范围是否限定为 metadata-only 只读清点？ | 限定为存在性、顶层项、数量、规模、mtime 和风险标记。 | 纳入内容读取；恢复 / 删除。 | 推荐方案是最小真实访问。 | 不证明文件内容质量。 |
| DQ-CP2-CR079-03 | security | 是否继续禁止 `.env`、凭据、账户和交易事实？ | 继续禁止。 | 同时授权凭据读取或交易事实查询。 | 推荐方案最小权限。 | 防止凭据和交易事实泄露。 |
| DQ-CP2-CR079-04 | runtime_authorization | CP2 approve 是否单独授权执行访问？ | 不授权；必须三门全部 approved 且 preflight PASS。 | CP2 后立即执行；一次性授权全部后续访问。 | 推荐方案符合门禁。 | 避免 CP2 越权变成执行许可。 |
| DQ-CP2-CR079-05 | risk_acceptance | 是否接受 metadata-only 清点可能仍无法证明 readiness？ | 接受，并把内容质量 / 恢复路径作为后续风险。 | 扩大到内容读取或恢复；取消访问。 | 推荐方案先获得低风险事实。 | 内容可用性仍可能未知。 |

不授权项：

- 不授权 CP2 单独触发 `reports/`、`data/` 访问；必须 CP2 / CP3 / CP5 全部 approve。
- 不授权读取文件内容、打印样本、解析 parquet/csv/json、计算内容 hash、复制、恢复、比对内容或删除。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 provider fetch、lake write、catalog publish、NAS promote、远端 Git、QMT/MiniQMT runtime、CR046 recovery、旧根删除/移动、optional groups、full tests 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
