---
source_cr: "CR-152"
status: "open"
created_at: "2026-07-02T12:38:36+08:00"
created_by: "host-orchestrator-inline"
updated_at: "2026-07-04T09:31:03+08:00"
checkpoint_source: "CP7-static-fixture-only-verification"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR152 后续事项跟踪台账

## 目的

本台账记录 CR152 CP7 验证期间发现、但不属于 CR152 CP6/CP7 授权文件 owner 的后续治理候选项。CR152 本体继续按 Machine Learning Strategy E2E first-wave scope 推进 CP8；本台账不授权移动历史测试文件、不修改 `tests/PROVENANCE.yaml`，也不扩大 CR152 的 local/static/fixture-only 边界。

## 当前事实

| 项 | 结论 | 证据 |
|---|---|---|
| CR152 CP7 effective validation mode | `static-fixture-only` | `process/checks/CP7-CR152-ML-STRATEGY-E2E-VERIFICATION.result.json` |
| CR152 targeted tests | `5 passed` | `process/evidence/CR152-CP7-VERIFICATION.index.json#commands[1]` |
| CR151/CR152 scoped regression | `34 passed` | `process/evidence/CR152-CP7-VERIFICATION.index.json#commands[2]` |
| taxonomy guardrail probe | expected scope-out failure | `process/evidence/CR152-CP7-VERIFICATION.index.json#commands[4]` |
| full pytest claim | not claimed | `process/checks/CP7-CR152-ML-STRATEGY-E2E-VERIFICATION.result.json#summary` |

## 授权边界

| 类别 | 当前授权 |
|---|---|
| CR152 source / test implementation within CP5-approved owner files | authorized |
| CR152 CP7 process evidence and tracking updates | authorized |
| CR150/CR151 root-level test rename | not-authorized in CR152 |
| `tests/PROVENANCE.yaml` historical coverage cleanup | not-authorized in CR152 |
| full test-suite green claim | not-authorized; taxonomy/provenance issue is scope-out |

## 结构化候选项

```yaml
follow_up_items:
  - id: "FU-CR152-001"
    title: "tests taxonomy and PROVENANCE.yaml coverage hygiene"
    kind: "requirement-change"
    lifecycle_status: "candidate"
    readiness_status: "not_ready"
    gate_status: "not_started"
    gate_profile: "compact"
    source_cr: "CR-152"
    source_checkpoint: "CP7-CR152-ML-STRATEGY-E2E-VERIFICATION"
    priority: 2
    formal_cr_path: ""
    implementation_ref: "process/checks/FU-CR152-001-TEST-TAXONOMY-PROVENANCE-HYGIENE-2026-07-04.md"
    blocked_by:
      - "final_packaging_or_commit_not_executed"
    impact_surface:
      - "tests/test_cr150_multifactor_framework_completion.py"
      - "tests/test_cr151_strategy_admission_statistical_gate.py"
      - "tests/PROVENANCE.yaml"
      - "tests/meta_flow/test_test_file_taxonomy.py"
    conflict_keys:
      quality:
        - "test_taxonomy_hygiene"
        - "test_provenance_coverage"
        - "legacy_CR150_CR151_root_level_tests"
    authorization_required:
      source_code_change: false
      test_layout_change: true
      provenance_change: true
      runtime: false
      real_lake_readonly: false
      trading_write: false
    next_action: "Implemented under explicit user authorization on 2026-07-04. Closure remains local until final packaging/commit; Git remote write remains unauthorized."
```

## 实施更新（2026-07-04）

| 项 | 结果 | 证据 |
|---|---|---|
| 用户授权 | 用户明确要求实施该治理工作 | 当前会话 |
| 根目录游离测试文件 | 2 个已移动到 `tests/research/` 并去掉 `test_crNNN_` 前缀 | `tests/research/test_multifactor_framework_completion.py`、`tests/research/test_strategy_admission_statistical_gate.py` |
| `PROVENANCE.yaml` 缺失条目 | 11 条已补齐 | `tests/PROVENANCE.yaml` |
| taxonomy guardrail | PASS: `2 passed in 0.06s` | `uv run pytest tests/meta_flow/test_test_file_taxonomy.py -q` |
| affected targeted tests | PASS: `111 passed in 1.20s` | 11 个受影响测试文件集合 |
| full pytest | `1603 passed, 4 failed in 43.66s` | 4 个失败均为 process/design-surface hygiene 既有/过程态检查，不属于本次 taxonomy/provenance 目标 |
| 实施证据 | PASS_WITH_RESIDUAL_UNRELATED_FAILURES | `process/checks/FU-CR152-001-TEST-TAXONOMY-PROVENANCE-HYGIENE-2026-07-04.md` |

## 后续候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|
| FU-CR152-001 | tests taxonomy and PROVENANCE.yaml coverage hygiene | candidate | requirement-change | 2 | test taxonomy / provenance coverage / legacy CR150-CR151 root-level tests |  | not_started | final packaging/commit not executed | Local implementation and verification evidence complete; keep indexed until final packaging/commit closure, no Git remote write authorized | CR152 CP7 |

## 风险接受与门禁规则

| 规则 | 处理 |
|---|---|
| CR152 CP7 release decision | May proceed as `PASS_WITH_RISK` because the issue is scope-out hygiene. |
| CP8 wording | Must state static-fixture-only validation and no full pytest claim; must not claim real model performance, production readiness, registry publication, runtime readiness, trading readiness or broker readiness. |
| Candidate execution | Requires separate CR or explicit user authorization. |
| Test file movement / provenance mutation | Forbidden inside CR152 unless the CR152 scope is explicitly reopened. |
