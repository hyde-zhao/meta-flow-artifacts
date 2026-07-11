---
discussion_id: "CP2-CR163-SCENARIO-DISCUSSION"
cr_id: "CR-163"
phase: "requirement-clarification"
status: "completed-awaiting-host-cp2-precheck"
owner: "meta-pm"
created_at: "2026-07-11T11:35:00+08:00"
question_broker: "host-orchestrator"
formal_gate_opened: false
---

# CP2 CR163 Scenario Discussion Log

## Discussion Objective

确认 CR163 的真实用户目标：让未来 candidate-producing research 在首次搜索前声明 experiment family，运行中 append-only 记录 trial / attempt / selection，结束时以可复算 hash seal，并把可信 lineage availability/ref/raw count 提供给既有 admission consumer；本 CR 不计算 statistical correction，也不授权真实 runtime/data 操作。

## Repository-Evidenced Candidate-Producing Inventory

| Inventory ID | 路径 / symbol | 分类 | 结论 | 证据摘要 |
|---|---|---|---|---|
| CPI-CR163-001 | `scripts/research/run_multifactor_strategy_research.py::main` → `scripts/legacy/cr/run_stage3_mature_multifactor_research.py::main` → `engine.mature_multifactor_research.run_stage3_mature_multifactor_research` | P0 entrypoint | include | public wrapper 最终调用 Stage 3 orchestration；orchestration 直接产出 strategy candidate 与 mature admission package。 |
| CPI-CR163-002 | `scripts/legacy/research/run_multifactor_strategy_candidates.py` → `engine.multifactor_strategy_candidates.run_strategy_research` | P0 entrypoint | include | legacy CR039 runner 读取既有 research artifacts，调用 strategy candidate builder/refiner 并形成 admission package。 |
| CPI-CR163-003 | `engine.mature_multifactor_research.build_strategy_candidate` | P0 hook | include, same chain as 001 | 直接构造 Stage 3 `StrategyCandidate`，不能被 wrapper 绕过，也不能作为额外 trial 双计数。 |
| CPI-CR163-004 | `engine.multifactor_strategy_candidates.build_strategy_candidates` | P0 hook | include, same chain as 002 | 直接构造 CR039 candidate 集合，instrumentation 必须能覆盖筛选前后状态。 |
| CPI-CR163-X01 | `engine.anomaly_discovery.run_anomaly_discovery` | factor discovery | exclude | 产出 anomaly/factor catalog candidates，不是本 CR 的 strategy-admission family。 |
| CPI-CR163-X02 | `engine.mature_multifactor_framework.build_project_strategy_candidate_from_cr039` | compatibility adapter | exclude from producer denominator | 归一化已有 CR039 candidate，不发起新 search/trial。 |
| CPI-CR163-X03 | statistical/cross-strategy/admission package modules | consumer | exclude; integration target | 读取 lineage/count/availability，不生产候选。 |
| CPI-CR163-X04 | `BacktestRunSpec` / single-run `ExperimentManifest` builders | metadata contract | exclude; identity link only | 描述单次 run，不能替代 family lifecycle。 |
| CPI-CR163-X05 | UC-59 ML / UC-60 event adapter surfaces | fixture/static compatibility | N/A for real runner instrumentation | 当前没有已授权 real ML/event candidate runner，保留 shared producer contract compatibility。 |

`rg` inventory method: 先检索 `ExperimentManifest`、`BacktestRunSpec`、`trial_count`、candidate/admission construction symbols，再只展开命中的两个 runner、两个 construction hooks 和相关 consumer/contract 文件。没有扫描或改写历史脚本全集。

## Scenario Gray Areas

| Gray Area | Why it matters | Impact surface | Status |
|---|---|---|---|
| GA-CR163-01 inventory denominator | 决定 100% coverage 的可验证分母和绕过风险。 | scope / architecture / validation / maintenance | needs user confirmation |
| GA-CR163-02 trial vs attempt vs seed | 决定 raw count 能否抵抗 retry 双计数和 post-hoc family shrinkage。 | semantics / risk / validation | needs user confirmation |
| GA-CR163-03 raw vs effective count | 决定 lineage facts 是否被误读为 statistical proof。 | claim ceiling / admission / follow-up scope | needs user confirmation |
| GA-CR163-04 seal correction policy | 决定 tamper detection、审计链和恢复方式。 | integrity / recovery / release | needs user confirmation |

## SGQ Question Batch for Host-Orchestrator Relay

### SGQ-CR163-001 — Freeze the producer inventory

- 问题：是否冻结 `CPI-CR163-001..004` 为 P0 分母，即两条实际候选生产链及各自 direct construction hook；anomaly discovery、compatibility adapter、consumer、single-run contract 和不存在的 real ML/event runner 明确 excluded/N/A？
- 推荐：A（冻结 4 项 mapping / 2 条去重生产链）。
- 选项：
  - A（推荐）：4/4 mapping 为 100%，wrapper + hook 按同一 producer chain 去重。范围最小且能覆盖绕过点；CP3 可设计共享 instrumentation contract。
  - B：只覆盖 public Stage 3 chain。实现更小，但 legacy CR039 runner 可绕过 lineage，P0 coverage 存在已知缺口。
  - C：把 anomaly、所有 adapter 和所有 consumer 也纳入 producer 分母。覆盖面更宽，但混淆 factor discovery/consumer 与 producer，显著扩大架构和验证范围。
- 影响：A 需要两条 producer chain 的 fixture/static integration；B 接受 entrypoint gap；C 会扩大 Story、文件 owner 和 CP7 范围。
- blocking：是；未确认前不得冻结 CP2 inventory / 100% coverage 分母。

### SGQ-CR163-002 — Freeze count semantics

- 问题：是否确认“不同参数或 seed = 不同 trial；同一 trial retry = 新 attempt、不增加 raw trial；failed/cancelled/excluded 仍保留并计入 raw trial；identical duplicate delivery 幂等”？
- 推荐：A（完整 append-only 语义）。
- 选项：
  - A（推荐）：采用上述完整语义，可防 retry 双计数和 post-hoc family shrinkage。
  - B：failed/cancelled/excluded 不计数。表面 count 更小，但会低估 search breadth，削弱准入可信度。
  - C：每个 attempt 都计为 trial。实现直观，但 retry 会膨胀 raw count并混淆 search decision 与 execution reliability。
- 影响：决定 stable identities、validator、fixture 和 admission count meaning。
- blocking：是；未确认前 CP3 无法冻结 event/count contract。

### SGQ-CR163-003 — Keep effective count unavailable

- 问题：本 CR 是否强制 `effective_trial_count=typed_unavailable`，ref 与 method 为空，直到独立 statistical producer CR？
- 推荐：A（保持 unavailable）。
- 选项：
  - A（推荐）：CR163 只提供 raw lineage facts；统计 correction 继续由 FU-CR161-002 或后续独立 CR 承接。
  - B：在 CR163 同时实现 effective count。会引入统计方法选择、验证与风险接受，超出当前目标包。
  - C：用 raw count 填充 effective count。字段看似完整但语义错误，会形成 statistical overclaim。
- 影响：A 保持五 Story 目标和清晰 claim ceiling；B/C 扩大或破坏统计边界。
- blocking：是；未确认前 CP2 claim ceiling 不可基线化。

### SGQ-CR163-004 — Immutable seal and supersession

- 问题：seal 后的纠错是否只能创建 superseding version，并保留旧 ref/hash/version chain？
- 推荐：A（immutable + supersession）。
- 选项：
  - A（推荐）：旧版本不可变；correction 追加并 supersede，完整审计链可验证。
  - B：允许在受控条件下原地改写 seal。存储更少，但 tamper 与合法修正难以区分。
  - C：发现错误即删除旧 family 并重建。操作简单，但丢失历史且违反 append-only 目标。
- 影响：决定 artifact contract、recovery、hash validation、rollback 与审计成本。
- blocking：是；未确认前 CP3 无法冻结 persistence/integrity architecture。

## Eight-Dimension Coverage Scan

| 维度 | 状态 | CR163 处理 |
|---|---|---|
| 角色与价值 | 已覆盖 | research owner / researcher / admission reviewer / maintainer / verifier 均有 outcome。 |
| 正向流程 | 已覆盖 | declare → record → validate → seal → consume。 |
| 异常与失败 | 已覆盖 | missing、duplicate conflict、count mismatch、failed/cancelled/excluded。 |
| 边界 | 已覆盖 | retry、seed、idempotent delivery、raw/effective boundary。 |
| 权限与安全 | 已覆盖 | 所有 real data/runtime/external write 均 denied；非 0 counter blocked。 |
| 恢复与兼容 | 已覆盖 | immutable supersession；single-run manifest 与 legacy runner compatibility。 |
| 可观测与审计 | 已覆盖 | stable identities、hash、refs、machine-readable reasons、version chain。 |
| 非功能与维护 | 已覆盖 | 4/4 inventory、determinism 10/10、0 in-place mutation、100% negative fixture block。 |

## Discussion Status

- 已完成：仓库事实调查、候选理解、推荐范围、12 个 P0 工程场景，以及 SGQ-CR163-001..004 全部 option A 的用户显式确认。
- 用户确认：inventory 为 2 条去重 producer chains / 4 个 instrumentation mappings；append-only count semantics；effective count unavailable；immutable seal + supersession。
- Scope impact：不扩展范围、不增加 Story 数；仅将 CR163-S03 归一为同时覆盖两条 producer chains 与 CPI-CR163-001..004 4/4 mappings。
- Claim ceiling：合法 instrumented future run 仅在 seal/completeness/ref/count/tamper 全 PASS 后才可 `ExperimentFamilyManifest=present`；uninstrumented 为 `typed_unavailable`；invalid/tampered 为 `blocked`；CR163 只令 C1 raw-lineage input-ready，不使 C1 computable。
- CR155：继续作为 blocked negative regression，不回填历史 lineage、不提升 admission。
- 门禁：Scenario discussion 已完成；meta-pm 仍未打开 formal CP2。host-orchestrator 可据此准备 CP2 precheck 与 Decision Brief。
- 不授权项：真实 lake/NAS/provider/credential、runtime、broker/trading、external framework、remote write、publish、historical backfill、effective-trial/statistical correction。

## User Confirmation Record

| Question | User response | Confirmed understanding | Status |
|---|---|---|---|
| SGQ-CR163-001 | Option A | 4 instrumentation mappings across 2 deduplicated producer chains；excluded/N/A 分母不变。 | confirmed |
| SGQ-CR163-002 | Option A | seed/parameter 区分 trial；retry 区分 attempt；failed/cancelled/excluded 保留并计数。 | confirmed |
| SGQ-CR163-003 | Option A | effective count 继续 typed_unavailable；只准备 C1 raw-lineage input，不提供 C1 computation。 | confirmed |
| SGQ-CR163-004 | Option A | sealed version immutable；correction 只能 append + supersede。 | confirmed |
