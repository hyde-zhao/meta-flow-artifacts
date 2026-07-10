---
checkpoint_id: "CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE"
checkpoint_name: "CR161 Strategy Admission Evidence Pipeline Hardening Scope"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-09T21:39:26+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-09T22:19:04+08:00"
auto_check_result: "process/checks/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-161"
  artifacts:
    - "process/changes/CR-161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-2026-07-09.md"
    - "process/context/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE-CONTEXT.yaml"
    - "process/checks/CP0-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-REQUEST-INTAKE.result.json"
    - "process/checks/CP1-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-USE-CASE-COMPLETENESS.result.json"
    - "process/checks/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.result.json"
---

# CP2 CR161 Strategy Admission Evidence Pipeline Hardening Scope 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP0-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-REQUEST-INTAKE.result.json` | PASS | 0 | CP0 请求受理通过，route plan 已生成。 |
| `process/checks/CP1-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | C1-C4、typed_unavailable、CR155 negative regression 和 no-runtime 边界已覆盖。 |
| `process/checks/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.result.json` | PASS | 0 | CP2 范围、授权边界和待决策项已收敛，可发起人工门禁。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR161 第一阶段范围：用 contract-first 的 evidence availability / typed_unavailable fail-closed 语义补强 strategy admission，而不是立即改造研究引擎或修复 CR155。 |
| 推荐动作 | `approve`：接受 Wave 1 contract-first、typed_unavailable fail-closed、CR155 negative regression、forward instrumentation follow-up、no-runtime/no-new-real-data authorization boundary。 |
| approve 后会发生什么 | CR161 进入 CP3 设计，输出 evidence contracts、availability model、decision table、CR155 negative regression 验证口径、后续 instrumentation / impact-capacity follow-up 设计边界。 |
| approve 不授权什么 | 不授权 source/test implementation、research-engine instrumentation、新 real lake 读取、lake 写入、NAS/provider/credential、runtime、paper/simulation/live/trading、broker/order、external framework、catalog/store/registry/model/prediction 写入、Git remote write 或 publish。 |
| 不确认会阻塞什么 | 阻塞 CR161 CP3 设计和后续产品基线刷新；C1-C4 blocker 继续保留在现有 CR151/154 slot / fixture 语义层。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 CP2 capsule、CR161 summary、CR161 正文、CP0/CP1/CP2 result 和 route plan；仅在冲突、字段不足、人工审计或深度评审时扩展到 CR151/154 summaries 和 CR155 packaged evidence。 |
| 全文档读取扩展 | 无。CP2 已使用 summary 和已知证据事实，不复制完整历史 CR。 |
| 缺失 / waived 理由 | N/A。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 5 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 2 | trial lineage 来源和 no-runtime/no-new-data 边界。 |
| agent 默认处理 | 3 | CP3 文档结构、状态枚举命名、产品基线章节锚点由 agent 按现有规范处理并留证据。 |
| 仅审计记录 | 3 | CR tracking 历史债务、CR160 closed delivered gate_status checker warning、CR032 legacy active/no-overlap。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP2-CR161-WAVE1-TRIAL-LINEAGE-SOURCE` | scope | Wave 1 的 trial lineage 从哪里来？当前研究引擎不记录 trial count、parameter search lineage 或 experiment family。 | 选择 contract-first：先定义 `ExperimentFamilyManifest` contract、availability status 和 validation；trial lineage 缺失时输出 `typed_unavailable`。 | A. 前向改造研究引擎自动记录；B. 从 CR155 既有 packaged evidence 反推；C. 暂缓 CR161。 | 推荐方案最小且符合 fail-closed；A 范围大并触及研究引擎 instrumentation；B 可能伪造历史 lineage；C 保留核心 blocker。 | 当前切片不能宣称完整 C1 计算能力，只能证明缺失时 fail-closed。 | 如果用户选择 A，必须重算 route plan，启用 HLD + Story/LLD + CP6；如果选择 B，CP3 必须证明历史 evidence 充足，否则 BLOCKED。 |
| `DQ-CP2-CR161-TYPED-UNAVAILABLE-FAIL-CLOSED` | admission_policy | `typed_unavailable` 是否阻断 admission？例如缺 trial count、p-values、PBO/DSR、fold-level metrics 时是否可降级为 NEEDS_REVIEW。 | 缺 mandatory evidence 时 fail-closed：阻断 statistical significance、performance robustness、paper candidate、production-like / scalable claims；不得 silent PASS。 | A. 只标记 NEEDS_REVIEW；B. 对 exploratory 放行但限制 wording；C. 允许人工 waiver。 | 推荐方案最符合生产级 admission；A 容易过度晋级；B 可作为 CP3 tier table 细节但不能放宽 paper candidate；C 高风险且需要 waiver policy。 | 会让旧 artifact 更容易被 blocked，但这是正确风险暴露。 | CP3 可细化 exploratory / research-note 的 wording ceiling，但不得允许 paper/simulation/runtime readiness。 |
| `DQ-CP2-CR161-CR155-NEGATIVE-REGRESSION` | verification | CR155 packaged evidence 缺 p-values / trial lineage / PBO / DSR 时，negative regression 怎么验收？ | 验收目标是 CR155 仍 `blocked_admission_failed`、`paper_candidate=false`；C1/C2 新 evidence 不可计算时可显式 `typed_unavailable`。 | A. 要求 CR155 历史 C1/C2 全部可计算；B. 不使用 CR155；C. 把 CR155 作为 remediation target。 | 推荐方案使用最强负样本且不伪造历史证据；A 不现实；B 浪费已验证样例；C 会把方法论 CR 污染成策略优化。 | CP7 必须验证 rerun consistency PASS 不覆盖 admission fail。 | 若 CP3/CP7 发现 CR155 evidence 与 summary 冲突，则暂停并回 CP2 修正 regression criteria。 |
| `DQ-CP2-CR161-FOLLOWUP-SPLIT` | implementation | research-engine forward instrumentation、impact/capacity computation 是否进入当前 CP6 最小实现？ | 不进入当前最小切片；作为 follow-up / later wave 记录：`FU-CR161-001` trial-lineage instrumentation、`FU-CR161-002` impact/capacity implementation。 | A. 当前 CR 一次性实现全部；B. 只做 C1/C2 不记录 C3/C4；C. 拆成四个正式 CR。 | 推荐方案保留 parent scope 但降低 CP6 风险；A 过大；B 丢失经济判别方向；C 增加审计碎片。 | 当前 CR 的 CP3 应设计接口和后续切换条件，但不承诺立即修改研究引擎。 | 如果 CP2 选择一次性实现，必须启用 CP4/CP5/CP6 并明确文件 owner / regression scope。 |
| `DQ-CP2-CR161-AUTHORIZATION-BOUNDARY` | runtime_authorization | CR161 是否授权新真实数据读取、runtime、paper/simulation、broker、外部框架或 publish？ | 不授权任何新操作。只允许本地设计、产品基线、静态读取既有 evidence；任何新数据/runtime/implementation 另走后续门禁。 | A. 授权新 real lake validation；B. 授权 simulation/paper dry-run；C. 授权外部框架辅助计算。 | 推荐方案边界清楚，避免方法论 CR 变 runtime CR；A/B/C 都是高风险不同目标。 | 防止 admission hardening 被误读为 paper/simulation/live 或真实 TCA 能力。 | 任何需要 `.env`、lake/NAS/provider、broker、runtime、external framework、Git remote 或 publish 的动作必须停止并另起授权。 |

### CP2 追加字段

| 项 | 内容 |
|---|---|
| 用户真实意图 | 补强 strategy admission 的统计和经济方法论，使缺 evidence 时 fail-closed，而不是让策略静默晋级。 |
| 场景覆盖 | trial lineage missing、p-value family missing、PBO/DSR missing、OOS fold metrics missing、CR155 blocked negative regression、impact/capacity deferred boundary。 |
| 认知盲区补充 | `ExperimentFamilyManifest` 的核心风险是 lineage 来源，不是 dataclass；CR155 negative regression 的目标是 blocked preservation，不是补齐历史 C1/C2。 |
| Scenario Gray Areas | SGQ-CR161-001 trial lineage source；SGQ-CR161-002 typed_unavailable ceiling；SGQ-CR161-003 CR155 historical evidence gap；SGQ-CR161-004 follow-up split；SGQ-CR161-005 authorization boundary。 |
| Deferred Ideas | research-engine instrumentation、full PBO/DSR computation, real TCA calibration, capacity curve implementation, CR155 strategy remediation。 |
| 用户选择影响 | approve 后进入 CP3 设计；不启动代码实现；不授权真实数据 / runtime。 |
| 回退方式 | `修改: <具体修改点>` 后重写 CP2；选择代码路径则重算 route plan 并进入 CP4/CP5/CP6；`reject` 则 CR161 保持 cp2_pending 或关闭为 rejected。 |

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 5 |
| 必须用户决策 | `DQ-CP2-CR161-WAVE1-TRIAL-LINEAGE-SOURCE`、`DQ-CP2-CR161-TYPED-UNAVAILABLE-FAIL-CLOSED`、`DQ-CP2-CR161-CR155-NEGATIVE-REGRESSION`、`DQ-CP2-CR161-FOLLOWUP-SPLIT`、`DQ-CP2-CR161-AUTHORIZATION-BOUNDARY` |
| 推荐回复 | `approve` |
| 如果你回复 approve | 表示接受上表 5 项推荐方案：contract-first、typed_unavailable fail-closed、CR155 blocked negative regression、forward instrumentation follow-up、无新增授权。 |
| 不表示授权 | 不表示授权 source/test implementation、research-engine instrumentation、新 real lake read/write、NAS/provider/credential、runtime/paper/simulation/live/trading/broker、external framework、catalog/store/registry write、Git remote write 或 publish。 |
| 修改: <具体修改点> | 用户可回复具体修改 trial-lineage source、typed_unavailable policy、CR155 regression criteria、follow-up split 或 authorization boundary。 |
| reject | 用户可拒绝本 CP2，CR161 不进入 CP3。 |

### CP3 设计关注点

| 关注点 ID | 来源 | CP3 必须处理的问题 | 推荐处理 |
|---|---|---|---|
| CP3-FOCUS-CR161-001 | CP2 DQ | Evidence availability 必须有 `present` / `typed_unavailable` / `not_applicable_with_reason` / `blocked` 等稳定状态。 | HLD 定义统一状态机和 evidence envelope。 |
| CP3-FOCUS-CR161-002 | CP2 DQ | `typed_unavailable` 对不同 claim tier 的 ceiling 必须明确。 | 用 decision table 映射 exploratory / candidate / paper / production-like claims。 |
| CP3-FOCUS-CR161-003 | CP2 DQ | CR151/154 integration 不得创建平行 gate 家族。 | 新 evidence contracts 作为上游 refs / summaries，被现有 gate 消费。 |
| CP3-FOCUS-CR161-004 | CP2 DQ | CR155 negative regression 必须可验证且不得补写历史事实。 | 产出 seed classification / verification checklist；缺 C1/C2 原始输入时显示 typed_unavailable。 |
| CP3-FOCUS-CR161-005 | CP2 DQ | research-engine instrumentation follow-up 的切换条件必须明确。 | 定义 `FU-CR161-001` 触发条件：需要 automatic trial lineage for future runs。 |
| CP3-FOCUS-CR161-006 | CP2 DQ | C3/C4 不得被完全丢弃。 | 在 HLD 记录 impact/capacity evidence contract 和 later wave / follow-up boundary。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR161 formal CR exists | 待审查 | `process/changes/CR-161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-2026-07-09.md` | `lifecycle_status=active`，`gate_status=cp2_pending`。 |
| CP0 request intake passed | 待审查 | `process/checks/CP0-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-REQUEST-INTAKE.result.json` | CP0 decision PASS。 |
| CP1 use-case completeness passed | 待审查 | `process/checks/CP1-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-USE-CASE-COMPLETENESS.result.json` | C1-C4 and negative regression coverage present. |
| CP2 context capsule ready | 待审查 | `process/context/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE-CONTEXT.yaml` | read_profile=compact。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | Wave 1 是否接受 contract-first，不前向改造研究引擎 | 待审查 | `DQ-CP2-CR161-WAVE1-TRIAL-LINEAGE-SOURCE` | 推荐 approve。 |
| 2 | `typed_unavailable` 是否必须 fail-closed | 待审查 | `DQ-CP2-CR161-TYPED-UNAVAILABLE-FAIL-CLOSED` | 推荐 approve。 |
| 3 | CR155 negative regression 是否只要求保持 blocked / `paper_candidate=false` | 待审查 | `DQ-CP2-CR161-CR155-NEGATIVE-REGRESSION` | 推荐 approve。 |
| 4 | forward instrumentation / impact-capacity implementation 是否作为 follow-up | 待审查 | `DQ-CP2-CR161-FOLLOWUP-SPLIT` | 推荐 approve。 |
| 5 | 不授权项是否完整 | 待审查 | `DQ-CP2-CR161-AUTHORIZATION-BOUNDARY` | 推荐 approve。 |
| 6 | CP3 是否必须避免平行 gate 家族 | 待审查 | `CP3-FOCUS-CR161-003` | 新 evidence 应接入 CR151/154。 |
| 7 | CR155 历史 evidence 不足是否已显式处理 | 待审查 | `CP3-FOCUS-CR161-004` | 不伪造 trial lineage。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck PASS | 待审查 | `process/checks/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.result.json` | 可发起人工确认。 |
| Pending decisions collected | 待审查 | Decision Brief | 5 项待用户确认。 |
| Context capsule ready | 待审查 | `process/context/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE-CONTEXT.yaml` | read_profile=compact。 |
| Unauthorized scope explicit | 待审查 | Decision Brief / context capsule | approve 不授权任何新外部、运行时、真实数据或实现操作。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR161 formal CR | `process/changes/CR-161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-2026-07-09.md` | 待审查 | 已创建。 |
| CR161 summary | `process/changes/summaries/CR-161.summary.json` | 待审查 | state-router / cr-tracking 轻量入口。 |
| CP0 result | `process/checks/CP0-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-REQUEST-INTAKE.result.json` | 待审查 | request intake。 |
| CP1 result | `process/checks/CP1-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-USE-CASE-COMPLETENESS.result.json` | 待审查 | use-case completeness。 |
| CP2 context capsule | `process/context/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE-CONTEXT.yaml` | 待审查 | compact profile。 |
| CP2 result JSON | `process/checks/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.result.json` | 待审查 | 自动预检。 |
| CP2 human checkpoint | `process/checkpoints/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.md` | 待审查 | 本文件。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-09T22:19:04+08:00
- 修改意见：CP2 approved. CP3 must explicitly cover the 7 evidence objects: ExperimentFamilyManifest, MultipleTestingEvidence, DataSnoopingEvidence, OverfitRiskEvidence, WalkForwardEvidence, EconomicCostEvidence, CapacityLiquidityEvidence.
- 风险接受项：Accept contract-first design-only first slice; actual FDR/PBO/DSR computation, trial-lineage instrumentation, fold-level OOS computation, impact/capacity implementation and real TCA remain follow-up / later CR scope.
- 已接受决策项：
  - DQ-CP2-CR161-WAVE1-TRIAL-LINEAGE-SOURCE
  - DQ-CP2-CR161-TYPED-UNAVAILABLE-FAIL-CLOSED
  - DQ-CP2-CR161-CR155-NEGATIVE-REGRESSION
  - DQ-CP2-CR161-FOLLOWUP-SPLIT
  - DQ-CP2-CR161-AUTHORIZATION-BOUNDARY
