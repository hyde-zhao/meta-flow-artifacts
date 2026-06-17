---
checkpoint_id: "CP5-CR080-DATA-REPORTS-RESTORE-READINESS"
checkpoint_name: "CR080 Data/reports Restore Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T06:42:48+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T06:57:22+08:00"
auto_check_result: "process/checks/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md"
context_capsule: "process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "story-planning"
  story_id: ""
  artifacts:
    - "process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md"
---

# CP5 CR080 Data/reports Restore Readiness 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` | PASS | 0 | 替代 LLD 门禁通过；执行仍需用户 approve、post-approval preflight PASS。 |

## Decision Brief

推荐决策：批准 CR080 CP5 execution readiness。用户 approve 后，仅授权先跑 post-approval preflight；preflight PASS 后才允许受控本地 restore/copy。任何 target 冲突、来源不足、denylist 异常、命令越界或 CR tracking 失败都必须 BLOCKED。

备选方案：只批准 preflight 不批准 copy；继续保持 CR080 pending；改为 NAS compare 或 provider rebuild。推荐方案优先，因为它能在安全边界内补齐 target root 的本地可用性缺口。

影响维度：执行授权、命令白名单、目标根写入、验证证据、失败处理、后续候选分流。

风险与回退：approve 不授权内容读取或删除；若 preflight 失败，不复制；若 copy 失败，不自动 cleanup，记录 partial 状态并回到人工决策。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| LLD clarification queue | N/A：本 CR 不修改代码、Story、接口或文件 owner，使用 CP5 readiness 替代 Story LLD 批次。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 10 | 0 | CP2/CP3 DQ 已在对应门中覆盖；CP5 只纳入执行授权问题。 |
| CR080 formal CR | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | scanned | 10 | 5 | preflight、copy、验证、失败和后续候选纳入。 |
| CP5 自动预检 | `process/checks/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` | scanned | 6 | 5 | 所有执行风险均进入 DQ 或 N/A。 |
| CP2 / CP3 context | `process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml`; `process/context/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONTEXT.yaml` | scanned | 5 | 2 | 范围和设计决策作为 CP5 前置条件。 |
| Story / LLD artifacts | `process/stories/*` | n/a | 0 | 0 | 本 CR 不涉及 Story 实现设计。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR080-01 | runtime_authorization | approve 后是否允许执行 post-approval preflight？ | 允许；preflight 只做元数据和状态检查，不复制。 | 不允许执行；只保留 CR080 文档。 | 推荐方案为安全执行做准备；备选无实际进展。 | preflight 会触碰目录元数据和 target 状态。 | 用户不批准时保持 pending。 |
| DQ-CP5-CR080-02 | runtime_authorization | preflight PASS 后是否允许受控 restore/copy？ | 允许本地 legacy -> target copy；不得内容读取、不得 delete、不得 checksum。 | 只批准 preflight；改为 NAS compare。 | 推荐方案能补齐 target；备选更保守但延迟恢复。 | 会向 target root 写入 `reports/` / `data/`。 | preflight 任一失败即 BLOCKED。 |
| DQ-CP5-CR080-03 | security | 验证证据是否限制为元数据摘要和禁止项计数？ | 是，只记录存在性、数量、规模、mtime、exclude 命中和禁止项计数。 | 写完整清单；写内容 hash。 | 推荐方案防泄露；备选可验证性更强但越界。 | 不证明内容质量或逐文件一致性。 | 需要内容质量证明时另起内容验证 CR。 |
| DQ-CP5-CR080-04 | implementation | copy 失败或 partial 状态是否禁止自动删除 / cleanup？ | 禁止；记录 partial 状态并请求人工决策。 | 自动删除新建目标；自动移动到隔离目录。 | 推荐方案避免破坏性操作；备选更整洁但风险高。 | 失败后可能留下 partial 资产。 | cleanup 需另起授权或重新发起 CP5。 |
| DQ-CP5-CR080-05 | follow_up_tracking | 执行后后续候选如何分流？ | 成功后再评审 CR077；NAS/provider/CR078 仍不自动启动。 | 成功后立即旧根退役；成功后立即远端 Git governance。 | 推荐方案保持最小步进；备选扩大到破坏性或外部写入。 | 旧根、远端和 provider 风险仍需独立门禁。 | 用户明确启动对应候选时切换。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权跳过 post-approval preflight。
- 不授权内容读取、样本打印、文件解析、内容 hash、checksum 校验或内容比对。
- 不授权复制 credential-like path；不授权读取或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 `rsync --delete`、覆盖已有非空 target、自动 merge、自动 cleanup、旧根删除/移动。
- 不授权 NAS/provider/lake/catalog、remote Git、QMT/MiniQMT runtime、CR046 recovery、optional groups 或 full tests。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP5 自动预检 PASS | approved | `process/checks/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` | 用户回复“同意”，接受推荐方案。 |
| CP5 context ready | approved | `process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml` | 用户回复“同意”，接受推荐方案。 |
| 替代 LLD 门禁说明存在 | approved | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | 用户回复“同意”，接受推荐方案。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否允许 post-approval preflight | approved | DQ-CP5-CR080-01 | 用户接受推荐方案。 |
| 2 | preflight PASS 后是否允许受控 copy | approved | DQ-CP5-CR080-02 | 用户接受推荐方案。 |
| 3 | 验证证据是否只写元数据 | approved | DQ-CP5-CR080-03 | 用户接受推荐方案。 |
| 4 | 失败是否不自动 cleanup | approved | DQ-CP5-CR080-04 | 用户接受推荐方案。 |
| 5 | 后续候选是否保持独立 | approved | DQ-CP5-CR080-05 | 用户接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话：用户回复“同意” | approve 后进入 post-approval preflight；不跳过检查。 |
| preflight 失败时不执行 copy | approved | DQ-CP5-CR080-02 | 任一失败转 BLOCKED。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 auto precheck | `process/checks/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` | approved | PASS。 |
| CP5 context | `process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml` | approved | ready-for-human-gate。 |
| CP5 manual checkpoint | `process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` | approved | 用户接受 DQ-CP5-CR080-01..05。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T06:57:22+08:00
- 修改意见：无。用户回复“同意”，接受 DQ-CP5-CR080-01..05 推荐方案。
- 风险接受项：接受 post-approval preflight、preflight PASS 后受控本地 copy、元数据验证范围、失败不自动 cleanup 和后续候选不自动启动。
