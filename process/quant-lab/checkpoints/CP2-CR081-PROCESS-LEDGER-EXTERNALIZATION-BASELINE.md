---
checkpoint_id: "CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE"
checkpoint_name: "CR081 Process Ledger Externalization Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T08:03:08+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T08:36:45+08:00"
auto_check_result: "process/checks/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE.md"
context_capsule: "process/context/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "requirement-clarification"
  artifacts:
    - "process/changes/CR-081-PROCESS-LEDGER-EXTERNALIZATION-SYMLINK-CUTOVER-2026-06-17.md"
---

# CP2 CR081 Process Ledger Externalization Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE.md` | PASS | 0 | 本门只确认范围基线和启动 CR081；不执行迁移或软链接 cutover。 |

## Decision Brief

推荐决策：批准 CR081 CP2 范围基线，将“过程文件外部过程项目管理 + 本项目软链接入口”正式纳入 high-risk standard CR。CP2 只确认目标、范围、owner 方向和不授权项，不单独授权执行。

备选方案：保持当前 in-repo `process/` owner；仅建立外部只读镜像；取消外部化路线；只对未来项目使用外部 ledger、不迁移当前项目。推荐方案优先，因为当前过程文件数量已经较大，且用户明确希望由独立过程项目管理。

影响维度：process ledger ownership、Git 跟踪边界、target root `/home/hyde/workspace/quant-lab`、legacy root retained audit、CR046 paused ledger、CR080 closed ledger、bootstrap / clone 可用性。

风险与回退：若用户 reject，CR081 保持 pending/cancelled，当前 `process/` 继续由项目仓管理；若后续执行失败，只回滚 process ledger 和 symlink cutover，不触碰 data/reports、NAS 内容、runtime 或远端 Git。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| SGQ / discussion | N/A：本 CR 是 repository/process governance 变更，不是新增产品场景发现。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | CR081 创建前无本 gate DQ。 |
| 用户显式语句 | 当前对话 | scanned | 5 | 5 | 过程项目、软链接、Git/NAS 管理边界纳入 DQ。 |
| CR081 formal CR | `process/changes/CR-081-PROCESS-LEDGER-EXTERNALIZATION-SYMLINK-CUTOVER-2026-06-17.md` | scanned | 8 | 5 | 范围、owner、阶段和不授权项纳入。 |
| STATE / CR-INDEX | `process/STATE.md`; `process/changes/CR-INDEX.yaml` | scanned | 4 | 2 | CR046 paused、CR080 closed、root authority 纳入影响面。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 本轮不是产品场景讨论。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮由 host-orchestrator 受理变更。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR081-01 | scope | 是否正式启动 CR081？ | 启动 CR081，并按 high-risk standard 门禁处理。 | 保持讨论态；取消外部化路线。 | 推荐方案把用户目标纳入可审计 CR；保持讨论态不会解决未跟踪过程文件膨胀。 | 新增正式 CR 和待审门禁。 | reject 时关闭为 cancelled 或保留候选。 |
| DQ-CP2-CR081-02 | scope | 本轮第一阶段是否只迁移 `process/`？ | 第一阶段仅处理 `process/`；`docs/design|features|quality|release` deferred。 | 同时迁移 docs；完全不迁移只做镜像。 | 推荐方案风险最低；同时迁移 docs 会扩大 owner 边界。 | docs 过程证据仍暂留当前项目。 | CP8 后再评估 docs 外置 CR。 |
| DQ-CP2-CR081-03 | architecture | 外部过程项目是否成为 `process/**` 唯一 owner？ | 是；本项目只保留本地入口。 | 只做外部备份；双写长期并存。 | 推荐方案职责清晰；双写长期易漂移。 | clone 后需 bootstrap 软链接。 | symlink 断链时回退到项目内 process backup。 |
| DQ-CP2-CR081-04 | security | 是否继续禁止 data/reports/NAS 内容、凭据和 runtime？ | 继续禁止，本 CR 只处理 process ledger。 | 同时纳入 NAS 内容体检；同时恢复 CR046。 | 推荐方案维持最小权限；备选跨越敏感数据和 runtime。 | 不验证 NAS 内容或数据文件。 | 需要 NAS/data/runtime 时另起 CR。 |
| DQ-CP2-CR081-05 | runtime_authorization | CP2 approve 是否授权执行迁移？ | 不授权；必须 CP2/CP3/CP5 全部 approve 后才允许 preflight 和 cutover。 | CP2 后立即执行；一次性授权全部后续动作。 | 推荐方案符合关键门控；备选边界过宽。 | 防止范围批准被误读为执行许可。 | 三门通过后进入 controlled preflight。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权创建外部过程项目、复制 / 移动 / 删除 `process/` 或创建 / 替换软链接。
- 不授权 `git rm --cached`、`git add`、`git commit`、`git push`、remote set-url、branch/default branch governance、history rewrite、force push 或 tag。
- 不授权访问 NAS 内容、创建 mount、写 NAS、删除 NAS 文件或 NAS promote。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权读取、解析、hash 或迁移 `data/`、`reports/` 内容。
- 不授权恢复 CR046、启动 QMT/MiniQMT runtime、交易相关操作、旧根删除/移动或 destructive cleanup。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR081 formal CR 存在 | 待审查 | `process/changes/CR-081-PROCESS-LEDGER-EXTERNALIZATION-SYMLINK-CUTOVER-2026-06-17.md` | 待用户确认。 |
| CP2 自动预检 PASS | 待审查 | `process/checks/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE.md` | 待用户确认。 |
| context ready | 待审查 | `process/context/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-CONTEXT.yaml` | 待用户确认。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否启动 CR081 | 待审查 | DQ-CP2-CR081-01 | 待用户确认。 |
| 2 | 第一阶段是否只处理 `process/` | 待审查 | DQ-CP2-CR081-02 | 待用户确认。 |
| 3 | 外部过程项目是否成为 owner | 待审查 | DQ-CP2-CR081-03 | 待用户确认。 |
| 4 | 敏感数据和 runtime 是否继续禁止 | 待审查 | DQ-CP2-CR081-04 | 待用户确认。 |
| 5 | CP2 是否不授权执行 | 待审查 | DQ-CP2-CR081-05 | 待用户确认。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 当前对话 | 等待回复。 |
| approve 后仍不执行迁移 | 待审查 | 不授权项 | 需 CP3/CP5 同步 approved 后才进入 preflight。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck | `process/checks/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE.md` | 待审查 | PASS。 |
| CP2 context | `process/context/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-CONTEXT.yaml` | 待审查 | ready-for-human-gate。 |
| CP2 manual checkpoint | `process/checkpoints/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE.md` | 待审查 | 本文件。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T08:36:45+08:00
- 修改意见：批准 CR081 继续推进；外部 process 项目路径修订为 `/home/hyde/workspace/process`，即与 `/home/hyde/workspace/quant-lab` 同级的 `process` 目录。
- 风险接受项：接受第一阶段只迁移 `process/`、不迁移 `docs/*` 过程文档；接受执行前先 mirror、备份、验证，再替换为 symlink。
