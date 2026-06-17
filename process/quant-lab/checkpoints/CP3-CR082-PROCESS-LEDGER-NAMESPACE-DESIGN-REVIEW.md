---
checkpoint_id: "CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-REVIEW"
change_id: "CR-082"
checkpoint_type: "manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T09:11:43+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T09:32:20+08:00"
auto_precheck: "process/checks/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONTEXT.yaml"
auto_final_authorization: false
---

# CP3 CR082 Process Ledger Namespace Design Review

## 自动预检摘要

- 自动预检结论：PASS
- 自动预检文件：`process/checks/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONSISTENCY.md`
- Context Capsule：`process/context/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONTEXT.yaml`
- 本检查点只确认 namespace 设计；在用户 approve 前，不授权目录移动、软链接替换、Git commit/push、NAS 内容访问、data/reports 内容访问、凭据读取、runtime 或 cleanup。

## Decision Brief

推荐决策：采用 “workspace-level process project + project namespace” 设计。外部 ledger 根 `/home/hyde/workspace/process` 作为多项目容器，`/home/hyde/workspace/process/quant-lab` 作为 quant-lab 的过程文件根；主项目入口 `/home/hyde/workspace/quant-lab/process` 指向 `../process/quant-lab`。

备选方案：继续把 `/home/hyde/workspace/process` 当作 quant-lab 专属根；采用 `/home/hyde/workspace/process/projects/quant-lab`；或为每个项目单独建平级 `process-<project>`。推荐方案优先，因为它满足用户“process 目录中分项目归档”的目标，同时不引入额外路径层级。

影响维度：外部 process README、quant-lab project README、`process` symlink、`LEDGER.md`、`ledger.yaml`、`scripts/link-engineering-ledger.sh`、CR tracking 与后续 CR078。

风险与回退：设计风险集中在路径引用残留和 clone 后恢复说明。回退方式是保留 root README 和 CR081 backup，执行验证失败则不关闭 CR082，并恢复 symlink 到 `../process`。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP3 自动预检 | `process/checks/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONSISTENCY.md` | scanned | 5 | 5 | 结构、入口、指针、README、future project convention 纳入。 |
| CR081 指针文件 | `LEDGER.md`; `ledger.yaml`; `scripts/link-engineering-ledger.sh` | scanned | 4 | 2 | 需要在执行阶段更新到 project namespace。 |
| 当前外部 README | `/home/hyde/workspace/process/README.md` | scanned | 2 | 1 | 需从 quant-lab 专属说明改为 workspace ledger container 说明。 |
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 5 | 0 | CP2 DQ 不重复纳入 CP3。 |
| discussion log / checkpoint | `process/discussions/*` | n/a | 0 | 0 | CR082 是结构治理 gate，不需要架构灰区多轮讨论。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR082-01 | architecture | external process project 是否采用多项目容器结构？ | `/home/hyde/workspace/process` 作为容器，`quant-lab/` 作为项目目录。 | flat root；`projects/quant-lab`。 | 推荐方案直接满足分项目归档；flat root 不满足目标。 | 后续项目可同级归档。 | 用户指定更深层路径时重写设计。 |
| DQ-CP3-CR082-02 | architecture | `quant-lab` 内是否承载当前全部过程目录？ | 将当前 `STATE.md`、`changes/`、`checks/`、`checkpoints/` 等搬入 `quant-lab/`。 | 只搬 `changes/`；只新增 README。 | 推荐方案保持 process 入口语义完整；部分搬迁会产生双源。 | 影响所有 `process/...` 相对路径。 | 验证失败时回退 flat root。 |
| DQ-CP3-CR082-03 | implementation | 主项目 `process` 软链接目标是否切到 `../process/quant-lab`？ | 执行阶段原子替换为 `../process/quant-lab`。 | 保持 `../process`；使用绝对 symlink。 | 推荐方案保持项目入口稳定且可移植；绝对链接迁移性差。 | 影响脚本和人工查阅入口。 | link check 失败时恢复 `../process`。 |
| DQ-CP3-CR082-04 | implementation | 指针文件如何更新？ | 更新 `LEDGER.md`、`ledger.yaml`、`scripts/link-engineering-ledger.sh` 到 `process/quant-lab`。 | 只更新 README；暂不更新脚本。 | 推荐方案保证 clone 恢复可用；只更新 README 会留下脚本漂移。 | 影响后续 bootstrap。 | 验证脚本失败则回退指针。 |
| DQ-CP3-CR082-05 | follow_up_tracking | 是否定义 future project namespace 约定？ | root README 说明未来项目使用 `/home/hyde/workspace/process/<project-name>/`，由各项目各自软链进入。 | 不定义；等下一个项目再决定。 | 推荐方案避免再次变更；不定义会重复讨论。 | 不自动创建其他项目目录。 | 未来项目有特殊要求时另行覆盖。 |

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
| CP2 baseline 已生成 | ready | `process/checkpoints/CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE.md` | 待同批确认。 |
| 当前结构已盘点 | ready | `process/checks/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONSISTENCY.md` | flat root，无 `quant-lab/`。 |
| 目标设计已描述 | ready | Decision Brief | 待用户确认。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 架构目标是否明确 | approved | DQ-CP3-CR082-01 | 用户回复“同意”，按 approve 处理。 |
| 2 | 项目目录承载范围是否明确 | approved | DQ-CP3-CR082-02 | 当前全部过程目录纳入 `quant-lab/`。 |
| 3 | symlink 目标是否明确 | approved | DQ-CP3-CR082-03 | 目标为 `../process/quant-lab`。 |
| 4 | 指针文件更新范围是否明确 | approved | DQ-CP3-CR082-04 | 覆盖 README、`LEDGER.md`、`ledger.yaml` 和 bootstrap 脚本。 |
| 5 | 不授权项是否完整 | approved | `### 不授权项` | 用户接受最小授权边界。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户回复 `approve` / `修改: <具体修改点>` / `reject` | approved | 当前对话：用户回复“同意” | 按 approve 处理。 |
| approve 后可进入 CP5 readiness 同批确认 | approved | 本文件 | CP3 批准成立。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 context | `process/context/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONTEXT.yaml` | approved | ready。 |
| CP3 auto precheck | `process/checks/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-CONSISTENCY.md` | approved | PASS。 |
| CP3 launch message | `process/checks/CP3-CR082-HUMAN-GATE-LAUNCH-MESSAGE.md` | approved | human gate check PASS。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T09:32:20+08:00
- 修改意见：无；用户回复“同意”按 approve 处理。
- 风险接受项：接受 DQ-CP3-CR082-01..05；本 checkpoint 不授权自动终验，执行仍需 CP5 同批批准和 post-approval preflight。
