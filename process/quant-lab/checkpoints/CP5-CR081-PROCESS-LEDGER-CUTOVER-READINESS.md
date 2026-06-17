---
checkpoint_id: "CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS"
checkpoint_name: "CR081 Process Ledger Cutover Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T08:03:08+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T08:36:45+08:00"
auto_check_result: "process/checks/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md"
context_capsule: "process/context/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "story-planning"
  artifacts:
    - "process/changes/CR-081-PROCESS-LEDGER-EXTERNALIZATION-SYMLINK-CUTOVER-2026-06-17.md"
---

# CP5 CR081 Process Ledger Cutover Readiness 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md` | PASS | 0 | 执行顺序已定义；未执行任何迁移。 |

## Decision Brief

推荐决策：批准 CR081 进入 post-approval preflight 和 controlled cutover readiness。若 CP2/CP3/CP5 均 approve，下一步仍先做 preflight；只有 preflight PASS 后才允许 mirror `process/` 到外部 ledger，再验证，再替换为 symlink。CP5 approve 不授权 Git commit/push 或 NAS 内容操作。

备选方案：只做外部 mirror 不 cutover；先创建外部项目但不改本项目 `process/`；reject 并保留当前 in-repo process；转 Spike 验证 symlink 兼容性。推荐方案优先，因为它完整实现用户目标，同时仍保留 preflight 和回滚。

影响维度：本地工作树、Git index、`process/STATE.md` 可读性、CR tracking consistency、外部 ledger root、bootstrap 脚本、新环境恢复、回滚备份。

风险与回退：执行失败时不删除外部 mirror；本地保留 `process` backup；可移除 symlink 并恢复目录。Git index 变更仅本地 staged，不 commit/push，后续需单独确认。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| LLD clarification | N/A：无应用 Story / LLD 批次；以 CP5 readiness 替代。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 10 | 0 | CP2/CP3 DQ 不重复纳入。 |
| CR081 formal CR | `process/changes/CR-081-PROCESS-LEDGER-EXTERNALIZATION-SYMLINK-CUTOVER-2026-06-17.md` | scanned | 9 | 5 | preflight、mirror、Git index、symlink、validation 纳入。 |
| 自动预检结果 | `process/checks/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md` | scanned | 5 | 5 | 全部为 execution readiness DQ。 |
| `.gitignore` / Git state | `.gitignore`; `git status --short` | scanned | 3 | 2 | tracked process count 和 ignore 规则留到 preflight。 |
| 运行 / 外部接口 | N/A | n/a | 0 | 0 | 本 CR 不触碰 runtime、data/reports 或 NAS 内容。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 非产品讨论。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR081-01 | implementation | 三门通过后是否先执行 preflight？ | 是；检查 `process/` 类型、外部 root、tracked count、断链、脚本兼容和 dirty worktree。 | 直接执行 mirror/cutover；只做手工检查。 | 推荐方案可阻断错误路径；直接执行风险高。 | 可能发现需用户补路径。 | preflight FAIL 时不执行 cutover。 |
| DQ-CP5-CR081-02 | implementation | 是否允许 mirror `process/` 到外部 ledger？ | 允许在 preflight PASS 后用 `rsync -a` mirror，禁止 `--delete`。 | 手工复制；`rsync --delete` 保持一致。 | 推荐方案保守且可验证；`--delete` 风险高。 | 外部 ledger 可能已有内容冲突。 | 目标非空冲突时 BLOCKED。 |
| DQ-CP5-CR081-03 | implementation | 是否允许本地 Git index 停止跟踪 `process/**`？ | 允许执行 `git rm --cached -r process`，但不 commit/push。 | 不改 Git index；直接 track symlink。 | 推荐方案让 owner 单一；不改 index 会继续双写。 | 产生大量 staged deletion，需要单独提交确认。 | 若用户拒绝 Git index 变更，则只做 mirror。 |
| DQ-CP5-CR081-04 | implementation | 是否允许本地 `process` 替换为 symlink？ | 允许在备份原目录并验证 mirror 后替换为 symlink。 | 只创建 `process_external` symlink；不替换 `process`。 | 推荐方案实现用户目标；备选低风险但工具仍读本地 process。 | symlink 断链会影响 Meta Flow。 | 断链时恢复 backup。 |
| DQ-CP5-CR081-05 | risk_acceptance | 是否接受 cutover 后的剩余风险？ | 接受：需要 bootstrap、外部 ledger 在线、禁止跟随 symlink 的 destructive 操作。 | 先转 Spike；只做 mirror 不 cutover。 | 推荐方案可落地；Spike 更稳但延迟目标。 | clone 后需重建 symlink，NAS 离线会影响流程。 | 验证失败或用户不接受风险时转 Spike/mirror-only。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权 Git commit、Git push、remote write、branch/default branch governance、history rewrite、force push 或 tag。
- 不授权 `rsync --delete`、`find -L`、跟随 symlink 的批量删除或 destructive cleanup。
- 不授权访问 NAS 内容、mount、promote、删除或写入 NAS 数据文件。
- 不授权读取 `.env`、凭据、data/reports 内容、样本解析、hash、provider/lake/catalog 或 runtime。
- 不授权迁移 `docs/design`、`docs/features`、`docs/quality`、`docs/release`。
- 不授权 CR046 recovery、old root retirement 或目标根删除/移动。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP5 自动预检 PASS | 待审查 | `process/checks/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md` | 待用户确认。 |
| execution context ready | 待审查 | `process/context/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS-CONTEXT.yaml` | 待用户确认。 |
| CP2 / CP3 推荐方案已写明 | 待审查 | CP2 / CP3 checkpoints | 可同轮 approve。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | preflight 是否必须先执行 | 待审查 | DQ-CP5-CR081-01 | 待用户确认。 |
| 2 | mirror 是否允许且禁止 delete | 待审查 | DQ-CP5-CR081-02 | 待用户确认。 |
| 3 | Git index 是否允许停止跟踪 process | 待审查 | DQ-CP5-CR081-03 | 待用户确认。 |
| 4 | process 是否允许替换为 symlink | 待审查 | DQ-CP5-CR081-04 | 待用户确认。 |
| 5 | 剩余风险是否接受 | 待审查 | DQ-CP5-CR081-05 | 待用户确认。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 当前对话 | 等待回复。 |
| approve 后进入 preflight 而非直接 cutover | 待审查 | 执行顺序 | preflight FAIL 时不执行迁移。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 auto precheck | `process/checks/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md` | 待审查 | PASS。 |
| CP5 context | `process/context/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS-CONTEXT.yaml` | 待审查 | ready-for-human-gate。 |
| CP5 manual checkpoint | `process/checkpoints/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md` | 待审查 | 本文件。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T08:36:45+08:00
- 修改意见：授权在 preflight PASS 后创建 `/home/hyde/workspace/process`、mirror 全量 `process/`、新增 README、停止当前项目 Git 跟踪 `process/**`、备份原目录并替换为 `process -> ../process` 软链接；不授权 commit/push。
- 风险接受项：接受外部 process 项目在线性、软链接断链、Git index 出现大量 staged deletion、旧目录以 sibling backup 保留等风险；不授权 data/reports、NAS 内容、凭据、runtime、CR046 recovery 或 destructive cleanup。
