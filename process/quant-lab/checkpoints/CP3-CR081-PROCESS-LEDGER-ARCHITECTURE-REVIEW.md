---
checkpoint_id: "CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-REVIEW"
checkpoint_name: "CR081 Process Ledger Architecture Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T08:03:08+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T08:36:45+08:00"
auto_check_result: "process/checks/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "solution-design"
  artifacts:
    - "process/changes/CR-081-PROCESS-LEDGER-EXTERNALIZATION-SYMLINK-CUTOVER-2026-06-17.md"
---

# CP3 CR081 Process Ledger Architecture Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONSISTENCY.md` | PASS | 0 | 推荐架构为 external ledger owner + ignored local symlink + bootstrap。 |

## Decision Brief

推荐决策：采用外部过程项目作为 `process/**` 的唯一 owner，本项目使用 ignored local symlink `process -> ${ENGINEERING_LEDGER_ROOT}/projects/quant-lab/process`。默认 ledger root 为 `/home/hyde/workspace/engineering-ledger`，但通过 `ENGINEERING_LEDGER_ROOT` 可切换到 NAS 或其他路径。

备选方案：跟踪 symlink 本身；使用 Git submodule；只做 mirror 不 cutover；改造 Meta Flow 支持 `PROCESS_ROOT` 配置。推荐方案优先，因为它满足用户的“本项目软链接过去”意图，同时避免 Git 记录私有绝对路径。

影响维度：symlink 可用性、clone/bootstrap、GitHub 展示、Meta Flow 固定路径、NAS 离线、脚本跟随 symlink 风险、跨项目 ledger 隔离。

风险与回退：如果 symlink 断链，bootstrap 重新创建；如果 external ledger 不可用，回退到本地 `process` backup；如果 Meta Flow 固定路径不兼容，停止 cutover 并保留 mirror。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| discussion | N/A：本 CR 为仓库/过程架构变更，用户已给出架构意图。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 5 | 0 | CP2 DQ 不重复纳入 CP3。 |
| 用户显式语句 | 当前对话 | scanned | 5 | 5 | Git/NAS、软链接、过程项目隔离纳入架构 DQ。 |
| CR081 formal CR | `process/changes/CR-081-PROCESS-LEDGER-EXTERNALIZATION-SYMLINK-CUTOVER-2026-06-17.md` | scanned | 7 | 5 | owner、symlink、bootstrap、docs deferred、storage 模式纳入。 |
| `.gitignore` | `.gitignore` | scanned | 2 | 1 | 需增加 ignored symlink 规则，执行阶段处理。 |
| scripts | `scripts/check_cr_tracking_consistency.py` | scanned | 1 | 1 | 固定 `process/STATE.md` 路径需 symlink 保持透明。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 无新增产品架构讨论。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR081-01 | architecture | 外部过程项目路径模型如何定义？ | 默认 `${ENGINEERING_LEDGER_ROOT:-/home/hyde/workspace/engineering-ledger}/projects/quant-lab/process`。 | 固定 NAS 绝对路径；固定 sibling 相对路径。 | 推荐方案可本地也可 NAS；固定路径更简单但不便移植。 | bootstrap 需要环境变量或默认路径。 | 用户指定 NAS 路径时切换。 |
| DQ-CP3-CR081-02 | architecture | 本项目是否跟踪 `process` symlink 本身？ | 不跟踪；`.gitignore` 忽略 `process` symlink，本项目只跟踪 bootstrap 和说明。 | 跟踪 symlink；使用 submodule。 | 推荐方案不暴露私有路径；跟踪 symlink 更直观但易断链。 | clone 后需运行 bootstrap。 | 多人协作强依赖时改 submodule。 |
| DQ-CP3-CR081-03 | implementation | 过程项目是 Git 还是 NAS？ | 推荐 Git worktree，可放 NAS 或用 NAS 备份；本项目只依赖路径。 | 仅 NAS 无 Git；仅本地无 NAS。 | Git 提供审计；NAS 提供集中存储；仅 NAS 回滚弱。 | 过程项目管理独立于本项目。 | 用户明确只用 NAS 时保留快照要求。 |
| DQ-CP3-CR081-04 | implementation | 新环境如何恢复 `process/`？ | 新增 `LEDGER.md`、`ledger.yaml`、`scripts/link-engineering-ledger.sh`。 | 手工建链接；写入 README 一段说明。 | 推荐方案可验证且机器可读；手工说明容易漂移。 | 增加少量项目文件。 | 脚本不兼容时回退手工步骤。 |
| DQ-CP3-CR081-05 | scope | 是否同时迁移 `docs/design|features|quality|release`？ | 不迁移，先只处理 `process/`。 | 同时迁移所有过程性 docs；只迁 `docs/quality` 和 `docs/release`。 | 推荐方案 blast radius 小；docs 同迁会扩大验证面。 | docs 仍占项目仓空间。 | CR081 CP8 后另起 docs externalization CR。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权创建外部过程项目、复制 / 移动 / 删除 `process/` 或创建 / 替换软链接。
- 不授权 `git rm --cached`、`git add`、`git commit`、`git push` 或远端治理。
- 不授权 NAS 内容读取、写入、删除、mount 或 promote。
- 不授权迁移 `docs/design`、`docs/features`、`docs/quality`、`docs/release`。
- 不授权数据、凭据、runtime、CR046 recovery、old root retirement 或 destructive cleanup。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 自动预检 PASS | 待审查 | `process/checks/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONSISTENCY.md` | 待用户确认。 |
| 架构 context ready | 待审查 | `process/context/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONTEXT.yaml` | 待用户确认。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 外部过程项目路径模型 | 待审查 | DQ-CP3-CR081-01 | 待用户确认。 |
| 2 | symlink 是否 tracked | 待审查 | DQ-CP3-CR081-02 | 待用户确认。 |
| 3 | Git / NAS 存储模型 | 待审查 | DQ-CP3-CR081-03 | 待用户确认。 |
| 4 | bootstrap 方案 | 待审查 | DQ-CP3-CR081-04 | 待用户确认。 |
| 5 | docs 是否 deferred | 待审查 | DQ-CP3-CR081-05 | 待用户确认。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 当前对话 | 等待回复。 |
| approve 后仍不执行 cutover | 待审查 | 不授权项 | 需 CP5 approved 后才进入 preflight。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck | `process/checks/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONSISTENCY.md` | 待审查 | PASS。 |
| CP3 context | `process/context/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-CONTEXT.yaml` | 待审查 | ready-for-human-gate。 |
| CP3 manual checkpoint | `process/checkpoints/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-REVIEW.md` | 待审查 | 本文件。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T08:36:45+08:00
- 修改意见：批准架构方案，但覆盖 DQ-CP3-CR081-01 的默认路径：外部 process 项目使用 `/home/hyde/workspace/process`，本项目 `process` 入口使用相对软链接 `process -> ../process`。
- 风险接受项：接受外部 process 项目不由当前业务项目 Git 直接跟踪；接受新环境需要通过 README / `ledger.yaml` / bootstrap 脚本恢复软链接。
