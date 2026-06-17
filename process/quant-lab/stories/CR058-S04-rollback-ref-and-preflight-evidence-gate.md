---
story_id: "CR058-S04"
status: "verified-with-risk-static-only"
change_id: "CR-058"
title: "Rollback Ref and Preflight Evidence Gate"
lld_policy:
  required_level: "full-lld"
feature_design_refs:
  - "FEAT-CR058-04"
dev_gate: "Rollback_ref checklist approved; missing evidence fails closed."
---

# CR058-S04 Rollback Ref and Preflight Evidence Gate

## 技术说明

本 Story 定义真实 repo-local move / rewrite 前的 rollback_ref 前置条件。本轮不创建 commit、不创建 bundle、不执行 manifest 生成。

必填 rollback_ref：

| rollback_ref | 必填时间 | 证明方式 | 失败处理 |
|---|---|---|---|
| `pre_cr058_commit` | 任何真实 move / rewrite 前 | 本地 Git commit hash | 缺失则阻断执行。 |
| `pre_cr058_git_bundle` | 任何不可逆批量动作前 | `git bundle verify` evidence 或用户风险接受豁免 | 缺失则阻断执行。 |
| `pre_reference_rewrite_manifest` | reference rewrite 前 | candidate list + preserve-audit allowlist | 缺失则阻断执行。 |

验收条件：三项未满足时，CR058 只能停留在设计 / no-op dry-run。
