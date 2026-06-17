---
check_id: "CP3-CR080-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T06:42:48+08:00"
checkpoint: "process/checkpoints/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-REVIEW.md"
---

# CP3 CR080 Human Gate Launch Message

请审查：`process/checkpoints/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-REVIEW.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、CR080 formal CR、CR079 inventory、CP3 自动预检和当前用户语句；候选问题 25 项，纳入本门待决策 5 项；discussion log 和委托 Agent 交还摘要为 N/A，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR080-01 | architecture | 是否采用本地 copy-first？ | legacy retained assets -> target root。 | NAS compare；provider rebuild。 | 推荐方案最快补齐本地 root。 | legacy 不足时需后续 CR。 |
| DQ-CP3-CR080-02 | implementation | 命令族是否收敛？ | `test/mkdir/find/stat/du/rsync`，禁 delete/checksum/远端。 | `cp -a`；checksum；delete。 | 推荐方案可审计且防误删。 | 不证明内容一致性。 |
| DQ-CP3-CR080-03 | security | credential-like path 如何处理？ | denylist 排除并脱敏计数。 | 全量复制；复制后清理。 | 推荐方案先避免敏感复制。 | 可能少复制敏感命名文件。 |
| DQ-CP3-CR080-04 | risk_acceptance | target 非空是否 fail-closed？ | 非空/冲突即 BLOCKED。 | 自动合并；覆盖。 | 推荐方案保守。 | target 已有部分内容时会停下。 |
| DQ-CP3-CR080-05 | implementation | 失败是否禁止自动 cleanup？ | 禁止，记录 partial 后人工决策。 | 自动删除；自动隔离。 | 推荐方案避免破坏性操作。 | 可能留下 partial。 |

不授权项：

- 不授权内容读取、样本打印、文件解析、内容 hash、checksum 校验或内容比对。
- 不授权 `rsync --delete`、远端 rsync、覆盖已有非空 target、自动 merge、自动 cleanup。
- 不授权复制 credential-like path；命中项只能脱敏计数。
- 不授权 NAS/provider/lake/catalog、remote Git、runtime、CR046、old root retirement 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
