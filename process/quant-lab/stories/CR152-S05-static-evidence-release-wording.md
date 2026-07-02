---
story_id: "CR152-S05-static-evidence-release-wording"
change_id: "CR-152"
title: "Static evidence and release wording"
status: "ready-for-verification"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
source_hld: "process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md"
lld_policy:
  required: true
  required_level: "technical-note"
  status: "approved"
depends_on:
  - "CR152-S01-pit-feature-label-contracts"
  - "CR152-S02-purged-embargo-cv-fixture-contract"
  - "CR152-S03-training-model-prediction-metadata"
  - "CR152-S04-ml-admission-gate-adapter"
implementation_allowed: true
authorization_boundary: "static/fixture evidence only; no real performance, registry publication, runtime readiness or trading readiness claims"
---

# CR152-S05 Static Evidence and Release Wording

## Goal

Close process evidence, CP7/CP8 wording and release boundary for CR152.

## Scope

- Return/evidence/release wording.
- Fixture-only claim boundary.
- Deferred later-wave list.
- No-real-training / no-real-data / no-registry-write release limitations.

## Acceptance Criteria

- Evidence states fixture-only contract semantics.
- Release wording does not claim real model performance, production readiness, registry publication, runtime readiness or trading readiness.
- Deferred later-wave items remain explicit.
- CP8 limitations include no real training, no real data validation and no registry/store/catalog writes.

## Technical Note Required For CP5

Story technical note covering wording sources, evidence refs, release limitations, deferred scope and no-real-operation assertions.

## 技术说明

### 设计依据

| 来源 | 路径 / ID | 本 Story 消费内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | Fixture-only claim boundary, deferred later-wave items |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | Metadata-only and no-registry-write boundary |
| CP4 Result | `process/checks/CP4-CR152-STORY-DAG-PARALLEL-SAFETY.result.json` | S05 technical-note policy and dependency order |
| Development Plan | `process/DEVELOPMENT-PLAN-CR152.yaml` | Evidence wording after S01-S04 design decisions stabilize |
| Feature refs | `docs/features/factor-research-loop/TEST-PLAN.md` / `TASKS.md` | CP7/CP8 wording alignment only; not a claim that S05 implements factor-research-loop functional behavior. |

### 文件影响

| 文件 / 路径 | 设计动作 | 说明 |
|---|---|---|
| `process/returns/CR152-*` | 规划 CP6/CP7 return packet wording | 只记录 fixture/static 证据和禁止项。 |
| `process/evidence/CR152-*` | 规划 evidence index wording | 不复制完整日志，不声明真实模型性能。 |
| `docs/release/*` | 后续 CP8 release wording input | 仅在 CP8 阶段按 release-readiness 规则生成或更新。 |
| `process/checks/CP7-*` / `process/checkpoints/CP8-*` | 后续验证 / 发布门禁文案 | 明确 no-real-training / no-registry-write / no-runtime。 |

### 接口 / 数据 / 权限变化

- 无新增源码接口。
- 无新增持久化数据结构。
- 无新增权限。
- 证据对象必须记录 `effective_validation_mode=static-fixture-only` 或等价字段。
- 证据对象不得包含 `real_model_performance=true`、`registry_published=true`、`runtime_ready=true`、`trading_ready=true` 等声明。

### 异常和回退

| 情况 | 处理 |
|---|---|
| CP6/CP7 evidence wording 暗示真实性能 | 回退 S05 technical-note 或 CP7 verification wording，阻断 CP8。 |
| 出现 registry/store/catalog write 证据 | BLOCKED，回退到 CP5/CP6，另起授权门禁。 |
| deferred later-wave items 被写成已完成 | 修正文案并在 CP8 risk / limitation 中保留。 |

### 测试入口

- CP7 verification 应检查 evidence index 中 no-real-operation / no-registry-write claims。
- CP8 release-readiness 应检查 release context 限制项。
- CP7 verification 应确认 S05 的 fixture/static wording 与 referenced Feature TEST-PLAN / TASKS 不冲突；若冲突，以 CR152 no-real-operation / no-registry-write 边界为准并回退文案。
- 若实现添加文案校验工具，targeted 命令应通过 `uv run --python 3.11 pytest -q ...` 或对应 `meta-flow` checker。

### 已知风险

| 风险 | 缓解 |
|---|---|
| Fixture-only 被误读为真实模型表现 | 所有 CP7/CP8 文案显式写“contract semantics only”。 |
| Metadata ref 被误读为 registry publish | Release context 明确 registry writer/publish/promote/upload/set_current 均未授权。 |
| Later-wave 被静默宣称完成 | Deferred list 必须跟随 release context。 |

### 偏离记录

本 Story 使用 `technical-note`，因为它只负责证据与发布文案，不新增核心数据模型或源码接口。若后续需要实现新的 release checker 或 schema validator，应升级为 full-lld 或新 Story。
