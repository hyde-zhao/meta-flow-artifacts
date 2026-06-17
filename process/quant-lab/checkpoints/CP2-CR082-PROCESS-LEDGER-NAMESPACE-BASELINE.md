---
checkpoint_id: "CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE"
change_id: "CR-082"
checkpoint_type: "manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T09:11:43+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T09:32:20+08:00"
auto_precheck: "process/checks/CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE.md"
context_capsule: "process/context/CP2-CR082-PROCESS-LEDGER-NAMESPACE-CONTEXT.yaml"
auto_final_authorization: false
---

# CP2 CR082 Process Ledger Namespace Baseline

## 自动预检摘要

- 自动预检结论：PASS
- 自动预检文件：`process/checks/CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE.md`
- Context Capsule：`process/context/CP2-CR082-PROCESS-LEDGER-NAMESPACE-CONTEXT.yaml`
- 本检查点只确认是否启动 CR082 project namespace normalization；在用户 approve 前，不授权目录移动、软链接替换、Git commit/push、NAS 内容访问、data/reports 内容访问、凭据读取、runtime 或 cleanup。

## Decision Brief

推荐决策：启动 CR082，将外部 process ledger 从当前 flat root `/home/hyde/workspace/process` 规范为按项目归档的 `process/quant-lab/`，并在后续批准执行时把 `/home/hyde/workspace/quant-lab/process` 软链接切换为 `../process/quant-lab`。

备选方案：保持 flat root 不调整；先启动 CR078 remote Git governance 后再调整 namespace；仅更新 README 说明而不移动文件。推荐方案优先，因为当前 process ledger 已外部化但尚未做 Git 远端治理，先补项目命名空间可以避免把 flat root 结构固化到后续远端仓库。

影响维度：external process project root、`process` symlink target、CR tracking files、README / pointer files、staged `process/**` removals、后续 CR078 remote governance。

风险与回退：主要风险是移动后软链接或脚本路径不一致。回退方式是在执行阶段保留 flat root staging / backup 证据，若验证失败则恢复 `process -> ../process` 并不进入 CR078。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR082-PROCESS-LEDGER-NAMESPACE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| 当前用户语句 | 当前对话 | scanned | 2 | 2 | 用户询问是否按项目归档并回复继续，纳入 scope / ordering。 |
| CR081 关闭证据 | `process/checkpoints/CP8-CR081-DELIVERY-READINESS.md` | scanned | 5 | 2 | external ledger 已完成，remote governance 未启动。 |
| 当前目录结构 | `/home/hyde/workspace/process` | scanned | 4 | 3 | 发现 flat root，无 `quant-lab/` 项目层。 |
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | CR081 已关闭；无 CR082 既有 pending 决策。 |
| discussion log / checkpoint | `process/discussions/*` | n/a | 0 | 0 | CR082 是 ledger namespace gate，不需要场景灰区讨论。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR082-01 | scope | 是否正式启动 CR082 project namespace normalization？ | 启动 CR082，范围限定为 external process ledger 项目命名空间和指针更新。 | 不启动；合并到 CR078。 | 推荐方案先修正结构边界；合并到 CR078 会把结构治理和远端治理耦合。 | CR082 将占用当前下一步执行队列。 | reject 时保持 CR081 flat root 结果。 |
| DQ-CP2-CR082-02 | scope | 项目归档目标目录是否采用 `/home/hyde/workspace/process/quant-lab`？ | 采用 `process/quant-lab/` 作为 quant-lab 项目过程文件根。 | 采用 `process/projects/quant-lab/`；继续 flat root。 | 推荐方案短路径、与用户示例一致；备选更通用但多一层。 | 影响 symlink、README 和恢复脚本路径。 | 用户指定其他命名空间时重写路径。 |
| DQ-CP2-CR082-03 | implementation | 现有 flat root 如何处理？ | 执行阶段仅搬入 `quant-lab/` 并保留必要 root README；不删除 CR081 backup。 | 保留 flat root 内容副本；立即清理旧层。 | 推荐方案减少重复且保留回退材料；立即清理风险高。 | 可能需要更新 root README 防止误读。 | 验证失败时恢复 flat root symlink。 |
| DQ-CP2-CR082-04 | follow_up_tracking | CR082 与 CR078 的顺序如何安排？ | CR082 先于 CR078，完成 namespace 后再进入 remote Git governance。 | 先做 CR078；取消 CR078。 | 推荐方案避免远端固化 flat root；先 CR078 会增加后续迁移成本。 | CR078 延后，但边界更清晰。 | 用户要求直接发布时切回 CR078。 |
| DQ-CP2-CR082-05 | security | CP2 approve 是否不授权 data/reports、凭据、NAS 内容、remote write、runtime 和 cleanup？ | 继续不授权，相关事项均需独立 CR 或明确运行授权。 | 同时授权 Git commit/push；同时授权 NAS 管理。 | 推荐方案保持最小权限；备选扩大外部写入面。 | 防止数据、凭据、远端和 runtime 风险。 | 用户另起具体操作 CR 时切换。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权在 CP2/CP3/CP5 全部 approve 前移动 `/home/hyde/workspace/process` 内容或替换 `process` 软链接。
- 不授权 Git commit / push / remote write / branch governance。
- 不授权读取或修改 `data/`、`reports/` 内容。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS 内容访问、provider fetch、lake write、catalog publish。
- 不授权 QMT/MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live 或 CR046 recovery。
- 不授权 old root retirement、删除 external process backup、删除 `/home/hyde/workspace/process` 或 destructive cleanup。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR081 已关闭当前交付 | ready | `process/checkpoints/CP8-CR081-DELIVERY-READINESS.md` | READY_WITH_RISK 已关闭。 |
| 当前 external process root 存在 | ready | `/home/hyde/workspace/process` | 已存在 flat root。 |
| 当前项目 symlink 存在 | ready | `/home/hyde/workspace/quant-lab/process -> ../process` | 后续可切换目标。 |
| CR082 正式 CR 已创建 | ready | `process/changes/CR-082-PROCESS-LEDGER-PROJECT-NAMESPACE-NORMALIZATION-2026-06-17.md` | 进入人工确认。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 变更范围是否清楚 | approved | Decision Brief | 用户回复“同意”，按 approve 处理。 |
| 2 | 目标目录是否明确 | approved | DQ-CP2-CR082-02 | 采用 `/home/hyde/workspace/process/quant-lab`。 |
| 3 | 与 CR078 顺序是否明确 | approved | DQ-CP2-CR082-04 | CR082 先于 CR078。 |
| 4 | 不授权项是否完整 | approved | `### 不授权项` | 用户接受最小授权边界。 |
| 5 | 回退条件是否存在 | approved | Decision Brief | 保留 CR081 backup 与 symlink 回退路径。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户回复 `approve` / `修改: <具体修改点>` / `reject` | approved | 当前对话：用户回复“同意” | 按 approve 处理。 |
| approve 后可进入 CP3/CP5 同批确认 | approved | 本文件 | CP2 批准成立。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR082 formal CR | `process/changes/CR-082-PROCESS-LEDGER-PROJECT-NAMESPACE-NORMALIZATION-2026-06-17.md` | approved | 用户确认启动。 |
| CP2 context | `process/context/CP2-CR082-PROCESS-LEDGER-NAMESPACE-CONTEXT.yaml` | approved | ready。 |
| CP2 auto precheck | `process/checks/CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE.md` | approved | PASS。 |
| CP2 launch message | `process/checks/CP2-CR082-HUMAN-GATE-LAUNCH-MESSAGE.md` | approved | human gate check PASS。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T09:32:20+08:00
- 修改意见：无；用户回复“同意”按 approve 处理。
- 风险接受项：接受 DQ-CP2-CR082-01..05；本 checkpoint 不授权自动终验，执行仍需 CP5 同批批准和 post-approval preflight。
