---
id: CR-101
title: Cross-Platform Strategy Delivery and Adapter Layer Realignment
status: closed-current-delivery
lifecycle_status: closed
readiness_status: ready_with_risk
gate_status: closed
cr_kind: architecture-realignment
gate_profile: standard
created_at: "2026-06-20T00:30:00+08:00"
owner: host-orchestrator
source_candidate: FU-CR091-005
legacy_candidate_id: CR091-FU-05
source_tracking: process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md
parent_cr: CR-091
workflow_mode: standard
impact_level: medium-high
rollback_to: "CR091 closed-current-delivery / CR100 closed-current-delivery / CR046 historical framework-first baseline"
current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
approval: "CP2 approved by user reply '同意' at 2026-06-20T08:17:33+08:00; accepts DQ-CP2-CR101-01..05 and does not authorize runtime/NAS/credential/trading/publish execution."
cp3_hld: "docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md"
cp3_gate: "approved by user reply '批准' at 2026-06-20T08:44:49+08:00; accepts DQ-CP3-CR101-01..05 and does not authorize runtime/NAS/credential/trading/publish execution."
cp5_lld_batch: "approved by user reply '批准' at 2026-06-20T09:30:00+08:00; accepts DQ-CP5-CR101-01..05 and allows restricted offline implementation only. No NAS, credential, QMT/MiniQMT/XtQuant/gateway runtime, simulation/live, trading or publish is authorized."
cp6_s01: "PASS at 2026-06-20T09:40:30+08:00; process/checks/CP6-CR101-S01-target-taxonomy-manifest-contract-CODING-DONE.md"
cp7_s01: "PASS at 2026-06-20T09:40:30+08:00; process/checks/CP7-CR101-S01-target-taxonomy-manifest-contract-VERIFICATION-DONE.md"
cp6_s02: "PASS at 2026-06-20T09:53:27+08:00; process/checks/CP6-CR101-S02-package-checker-fixture-fail-closed-CODING-DONE.md"
cp7_s02: "PASS at 2026-06-20T09:53:27+08:00; process/checks/CP7-CR101-S02-package-checker-fixture-fail-closed-VERIFICATION-DONE.md"
cp6_s03: "PASS at 2026-06-20T09:53:27+08:00; process/checks/CP6-CR101-S03-runner-adapter-evidence-boundary-CODING-DONE.md"
cp7_s03: "PASS at 2026-06-20T09:53:27+08:00; process/checks/CP7-CR101-S03-runner-adapter-evidence-boundary-VERIFICATION-DONE.md"
cp6_s04: "PASS at 2026-06-20T10:00:48+08:00; process/checks/CP6-CR101-S04-docs-validation-and-follow-up-gates-CODING-DONE.md"
cp7_s04: "PASS at 2026-06-20T10:00:48+08:00; process/checks/CP7-CR101-S04-docs-validation-and-follow-up-gates-VERIFICATION-DONE.md"
cp8_delivery_readiness: "approved by user reply '批准，并推送到远端' at 2026-06-20T10:13:18+08:00; accepts DQ-CP8-CR101-01..05 and closes CR101 current offline delivery as READY_WITH_RISK. This approval does not authorize real NAS, credential/account/raw-log read, QMT/MiniQMT/XtQuant/gateway runtime, simulation/live, trading, provider/lake/catalog publish, CR089/CR020/order-write auto-start, or follow-up real validation gate execution."
---

# CR-101 Cross-Platform Strategy Delivery and Adapter Layer Realignment

## 变更摘要

本 CR 将 `FU-CR091-005` / legacy `CR091-FU-05` 从 follow-up candidate 转为正式 CR。目标是在不访问真实 NAS、不读取凭据、不启动 QMT / MiniQMT / XtQuant / gateway runtime、不执行 simulation / live / 交易或 publish 的边界内，重对齐两条架构线：

- 策略交付按跨平台 target taxonomy 设计；当前唯一 implemented target 是 `qmt_terminal_direct` / QMT direct-run。
- `strategy_runner` 是 quant-lab 侧 runner；MiniQMT 只是当前 broker / execution adapter target，不是 runner 宿主。
- manifest / entrypoint / validation / package layout 保留 future target / adapter 扩展位，但本轮不实现 Goldminer 或其他运行平台。
- checker、fixture、包结构和 evidence redaction 需要从历史 `miniqmt_runner` 默认假设重对齐到 target / adapter 双边界。

## 启动冲突预检

| 检查项 | 结果 | 证据 | 处理结论 |
|---|---|---|---|
| workspace route 健康 | PASS | `meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` -> `process_link_health: ok` | 允许写入 process 账本 |
| CR tracking 严格检查 | PASS | `meta-flow check cr-tracking --strict-warnings` -> `OK` | 无 active formal CR |
| 正式 CR ID 可分配 | PASS | 现有最高正式 CR 为 `CR-100` | 分配 `CR-101` |
| 与 `CR-089` 冲突 | PASS_WITH_RISK | `CR-089` 为 `blocked-readiness-approved`，影响面包含 QMT interface / NAS exchange / runtime authorization | 本轮不恢复 CR089、不连接真实系统；只做 target / adapter 边界重对齐 |
| 与 `FU-CR091-003` 冲突 | PASS | `FU-CR091-003` blocked_by `FU-CR091-005` | order-write / submit-cancel 继续暂缓 |
| 与 `RA-CR097-001` 冲突 | PASS | runtime authorization candidate，未启动 | 本轮不做 real readonly retest |
| 权限 / 安全边界 | PASS | 用户显式不授权 NAS、凭据、runtime、simulation/live、交易或 publish | CP2 必须保留 runtime_authorization 决策项 |

## 五维度影响分析

| 维度 | 是否影响 | 结论 |
|---|---:|---|
| 需求层 | 是 | 当前需求真相已由 `CURRENT-REQUIREMENT-BASELINE.yaml` 重解释：QMT direct-run 是当前策略交付 target，MiniQMT 是当前 adapter target。 |
| 场景层 | 是 | 本轮覆盖离线 target / adapter contract、manifest schema、package layout、checker 和 redacted evidence；不覆盖真实 QMT / NAS / gateway run。 |
| 计划层 | 是 | `FU-CR091-005` 转正式 `CR-101`；`CR-089`、`FU-CR091-003`、`RA-CR097-001` 继续等待独立门禁。 |
| 安全层 | 是 | 不授权 NAS、凭据、账户 / 持仓 / 委托 / 成交 / 原始日志读取，不启动 runtime，不交易，不 publish。 |
| 交付层 | 是 | 预计后续影响 `docs/qmt/*HLD*`、manifest / checker / fixture / `trading/strategy_runner/*` 相关离线代码和测试；CP2 通过前不实施。 |

## 文档处理决策

| 文档 / 对象 | 处理方式 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md` | 原文档更新 | 保留候选行，改为 active 并链接 CR101 | 记录候选转正式。 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR091 / CR089 / CR100 历史行，新增或更新 CR101 active 行 | 机器状态同步。 |
| `process/STATE.md` | 原文档更新 | 保留历史状态，更新 `active_change` / `cr_tracking` / human gate pending | 标记 CP2 pending。 |
| `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | 不变 | 当前基线已表达本 CR 的启动真相 | 如 CP2 修改范围，再追加修订。 |
| `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md` | 按需原文档更新或新 HLD 替代 | 保留历史 HLD 为 CR091 证据 | CP3 决定是否新增 CR101 HLD。 |
| `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md` | 按需引用 | 保留为 readonly adapter 历史证据 | 仅在 CP3 / CP5 需要时读取。 |
| `trading/strategy_runner/*` | CP5 后按范围更新 | 代码历史由 Git / CR101 实现证据保留 | CP2 不改代码。 |

## 当前推荐范围

| 范围项 | 推荐处理 |
|---|---|
| target taxonomy | 定义 `delivery_targets[]`，当前 implemented 仅 `qmt_terminal_direct`，future / deferred target 只保留 schema 扩展位。 |
| strategy core | 平台无关，不 import QMT / MiniQMT / XtQuant / Goldminer SDK。 |
| QMT direct-run target | 当前唯一策略交付 target；后续提供 entrypoint / contract / smoke checklist / 脱敏 evidence schema。 |
| quant-lab runner adapter layer | 定义 runner -> broker / execution adapter protocol；runner 只依赖 protocol。 |
| MiniQMT gateway adapter | 当前唯一 runner adapter target；只作为 API gateway adapter，不承载策略运行。 |
| Goldminer / other adapters | future extension point；本轮不实现、不安装、不运行。 |
| manifest / checker | 从 `miniqmt_runner` 强制项改为 target / adapter 可扩展 schema；校验当前 QMT target 和 MiniQMT adapter contract。 |
| evidence redaction | 修复否定性审计字段误触发风险，避免凭据、账号、持仓、委托、成交或原始日志进入 evidence。 |
| package generation | 后续重新生成本地验证包；不再默认包含 `targets/miniqmt_runner/entry.py`。 |

## 不授权范围

- 不访问、列取、读取、复制、写入、挂载、发布或删除真实 NAS。
- 不读取 Windows/Linux `.env`、NAS 凭据、token、secret、账号、账户、资金、持仓、委托、成交或原始日志。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不由 agent 代跑 QMT direct-run 或 MiniQMT gateway runtime 验证。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不把 CP2 设计通过解释为运行授权；真实运行和 publish 类动作必须另起逐 run 授权。

## CP2 待决策摘要

| 决策 ID | 类型 | 问题 | 推荐方案 |
|---|---|---|---|
| DQ-CP2-CR101-01 | scope | 是否启动 CR101 并冻结本轮范围？ | 启动 standard CR，只做设计 / schema / fixture / checker / package layout 重对齐。 |
| DQ-CP2-CR101-02 | architecture | runner 与 MiniQMT 的关系如何冻结？ | runner 属于 quant-lab；MiniQMT 是当前 adapter target。 |
| DQ-CP2-CR101-03 | implementation | 当前策略交付 target 支持哪些平台？ | 当前 implemented 仅 QMT direct-run；manifest 保留 future target 扩展位。 |
| DQ-CP2-CR101-04 | runtime_authorization | 本 CR 是否授权真实验证？ | 不授权；真实 QMT / MiniQMT / NAS / gateway / simulation/live 后续逐 run gate。 |
| DQ-CP2-CR101-05 | risk_acceptance | 是否接受先做离线重对齐的风险？ | 接受；本 CR 不声明真实 runtime ready。 |

## 验收标准

| AC | 验收项 | 当前状态 | 证据 |
|---|---|---|---|
| CR101-AC-01 | CP2 明确 scope / risk / runtime authorization，用户确认前不实施代码 | PASS | `process/checkpoints/CP2-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-SCOPE-REVIEW.md` |
| CR101-AC-02 | CP3 HLD 明确 target taxonomy、adapter protocol、manifest schema 和切换条件 | PASS | `docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md`；`process/checkpoints/CP3-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-HLD-REVIEW.md` |
| CR101-AC-03 | CP5 实现设计覆盖受影响文件、fixture、checker 和 package layout | PASS | `process/checkpoints/CP5-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-LLD-BATCH.md`；`process/STORY-BACKLOG-CR101.md`；`process/DEVELOPMENT-PLAN-CR101.yaml`；`process/stories/CR101-S0*-*-LLD.md` |
| CR101-AC-04 | CP6 / CP7 仅执行离线验证，不触碰真实 NAS / runtime / 凭据 / 交易 / publish | PASS | S01-S04 已通过 CP6/CP7；证据见 `process/checks/CP6-CR101-S0*-*.md` 与 `process/checks/CP7-CR101-S0*-*.md`。 |
| CR101-AC-05 | CP8 明确 READY 或 READY_WITH_RISK，并列出后续真实验证 gate | PASS | `process/checkpoints/CP8-CR101-DELIVERY-READINESS.md` 已 approved；S04 已生成 `docs/qmt/CR101-VALIDATION-AND-FOLLOW-UP-GATES.md`。 |

## 当前结论

CR101 当前离线交付已由用户在 CP8 批准关闭为 `closed-current-delivery / READY_WITH_RISK`。CP2、CP3、CP5 和 CP8 均已批准；S01-S04 均已完成受限离线实现与 CP6/CP7 验证。关闭只覆盖离线 contract / checker / evidence / docs 交付，不授权真实 NAS、凭据、账户 / 持仓 / 委托 / 成交 / 原始日志读取、QMT/MiniQMT/XtQuant/gateway runtime、simulation/live、交易、provider/lake/catalog publish，也不自动启动 CR089、CR020、order-write 或后续真实验证 gate。
