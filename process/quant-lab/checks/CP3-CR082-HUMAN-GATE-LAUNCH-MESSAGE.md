---
check_id: "CP3-CR082-HUMAN-GATE-LAUNCH-MESSAGE"
type: "human_gate_launch_message"
status: "draft"
owner: "host-orchestrator"
created_at: "2026-06-17T09:11:43+08:00"
checkpoint: "process/checkpoints/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-REVIEW.md"
---

# CP3 CR082 Human Gate Launch Message

请审查：`process/checkpoints/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-REVIEW.md`

自动预检结论：PASS，阻断项 0。

Context Capsule：`process/context/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONTEXT.yaml`，状态 ready，read_profile=compact。

决策收集覆盖：已扫描 CP3 自动预检、CR081 指针文件、当前外部 README、STATE pending queue；候选问题 11 项，纳入待决策 5 项；discussion log 为 N/A，原因见 checkpoint 的 Decision Collection Coverage。

本轮待人工决策项：5。

如果你回复 approve，表示你接受以下 5 项推荐方案，不表示授权下方不授权项。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR082-01 | architecture | external process project 是否采用多项目容器结构？ | `/home/hyde/workspace/process` 作为容器，`quant-lab/` 作为项目目录。 | flat root；`projects/quant-lab`。 | 推荐方案直接满足分项目归档。 | 后续项目可同级归档。 |
| DQ-CP3-CR082-02 | architecture | `quant-lab` 内是否承载当前全部过程目录？ | 将当前 `STATE.md`、`changes/`、`checks/`、`checkpoints/` 等搬入 `quant-lab/`。 | 只搬 `changes/`；只新增 README。 | 推荐方案保持 process 入口语义完整。 | 影响所有 `process/...` 相对路径。 |
| DQ-CP3-CR082-03 | implementation | 主项目 `process` 软链接目标是否切到 `../process/quant-lab`？ | 执行阶段原子替换为 `../process/quant-lab`。 | 保持 `../process`；使用绝对 symlink。 | 推荐方案保持项目入口稳定且可移植。 | 影响脚本和人工查阅入口。 |
| DQ-CP3-CR082-04 | implementation | 指针文件如何更新？ | 更新 `LEDGER.md`、`ledger.yaml`、`scripts/link-engineering-ledger.sh` 到 `process/quant-lab`。 | 只更新 README；暂不更新脚本。 | 推荐方案保证 clone 恢复可用。 | 影响后续 bootstrap。 |
| DQ-CP3-CR082-05 | follow_up_tracking | 是否定义 future project namespace 约定？ | root README 说明未来项目使用 `/home/hyde/workspace/process/<project-name>/`。 | 不定义；等下一个项目再决定。 | 推荐方案避免再次变更。 | 不自动创建其他项目目录。 |

不授权项：

- 不授权在 CP2/CP3/CP5 全部 approve 前移动 `/home/hyde/workspace/process` 内容或替换 `process` 软链接。
- 不授权 Git commit / push / remote write / branch governance。
- 不授权读取或修改 `data/`、`reports/` 内容。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS 内容访问、provider fetch、lake write、catalog publish。
- 不授权 QMT/MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live 或 CR046 recovery。
- 不授权 old root retirement、删除 external process backup、删除 `/home/hyde/workspace/process` 或 destructive cleanup。

推荐回复：

```text
approve
修改: <具体修改点>
reject
```
