---
checkpoint_id: "CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE"
checkpoint_name: "CR160 Stage 4 Observation Review Workflow Scope"
type: "auto_then_manual"
status: "pending"
owner: "host-orchestrator"
created_at: "2026-07-08T22:55:00+08:00"
reviewed_by: ""
reviewed_at: ""
auto_check_result: "process/checks/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-160"
  artifacts:
    - "process/changes/CR-160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-2026-07-08.md"
    - "process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml"
    - "process/checks/CP0-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-REQUEST-INTAKE.result.json"
    - "process/checks/CP1-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-USE-CASE-COMPLETENESS.result.json"
    - "process/checks/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.result.json"
---

# CP2 CR160 Stage 4 Observation Review Workflow Scope 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP0-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-REQUEST-INTAKE.result.json` | PASS | 0 | CP0 请求受理通过，route plan 已生成。 |
| `process/checks/CP1-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | CR157 deferred baseline、Stage 4 场景、产品文档更新面和 no-runtime 边界已覆盖。 |
| `process/checks/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.result.json` | PASS | 0 | CP2 范围、授权边界、CP6 N/A 口径和待决策项已收敛，可发起人工门禁。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR160 的 Stage 4 observation review workflow 范围，使后续设计只定义 review workflow、observation plan contract、gate approval decision table、checklist、CR155 fail-closed 样例和 authorization boundary。 |
| 推荐动作 | `approve`：接受双轨 fail-closed evidence profile、CR155 `blocked_admission_failed` 样例、纯设计交付、无新增操作授权、CP4/CP5/CP6 N/A 且设计交付物进入 CP7 验证输入。 |
| approve 后会发生什么 | CR160 进入 CP3 设计；随后更新产品基线，输出 Stage 4 HLD / gate contract / review checklist / CR155 classification evidence，并在 CP7 验证 contract completeness、checklist testability、CR155 fail-closed classification 和 no-overclaim wording。 |
| approve 不授权什么 | 不授权 simulation/paper/live/trading/runtime、QMT/MiniQMT/xtquant/gateway、broker/order、credential/env 读取、新 real lake 读取、lake/NAS/provider/catalog/store/registry/model/prediction 写入、Git remote write 或 publish。 |
| 不确认会阻塞什么 | 阻塞 CR160 CP3 设计、产品基线更新、Stage 4 review gate contract、CP7 设计验证和 CP8 交付关闭。CR157 的 `observation_plan_ref` / `stage4_observation_gate_approved` 将继续保持未定义语义。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 CP2 capsule、CR160 summary、CR160 正文、CP0/CP1/CP2 result 和 route plan；仅在冲突、字段不足、人工审计或深度评审时扩展到 CR155 evidence、CR157 product baseline 或代码合同。 |
| 全文档读取扩展 | 3 组：`RE-20260708T135800Z0000-cr160formal`、`RE-20260708T135800Z0000-cr155evid`、`RE-20260708T135800Z0000-cr157prod`。 |
| 缺失 / waived 理由 | N/A。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` / `process/state/GATE-LEDGER.ndjson` | scanned | 1 | 0 | 当前 pending gate 为 CR160 CP2；GATE ledger 无更早 CR160 人工决策。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | CP2 前尚未委托 meta-pm 子 agent；本轮由 host-orchestrator 根据用户评审直接生成 scope gate。 |
| 自动预检结果 | `process/checks/CP0-*`、`process/checks/CP1-*`、`process/checks/CP2-*` | scanned | 2 | 2 | CP6 route mismatch 和 scope_authz_consistency NEEDS_REVIEW 均纳入 CP2 决策和说明。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 用户已直接给出 CP0 审查和 CP2 推荐；无独立多轮 Scenario Gray Areas 讨论日志，记录为 N/A with reason。 |
| 下游正式产物 | `engine/mature_multifactor_framework.py`、CR155 evidence、CR157 product baseline refs | scanned | 5 | 5 | observation plan contract、stage4 gate approval、CR155 blocked sample、authorization boundary、deliverable shape 进入待决策清单。 |
| 用户显式选择题 | 当前对话 / CR160 CP0 审查意见 | scanned | 5 | 5 | 用户建议双轨 fail-closed、CR155 blocked seed、纯设计、无新增授权、CP4/CP5 waived/CP6 N/A/CP7 输入。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 5 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 1 | `DQ-CP2-CR160-AUTHORIZATION-BOUNDARY` 明确 no-runtime / no-new-real-lake / no-publish。 |
| agent 默认处理 | 3 | CP2 approve 后的文档章节顺序、HLD 文件标题、checklist 表格字段由 agent 按现有文档规范处理并留证据。 |
| 仅审计记录 | 4 | CR155 READY_WITH_RISK vs admission FAIL 分离、route planner 不支持 literal `standard`、CR159 legacy tracking debt、CP6 N/A 修正文案。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CP2-CR160-EVIDENCE-PROFILE` | scope | Observation review 的输入是 contract-level evidence、real-data evidence，还是双轨？这决定 CR157/CR158 contract artifact 和 CR155 real-data fail-closed 样例能否被同一 workflow 审查。 | 双轨 fail-closed：`contract_only` lane 只能输出 `contract_review_complete` / `blocked` / `needs_real_data_evidence` 等低等级结论；`real_data_validated` lane 仍需 admission 非阻断才可进入 observation review。 | A. contract-only；B. real-data only；C. 暂缓定义 evidence profile，只写 checklist。 | 推荐方案同时覆盖 contract artifact 和真实数据反例，并防止 contract lane 输出 `paper_candidate=true` / `simulation_ready`；A 无法消费 CR155 反例；B 会排除 CR157/CR158 contract artifact；C 会让 Stage 4 gate 继续语义不完整。 | 影响 observation review input contract、decision table、后续验证矩阵和 no-overclaim 规则。 | 若 CP3 发现双轨表过复杂，可保留双轨概念并缩小状态枚举；若后续要求 real-data only，必须另起 product-scope CR 修改基线。 |
| `DQ-CP2-CR160-CR155-SEED-CLASSIFICATION` | scope | CR155 的 `READY_WITH_RISK` artifact closure、`BLOCKED` admission package、`FAIL` statistical gate、`paper_candidate=false` 应如何进入 CR160？ | 将 CR155 分类为 `blocked_admission_failed`，只作为 fail-closed 标准样例，不作为 observation candidate。 | A. 暂不使用 CR155 样例；B. 误用 CR155 为 observation candidate；C. 将 CR155 仅作为文字风险说明，不进入 decision table。 | 推荐方案利用 CR155 的确定性 rerun + 正确阻断证据验证 fail-closed；A 浪费最强反例；B 会越权提升 blocked artifact；C 可验证性不足。 | 防止 READY_WITH_RISK 被误读为 admission ready，并要求 CP7 复现该分类。 | 若发现 CR155 evidence 字段冲突，CP3/CP7 必须 BLOCKED 并回到 CP2 修正 seed classification。 |
| `DQ-CP2-CR160-DELIVERABLE-SHAPE` | implementation | CR160 交付设计文档、代码，还是两者？这决定是否触发 Story/LLD/CP6 实现路径。 | 纯设计交付：`docs/design/HLD-STAGE4-OBSERVATION-REVIEW-WORKFLOW.md`、`docs/quality/OBSERVATION-REVIEW-CHECKLIST.md`、`process/checks/CR160-CR155-SEED-CLASSIFICATION.md` 和产品基线更新；不新增代码。 | A. 增加 `engine/observation_review.py` schema/static checker + fixture tests；B. 只更新产品需求，不做 HLD/checklist；C. 暂缓 CR160。 | 推荐方案先定义语义，风险最低且符合 Stage 4 缺口性质；A 会扩大到代码实现并需重算 CP4/CP5/CP6；B 无法交付 gate contract；C 保持 CR157 语义缺口。 | 影响 CP3/CP7 交付物、是否需要 Story decomposition、是否需要 implementation dispatch。 | 若用户选择 A，必须重算 route plan，启用 CP4/CP5/CP6 并补 Story/LLD；若 CP3 发现设计不足以验证，可在 CP8 生成代码 follow-up。 |
| `DQ-CP2-CR160-AUTHORIZATION-BOUNDARY` | runtime_authorization | CR160 是否授权 simulation/paper/runtime 或新的 real lake 操作？同时如何解释 `scope_authz_consistency=NEEDS_REVIEW` 中 CR155 real lake evidence 与 no-new-lake-read 的张力？ | 不授权任何新操作。CR160 只消费 CR155 既有 real lake validation evidence 作为 fail-closed 样例；“消费既有 evidence”不等于“授权新 lake 读取”。 | A. 另起 runtime authorization CR；B. 另起 real-data validation CR；C. 暂停使用 CR155 evidence，避免 real-data 字样。 | 推荐方案边界最清楚且保留 CR155 样例价值；A/B 属于不同高风险目标；C 会降低 fail-closed 验证质量但可作为保守退路。 | 防止 observation review 被误读为 paper/simulation/live/real-data execution authorization。 | 任何新 real lake 读取、provider/NAS、runtime、paper/live、broker/order、registry/store/catalog 写入都必须停止并另起授权门禁。 |
| `DQ-CP2-CR160-CP4-CP5-ROUTE` | implementation | 纯设计路径下 CP4/CP5/CP6 如何处理，设计交付 evidence 归属哪个 CP？ | 纯设计：CP4 N/A、CP5 N/A、CP6 N/A；设计交付物在 CP7 前作为验证输入记录。CP7 通过 meta-dev/meta-qa design review 或批准的 inline-fallback 验证 contract completeness、checklist testability、CR155 classification 和 no-overclaim wording。 | A. 强制所有设计走 Story/LLD/CP6；B. 静默跳过 CP4/CP5/CP6；C. 改为代码路径并启用标准 CP4/CP5/CP6。 | 推荐方案与 route plan 和无代码事实一致；A 增加无意义过程负担；B 不符合审计规则；C 只有选择 schema/checker 代码实现时合理。 | 影响 route plan、agent dispatch evidence、CP7 验证输入和 CP8 release readiness。 | 若 CP2 或 CP3 改成含代码实现，必须重算 route plan 并回到 CP4/CP5；若缺 meta-dev/meta-qa dispatch 能力，需用户批准 inline-fallback，否则 CP7 不得 PASS。 |

### CP2 追加字段

| 项 | 内容 |
|---|---|
| 用户真实意图 | 补齐 CR157 Stage 4 observation review 语义缺口，设计 gate 和 review workflow，不启动 runtime、paper、simulation 或新真实数据访问。 |
| 场景覆盖 | contract-only evidence、real-data validated evidence、blocked admission failed artifact、runtime authorization request、no-overclaim wording。 |
| 认知盲区补充 | READY_WITH_RISK 是 artifact scope closure，不等于 admission ready；CR155 的真实验证证据是已存在证据引用，不是新 lake 访问授权；CP6 在纯设计路径下应保持 N/A。 |
| Scenario Gray Areas | SGQ-CR160-001：contract-level vs real-data evidence；SGQ-CR160-002：CR155 seed classification；SGQ-CR160-003：设计文档 vs 代码实现；SGQ-CR160-004：runtime authorization boundary；SGQ-CR160-005：CP4/CP5/CP6 route。 |
| Deferred Ideas | schema/static checker、fixture tests、paper/simulation authorization gate、runtime observation execution gate、real-data revalidation gate、strategy remediation workflow。 |
| 用户选择影响 | approve 后进入 CP3 设计；不启动 Story/LLD/代码实现；CP4/CP5/CP6 保持 N/A；CP7 做设计验证。CP3 必须把 checklist 扩展到 Stage 1/2/3 和横切层，不得只审 Stage 3 admission package。 |
| 回退方式 | `修改: <具体修改点>` 后重写 CP2；选择代码路径则重算 route plan 并进入 CP4/CP5/CP6；`reject` 则 CR160 保持 cp2_pending 或关闭为 rejected，不进入 CP3。 |
| discussion log / checkpoint | 本轮由用户直接提供 CP0 审查与 CP2 推荐；无独立多轮 discussion log，记录为 N/A with reason。 |

### 用户需决策事项 summary

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 5 |
| 必须用户决策 | `DQ-CP2-CR160-EVIDENCE-PROFILE`、`DQ-CP2-CR160-CR155-SEED-CLASSIFICATION`、`DQ-CP2-CR160-DELIVERABLE-SHAPE`、`DQ-CP2-CR160-AUTHORIZATION-BOUNDARY`、`DQ-CP2-CR160-CP4-CP5-ROUTE` |
| 推荐回复 | `approve` |
| 如果你回复 approve | 表示接受上表 5 项推荐方案：双轨 fail-closed、CR155 blocked sample、纯设计、无新增授权、CP4/CP5/CP6 N/A 且 CP7 验证设计交付。 |
| 不表示授权 | 不表示授权 simulation/paper/live/trading/runtime、QMT/MiniQMT/xtquant/gateway、broker/order、credential/env、新 real lake 读取、lake/NAS/provider/catalog/store/registry/model/prediction 写入、Git remote write 或 publish。 |
| 修改: <具体修改点> | 用户可回复具体修改 evidence profile、CR155 seed classification、deliverable shape、authorization boundary 或 route handling。 |
| reject | 用户可拒绝本 CP2，CR160 不进入 CP3。 |

### CP3 设计关注点

| 关注点 ID | 来源 | CP3 必须处理的问题 | 推荐处理 |
|---|---|---|---|
| CP3-FOCUS-CR160-001 | CP2 DQ | `ObservationReviewInput`、`EvidenceProfile`、`AdmissionReadiness`、`ObservationDecision`、`AuthorizationBoundary` 的状态链必须 fail-closed。 | CP3 HLD 中用 decision table 表达；所有 unknown / inconsistent evidence 默认 blocked 或 needs_real_data_evidence。 |
| CP3-FOCUS-CR160-002 | CP2 DQ | `contract_only` lane 不能产生 `paper_candidate=true`、`simulation_ready=true` 或 `runtime_authorized=true`。 | 在 gate contract 和 checklist 中写成硬性 no-overclaim rule。 |
| CP3-FOCUS-CR160-003 | CP2 DQ | CR155 seed classification 必须可在 CP7 复核。 | 产出 `process/checks/CR160-CR155-SEED-CLASSIFICATION.md`，列出 BLOCKED / FAIL / false / deterministic replay 事实。 |
| CP3-FOCUS-CR160-004 | CP2 DQ | 设计-only 路径仍需可验证。 | CP7 验证输入包括 HLD、checklist、classification sample 和 no-overclaim wording；meta-dev/meta-qa dispatch 或 approved inline-fallback 必须留证据。 |
| CP3-FOCUS-CR160-005 | 用户全流程评审 | Checklist 不能只覆盖 Stage 3；必须分层覆盖 Stage 1 数据基础、Stage 2 研究生产、Stage 3 研究机门禁和横切授权 / no-overclaim。 | `docs/quality/OBSERVATION-REVIEW-CHECKLIST.md` 至少包含 Stage 1 PIT/universe/lineage、Stage 2 factor methodology / typed_unavailable handling、Stage 3 statistical gate / OOS / economic significance / capacity / impact / rerun、cross-cutting counters / authorization / no-overclaim。 |
| CP3-FOCUS-CR160-006 | 用户全流程评审 | `observation_plan_ref` 需要区分模板与实例，避免把 CR160 设计模板误当成 Stage 3 已产出的具体计划。 | CP3 定义 `observation_plan_template` 内容规范；Stage 3 package 未来产出 `observation_plan_instance`，review workflow 审查 instance 对 template 的合规性。 |
| CP3-FOCUS-CR160-007 | 用户全流程评审 | Observation plan 缺少时间维度会导致 Stage 4 无法审查观察周期和退出条件。 | template 必须定义 observation period、checkpoint frequency、tracking metrics、exit criteria、pause / remediation trigger。 |
| CP3-FOCUS-CR160-008 | 用户全流程评审 | 审查主体需要明确，避免 checklist 被误解为自动 gate 或 runtime authorization。 | CP3 明确 checklist 是人工审查工具，decision table 是审查后的分类框架；自动化 checker 若需要，必须作为后续 follow-up CR。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR160 formal CR exists | 待审查 | `process/changes/CR-160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-2026-07-08.md` | `lifecycle_status=active`，`gate_status=cp2_pending`。 |
| CP0 request intake passed | 待审查 | `process/checks/CP0-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-REQUEST-INTAKE.result.json` | CP0 decision PASS。 |
| CP1 product baseline completeness passed | 待审查 | `process/checks/CP1-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-USE-CASE-COMPLETENESS.result.json` | Stage 4 review scenarios and no-runtime boundary are covered. |
| CP2 context capsule ready | 待审查 | `process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml` | read_profile=compact。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | Evidence profile 是否接受双轨 fail-closed 设计 | 待审查 | `DQ-CP2-CR160-EVIDENCE-PROFILE` | 推荐 approve。 |
| 2 | CR155 是否仅作为 `blocked_admission_failed` 样例 | 待审查 | `DQ-CP2-CR160-CR155-SEED-CLASSIFICATION` | 推荐 approve。 |
| 3 | CR160 是否走纯设计交付，不新增代码实现 | 待审查 | `DQ-CP2-CR160-DELIVERABLE-SHAPE` | 推荐 approve。 |
| 4 | CR155 既有 evidence 引用是否不构成新 real lake 授权 | 待审查 | `DQ-CP2-CR160-AUTHORIZATION-BOUNDARY` | 推荐 approve。 |
| 5 | CP4/CP5/CP6 是否按纯设计路径 N/A，设计交付物归入 CP7 验证输入 | 待审查 | `DQ-CP2-CR160-CP4-CP5-ROUTE` | 推荐 approve。 |
| 6 | route plan 与 CR 正文 CP6 口径是否一致 | 待审查 | `process/checks/CP0-CR160.route-plan.json`、CR160 Checkpoint Index | 已修正为 CP6 N/A。 |
| 7 | 不授权项是否完整 | 待审查 | `process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml#scope_summary.out_of_scope` | approve 不授权任何 runtime / data / broker / publish 操作。 |
| 8 | CP3 是否必须覆盖 Stage 1→2→3→4→5 合同链 | 待审查 | `CP3-FOCUS-CR160-005` | 推荐 approve 后作为 CP3 HLD 和 checklist 的硬性设计约束。 |
| 9 | `observation_plan_ref` 是否拆分 template / instance | 待审查 | `CP3-FOCUS-CR160-006` | 推荐 approve 后由 CP3 明确 template 与 instance 的关系。 |
| 10 | observation plan 是否包含时间维度和人工审查主体 | 待审查 | `CP3-FOCUS-CR160-007`、`CP3-FOCUS-CR160-008` | 推荐 approve 后纳入 CP3 HLD / checklist。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck PASS | 待审查 | `process/checks/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.result.json` | 可发起人工确认。 |
| Pending decisions collected | 待审查 | Decision Brief | 5 项待用户确认。 |
| Context capsule ready | 待审查 | `process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml` | read_profile=compact。 |
| Unauthorized scope explicit | 待审查 | Decision Brief / context capsule | approve 不授权任何新外部或运行时操作。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR160 formal CR | `process/changes/CR-160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-2026-07-08.md` | 待审查 | 已创建并修正 CP6 N/A 口径。 |
| CR160 summary | `process/changes/summaries/CR-160.summary.json` | 待审查 | state-router / cr-tracking 轻量入口。 |
| CP0 result | `process/checks/CP0-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-REQUEST-INTAKE.result.json` | 待审查 | request intake。 |
| CP1 result | `process/checks/CP1-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-USE-CASE-COMPLETENESS.result.json` | 待审查 | product baseline completeness。 |
| CP2 context capsule | `process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml` | 待审查 | compact profile。 |
| CP2 result JSON | `process/checks/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.result.json` | 待审查 | 自动预检。 |
| CP2 human checkpoint | `process/checkpoints/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.md` | 待审查 | 本文件。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-08T23:20:00+08:00
- 修改意见：approve；接受 5 项推荐方案，并保留全流程评审新增的 CP3 设计约束：Stage 1/2/3/横切 checklist 分层、observation_plan_template vs observation_plan_instance、时间维度、人工审查主体。
- 风险接受项：N/A；本 CP2 不接受 runtime / new real lake / broker / provider / NAS / credential / publish 风险。
- 已接受决策项：`DQ-CP2-CR160-EVIDENCE-PROFILE`、`DQ-CP2-CR160-CR155-SEED-CLASSIFICATION`、`DQ-CP2-CR160-DELIVERABLE-SHAPE`、`DQ-CP2-CR160-AUTHORIZATION-BOUNDARY`、`DQ-CP2-CR160-CP4-CP5-ROUTE`
