---
checkpoint_id: "CP3-CR152-ML-STRATEGY-E2E-HLD"
checkpoint_name: "CR152 ML Strategy E2E Framework Architecture Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-02T10:07:16+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-02T10:25:41+08:00"
auto_check_result: "process/checks/CP3-CR152-ML-STRATEGY-E2E-HLD-CONSISTENCY.result.json"
target:
  phase: "solution-design"
  cr_id: "CR-152"
  artifacts:
    - "process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md"
    - "process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md"
    - "process/discussions/CP3-CR152-HLD-DISCUSSION-LOG.md"
    - "process/context/CP3-CR152-ML-STRATEGY-E2E-HLD-CONTEXT.yaml"
---

# CP3 CR152 ML Strategy E2E Framework 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR152-ML-STRATEGY-E2E-HLD-CONSISTENCY.result.json` | PASS | 0 | HLD、ADR、Architecture Gray Areas、contract convergence、metadata vs registry boundary 和不授权边界均已就绪。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR152 的 ML Strategy E2E Framework 架构，让后续 Story planning 和 CP5 设计只围绕本地/static/fixture ML first-wave 契约、ML-specific gate 和 adapter 展开。 |
| 推荐动作 | `approve`：批准 metadata-only ML contract extension、ML-specific admission gate + CR151 四态 adapter、现有 contract 扩展策略和 deterministic fixture-only validation boundary。 |
| approve 后会发生什么 | 进入 CP4 / Story planning：可拆分 CR152 Story、准备 Feature Design Matrix / Story backlog / Development Plan；之后仍需 CP5 全量设计证据确认，才允许源码实现。 |
| approve 不授权什么 | 不授权源码实现；不授权真实训练、真实数据验证、model registry write/publish、lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/store write。 |
| 不确认会阻塞什么 | 阻塞 CR152 Story planning、LLD、实现、验证和 release readiness。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR152-ML-STRATEGY-E2E-HLD-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 CR152 summary、CR152 正文、CP2 approval、CP3 HLD/ADR/discussion log。 |
| 全文档读取扩展 | 1 次：为确认 contract anchors，读取 `engine/research_production_contracts.py`、`engine/training_snapshot_contract.py`、`engine/research_manifest.py`、`engine/strategy_admission_statistical_gate.py`、`engine/strategy_admission_package.py` 片段。 |
| 缺失 / waived 理由 | N/A。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.current.json.human_gate_decisions` | scanned | 0 | 0 | CP3 前无未决人工队列。 |
| CP2 approved decisions | `process/checkpoints/CP2-CR152-ML-STRATEGY-E2E-SCOPE.md` | scanned | 7 | 0 | 已由用户 approve；CP3 不重复收集。 |
| User CP3 focus items | 当前对话 / CP3 context capsule | scanned | 3 | 3 | 全部转为 HLD/ADR/CP3 check 项。 |
| Architecture Gray Areas | `process/discussions/CP3-CR152-HLD-DISCUSSION-LOG.md` | scanned | 5 | 4 | AGA-04 已由 CP2 label extensibility decision 解决；其余进入 CP3 DQ。 |
| HLD | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | scanned | 4 | 4 | 与 ADR 决策一致。 |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | scanned | 4 | 4 | ADR-CR152-001..004。 |
| CP3 auto result | `process/checks/CP3-CR152-ML-STRATEGY-E2E-HLD-CONSISTENCY.result.json` | scanned | 4 | 4 | Pending human decisions included in result JSON. |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 0 | 本 CP3 不授权任何高风险运行面或 registry write。 |
| agent 默认处理 | 4 | Story ID、fixture 文件名、具体 dataclass 字段命名、adapter helper 命名可在 CP4/CP5 默认处理并留证据。 |
| 仅审计记录 | 3 | CP2 discussion N/A 合理性、CR151 status reuse rationale、deferred data-lake candidates 继续 deferred。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR152-001 | architecture | 是否批准 ML-specific admission gate + CR151 四态 status / admission package adapter？ | 定义 `MLAdmissionGateSummary` 和 adapter，复用 `PASS / FAIL / NEEDS_REVIEW / BLOCKED` 语义，并记录 admission package linkage。 | A. 直接复用 CR151 `StrategyAdmissionStatisticalGate`；B. 完全独立 ML status model；C. 暂不做 ML gate。 | 推荐方案保留 ML 证据语义并保持跨策略 governance 一致；A 会污染 CR151 统计报告语义；B 造成状态割裂；C 无法闭环 UC-59。 | 影响 CP4 Story split、CP5 adapter design、CP7/CP8 wording。 | 若 CP5 证明 adapter 过大，可先实现 thin mapping，但不得丢四态语义。 |
| DQ-CP3-CR152-002 | security | 是否批准 model artifact metadata 与 model registry write 的边界？ | First-wave 只允许 `TrainingSnapshotSpec` / `ModelArtifactRef` metadata/ref/hash/linkage；禁止 registry write、publish、promote、upload、set_current、catalog pointer mutation 和 store writes。 | A. 加入 local registry writer；B. 集成外部 registry framework；C. 允许 catalog pointer mutation 但不写模型。 | 推荐方案符合 CP2 no-write；备选均触发存储/运行/依赖授权风险。 | 防止 CP5/CP6 将 metadata contract 误实现为 registry writer。 | 任何 registry/store/catalog 写入必须另起 runtime/storage authorization gate 或正式 CR。 |
| DQ-CP3-CR152-003 | architecture | 是否批准扩展 / 组合现有 contracts，而不是重建平行 ML contract family？ | 基于 `ResearchDatasetSpec`、`LeakagePolicy`、`TrainingSnapshotSpec`、`ModelArtifactRef`、CR151 status 和 `StrategyAdmissionPackage` 做 companion extension。 | A. 新建 `ml_contracts.py` 完整平行 family；B. 文档-only；C. 只扩展 admission package。 | 推荐方案满足 ML-1 contract convergence；A 增加迁移和重复语义；B 不能交付 foundation；C 缺 dataset/label/CV 输入链。 | 影响 CP5 文件影响范围和契约映射。 | 若某 anchor 有缺陷，优先修复或窄包装 anchor，不静默 fork 语义。 |
| DQ-CP3-CR152-004 | implementation | 是否批准 deterministic fixture-only validation boundary？ | 使用本地 deterministic fixtures，覆盖 PIT、leakage、purged/embargo 和 artifact metadata 正/负案例；所有 forbidden operation counters 为 0。 | A. Tiny schema-only fixture；B. 真实 lake validation；C. 外部框架生成 fixture。 | 推荐方案能验证核心语义且不越权；A 容易空壳；B/C 越权。 | 影响 CP5 test plan、CP6 tests、CP7 evidence。 | 如后续需真实数据验证，另起数据/runtime gate；本轮 fixture 不能宣称真实模型性能。 |

### CP3 追加字段

| 字段 | 内容 |
|---|---|
| 候选架构适用条件 | 适用于 ML strategy E2E first-wave 本地/static/fixture 契约基础；不适用于真实训练、真实数据验证、registry publish 或 runtime readiness。 |
| 优化项 | 契约收敛、no-registry-write 边界明确、ML gate 与 CR151 status 语义统一、fixture 可验证。 |
| 牺牲项 | 不一次性实现 triple-barrier 算法、meta-label training、feature importance、sample uniqueness、drift/retrain、真实 model registry。 |
| 影响面 | `engine/research_production_contracts.py`、`engine/training_snapshot_contract.py`、`engine/research_manifest.py`、未来 ML gate companion module、tests、process evidence。 |
| 切换条件 | 若用户改为生产级模型注册或真实训练，必须重开 scope 和 runtime/storage authorization。 |
| Use Case -> Architecture Traceability | HLD §10 覆盖 UC-59、ML-GAP-1、ML-GAP-2、ML-GAP-3、ML-GAP-12 和 CR151 gate relation。 |
| 关键场景模拟 | HLD §9：fixture assembly、fail-closed leakage、purged/embargo overlap、registry write counter nonzero。 |
| 未决风险 | 无阻断；fixture-only 不能宣称真实模型性能；registry write 仍未授权。 |
| discussion log / checkpoint | `process/discussions/CP3-CR152-HLD-DISCUSSION-LOG.md` / `process/checks/CP3-CR152-DISCUSSION-CHECKPOINT.json` |

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 4 |
| 必须用户决策 | `DQ-CP3-CR152-001` architecture、`DQ-CP3-CR152-002` security、`DQ-CP3-CR152-003` architecture、`DQ-CP3-CR152-004` implementation |
| 推荐回复 | `approve` |
| 如果你回复 approve | 表示接受上表 4 项推荐方案，并允许进入 CP4 Story planning。 |
| 不表示授权 | 不表示授权源码实现、真实训练、真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/model registry/store write。 |
| 修改: <具体修改点> | 用户可指定 DQ ID 和修改内容。 |
| reject | 用户可拒绝 CP3，CR152 停留在 solution-design。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 approved | pending_user_review | `process/checkpoints/CP2-CR152-ML-STRATEGY-E2E-SCOPE.md` | 用户已于 2026-07-02T09:59:41+08:00 approve CP2。 |
| HLD draft exists | approved | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | 用户批准。 |
| ADR draft exists | approved | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | 用户批准 ADR-CR152-001..004 推荐方案。 |
| CP3 discussion evidence exists | approved | `process/discussions/CP3-CR152-HLD-DISCUSSION-LOG.md` | Architecture Gray Areas 与 advisor lane 证据已审查。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | HLD 是否覆盖问题定义、目标、候选方案、推荐方案、风险和回退 | approved | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | 通过。 |
| 2 | Architecture Gray Areas 是否前置并影响方案 | approved | `process/discussions/CP3-CR152-HLD-DISCUSSION-LOG.md` | 通过。 |
| 3 | ADR 是否有推荐、备选、优劣、影响和切换条件 | approved | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | 通过。 |
| 4 | 是否明确 metadata contract vs model registry write 边界 | approved | HLD §8、ADR-CR152-002 | 通过；CP5/CP6 仍不得实现 registry writer。 |
| 5 | 是否复用 / 扩展现有 ResearchDatasetSpec / TrainingSnapshotSpec / ModelArtifactRef | approved | HLD §6、ADR-CR152-003 | 通过。 |
| 6 | 是否保持 no-runtime / no-credential / no-lake / no-NAS / no-registry-write 边界 | approved | HLD §12、ADR Not Authorized | 通过。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 自动预检通过 | PASS | `process/checks/CP3-CR152-ML-STRATEGY-E2E-HLD-CONSISTENCY.result.json` | PASS / 0 blockers。 |
| 4 项 DQ 完成确认 | approved | 本文件 Decision Brief | 用户批准 DQ-CP3-CR152-001..004 推荐方案。 |
| 可进入 CP4 Story planning | approved | 本文件人工审查结果 | 允许进入 CP4 Story planning；仍需 CP5 后才允许源码实现。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 context capsule | `process/context/CP3-CR152-ML-STRATEGY-E2E-HLD-CONTEXT.yaml` | approved |  |
| CP3 result JSON | `process/checks/CP3-CR152-ML-STRATEGY-E2E-HLD-CONSISTENCY.result.json` | approved |  |
| HLD | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | approved |  |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | approved |  |
| Discussion log | `process/discussions/CP3-CR152-HLD-DISCUSSION-LOG.md` | approved |  |
| Discussion checkpoint | `process/checks/CP3-CR152-DISCUSSION-CHECKPOINT.json` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-02T10:25:41+08:00
- 修改意见：
  - CP5 LLD 必须消除 HLD §9 Flow 2 中 `triple_barrier` active method 的语义模糊；推荐 first wave 选中 `triple_barrier` 直接 `BLOCKED`，因为算法未实现。
  - CP4 必须产出 Feature Design Matrix，并确认每个 Story 的 `lld_policy.required_level`。
  - CP5 若需 adapter 实现细节或 contract delta 精确字段映射的架构深化，应考虑调度 meta-se 子 agent，或显式记录 inline-fallback 批准。
- 风险接受项：N/A
- 已接受决策项：`DQ-CP3-CR152-001`、`DQ-CP3-CR152-002`、`DQ-CP3-CR152-003`、`DQ-CP3-CR152-004`
