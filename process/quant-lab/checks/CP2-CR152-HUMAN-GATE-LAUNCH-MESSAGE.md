CR152 CP2 范围门禁已准备好。

审批者摘要：

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR152 Machine Learning Strategy E2E Framework Foundation 的范围，让后续设计只围绕本地/static/fixture ML 契约和 admission gate 展开。 |
| 推荐动作 | `approve`：批准 CR152 Scope Baseline、first-wave 范围、no-runtime 安全边界、ML gate 关系、statistical gate opt-in 语义、label 扩展位和 fixture data contract。 |
| approve 后会发生什么 | CR152 进入 CP3 设计；下一步会设计 ML-specific admission gate、PIT feature matrix、label policy、purged + embargo CV、training/model/prediction artifact metadata 和 fixture contract。 |
| approve 不授权什么 | 不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/feature-store/label-store/model-registry/prediction-store 写入；不授权真实训练、真实数据验证或源码实现。 |
| 不确认会阻塞什么 | 阻塞 CR152 的 CP3 设计、Story 拆解、LLD、实现、验证和 release readiness。 |

自动预检结论：

| 文件 | 结论 | 阻断项 |
|---|---|---:|
| `process/checks/CP0-CR152-ML-STRATEGY-E2E-REQUEST-INTAKE.result.json` | PASS | 0 |
| `process/checks/CP1-CR152-ML-STRATEGY-E2E-USE-CASE-COMPLETENESS.result.json` | PASS | 0 |
| `process/checks/CP2-CR152-ML-STRATEGY-E2E-SCOPE.result.json` | PASS | 0 |

Context Capsule：

| 字段 | 内容 |
|---|---|
| Context Capsule | `process/context/CP2-CR152-ML-STRATEGY-E2E-SCOPE-CONTEXT.yaml` |
| read_profile | compact |
| 读取扩展 | `READ-CR152-CP2-ROADMAP-REMEDIATION-HANDOFF-2026-07-02T092827+0800` |

决策收集覆盖：

| 来源 | 扫描状态 | 纳入待决策数 |
|---|---|---:|
| STATE pending queue | scanned | 0 for CR152; existing RA-CR149 future metadata item archived as deferred |
| Handoff | scanned | 2 |
| Roadmap | scanned | 4 |
| Remediation plan | scanned | 6 |
| E2E review | scanned | 2 |
| CP0 / CP1 / CP2 auto results | scanned | 0 |
| 用户显式评审意见 | scanned | 4 |

决策分层：

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 7 | 进入本轮待人工决策项 |
| 高风险策略确认 | 0 | 本 CP2 不授权高风险运行面 |
| agent 默认处理 | 3 | CP2 通过后的命名、fixture 文件布局、低风险字段命名由 agent 在 CP3/CP5 留证据处理 |
| 仅审计记录 | 5 | CR151 closure、STATE v2 caveat、CR-INDEX warnings、deferred data-lake candidates、SSH fetch failure with HTTPS ls-remote confirmation |

本轮待人工决策项: 7

本轮待人工决策项数量：7

| 决策 ID | 类型 | 推荐方案 |
|---|---|---|
| `DQ-CP2-CR152-SCOPE` | scope | 批准 CR152 作为本地/static/fixture ML Strategy E2E Framework Foundation；CP2 后只进入 CP3 设计，不进入真实训练、真实数据验证或 runtime。 |
| `DQ-CP2-CR152-FIRST-WAVE` | implementation | first wave 覆盖 PIT feature matrix、label policy / leakage guard、purged + embargo CV、training snapshot / model artifact metadata、prediction artifact、ML admission gate。 |
| `DQ-CP2-CR152-SECURITY-AUTHZ` | security | 确认 CP2 不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/model registry/real training。 |
| `DQ-CP2-CR152-ML-GATE-RELATION` | architecture | 新建 ML-specific admission gate，并通过 adapter 复用 CR151 四态 status 语义和 admission package linkage。 |
| `DQ-CP2-CR152-STAT-GATE-OPT-IN` | implementation | CR152 先保持 statistical gate opt-in，但 admission package 必须显式记录 `gate_present` / `gate_required` / `gate_status`；默认强制留给 CR154 或后续 admission governance。 |
| `DQ-CP2-CR152-LABEL-POLICY-EXTENSIBILITY` | architecture | 预留 `fixed_window` / `triple_barrier` / `meta_label` 枚举位和 spec slot；本轮只实现 fixed-window / 静态校验，不实现 triple-barrier 算法或真实 meta-label training。 |
| `DQ-CP2-CR152-FIXTURE-DATA-CONTRACT` | implementation | 定义 deterministic fixture，支持 purged + embargo CV 的最小可验证语义，并包含 pass 与 fail-closed leakage / overlap case。 |

如果你回复 approve，表示接受以上 7 项推荐方案。

不表示授权 / 不授权项：

- 真实 lake read/write
- NAS read/write/sync/restore/chmod/chgrp/metadata normalization
- provider fetch
- QMT / MiniQMT / xtquant / gateway runtime
- simulation / paper / live / trading runtime
- broker read/write/submit/cancel/real account query
- credential 或 `.env` read
- external framework clone/install/run
- Git remote write
- catalog pointer mutation
- feature store / label store / model registry / prediction store writes
- real model training or real data validation

也可以回复 `修改: <具体修改点>` 调整范围、first wave、gate 关系、label 扩展位、fixture contract 或授权边界；也可以回复 `reject` 拒绝本 CP2。

Checkpoint: `process/checkpoints/CP2-CR152-ML-STRATEGY-E2E-SCOPE.md`
