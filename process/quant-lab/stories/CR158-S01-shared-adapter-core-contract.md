---
story_id: CR158-S01-shared-adapter-core-contract
title: Shared adapter core and validation result contract
story_slug: shared-adapter-core-contract
status: verified-with-risk
priority: P0
wave: CR158-W1-CORE
depends_on: []
dependency_contracts: []
feature_design_refs:
- docs/features/factor-research-loop/DESIGN.md
- docs/features/factor-research-loop/TEST-PLAN.md
- docs/features/factor-research-loop/TASKS.md
- docs/features/runtime-authorization-safety/DESIGN.md
lld_policy:
  required_level: full-lld
  trigger_reasons:
  - shared-contract
  - cross-story-contract
  - security-boundary
  - failure-model
  rationale: Defines StrategyTypeAdapterCore and AdapterValidationResult consumed
    by all CR158 implementation Story.
  waiver_reason: ''
  revisit_condition: If CP5 finds shared core too broad or too narrow, return to CP3/CP5
    design clarification.
  evidence_path: process/stories/CR158-S01-shared-adapter-core-contract-LLD.md
file_ownership:
  primary:
  - engine/strategy_type_adapters.py
  - tests/research/test_strategy_type_adapter_core.py
  shared:
  - docs/features/factor-research-loop/DESIGN.md
  - docs/features/runtime-authorization-safety/DESIGN.md
  merge_owner: CR158-S01-shared-adapter-core-contract
  forbidden:
  - .env
  - data/**
  - /home/hyde/data/**
lld_gate:
  design_evidence_type: full-lld
  design_evidence_path: process/stories/CR158-S01-shared-adapter-core-contract-LLD.md
  status: approved
  confirmed: true
  confirmed_at: '2026-07-05T18:05:00+08:00'
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
  implementation_allowed: local_static_fixture_only
created_at: '2026-07-05T17:35:00+08:00'
updated_at: '2026-07-05T17:55:00+08:00'
implementation_allowed: local_static_fixture_only
cp5_approved_at: '2026-07-05T18:05:00+08:00'
cp5_approved_by: user
cp6_result_ref: process/checks/CP6-CR158-EVENT-ML-STRATEGY-ADAPTER-IMPLEMENTATION.result.json
cp7_result_ref: process/checks/CP7-CR158-EVENT-ML-STRATEGY-ADAPTER-VERIFICATION.result.json
release_gate_ref: process/checkpoints/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.md
verification_gate:
  status: pass_with_risk
  decision: PASS_WITH_RISK
  ref: process/checks/CP7-CR158-EVENT-ML-STRATEGY-ADAPTER-VERIFICATION.result.json
closure_status: closed-current-delivery-ready-with-risk
cp8_result_ref: process/checks/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.result.json
closed_at: '2026-07-05T19:20:00+08:00'
---

## 目标

定义 CR158 event/ML adapters 共用的 thin `StrategyTypeAdapterCore` 与 `AdapterValidationResult`，为后续 event、ML、evidence、guard 和 release Story 提供冻结合同。

## 开发上下文（dev_context）

- 输入文件：`docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md`、`docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md`、`docs/product/REQUIREMENTS.md`
- 输出文件：planned `engine/strategy_type_adapters.py`、planned `tests/research/test_strategy_type_adapter_core.py`
- 设计约束：core 只包含 strategy type、input refs、output signal refs、evidence refs、blocked reason refs、authorization flags、handoff refs。
- 平台目标：local/static/fixture only；no runtime。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| N/A | N/A | N/A | N/A | 本 Story 是 CR158 contract root。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `engine/strategy_type_adapters.py` | 当前 Story 定义 core 合同，后续 Story 只能扩展本 LLD 冻结的 extension slots。 |
| primary | `tests/research/test_strategy_type_adapter_core.py` | 当前 Story 拥有 shared core contract tests。 |
| forbidden | `.env`, `data/**`, `/home/hyde/data/**` | 不得读取或写入真实数据、凭据或外部运行态。 |

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | full-lld (see LLD file) |
| 设计依据 | CR158 HLD §4/§5、ADR-CR158-001、FEATURE-DESIGN-MATRIX CR158 增量 |
| 文件影响 | planned create/modify `engine/strategy_type_adapters.py`; planned create `tests/research/test_strategy_type_adapter_core.py` |
| 接口 / 数据 / 权限变化 | 新增本地 typed contract；不新增 runtime 权限。 |
| 异常、失败与回退 | 缺 mandatory core refs 或 forbidden flag 非法时 blocked。 |
| 测试入口 | planned `tests/research/test_strategy_type_adapter_core.py` |
| 风险与重访条件 | 若 core 变成 fat schema，回到 CP5 设计澄清或拆分子 CR。 |

## 量化验收标准（acceptance_criteria）

- [ ] Shared core 字段类别 >= 7。
- [ ] Event-only 字段数 == 0 in shared core。
- [ ] ML-only 字段数 == 0 in shared core。
- [ ] Forbidden operation authorization defaults are all false.
