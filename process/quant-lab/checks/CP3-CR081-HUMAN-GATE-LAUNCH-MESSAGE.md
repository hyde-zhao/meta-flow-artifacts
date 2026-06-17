---
check_id: "CP3-CR081-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T08:03:08+08:00"
checkpoint: "process/checkpoints/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-REVIEW.md"
---

# CP3 CR081 Human Gate Launch Message

请审查：`process/checkpoints/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-REVIEW.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONTEXT.yaml`，状态 ready-for-human-gate，read_profile=compact。

决策收集覆盖：已扫描 STATE pending queue、用户显式语句、CR081 formal CR、`.gitignore`、CR tracking 脚本和 discussion log；候选问题 15 项，纳入本门待决策 5 项；N/A 来源原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR081-01 | architecture | 外部过程项目路径模型如何定义？ | `${ENGINEERING_LEDGER_ROOT:-/home/hyde/workspace/engineering-ledger}/projects/quant-lab/process`。 | 固定 NAS 绝对路径；固定 sibling 相对路径。 | 推荐方案可本地也可 NAS。 | bootstrap 需要默认路径或 env。 |
| DQ-CP3-CR081-02 | architecture | 本项目是否跟踪 `process` symlink？ | 不跟踪；用 `.gitignore` 忽略 symlink。 | 跟踪 symlink；使用 submodule。 | 推荐方案不暴露私有路径。 | clone 后需 bootstrap。 |
| DQ-CP3-CR081-03 | implementation | 过程项目是 Git 还是 NAS？ | 推荐 Git worktree，可放 NAS 或用 NAS 备份。 | 仅 NAS 无 Git；仅本地无 NAS。 | Git 提供审计，NAS 提供集中存储。 | 管理由过程项目负责。 |
| DQ-CP3-CR081-04 | implementation | 新环境如何恢复 `process/`？ | 新增 `LEDGER.md`、`ledger.yaml`、link script。 | 手工建链接；仅 README 说明。 | 推荐方案可验证。 | 增加少量项目文件。 |
| DQ-CP3-CR081-05 | scope | 是否同时迁移 docs 过程目录？ | 不迁移，先只处理 `process/`。 | 同时迁移所有 docs；只迁 quality/release。 | 推荐方案 blast radius 小。 | docs 仍占项目仓空间。 |

不授权项：

- 不授权创建外部过程项目、复制 / 移动 / 删除 `process/` 或创建 / 替换软链接。
- 不授权 `git rm --cached`、`git add`、`git commit`、`git push` 或远端治理。
- 不授权 NAS 内容读取、写入、删除、mount 或 promote。
- 不授权迁移 `docs/design`、`docs/features`、`docs/quality`、`docs/release`。
- 不授权数据、凭据、runtime、CR046 recovery、old root retirement 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
