---
handoff_id: CR173-CP5-META-SE-DESIGN-REMEDIATION
cr_id: CR-173
from: host-orchestrator
to: meta-se
codex_agent_name: meta-se-critical
reasoning_profile: critical
status: returned
created_at: "2026-07-16T16:27:25+08:00"
completed_at: "2026-07-16T16:47:30+08:00"
dispatch_trigger: CP5-round1-required-findings-design-baseline-remediation
context_ref: process/context/CP5-CR173.context.json
review_findings_ref: process/checks/CP5-CR173-LLD-REVIEW-FINDINGS.md
review_summary_ref: process/checks/CP5-CR173-LLD-REVIEW-SUMMARY.md
dispatch_ref: AD-CR173-CP5-META-SE-CRITICAL-REMEDIATION-20260716T162725+0800
agent_id: /root/se_chu_cr173
result_status: pass_for_lld_resync
return_ref: process/handoffs/CR173-CP5-META-SE-DESIGN-REMEDIATION-RETURN-SUMMARY.md
---

# CR-173 CP5 Meta-SE-Critical Design Remediation Handoff

## 目标

在不扩大 CR-173 estimator-only 授权的前提下，修复 CP5 Round 1 的三项 required finding 与一项 freshness finding，并把权威语义统一回写到 HLD、Domain Map、ADR、Feature 设计和 Story 规划。不得修改三份 LLD；LLD 由 meta-dev 在权威基线整改后单独同步。

## 必须整改

1. 七字段中的 `effective_trial_computation_ref` 冻结为 stable content-addressed result identity；逐执行追加审计建模为七字段之外的 `ComputationAttemptAudit`，明确 owner、identity、linkage、F01-F08 canonical attempt basis 与当前 persistence/retention=N/A。
2. 所有非 canonical token（含 `NaN` / `Inf`）唯一映射 F03；F04 只覆盖成功解析为有限 exact rational 后的矩阵域失败。
3. 把 CR173 新代码 public dependency/call/diff/write 四个 zero counter 与 CP7 的 12/12 read-only public regression inventory 分开计数；existing expected edits 保持 0。
4. 刷新 Feature/Story 的 CP4/CP5 point-in-time 状态，但 `design_evidence_confirmed=false`、`lld_confirmed=false` 必须保持。

## 禁止项

- 不修改源码、测试、fixture 或 public C1 production contract。
- 不修改三份 Story LLD、STATE、checkpoint 或 ledgers。
- 不发起或批准 CP5，不执行真实数据、runtime、trading 或远程操作。
