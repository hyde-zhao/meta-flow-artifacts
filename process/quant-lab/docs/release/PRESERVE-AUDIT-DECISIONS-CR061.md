---
status: "draft-for-cp5-review"
version: "1.0"
change_id: "CR-061"
title: "CR061 Preserve-audit Decisions"
created_by: "host-orchestrator"
created_at: "2026-06-15T07:29:19+08:00"
sensitive_content_read_authorized: false
---

# CR061 Preserve-audit Decisions

CR061 继承 CR058 / CR060 的 preserve-audit 原则：历史过程证据中的 `local_backtest` 是当时事实，不因为 package/import/layout 迁移而批量改写。

## 1. Preserve-audit allowlist

| allowlist_id | path_pattern | reason | allowed_action | exemption_allowed | exemption_required_fields | owner | verification_rule |
|---|---|---|---|---:|---|---|---|
| PAG-CR061-001 | `process/**` | 运行态过程证据是历史审计链。 | preserve_audit | true | `path, owner, reason, risk, rollback_ref, approval_decision_id` | host-orchestrator | 历史证据不得进入 package/import rewrite。 |
| PAG-CR061-002 | `process/checks/**` | 自动检查结果是门禁证据。 | preserve_audit | true | 同上 | host-orchestrator | CP 证据保持原文。 |
| PAG-CR061-003 | `process/checkpoints/**` | 人工确认和用户审查结果不可重写语境。 | preserve_audit | true | 同上 | host-orchestrator | checkpoint 不做机械替换。 |
| PAG-CR061-004 | `process/handoffs/**` | 调度证据保留原 workflow_id、agent_id 和时间线。 | preserve_audit | true | 同上 | host-orchestrator | handoff 不进入 rewrite。 |
| PAG-CR061-005 | `process/stories/**` historical non-CR061 | 历史 Story / LLD / IMPLEMENTATION 是当时事实。 | preserve_audit | true | 同上 | story owner | 非 CR061 历史 Story 不批量改写。 |
| PAG-CR061-006 | `DEV-LOG.md` | 开发日志是历史时间线。 | preserve_audit | true | 同上 | host-orchestrator | 不批量替换。 |
| PAG-CR061-007 | `checkpoints/**` legacy fallback | legacy fallback 证据保留原路径和语境。 | preserve_audit | true | 同上 | host-orchestrator | 不批量替换。 |
| PAG-CR061-008 | notebooks / reports historical titles | 可能是历史输出或论文/实验上下文。 | manual_review | true | 同上 | meta-doc / research owner | 不执行 notebook，不改报告正文。 |

## 2. Sensitive filter

| sensitive_rule_id | match_surface | pattern_or_reference | filter_result | reason | verification_rule |
|---|---|---|---|---|---|
| SFG-CR061-001 | path_metadata | `.env` / `*.env` | excluded | 禁止读取或迁移真实环境文件。 | 文档不得包含敏感正文。 |
| SFG-CR061-002 | reference_metadata | token / password / private key / cookie / session / pem | blocked_sensitive | 凭据类引用默认 fail-closed。 | 不打印、不读取、不记录值。 |
| SFG-CR061-003 | reference_metadata | account / cash / position / order / fill raw facts | blocked_sensitive | 未脱敏账户和交易事实不进入候选。 | 只记录边界。 |
| SFG-CR061-004 | path_metadata | untracked data / external lake body / NAS archive body | excluded | 当前未授权扫描外置数据。 | 不扫描、不统计、不复制。 |
| SFG-CR061-005 | reference_metadata | `MARKET_DATA_LAKE_ROOT` | keep | 数据湖入口保持兼容，不替换。 | CR064 前不得 root replacement。 |
| SFG-CR061-006 | reference_metadata | QMT / MiniQMT runtime / trading boundary | manual_review | 交易边界可能是组件语义，不机械改名。 | 不启动 runtime。 |

## 3. 单项豁免合同

| 字段 | 必填 | 说明 |
|---|---:|---|
| `path` | yes | repo-relative exact path。 |
| `owner` | yes | 对历史语境负责的 owner。 |
| `reason` | yes | 为什么移出 preserve-audit。 |
| `risk` | yes | 审计破坏、语义误改、回滚复杂度。 |
| `rollback_ref` | yes | 对应 `ROLLBACK-REF-CR061.md` 中已满足的 ref。 |
| `approval_decision_id` | yes | CP5/CP8 或二次执行授权中的用户决策 ID。 |

当前豁免数量：0。

## 4. 未授权项

本文不授权历史 `process/**`、checks、checkpoints、handoffs、Story、LLD 或 `DEV-LOG.md` 的机械改写，不授权凭据读取，不授权 NAS / lake / provider / QMT / MiniQMT / git remote 操作，不恢复 CR046。
