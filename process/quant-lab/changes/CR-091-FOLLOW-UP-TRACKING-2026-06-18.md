---
source_cr: "CR-091"
status: "open"
created_at: "2026-06-18T15:58:59+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-24T18:58:00+08:00"
checkpoint_source: "CP8"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR091 后续事项跟踪台账

## 目的

本台账记录 CR091 CP8 后续候选事项。候选项未启动前不得预创建正式 CR；只有用户决定推进某一项时，才在 `process/changes/` 下创建正式 CR，并把本台账中的状态改为 `active`。

CR091 已于 `2026-06-18T15:40:09+08:00` 关闭为 `closed-current-delivery / READY_WITH_RISK`。本台账不重开 CR091，也不授权真实 runtime、NAS、凭据、账户或交易动作。

## 分流总览

| 类别 | 数量 | 阻断当前交付 | 说明 |
|---|---:|---|---|
| 关闭范围 | 1 | 否 | CR091 离线 runner 研究 / 设计 / 实现 / fixture 验证已关闭 |
| 不授权范围 | 1 | 否 | 真实运行、NAS、凭据、账户、交易写和 publish 均不授权 |
| 风险接受项 | 3 | 否 | `R-CR091-CP7-001..003` 已由 CP8 接受 |
| 后续 CR 候选项 | 9 | 否 | `CR091-FU-01..05` 中 `CR091-FU-05` 已由 CR101 关闭；CR101 后续真实验证 gate 已规范为 `RA-CR101-001..003` / `FU-CR101-001` |
| 取消 / deferred 项 | 1 | 否 | 真实交易、真实账户和真实运行全部后置 |

## 结构化候选项

```yaml
follow_up_items:
  - id: FU-CR091-001
    legacy_ids:
      - CR091-FU-01
      - CR-092
    title: Real QMT readonly runtime smoke design gate
    kind: runtime-authorization
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: runtime
    formal_cr_path: process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: transferred_to_cr092_follow_up_tracking
  - id: FU-CR091-002
    legacy_ids:
      - CR091-FU-02
      - CR-100
    title: NAS package exchange gate
    kind: implementation-gate
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: compact
    formal_cr_path: process/changes/CR-100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-GATE-2026-06-19.md
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: real_nas_gate_requires_new_authorization
  - id: FU-CR091-005
    legacy_ids:
      - CR091-FU-05
    title: Cross-platform strategy delivery and adapter layer realignment
    kind: architecture-realignment
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: standard
    formal_cr_path: process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: reframed
    reframed_by: FU-CR091-005
    reframe_summary: Quant-lab owns the runner; MiniQMT is the current adapter target, while strategy delivery keeps a cross-platform target contract with QMT direct-run supported now.
    next_action: CR101 closed-current-delivery / READY_WITH_RISK; child validation gates require independent authorization.
  - id: RA-CR101-001
    legacy_ids:
      - QMT-DIRECT-RUN-VALIDATION-FU
    title: QMT direct-run validation authorization gate
    kind: runtime-authorization
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: runtime
    formal_cr_path: process/changes/CR-104-QMT-MINIQMT-TRADING-DAY-RUNTIME-VALIDATION-GATE-2026-06-21.md
    blocked_by: []
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: n/a
    next_action: CR104 is closed; optional future QMT terminal direct-run evidence is tracked as CR138-FU-RT-02.
  - id: RA-CR101-002
    legacy_ids:
      - MINIQMT-GATEWAY-ADAPTER-VALIDATION-FU
    title: MiniQMT gateway adapter validation authorization gate
    kind: runtime-authorization
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: runtime
    formal_cr_path: process/changes/CR-104-QMT-MINIQMT-TRADING-DAY-RUNTIME-VALIDATION-GATE-2026-06-21.md
    blocked_by: []
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: n/a
    next_action: CR104 readonly runtime validation is closed; future adapter implementation planning is tracked as CR138-FU-RT-01 after CR138 fixture CP7.
  - id: RA-CR101-003
    legacy_ids:
      - NAS-REAL-EXCHANGE-FU
    title: NAS real package exchange validation authorization gate
    kind: runtime-authorization
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: runtime
    formal_cr_path: process/changes/CR-102-NAS-REAL-PACKAGE-EXCHANGE-VALIDATION-AUTHORIZATION-GATE-2026-06-20.md
    blocked_by: []
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: n/a
    next_action: CR102 is closed; additional NAS work is excluded from CR138 and requires a separate storage/NAS gate.
  - id: FU-CR101-001
    legacy_ids:
      - ORDER-WRITE-SIMULATION-LIVE-FU
      - ORDER-WRITE-FU
      - CR091-FU-03
    title: Order-write / simulation / live design authorization gate
    kind: runtime-authorization
    lifecycle_status: candidate
    readiness_status: n/a
    gate_status: not_started
    gate_profile: standard
    formal_cr_path: ""
    blocked_by:
      - RA-CR101-001
      - RA-CR101-002
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: not_recommended_until_readonly_validation_gates_are_settled
  - id: FU-CR091-003
    legacy_ids:
      - CR091-FU-03
    title: Order-write / submit-cancel design gate
    kind: runtime-authorization
    lifecycle_status: superseded
    readiness_status: n/a
    gate_status: not_started
    gate_profile: standard
    formal_cr_path: ""
    superseded_by: FU-CR101-001
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: use_FU-CR101-001_as_current_candidate_key
  - id: FU-CR091-004
    legacy_ids:
      - CR091-FU-04
      - CR-093
    title: Ledger hygiene
    kind: implementation-gate
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    gate_profile: compact
    formal_cr_path: process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md
    current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
    historical_baseline_status: retained
    next_action: closed
```

## 后续 CR / Spike 候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR091-FU-01 | Real QMT readonly runtime smoke design gate | closed | CR | 1 | runtime_authorization; per_run_authorization; readonly_query_positions; credential_boundary; redacted_evidence | `process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md` | `CR-092` | closed | CR092 已关闭为 READY_WITH_RISK；真实运行仍未授权 | 后续候选转入 `process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md` | DQ-CP8-CR091-04 |
| CR091-FU-02 | NAS package exchange gate | closed | CR | 2 | nas_package_exchange; package_pull; package_publish; credential_boundary | `process/changes/CR-100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-GATE-2026-06-19.md` | `CR-100` | closed | CR100 已关闭为 READY_WITH_RISK；真实 NAS 仍未授权 | 后续真实 NAS publish / pull / copy / 校验需独立 gate | DQ-CP8-CR091-04 |
| CR091-FU-05 | Cross-platform strategy delivery and adapter layer realignment | closed | CR | 1 | strategy_package_contract; cross_platform_delivery_target; qmt_direct_run_target; quant_lab_runner_adapter_layer; miniqmt_gateway_adapter; goldminer_future_adapter; manifest_schema; evidence_redaction | `process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md` | `CR-101` | closed | 用户已批准 CR101 CP8；CR101 当前离线交付关闭为 READY_WITH_RISK。真实系统仍未授权或验证。 | 后续 QMT direct-run、MiniQMT gateway、NAS real exchange、order-write / simulation / live 均保留为独立 candidate gate，不自动启动。 | USER-20260619-RUNNER-ADAPTER-AND-CROSS-PLATFORM-DELIVERY / DQ-CP8-CR101-01..05 |
| RA-CR101-001 | QMT direct-run validation authorization gate | closed-current-delivery | CR | 1 | qmt_direct_run_target; qmt_terminal_runtime; runtime_authorization; redacted_evidence; credential_boundary | `process/changes/CR-104-QMT-MINIQMT-TRADING-DAY-RUNTIME-VALIDATION-GATE-2026-06-21.md` | legacy=QMT-DIRECT-RUN-VALIDATION-FU / non-runtime=CR-103 / runtime=CR-104 / replaced_by=CR138-FU-RT-02 | closed | CR104 已关闭；QMT terminal direct-run 未在 CR104 中覆盖 | 如仍需要 QMT terminal direct-run evidence，按 CR138-FU-RT-02 新建 scoped runtime gate | DQ-CP8-CR101-04 / CR138 audit |
| RA-CR101-002 | MiniQMT gateway adapter validation authorization gate | closed-current-delivery | CR | 2 | quant_lab_runner_adapter_layer; miniqmt_gateway_adapter; readonly_gateway; runtime_authorization; credential_boundary | `process/changes/CR-104-QMT-MINIQMT-TRADING-DAY-RUNTIME-VALIDATION-GATE-2026-06-21.md` | legacy=MINIQMT-GATEWAY-ADAPTER-VALIDATION-FU / non-runtime=CR-103 / runtime=CR-104 / replaced_by=CR138-FU-RT-01 | closed | CR104 readonly runtime validation 已关闭；CR138 先实现 fixture-only Gateway service layer | 真实 adapter 实现规划转 CR138-FU-RT-01，需 CR138 fixture CP7 后另行授权 | DQ-CP8-CR101-04 / CR138 audit |
| RA-CR101-003 | NAS real package exchange validation authorization gate | closed-current-delivery | CR | 3 | nas_package_exchange; package_publish; package_pull; package_copy; credential_boundary | `process/changes/CR-102-NAS-REAL-PACKAGE-EXCHANGE-VALIDATION-AUTHORIZATION-GATE-2026-06-20.md` | legacy=NAS-REAL-EXCHANGE-FU / formal=CR-102 | closed | CR102 已关闭；额外 NAS 动作不属于 CR138 | 从 CR138 后续功能计划排除；额外 NAS 需独立 storage/NAS gate | DQ-CP8-CR101-04 / CR138 audit |
| FU-CR101-001 | Order-write / simulation / live design authorization gate | candidate | CR | 4 | submit_cancel; order_write; simulation_live; trading_runtime_boundary |  | legacy=ORDER-WRITE-SIMULATION-LIVE-FU / ORDER-WRITE-FU / CR091-FU-03 | 未启动 | 最高风险，应等待 readonly runtime / adapter evidence 稳定后再评审 | 不建议立即启动 | DQ-CP8-CR101-04 |
| CR091-FU-03 | Order-write / submit-cancel design gate | superseded | CR | 4 | submit_cancel; order_write; simulation_live; trading_runtime_boundary |  | superseded_by=FU-CR101-001 | 未启动 | 已由 CR101 后续 gate 命名规范重解释 | 不作为当前候选主键；保留历史别名 | DQ-CP8-CR091-04 / USER-20260619-RUNNER-ADAPTER-AND-CROSS-PLATFORM-DELIVERY |
| CR091-FU-04 | Ledger hygiene | closed | CR | 4 | CR019 tracking; CR025 nested active_change; cr_tracking consistency | `process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md` | `CR-093` | closed | CR093 已关闭为 READY_WITH_RISK；R-CR093-01/02 已由用户接受 | 当前交付已关闭；后续 warning cleanup / checker convergence 需用户另选候选后新建 CR | DQ-CP8-CR091-04 |

## 2026-06-19 架构重对齐补充

用户已明确两项架构约束：

1. runner 是 quant-lab 的 runner，不是 MiniQMT runner；MiniQMT 只是当前第一个接口适配目标，后续可能接掘金量化等其他接口。
2. 策略交付同样要按跨平台模型设计；当前只实现 / 验证 QMT direct-run target，但 manifest、entrypoint、validation 和 package layout 必须保留可扩展 target / adapter 边界。

因此，后续正式 CR 的推荐顺序已调整为以下历史计划；其中 `CR091-FU-05` 已由 CR101 承接并关闭，旧式自由文本 gate ID 仅作为 `legacy_id` 保留：

| 顺序 | 候选 | 推荐目标 | 当前支持范围 | 明确不做 |
|---:|---|---|---|---|
| 1 | `FU-CR091-005` / legacy `CR091-FU-05` | 重对齐策略包 target taxonomy、quant-lab runner adapter protocol、QMT direct-run 当前 target、MiniQMT gateway 当前 adapter contract | 已由 `CR-101` 关闭为 READY_WITH_RISK | 真实 QMT/MiniQMT/NAS/runtime/交易仍未授权 |
| 2 | `RA-CR101-001` / legacy `QMT-DIRECT-RUN-VALIDATION-FU` | 在用户环境证明 QMT 可直接加载当前策略 target | 需要独立逐 run 授权和脱敏 evidence | 不由 agent 自动代跑，不读取账号 / 日志原文 |
| 3 | `RA-CR101-002` / legacy `MINIQMT-GATEWAY-ADAPTER-VALIDATION-FU` | 证明 quant-lab runner 通过 MiniQMT gateway adapter 完成只读链路 | 仅 health / capabilities / query_positions，只保留脱敏 evidence | 不把策略放到 MiniQMT runner 中运行 |
| 4 | `FU-CR101-001` / legacy `ORDER-WRITE-SIMULATION-LIVE-FU` | order-write / submit-cancel / simulation-live 设计门禁 | 等 adapter protocol、QMT target 和只读 evidence 稳定后再启动 | 不建议当前启动 |

`FU-CR091-005` 的正式 CR 为 `CR-101 Cross-Platform Strategy Delivery and Adapter Layer Realignment`，已于 `2026-06-20T10:13:18+08:00` 关闭。后续候选不得复用旧启动交接作为当前入口。

## 2026-06-20 CR101 后续授权 Gate

CR101 S04 将以下对象登记为后续候选 gate。它们不是当前授权范围，不能因 CR101 CP8 approve 自动启动。

| Candidate ID | Legacy ID | 状态 | 目标 | 启动条件 | 不授权边界 |
|---|---|---|---|---|---|
| RA-CR101-001 | QMT-DIRECT-RUN-VALIDATION-FU | candidate | 真实 QMT terminal direct-run 验证 | 用户明确发起 runtime_authorization CR，并确认 evidence 脱敏方式 | 当前不授权 agent 代跑 QMT、读取账号/日志或声明真实 ready。 |
| RA-CR101-002 | MINIQMT-GATEWAY-ADAPTER-VALIDATION-FU | candidate | MiniQMT gateway readonly adapter 验证 | 用户明确发起 readonly runtime gate，限定 health / capabilities / query_positions | 当前不授权 gateway connect、账户原文、订单写入或 runtime 启动。 |
| RA-CR101-003 | NAS-REAL-EXCHANGE-FU / CR-102 | active / real-nas-package-exchange-validated-pass-pending-cp8 | 真实 NAS package exchange 验证 | 研究机侧 NAS package exchange 已执行 PASS；执行机侧逐文件 relative path / sha256 / bytes 验证 PASS，已记录 `runs/RUN-EXEC-20260621-003.md` | 当前不读取凭据/env/账户，不启动 runtime；等待 CR102 CP8 收口确认或用户选择后续 runtime gate。 |
| FU-CR101-001 | ORDER-WRITE-SIMULATION-LIVE-FU | candidate | order-write / simulation / live 设计门禁 | S01-S04 完成且用户明确接受高风险门禁 | 当前不授权 submit/cancel/buy/sell、simulation/live 或交易。 |

S04 收口文档路径：`docs/qmt/CR101-VALIDATION-AND-FOLLOW-UP-GATES.md`。

## CR 冲突预检

| 检查项 | 结果 | 证据 | 处理结论 |
|---|---|---|---|
| 是否已有 `STATE.md.active_change` | PASS | CR092 创建前顶层 `active_change` 为空 | 允许启动 CR092 |
| 是否存在未关闭正式 CR | PASS_WITH_RISK | `meta-flow check cr-tracking` 报历史旧账，但无当前 active formal CR 锁 | 历史旧账转 CR091-FU-04 |
| 正式文档影响面是否重叠 | PASS | CR091 / CR089 相关但均非 active execution | CR092 只做设计门禁 |
| Story / LLD 批次是否重叠 | PASS | CR092 新建 scoped LLD 批次 | 不复用 CR091 LLD 为执行授权 |
| 文件 owner 是否冲突 | PASS | 当前不改代码 | CP5 前不得实施 |
| 外部接口 / 安全 / 运行授权是否重叠 | PASS_WITH_RISK | 真实 runtime 高风险 | 进入 standard 模式和人工门禁 |
| `STATE.md.active_change` 是否指向已关闭 CR | PASS_WITH_RISK | 顶层已切为 CR092；历史嵌套 CR025 旧账仍存在 | 不阻塞 CR092，保留给 CR091-FU-04 |
| 台账候选与正式 CR 文件是否同步 | PASS | 本台账 `CR091-FU-01` 链接 CR092 | 已同步 |

## 状态索引同步

| 对象 | 路径 | 同步要求 | 当前状态 |
|---|---|---|---|
| 运行时状态 | `process/STATE.md.cr_tracking` | 记录 active、blocked、candidate、spike_candidate、stale_status_conflicts | CR093 closed-current-delivery；无 active formal CR |
| CR 索引 | `process/changes/CR-INDEX.yaml` | 记录每个候选项的状态、正式 CR 路径、影响面、blocked_by 和下一步 | CR093 closed-current-delivery |
| 一致性检查 | `meta-flow check cr-tracking --project-root .` | 新增台账、启动候选、关闭 CR 或状态冲突修复后执行 | CR019 / CR025 旧账阻断已治理；剩余 broader warnings 为 warning-only |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 | 来源 |
|---|---|---|---|---|
| NA-CR091-01 | QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、账户、submit/cancel、simulation/live、provider/lake/publish | CR091 CP8 只关闭离线 runner 交付；CR092 创建也只进入设计门禁 | 必须在独立人工门禁和逐 run 授权中明确 | DQ-CP8-CR091-03 |

## 风险接受项

| 项目 ID | 风险 | 接受条件 | 回退 / 切换条件 | 来源 |
|---|---|---|---|---|
| RA-CR091-01 | 离线 CP7 不证明真实 runtime ready | CP8 接受 READY_WITH_RISK | 启动 runtime authorization gate | R-CR091-CP7-001 |
| RA-CR091-02 | 历史 cr-tracking 旧账仍存在 | 不阻塞 CR091 / CR092 当前设计门禁 | 启动 ledger hygiene | R-CR091-CP7-002 |
| RA-CR091-03 | evidence output path 未启用 | 后续真实 evidence 需独立 gate | 纳入 CR092 或 future evidence output gate | R-CR091-CP7-003 |

## 关闭范围

| 项目 ID | 已关闭内容 | 关闭证据 | 来源 |
|---|---|---|---|
| CLOSE-CR091-01 | CR091 离线 runner 研究 / 设计 / 实现 / fixture 验证 | `process/checkpoints/CP8-CR091-DELIVERY-READINESS.md` | CP8 |

## 取消 / Deferred 项

| 项目 ID | 内容 | 状态 | 原因 | 可重启条件 | 来源 |
|---|---|---|---|---|---|
| DEF-CR091-01 | 真实交易、真实账户、真实运行 | deferred | 不在 CR091 关闭范围；CR092 也先做设计门禁 | 用户明确授权并通过对应门禁 | CP8 |
