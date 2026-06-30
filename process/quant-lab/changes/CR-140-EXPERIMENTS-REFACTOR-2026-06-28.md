---
cr_id: "CR-140"
title: "experiments/ 目录重构与并行因子栈消除"
cr_type: "refactor"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 experiments/ 目录结构重构、文件归位、导入面迁移、测试基线和 turnover 因子评估侧 engine 适配；按正式 CR 执行，不走 fast-lane。"
rollback_to: "pre-CR140 clean worktree + process/context/EXPERIMENTS-REFACTOR-PLAN-2026-06-28.md"
approval_result: "phase0-authorized-by-user"
created_at: "2026-06-30T14:03:30+08:00"
created_by: "host-orchestrator"
approved_by: "user-prior-no-risk-auto-continue-authorization"
approved_at: "2026-06-30T16:08:00+08:00"
source: "user-request"
linked_issue: ""
parent_cr: ""
source_checkpoint: "process/context/EXPERIMENTS-REFACTOR-PLAN-2026-06-28.md"
source_decision_id: "USER-20260630-START-CR140"
follow_up_type: "experiments-refactor"
risk_class: "structure-refactor-research-semantics"
owner: "host-orchestrator"
revisit_condition: "用户要求接入真实 lake/NAS/provider/catalog/QMT/live/runtime，或 Phase 3 离线 fixture 无法证明 golden fixture 等价时，停止当前 Phase 并发起独立授权或设计澄清。"
acceptance_criteria: "Phase 0 记录 pytest 基线并完成失败分类；Phase 1 完成经职责判定后的非实验文件归位且旧 import/string 仅保留历史说明；Phase 2 在 helper 等价 diff 后收敛重复 helper 并补实验编号索引；Phase 3 仅用合成 lake fixture 或注入层完成 turnover 重构等价 golden fixture 和 engine 适配；Phase 5 清理残留并通过全量 pytest 对比、入口检查和静态扫描。"
close_condition: "Phase 0-3 + Phase 5 完成，CR140 验证清单通过，无基线外新增失败；Phase 3 只能声明 turnover 新旧实现已在合成 fixture / 注入层上完成重构等价验证，不能声明真实数据研究语义已验证、不能声明 simulation/live/QMT 准入；未执行任何未授权真实 lake/NAS/provider/catalog/QMT/live/runtime 写入或读取验证。"
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
approval_focus: "implementation"
split_rationale: "本轮目标是 experiments/ 目录结构与研究因子评估侧的内部重构，独立于 CR139 数据湖整改 backlog，且明确不触碰真实湖写入、provider/catalog pointer、NAS、QMT/live/runtime。"
why_not_merge_with_parent: "CR139 Story backlog 已在当前状态摘要中闭环；CR140 的影响面集中在 experiments/、engine research reporting、script entrypoint 和离线 fixture 验证，和 CR139 真实 lake/current truth 指针整改不应混合归因。"
why_not_story_or_task: "涉及多文件迁移、导入面、测试基线、文档索引和 Phase 3 golden fixture，具有跨 Phase 回退与审计边界，需要正式 CR。"
decision_burden: "medium"
approve_effect: "授权进入 Phase 0 离线 pytest 基线记录；不授权 Phase 1-3 越过门禁、不授权真实数据湖验证。"
reject_effect: "CR140 停留在登记态，experiments/ 重构不启动。"
not_authorized_by_approve:
  - "real_data_lake_link_or_mount"
  - "nas_access"
  - "provider_fetch"
  - "lake_write"
  - "catalog_pointer_write"
  - "qmt_miniqmt_xtquant_gateway_runtime"
  - "simulation_live_or_trading"
  - "git_remote_write"
product_baseline_refresh_required: false
required_phase: ""
required_agent: ""
required_gate: ""
block_story_decomposition_until: ""
affected_product_docs: []
affected_use_cases: []
routing_design_ref: "process/context/EXPERIMENTS-REFACTOR-PLAN-2026-06-28.md"
conflict_keys:
  - "experiments-directory-refactor"
  - "research-reporting-module-location"
  - "data-lake-readiness-audit-entrypoint"
  - "turnover-factor-engine-adapter"
  - "offline-golden-fixture"
impact_surface:
  - "experiments/run_experiment_turnover_factor.py"
  - "experiments/run_data_lake_readiness_audit.py"
  - "experiments/lake_input_contract.py"
  - "experiments/training_snapshot_contract.py"
  - "scripts/legacy/cr/cr012_limited_window_lake_repair.py"
  - "experiments/reporting.py"
  - "experiments/production_current_truth_rerun.py"
  - "engine/research_reporting.py"
  - "engine/production_current_truth_rerun.py"
  - "scripts/data_lake/run_data_lake_readiness_audit.py"
  - "scripts/data_lake/repair_market_data.py"
  - "scripts/quality/check_script_entrypoints.py"
  - "docs/components/SCRIPT-ENTRYPOINTS.md"
  - "docs/components/EXPERIMENTS.md"
  - "tests"
  - "tests/fixtures/turnover_golden/"
  - "process/context/EXPERIMENTS-REFACTOR-PLAN-2026-06-28.md"
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_NAS"
  - "NO_TRADING"
  - "NO_PROVIDER_FETCH"
  - "NO_REAL_LAKE_LINK_OR_MOUNT"
  - "NO_LAKE_WRITE"
  - "NO_CATALOG_POINTER_WRITE"
  - "NO_REMOTE_WRITE"
---

# CR-140 experiments/ 目录重构与并行因子栈消除

## 变更描述

本 CR 登记 `process/context/EXPERIMENTS-REFACTOR-PLAN-2026-06-28.md` 中的单一可执行方案，目标是在不改变研究语义、不接入真实数据湖、不触碰运行时出口的前提下完成 `experiments/` 目录重构：

- Phase 0：记录当前 pytest 基线。
- Phase 1：将非实验文件归位到 `engine/` 或 `scripts/data_lake/`，同步导入面和稳定入口。
- Phase 2：收敛重复 helper，补 `docs/components/EXPERIMENTS.md` 实验编号索引。
- Phase 3：turnover 因子评估侧切 engine，但只能用合成 lake fixture 或注入层冻结 golden fixture。
- Phase 4：明确不做，拆后续独立 CR。
- Phase 5：清理残留并终验。

## 启动前事实

| 事实 | 当前结论 | 处理动作 |
|---|---|---|
| worktree 状态 | `git status --short` 两次为空 | 本轮启动时没有发现未提交变更；后续每次写入前继续复核。 |
| process route health | `uv run --python 3.11 meta-flow workspace check` PASS，artifact git dirty 为 clean | 可写入 `process/` 运行态文件。 |
| CR139 状态 | `STATE.current.json` / `process/STATE.md` 摘要写明 40/40 Story 已闭环；CR139 正文 frontmatter 曾停在 active/cp7_pending，导致 CR140 登记后 cr-tracking 报 multiple active formal CRs | 已作为范围外治理修正收敛为 `closed-current-delivery`，并写入 `process/checks/CR140-CR139-STATUS-RECONCILIATION-AUDIT-2026-06-30.md`；不改写 CR139 历史执行证据。 |
| 数据湖边界 | 用户明确要求不链接真实数据湖；Phase 3 用合成 lake fixture 或注入层 | 真实湖只读验证必须另行申请授权。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `refactor` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `active` |
| 就绪状态 | `not_ready` |
| 门禁状态 | `cp2_pending` |
| 门禁模板 | `standard` |

## 结构化权限策略

```yaml
authorization_policy:
  nas:
    access: false
    list: false
    read: false
    write: false
    publish: false
    delete: false
  provider_lake_catalog:
    provider_fetch: false
    real_lake_link_or_mount: false
    lake_read_validation: false
    lake_write: false
    catalog_pointer_write: false
  credentials:
    env_read: false
    secret_read: false
    account_read: false
  runtime:
    qmt: false
    miniqmt: false
    xtquant: false
    gateway: false
  trading:
    submit: false
    cancel: false
    simulation: false
    live: false
```

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/context/EXPERIMENTS-REFACTOR-PLAN-2026-06-28.md` | 不变 | 作为本 CR 范围基线保留 | 不适用 | baseline |
| `docs/components/SCRIPT-ENTRYPOINTS.md` | 原文档更新 | 保留既有 readiness CLI 说明；新增 production_strict readiness audit 差异说明 | 文档现有修订记录或新增条目 | pending |
| `docs/components/EXPERIMENTS.md` | 原文档更新 | 保留既有组件说明；新增实验编号索引 | 文档现有修订记录或新增条目 | pending |
| `process/changes/CR-INDEX.yaml` / `CR-INDEX.json` | 原文档更新 | 保留 CR139 既有不一致，不在本轮静默清理 | 本 CR 条目 | in-progress |
| `process/STATE.md` / `process/state/STATE.current.json` | 原文档更新 | 保留 CR139 已闭环摘要，当前 active scope 切到 CR140 | 状态更新时间 | in-progress |
| `process/checks/CR140-CR139-STATUS-RECONCILIATION-AUDIT-2026-06-30.md` | 新增 | 审计记录 CR139 状态收敛的治理边界 | 不适用 | completed |
| `process/checks/CR140-PHASE0-BASELINE-FAILURE-CLASSIFICATION-2026-06-30.md` | 新增 | Phase 0 失败分类，支持后续新增/修复失败判断 | 不适用 | completed |
| `process/checks/CR140-PLAN-REVIEW-ADJUSTMENTS-2026-06-30.md` | 新增 | 用户评审后的计划修订门 | 不适用 | completed |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义产品 REQ-* | 产品需求基线 | false | 不刷新产品基线；CR140 是内部结构重构。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | pytest 基线、turnover golden fixture | true | Phase 0 冻结基线；Phase 3 新增离线 golden fixture 覆盖边界语义。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | `EXPERIMENTS-REFACTOR-PLAN`、Phase 0-3/5 | true | 按 Phase 分段执行；Phase 4 拆后续 CR。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | lake/NAS/provider/catalog/runtime 边界 | true | 明确禁止真实 lake/NAS/provider/catalog/QMT/live/runtime；真实湖只读验证另行授权。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | engine/scripts/tests/docs/components | true | 每 Phase 后全量 pytest 对比 Phase 0 基线，补入口检查与静态 import scan。 |

## Phase 0 进入条件

- [x] 已读取执行计划、机器状态、CR 索引、目标脚本和 legacy repair 脚本。
- [x] 已复核 `git status --short`，启动时未发现 dirty worktree。
- [x] 已确认 CR139 状态需要作为启动前风险记录，不作为本轮隐式执行范围。
- [x] 已确认不链接真实数据湖，Phase 3 只能使用离线 fixture 或注入层。
- [x] 运行 `uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider` 并记录 Phase 0 基线：`46 failed, 1428 passed in 49.07s`。

Phase 0 证据：

- `process/checks/CR140-PHASE0-PYTEST-BASELINE-2026-06-30.md`
- `process/checks/CR140-PHASE0-BASELINE-FAILURES.txt`
- `process/checks/CR140-PHASE0-BASELINE-FAILURE-CLASSIFICATION-2026-06-30.md`

## Phase 1 迁移证据

Phase 1 已完成本轮识别出的 5 个非实验文件归位：

- `experiments/reporting.py` -> `engine/research_reporting.py`
- `experiments/production_current_truth_rerun.py` -> `engine/production_current_truth_rerun.py`
- `experiments/lake_input_contract.py` -> `engine/experiment_lake_input_contract.py`
- `experiments/training_snapshot_contract.py` -> `engine/training_snapshot_contract.py`
- `experiments/run_data_lake_readiness_audit.py` -> `scripts/data_lake/run_data_lake_readiness_audit.py`

Phase 1 证据：

- `process/checks/CR140-PHASE1-1-REPORTING-MIGRATION-2026-06-30.md`
- `process/checks/CR140-PHASE1-2-PRODUCTION-CURRENT-TRUTH-RERUN-MIGRATION-2026-06-30.md`
- `process/checks/CR140-PHASE1-CONTRACT-HELPERS-MIGRATION-2026-06-30.md`
- `process/checks/CR140-PHASE1-3-DATA-LAKE-READINESS-AUDIT-MIGRATION-2026-06-30.md`

Phase 1 全量 pytest 在源码 / artifact 未提交状态下出现 `48 failed, 1426 passed`，其中新增的 2 个失败为 CR132 dirty-workspace hygiene 对本轮未提交 source 变更与 CR139 收敛文件的拦截。按计划先提交 Phase 1 source slices 与 CR140 artifact evidence，再复跑全量 pytest 对比 Phase 0 failure set；不得把 dirty-workspace 失败计入业务回归。

Phase 1 baseline comparison：

- `process/checks/CR140-PHASE1-BASELINE-COMPARISON-2026-06-30.md`
- source / artifact clean 后全量 pytest：`46 failed, 1428 passed in 43.57s`
- 新增失败：0
- 修复的基线失败：0
- 保持失败：46，与 Phase 0 failure set 一致。

## Phase 2 helper 收敛证据

Phase 2 已完成 helper 收敛：

- 新增 `engine/experiment_report_helpers.py`
- 06/07、08、09、10、12 的 Markdown table / value formatting 改为共享 helper 包装，保留各自 `PERCENT_FIELDS`。
- 06/07、08、09、10、12、13 的 `_resolve_date_range` 本地副本已删除并复用共享实现。
- 13 的 `_markdown_table` / `_format_value` 因中文 `对比维度` / `PERCENT_DIMENSIONS` / `—` 处理非等价，保留本地实现。
- `docs/components/EXPERIMENTS.md` 已补实验编号索引。

证据：`process/checks/CR140-PHASE2-HELPER-CONSOLIDATION-2026-06-30.md`。

Phase 2 baseline comparison：

- `process/checks/CR140-PHASE2-BASELINE-COMPARISON-2026-06-30.md`
- source / artifact clean 后全量 pytest：`46 failed, 1428 passed in 43.27s`
- 新增失败：0
- 修复的基线失败：0
- 保持失败：46，与 Phase 0 failure set 一致。

## Phase 3 turnover 离线等价证据

Phase 3 已完成 turnover 因子评估侧 engine adapter：

- `calculate_abnormal_turnover` 委托 `engine.factor_calculators.calculate_abnormal_turnover_21_252`，保留旧实验窗口、min periods 与 `clip(0.01, 10.0)` 语义。
- `run_experiment_a` 委托 `engine.factor_statistics.single_sort_returns`。
- `run_experiment_b` 委托 `engine.factor_statistics.independent_double_sort_returns`。
- `run_experiment_c` 委托 `engine.factor_statistics.conditional_double_sort_returns`。
- adapter 将 engine 输出列转换回旧实验输出形态，Newey-West t-test 仍保留在实验层。
- `engine.factor_statistics._quantile_groups` 补齐旧实验对低样本 / 全常数输入不分组的行为。

证据：`process/checks/CR140-PHASE3-TURNOVER-SYNTHETIC-PARITY-2026-06-30.md`。

验证：

- `py_compile` PASS。
- Phase 3 targeted pytest：`11 passed in 3.69s`。
- `git diff --check` PASS。

Phase 3 声明边界：

- 允许声明：turnover 新旧实现已在合成 fixture / 注入层上完成重构等价验证。
- 禁止声明：真实数据湖研究语义已验证。
- 禁止声明：simulation/live/QMT 准入已验证。
- 未访问真实 lake、NAS、provider、catalog pointer、QMT/live/runtime。

Phase 3 baseline comparison：

- `process/checks/CR140-PHASE3-BASELINE-COMPARISON-2026-06-30.md`
- source / artifact clean 后全量 pytest：`46 failed, 1431 passed in 43.53s`
- 新增失败：0
- 修复的基线失败：0
- 保持失败：46，与 Phase 0 failure set 一致。
- 新增通过：3，来自 `tests/test_cr140_turnover_engine_adapter.py`。

## Phase 5 与 CP8 关闭证据

Phase 5 final validation：

- `process/checks/CR140-PHASE5-FINAL-VALIDATION-2026-06-30.md`
- source / artifact clean 后最终全量 pytest：`46 failed, 1431 passed in 43.66s`
- 新增失败：0
- `experiments/` 顶层只剩实验入口。
- 当前源码 / 组件文档范围旧 import / 旧路径残留：0。
- process artifact hygiene：PASS，`unclassified=0`。
- CR tracking：关闭前 PASS。
- `git diff --check`：PASS。

CP8 closure：

- `process/checkpoints/CP8-CR140-DELIVERY-READINESS.md`
- 关闭状态：`closed-current-delivery / READY_WITH_RISK`
- 未发起人工门禁原因：用户已授权“无风险继续推进，不发起人工门禁”；最终验证未发现新增风险。
- `READY_WITH_RISK` 风险含义：真实 lake 研究语义未验证，Phase 0 红基线仍存在；不是 CR140 新增回归风险。

CP8 post-close manual review fillback：

- 用户于 2026-06-30 独立复跑关键验证并确认叙述客观准确、证据可复现。
- 用户用 `comm -13` / `comm -23` 双向逐行对比确认最终 46 条失败集合与 Phase 0 完全相等，0 新增 / 0 消失。
- 人工审查结论：不重开 CR140；补登 CP8 人工审查回填，并明确 46 条基线失败仍是后续债务。

Post-close risk supplement：

- `process/checks/CR140-POST-CLOSE-RISK-SUPPLEMENT-2026-06-30.md`
- `process/changes/CR-140-FOLLOW-UP-TRACKING-2026-06-30.md`
- source commit：`a1f9026 test(experiments): cover turnover adapter quantile edges`
- 补登记风险：Phase 1 范围 3 -> 5 扩张、Phase 2 helper wrapper / 等价证据说明、Phase 3 低样本 / 重复值 quantile 边界覆盖、46 条红基线债归属、本地 commits 未 push 风险。
- 补强验证：targeted `13 passed in 3.75s`；全量 pytest `46 failed, 1433 passed in 44.38s`，新增失败 0。
- 新增候选：`FU-CR140-001` 真实 lake readonly semantic validation、`FU-CR140-002` 剩余实验 engine convergence、`FU-CR140-003` 红基线债 triage、`FU-CR140-004` remote persistence gate。

## 计划评审后的目标校正

本轮 CR140 的目标不是证明端到端生产链路可用，而是修正研究层与生产/数据工具层的代码边界，并证明一个高风险实验切 engine 的可复制方法。

校正后的目标表述：

> 整理研究实验层与生产/数据工具层的代码边界，并用离线 fixture 证明 turnover 实验评估侧迁移到 engine 后的重构等价性；本轮不验证真实数据湖、不产生模拟盘/实盘准入、不改变生产 current truth。

这意味着：

- Phase 1 后不得宣称 `experiments/` 已完全只剩实验入口，除非 `lake_input_contract.py` 与 `training_snapshot_contract.py` 已完成归位；否则必须把这两个 CR139 契约 helper 标为后续范围。
- turnover 切 engine 是示范性收敛，不代表 `experiments/` 整体评估逻辑已完成 engine 化；`exp15/16/17_21` 等仍按 Phase 4 拆独立 CR。
- Phase 3 golden fixture 只证明新实现等于旧实现在合成数据上的输出，不证明真实数据上的研究结论仍成立。
- turnover 研究产物当前未发现被 `engine/`、`scripts/`、`strategies/`、`trading/` 作为模拟盘/实盘准入直接消费；未来新增 consumer 必须单独审查。

## 计划调整项（进入 Phase 1 前生效）

| 调整项 | 执行要求 |
|---|---|
| CR139 状态收敛治理说明 | 作为范围外附带治理修正记录在 `CR140-CR139-STATUS-RECONCILIATION-AUDIT`；CP8 终验必须列出。 |
| Phase 0 失败分类 | 后续 Phase 不能只看失败总数；必须报告新增失败、修复的基线失败和保持失败。 |
| Phase 1 commit / 回退粒度 | 1.1 reporting、1.2 current truth rerun、1.3 readiness audit 三段分别验证，形成可独立提交/回退的变更组。 |
| Phase 1 职责判定 | `production_current_truth_rerun.py` 先判断是 reusable engine contract 还是 data governance script；不得默认进入 `engine/`。 |
| Phase 1 覆盖范围校正 | `lake_input_contract.py` 与 `training_snapshot_contract.py` 必须纳入归位判断；若本轮不迁，明确后续 CR 候选。 |
| Phase 2 helper 等价 | 删除本地 helper 前先 diff 本地副本与基座；非 byte-equivalent 或语义不等价则保留本地实现并登记原因。 |
| Phase 3 close wording | 只能写“合成 fixture 上重构等价验证通过”，不得写“真实数据研究语义已验证”。 |
| Phase 3 fixture 覆盖 | 至少覆盖低样本、全常数、重复值、valid_mask、rebalance_dates、spread 方向；未覆盖真实数据形状列为后续真实湖只读验证候选。 |
| Phase 5 回退复验 | 任一 Phase revert 后必须重跑全量 pytest 并对比 Phase 0 failure set，确认没有新增失败后才能继续。 |

## 回退决策

- 影响范围：局部结构重构，但跨 `experiments/`、`engine/`、`scripts/`、`tests/` 与组件文档。
- 回退到阶段：`pre-CR140 clean worktree + process/context/EXPERIMENTS-REFACTOR-PLAN-2026-06-28.md`
- 回退条件：任一 Phase 后出现基线外新增失败且两次尝试未修复，回退该 Phase 的代码变更并记录阻塞；回退后必须重跑全量 pytest 并对比 Phase 0 failure set，确认回到无新增失败状态后才能继续。

## Phase 3 离线约束

Phase 3 必须先冻结旧实现输出，再切 engine 适配；冻结与对比只能使用合成 lake fixture 或注入层。未经单独授权，不得执行真实 lake 只读验证，也不得读取、写入或发布 provider/catalog pointer。

Phase 3 验收措辞必须受限：

- 允许声明：新旧实现已在合成 fixture / 注入层上完成重构等价验证。
- 禁止声明：turnover 因子真实数据研究语义已验证。
- 禁止声明：该实验产物可进入 simulation/live/QMT 准入。
- 未覆盖的真实数据形状必须进入后续真实湖只读验证候选，且该验证需要单独授权。
