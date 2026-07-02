---
checkpoint_id: "CP2-CR152-ML-STRATEGY-E2E-SCOPE"
checkpoint_name: "CR152 Machine Learning Strategy E2E Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-02T09:28:27+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-02T09:59:41+08:00"
auto_check_result: "process/checks/CP2-CR152-ML-STRATEGY-E2E-SCOPE.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-152"
  artifacts:
    - "process/changes/CR-152.md"
    - "process/context/CP2-CR152-ML-STRATEGY-E2E-SCOPE-CONTEXT.yaml"
    - "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md"
    - "docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md"
---

# CP2 CR152 Machine Learning Strategy E2E Scope Baseline 人工审查

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR152 formal CR exists | PASS | `process/changes/CR-152.md` | `lifecycle_status=active`，`gate_status=cp2_pending`。 |
| CP0 request intake passed | PASS | `process/checks/CP0-CR152-ML-STRATEGY-E2E-REQUEST-INTAKE.result.json` | CP0 decision PASS。 |
| CP1 use-case completeness passed | PASS | `process/checks/CP1-CR152-ML-STRATEGY-E2E-USE-CASE-COMPLETENESS.result.json` | 基于既有 UC-59 / roadmap / remediation plan 增量通过。 |
| CR151 baseline closed | PASS | `process/checks/CP8-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-RELEASE-READINESS.result.json` | CR151 closed as READY_WITH_RISK；STATE v2 hygiene accepted as process caveat。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | CR152 是否只覆盖本地/static/fixture ML strategy foundation | PASS | `process/context/CP2-CR152-ML-STRATEGY-E2E-SCOPE-CONTEXT.yaml` | 建议 approve。 |
| 2 | CP1 是否显式执行 | PASS | `process/checks/CP1-CR152-ML-STRATEGY-E2E-USE-CASE-COMPLETENESS.result.json` | 建议 approve。 |
| 3 | first wave 六项 prerequisites 是否来自 roadmap / remediation plan | PASS | `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md#7-近期建议`、`docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md#54-CP2-必须决策项` | 建议 approve。 |
| 4 | ML gate 与 CR151 gate 关系是否进入待决策项 | PASS | `DQ-CP2-CR152-ML-GATE-RELATION` | 建议 approve 新建 ML-specific gate + adapter。 |
| 5 | statistical gate opt-in / 强制时机是否进入待决策项 | PASS | `DQ-CP2-CR152-STAT-GATE-OPT-IN` | 建议 CR152 继续 opt-in，强制默认留给 CR154 / governance。 |
| 6 | label policy 是否预留 triple_barrier / meta_label 扩展位 | PASS | `DQ-CP2-CR152-LABEL-POLICY-EXTENSIBILITY` | 建议预留枚举和 spec slot，不实现算法。 |
| 7 | fixture data contract 是否足以支持 purged + embargo CV 验证 | PASS | `DQ-CP2-CR152-FIXTURE-DATA-CONTRACT` | 建议作为 first-wave 隐含 prerequisite 显式确认。 |
| 8 | 是否没有授权真实 lake/NAS/provider/QMT/runtime/trading/broker/credential/external framework | PASS | `process/changes/CR-152.md#不授权范围` | 建议 approve。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP2 auto precheck PASS | PASS | `process/checks/CP2-CR152-ML-STRATEGY-E2E-SCOPE.result.json` | 可发起人工确认。 |
| Pending decisions collected | PASS | 下方 Decision Brief | 7 项待用户确认。 |
| Context capsule ready | PASS | `process/context/CP2-CR152-ML-STRATEGY-E2E-SCOPE-CONTEXT.yaml` | read_profile=compact。 |
| Unauthorized scope explicit | PASS | 下方 Decision Brief | approve 不授权任何 runtime、真实数据或外部操作。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| CR152 formal CR | `process/changes/CR-152.md` | PASS | 已创建并记录五维影响分析。 |
| CR152 summary | `process/changes/summaries/CR-152.summary.json` | PASS | state-router / cr-tracking 轻量入口。 |
| CP0 result | `process/checks/CP0-CR152-ML-STRATEGY-E2E-REQUEST-INTAKE.result.json` | PASS | request intake。 |
| CP1 result | `process/checks/CP1-CR152-ML-STRATEGY-E2E-USE-CASE-COMPLETENESS.result.json` | PASS | 增量 use-case completeness。 |
| CP2 context capsule | `process/context/CP2-CR152-ML-STRATEGY-E2E-SCOPE-CONTEXT.yaml` | PASS | compact profile。 |
| CP2 result JSON | `process/checks/CP2-CR152-ML-STRATEGY-E2E-SCOPE.result.json` | PASS | 自动预检。 |
| CP2 human checkpoint | `process/checkpoints/CP2-CR152-ML-STRATEGY-E2E-SCOPE.md` | PASS | 本文件。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR152 Machine Learning Strategy E2E Framework Foundation 的范围，让后续设计只围绕本地/static/fixture ML 契约和 admission gate 展开。 |
| 推荐动作 | `approve`：批准 CR152 Scope Baseline、first-wave 范围、no-runtime 安全边界、ML gate 关系、statistical gate opt-in 语义、label 扩展位和 fixture data contract。 |
| approve 后会发生什么 | CR152 进入 CP3 设计；下一步会设计 ML-specific admission gate、PIT feature matrix、label policy、purged + embargo CV、training/model/prediction artifact metadata 和 fixture contract。 |
| approve 不授权什么 | 不授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/feature-store/label-store/model-registry/prediction-store 写入；不授权真实训练、真实数据验证或源码实现。 |
| 不确认会阻塞什么 | 阻塞 CR152 的 CP3 设计、Story 拆解、LLD、实现、验证和 release readiness。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR152-ML-STRATEGY-E2E-SCOPE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 CR152 summary、CR152 正文、roadmap、remediation plan。 |
| 全文档读取扩展 | 1 次；用户明确要求读取 handoff、STATE、roadmap、remediation plan、E2E review、CR151 closure evidence。 |
| 缺失 / waived 理由 | N/A。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.current.json.human_gate_decisions` | scanned | 1 | 0 | Existing RA-CR149 future NAS metadata decision is archived as deferred and outside CR152 active scope. |
| Handoff | `process/handoffs/NEXT-SESSION-CR152-ML-STRATEGY-E2E-CP0-CP2-2026-07-02.md` | scanned | 2 | 2 | ML gate relation and statistical gate opt-in decisions. |
| Roadmap | `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` | scanned | 4 | 4 | CR152 next route, first wave, auth boundary, deferred lake candidates. |
| Remediation plan | `docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md` | scanned | 6 | 6 | ML-1..ML-8, CP2 must decisions, first wave, hygiene. |
| E2E review | `docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` | scanned | 3 | 2 | ML-GAP-3 label extensibility and ML-GAP-1 purged CV fixture need included; outdated ML-GAP-4 fact corrected by remediation plan. |
| CP0 / CP1 / CP2 auto results | `process/checks/CP0-*`, `process/checks/CP1-*`, `process/checks/CP2-*` | scanned | 0 | 0 | PASS; no blockers. |
| 用户显式评审意见 | 当前对话 | scanned | 6 | 4 | CP1, STATE refresh, human-gate check are process actions; label extensibility and fixture contract become CP2 decisions. |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 7 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 0 | 本 CP2 不授权高风险运行面；全部列为不授权。 |
| agent 默认处理 | 3 | CP2 通过后的命名、fixture 文件布局、低风险字段命名由 agent 在 CP3/CP5 留证据处理。 |
| 仅审计记录 | 5 | CR151 closure、STATE v2 hygiene caveat、CR-INDEX warnings、deferred data-lake candidates、SSH fetch failure with HTTPS ls-remote confirmation。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR152-SCOPE | scope | 是否批准 CR152 作为本地/static/fixture ML Strategy E2E Framework Foundation 的正式范围？ | 批准 CR152；CP2 后只进入 CP3 设计，不进入真实训练、真实数据验证或 runtime。 | A. 缩小为只做 purged CV；B. 扩大到真实训练；C. 暂停 CR152 回到数据湖治理。 | 推荐方案与 roadmap 路线 A 一致且范围可控；A 无法形成 E2E foundation；B 触发高风险授权；C 延迟策略框架补齐。 | 影响 CR152 后续 CP3/CP5/CP6/CP7/CP8；不影响 deferred lake candidates。 | 若 CP3 发现范围过大，可拆分到 CR153/CR154 或后续 ML wave。 |
| DQ-CP2-CR152-FIRST-WAVE | implementation | CR152 first wave 应覆盖哪些 hard prerequisites？ | 覆盖 PIT feature matrix、label policy / leakage guard、purged + embargo CV、training snapshot / model artifact metadata、prediction artifact、ML admission gate。 | A. 只做 contract inventory；B. 加入 feature importance / sample weighting / drift；C. 只做 admission gate。 | 推荐方案来自 roadmap/remediation plan；A 过窄；B 增大首轮复杂度；C 缺少 admission 输入链。 | 决定 CP3 架构和 CP5 Story 边界。 | 若 CP3 发现 first wave 仍过大，可拆分为 ML-1/ML-2 子波次。 |
| DQ-CP2-CR152-SECURITY-AUTHZ | security | CP2 是否保持 local/static/fixture-only，不授权真实外部操作？ | 保持全部不授权：真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog/model registry 等均不触发。 | A. 授权真实 lake read；B. 授权真实训练；C. 授权外部框架安装运行。 | 推荐方案风险最低且符合当前目标；备选均需单独 runtime/data/security gate。 | 避免把 ML foundation 设计混入运行时风险。 | 如后续需要真实训练或真实数据验证，另起 runtime_authorization 或正式 CR。 |
| DQ-CP2-CR152-ML-GATE-RELATION | architecture | ML admission gate 与 CR151 `StrategyAdmissionStatisticalGate` 的关系是什么？ | 新建 ML-specific admission gate，并通过 adapter 复用 CR151 四态 status 语义和 admission package linkage。 | A. 直接扩展 CR151 gate 承载 ML report；B. 完全独立新 gate，不复用状态语义；C. 暂不做 ML admission gate。 | 推荐方案避免把 ML 专项方法塞进多因子 gate，同时保持三类策略 admission 状态一致；A 会污染 CR151 模型；B 造成状态割裂；C 无法闭环 E2E。 | 影响后续 admission package、completion map 和 CR154 governance。 | 若 CP3 发现 adapter 复杂度过高，可先定义 status mapping contract，延后 package linkage。 |
| DQ-CP2-CR152-STAT-GATE-OPT-IN | implementation | UC-58 / UC-59 statistical gate 何时从 opt-in 变成默认或强制？ | CR152 先保持 opt-in，但 admission package 必须显式记录 `gate_present` / `gate_required` / `gate_status`；默认强制留给 CR154 或后续 admission governance。 | A. CR152 直接强制 gate；B. 继续允许缺失 gate 且不记录；C. 只对 ML 强制、UC-58 不变。 | 推荐方案避免破坏历史行为，又防止新框架隐藏 gate 状态；A 风险高；B 可审计性差；C 容易造成跨策略不一致。 | 影响 UC-58/UC-59 调用方兼容性。 | 若 CR154 批准治理策略，可切换为默认或强制。 |
| DQ-CP2-CR152-LABEL-POLICY-EXTENSIBILITY | architecture | first-wave label policy 是否预留 triple_barrier / meta_label 扩展位？ | 预留 `fixed_window` / `triple_barrier` / `meta_label` 枚举位和 spec slot；本轮只实现 fixed-window / 静态校验，不实现 triple-barrier 算法或真实 meta-label training。 | A. 只支持 fixed_window；B. 本轮完整实现 triple_barrier 和 meta_label；C. 后续另建平行 label contract。 | 推荐方案防止未来 ML-3 改已验证契约或平行实现；A 会固化 ML-GAP-3；B 扩大范围；C 违背 contract convergence。 | 影响 label contract 长期兼容性和 CP5 设计边界。 | 若 CP3 发现字段设计不成熟，可保留 extension slot 但降低字段强校验。 |
| DQ-CP2-CR152-FIXTURE-DATA-CONTRACT | implementation | local/fixture-only 模式下，fixture 需要满足什么最小合同？ | 定义 deterministic fixture：至少覆盖 entity/date 主键、decision_time、feature_available_at、label window、fold boundary、purge window、embargo gap、prediction timestamp，并包含可通过和 fail-closed leakage / overlap case；CP3/CP5 再精确定量。 | A. 不定义 fixture contract；B. 要求真实 lake 样本；C. 本轮只做 schema 不做可切 fold fixture。 | 推荐方案保证 purged + embargo CV 可验证且不触碰真实数据；A 容易空壳；B 越权；C 不足以验证 split policy。 | 影响 CP6/CP7 验证可信度。 | 若 CP5 设计发现 fixture 太大，可保留最小 2-fold 验证并把规模扩展后置。 |

### CP2 追加字段

| 项 | 内容 |
|---|---|
| 用户真实意图 | 先补齐本地 ML 策略框架基础，不启动真实训练或真实运行。 |
| 场景覆盖 | UC-59 ML strategy E2E 的 first-wave foundation：PIT feature matrix、label、CV、snapshot/model/prediction artifacts、admission gate。 |
| 认知盲区补充 | label policy 需预留 triple_barrier / meta_label；fixture 必须足以验证 purged + embargo CV。 |
| Scenario Gray Areas | SGQ-CR152-001：ML gate 与 CR151 gate 关系；SGQ-CR152-002：statistical gate opt-in / mandatory timing；SGQ-CR152-003：label extensibility；SGQ-CR152-004：fixture contract。 |
| Deferred Ideas | feature importance、sample uniqueness、drift/retrain、fractional differentiation、class imbalance、MinBTL/MinTRL、real training、model registry、paper/live shadow。 |
| 用户选择影响 | approve 后进入 CP3 设计；不会启动实现或真实外部操作。 |
| 回退方式 | 修改范围后重写 CP2；reject 则 CR152 保持 cp2_pending / cancelled，不进入 CP3。 |
| discussion log / checkpoint | 本轮由用户评审意见直接形成 CP2 Decision Brief；无独立多轮 SGQ log，记录为 N/A with reason。 |

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 7 |
| 必须用户决策 | `DQ-CP2-CR152-SCOPE`、`DQ-CP2-CR152-FIRST-WAVE`、`DQ-CP2-CR152-SECURITY-AUTHZ`、`DQ-CP2-CR152-ML-GATE-RELATION`、`DQ-CP2-CR152-STAT-GATE-OPT-IN`、`DQ-CP2-CR152-LABEL-POLICY-EXTENSIBILITY`、`DQ-CP2-CR152-FIXTURE-DATA-CONTRACT` |
| 推荐回复 | `approve` |
| 如果你回复 approve | 表示接受上表 7 项推荐方案。 |
| 不表示授权 | 不表示授权真实 lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote/catalog pointer/model registry/real training/real data validation。 |
| 修改: <具体修改点> | 用户可回复具体修改范围、first wave、gate 关系、label 扩展位、fixture contract 或授权边界。 |
| reject | 用户可拒绝本 CP2，CR152 将不进入 CP3。 |

### CP3 设计关注点

| 关注点 ID | 来源 | CP3 必须处理的问题 | 推荐处理 |
|---|---|---|---|
| CP3-FOCUS-CR152-001 | 用户 CP2 approval feedback | `model artifact metadata` 与 `model registry write` 的代码层边界必须清楚。first-wave 只做 training snapshot / model artifact metadata 契约和数据结构，不授权 model registry writes。 | CP3 HLD / ADR 明确 metadata contract 是本地/static artifact description；registry writer / publish / pointer mutation / external registry integration 全部 out of scope，需要单独 runtime / registry authorization gate。 |
| CP3-FOCUS-CR152-002 | 用户 CP2 approval feedback | `TrainingSnapshotSpec` / `ModelArtifactRef` 应扩展现有契约，不得重建平行契约。 | CP3 默认读取并基于 `engine/research_production_contracts.py` 与相关现有 contract 做 extension plan，符合 ML-1 contract convergence。 |
| CP3-FOCUS-CR152-003 | 用户 CP2 approval feedback | CP2 discussion log N/A 是合理的，但 CP3 若展开架构灰区，必须补 CP3 discussion checkpoint。 | CP3 若有 Architecture Gray Areas，写 `process/discussions/CP3-CR152-HLD-DISCUSSION-LOG.md` 与 `process/checks/CP3-CR152-DISCUSSION-CHECKPOINT.json`；不得只在对话中处理。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-02T09:59:41+08:00
- 修改意见：approve；CP3 必须关注 model artifact metadata vs model registry write 边界、TrainingSnapshotSpec / ModelArtifactRef 扩展而非重建、CP3 架构灰区需补 discussion checkpoint。
- 风险接受项：
- 已接受决策项：`DQ-CP2-CR152-SCOPE`、`DQ-CP2-CR152-FIRST-WAVE`、`DQ-CP2-CR152-SECURITY-AUTHZ`、`DQ-CP2-CR152-ML-GATE-RELATION`、`DQ-CP2-CR152-STAT-GATE-OPT-IN`、`DQ-CP2-CR152-LABEL-POLICY-EXTENSIBILITY`、`DQ-CP2-CR152-FIXTURE-DATA-CONTRACT`
