---
handoff_id: "NEXT-SESSION-CR150-UC58-60-CONFIRMATION-AND-LINKAGE-2026-07-01"
from_agent: "host-orchestrator"
to_agent: "next-host-orchestrator"
status: "handoff-created"
created_at: "2026-07-01T16:35:00+08:00"
workflow_id: "ROADMAP-QUANT-RESEARCH-PRODUCTION"
change_id: "CR-150"
handoff_reason: "User started Codex from the wrong project directory and will restart from quant-lab."
dispatch:
  required: false
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "host-orchestrator"
  canonical_role: "host-orchestrator"
  codex_agent_name: ""
  reasoning_profile: "default"
  dispatch_trigger: "context-reset-and-project-root-correction"
  agent_path: ""
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: "handoff-only for restarting Codex in /home/hyde/workspace/quant-lab; no subagent execution claimed"
  fallback_reason: "Current request is a next-session context handoff, not a functional subagent dispatch."
  approved_by: "user"
  approved_at: "2026-07-01T16:35:00+08:00"
question_permission:
  can_ask_user: true
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Confirm UC-58/UC-59/UC-60, external references, and no-authorization boundary; ask implementation clarification only after user approves the scenario baseline."
  forbidden_question_scope: "runtime authorization, credentials, provider fetch, NAS sync/write, catalog pointer mutation, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, broker write, Git remote write"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: ""
  context_ref: "process/handoffs/NEXT-SESSION-CR150-UC58-60-CONFIRMATION-AND-LINKAGE-2026-07-01.md"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 10
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "Restart from corrected project root; next session needs enough formal context to confirm UC-58/59/60 and then resume CR150 linkage implementation."
  allowed_reads:
    - "process/USE-CASES.md"
    - "process/changes/CR-150-MULTIFACTOR-FRAMEWORK-COMPLETION-2026-07-01.md"
    - "process/changes/CR-INDEX.json"
    - "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md"
    - "process/handoffs/NEXT-SESSION-CR150-MULTIFACTOR-FRAMEWORK-COMPLETION-2026-07-01.md"
    - "process/evidence/CR147-PHASE2-RESEARCH-PRODUCTION-CONTRACT-AUDIT.index.json"
    - "process/evidence/CR147-PHASE2-FEATURE-AVAILABILITY-GATE.index.json"
    - "process/evidence/CR148-PHASE3-BACKTEST-FOUNDATION-ASSET-MAP.index.json"
    - "process/evidence/CR148-PHASE3-BACKTEST-RUN-SPEC.index.json"
    - "process/evidence/CR148-PHASE3-BACKTEST-REPORT-PACK.index.json"
    - "process/evidence/CR148-PHASE3-BACKTEST-COST-RISK-ATTRIBUTION.index.json"
  must_read:
    - "process/USE-CASES.md"
    - "process/changes/CR-150-MULTIFACTOR-FRAMEWORK-COMPLETION-2026-07-01.md"
    - "process/changes/CR-INDEX.json"
  read_if_needed:
    - "engine/multifactor_contracts.py"
    - "engine/factor_panel_contracts.py"
    - "engine/factor_portfolio_practice.py"
    - "engine/factor_registry.py"
    - "engine/factor_library.py"
    - "engine/mature_multifactor_research.py"
    - "engine/portfolio.py"
    - "engine/mature_multifactor_framework.py"
    - "engine/backtest.py"
    - "engine/strategy_admission_package.py"
    - "tests/test_stage2_mature_multifactor_framework.py"
    - "tests/test_stage3_mature_multifactor_research.py"
    - "tests/test_cr030_factor_panel_label_window_gates.py"
    - "tests/test_cr030_strategy_admission_package.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/stories/*-LLD.md"
    - "scripts/legacy/**"
    - "full conversation transcript"
    - "full historical CR139 evidence"
---

# 下一会话交接：CR150 场景确认与多因子 linkage 实施

## 先纠正启动目录

本轮用户说明 Codex 启动项目目录错了。下一会话必须从 quant-lab 项目根启动：

```bash
cd /home/hyde/workspace/quant-lab
```

不要继续在 `/home/hyde/workspace/meta-flow` 作为工作根推进 quant-lab 代码和 process 任务。

已执行 process 路由健康检查：

```text
uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab
process_link_health: ok
project_root: /home/hyde/workspace/quant-lab
link_path: /home/hyde/workspace/quant-lab/process
routing_mode: symlink
actual_target: /home/hyde/workspace/meta-flow-artifacts/process/quant-lab
state_path: /home/hyde/workspace/quant-lab/process/STATE.md
artifact_git_dirty: dirty
```

## 本轮已完成

本轮只做了场景草案增量，没有进入 CR150 代码实现。

已更新：

- `process/USE-CASES.md`

关键改动：

- frontmatter `version` 从 `1.17` 改为 `1.18`
- `total_use_cases` 从 `57` 改为 `60`
- 修订记录新增 `1.18`
- 新增 `## CR-150 / 生产路线外部依据与本项目取舍（待确认）`
- 新增 `UC-58：多因子策略端到端生产全流程（待确认）`
- 新增 `UC-59：机器学习策略端到端生产全流程（待确认）`
- 新增 `UC-60：事件驱动策略端到端生产全流程（待确认）`
- 新增 `## UC-58 至 UC-60 当前不授权边界（待确认）`
- 覆盖自检表扩展到 UC-60
- 治理变更记录追加 `1.18`

外部依据已写入 `process/USE-CASES.md`，包括 Qlib、Alphalens、Pyfolio、Zipline、Backtrader、NautilusTrader、QuantConnect LEAN、RQAlpha、vn.py、FinRL、Backtest Overfitting / PBO、Advances in Financial Machine Learning / Purged CV / Embargo 等。写法明确：这些只作为流程设计参考，不作为默认依赖、框架迁移、源码移植或运行授权。

## 当前待用户确认

下一会话首先向用户确认以下三项。用户回复 `approve` 表示一次性接受推荐方案。

1. 是否接受 `UC-58` 多因子端到端生产全流程。
2. 是否接受 `UC-59` 机器学习策略端到端生产全流程，作为后续 CR 输入，不并入 CR150 实现。
3. 是否接受 `UC-60` 事件驱动策略端到端生产全流程，作为后续 CR 输入，不并入 CR150 实现。

同时确认当前不授权边界：

- 不授权 provider fetch。
- 不授权真实 lake read/write。
- 不授权 catalog pointer mutation。
- 不授权 NAS sync/write/restore。
- 不授权 credential read。
- 不授权 QMT / MiniQMT / xtquant / gateway runtime。
- 不授权 simulation、live、paper runtime。
- 不授权 broker write、submit / cancel、账户 / 持仓 / 成交真实查询。
- 不授权 Git remote write。
- 不授权依赖变更、外部框架 clone / install / run、源码级迁移或替换 lightweight 主路径。

## CR150 当前真实范围

CR150 仍是唯一 active formal CR。目标不是重建多因子契约，而是：

```text
基于 CR147/CR148 已完成契约与 backtest foundation，
补齐 daily multifactor baseline 的 end-to-end metadata linkage。
```

下一步实现应只覆盖 UC-58 中可落地的 CR150 子集：

```text
factor spec
  -> factor run / factor panel
  -> label / PIT / leakage gate refs
  -> signal set
  -> portfolio construction
  -> BacktestRunSpec
  -> report pack
  -> admission package
```

实现目标：

- 可追溯 refs
- deterministic hash chain
- forbidden / permission counters 汇总为 0
- ML / event-driven / live / NAS / runtime deferred routing

## 已修正的计划判断

不要重复 CR148 asset map。

CR148 已覆盖：

- `engine/mature_multifactor_framework.py`
- `engine/backtest.py`
- `engine/research_manifest.py`
- `engine/strategy_admission_package.py`

CR150 Phase A 应改为“增量 asset map + linkage gap 分析”，重点看：

- `engine/multifactor_contracts.py`
- `engine/factor_panel_contracts.py`
- `engine/factor_portfolio_practice.py`
- `engine/factor_registry.py`
- `engine/factor_library.py`
- `engine/mature_multifactor_research.py`
- `engine/portfolio.py`

每个链路节点标三态：

```text
existing_contract
linkage_missing
true_gap
```

## 测试路径注意

当前 quant-lab 的 `tests/` 是平铺结构，不存在旧 handoff 里写的 `tests/research/*` 或 `tests/backtest/*`。

候选相关测试文件：

- `tests/test_stage2_mature_multifactor_framework.py`
- `tests/test_stage3_mature_multifactor_research.py`
- `tests/test_cr030_factor_panel_label_window_gates.py`
- `tests/test_cr030_strategy_admission_package.py`
- `tests/test_cr030_multifactor_combiner.py`
- `tests/test_multifactor_strategy_candidates.py`

新增 CR150 测试建议命名：

```text
tests/test_cr150_multifactor_framework_completion.py
```

不要创建 `tests/research/` 或 `tests/backtest/` 子目录。

## 建议下一会话流程

1. 在 `/home/hyde/workspace/quant-lab` 启动 Codex。
2. 读取本交接文档。
3. 读取 `process/USE-CASES.md` 中 v1.18 新增的外部依据、UC-58、UC-59、UC-60、不授权边界。
4. 向用户发起一次性确认：是否 `approve` UC-58/59/60 + 外部依据 + 当前不授权边界。
5. 若用户 `approve`：
   - 保持 `UC-59` / `UC-60` 为后续 CR 输入，不进入 CR150 实现。
   - 进入 CR150 Phase A'：增量 asset map + linkage gap 分析。
6. 若用户要求修改：
   - 先更新 `process/USE-CASES.md` v1.18 草案。
   - 不进入代码实现，直到用户确认。

## 产物建议

用户确认后，CR150 的下一批产物建议为：

- `process/evidence/CR150-MULTIFACTOR-FRAMEWORK-COMPLETION-MAP-2026-07-01.json`
- `process/evidence/CR150-MULTIFACTOR-FRAMEWORK-COMPLETION-MAP.index.json`
- `process/checks/CP6-CR150-MULTIFACTOR-FRAMEWORK-COMPLETION.result.json`
- `tests/test_cr150_multifactor_framework_completion.py`

命名中使用 `COMPLETION` 而不是单纯 `ASSET-MAP`，避免误导为重复 CR148。

## 禁止事项

下一会话在用户确认前不得：

- 修改 engine / tests 代码。
- 执行 provider fetch、真实 lake read/write、NAS sync、catalog pointer mutation。
- 启动 QMT / MiniQMT / xtquant / gateway。
- 执行 simulation / live / trading / broker write。
- 安装或运行外部框架。
- 把 handoff-only 交接解释为子 agent 已执行。

## 当前状态摘要

```yaml
project_root: /home/hyde/workspace/quant-lab
process_root: /home/hyde/workspace/quant-lab/process
process_route_health: ok
active_cr: CR-150
current_use_cases_version: "1.18"
current_use_cases_status: draft
pending_user_decision: "approve UC-58/UC-59/UC-60 + external references + no-authorization boundary"
implementation_started: false
runtime_authorized: false
```
