---
checkpoint_id: "CP2-CR164-MULTIPLE-TESTING-PBO-DSR-SCOPE"
checkpoint_name: "CR164 Requirement, Scenario and Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-12T19:08:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-12T19:59:27+08:00"
auto_check_result: "process/checks/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-SCOPE.result.json"
decision_brief_profile: "compact"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md", "docs/product/RELEASE-SLICES.md", "docs/product/BACKLOG.md"]
---

# CP2 CR164 Requirement, Scenario and Scope Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-SCOPE.result.json` | PASS | 0 | 15 项检查全部 PASS，waiver=0。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 把 CR161 的 typed-unavailable 统计证据合同与 CR163 sealed experiment-family lineage 连接为可计算、可复跑、可独立验证且 fail-closed 的 BH/WRC-SPA/PBO-CSCV/DSR 证据基线。 |
| 推荐动作 | `approve`：确认 CR164 产品基线、方法范围、量化验收、claim ceiling、兼容边界和 deny-default 路由，进入 CP3 架构设计。 |
| approve 后会发生什么 | 主编排器将真实拉起 `meta-se-critical`，冻结 evidence schema、method calculation/provenance、stationary-bootstrap policy、disagreement decision table、模块边界、Feature design triggers 和正式 Story planning 输入。 |
| approve 不授权什么 | 不授权源码/测试实现、真实统计批次、生产数据/凭据/NAS/provider、外部框架、broker/simulation/live/trading、catalog/store/registry write、部署/tag/publish 或 Git remote write。 |
| 不确认会阻塞什么 | CP3/HLD、正式 Story/DAG、CP5 全量设计证据和后续实现均保持阻断。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。 |
| 全文档读取扩展 | 2 次：CR164 discussion log 与 meta-pm return summary 的 CP2 human audit；均写入 READ-EXPANSION-LEDGER。 |
| 缺失 / waived 理由 | 无；waiver=0。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` / `QUESTION-LEDGER.ndjson` | scanned | 4 | 0 | SGQ-001..004 已 resolved-by-user；等待正式 CP2 整体批准。 |
| 委托 Agent 交还摘要 | `process/handoffs/CR164-CP2-META-PM-RETURN-SUMMARY.md` | scanned | 7 | 0 | 4 个 SGQ resolved；3 个 OPEN 分类为 non-blocking CP3 obligations。 |
| 自动预检结果 | `process/checks/CP1-CR164-*` / `CP2-CR164-*` | scanned | 0 | 0 | CP1/CP2 precheck 均 PASS。 |
| discussion log / checkpoint | `process/discussions/CP2-CR164-*` / `process/checks/CP2-CR164-DISCUSSION-CHECKPOINT.json` | scanned | 4 | 0 | 4/4 confirmed A，remaining=0。 |
| 下游正式产物 | HLD / ADR / Feature DESIGN / LLD | n/a | 0 | 0 | CP2 前尚未生成，按 route 正常。 |
| 用户显式选择题 | `process/context/CR164-CP2-SGQ-BATCH.yaml` | scanned | 4 | 0 | 用户批准修订版 A；仅产品范围，不是 CP2 formal approval。 |
| CP2 基线整体批准 | 当前 checkpoint | scanned | 4 | 4 | 以下 DQ 需要正式 gate approve。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 1 | DQ-CP2-CR164-004 确认 deny-default 与仅设计路由。 |
| agent 默认处理 | 3 | CP3 冻结 DSR schema、bootstrap policy、disagreement reason-code table；不得改变 CP2 claim ceiling。 |
| 仅审计记录 | 4 | SGQ-CR164-001..004 已确认 A。 |

### 已解决 Scenario Gray Areas

| SGQ | 用户结论 | 基线化结果 |
|---|---|---|
| SGQ-CR164-001 | A | BH + WRC/SPA + PBO/CSCV + DSR；mandatory claim-relevant conservative aggregation；no OR-pass。 |
| SGQ-CR164-002 | A | effective count 保持 typed_unavailable；DSR raw-count 来源必须显式且不得 alias effective count。 |
| SGQ-CR164-003 | A | 方法 minima 与 10 项集中量化 AC 冻结。 |
| SGQ-CR164-004 | A | UC-58 implementation；UC-59/60 compatibility-only。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP2-CR164-001` | scope | 是否批准四方法 MVP 与 conservative no-OR-pass claim ceiling？ | 批准 BH + WRC/SPA + PBO/CSCV + DSR；任何 claim-relevant mandatory FAIL/BLOCKED/unavailable 均不能被其他 PASS 覆盖。 | A. 缩减为 BH-only 后重跑 CP1/CP2；B. 暂停 CR164，保持全部统计证据 typed_unavailable。 | 推荐方案闭合 CR154 Gate-1 四类证据并保持严格 fail-closed；BH-only 较小但无法支持 robustness/Sharpe/data-snooping claims；暂停最保守但无当前价值。 | 决定 CP3 method/schema 边界、Story 数与 positive claim 条件。 | 方法集合变化回退 CP2；实现无法满足某方法时回 CP3/转 Spike，但不得静默降低 claim ceiling。 |
| `DQ-CP2-CR164-002` | scope | 是否批准 method-specific minima、10 项 QAC，以及 raw-count DSR/effective-count ceiling？ | 批准当前阈值与 QAC；`effective_trial_count` 保持 typed_unavailable，DSR schema 明确 raw-count provenance/non-alias。 | A. 采用更严格的 10-candidate/8-split/60-sample floor 后重跑 CP1/CP2；B. 延后 DSR 与 effective-count 相关 claims。 | 推荐阈值可验证且不制造 effective-count 过度声明；更严格方案减少假信心但增加 unavailable；延后 DSR 缩小价值。 | 决定数值边界、fixture 分母、CP7 量化出口和 deflated-performance claim ceiling。 | 阈值或 count 语义变化回退 CP2；CP3 只能细化 schema，不能放宽。 |
| `DQ-CP2-CR164-003` | scope | 是否批准 UC-58 实现、UC-59/60 compatibility-only、五个产品规划候选及三个 deferred 项？ | 批准；ML/event 无相同 sealed inputs 时 fail closed，不实现 real adapters；effective estimator 和 real recomputation 保持 deferred。 | A. 把 fixture/static ML/event adapters 纳入本轮并重跑 CP1/CP2；B. 删除 ML/event compatibility contract，仅保留 multifactor。 | 推荐方案防止 schema 分叉且控制实现范围；扩大 adapters 增加跨模块负担；删除 compatibility 会制造未来漂移。 | 决定 CP3 consumer boundary、正式 Story 拆解与 deferred backlog。 | 兼容或 deferred 范围变化回退 CP2；真实运行需求另起授权 CR。 |
| `DQ-CP2-CR164-004` | security | 是否批准 deny-default 权限与 architecture-major 路由：CP2 后只进入设计，CP5 前不实现，任何真实统计/data/runtime/external/write 操作仍需独立授权？ | 批准 CP3→CP4→CP5→CP6→fresh independent CP7→CP8 路由；当前仅允许设计。 | A. 暂停在 CP2；B. 另起 runtime-authorization CR 后再评估真实运行，但不混入当前 gate。 | 推荐方案允许本地设计推进且不放宽权限；暂停无风险但阻塞价值；混入真实运行会扩大审计与安全边界。 | 决定是否可启动 `meta-se-critical`；不授权代码实现或运行。 | 任何 credential/production data/NAS/provider/external/broker/trading/publish/write 需求出现时立即 BLOCKED 并单独授权。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 DQ-CP2-CR164-001..004 的推荐方案并进入 CP3。 |
| 备选方案 | `修改: <决策 ID + 修改点>` 后回到 requirement-clarification；或 `reject` 暂停/退回 CR164。 |
| 影响维度 | 用户价值、方法完整性、可验证性、数值风险、维护成本、兼容性、安全权限和交付。 |
| 优劣分析 | 推荐方案覆盖四方法 evidence 与既有 Gate-1 slots，同时把真实运行、effective estimator 和 real adapters 隔离到后续边界。 |
| 风险与回退 | scope/threshold/claim ceiling 变化回 CP2；方法实现与 schema 问题回 CP3/CP5；权限扩大立即 BLOCKED。 |
| 用户需决策事项 | `DQ-CP2-CR164-001`、`DQ-CP2-CR164-002`、`DQ-CP2-CR164-003`、`DQ-CP2-CR164-004`。 |

### CP2 追加摘要

- 用户真实意图：将 typed-unavailable 方法槽位推进为 sealed-lineage-bound、可计算、可复跑、可独立验证的统计证据，不制造过度声明。
- 场景覆盖：13 个 P0，覆盖正向、缺失/部分输入、method disagreement、minima、raw/effective non-alias、determinism、NaN/Inf/degenerate、hash/count/membership mismatch、permission、CR155 regression、compatibility 与 precheck。
- 认知盲区补充：现有代码包含报告 validator/threshold consumer 与 BH helper，但存在性不等于计算 provenance；CP3 必须区分 calculator、evidence contract 与 consumer。
- Deferred Ideas：effective-trial estimator、real ML/event adapters、real research/history recomputation。
- 用户选择影响：四方法 mandatory、10 项 QAC、UC-58 implementation、UC-59/60 compatibility-only；正式 Story 数仍由 CP3/CP4 决定。
- 回退方式：scope/AC 变化回 CP2；method/schema/decision-table 问题回 CP3；CP5 前不实现。
- discussion log：`process/discussions/CP2-CR164-SCENARIO-DISCUSSION-LOG.md`。
- discussion checkpoint：`process/checks/CP2-CR164-DISCUSSION-CHECKPOINT.json`。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP0/CP1/CP2 自动预检通过 | 通过 | CP0/CP1/CP2 result JSON | 自动预检 PASS。 |
| SGQ 4/4 confirmed | 通过 | Discussion checkpoint / SGQ batch | 用户确认全部修订版 A。 |
| 八份产品基线完成 | 通过 | `docs/product/*` CR164 revisions | 8/8 revision records。 |
| Capsule ready | 通过 | CP2 context | compact capsule-first。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 批准四方法 MVP 与 no-OR-pass claim ceiling | 通过 | DQ-CP2-CR164-001 | 接受推荐方案。 |
| 2 | 批准 minima、10 项 QAC 与 raw/effective count ceiling | 通过 | DQ-CP2-CR164-002 | 接受推荐方案。 |
| 3 | 批准 UC-58/59/60 范围、五候选与 deferred | 通过 | DQ-CP2-CR164-003 | 接受推荐方案。 |
| 4 | 批准 deny-default 权限与 architecture-major 路由 | 通过 | DQ-CP2-CR164-004 | 接受推荐方案；同时要求后续不拉起子 Agent。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 产品基线可作为 CP3 输入 | 通过 | 本 checkpoint | 允许进入 CP3 设计。 |
| 用户明确 approve / 修改 / reject | 通过 | 人工审查结果 | 用户要求继续推进，按当前唯一 pending gate 记为批准。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 产品基线 | `docs/product/*` CR164 sections | 通过 | 8/8 complete。 |
| CP2 result | `process/checks/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-SCOPE.result.json` | 通过 | PASS。 |
| CP2 capsule | `process/context/CP2-CR164-MULTIPLE-TESTING-PBO-DSR-CONTEXT.yaml` | 通过 | ready。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-12T19:59:27+08:00
- 修改意见：接受 `DQ-CP2-CR164-001..004` 推荐方案并继续推进；禁止拉起任何子 Agent，后续阶段使用经用户批准的 `inline-fallback`。
- 风险接受项：无；本 CP2 不接受或授权 runtime/data/credential/external-write 风险。
