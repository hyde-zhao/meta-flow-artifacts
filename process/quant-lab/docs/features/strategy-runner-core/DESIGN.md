---
status: "active"
version: "1.0"
feature_id: "strategy-runner-core"
feature_name: "Strategy Runner Core"
source_cr: "CR-128"
source_design: "process/docs/design/RUNNER-CORE-MVP-DESIGN-CR126.md"
related_crs:
  - "CR-091"
  - "CR-098"
  - "CR-101"
  - "CR-126"
  - "CR-128"
---

# Feature Design: Strategy Runner Core

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-23 | host-orchestrator | 从 CR091/CR098/CR101/CR126 收敛 offline runner core；将 CR102/103/104 运行时验证碎片移出默认 feature 面。 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 提供 offline-first strategy runner core，将本地 strategy package 转换为 target portfolio、order intent draft、脱敏 evidence 和统一 RunResult。 |
| 推荐方案 | `RunSpec` + package loader + adapter registry + fake readonly boundary + evidence summary + `RunResult`。 |
| 默认权限 | `not_authorization=true`、`qmt_allowed=false`；不读 env、不访问 NAS、不启动 runtime、不交易。 |
| 下游 CR | CR128 实现；外部权限域必须另起 CR129+。 |

## 范围

| In Scope | Out of Scope |
|---|---|
| 离线读取 manifest、checksum、payload。 | NAS mount/list/copy/publish/pull/delete。 |
| 根据 adapter contract 生成 target portfolio 和 order intent draft。 | QMT / MiniQMT / XtQuant / gateway runtime。 |
| 通过 fake readonly boundary 生成脱敏 evidence summary。 | 真实 account / positions / orders / fills / logs。 |
| 输出统一 `RunResult`，包含 status、blocked reasons、counts、evidence summary 和 forbidden counters。 | provider fetch、lake write、catalog publish/current pointer。 |
| CLI / script entry 只运行 offline fixture/package。 | submit/cancel/buy/sell/simulation/live。 |

## 模块职责

| 模块 | 路径 | 职责 | 约束 |
|---|---|---|---|
| RunSpec | `trading/strategy_runner/run_spec.py` | 定义 runner 输入合同、offline mode、路径与权限 flags。 | `mode=offline`；外部授权 flags 必须 false。 |
| Runner | `trading/strategy_runner/runner.py` | 编排 package loader、adapter、fake readonly boundary、evidence 和 result。 | 不创建真实 transport，不读取 env，不访问外部系统。 |
| RunResult | `trading/strategy_runner/result.py` | 稳定输出 status、count、blocked reasons 和 summary。 | 只包含脱敏摘要和计数。 |
| Existing Package Loader | `trading/strategy_runner/package_loader.py` | manifest / checksum / authorization flags fail-closed。 | 复用现有合同。 |
| Existing Adapter | `trading/strategy_runner/adapters.py` | multifactor / legacy / strategy_package payload dispatch。 | 未知 payload、order-write capability fail closed。 |
| Existing Readonly Boundary | `trading/strategy_runner/readonly_gateway.py` | 默认 fake readonly result。 | 真实 client 路径不由 CR128 调用。 |
| Existing Evidence | `trading/strategy_runner/evidence.py` | 脱敏 evidence summary 与 forbidden counters。 | redaction fail closed。 |

## Feature 目录治理决策

| 历史对象 | 当前处理 | 理由 |
|---|---|---|
| `process/docs/features/cr102-nas-real-package-exchange-authorization` | 从默认 feature 面移除；历史由 CR102 / CP8 / release context 追溯。 | NAS package exchange 是外部权限域，不属于 offline runner core。 |
| `process/docs/features/cr103-qmt-miniqmt-non-trading-day-validation` | 从默认 feature 面删除。 | 非交易日 runtime validation 是历史验证证据，不应作为长期 feature 入口。 |
| `process/docs/features/cr104-qmt-miniqmt-trading-day-validation` | 从默认 feature 面删除。 | 交易日 runtime input checklist 是授权运行材料，不属于 runner core feature。 |
| `process/docs/features/qmt-miniqmt-dual-target-framework` | 保留为历史 cross-target framework。 | 它描述较大的 QMT/MiniQMT 交付框架；CR128 只落地 offline runner core。 |

## Gotchas

| 风险 | 规避 |
|---|---|
| 把 runner core 误解为 runtime-ready。 | API、CLI、result 均固定 `not_authorization=true` 和 `qmt_allowed=false`。 |
| 为了 evidence 调用真实 gateway。 | CR128 只使用默认 fake readonly boundary。 |
| feature 目录继续混入 runtime 授权碎片。 | CR102/103/104 运行时材料保留在 CR 历史，不作为长期 feature 入口。 |
| result 泄露敏感字段。 | evidence summary 走 `assert_redacted`，负向测试覆盖敏感字段。 |
