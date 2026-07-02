---
story_id: "CR152-S03-training-model-prediction-metadata"
title: "Training, model and prediction metadata"
story_slug: "training-model-prediction-metadata"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "host-orchestrator"
created_at: "2026-07-02T10:37:00+08:00"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "TrainingSnapshotSpec / ModelArtifactRef extension"
    - "metadata vs model registry write boundary"
    - "prediction artifact metadata"
  rationale: "CR152 must link ML artifact metadata without introducing registry/store writes."
open_items: 0
---

# LLD: CR152-S03 — Training, Model and Prediction Metadata

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | Metadata vs registry write boundary, contract delta table |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | ADR-CR152-002 / 003 |
| CP5 Context | `process/context/CP5-CR152-ML-STRATEGY-E2E-CONTEXT.yaml` | No registry/store/catalog write authorization |
| Existing anchor | `engine/training_snapshot_contract.py` | `TrainingSnapshotSpec` |
| Existing anchor | `engine/research_manifest.py` | `ModelArtifactRef` |

## 1. Goal

Define training snapshot, model artifact and prediction artifact metadata linkage without writing model registries, stores, catalog pointers or artifact binaries.

## 2. Requirements（Functional / Non-Functional）

### Functional

- Extend or compose `TrainingSnapshotSpec` with ML PIT/label/split fingerprints.
- Extend or compose `ModelArtifactRef` metadata with training snapshot ref, config hash, feature schema hash and limitations.
- Define prediction artifact metadata as ref/hash/linkage only.
- Add forbidden operation counters for registry/store/catalog mutations.

### Non-Functional

- Metadata-only and local/static.
- No real training, artifact persistence, upload, publish, registry write or store write.
- Preserve existing source anchors and avoid parallel manifest family.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/training_snapshot_contract.py` | Training snapshot metadata linkage or companion fields | S03 must not break CR139 validation. |
| `engine/research_manifest.py` | Model artifact metadata companion around `ModelArtifactRef` | Keep model ref as anchor. |
| `engine/research_production_contracts.py` | Read-only compatibility inspection only by default | S01/S02 own this shared contract file. S03 must not write it unless CP6 stops and records CP5 / merge-owner confirmation. |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Metadata and no-registry-write tests | Shared with S01/S02/S04. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 修改 | `engine/training_snapshot_contract.py` | Add ML companion metadata or helper validation around existing snapshot spec. |
| 修改 | `engine/research_manifest.py` | Add model/prediction metadata companion around `ModelArtifactRef`. |
| 修改 | `tests/research/test_ml_strategy_e2e_contracts.py` | Add metadata serialization and forbidden operation tests. |

## 5. 数据模型与持久化设计

No persistence or registry write is introduced.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `MLTrainingSnapshotMetadata` | dataclass | `training_snapshot_id`, `pit_feature_matrix_id`, `label_policy_id`, `cv_policy_id`, `feature_schema_hash`, `lineage_refs` | Companion to `TrainingSnapshotSpec`. |
| `MLModelArtifactMetadata` | dataclass | `model_id`, `model_artifact_ref`, `training_snapshot_ref`, `config_hash`, `feature_schema_hash`, `limitations` | Companion to `ModelArtifactRef`. |
| `MLPredictionArtifactMetadata` | dataclass | `prediction_id`, `model_id`, `prediction_ref`, `prediction_hash`, `prediction_timestamp_field`, `lineage_refs` | Metadata only, no prediction store write. |
| `MLArtifactOperationCounters` | mapping | registry/store/catalog fields must be zero | Guardrail consumed by S04. |

Forbidden counters must include at least: `model_registry_write`, `model_registry_publish`, `model_registry_promote`, `artifact_upload`, `catalog_pointer_mutation`, `feature_store_write`, `label_store_write`, `prediction_store_write`.

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `validate_ml_training_snapshot_metadata(meta)` | mapping/dataclass | tuple issues | S04/tests | Required refs and hashes. |
| `validate_ml_model_artifact_metadata(meta)` | mapping/dataclass | tuple issues | S04/tests | Anchors to `ModelArtifactRef`. |
| `validate_ml_prediction_artifact_metadata(meta)` | mapping/dataclass | tuple issues | S04/tests | Prediction metadata only. |
| `validate_ml_artifact_operation_counters(counters)` | mapping | tuple issues | S04/tests | Nonzero write/publish/store counters -> `BLOCKED`. |

## 7. 核心处理流程

1. S01/S02 produce feature/label/CV metadata identifiers.
2. Caller provides existing `TrainingSnapshotSpec` / `ModelArtifactRef` values or mappings.
3. S03 companion metadata links snapshot/model/prediction refs.
4. Validators collect missing-field or forbidden-operation issues.
5. S04 aggregates issues into ML gate status.

## 8. 技术设计细节

- Do not add functions named or behaving as `register`, `publish`, `promote`, `upload`, `set_current`, `sync`, `persist` or `mutate` for model/store/catalog state.
- If CP6 needs a new module, it must be a metadata companion module, not a registry writer.
- Existing `ModelArtifactRef.artifact_path` remains a reference string; it does not mean the artifact was written during CR152.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| Registry safety | Explicit forbidden counters and no writer APIs. | Unit tests with nonzero counters. |
| Runtime safety | No train/predict execution. | Import/static tests only. |
| Compatibility | Existing anchors stay source of truth. | Existing training snapshot tests remain targeted regression candidates. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| Training metadata links snapshot | valid refs/hashes | validate | no issues | pytest |
| Model metadata missing hash | missing artifact hash | validate | `BLOCKED` issue | pytest |
| Prediction metadata serializes | valid prediction ref | to_dict / validate | JSON-safe metadata | pytest |
| Registry write counter nonzero | counter=1 | validate counters | `BLOCKED` issue | pytest |
| Catalog pointer mutation nonzero | counter=1 | validate counters | `BLOCKED` issue | pytest |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR152-S03-T01 | 修改 | `engine/training_snapshot_contract.py` | Add ML training metadata companion / validator. | snapshot metadata tests |
| CR152-S03-T02 | 修改 | `engine/research_manifest.py` | Add model and prediction metadata companions / validators. | artifact metadata tests |
| CR152-S03-T03 | 修改 | `tests/research/test_ml_strategy_e2e_contracts.py` | Add no-registry-write tests. | forbidden counter tests |
| CR152-S03-T04 | 只读检查 | `engine/research_production_contracts.py` | Inspect only for compatibility with S01/S02 anchors. If implementation needs to modify this file, stop and confirm merge-owner / CP5 queue before editing. | owner-boundary evidence |

## 12. 风险、难点与预研建议

| 风险 / 难点 | 影响 | 缓解措施 |
|---|---|---|
| Metadata helper drifts into writer | Scope violation | Explicit forbidden verbs and counters in LLD / tests. |
| Existing anchors require incompatible fields | Parallel contract temptation | Use narrow companion object; do not fork anchor semantics. |
| Optional bridge expands into shared contract write | File owner conflict with S01/S02 | Prefer `engine/training_snapshot_contract.py` / `engine/research_manifest.py`. Treat `engine/research_production_contracts.py` as read-only for S03 unless CP5 / merge-owner confirmation is recorded. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

Rollback removes companion metadata and tests. No registry/store/catalog cleanup is needed because no write path is introduced.

## 14. Definition of Done

- [ ] Training metadata links existing snapshot anchor.
- [ ] Model metadata links existing `ModelArtifactRef` anchor.
- [ ] Prediction metadata is metadata-only.
- [ ] Forbidden registry/store/catalog counters block.
- [ ] No registry writer or artifact persistence API is introduced.

## 人工确认区

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 AC | pending | 第 2 / 10 / 14 节 |
| 2 | No-registry-write 边界明确 | pending | 第 5 / 8 / 10 节 |
| 3 | Existing anchors preserved | pending | 第 0 / 3 / 5 节 |

**人工审查结果回填**：

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
