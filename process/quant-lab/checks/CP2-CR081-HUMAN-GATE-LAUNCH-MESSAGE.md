---
check_id: "CP2-CR081-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T08:03:08+08:00"
checkpoint: "process/checkpoints/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE.md"
---

# CP2 CR081 Human Gate Launch Message

请审查：`process/checkpoints/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、用户显式语句、CR081 formal CR、STATE / CR-INDEX、discussion log 和委托 Agent 交还摘要；候选问题 17 项，纳入本门待决策 5 项；N/A 来源原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP2-CR081-01 | scope | 是否正式启动 CR081？ | 启动 CR081，并按 high-risk standard 门禁处理。 | 保持讨论态；取消外部化路线。 | 推荐方案将目标纳入可审计 CR。 | 新增正式 CR 和待审门禁。 |
| DQ-CP2-CR081-02 | scope | 本轮第一阶段是否只迁移 `process/`？ | 第一阶段仅处理 `process/`，docs deferred。 | 同时迁移 docs；只做 mirror。 | 推荐方案风险最低。 | docs 过程证据暂留当前项目。 |
| DQ-CP2-CR081-03 | architecture | 外部过程项目是否成为 `process/**` owner？ | 是，本项目只保留本地入口。 | 只做外部备份；长期双写。 | 推荐方案职责清晰。 | clone 后需 bootstrap。 |
| DQ-CP2-CR081-04 | security | 是否继续禁止 data/reports/NAS 内容、凭据和 runtime？ | 继续禁止。 | 同时纳入 NAS 内容体检；同时恢复 CR046。 | 推荐方案最小权限。 | 不验证 NAS 内容或数据文件。 |
| DQ-CP2-CR081-05 | runtime_authorization | CP2 approve 是否授权执行迁移？ | 不授权，需 CP2/CP3/CP5 全部 approve。 | CP2 后立即执行；一次性授权。 | 推荐方案符合关键门控。 | 防止范围批准被误读为执行许可。 |

不授权项：

- 不授权创建外部过程项目、复制 / 移动 / 删除 `process/` 或创建 / 替换软链接。
- 不授权 `git rm --cached`、`git add`、`git commit`、`git push` 或远端治理。
- 不授权访问 NAS 内容、创建 mount、写 NAS、删除 NAS 文件或 NAS promote。
- 不授权读取 `.env`、凭据、data/reports 内容、样本解析、hash、provider/lake/catalog 或 runtime。
- 不授权恢复 CR046、旧根删除/移动或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
