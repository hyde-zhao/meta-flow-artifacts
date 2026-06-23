---
cr_id: "CR-131"
title: "Design Surface Archive Cleanup"
cr_type: "process"
cr_kind: "implementation-gate"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "compact"
status: "closed-current-delivery"
impact_level: "low"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR131 design root and archive-index state"
approval_result: "user-authorized-design-surface-cleanup"
created_at: "2026-06-23T17:55:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T17:55:00+08:00"
source: "user-request"
parent_cr: "CR-130"
source_checkpoint: "process/checks/CP7-CR130-DESIGN-BOUNDARY-DEDUP-VERIFICATION-DONE.md"
source_decision_id: "USER-20260623-START-CR131"
follow_up_type: "design-surface-archive-cleanup"
risk_class: "design-docs-archive-only-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "CR-named historical design docs are removed from the default process/docs/design surface and retained under process/archive/design-cr-docs; default design root keeps only current canonical design entries and archive index; HLD/ADR runner authority checks still pass against archived CR046/CR126 refs; design surface checker prevents CR-named docs from reappearing in default design root; checks pass; no runtime/NAS/QMT/credential/provider/lake/catalog/trading/Git remote write is performed."
close_condition: "CP6/CP7 verification passes and CR131 evidence records archive moves, index/checker/test changes and non-authorized boundaries."
conflict_keys: ["design-surface-cleanup", "cr-named-design-archive", "design-root-current-authority"]
impact_surface: ["process/docs/design", "process/archive/design-cr-docs", "scripts", "tests", "process/checks", "process/context"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_REMOTE_WRITE", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: []
cr_index_path: "process/changes/CR-INDEX.yaml"
historical_baseline_status: "closed"
---

# CR-131 Design Surface Archive Cleanup

## 变更描述

用户指出 design 目录仍有大量冗余 CR 命名设计文档，要求继续治理完。CR131 将默认 design surface 收敛为当前权威入口，把 CR 命名历史 HLD / ADR / Feature Matrix / governance notes 移入 `process/archive/design-cr-docs/` 并建立归档索引。

本 CR 不删除历史设计正文；仅移动默认入口位置、更新引用、增加检查器，防止 CR 命名历史文档再次污染默认 design 根目录。

授权范围只覆盖 design 文档归档、归档索引、轻量检查脚本 / 测试和过程证据；不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或 Git remote write。

## 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/docs/design/*CR*.md` | 移动到归档区 | 保留完整文件于 `process/archive/design-cr-docs/` | `process/archive/design-cr-docs/ARCHIVE-INDEX.md` | approved |
| `process/docs/design/HLD.md` | 原文档更新 | 默认入口保留，CR046/CR126 引用改为归档路径 | 文档内 `## 修订记录` | approved |
| `process/docs/design/ARCHITECTURE-DECISION.md` | 原文档更新 | 默认入口保留，CR046/CR126 引用改为归档路径 | 文档内 `## 修订记录` | approved |
| `scripts/check_design_runner_boundary.py` | 原文档更新 | 支持 archived CR046/CR126 refs | CP6/CP7 证据 | approved |
| `scripts/check_design_surface.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `tests/test_cr131_design_surface.py` | 新增 | N/A | CP6/CP7 证据 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否改变产品/runner 需求 | design docs surface | false | 只改变文档入口与归档位置，不改需求。 |
| 场景层 | 是否改变运行/验证场景 | historical CR design docs | false | 历史文档保留在 archive，可追溯。 |
| 计划层 | 是否改变后续 CR 路由 | future design docs | true | 默认 design root 只允许 current canonical entries；CR-specific 设计进入 archive 或对应 CR 证据。 |
| 安全层 | 是否触发外部权限 | runtime/NAS/QMT/credential/provider/lake/catalog/trading | false | 全部继续未授权。 |
| 交付层 | 是否新增可验证资产 | design surface checker/test/process evidence | true | 新增 checker/test 和 CP6/CP7 证据。 |

## 不授权范围

- 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不修改 `.gitignore`。
- 不执行 commit、push 或 remote write。

## 处理结论

- 当前状态：closed-current-delivery / ready。
- 用户已授权：design 文档归档、归档索引、轻量检查脚本 / 测试和过程证据。
- 用户未授权：runtime/NAS/QMT/凭据/provider/lake/catalog/trading/Git remote write。
- 关闭证据：`process/checks/CP6-CR131-DESIGN-SURFACE-ARCHIVE-CLEANUP-IMPLEMENTATION-DONE.md`、`process/checks/CP7-CR131-DESIGN-SURFACE-ARCHIVE-CLEANUP-VERIFICATION-DONE.md`、`process/archive/CR-131/evidence-index.json`。
