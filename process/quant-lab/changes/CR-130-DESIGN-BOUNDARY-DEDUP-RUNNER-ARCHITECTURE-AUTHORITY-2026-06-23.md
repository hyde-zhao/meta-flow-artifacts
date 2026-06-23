---
cr_id: "CR-130"
title: "Design Boundary Dedup: Runner Architecture Authority"
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
rollback_to: "pre-CR130 design docs and design-boundary checker state"
approval_result: "user-authorized-design-boundary-dedup"
created_at: "2026-06-23T17:20:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T17:20:00+08:00"
source: "user-request"
parent_cr: "CR-129"
source_checkpoint: "process/checks/CP7-CR129-FEATURE-BOUNDARY-DEDUP-VERIFICATION-DONE.md"
source_decision_id: "USER-20260623-START-CR130"
follow_up_type: "design-boundary-dedup"
risk_class: "design-docs-and-lightweight-guardrail-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "HLD current index no longer treats CR046 as current runner authority; ADR current index marks CR046 as legacy cross-target ADR cluster and adds runner core authority; CR046 HLD/ADR include legacy markers and Strategy Runner Core boundary; CR126 runner MVP design is identified as CR128 implementation intake/source design rather than long-term feature authority; lightweight checker/tests pass; no runtime/NAS/QMT/credential/provider/lake/catalog/trading/Git remote write is performed."
close_condition: "CP6/CP7 verification passes and CR130 evidence records design docs, checker/test changes and non-authorized boundaries."
conflict_keys: ["runner-architecture-authority", "design-doc-boundary", "cr046-legacy-cross-target", "cr126-runner-source-design"]
impact_surface: ["process/docs/design", "scripts", "tests", "process/checks", "process/context"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_REMOTE_WRITE", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: []
cr_index_path: "process/changes/CR-INDEX.yaml"
historical_baseline_status: "closed"
---

# CR-130 Design Boundary Dedup: Runner Architecture Authority

## 变更描述

用户授权启动 CR130，用于收口 design 目录中 CR 命名 HLD / ADR 与 `strategy-runner-core` 的权威边界。目标不是重写架构，而是把长期 design index、CR046 历史双目标框架、CR126 runner MVP source design 和 CR128/CR129 后的 offline runner implementation authority 对齐。

授权范围只覆盖 design 文档边界、轻量检查脚本 / 测试和过程证据；不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或 Git remote write。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/docs/design/HLD.md` | 原文档更新 | 保留 HLD current index，更新 runner authority 关系 | 文档内 `## 修订记录` | approved |
| `process/docs/design/ARCHITECTURE-DECISION.md` | 原文档更新 | 保留 ADR current index，标注 CR046 legacy cluster 并新增 runner core authority row | 文档内 `## 修订记录` | approved |
| `process/docs/design/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | 原文档更新 | 保留 CR046 历史 HLD，降级为 legacy cross-target framework | 文档内 `## 修订记录` | approved |
| `process/docs/design/ARCHITECTURE-DECISION-CR046.md` | 原文档更新 | 保留 CR046 ADR 历史结论，增加权威转移说明 | 文档内 `## 修订记录` | approved |
| `process/docs/design/RUNNER-CORE-MVP-DESIGN-CR126.md` | 原文档更新 | 保留为 CR128 implementation intake/source design，不作为长期 feature authority | 文档内 `## 修订记录` | approved |
| `scripts/check_design_runner_boundary.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `tests/test_cr130_design_runner_boundary.py` | 新增 | N/A | CP6/CP7 证据 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否改变 runner core 功能需求 | runner architecture authority | false | 只收敛 design 权威入口，不改 CR128 runner 行为。 |
| 场景层 | 是否新增 runtime / trading 场景 | CR046 / CR126 设计边界 | false | 保留历史追溯，新增不授权边界说明。 |
| 计划层 | 是否改变后续 CR 路由 | CR129+ runtime/NAS/QMT/provider/lake/catalog/trading gates | true | 明确必须另起授权门禁，不从 CR046/CR126 设计文档直接恢复。 |
| 安全层 | 是否触发外部权限 | runtime/NAS/QMT/credential/provider/lake/catalog/trading | false | 全部继续未授权；轻量检查只扫描本地文档。 |
| 交付层 | 是否新增可验证资产 | design boundary checker/test/process evidence | true | 新增 checker/test、CP6/CP7 证据和 context。 |

## 实现范围

| Slice | 输出 | 验证 |
|---|---|---|
| S1 HLD index boundary | `HLD.md` v1.2 / CR130 runner authority section | checker |
| S2 ADR index boundary | `ARCHITECTURE-DECISION.md` v1.3 / runner core authority row | checker |
| S3 CR046 legacy markers | CR046 HLD/ADR legacy cross-target markers and boundary section | checker |
| S4 CR126 source design marker | CR126 runner MVP design marked as CR128 implementation intake/source design | checker |
| S5 Guardrail | `scripts/check_design_runner_boundary.py` and tests | pytest / script |
| S6 Evidence | CR130 summary/context/CP6/CP7 | meta-flow checks |

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
- 用户已授权：design 文档边界、轻量检查脚本 / 测试和过程证据。
- 用户未授权：runtime/NAS/QMT/凭据/provider/lake/catalog/trading/Git remote write。
- 关闭证据：`process/checks/CP6-CR130-DESIGN-BOUNDARY-DEDUP-IMPLEMENTATION-DONE.md`、`process/checks/CP7-CR130-DESIGN-BOUNDARY-DEDUP-VERIFICATION-DONE.md`、`process/archive/CR-130/evidence-index.json`。
