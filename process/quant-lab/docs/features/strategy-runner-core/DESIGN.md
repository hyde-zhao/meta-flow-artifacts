---
status: "active"
version: "1.4"
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
  - "CR-133"
  - "CR-134"
  - "CR-135"
  - "CR-136"
---

# Feature Design: Strategy Runner Core

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-23 | host-orchestrator | 从 CR091/CR098/CR101/CR126 收敛 offline runner core；将 CR102/103/104 运行时验证碎片移出默认 feature 面。 |
| 1.1 | 2026-06-23 | host-orchestrator | CR133 增加 offline RunSpec JSON/YAML 文件入口和 `python -m trading.strategy_runner.cli` CLI。 |
| 1.2 | 2026-06-23 | host-orchestrator | CR134 增加 successful offline run 的 lightweight evidence index 输出。 |
| 1.3 | 2026-06-23 | host-orchestrator | CR135 增加 successful offline run artifact bundle 与 bundle inspect/replay，只读取本地 bundle，不重新执行 strategy package。 |
| 1.4 | 2026-06-23 | host-orchestrator | CR136 增加 artifact bundle schema / compatibility validation；缺文件、坏 JSON、hash/size 不一致、状态/权限边界不一致均 fail closed。 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 提供 offline-first strategy runner core，将本地 strategy package 转换为 target portfolio、order intent draft、脱敏 evidence、统一 RunResult、轻量 evidence index、本地 run artifact bundle 和可 fail-closed 校验的 bundle 合同。 |
| 推荐方案 | `RunSpec` + package loader + adapter registry + fake readonly boundary + evidence summary + `RunResult` + evidence index + artifact bundle + bundle validation。 |
| 默认权限 | `not_authorization=true`、`qmt_allowed=false`；不读 env、不访问 NAS、不启动 runtime、不交易。 |
| 下游 CR | CR128 实现；外部权限域必须另起 CR129+。 |

## 范围

| In Scope | Out of Scope |
|---|---|
| 离线读取 manifest、checksum、payload。 | NAS mount/list/copy/publish/pull/delete。 |
| 根据 adapter contract 生成 target portfolio 和 order intent draft。 | QMT / MiniQMT / XtQuant / gateway runtime。 |
| 通过 fake readonly boundary 生成脱敏 evidence summary。 | 真实 account / positions / orders / fills / logs。 |
| 输出统一 `RunResult`，包含 status、blocked reasons、counts、evidence summary 和 forbidden counters。 | provider fetch、lake write、catalog publish/current pointer。 |
| CLI / script entry 只运行 offline fixture/package；支持 JSON/YAML RunSpec 文件入口。 | submit/cancel/buy/sell/simulation/live。 |
| successful run 可输出 lightweight evidence index，索引引用 `RunResult` 与脱敏 evidence summary 摘要。 | blocked run 写入误导性 pass index；索引复制完整 target portfolio / order details / raw evidence 正文。 |
| successful run 可输出 run artifact bundle，包含 `run-result.json`、`runner-evidence.index.json`、`run-spec.snapshot.json`、`manifest.json`。 | replay / inspect 重新执行 strategy package、访问外部系统，或 blocked run 写入 misleading pass bundle。 |
| inspect / replay / validate 会校验 bundle schema、required files、bytes/sha256、run id/status/pass、授权边界和 evidence ref 一致性。 | 对缺文件、坏 JSON、不兼容 schema、hash/size 不一致、状态不一致或授权边界异常的 bundle 继续读取或声明可 replay。 |

## 模块职责

| 模块 | 路径 | 职责 | 约束 |
|---|---|---|---|
| RunSpec | `trading/strategy_runner/run_spec.py` | 定义 runner 输入合同、offline mode、路径与权限 flags。 | `mode=offline`；外部授权 flags 必须 false。 |
| Runner | `trading/strategy_runner/runner.py` | 编排 package loader、adapter、fake readonly boundary、evidence 和 result。 | 不创建真实 transport，不读取 env，不访问外部系统。 |
| RunResult | `trading/strategy_runner/result.py` | 稳定输出 status、count、blocked reasons 和 summary。 | 只包含脱敏摘要和计数。 |
| Evidence Index | `trading/strategy_runner/evidence_index.py` | 由 successful `RunResult` 生成轻量 evidence index JSON。 | 只写引用、状态、计数、授权边界和脱敏 evidence summary excerpt；blocked run 不写。 |
| Artifact Bundle | `trading/strategy_runner/artifact_bundle.py` | 由 successful offline run 生成本地 bundle，并支持 validate / inspect / replay 读取。 | 只读写本地 bundle 文件；replay 不重新运行 strategy package；坏 bundle fail closed；blocked run 不写 pass bundle。 |
| CLI | `trading/strategy_runner/cli.py` | 从 JSON/YAML RunSpec 文件启动 offline runner，并提供 bundle inspect / validate。 | 只接收本地 spec 或 bundle；invalid spec / corrupt bundle fail-closed。 |
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
| blocked run 留下看似 pass 的 index。 | runner 只在 `result.passed` 时写 evidence index；blocked spec / blocked package 不写 index。 |
| replay 被误解为重新执行策略包。 | CR135 replay 只读取 bundle 内 `run-result.json`；重新执行必须走新的 runner run。 |
| 坏 bundle 被当成历史事实继续读取。 | CR136 validate 在 inspect/replay 前检查 schema、required files、bytes/sha256、状态一致性和授权边界；失败返回 blocked / raise ValueError。 |
