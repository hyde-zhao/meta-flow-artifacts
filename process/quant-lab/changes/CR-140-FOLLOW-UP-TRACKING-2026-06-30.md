---
source_cr: "CR-140"
status: "closed-with-candidates"
created_at: "2026-06-30T16:48:00+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-30T17:35:00+08:00"
checkpoint_source: "CP8-post-close-review"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR140 后续事项跟踪台账

## 目的

本台账记录 CR140 关闭后的风险补登与候选事项。CR140 已关闭为 `closed-current-delivery / READY_WITH_RISK`；本台账不重开 CR140，不预创建正式 CR，不授权真实 lake、NAS、provider、lake/catalog write、QMT/live/runtime、simulation/trading 或 Git remote write。

## 结构化候选项

```yaml
follow_up_items:
  - id: "FU-CR140-001"
    title: "Real lake readonly turnover semantic validation"
    kind: "runtime-authorization"
    lifecycle_status: "candidate"
    readiness_status: "n/a"
    gate_status: "not_started"
    gate_profile: "standard"
    source_cr: "CR-140"
    source_decision_id: "CP8-CR140-POST-CLOSE-REVIEW"
    priority: 2
    formal_cr_path: ""
    blocked_by:
      - "requires_explicit_user_selection"
      - "requires_real_lake_readonly_authorization"
    impact_surface:
      - "turnover_factor_real_lake_semantic_validation"
      - "readonly_lake_access"
    conflict_keys:
      security_runtime:
        - "real_lake_read_authorization_required"
        - "no_lake_write"
        - "no_catalog_pointer_write"
        - "no_runtime"
    authorization_required:
      real_lake_readonly: true
      lake_write: false
      catalog_pointer_write: false
      runtime: false
      trading_write: false
    next_action: "仅当用户明确要求真实 lake readonly semantic validation 时启动独立授权 CR。"
  - id: "FU-CR140-002"
    title: "Remaining experiments engine convergence"
    kind: "requirement-change"
    lifecycle_status: "candidate"
    readiness_status: "n/a"
    gate_status: "not_started"
    gate_profile: "standard"
    source_cr: "CR-140"
    source_decision_id: "CP8-CR140-POST-CLOSE-REVIEW"
    priority: 3
    formal_cr_path: ""
    blocked_by:
      - "requires_explicit_user_selection"
      - "requires_engine_extension_contract_review"
    impact_surface:
      - "experiments/run_experiment_17_21_factor_suite.py"
      - "experiments/run_experiment_23_29_ml_factor_suite.py"
      - "engine/factor_statistics.py"
      - "engine/factor_registry.py"
    conflict_keys:
      implementation:
        - "remaining_parallel_experiment_evaluation_logic"
        - "engine_extension_contract"
    authorization_required:
      source_code_change: true
      real_lake_readonly: false
      runtime: false
      trading_write: false
    next_action: "仅当用户要求继续收敛剩余实验评估栈时启动独立 refactor CR。"
  - id: "FU-CR140-003"
    title: "Red baseline debt triage and closure"
    kind: "requirement-change"
    lifecycle_status: "active"
    readiness_status: "not_ready"
    gate_status: "cp2_pending"
    gate_profile: "standard"
    source_cr: "CR-140"
    source_decision_id: "CP8-CR140-POST-CLOSE-REVIEW"
    priority: 1
    formal_cr_path: "process/changes/CR-141-RED-BASELINE-DEBT-TRIAGE-2026-06-30.md"
    related_active_cr: "CR-141"
    blocked_by: "related_active_cr=CR-141; formalized_as_CR-141"
    impact_surface:
      - "46_phase0_baseline_failures"
      - "tests/test_script_entrypoint_naming.py"
      - "tests/test_market_data_contracts.py"
      - "tests/test_experiment_17_21_factor_suite.py"
      - "tests/test_experiment_23_29_ml_factor_suite.py"
      - "CR139_debt"
      - "redesign_intermediate_debt"
    conflict_keys:
      quality:
        - "red_baseline_debt"
        - "cr139_root_script_naming_debt"
        - "market_data_engine_import_boundary"
        - "remaining_experiment_suite_failures"
    authorization_required:
      source_code_change: true
      test_change: true
      runtime: false
      real_lake_readonly: false
      trading_write: false
    next_action: "已正式化为 CR-141；先分类 46 条失败，再修复低风险静态/guardrail 债，E3 重叠项 deferred。"
  - id: "FU-CR140-004"
    title: "Local commits remote persistence gate"
    kind: "requirement-change"
    lifecycle_status: "candidate"
    readiness_status: "n/a"
    gate_status: "not_started"
    gate_profile: "compact"
    source_cr: "CR-140"
    source_decision_id: "CP8-CR140-POST-CLOSE-REVIEW"
    priority: 2
    formal_cr_path: ""
    blocked_by:
      - "requires_explicit_git_remote_write_authorization"
    impact_surface:
      - "source_repo_branch_ahead_7"
      - "artifact_repo_main_ahead_8_before_this_tracking_commit"
      - "git_remote_persistence"
    conflict_keys:
      release:
        - "git_remote_write_not_authorized"
        - "local_commit_loss_risk"
    authorization_required:
      git_remote_write: true
      runtime: false
      real_lake_readonly: false
      trading_write: false
    next_action: "如用户要求持久化本地 CR140 成果到远端，启动独立 Git remote write 授权门；本台账不执行 push。"
```

## 后续 CR / Spike 候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| FU-CR140-001 | Real lake readonly turnover semantic validation | candidate | runtime-authorization | 2 | real_lake_read_authorization_required; turnover_factor_real_lake_semantic_validation |  | blocked_by=requires_explicit_user_selection; requires_real_lake_readonly_authorization | not_started | 真实 lake readonly 未授权 | 用户明确选择后启动独立授权 CR | CP8-CR140-POST-CLOSE-REVIEW |
| FU-CR140-002 | Remaining experiments engine convergence | candidate | requirement-change | 3 | remaining_parallel_experiment_evaluation_logic; engine_extension_contract |  | blocked_by=requires_explicit_user_selection; requires_engine_extension_contract_review | not_started | 需独立重构 CR | 用户明确选择后启动独立 refactor CR | CP8-CR140-POST-CLOSE-REVIEW |
| FU-CR140-003 | Red baseline debt triage and closure | active | requirement-change | 1 | red_baseline_debt; cr139_root_script_naming_debt; market_data_engine_import_boundary; remaining_experiment_suite_failures | `process/changes/CR-141-RED-BASELINE-DEBT-TRIAGE-2026-06-30.md` | related_active_cr=CR-141; formalized_as_CR-141 | cp2_pending | 已由用户选择并正式化为 CR-141 | 由 CR-141 先分类 46 条失败，再修复低风险静态/guardrail 债，E3 重叠项 deferred | USER-20260630-CONTINUE-EXPERIMENT-REMEDIATION |
| FU-CR140-004 | Local commits remote persistence gate | candidate | requirement-change | 2 | git_remote_write_not_authorized; local_commit_loss_risk |  | blocked_by=requires_explicit_git_remote_write_authorization | not_started | Git remote write 未授权 | 用户明确授权后再启动 remote persistence gate；本台账不 push | CP8-CR140-POST-CLOSE-REVIEW |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 | 来源 |
|---|---|---|---|---|
| NA-CR140-01 | 真实 lake readonly validation | CR140 只授权 synthetic fixture parity | 启动 FU-CR140-001 对应正式 CR 并单独授权 | CP8-CR140 |
| NA-CR140-02 | NAS / provider fetch / lake write / catalog pointer write | CR140 明确禁止 | 新建独立高风险授权 CR | CP8-CR140 |
| NA-CR140-03 | QMT / MiniQMT / xtquant / gateway runtime / simulation / live / trading | CR140 明确禁止 | 新建 runtime/trading 授权门 | CP8-CR140 |
| NA-CR140-04 | Git remote write / push | CR140 明确禁止 remote write | 启动 FU-CR140-004 对应授权门 | CP8-CR140-POST-CLOSE-REVIEW |

## 关闭范围

| 项目 ID | 已关闭内容 | 关闭证据 | 来源 |
|---|---|---|---|
| CLOSE-CR140-01 | experiments 非实验文件归位 | `process/checks/CR140-PHASE1-BASELINE-COMPARISON-2026-06-30.md` | CR140 |
| CLOSE-CR140-02 | helper 收敛 | `process/checks/CR140-PHASE2-HELPER-CONSOLIDATION-2026-06-30.md` | CR140 |
| CLOSE-CR140-03 | turnover synthetic adapter parity | `process/checks/CR140-PHASE3-TURNOVER-SYNTHETIC-PARITY-2026-06-30.md`、`process/checks/CR140-POST-CLOSE-RISK-SUPPLEMENT-2026-06-30.md` | CR140 |
| CLOSE-CR140-04 | final validation no new failures | `process/checks/CR140-PHASE5-FINAL-VALIDATION-2026-06-30.md` | CR140 |

## Deferred 项

| 项目 ID | 内容 | 状态 | 原因 | 可重启条件 | 来源 |
|---|---|---|---|---|---|
| DEF-CR140-01 | 真实数据湖 turnover semantic validation | deferred | 未授权真实 lake readonly | 用户选择 FU-CR140-001 | CP8-CR140 |
| DEF-CR140-02 | 剩余实验评估栈 engine 化 | deferred | Phase 4 明确拆分后续 CR | 用户选择 FU-CR140-002 | CP8-CR140 |
| DEF-CR140-03 | 46 条红基线债清零 | active-as-CR-141 | CR140 只承诺不新增失败；用户已选择继续推进 | CR-141 执行分类与低风险收敛 | CP8-CR140-POST-CLOSE-REVIEW / USER-20260630-CONTINUE-EXPERIMENT-REMEDIATION |
| DEF-CR140-04 | push 本地 CR140 commits | deferred | Git remote write 未授权 | 用户选择 FU-CR140-004 并授权 remote write | CP8-CR140-POST-CLOSE-REVIEW |
