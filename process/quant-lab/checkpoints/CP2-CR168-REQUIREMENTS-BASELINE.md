---
checkpoint: CP2
cr_id: CR-168
title: "Economic Cost / Slippage / Impact Computable Evidence Producer Foundation — 需求、场景与范围基线"
status: approved
gate_profile: architecture-major
automatic_precheck: PASS
context_ref: process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml
result_ref: process/checks/CP2-CR168-REQUIREMENTS-BASELINE.result.json
decision_brief_profile: compact
opened_at: "2026-07-13T17:45:00+08:00"
opened_by: host-orchestrator
revision: 3
reopened_at: "2026-07-13T22:45:00+08:00"
previous_user_decision: changes_requested
approved_at: "2026-07-14T09:45:10+08:00"
approved_by: user
---

# CP2 — CR168 需求 / 场景 / 范围基线人工门禁

自动预检结论：`PASS`；人工审查结论：`approved`。本次批准只解锁 CP3 solution-design。

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 在 Stage 2 已完成的合同基础上，为未来 Stage 3 策略生产建立 fixture/static-only 的 C3 Economic Cost / Slippage / Impact approximation typed evidence producer foundation；复用 CR166 neutral envelope，不连接真实数据，不启动 Stage 3。 |
| 推荐动作 | `approve`：接受下方 5 项修订后推荐方案，然后只进入 CP3 solution-design；不直接拆 Story、写 LLD、实现或验证。用户此前的 `修改:` 已应用，但不等于批准。 |
| approve 后会发生什么 | Host Orchestrator 回填 CP2 审批和 gate ledger，生成 capsule-first 的 CP3 handoff；meta-se/inline 等价职责在 CP3 冻结 schema、方法、C3/C4 header 与 Gate 4 projection architecture。 |
| approve 不授权什么 | 不授权真实数据/凭据、真实 TCA/impact calibration、C4 calculator、event producer、FU-007 aggregate integration、Stage 3、runtime/trading、publish/deploy 或 Git remote write。 |
| 不确认会阻塞什么 | CR168 保持 active/not_ready，HLD/CP3、Story、LLD、实现和验证全部继续阻塞。 |

### 用户真实意图与评审修正

- 用户要求正式启动 `FU-CR161-004`，实际未占用编号为 `CR-168`，并连续推进到 CP2。
- 用户给出的代码评审确认：Gate 4 是 C3+C4 联合门禁；C3 只投影四个 cost/impact 字段，C4 三个 capacity/liquidity 字段保持 `typed_unavailable` 并 fail-closed。
- 用户随后以 `修改:` 收紧 Gate 4 推荐方案：C4 `reserved/not-built/typed_unavailable` 必须投影为三个 refs absent 且不带字段级/通用 na-reason；任何 reason 逃逸由 projection `BLOCKED/REJECTED`，不得修改 canonical Gate 4 validator 或 aggregate orchestration。
- 代码事实复核确认 canonical Gate 4 对 absent+na-reason 存在宽松路径；本轮整改限定为 CR168 adapter 的精确 8-key denylist、strict allowlist、调用前拒绝、逃逸路径 canonical 调用数为 0，以及合法 absent 路径调用后的非 PASS 断言。该整改不代表 canonical Gate 4 已全局修复。
- `cost_underestimation_status` 已进入 C3 输入/输出合同；fixture 已精确为 daily multifactor synthetic 与 daily/ML multi-strategy-type compatibility；跨字段 currency/price-basis/calendar 仅在无显式转换声明时构成失败。
- 用户明确要求不拉起子 Agent；产品基线由 Host Orchestrator 按批准的 `meta-pm inline-fallback` 完成，handoff/dispatch/ledger 已记录。

### 产品范围摘要

In Scope：

1. 把 CR166 的 `economic_cost@reserved` 演进为 `1` 个 active C3 typed component / `1` 个 active schema version。
2. 冻结 9/9 输入字段族的 required/optional/N/A/authorization 产品规则。
3. 用显式 synthetic/static 参数形成 fee、tax/levy、spread/slippage、impact approximation、total cost、gross-to-net reconciliation、availability/reason/lineage/hash。
4. 对 10/10 P0 fail-closed 类别提供精确 fixture 合同，10 次规范化输入得到 1 个 canonical hash。
5. 提供 `1` 条 C3-to-Gate-4 compatibility projection；C4 unavailable 映射为三个 refs absent-no-na-reason；字段级/通用 na-reason 逃逸由 projection 阻断；canonical Gate 4/aggregate orchestration 修改=`0`，capacity/aggregate PASS=`0`。
6. 提供 2/2 fixture 族：daily multifactor synthetic；daily multifactor + ML multi-strategy-type compatibility。
7. 维持 Stage、权限、CR155 admission 和 follow-up 边界。

Out of Scope：

- 真实 lake/NAS/provider/data vendor/credential/secret 和真实 order/fill/quote/book/flow/ADV/liquidity 数据。
- 真实 TCA、真实 market-impact calibration、真实 capacity sizing、历史灌入/回填/重算/参数估计。
- C4 capacity/liquidity/ADV/alpha-decay calculator（`FU-CR161-005`）。
- C1-C4 aggregate orchestration、全链路 existing-gate integration、final StrategyAdmissionPackage 与 CR155 综合 promotion decision（`FU-CR161-007`）。
- event-specific economic-cost producer。
- runtime、QMT/broker/simulation/paper/live/trading/order、catalog/store/registry/lake pointer write、publish/deploy/tag/release/Git remote write。
- Stage 3 启动、runtime/production/real-evidence readiness 声明或 CR155 admission promotion。

### 场景覆盖与认知盲区

| 项目 | 当前值 | 说明 |
|---|---:|---|
| 新增 Use Case | 1 | `UC-58-CR168` |
| CR168 Requirements | 9 | `REQ-CR168-001..009` |
| 量化 QAC | 15 | 均为精确值，无模糊下限 |
| CR168 Scenarios | 17 | P0=`16`，P1=`1`；其中 10 个 C3 输入 fail-closed 类别不变，另有 1 个 projection guard 负向场景 |
| Matrix coverage | 17/17 | missing scenario=`0`，missing requirement ref=`0` |
| 用户可见 SGQ | 1 confirmed | `SGQ-CR168-000` |
| 待 CP2 决策 | 5 | 方法、共享合同、Gate 4、fixture、claim ceiling |

认知盲区已转化为门控：静态 impact approximation 可能被误读为真实 TCA；共享 header 可能预占 C4 语义；C4 unavailable 若被翻译成字段级或通用 na-reason，canonical Gate 4 可能错误 PASS；strategy-type compatibility 可能被误读为真实 ML/event runtime；CR155 lifecycle closed 可能被误读为 admission 已通过。上述风险均有明确 claim ceiling、projection-side guard、零计数或 fail-closed 控制。

### Scenario Gray Areas 处理结果

- 已解决：Gate 4 事实定位、`cost_underestimation_status`、multi-strategy-type fixture、跨字段 basis/currency/calendar 触发条件。
- 待用户决定：`SGQ-CR168-001..005`，对应下方 5 个 DQ；`SGQ-CR168-003` 推荐方案已按本轮 `修改:` 收紧，仍需重新批准。
- 讨论日志：`process/discussions/CP2-CR168-SCENARIO-DISCUSSION-LOG.md`。
- 恢复点：`process/checks/CP2-CR168-DISCUSSION-CHECKPOINT.json`。

### Deferred Ideas

| 项目 | 状态 | 路由 |
|---|---|---|
| C4 capacity/liquidity/ADV/alpha-decay producer | candidate | `FU-CR161-005` |
| C1-C4 aggregate integration / CR155 综合 regression | candidate | `FU-CR161-007` |
| 真实 TCA / market-impact calibration / real-data parameter estimation | not-authorized | 独立数据、方法、权限、审计与 runtime CR |
| event-specific economic-cost semantics | deferred | 独立 event semantics CR |

### Context Capsule Summary

- capsule：`process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml`
- read_profile：`compact`
- 默认读取策略：先读 capsule、CR168 summary、CP1 result、产品证据和本 checklist；产品全文只在具体决策审计时按需展开。
- 全文档读取：本轮因用户精确修改 Gate 4 契约触发 deep-review，已对 CR168 formal CR、8 个产品文档与 current baseline 记录 10 条 read-expansion；`process/archive/**`、全部 CR166 Story/LLD/测试报告和完整会话仍未进入默认上下文。

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---|
| 用户启动提示词、代码评审与 CP2 修改意见 | complete | 16 | 5 | 4 项早期事实修正已 resolved；1 项 Gate 4 推荐方案已 changes-requested 并回写；F2/F3/F4/F6/F7 进入 CP3 义务，F5 进入风险；5 项 DQ 统一重审 |
| CR168 formal CR / summary / conflict precheck | complete | 5 | 0 | 已归入同 5 个 DQ，无重复新增 |
| meta-pm inline handoff/return | complete | 5 | 0 | 与产品文档一致；无独立新增问题 |
| USE-CASES / REQUIREMENTS | complete | 5 | 5 | OPEN questions 全部进入 DQ |
| SCENARIOS / TEST-MATRIX | complete | 1 | 0 | 新增 `SC-CR168-B02` 后 17/17 覆盖，无未映射缺口；这是用户指定 guard，不新增 DQ |
| STORY-MAP / MVP-SCOPE / RELEASE-SLICES / BACKLOG | complete | 4 | 0 | 均为 Deferred/Out-of-Scope 或既有 DQ 的派生，不重复提问 |
| discussion log / checkpoint | complete | 5 | 5 | `SGQ-CR168-001..005` 全部映射 |
| CURRENT-REQUIREMENT-BASELINE | complete | 0 | 0 | 只记录当前边界，无新增取舍 |
| CR166 envelope / Gate 4 当前代码事实 | complete | 2 | 0 | Gate 4 联合事实与 na-reason 逃逸路径已由用户评审解决为 projection-side guard，不再作为独立选项 |
| 权限、安全、真实数据、remote-write 边界 | complete | 1 | 1 | 汇入 claim ceiling/security DQ；禁止操作不作为可批准扩大选项 |
| **汇总** | **10/10 complete** | **44（含跨来源重复）** | **5（去重）** | resolved/user-directed=`11`；decision-item=`5`；non-blocking-open=`1`（verifier independence）；N/A/deferred 见上表 |

### 决策分层

| 层级 | 数量 | 说明 |
|---|---:|---|
| 必须用户决策 | 5 | 下方 5 个 DQ；未决时不得进入 CP3 |
| 高风险策略确认 | 1 | claim ceiling/security DQ；真实数据/runtime 扩大不在可选项内 |
| agent 默认处理 | 0 | 公共方法和接口细节必须留给 CP3，不在 CP2 静默冻结 |
| 仅审计记录 | 10 | 用户已确认的四项早期修正 + F2/F3/F4/F6/F7 CP3 路由 + F5 非阻断风险；均不替代 CP2 approve |

本轮待人工决策项：5。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR168-METHOD | scope | C3 是否包含 transparent impact approximation？ | 纳入 fee/tax/spread/slippage/impact 分解，但 impact 只使用显式 synthetic/static 参数，并输出 `cost_underestimation_status`、limitations、`no_real_tca_claim=true`。 | 只计算 fee/tax/spread/slippage，把 impact approximation 延后到独立 CR。 | 推荐方案给 Gate 4 提供完整 C3 cost/impact 语义，审计价值高；代价是 CP3 要冻结方法族、参数与风险状态。备选更简单，但 C3 对 impact 的解释能力和 Gate 4 兼容价值下降。 | 影响 schema、10 类负向场景、fixture 和 Gate 4 C3 fields；任何真实数据校准仍被禁止。 | 若 CP3 无法形成透明、确定、静态参数方法和 fail-closed decision table，则回退备选并重开 CP2。 |
| DQ-CP2-CR168-C3-C4 | architecture | 是否在本 CR 冻结 C3/C4 最小共享输入 header？ | 冻结 identity、basis、unit/currency/calendar、lineage/auth 的最小共享 header；C4-exclusive capacity/ADV/liquidity/alpha-decay 字段保持 reserved，C4 calculator=0。 | C3 完全独立设计；C4 后续再建立共享合同并接受迁移/兼容成本。 | 推荐方案减少未来 C4 重复和 envelope 迁移，同时通过 owner 表防止预占；备选局部更简单，但未来字段重命名、迁移和兼容测试成本更高。 | 影响公共 schema、字段 owner、版本演进和 CP3 ADR；若边界失控可能把 C4 偷渡进 CR168。 | 若 CP3 无法给出明确 shared/exclusive owner table 或会扩大 C4 计算，则采用独立 C3 备选并重开 CP2。 |
| DQ-CP2-CR168-GATE4 | implementation | existing-gate integration 在 CR168 做到何种粒度？ | 实现 `1` 条 C3-to-Gate-4 compatibility projection；只填 C3 四字段；C4 `reserved/not-built/typed_unavailable` 必须映射为三个 C4 refs absent，且不得输出字段级 `*_na_reason` / `*_n_a_reason` 或通用 `na_reason` / `n_a_reason`。任何 reason 逃逸输入由 projection `BLOCKED/REJECTED`；不得修改 canonical Gate 4 validator 或 aggregate orchestration；capacity/aggregate PASS=0。新增 P0 `SC-CR168-B02`，原 B01 保留。 | CR168 只输出 C3 component，把所有 Gate 4 projection 延后 `FU-CR161-007`。 | 推荐方案能验证 C3 contract 与现有 consumer 的真实兼容性，并在 projection 侧封死 canonical Gate 4 的 N/A PASS 逃逸；代价是 CP3 必须冻结完整 availability→flat-payload 映射和 reason-key denylist。备选隔离更强，但无法在本 CR 证明现有 Gate 4 消费契约。 | 影响 cross-strategy reliability Gate 4 的适配测试；错误翻译会虚假产生 Gate 4/capacity/aggregate PASS。canonical Gate 4 与 aggregate orchestration 修改数必须为 0。 | 若 CP3 无法以 projection-side guard 证明确定性 fail-closed，或必须修改 canonical Gate 4/aggregate orchestration，则切换备选并重开 CP2。 |
| DQ-CP2-CR168-FIXTURE | scope | fixture 适用面是否采用两族？ | 精确采用 `2/2`：daily multifactor synthetic；daily multifactor + ML multi-strategy-type compatibility。event-specific 明确 N/A/deferred。 | 只保留 daily multifactor synthetic 一族，把 ML compatibility 延后。 | 推荐方案验证 C3 算术不依赖 strategy type，并复用既有 ML package contract；代价是多一组 attach/compatibility 验证。备选范围更小，但不能证明 neutral envelope 的跨策略一致性。 | 影响 TEST-MATRIX、兼容性 claim 和后续 Story；不授权真实 model training 或 event feed。 | 若 CP3 发现 ML package 无稳定 attach point 且需 runtime/model 语义，采用 daily-only 备选并重开 CP2。 |
| DQ-CP2-CR168-CLAIM | security | 是否批准当前 claim ceiling 与不授权边界？ | 保持 Stage2 complete=true、Stage3 started=false；在实际完成前 C3 foundation=false；真实 TCA/calibration/data/runtime=false；C4=0、event=false、CR155 promotion=false。 | 暂缓 CR168 并维持 `economic_cost@reserved`，直到用户愿意批准该保守 ceiling；不提供扩大真实数据/runtime 的 CP2 选项。 | 推荐方案允许安全进入 CP3，且防止 fixture 被误读为真实能力；备选零风险但不产生 C3 foundation。 | 这是高风险安全/声明边界。任何真实数据、TCA、runtime 或 CR155 提升都需要新授权和新 CR。 | 若后续要求扩大真实数据/runtime/交易/发布权限，立即停止当前 route，另起高风险 CR；不得在 CR168 内隐式切换。 |

### 用户需决策事项

回复 `approve` 表示同时接受以下推荐方案：`DQ-CP2-CR168-METHOD`、`DQ-CP2-CR168-C3-C4`、`DQ-CP2-CR168-GATE4`、`DQ-CP2-CR168-FIXTURE`、`DQ-CP2-CR168-CLAIM`。

如只修改部分，请回复 `修改: <具体修改点>` 并明确一个或多个决策 ID；例如：`修改: DQ-CP2-CR168-GATE4 采用备选方案，其余按推荐方案`。

### 量化成功标准

| 指标 | 目标 |
|---|---:|
| typed C3 component 类型 / active schema version | 1 / 1 |
| 输入字段族规则覆盖 | 9/9 |
| fixture 族 | 2/2 |
| P0 fail-closed 类别 | 10/10 |
| deterministic reruns | 10 次 -> 1 canonical hash |
| C3-to-Gate-4 projection | 1 |
| Gate 4 safe absent path / reason-escape rejection 场景 | 1 / 1（合计 2/2） |
| 字段级/通用 na-reason 逃逸产生的 Gate 4/capacity/aggregate PASS | 0 |
| canonical Gate 4 validator / aggregate orchestration 修改 | 0 / 0 |
| C4 缺失产生的 capacity/aggregate PASS | 0 |
| 平行 gate/envelope/registry | 0 |
| C4 calculator / event-specific producer | 0 / 0 |
| 真实数据/TCA/runtime/broker/trading 操作 | 各 0 |
| CR155 admission promotion | 0 |
| CR168 新增代码路径引入失败 | 0 |
| 带 process 前缀的错误质量文档引用 | 0 |

### Claim Ceiling

| Claim | CP2 当前值 |
|---|---|
| `stage2_complete` | `true` |
| `stage3_started` | `false` |
| `c3_fixture_static_foundation` | `false`，仅实际完成后才能改为 true |
| `real_tca_available` | `false` |
| `real_market_impact_calibrated` | `false` |
| `real_data_connected` | `false` |
| `runtime_ready` | `false` |
| `c4_calculators` | `0` |
| `event_specific_producer` | `false` |
| `cr155_promoted` | `false` |

### 风险和未决问题

| 风险 ID | 风险 | 当前控制 | 状态 |
|---|---|---|---|
| R-CR168-GATE4-C3-C4-SEMANTIC | 把 joint Gate 4 误写为 C3-only，或用字段级/通用 na-reason 让 C4 unavailable 走到虚假 PASS | DQ-GATE4 + absent-no-na-reason 映射 + projection reason-key denylist + B01/B02 + PASS=0 | OPEN，待 CP2/CP3 |
| R-CR168-COST-UNDERSTATEMENT | 静态模型低估成本却被误读为充分 | `cost_underestimation_status` + limitations + fail-closed | OPEN，待 CP2/CP3 |
| R-CR168-UNIT-CURRENCY-BASIS | 跨字段单位/币种/日历混用 | 9-family contract + explicit conversion declaration | OPEN，待 CP2/CP3 |
| R-CR168-TRUE-TCA-OVERCLAIM | approximation 被宣传为真实 TCA/calibration | `no_real_tca_claim=true` + claim ceiling | OPEN，待 CP2/CP8 |
| R-CR168-CR155-PROMOTION | C3 present 被错误解释为 CR155 admission 可提升 | C4 unavailable、CR155 BLOCKED/paper=false、promotion=0 | OPEN，贯穿 CP7/CP8 |
| R-CR168-LEGACY-METADATA | 全局历史 CR tracking/ledger strict 检查存在旧债务 | CR168-specific tracking/conflict=PASS；不扩张本 CR 修复范围 | ACCEPTED-NONBLOCKING，来源为创建前基线 |
| R-CR168-INLINE-INDEPENDENCE | meta-pm 未由独立子 Agent 执行 | 用户显式批准 inline fallback；handoff/dispatch/ledger 透明记录 | ACCEPTED-BY-USER |
| R-CR168-VERIFIER-INDEPENDENCE | 后续 CP7 若继续 inline，可能缺少独立 verifier 视角 | 引用 `FU-CR161-006`；CP7 如实标注，CP8 Decision Brief 强制暴露并由用户决定风险接受 | OPEN-NONBLOCKING；不改变当前不拉子 Agent 授权 |

### 不授权项

- 真实数据、凭据、provider/NAS/lake、真实订单/成交/盘口/ADV/流动性读取。
- 真实 TCA、真实 impact calibration、真实 capacity sizing 或历史参数回填。
- C4 calculator、event producer、FU-007 aggregate integration。
- canonical Gate 4 validator 或 aggregate orchestration 修改；若 projection-side guard 不足，必须回退备选并重开 CP2。
- runtime、QMT/broker、simulation/paper/live、交易与订单操作。
- catalog/store/registry/lake pointer write、publish/deploy/tag/release、Git push/remote merge/remote write。
- Stage 3 启动、runtime/production/real-evidence-ready 声明。
- CR155 admission promotion、unblock 或 regression-pass reinterpretation。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP0 通过 | PASS | `process/checks/CP0-CR168-REQUEST-INTAKE.result.json` | 待人工审查 |
| CP1 通过 | PASS | `process/checks/CP1-CR168-USE-CASE-COMPLETENESS.result.json` | 待人工审查 |
| 产品基线完整 | PASS | 8 个产品文档 + current requirement baseline | 待人工审查 |
| discussion / SGQ 可追溯 | PASS | discussion log + checkpoint | 待人工审查 |
| CP2 capsule 可读 | PASS | `process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml` | 待人工审查 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | 功能需求完整 | APPROVED | REQ-CR168-001..009 | 用户接受修订版推荐范围。 |
| 2 | 非功能需求量化 | APPROVED | NFR-CR168-001..004 | 保持精确量化。 |
| 3 | In/Out/Deferred 范围清晰 | APPROVED | MVP-SCOPE / BACKLOG | canonical 全局硬化继续 deferred。 |
| 4 | 15 项 QAC 精确 | APPROVED | REQUIREMENTS | 数量不变。 |
| 5 | 技术/兼容/权限约束完整 | APPROVED | current baseline / claim ceiling | 只解锁 CP3。 |
| 6 | 风险与依赖已识别 | APPROVED | 风险表 / DQ | Gate4 风险采用 adapter-local containment。 |
| 7 | 需求冲突已解决或进入决策 | APPROVED | conflict precheck / DQ | 五项 DQ 已收敛。 |
| 8 | 变更机制明确 | APPROVED | CR168 / CR-INDEX / ledger | 后续范围变化需重开相应门禁。 |
| 9 | REQUEST→UC→REQ→Scenario→Matrix 可追溯 | APPROVED | 产品证据 | 17/17 保持。 |
| 10 | Scenario Gray Areas 已处理 | APPROVED | discussion checkpoint | remaining gray areas=0。 |
| 11 | Deferred Ideas 已隔离 | APPROVED | BACKLOG / follow-up tracking | FU005/FU007 未启动。 |
| 12 | 用户可见 SGQ 已确认 | APPROVED | SGQ-CR168-000..005 | 五项 CP2 DQ 已批准。 |
| 13 | 八维扫描后台化 | APPROVED | USE-CASES / discussion | CP3 义务已显式。 |
| 14 | 17 个工程场景可追踪 | APPROVED | SCENARIOS / TEST-MATRIX | 10/10 C3 输入 fail-closed + B01/B02 Gate 4 guard。 |
| 15 | MVP 与产品 outcome 可确认且未提前拆 Story | APPROVED | STORY-MAP / MVP / RELEASE-SLICES | 正式 Story 仍为 0。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| P0/P1 产品基线无阻塞缺口 | PASS | CP2 result |  |
| 五项人工决策收敛 | APPROVED | Decision Brief | 五项推荐方案全部接受。 |
| 人工确认完成 | APPROVED | 本文件人工审查结果 | 2026-07-14。 |
| 只解锁 CP3，不解锁实现 | APPROVED | route plan / claim ceiling | Story/LLD/实现/验证仍禁止。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 自动结果 | `process/checks/CP2-CR168-REQUIREMENTS-BASELINE.result.json` | PASS |  |
| CP2 自动摘要 | `process/checks/CP2-CR168-REQUIREMENTS-BASELINE.summary.md` | PASS |  |
| CP2 Context Capsule | `process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml` | PASS |  |
| 产品基线证据 | `process/checks/CR168-PRODUCT-BASELINE-EVIDENCE.json` | PASS |  |
| Discussion checkpoint | `process/checks/CP2-CR168-DISCUSSION-CHECKPOINT.json` | PASS |  |
| 人工门禁审查稿 | `process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md` | APPROVED |  |

## 人工审查结果

### 修订处理记录

| 时间 | 用户结论 | 处理 | 当前结果 |
|---|---|---|---|
| 2026-07-13 | `修改: DQ-CP2-CR168-GATE4 ...` | 已更新需求、场景、矩阵、范围、CR、current baseline、CP3 义务与风险；新增 `SC-CR168-B02`，其余四项 DQ 推荐不变。 | `changes_requested` 已处理；本轮人工结果重置为 `pending`，等待重新 `approve`。 |
| 2026-07-14 | `按照建议完成整改，整改后批准，并推进到下一个人工门禁。` | 复核 canonical Gate 4 代码事实；把整改限定为 CR168 adapter-local containment；更新产品基线、CP2 证据与后续 CP3 设计义务。 | `approved`；五项 DQ 均按推荐方案接受，仅解锁 CP3。 |

- 结论：`approved`
- 审查人：用户
- 审查时间：`2026-07-14T09:45:10+08:00`
- 修改意见：按评审建议完成 projection-side 整改，并推进到下一个人工门禁。
- 风险接受项：接受 CR168 adapter-local containment 和 canonical 全局硬化 deferred；不接受扩大真实数据/runtime/C4/aggregate/Stage3 权限。

CP2 已关闭为 approved；后续人工决策在 CP3 审查稿中进行。本次批准不表示授权任何“不授权项”。
