---
story_id: "CR153-S05-event-trace-evidence-wording"
change_id: "CR-153"
title: "Event-to-signal/order-intent trace and static evidence wording"
status: "lld-ready-for-review"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
source_hld: "process/docs/design/HLD-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-EVENT-DRIVEN-STRATEGY-E2E-FRAMEWORK.md"
lld_policy:
  required: true
  required_level: "technical-note"
  status: "pending"
depends_on:
  - "CR153-S01-event-research-time-pit-contracts"
  - "CR153-S02-event-study-method-test-slots"
  - "CR153-S03-event-bias-risk-audit-slots"
  - "CR153-S04-event-admission-gate-adapter"
implementation_allowed: false
authorization_boundary: "static/fixture evidence only; no real feed, order flow, broker operation, runtime readiness, event store/catalog/registry write or production readiness claims"
---

# CR153-S05 Event-to-Signal/Order-Intent Trace and Static Evidence Wording

## Goal

Close event trace metadata and downstream evidence wording while preserving no-runtime and no-real-order boundaries.

## Scope

- Event -> signal -> target/order-intent trace metadata refs.
- Process return/evidence wording.
- CP7/CP8 release limitations and CR154 dependency.
- No-real-feed / no-runtime / no-order-flow assertions.

## Acceptance Criteria

- Trace metadata is refs only and cannot create, submit, cancel, query, persist or mutate broker/runtime/order state.
- Evidence states fixture-only contract semantics.
- Release wording does not claim real event alpha, feed quality, event store/catalog/model registry publication, runtime readiness or trading readiness.
- Deferred CR154 risks remain explicit.
- Forbidden operation counters and limitations are visible in CP7/CP8 artifacts.
- CP5 technical note must enumerate exact CR153 return/evidence/release wording artifacts or mark each artifact family N/A; broad wildcard ownership is not sufficient for implementation.

## Technical Note Required For CP5

Story technical note covering trace refs, evidence refs, release limitations, CR154 deferred risks, forbidden operation assertions and wording tests.

The technical note must include an artifact target table with these rows at minimum:

| Artifact family | Required CP5 treatment |
|---|---|
| Return packet | Exact `process/returns/CR153-*.return.json` path(s), or N/A with reason. |
| Evidence index | Exact `process/evidence/CR153-*.index.json` path(s), or N/A with reason. |
| CP7 wording | Exact `process/checks/CP7-CR153-*.result.json` / summary path(s), or N/A with reason. |
| CP8 / release context | Exact `process/checkpoints/CP8-CR153-*.md` and `process/release/RELEASE-CONTEXT-CR153.yaml` path(s), or N/A with reason. |
| Release notes | Exact `docs/release/RELEASE-NOTES.md` CR153 section treatment only if CP8 explicitly requires a release notes update; otherwise N/A with reason. |

## 技术说明

### 设计依据

CR153-S05 只收敛 event -> signal -> target/order-intent 的 trace refs 与后续证据措辞，不新增事件研究算法、真实事件 feed、listener、runtime、broker 或 order flow。设计依据来自 CR153 CP5 context、HLD v0.2、ADR v0.2、Story Backlog 与 Development Plan：

- HLD §3、§9、§13 明确 CR153 是 local/static/fixture-only 的 metadata contract，禁止真实 feed/listener、event store/catalog/registry write、runtime/trading 与真实数据验证。
- HLD §8、§10、§11 将 trace contract 定义为 metadata refs：输入是 event gate summary、signal refs、target/order-intent metadata refs；输出只进入 admission package trace evidence；任何真实 broker/order operation counter 非零都必须 `BLOCKED`。
- ADR-CR153-002 要求 event metadata 不等于 event store/catalog/registry write；ADR-CR153-004 要求 deterministic fixture-only validation；ADR-CR153-005 要求 CP7/CP8 持续暴露 CR154 deferred risks。
- Story Backlog 与 Development Plan 要求 S05 的 CP5 technical-note 枚举精确 CR153 return/evidence/release wording artifacts 或逐项给出 N/A，不允许宽泛 `process/returns/*`、`process/evidence/*` 或 `docs/release/*` owner。

本 Story 的实现语义是“证据措辞和路径约束”，不是业务运行能力。`trace_ref`、`signal_ref`、`target_ref`、`order_intent_ref`、`event_gate_ref`、`limitations` 与 `forbidden_operation_counters` 均为静态证据引用字段或检查结果字段；它们不得被解释为可创建、提交、取消、查询、持久化或变更任何 broker/runtime/order state。

### 文件影响

S05 的设计与后续实现只允许在 CR153 证据、验证和发布上下文中写入措辞，不拥有源码、测试源码或宽泛 release 文档目录。

| Artifact family | 精确 CR153 target | CP5 处理 |
|---|---|---|
| Return packet | `process/returns/CR153-S05-event-trace-evidence-wording.return.json` | 需要。CP6 完成时记录 S05 的 Story return，必须包含 trace refs only、fixture-only、no real feed / no runtime / no trading readiness、CR154 deferred risks 和 forbidden operation counters 的摘要引用。 |
| Evidence index | `process/evidence/CR153-S05-event-trace-evidence-wording.index.json` | 需要。CP6 完成时索引 S05 证据，至少指向 Story 卡片技术说明、实现摘要或 CP6 结果、trace/evidence wording 检查入口；不得复制完整日志或泛化到 `process/evidence/*`。 |
| CP7 wording | `process/checks/CP7-CR153-S05-event-trace-evidence-wording.result.json`；`process/checks/CP7-CR153-S05-event-trace-evidence-wording.result.summary.md` | 需要。CP7 验证措辞必须声明 local/static/fixture-only，证明的是 contract semantics；不得声明真实 event alpha、真实 feed 覆盖、event store/catalog/model registry publication、runtime readiness、paper/live readiness 或 trading readiness。 |
| CP8 / release context | `process/checkpoints/CP8-CR153-EVENT-DRIVEN-STRATEGY-E2E-RELEASE-READINESS.md`；`process/release/RELEASE-CONTEXT-CR153.yaml` | 需要。CP8 人工门禁与 release context 必须携带 CR153 no-runtime/no-store 限制、CR154 deferred risks、fixture-only 证据范围和 forbidden operation counters 结论。 |
| Release notes | N/A，除非 CP8 明确要求更新 `docs/release/RELEASE-NOTES.md` 的 CR153 section | 默认不写。只有 CP8 release readiness 明确要求对用户可见 release notes 增加 CR153 section 时，S05 才允许更新 `docs/release/RELEASE-NOTES.md` 中的 CR153 小节；不得声明或占用宽泛 `docs/release/*` owner。 |

不影响文件：

- 不修改 `engine/**`、`tests/**`、`data/**`、`.env`、credential、catalog、registry、store 或 runtime 配置。
- 不创建 event feed/listener/lake/NAS/provider/broker/order/runtime 相关入口。
- 不把 release wording 写成真实交付或生产 readiness 证明。

### 接口 / 数据 / 权限变化

接口变化：无新增运行时 API、CLI、provider API、broker API、listener API、catalog API 或 registry API。S05 仅规定证据对象中可以出现的静态引用和措辞字段。

数据变化：无新增持久化数据模型、数据库表、lake/NAS 文件、event store、catalog pointer、model registry entry、feature/label/prediction store 记录或 production current truth。允许的字段只作为证据引用或摘要措辞出现：

- `event_trace_ref`
- `event_gate_ref`
- `signal_ref`
- `target_ref`
- `order_intent_ref`
- `fixture_contract_ref`
- `forbidden_operation_counters`
- `limitations`
- `cr154_deferred_risk_refs`

权限变化：无新增权限。CR153-S05 明确不授权以下行为：

- 真实 event feed、provider fetch、live listener。
- 真实 lake、NAS、catalog current truth、event store、feature store、label store、prediction store、model registry、registry/catalog publication。
- QMT、MiniQMT、xtquant、gateway、simulation、paper、live、broker、account query、market query、submit/cancel/order flow。
- credential、token、`.env`、session 或账号信息读取。
- runtime readiness、production readiness、trading readiness 或真实 alpha 声明。

### 异常和回退

| 条件 | 处理 |
|---|---|
| Evidence wording 暗示真实 feed、真实 alpha、runtime/paper/live/trading readiness | CP7/CP8 视为失败；回退到 fixture-only contract semantics wording。 |
| Trace refs 被实现成 submit/cancel/query/persist/mutate order 或 broker/runtime state | 立即阻断，状态应为 `BLOCKED`；该行为超出 CR153-S05，需要新 CR 和授权门禁。 |
| Forbidden operation counters 缺失或无法证明为零 | 证据不得通过；措辞必须改为 `BLOCKED` 或 `NEEDS_REVIEW`，不得保留 PASS。 |
| CP8 要求 release notes 但未限定 CR153 section | 不得宽泛修改 `docs/release/*`；先收敛到 `docs/release/RELEASE-NOTES.md` 的 CR153 section 或标记为 N/A。 |
| CR154 deferred risks 被删除或写成已解决 | CP7/CP8 失败；恢复为显式 residual risk / deferred dependency。 |
| 上游 S01-S04 设计字段名或 gate 状态发生变化 | S05 只同步证据引用和 wording，不重定义上游契约；必要时回到 CP5 统一确认。 |

回退策略：删除或改写超范围 claim，保留 CR153-S05 的静态 evidence refs；release notes 默认 N/A，只有 CP8 明确要求时再新增 CR153 小节；所有真实运行、真实数据、真实交易相关能力回退为“不授权 / deferred / requires future CR”。

### 测试入口

S05 不新增源码测试。CP6/CP7 可通过以下静态和 fixture-only 入口验证措辞与边界：

- `uv run --python 3.11 pytest -q tests/research/test_event_driven_strategy_e2e_contracts.py`：验证上游 S01-S04 的静态 event contract、trace refs 和 forbidden operation counters 语义。S05 只消费结果 wording，不扩大测试范围。
- `uv run --python 3.11 pytest -q tests/research/test_strategy_admission_statistical_gate.py tests/research/test_strategy_admission_package.py`：验证 admission package 的 no-runtime/no-trading claim blocker 仍可被下游证据引用。
- CP7 静态检查入口：检查 `process/checks/CP7-CR153-S05-event-trace-evidence-wording.result.json` 与 `.result.summary.md` 是否包含 fixture-only、no real feed、no real alpha、no event store/catalog/model registry publication、no runtime/trading readiness 和 CR154 deferred risk wording。
- CP8 静态检查入口：检查 `process/checkpoints/CP8-CR153-EVENT-DRIVEN-STRATEGY-E2E-RELEASE-READINESS.md` 与 `process/release/RELEASE-CONTEXT-CR153.yaml` 是否保留 CR153 限制和 release notes N/A / CR153 section 决策。

所有验证均为 local/static/fixture-only；不得为了验证 S05 读取真实 feed、lake、NAS、provider、credential、store、catalog、registry、runtime、broker 或 order flow。

### 已知风险

- 上游 S01-S04 的具体字段名和 CP6 证据结构在 CP5 统一确认前仍可能调整；S05 后续实现只能引用已确认字段，不得自行重定义 event gate 或 trace contract。
- CP7/CP8 wording 容易被误读为“event-driven strategy production ready”。必须持续使用 fixture-only、contract semantics、no runtime readiness、no trading readiness 和 CR154 deferred risks 的固定措辞。
- Release notes 默认 N/A 可能与 CP8 人工发布要求发生冲突；若 CP8 明确要求用户可见说明，只允许更新 `docs/release/RELEASE-NOTES.md` 的 CR153 section，并保持 no-real-feed/no-runtime/no-trading 限制。
- CR154 deferred 风险包括 full CV、survivorship-free universe、capacity/impact、regime、reconciliation 和更完整 runtime/compliance state machine；CR153 不得把这些风险写成已关闭。

### 偏离记录

- 无 lld_policy 偏离：本 Story 按 `required_level: technical-note` 在 Story 卡片中提供技术说明，不生成独立 LLD。
- 无 artifact owner 扩大：将 Backlog 中的 wildcard owner 收敛为精确 CR153-S05 return/evidence/CP7 路径和 CR153 release context；`docs/release/RELEASE-NOTES.md` 仅在 CP8 明确要求 CR153 section 时适用，否则 N/A。
- 无授权扩大：本技术说明不授权真实 feed/listener/lake/NAS/provider/runtime/broker/credential/store/catalog/registry/order flow。
