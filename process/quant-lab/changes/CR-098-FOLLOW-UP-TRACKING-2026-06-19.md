---
tracking_id: "CR098-FOLLOW-UP-TRACKING"
source_cr: "CR-098"
source_checkpoint: "process/checkpoints/CP8-CR098-DELIVERY-READINESS.md"
status: "active-follow-up-tracking"
created_at: "2026-06-19T12:31:24+08:00"
owner: "host-orchestrator"
---

# CR098 Follow-up Tracking

## 分类摘要

| 分类 | 数量 | 是否阻塞当前交付 | 说明 |
|---|---:|---|---|
| 关闭范围 | 1 | 否 | CR098 当前 runner readonly integration 离线交付已关闭为 `READY_WITH_RISK` |
| 不授权范围 | 1 | 否 | HMAC secret、Windows `.env`、gateway 启动、runner runtime、账户原文、NAS、交易写、simulation/live、provider/lake/publish 均不授权 |
| 风险接受项 | 3 | 否 | 真实 runner runtime 未执行、非空持仓 / 交易日路径未证明、CP6/CP7 inline fallback WAIVED 已由 CP8 接受 |
| 后续 CR 候选项 | 4 | 否 | 推荐下一项为 `CR098-FU-01`；所有候选均未启动，必须由用户在清理上下文后明确启动 |

## 后续 CR / Spike 候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR098-FU-01 | Runner real readonly smoke per-run authorization | candidate-not-started | CR | 1 | runtime_authorization; qmt_runner; readonly_gateway; hmac_client_env; redacted_evidence; credential_boundary |  |  | 未启动 | 需要用户明确启动并逐 run 授权；不得复用 CR097 / CR098 approval 作为运行授权 | 推荐作为清理上下文后的下一个 CR；启动前先读 `process/context/CR098-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-19.md` | DQ-CP8-CR098-02 |
| CR097-FU-01 | Non-empty / trading-day readonly retest | candidate-not-started | CR | 2 | runtime_authorization; readonly_query_positions; evidence_coverage; non_empty_positions; trading_day_path |  |  | 未启动 | 需要非空持仓或交易日条件，且必须独立逐 run 授权 | 等待用户选择 | DQ-CP8-CR098-03 / DQ-CP8-CR097-02 |
| CR091-FU-02 | NAS package exchange gate | candidate-not-started | CR | 3 | nas_package_exchange; package_delivery; credential_boundary; no_runtime_connection |  |  | 未启动 | NAS 访问和 package exchange 不在 CR098 范围内 | 等待用户选择 | CR091 / CR098 not-authorized boundary |
| ORDER-WRITE-FU | Order-write / simulation / live design gate | candidate-not-started | CR | 4 | order_write; submit_cancel; simulation_live; trading_runtime_boundary; account_permission |  |  | 未启动 | 交易写风险最高，必须独立设计、门禁和逐项授权 | 不建议立即启动；需要用户明确提出并接受更高风险门禁 | CR098 not-authorized boundary |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 | 来源 |
|---|---|---|---|---|
| NA-CR098-01 | HMAC secret、Windows `.env`、gateway 启动、runner runtime、账户原文、NAS、交易写、simulation/live、provider/lake/publish、真实 release execution / publish、CR089 auto-start、CR020 gateway route restore | CR098 CP8 approve 只关闭当前离线 runner readonly integration 交付，不授权外部动作 | 另起独立 CR，并在该 CR 内获得明确逐 run 授权 | CP8-CR098 |

## 风险接受项

| 风险 ID | 状态 | 内容 | 后续处理 | 来源 |
|---|---|---|---|---|
| R-CR098-01 | accepted | 真实 runner runtime smoke 未执行 | 需要真实证明时启动 `CR098-FU-01` | CP8-CR098 |
| R-CR098-02 | accepted | 非空持仓 / 交易日路径未证明 | 需要更广覆盖时启动 `CR097-FU-01` 或在 `CR098-FU-01` 中纳入条件说明 | CP8-CR098 |
| R-CR098-03 | accepted | CP6/CP7 使用 host-orchestrator inline fallback，未通过独立子 agent lane | 后续高风险 runtime / order-write gate 建议补独立 review lane | CP8-CR098 |

## 推荐下一步

清理上下文后，推荐先读取 `process/context/CR098-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-19.md`。如果用户明确要求继续推进，则推荐启动 `CR098-FU-01 Runner real readonly smoke per-run authorization`，目标是证明 runner 真实消费 Windows gateway readonly path；该启动不继承 CR098 的 approve，也不自动授权读取 secret/env、启动 runtime 或查询账户。
