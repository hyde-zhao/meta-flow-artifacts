---
checkpoint_id: "CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS"
change_id: "CR-082"
checkpoint_type: "manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T09:11:43+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T09:32:20+08:00"
auto_precheck: "process/checks/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS.md"
context_capsule: "process/context/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS-CONTEXT.yaml"
auto_final_authorization: false
---

# CP5 CR082 Process Ledger Namespace Readiness

## 自动预检摘要

- 自动预检结论：PASS
- 自动预检文件：`process/checks/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS.md`
- Context Capsule：`process/context/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS-CONTEXT.yaml`
- 本检查点确认执行准备度；只有 CP2/CP3/CP5 全部 approve 后，才授权受控 preflight、namespace move、symlink update 和指针更新。仍不授权 Git commit/push、NAS 内容访问、data/reports 内容访问、凭据读取、runtime 或 cleanup。

## Decision Brief

推荐决策：批准 CR082 进入受控执行：先做 post-approval preflight，再将当前外部 ledger 内容移动到 `/home/hyde/workspace/process/quant-lab/`，更新 `/home/hyde/workspace/quant-lab/process -> ../process/quant-lab`，更新 root/project README 和指针文件，最后运行 symlink、bootstrap、CR tracking、human gate 和 scoped diff 校验。

备选方案：只生成设计不执行；只创建 `quant-lab/` 空目录；先做 CR078 remote governance。推荐方案优先，因为它完成用户当前提出的项目归档目标，并为 CR078 形成正确发布结构。

影响维度：文件路径、软链接、指针文件、README、CR tracking、已 staged 的 `process/**` removals 状态、后续 Git governance。

风险与回退：若 preflight 发现目标已存在且非预期、symlink 失败、指针文件验证失败或 CR tracking 校验失败，应停止并保持/恢复原 symlink；不删除 CR081 backup，不执行 Git commit/push。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP5 自动预检 | `process/checks/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS.md` | scanned | 5 | 5 | execution authorization、validation、rollback、follow-up 纳入。 |
| CP2/CP3 materials | CP2/CP3 checkpoint 草案 | scanned | 10 | 0 | 已在对应 gate 纳入，不重复。 |
| CR081 关闭风险 | `process/release/RELEASE-CONTEXT-CR081.yaml` | scanned | 3 | 2 | staged removals 和 backup 保留纳入。 |
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 10 | 0 | CP2/CP3 DQ 不重复纳入 CP5。 |
| discussion log / checkpoint | `process/discussions/*` | n/a | 0 | 0 | CR082 是受控迁移 readiness gate。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR082-01 | runtime_authorization | CP5 approve 后是否授权受控 namespace move？ | 授权 post-approval preflight 通过后执行 `process/* -> process/quant-lab/` 的受控移动。 | 只做 dry-run；不执行。 | 推荐方案完成用户目标；dry-run 会留下 flat root。 | 文件路径变化，需严格验证。 | preflight 失败则不执行。 |
| DQ-CP5-CR082-02 | implementation | 是否授权更新 quant-lab 项目 symlink？ | 授权将 `/home/hyde/workspace/quant-lab/process` 切换到 `../process/quant-lab`。 | 保持 `../process`；创建第二个 symlink。 | 推荐方案让项目入口直达项目目录；第二个 symlink 增加混乱。 | 影响日常查阅入口。 | link check 失败则恢复 `../process`。 |
| DQ-CP5-CR082-03 | implementation | 是否授权更新 README 和指针文件？ | 更新 root README、project README、`LEDGER.md`、`ledger.yaml`、`scripts/link-engineering-ledger.sh`。 | 只更新 README；只更新 symlink。 | 推荐方案消除路径漂移；部分更新会影响 clone 恢复。 | 影响后续 CR078 发布内容。 | 验证失败则回退指针。 |
| DQ-CP5-CR082-04 | risk_acceptance | 是否接受 staged `process/**` removals 在 CR082 后继续不提交？ | 接受继续保留 staged removals，等待 CR078 remote governance 统一处理。 | 立即 commit；restore staged removals。 | 推荐方案避免未授权远端写入；restore 会干扰外部化事实。 | Git status 仍会显示大量 staged deletions。 | 用户启动 CR078 后处理。 |
| DQ-CP5-CR082-05 | follow_up_tracking | CR082 完成后的下一步是否回到 CR078？ | CR082 CP8 关闭后优先启动 CR078 remote Git governance / publication gate。 | 启动 old root retirement；暂停全部后续 CR。 | 推荐方案处理当前最明显 Git 边界问题；old root retirement 仍未就绪。 | 后续仍需人工门禁。 | 用户指定其他优先级时调整。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权跳过 post-approval preflight 直接移动目录。
- 不授权 Git commit / push / remote write / branch governance。
- 不授权读取或修改 `data/`、`reports/` 内容。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS 内容访问、provider fetch、lake write、catalog publish。
- 不授权 QMT/MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live 或 CR046 recovery。
- 不授权 old root retirement、删除 external process backup、删除 `/home/hyde/workspace/process` 或 destructive cleanup。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 baseline ready | ready | `process/checkpoints/CP2-CR082-PROCESS-LEDGER-NAMESPACE-BASELINE.md` | 待同批确认。 |
| CP3 design ready | ready | `process/checkpoints/CP3-CR082-PROCESS-LEDGER-NAMESPACE-DESIGN-REVIEW.md` | 待同批确认。 |
| CP5 auto precheck PASS | ready | `process/checks/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS.md` | PASS。 |
| 执行前置边界明确 | ready | Decision Brief / 不授权项 | 不含 Git remote、data/reports、NAS、runtime。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | post-approval preflight 是否要求明确 | approved | DQ-CP5-CR082-01 | 用户回复“同意”，按 approve 处理。 |
| 2 | symlink update 是否明确 | approved | DQ-CP5-CR082-02 | 授权 preflight 通过后切换到 `../process/quant-lab`。 |
| 3 | 指针文件更新是否明确 | approved | DQ-CP5-CR082-03 | 授权更新 README、`LEDGER.md`、`ledger.yaml` 和 bootstrap 脚本。 |
| 4 | staged removals 风险是否接受 | approved | DQ-CP5-CR082-04 | 继续保留 staged removals，等待 CR078。 |
| 5 | 后续 CR 分流是否明确 | approved | DQ-CP5-CR082-05 | CR082 CP8 后优先 CR078。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户回复 `approve` / `修改: <具体修改点>` / `reject` | approved | 当前对话：用户回复“同意” | 按 approve 处理。 |
| approve 后可执行受控 namespace move | approved | 本文件 | 授权 post-approval preflight 通过后的受控 namespace move。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 context | `process/context/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS-CONTEXT.yaml` | approved | ready。 |
| CP5 auto precheck | `process/checks/CP5-CR082-PROCESS-LEDGER-NAMESPACE-READINESS.md` | approved | PASS。 |
| CP5 launch message | `process/checks/CP5-CR082-HUMAN-GATE-LAUNCH-MESSAGE.md` | approved | human gate check PASS。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T09:32:20+08:00
- 修改意见：无；用户回复“同意”按 approve 处理。
- 风险接受项：接受 DQ-CP5-CR082-01..05；授权 post-approval preflight 通过后的受控 namespace move、symlink update 和 pointer update；仍不授权自动终验、Git commit/push、remote、data/reports、凭据、NAS 内容、runtime、CR046、backup 删除或 cleanup。
