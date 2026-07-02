请审查：`process/checkpoints/CP3-CR152-ML-STRATEGY-E2E-HLD-REVIEW.md`

自动预检结论：PASS

Context Capsule：`process/context/CP3-CR152-ML-STRATEGY-E2E-HLD-CONTEXT.yaml`（read_profile=compact，完整来源见 checklist）

审批者摘要：
- 本次确认服务的整体目标：确认 CR152 的 ML Strategy E2E Framework 架构，让后续 Story planning 和 CP5 设计只围绕本地/static/fixture ML first-wave 契约、ML-specific gate 和 adapter 展开。
- 推荐动作：`approve`，批准 metadata-only ML contract extension、ML-specific admission gate + CR151 四态 adapter、现有 contract 扩展策略和 deterministic fixture-only validation boundary。
- approve 后会发生什么：进入 CP4 / Story planning；之后仍需 CP5 全量设计证据确认，才允许源码实现。
- approve 不授权什么：不授权源码实现；不授权真实训练、真实数据验证、model registry write/publish、lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/store write。
- 不确认会阻塞什么：阻塞 CR152 Story planning、LLD、实现、验证和 release readiness。

本轮待人工决策项：4
本轮待人工决策项: 4

决策收集覆盖：已扫描 7 个来源，发现候选问题 27 个，纳入待决策 4 个；N/A / 缺失来源 0 个。

决策分层：
- 必须用户决策：4
- 高风险策略确认：0
- agent 默认处理：4
- 仅审计记录：3

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR152-001 | architecture | 是否批准 ML-specific admission gate + CR151 四态 status / admission package adapter？ | 定义 `MLAdmissionGateSummary` 和 adapter，复用 `PASS / FAIL / NEEDS_REVIEW / BLOCKED` 语义，并记录 admission package linkage。 | A. 直接复用 CR151 gate；B. 完全独立 ML status model；C. 暂不做 ML gate。 | 推荐方案保留 ML 证据语义并保持跨策略 governance 一致；备选会污染 CR151、状态割裂或无法闭环 UC-59。 | 影响 CP4 Story split、CP5 adapter design、CP7/CP8 wording。 |
| DQ-CP3-CR152-002 | security | 是否批准 model artifact metadata 与 model registry write 的边界？ | First-wave 只允许 `TrainingSnapshotSpec` / `ModelArtifactRef` metadata/ref/hash/linkage；禁止 registry write、publish、promote、upload、set_current、catalog pointer mutation 和 store writes。 | A. 加入 local registry writer；B. 集成外部 registry framework；C. 允许 catalog pointer mutation。 | 推荐方案符合 CP2 no-write；备选均触发存储/运行/依赖授权风险。 | 防止 CP5/CP6 将 metadata contract 误实现为 registry writer。 |
| DQ-CP3-CR152-003 | architecture | 是否批准扩展 / 组合现有 contracts，而不是重建平行 ML contract family？ | 基于 `ResearchDatasetSpec`、`LeakagePolicy`、`TrainingSnapshotSpec`、`ModelArtifactRef`、CR151 status 和 `StrategyAdmissionPackage` 做 companion extension。 | A. 新建完整平行 family；B. 文档-only；C. 只扩展 admission package。 | 推荐方案满足 ML-1 contract convergence；备选会增加迁移、无法交付或缺输入链。 | 影响 CP5 文件影响范围和契约映射。 |
| DQ-CP3-CR152-004 | implementation | 是否批准 deterministic fixture-only validation boundary？ | 使用本地 deterministic fixtures，覆盖 PIT、leakage、purged/embargo 和 artifact metadata 正/负案例；所有 forbidden operation counters 为 0。 | A. Tiny schema-only fixture；B. 真实 lake validation；C. 外部框架生成 fixture。 | 推荐方案能验证核心语义且不越权；备选容易空壳或越权。 | 影响 CP5 test plan、CP6 tests、CP7 evidence。 |

不授权项：
- source implementation before CP5 approval
- real model training or real data validation
- model registry write / publish / promote / upload / set_current
- feature store / label store / prediction store writes
- real lake read or write
- NAS read/write/sync/chmod/chgrp/metadata normalization
- provider fetch
- QMT/MiniQMT/xtquant runtime
- simulation/paper/live/trading runtime
- broker read/write/submit/cancel/account query
- credential or `.env` read
- external framework clone/install/run
- Git remote write
- catalog pointer mutation

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

如果你回复 approve，表示接受上表全部推荐方案，并允许进入 CP4 Story planning。

不表示授权：源码实现、真实训练、真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/model registry/store write 操作。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

可回复：

approve

修改: <具体修改点>

reject
