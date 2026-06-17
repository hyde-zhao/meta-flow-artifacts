# CP2-CR062 Requirements Baseline 人工检查点

## 自动预检摘要

自动预检结论：PASS。CR062 可作为 CR061 后续正式 Git Remote Cutover / Repository Publication Gate 启动；当前只冻结范围、GitHub target candidate、publication scope、扫描计划和不授权边界，不执行任何 Git 远端、commit、push、tag、branch rename、history rewrite、NAS/data lake/provider/runtime/凭据、physical move 或 bulk import rewrite。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR062-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在字段不足或人工审计时读取完整文档 |
| 全文档读取扩展 | 已读取 AGENTS、POST-CR061 handoff、STATE、CR-INDEX、CR061 CR、CR061 CP8、CR061 release context |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前无未批准 CR062 DQ。 |
| 自动预检结果 | `process/checks/CP2-CR062-REQUIREMENTS-BASELINE.md` | scanned | 5 | 5 | 范围、GitHub target、不授权边界转 DQ。 |
| Handoff | `process/context/POST-CR061-MIGRATION-HANDOFF-CONTEXT.yaml` | scanned | 6 | 4 | recommended_next_cr 和 strictly_not_authorized 转 DQ。 |
| CR061 CP8 | `process/checkpoints/CP8-CR061-BATCH-B-DELIVERY-READINESS.md` | scanned | 5 | 2 | follow-up 与 non-authorized 延续。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | 用户明确启动 CR062。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR062-01 | scope | 是否正式启动 CR062，承接 CR061 后续 Git publication gate？ | 启动 CR062，并登记为迁移并行推进对象；CR046 继续暂停。 | 备选 A：保持为 follow-up candidate；备选 B：先做 bulk relayout；备选 C：先做 NAS/data lake。 | 推荐方案先建立 GitHub 可追踪基线；candidate 会停滞；bulk / NAS / lake 风险更高。 | 当前只创建门禁，不执行 Git 写动作。 | 用户选择其他顺序时，CR062 保持 candidate 或 superseded。 |
| DQ-CP2-CR062-02 | scope | CR062 当前范围是否限定为 repository publication gate，不包含 physical move / NAS / lake / runtime？ | 限定为 GitHub publication gate。 | 备选 A：合并 physical relayout；备选 B：合并 NAS/lake；备选 C：合并 runtime。 | 推荐方案风险分层；合并会扩大回滚面和权限面。 | 后续 CR 数量增加，但边界清晰。 | 用户要求合并时重新做 CR 影响分析。 |
| DQ-CP2-CR062-03 | security | 是否接受 publication include/exclude policy 作为 CP5 前置？ | 接受；post-approval 必须先冻结 allowlist / denylist。 | 备选 A：发布全仓库当前状态；备选 B：只发布 code-only。 | 推荐方案兼顾可审计和安全；全仓库风险高；code-only 会丢迁移证据。 | 需要额外扫描和人工复核。 | manifest 出现 forbidden path 时 BLOCKED。 |
| DQ-CP2-CR062-04 | runtime_authorization | 是否确认 CP2/CP3/CP5 批准前不执行 Git 写动作？ | 确认；CP5 approve 前不 remote/add/commit/push/tag/branch rename/history rewrite。 | 备选 A：立即执行 remote add；备选 B：立即 commit/push。 | 推荐方案遵守门禁；备选均越过当前授权。 | 发布会延后一轮用户确认。 | 用户 approve 后按 CP5 DQ 再执行。 |
| DQ-CP2-CR062-05 | follow_up_tracking | CR062 之后的迁移顺序如何保留？ | Git publication 后再分 CR 处理 bulk relayout、NAS/archive、data lake/catalog、runtime/CR046 recovery。 | 备选 A：CR062 后直接 bulk relayout；备选 B：CR062 后先 NAS/lake；备选 C：恢复 CR046。 | 推荐方案保持可追踪；备选需要新风险分析。 | 完整迁移仍未完成。 | 用户明确改序时新建或重排后续 CR。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP2-CR062-01..05 推荐方案。 |
| 备选方案 | 保持 candidate、先 bulk relayout、先 NAS/lake、只发布 code-only、恢复 CR046。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全 / 权限、交付影响。 |
| 优劣分析 | 见上表。 |
| 风险与回退 | CP5 approve 前没有 Git 写动作；后续若扫描失败则 fail-closed。 |
| 用户需决策事项 | DQ-CP2-CR062-01..05。 |

不授权项：CP2 通过不授权 Git remote add / set-url、commit、push、tag、branch rename、history rewrite、force push、NAS/data lake/provider/catalog、凭据读取、QMT/MiniQMT runtime、CR046 recovery、physical move 或 bulk import rewrite。

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR061 已关闭 | PASS |
| 用户明确启动 CR062 | PASS |
| CR046 不恢复 | PASS |
| 禁止操作边界明确 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 影响分析完成 | PASS |
| 决策项完整 | PASS |
| 不授权项列明 | PASS |
| gate-only 边界明确 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户确认后可进入 CR062 CP3 / CP5 审查 | PASS |

## Deliverables

- `process/checks/CP2-CR062-REQUIREMENTS-BASELINE.md`
- `process/checkpoints/CP2-CR062-REQUIREMENTS-BASELINE.md`
- `process/context/CP2-CR062-REQUIREMENT-CONTEXT.yaml`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T09:59:22+08:00
- 修改意见：用户回复“同意”，按 `approve` 解析，接受 DQ-CP2-CR062-01..05 的推荐方案。
- 风险接受项：接受 CR062 正式启动为 repository publication gate；CR046 继续暂停；CP2 本身不授权 Git remote add / set-url、commit、push、tag、branch rename、history rewrite、force push、NAS/data lake/provider/catalog/runtime/凭据、physical move、bulk import rewrite 或 CR046 recovery。
