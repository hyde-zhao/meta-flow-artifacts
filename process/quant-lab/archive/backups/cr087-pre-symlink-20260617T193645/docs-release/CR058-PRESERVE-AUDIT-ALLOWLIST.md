---
status: "implemented-cp6-static"
version: "1.0"
change_id: "CR-058"
title: "CR058 Preserve-audit Allowlist and Sensitive Filter Gate"
created_by: "meta-dev"
created_at: "2026-06-14T15:40:00+08:00"
source_story: "process/stories/CR058-S03-preserve-audit-allowlist-and-sensitive-filter-gate.md"
source_lld: "process/stories/CR058-S03-preserve-audit-allowlist-and-sensitive-filter-gate-LLD.md"
execution_mode: "static-only-no-op"
sensitive_content_read_authorized: false
---

# CR058 Preserve-audit Allowlist

## 1. 范围和结论

本文固化 CR058 的历史审计保护清单和敏感输入过滤门禁。当前只基于路径族和引用元数据分类，不打开敏感文件正文，不批量扫描 untracked data，不改写历史证据。

| 项目 | 结论 |
|---|---|
| 默认策略 | preserve-audit 优先于 mechanical-candidate。 |
| 历史证据 | 历史过程、检查点、handoff、Story、LLD、DEV-LOG 保留当时语境。 |
| 敏感过滤 | 只按路径 / 引用元数据阻断；不读取正文、不打印凭据。 |
| 单项豁免 | 必须有 path、owner、reason、risk、rollback_ref、approval_decision_id。 |
| 当前执行 | `execute_allowed=false`；不做任何历史 rewrite。 |

## 2. Preserve-audit allowlist

| allowlist_id | path_pattern | reason | allowed_action | exemption_allowed | exemption_required_fields | owner | verification_rule |
|---|---|---|---|---:|---|---|---|
| PAG-CR058-001 | `process/**` | 运行态过程证据是历史审计链，保留当时项目名和上下文。 | preserve_audit | true | `path, owner, reason, risk, rollback_ref, approval_decision_id` | host-orchestrator | 历史证据不得进入 mechanical rewrite。 |
| PAG-CR058-002 | `process/checks/**` | 自动检查结果是门禁证据。 | preserve_audit | true | 同上 | host-orchestrator / meta-dev / meta-qa | CP5/CP6/CP7/CP8 证据保持原文。 |
| PAG-CR058-003 | `process/checkpoints/**` | 人工确认和用户审查结果不可重写语境。 | preserve_audit | true | 同上 | host-orchestrator | checkpoint 审查表不做机械替换。 |
| PAG-CR058-004 | `process/handoffs/**` | 调度证据需要保留原 workflow_id、agent_id 和时间线。 | preserve_audit | true | 同上 | host-orchestrator | handoff 不进入 rewrite。 |
| PAG-CR058-005 | `process/stories/**` historical non-CR058 | 历史 Story / LLD / IMPLEMENTATION 是当时事实记录。 | preserve_audit | true | 同上 | story owner | 非 CR058 历史 Story 不批量改写。 |
| PAG-CR058-006 | `process/context/**` historical capsules | capsule 是阶段上下文恢复证据。 | preserve_audit | true | 同上 | host-orchestrator | 历史 capsule 保留。 |
| PAG-CR058-007 | `process/changes/CR-*.md` historical CR | 正式变更单是审计对象。 | preserve_audit | true | 同上 | host-orchestrator | 历史 CR 不批量替换。 |
| PAG-CR058-008 | `DEV-LOG.md` historical entries | 开发日志是时间线证据。 | preserve_audit | true | 同上 | host-orchestrator / meta-dev | 历史日志不机械替换。 |
| PAG-CR058-009 | `checkpoints/**` legacy fallback | legacy fallback 证据保留原路径和语境。 | preserve_audit | true | 同上 | host-orchestrator | legacy checkpoint 不批量替换。 |

## 3. Sensitive filter

| sensitive_rule_id | match_surface | pattern_or_reference | filter_result | reason | verification_rule |
|---|---|---|---|---|---|
| SFG-CR058-001 | path_metadata | `.env` / `*.env` | excluded | 禁止读取或迁移真实环境文件。 | 文档不得包含敏感正文。 |
| SFG-CR058-002 | reference_metadata | token / password / private key / cookie / session / pem | blocked_sensitive | 凭据类引用默认 fail-closed。 | 不打印、不读取、不记录值。 |
| SFG-CR058-003 | reference_metadata | account / cash / position / order / fill raw facts | blocked_sensitive | 未脱敏账户和交易事实不进入候选。 | 只记录边界，不记录事实。 |
| SFG-CR058-004 | path_metadata | untracked data / external lake body / NAS archive body | excluded | 当前未授权扫描外置数据。 | 不扫描、不统计、不复制。 |
| SFG-CR058-005 | reference_metadata | `MARKET_DATA_LAKE_ROOT` | manual_review | 数据湖入口保持兼容，不替换。 | 确认 `keep`，不做 root replacement。 |
| SFG-CR058-006 | reference_metadata | QMT / MiniQMT runtime / trading boundary | manual_review | 交易边界可能是组件语义，不机械改名。 | QMT owner review；不启动 runtime。 |
| SFG-CR058-007 | path_metadata | security / secret boundary governance docs | manual_review | 路径名可能是治理文档，不等于可读取凭据。 | 不读正文，人工确认语义。 |

## 4. 单项豁免合同

从 preserve-audit allowlist 中移出单项，必须新增人工决策项并满足全部字段。

| 字段 | 必填 | 说明 |
|---|---:|---|
| `path` | yes | repo-relative exact path。 |
| `owner` | yes | 对历史语境负责的 owner。 |
| `reason` | yes | 为什么移出 preserve-audit。 |
| `risk` | yes | 审计破坏、语义误改、回滚复杂度。 |
| `rollback_ref` | yes | 对应 `CR058-ROLLBACK-GATE.md` 中已满足的 ref。 |
| `approval_decision_id` | yes | CP5/CP8 或后续 gate 的用户决策 ID。 |

当前豁免数量：0。

## 5. Fail-closed 规则

| 规则 ID | 条件 | 结果 |
|---|---|---|
| PAG-GATE-001 | preserve-audit 路径进入 `replace_candidate` 或 `move_candidate` | FAIL，改为 `preserve_audit`。 |
| PAG-GATE-002 | 敏感路径 / 引用进入 candidate list | FAIL，改为 `blocked_sensitive` 或 `excluded`。 |
| PAG-GATE-003 | 单项豁免缺字段 | FAIL，保持 preserve-audit。 |
| PAG-GATE-004 | 需要读取正文才能判断 | FAIL，转 `manual_review`；当前不读正文。 |

## 6. 未授权项

本文不授权历史 `process/**`、checks、checkpoints、handoffs、Story、LLD 或 `DEV-LOG.md` 的机械改写，不授权凭据读取，不授权 NAS / lake / provider / QMT / MiniQMT / git remote 操作，不恢复 CR046。
