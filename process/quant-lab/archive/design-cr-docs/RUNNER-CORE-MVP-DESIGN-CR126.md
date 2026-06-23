---
title: "Runner Core MVP Design / CR128 Scope"
source_cr: "CR-126"
target_cr: "CR-128"
status: "ready-for-cr128-intake"
retained_as: "cr128_implementation_intake_source_design"
feature_authority: "process/docs/features/strategy-runner-core/DESIGN.md"
owner: "host-orchestrator"
created_at: "2026-06-23"
authorization: "process/docs design only; no source, test, runtime or external access authorized"
---

# Runner Core MVP Design / CR128 Scope

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-23 | host-orchestrator | 基于 CR126 CP2/CP5 approve 生成压缩版 runner core MVP 范围。 |
| v0.2 | 2026-06-23 | host-orchestrator | CR127 已用于 State v2 / Process Retention Policy Landing，runner core implementation 顺延为 CR128。 |
| v0.3 | 2026-06-23 | host-orchestrator | CR130 明确本文是 CR128 implementation intake/source design，不是长期 feature authority；长期权威指向 `strategy-runner-core`。 |

## 文档定位

| 项 | 边界 |
|---|---|
| 当前定位 | 本文是 CR126/CR128 runner core MVP 的 implementation intake / source design，用于解释 CR128 为什么按 offline-first runner core 落地。 |
| 长期权威 | 长期 Feature authority 归 `process/docs/features/strategy-runner-core/DESIGN.md`。 |
| 本文不负责 | 不作为后续 runner feature 的唯一设计入口，不授权真实 runtime/NAS/QMT/provider/lake/catalog/trading，也不替代 CR129 后的 feature boundary。 |

## 1. 目标

CR128 的目标是一次性完成 offline-first runner core MVP，不再把 runner 本体拆成多个正式 CR。RDS-01、RDS-02、RDS-03 只是 CR128 内部实现 slice，用于组织开发顺序和验收证据。

CR128 默认不触碰外部权限域。若完成既定目标时确实需要 runtime、NAS、QMT、凭据、provider、lake、catalog、publish、simulation 或 live/trading，必须先停止并由 host-orchestrator 发起独立授权门禁；用户已说明未来可以授权外部权限，但当前 CR126/CR128 默认不包含该授权。

## 2. 范围

| 类别 | In Scope for CR128 | Out of Scope / 需独立门禁 |
|---|---|---|
| package intake | 离线读取 manifest、checksum、payload 和 authorization flags。 | NAS mount/list/copy/pull/publish、真实 package exchange。 |
| run spec | 定义 offline mode、run id、package path、output path、forbidden scope、not_authorization。 | 从 `.env`、账号、session、gateway 或 runtime 自动推导配置。 |
| adapter dispatch | 将 multifactor、legacy、strategy_package payload 转为 target portfolio 和 order intent draft。 | 调用 QMT、MiniQMT、XtQuant、provider、lake 或 catalog。 |
| orchestration | 串接 package loader、adapter、readonly boundary、evidence 和 result。 | submit/cancel/buy/sell、simulation/live、真实账户查询。 |
| result/evidence | 输出统一 RunResult、脱敏 evidence summary 和 forbidden counters。 | 保存 raw account、positions、orders、fills、QMT logs、token 或 secret。 |
| CLI/API | 暴露最小 offline runner entry。 | runtime health/capabilities/query_positions 的真实 gateway 集成。 |
| tests | 覆盖 offline happy path、fail-closed、redaction、no-runtime/no-env/no-NAS/no-provider/no-lake/no-trading。 | 真实 runtime smoke、NAS real exchange、provider fetch、lake write、catalog publish。 |

## 3. 模块边界

| 模块 | 候选路径 | 职责 | 约束 |
|---|---|---|---|
| Run Spec | `trading/strategy_runner/run_spec.py` | 定义 runner 输入合同、offline mode、路径、禁止范围和 authorization flags。 | 必须显式 `not_authorization=true`、`qmt_allowed=false`；不得读取 `.env`。 |
| Runner Orchestration | `trading/strategy_runner/runner.py` | 编排 package loader -> adapter -> target portfolio -> order intent -> evidence -> result。 | 默认 fail closed；不得创建真实 transport 或外部 client。 |
| Run Result | `trading/strategy_runner/result.py` | 统一 status、blocked reasons、target count、order intent count、evidence summary、counters。 | 输出只能包含脱敏摘要和计数，不包含原始敏感对象。 |
| Public API | `trading/strategy_runner/__init__.py` | 导出正式 offline runner API。 | 保持向后兼容；不暴露外部权限开关为默认执行入口。 |
| CLI / Script Entry | 优先评估 `quant_lab/cli.py`，否则新增 `scripts/run_strategy_runner.py` | 暴露最小 offline runner 命令。 | 必须显式 offline；不得隐式读取 runtime/env/NAS。 |
| Existing Loader | `trading/strategy_runner/package_loader.py` | 复用 manifest、checksum、payload、authorization flags 校验。 | authorization flags 必须为 false。 |
| Existing Adapter | `trading/strategy_runner/adapters.py` | 复用 multifactor / legacy / strategy_package adapter dispatch。 | 未知 payload 和 order-write capability fail closed。 |
| Existing Readonly Boundary | `trading/strategy_runner/readonly_gateway.py` | 仅作为 fake/default readonly boundary。 | 真实 QMT client 仍需 CR129+ runtime authorization。 |
| Existing Evidence | `trading/strategy_runner/evidence.py` | 复用 redaction 和 forbidden counters。 | redaction 必须 fail closed。 |

## 4. CR128 内部 Slice

| Slice | 目标 | 主要输出 | 验证重点 |
|---|---|---|---|
| S1 Contract | 建立 `RunSpec`、mode、forbidden scope、`RunResult` schema。 | `run_spec.py`、`result.py`、schema fixture。 | invalid mode/path/authorization fail closed；`not_authorization=true`、`qmt_allowed=false`。 |
| S2 Orchestration | 串接 package -> adapter -> target portfolio -> order intent。 | `runner.py`、public API。 | fixture package 完整跑通 offline runner；未知 adapter fail closed。 |
| S3 Evidence | 输出脱敏 evidence summary 和 forbidden counters。 | evidence integration、result counters。 | redaction fail closed；敏感字段不进入结果。 |
| S4 CLI/API | 暴露最小 offline runner entry。 | `__init__.py` 导出、CLI 或 script entry。 | CLI/API 不读取 `.env`、不访问 NAS、不启动 runtime。 |
| S5 Regression | 锁定 CR091/CR098 相关回归和 CR128 测试。 | `tests/test_cr128_runner_core_mvp.py` 等。 | no-runtime/no-env/no-provider/no-lake/no-trading 边界。 |
| S6 Delivery | 形成 CP6/CP7/CP8 证据和交付说明。 | CP6/CP7/CP8 process evidence。 | 验证证据可追溯到 CR126 决策和 CR128 AC。 |

## 5. CR128 验收标准

| # | 验收标准 | 验证方式 |
|---:|---|---|
| 1 | 本地 fixture package 能跑完整 offline runner。 | CR128 单元或契约测试。 |
| 2 | runner 输出统一 `RunResult`。 | result schema 断言。 |
| 3 | 输出包含 target portfolio、order intent count、adapter status、evidence status、forbidden counters。 | happy path fixture 断言。 |
| 4 | `not_authorization=true`。 | RunSpec / RunResult 断言。 |
| 5 | `qmt_allowed=false`。 | RunSpec / boundary test。 |
| 6 | 不读取 `.env`。 | monkeypatch / forbidden read test。 |
| 7 | 不访问 NAS。 | path/transport guard test。 |
| 8 | 不启动 QMT / MiniQMT / XtQuant / gateway。 | import / transport / mock guard test。 |
| 9 | 不调用 provider。 | mock guard test。 |
| 10 | 不写 lake。 | filesystem/catalog guard test。 |
| 11 | 不 publish catalog。 | catalog guard test。 |
| 12 | 不执行 submit/cancel/buy/sell/simulation/live。 | forbidden operation counter / fail-closed test。 |
| 13 | 敏感字段 redaction fail closed。 | redaction negative tests。 |
| 14 | 负向输入 fail closed。 | malformed package、unknown adapter、bad flags tests。 |
| 15 | 最小测试集通过。 | `uv run --python 3.11 pytest ...`，具体范围由 CR128 CP5 决定。 |

## 6. 不授权项

当前 CR126 CP6 和 CR128 默认均不授权以下动作，除非另行发起并通过独立授权门禁：

- 读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 执行 `scripts/collect_cr099_runner_readonly_smoke.py` 或 `scripts/run_cr104_qmt_miniqmt_validation.ps1`。
- 执行 submit/cancel、buy/sell、simulation/live。
- 执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 修改 `.gitignore`。
- commit、push 或 remote write。

## 7. CR129+ 触发条件

| 后续 CR | 触发条件 | 默认状态 |
|---|---|---|
| CR129 Readonly Runtime Authorization Gate | 需要真实 QMT/gateway health、capabilities、query_positions。 | 不启动。 |
| CR130 Readonly Runtime Integration | CR129 已授权，且需要把 readonly runtime result 接入 runner evidence。 | 不启动。 |
| CR131 NAS Package Exchange Integration | runner 需要直接消费或同步真实 NAS package exchange。 | 不启动。 |
| CR132 Provider / Lake / Catalog Integration | runner 需要 provider fetch、lake write、catalog publish/current pointer。 | 不启动。 |
| CR133 Simulation / Live Execution Gate | 用户明确要求 submit/cancel/buy/sell/simulation/live。 | 不启动，高风险。 |

## 8. Gotchas

| 风险 | 规避方式 |
|---|---|
| 把 offline runner 误写成 runtime-ready。 | 所有 public API、CLI 和 evidence 必须保留 `not_authorization=true`、`qmt_allowed=false` 语义。 |
| 把 RDS slice 升级为多个正式 CR。 | RDS-01/02/03 只作为 CR128 内部开发顺序；正式 CR 只有 CR128。 |
| 为了完成测试而读取 `.env` 或真实 NAS/QMT。 | CR128 测试必须使用 fixture、fake transport 和 mock guard；任何外部访问都先停下。 |
| evidence 中泄露敏感字段。 | redaction fail closed，负向样例必须覆盖 token/account/session/raw order/fill/log 字段。 |
| 用户“可以授权外部权限”被误解为当前已授权。 | 当前没有外部动作授权；需要时另起 CR129+ 并再次请求用户确认。 |
