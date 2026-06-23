---
cr_id: "CR-126"
title: "Quant-Lab Runner Development Readiness / Runner Entry Design"
cr_type: "process"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "production"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR126 process state; no source/runtime/external state changes are authorized"
source: "user-request"
parent_cr: "CR-125"
source_checkpoint: "process/checkpoints/CP8-CR125-DELIVERY-READINESS.md"
follow_up_type: "runner-development-readiness-runner-entry-design"
risk_class: "medium-runner-entry-design-and-runtime-boundary"
owner: "host-orchestrator"
created_at: "2026-06-23T13:40:14+08:00"
updated_at: "2026-06-23"
acceptance_criteria: "CP2 confirms CR126 scope, non-authorized boundaries and current runner/adapter read-only discovery. CP5 confirms CR126 CP6 process/docs design authorization. CP6 produces Runner Core MVP Design / CR127 Scope. CP7 verifies the design statically. CP8 confirms closure."
close_condition: "CP8 approved or rejected by user; no runtime, NAS, QMT, credentials, provider/lake/catalog, source/checker/tests, .gitignore, commit or push actions occur in this gate-prep CR."
---

# CR-126 Quant-Lab Runner Development Readiness / Runner Entry Design

## 变更来源

用户要求停止继续清理历史文档，进入 quant-lab runner 后续开发前的最小入口设计与范围确认。当前只做 CP2/CP5 门禁准备，梳理 runner 当前代码入口、adapter 边界、第一批可执行开发 slice 和不授权项。

## 当前目标

1. 将 CR125 关闭后的工作重心从历史文档治理切换到 runner development readiness。
2. 只读复盘当前 runner 入口和 adapter 边界，不启动 runtime、不访问外部系统。
3. 冻结 runner core MVP 的集中实现范围：CR127 作为唯一 runner 本体实现 CR，RDS-01/02/03 仅作为 CR127 内部执行 slice。
4. 明确本 CR 不授权项，避免把 runner 入口设计误扩展为 QMT/NAS/provider/lake/catalog 或真实交易。

## 最新压缩规划

用户已纠正：runner 本身是功能集中的模块，不应把 `run_spec`、runner entry、result、evidence、安全测试拆成多个正式 CR。当前规划收敛为：

| CR | 定位 | 状态 | 授权边界 |
|---|---|---|---|
| CR126 | Runner Entry Design / Scope Gate | 当前 active | 只冻结 runner MVP 范围、模块边界、禁止项和 CR127 实现范围；不改源码、不运行测试、不启动 runtime。 |
| CR127 | Runner Core MVP Implementation | 未启动 | 集中完成 offline runner core：run spec、orchestration、result/evidence、CLI/API、测试、CP7 验证与交付；默认仍不授权 runtime/NAS/QMT/provider/lake/catalog/trading。 |
| CR128+ | 外部权限域可选后续 | 不启动 | 仅当用户明确要求触碰 readonly runtime、NAS exchange、provider/lake/catalog、simulation/live 等外部权限域时才创建。 |

## 当前只读发现

| 对象 | 路径 | 当前职责 | 本轮结论 |
|---|---|---|---|
| runner package API | `trading/strategy_runner/__init__.py` | 对外导出 package loader、adapter、readonly gateway、evidence、package exchange 等接口。 | 可作为后续 runner 入口收敛的发现面，但本轮不改。 |
| package loader | `trading/strategy_runner/package_loader.py` | 离线读取 manifest / checksum / payload，要求 authorization flags 为 false，禁止 order-write capabilities。 | 适合作为第一批 runner intake contract 基线。 |
| adapter registry | `trading/strategy_runner/adapters.py` | 将 multifactor / legacy / strategy_package payload 转为 target portfolio 与 order intent draft；未知输入 fail closed。 | 适合作为 runner adapter 边界基线。 |
| readonly gateway wrapper | `trading/strategy_runner/readonly_gateway.py` | 默认 fake transport；仅允许 health / capabilities / query_positions；真实 QMT client 缺少 runtime authorization 时 blocked。 | 只作为边界说明；不在本 CR 触发真实 client。 |
| evidence summary | `trading/strategy_runner/evidence.py` | 生成脱敏 evidence summary，聚合 forbidden operation counters，敏感字段 fail closed。 | 适合作为后续 runner evidence contract 基线。 |
| offline checker | `scripts/check_cr091_strategy_runner_package.py` | 使用 package loader、adapter、fake readonly gateway 和 evidence 进行离线 package check。 | 可作为后续最小离线入口候选；本轮不执行。 |
| runtime collector | `scripts/collect_cr099_runner_readonly_smoke.py` | 真实 readonly smoke collector，会读取 env 并访问 gateway。 | 明确冻结为禁止项；本 CR 不执行、不改、不作为 CP6 输入。 |
| CR091/CR098 tests | `tests/test_cr091_strategy_runner_contracts.py`, `tests/test_cr098_runner_readonly_integration.py` | 覆盖 package/adapter/readonly/evidence fail-closed 行为。 | 可作为后续最小回归候选；本轮不运行测试。 |

## 建议入口边界

| 层级 | 建议入口 | 允许用途 | 禁止扩展 |
|---|---|---|---|
| Intake | `load_strategy_package()` / `StrategyPackage.to_adapter_payload()` | 离线 package manifest、checksum、authorization flags、payload contract。 | 不读 NAS、不拉包、不发布、不读凭据。 |
| Adapter | `adapt_strategy_payload()` / `AdapterRegistry` | 离线 payload 到 target portfolio / order intent draft 的转换。 | 不调用 QMT、provider、lake、catalog 或 runtime。 |
| Readonly boundary | `ReadonlyGatewayClient()` 默认 fake path | 仅用于说明现有 fake/offline contract；后续若执行也必须显式离线。 | 不用 `from_transport()`，不传真实 base_url、auth provider 或 env。 |
| Evidence | `build_evidence_summary()` | 只记录脱敏 summary 和 forbidden counters。 | 不保存 raw account、positions、orders、fills、QMT logs、secret。 |

## CR127 内部执行 Slice 候选

| Slice | 建议优先级 | 内容 | 当前授权 |
|---|---:|---|---|
| RDS-01 Contract | 1 | `RunSpec` / mode / forbidden scope / result schema。 | CR126 CP6 仅写设计材料；源码实现归入 CR127。 |
| RDS-02 Orchestration | 2 | package loader -> adapter -> target portfolio -> order intent。 | CR127 内部实现 slice，不单独升级为正式 CR。 |
| RDS-03 Evidence / CLI / Regression | 3 | evidence summary / counters / redaction、offline CLI/API、CR091/CR098 相关回归和 CR127 测试。 | CR127 内部实现与验证 slice；本 CR 不跑测试。 |
| External Gates | N/A | readonly runtime、NAS exchange、provider/lake/catalog、simulation/live。 | 不属于 runner core MVP；仅用户明确要求触碰外部权限域时启动 CR128+。 |

## 当前不授权项

| 类别 | 状态 | 说明 |
|---|---:|---|
| runtime / QMT / MiniQMT / XtQuant / gateway | false | 不启动、不连接、不 query、不执行 collector。 |
| NAS / package real exchange | false | 不 mount、不 list、不 copy、不 publish、不 pull。 |
| credential / `.env` / token / account / session | false | 不读取、不打印、不引用原文。 |
| provider / lake / catalog / publish | false | 不 fetch、不 write、不 publish、不改 current pointer。 |
| source / checker / tests / `.gitignore` 修改 | false | 本 CR 只写 process 门禁材料。 |
| git add / commit / push / remote write | false | 本 CR 不纳入发布。 |
| submit / cancel / buy / sell / simulation / live | false | 必须另起高风险 gate。 |

## 五维度影响分析

| 维度 | 影响 | 结论 |
|---|---|---|
| 需求层 | true | 从文档清理切换到 runner 开发入口范围确认。 |
| 场景层 | true | 覆盖离线 package intake、adapter dispatch、readonly boundary、evidence summary。 |
| 计划层 | true | 建议首批 RDS-01..04，但本轮只准备 CP2/CP5。 |
| 安全 / 权限层 | true | 明确冻结 runtime、NAS、凭据、provider/lake/catalog、交易写操作。 |
| 交付层 | true | 输出后续 runner 开发最短路径入口，而不是继续横向历史文档清理。 |

## 文档处理决策

| 对象 | 当前处理决策 | 后续要求 |
|---|---|---|
| 历史 docs cleanup | 停止横向推进 | 除非直接阻塞 runner 开发，不再启动清理型 CR。 |
| CR102/103/104 验证记录 | 保留为历史证据输入 | 只在 runner 设计需要时读取摘要，不再默认清理。 |
| runner design artifacts | 待 CP5 决策 | CP5 approve 后可写 process/docs 设计材料；源码实现集中到 CR127 Runner Core MVP Implementation。 |

## 冲突预检

| 冲突对象 | 状态 | 处理结论 |
|---|---|---|
| CR125 历史文档清理 | resolved | CR125 已 CP8 关闭，当前不再恢复横向清理。 |
| CR099 / CR104 runtime validation | frozen | 仅作为边界和历史证据，不执行 collector、PowerShell 或 runtime。 |
| QMT / NAS / provider / lake / catalog | blocked-by-scope | 本 CR 明确不授权。 |
| 源码实现 | blocked-by-gate | CP2/CP5 gate prep 不授权源码/checker/tests 修改。 |

## 推荐后续路径

1. 用户审查 CP2/CP5 决策项。
2. 若 `approve`，进入 CR126 后续 CP6 设计材料阶段，仅生成压缩版 Runner Core MVP Design / CR127 Scope、slice plan、regression plan 等 process/docs 产物。
3. CR126 完成后再启动 CR127 Runner Core MVP Implementation，一次性实现 offline runner core；RDS-01/02/03 是 CR127 内部 slice，不再拆成多个正式 CR。
4. runtime / QMT / NAS / provider/lake/catalog / simulation/live 仍不属于 runner core MVP，只有用户明确要求触碰外部权限域时才启动 CR128+。

## CP6/CP7/CP8 证据摘要

| 阶段 | 路径 | 结论 | 说明 |
|---|---|---|---|
| CP6 | `process/checks/CP6-CR126-RUNNER-CORE-MVP-DESIGN-DONE.md` | PASS | 已生成 `process/docs/design/RUNNER-CORE-MVP-DESIGN-CR126.md`。 |
| CP7 | `process/checks/CP7-CR126-RUNNER-CORE-MVP-DESIGN-VERIFICATION-DONE.md` | PASS | static-only verification；未运行源码测试或 runtime。 |
| CP8 | `process/checkpoints/CP8-CR126-DELIVERY-READINESS.md` | approved | 用户终验通过；CR126 关闭，不启动 CR127。 |
