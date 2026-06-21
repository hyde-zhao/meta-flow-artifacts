---
checkpoint_id: "CP2-CR106-REMEDIATION-BASELINE"
checkpoint_name: "CR106 Remediation Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-21T21:13:33+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-21T21:13:33+08:00"
auto_check_result: "process/checks/CP2-CR106-REMEDIATION-BASELINE.md"
context_capsule: "process/context/CP2-CR106-REMEDIATION-BASELINE-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "requirement-clarification"
  artifacts:
    - "process/changes/CR-106.md"
---

# CP2 CR106 Remediation Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR106-REMEDIATION-BASELINE.md` | PASS | 0 | 本门只确认 P1 整改基线，不授权业务整改执行。 |

## Decision Brief

推荐决策：批准 CR106 P1 remediation baseline。批准后进入 P2 remediation design，先设计 package identity fallout 与 CR tracking historical hygiene 的整改方案、Story/Wave 和验证矩阵；不直接进入业务代码整改。

备选方案：暂缓确认，等待 CR102-CR104 在主线关闭后再确认；或缩小 P1 只保留 package identity fallout，不纳入 CR tracking hygiene；或 reject 当前基线并重新起草范围。

影响维度：process adoption 稳定性、package identity 兼容、历史 CR tracking 可审计性、后续业务整改准入、非 runtime 验证范围、人工门禁和不授权边界。

风险与回退：若后续发现 CR102-CR104 仍阻断 P2，则 P2 保持 pending，不进入设计执行；若 package identity 改名引发兼容问题，后续单独 CR 增加 alias / migration note；若用户 reject，本 CR 保持 active/pending 并回到范围修订。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR106-REMEDIATION-BASELINE-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| SGQ / discussion | N/A：本 CR 是 process/adoption remediation baseline，不是新增产品场景发现。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 本轮未发现独立 pending queue 项。 |
| 用户显式语句 | 当前对话 | scanned | 3 | 3 | 继续 CR105、先收敛 package identity、确认基线纳入 DQ。 |
| CR105 / CR106 formal CR | `process/changes/CR-105.md`; `process/changes/CR-106.md` | scanned | 7 | 4 | 范围、阶段、CR102-CR104、P2 拆分纳入。 |
| 自动预检结果 | `process/checks/CP2-CR106-REMEDIATION-BASELINE.md` | scanned | 2 | 1 | cr-tracking WARN 纳入风险接受。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 本轮不是产品场景讨论。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮由 host-orchestrator 直接确认 process baseline。 |
| 下游正式产物 | HLD / LLD / REVIEW / release docs | n/a | 0 | 0 | P2 尚未开始，不存在下游设计或验证产物。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR106-01 | scope | 是否批准 CR106 P1 remediation baseline？ | 批准；P1 冻结范围、风险、阶段和 P2 入口。 | 暂缓确认；reject 并重写基线。 | 推荐方案可进入设计；暂缓可等待 CR102-CR104 关闭但延后整改。 | 影响后续 P2 是否能启动。 | reject 或修改时回到 CR106 范围修订。 |
| DQ-CP2-CR106-02 | implementation | P2 是否仍保持设计阶段，不改业务代码？ | 保持设计阶段，只输出 HLD/Story/Wave/验证矩阵。 | 直接进入业务整改；或只做文档清理。 | 推荐方案降低返工和越界风险；直接整改速度快但缺设计证据。 | 控制业务代码变更准入。 | CP3/CP5 未通过不得进入 P3。 |
| DQ-CP2-CR106-03 | risk_acceptance | CR102-CR104 多 active 是否阻断 P1 基线确认？ | 不阻断 P1；按用户说明等待主线关闭，不在本轮处理。 | 等主线关闭后再确认；或本轮关闭它们。 | 推荐方案尊重当前分工；等待会延后 P2；本轮关闭会扩大范围。 | 默认 cr-tracking 仍会因多 active 报错。 | 若主线未关闭且影响 P2，则 P2 pending。 |
| DQ-CP2-CR106-04 | implementation | P2 是否拆分 package identity fallout 与 CR tracking hygiene？ | 拆分为两个设计对象，P2 冻结具体 CR 数。 | 合并为一个大整改对象。 | 拆分降低风险、便于验证；合并减少门禁但复杂度更高。 | 影响后续 CR 数和 Wave 组织。 | P2 设计后可调整为 1 或 2 个正式 CR。 |

### 用户需决策事项

| 决策 ID | 用户需决策事项 | 当前结论 |
|---|---|---|
| DQ-CP2-CR106-01 | 是否批准 CR106 P1 remediation baseline | approved |
| DQ-CP2-CR106-02 | 是否确认 P2 只做设计、不改业务代码 | approved |
| DQ-CP2-CR106-03 | 是否接受 CR102-CR104 暂不阻断 P1 | approved |
| DQ-CP2-CR106-04 | 是否将 package identity fallout 与 CR tracking hygiene 拆分为 P2 设计对象 | approved |

### 不授权项

如果你回复 approve，表示接受本文件 4 项推荐方案；不表示授权以下操作：

- 不授权读取 `.env`、token、API key、cookie、私钥、账号或任何凭据。
- 不授权启动 runtime、SaaS、trace upload、production write、真实交易或真实发布。
- 不授权启动 CR-033。
- 不授权处理、关闭或绕过 CR102-CR104；它们按用户说明由主线后续关闭。
- 不授权进入业务代码整改；P2 只做整改设计，P3 才能在独立批准后实施。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR106 formal CR 存在 | approved | `process/changes/CR-106.md` | 用户已确认继续基线。 |
| CP2 自动预检 PASS | approved | `process/checks/CP2-CR106-REMEDIATION-BASELINE.md` | 无阻断项。 |
| context ready | approved | `process/context/CP2-CR106-REMEDIATION-BASELINE-CONTEXT.yaml` | ready-for-human-gate。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否批准 CR106 P1 基线 | approved | DQ-CP2-CR106-01 | 批准。 |
| 2 | P2 是否保持设计阶段 | approved | DQ-CP2-CR106-02 | 批准。 |
| 3 | CR102-CR104 是否不阻断 P1 | approved | DQ-CP2-CR106-03 | 批准；本轮不处理。 |
| 4 | P2 是否拆分 identity 与 CR tracking hygiene | approved | DQ-CP2-CR106-04 | 批准，P2 冻结具体 CR 数。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话：“好的，继续确认基线” | 视为批准 P1 基线。 |
| approve 后仍不执行业务整改 | approved | 不授权项 | 后续进入 P2 设计，不进入 P3 执行。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck | `process/checks/CP2-CR106-REMEDIATION-BASELINE.md` | approved | PASS。 |
| CP2 context | `process/context/CP2-CR106-REMEDIATION-BASELINE-CONTEXT.yaml` | approved | ready-for-human-gate。 |
| CP2 manual checkpoint | `process/checkpoints/CP2-CR106-REMEDIATION-BASELINE.md` | approved | 本文件。 |
| CR106 baseline | `process/changes/CR-106.md` | approved | P1 baseline approved。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-21T21:13:33+08:00
- 修改意见：批准 CR106 P1 整改基线；下一步进入 P2 remediation design，不直接进入业务整改。
- 风险接受项：接受 CR102-CR104 暂不处理且可能导致默认 cr-tracking 多 active 报错；接受历史 follow-up / spike candidate 索引 WARN 留给后续 CR tracking hygiene。
