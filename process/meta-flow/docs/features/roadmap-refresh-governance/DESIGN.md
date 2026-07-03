---
status: draft
version: "1.0"
feature_id: "FEAT-PG-006"
feature_name: "Roadmap Refresh Governance"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR-F-S01 roadmap refresh result schema and checker"
  - "CR-F-S02 process-only cascade and Gate Ledger event"
lld_policy_summary: "full-lld required for result schema, checker, process-only cascade, event contract, rollback and security boundary."
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Roadmap Refresh Governance

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 建立 ROADMAP-REFRESH 独立 result/checker、process-only cascade、Gate Ledger event 与发布库不写入边界 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 让 roadmap refresh 成为可审计、可回滚、可被 CP5/后续验证消费的独立治理机制。 |
| 推荐方案 | 新增 ROADMAP-REFRESH 专用 result schema、独立 checker、过程侧 cascade 和 Gate Ledger `roadmap_refresh` event；不复用 CP result，不自动写发布库。 |
| 关键取舍 | 增加一个专用 result/checker 的维护成本，换取 CP result 语义不被污染、跨仓授权边界清楚、refresh 输出可单独验证。 |
| 下游 Story | CR-F-S01、CR-F-S02 |
| LLD 策略 | 两个 Story 均为 full-lld，因为涉及 data-model、checker、event-contract、runtime authorization 和 rollback。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-RF-01 | HLD 明确 roadmap refresh 需要独立 result，不复用 CP result。 | CR / milestone / project state 变化后触发 refresh。 | result schema、checker 和 Gate Ledger 事件需要稳定契约。 | CP result 被过度泛化，checkpoint 与 roadmap 语义混乱。 |
| P-RF-02 | Roadmap refresh 会发现发布库文档或测试策略陈旧，但未授权自动修改发布库。 | quant-lab 或其他项目的 release repo refs 被 stale-check 命中。 | 需要 process-only cascade 和 stale/follow-up 输出。 | 未授权写发布库、跨仓事务失败、回滚不可审计。 |
| P-RF-03 | refresh 结果需要被后续 stale-check、FU-RF tracking 和 gate 审计消费。 | `UPDATED_WITH_DOC_IMPACTS` 或 `BLOCKED` 出现。 | 需要稳定 machine-readable result、summary 和 event refs。 | 下游只能读散落日志，无法追踪 refresh 决策。 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| Blueprint | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md` | FEAT-PG-006 负责 ROADMAP-REFRESH result/checker/cascade/Gate Ledger，禁止写发布库。 |
| Domain Map | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DOMAIN-MAP.md` | OBJ-PG-012 Roadmap Refresh Result、OBJ-PG-013 Roadmap Refresh Event、RULE-PG-008/009。 |
| Dependency Map | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DEPENDENCY-MAP.md` | FEAT-PG-006 允许依赖 FEAT-PG-003/007/008，禁止依赖 CP result checker 和发布库自动写入。 |
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | ADR-PG-003/005、关键流程 Roadmap Refresh Cascade、HLD §17 触发 full-lld。 |
| ADR | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md` | ADR-PG-003 过程库写边界；ADR-PG-005 独立 result/checker。 |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-006 required；CR-F-S01/CR-F-S02 均 full-lld。 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 定义 `ROADMAP-REFRESH-*.result.json` 的独立 schema、decision 枚举、machine_updates、stale_items、follow_up_candidates 和 event_refs。 | HLD §18 / Domain OBJ-PG-012 |
| Goal | 提供独立 `roadmap-refresh` checker，不调用 CP result checker，也不把 CP result 当作 refresh 结果。 | ADR-PG-005 / RULE-PG-008 |
| Goal | 建立 process-only cascade：自动写入范围仅限过程归档库和 process-side project objects。 | ADR-PG-003 / RULE-PG-009 |
| Goal | 追加 Gate Ledger `roadmap_refresh` event，记录 gate、status、result_ref 和 follow-up refs。 | Domain OBJ-PG-013 |
| Non-Goal | 不自动修改 quant-lab 或其他发布库的代码、tests、正式 docs、release docs。 | ADR-PG-003 |
| Non-Goal | 不关闭、恢复或完成 CR-036，不进入 CP5 或实现授权。 | CR-037 context |
| Non-Goal | 不把 ROADMAP-REFRESH result 合并进 CP result schema。 | ADR-PG-005 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| Roadmap Refresh Result | schema、decision、machine updates、stale/follow-up refs、summary refs。 | CP result、checkpoint result、Story evidence result。 | checkpoint-manager、CP result checker | ADR-PG-005 |
| Process-only Cascade | 更新过程侧 `PROJECT.current.json`、ROADMAP、MILESTONES 或其 refs。 | 自动修改 release repo 的 code/tests/docs。 | FEAT-PG-003、FEAT-PG-009 | ADR-PG-003 |
| Gate Ledger Event | 追加 `roadmap_refresh` event，并引用 result。 | 代替 human gate 或 CP checkpoint。 | Gate Ledger、host-orchestrator | HLD §11 |
| Stale / Follow-up 输出 | 输出 `stale_items`、`follow_up_candidates`。 | 定义 FU-RF 编号和 tracking 生命周期。 | FEAT-PG-007、FEAT-PG-008 | Dependency Map FLOW-PG-003 |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| CLI / checker | `meta_flow/`、`scripts/` 或既有 checker 模块 | 后续 LLD 需定位实际 CLI 和 checker 文件。 | modify/create，需在 Story LLD 中锁定文件所有权。 |
| Process project objects | `process/project/`、`process/state/PROJECT.current.json` | 过程侧 project state、roadmap、milestones。 | read/write process-side only。 |
| Gate Ledger | `process/state/GATE-LEDGER.ndjson` | gate event ledger。 | append event only，保持 ledger contract。 |
| Release repo refs | quant-lab 或其他目标项目发布库路径 | 只读 stale evaluation 输入。 | read-only，不写入。 |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| 数据 | HLD 定义了 result decision，但尚未冻结字段级 schema。 | 缺少 `ROADMAP-REFRESH-*.result.json` 机器契约。 | 必须独立于 CP result。 |
| 接口 | HLD 给出 refresh cascade 概念流程。 | 缺少 CLI/checker 输入输出、错误模型和 event append 契约。 | 不执行 runtime 或 production write。 |
| 测试 | Feature Matrix 要求 full-lld。 | 缺少 unit/integration/contract/security 测试计划。 | CP5 前只形成设计证据。 |
| 运维 / 发布 | 过程库与发布库边界已确认。 | 需要失败时 `BLOCKED`/`FAILED` 的恢复方式。 | 发布库只读，自动写入仅限 process artifact。 |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| Result 类型 | 使用 `ROADMAP-REFRESH-<timestamp>.result.json`，包含 `schema_version`、`decision`、`source_refs`、`machine_updates`、`must_check`、`stale_items`、`follow_up_candidates`、`event_refs`。 | 符合 roadmap refresh 的业务语义，并可被 checker 单独验证。 | 新增 schema 和 fixture。 |
| Decision 枚举 | `NO_CHANGE`、`UPDATED`、`UPDATED_WITH_DOC_IMPACTS`、`BLOCKED`、`FAILED`。 | 与 HLD 成功标准一致，可表达过程更新、文档影响和阻塞。 | 需要明确定义每个状态的必填字段。 |
| Checker | 新建独立 roadmap-refresh checker，校验 schema、decision-specific required fields、forbidden release writes、event refs。 | 不污染 CP result checker，降低跨域耦合。 | CLI UX 需额外入口或聚合包装。 |
| Cascade | refresh 只写 process artifact repository；release repo 只生成 stale/follow-up candidates。 | 满足 ADR-PG-003 和安全边界。 | 发布库修复需要后续 CR / FU-RF。 |
| Gate Ledger | append `event_type=roadmap_refresh`，包含 `result_ref`、`decision`、`gate_ref`、`status`。 | 让 gate 审计能追溯 refresh 发生原因和结果。 | 需要 ledger event contract 测试。 |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| DQ-RF-001 | Option A：独立 result/checker | 语义清晰；CP result 不膨胀；便于 stale/FU-RF 消费。 | 新增 checker 维护成本。 | roadmap、project-governance、gate-ledger | 推荐 | 保持；仅 CLI UX 可聚合，不合并 schema。 |
| DQ-RF-001 | Option B：复用 CP result/checker | 初始实现少。 | CP result 语义污染；roadmap decision 难表达。 | checkpoint、roadmap | 备选 | 仅当 roadmap refresh 被取消且只保留 CP 自动检查时考虑。 |
| DQ-RF-002 | Option A：process-only cascade | 授权边界清楚；回滚局限在过程侧。 | 发布库修复需要后续流程。 | security、runtime_authorization、release | 推荐 | 若未来要写发布库，必须另开 CR + human gate。 |
| DQ-RF-002 | Option B：跨仓自动修改发布库 | 自动化程度高。 | 未授权写入、跨仓事务和回滚风险高。 | release、security | 备选，不采用 | 只有获得明确发布库写授权且有回滚设计时可重新评审。 |

## 模块变更

| Module | 变更 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| roadmap refresh command | 触发 refresh、收集 project/CR/milestone 输入、生成 result。 | trigger、project refs、CR refs、roadmap refs | `ROADMAP-REFRESH-*.result.json`、summary | 输入缺失 -> `BLOCKED`；运行异常 -> `FAILED`。 |
| roadmap refresh checker | 校验 result schema、decision required fields、forbidden release writes。 | result path、optional ledger path | PASS/FAIL findings | schema invalid、release write path detected -> FAIL。 |
| process cascade writer | 只更新 process-side project objects 或 refs。 | machine_updates | updated process artifacts | 目标不在 process artifact boundary -> reject and `BLOCKED`。 |
| Gate Ledger appender | 追加 `roadmap_refresh` event。 | result_ref、gate_ref、decision | ledger event | ledger append failure -> result 标记 `BLOCKED` 或补偿为 follow-up。 |
| release repo stale collector | 只读发布库 refs 并生成 stale/follow-up candidates。 | release refs | `stale_items`、`follow_up_candidates` | 需要写发布库 -> 转 stale/FU-RF，不写入。 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| Roadmap Refresh Result | FEAT-PG-006 | `schema_version`、`result_id`、`decision`、`source_refs`、`machine_updates`、`must_check`、`stale_items`、`follow_up_candidates`、`event_refs`、`errors`。 | triggered -> evaluated -> no_change / updated / updated_with_doc_impacts / blocked / failed | 新 result 类型，不影响 CP result。 |
| Machine Update | FEAT-PG-006 | `target_ref`、`operation`、`before_hash`、`after_hash`、`rollback_ref`。 | proposed -> applied / rejected | `target_ref` 必须在 process artifact boundary。 |
| Stale Item | FEAT-PG-008 consumer | `object_ref`、`expected_semantic`、`observed_semantic`、`severity`、`recommended_follow_up`。 | open -> converted / waived | 只报告，不自动修复正式文档。 |
| Gate Ledger Event | FEAT-PG-006 | `event_type=roadmap_refresh`、`result_ref`、`decision`、`gate_ref`、`status`、`actor`。 | appended / rejected | 保持 ledger append-only。 |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| IF-RF-01 | host-orchestrator / CLI | roadmap refresh command | trigger reason、project_ref、roadmap_ref、optional CR/milestone refs、dry_run flag | result path、summary path、decision | missing input -> `BLOCKED`；unexpected exception -> `FAILED`。 |
| IF-RF-02 | roadmap refresh command | process cascade writer | machine_updates with process-only target refs | applied update refs and rollback refs | target outside process -> reject。 |
| IF-RF-03 | roadmap refresh command | Gate Ledger appender | event payload with `result_ref` and decision | appended event id | ledger schema invalid -> `BLOCKED`。 |
| IF-RF-04 | checkpoint / QA / stale-check | roadmap refresh checker | result path and optional ledger path | PASS / FAIL findings | CP result passed in as result -> FAIL wrong result type。 |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| FLOW-RF-01 | CR、milestone 或 project state 变化后触发 refresh。 | 收集 refs -> 评估 roadmap delta -> 写 process-side updates -> 生成 result -> append Gate Ledger event -> 运行 checker。 | 输入缺失或 target 越界时输出 `BLOCKED`，不写发布库。 | result decision 为 `NO_CHANGE`/`UPDATED`/`UPDATED_WITH_DOC_IMPACTS`/`BLOCKED`/`FAILED`。 | result_ref、event_id、checker findings。 |
| FLOW-RF-02 | 发现发布库 HLD/TEST/RELEASE docs 语义陈旧。 | 只读 release refs -> 生成 stale_items -> 生成 FU-RF/SP-RF/RA-RF candidate refs。 | 需要自动改发布库时拒绝并记录安全 finding。 | `UPDATED_WITH_DOC_IMPACTS` 或 `BLOCKED`。 | stale item count、follow-up candidate count。 |
| FLOW-RF-03 | Gate Ledger append 失败。 | 保留 result -> 标记 event append failure -> 返回 `BLOCKED` 或 `FAILED`。 | 不重写 ledger 历史，不绕过 ledger checker。 | result.errors 增加 ledger failure。 | ledger append error。 |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| DQ-RF-001 | CP5 前确认 result/checker 形态。 | host-orchestrator / user | 独立 ROADMAP-REFRESH result/checker。 | 复用 CP result。 | 不授权实现，不授权 CP5。 |
| DQ-RF-002 | 发现发布库陈旧项且用户希望自动修复。 | user / security reviewer | 转 FU-RF 或正式 CR。 | 新增发布库写授权 CR。 | 当前 Feature 不授权写发布库。 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| F-RF-01 | result 缺少 decision-specific required fields。 | checker FAIL。 | CP5 / QA 阻断。 | 修正 result schema 或 producer。 | TEST-PLAN SEC-RF-01 / CONTRACT-RF-01 |
| F-RF-02 | machine_updates 指向 release repo code/tests/docs。 | producer 拒绝，result=`BLOCKED`。 | 生成 stale/FU-RF 候选。 | 单独 CR + human gate。 | TEST-PLAN SEC-RF-02 |
| F-RF-03 | 尝试用 CP result checker 校验 ROADMAP-REFRESH。 | checker FAIL wrong type。 | 明确提示使用 roadmap-refresh checker。 | 改用独立 checker。 | TEST-PLAN CONTRACT-RF-02 |
| F-RF-04 | Gate Ledger event append 失败。 | result 记录错误，不伪造成功事件。 | refresh 不视为完整成功。 | 修复 ledger 写入后重跑或记录 blocked。 | TEST-PLAN INTEG-RF-02 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| SEC-RF-01 | ROADMAP-REFRESH 自动写入范围只允许 process artifact repository。 | `machine_updates[*].target_ref` 指向非 process 路径。 | reject update，result=`BLOCKED`，输出 finding。 | TEST-PLAN SEC-RF-02 |
| SEC-RF-02 | 发布库 code/tests/formal docs 只读，不自动修改。 | stale evaluation 命中 release repo refs。 | 只生成 stale_items/follow_up_candidates。 | TEST-PLAN INTEG-RF-03 |
| SEC-RF-03 | 不复用 CP result，不把 refresh result 写成 checkpoint result。 | checker 收到 CP result 或 result_type 不匹配。 | FAIL wrong result type。 | TEST-PLAN CONTRACT-RF-02 |
| SEC-RF-04 | Gate Ledger event append-only，不重写历史。 | 需要修正旧 event。 | 追加 correction event 或 blocked，不原地改 ledger。 | TEST-PLAN SEC-RF-03 |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| Result schema | unit / contract | 五类 decision 的 required fields。 | schema fixture tests。 | 无。 |
| Checker | unit / integration | 独立 checker 校验 PASS/FAIL、拒绝 CP result。 | checker tests。 | 无。 |
| Process-only cascade | integration / security | process 更新成功、release repo target 被拒绝。 | temp fixture workspace。 | 发布库真实写入只做人工审查，不自动化写。 |
| Gate Ledger event | contract / integration | event_type、result_ref、decision、status。 | ledger fixture tests。 | 无。 |
| Stale / follow-up 输出 | integration / manual | `UPDATED_WITH_DOC_IMPACTS` 输出 stale_items 和 follow_up_candidates。 | fixture + manual review。 | 语义陈旧判断需人工抽样确认。 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 冻结 ROADMAP-REFRESH result schema 和 decision-specific required fields。 | CP3 已批准，CP5 未开始。 | schema draft / fixture plan。 | TEST-PLAN unit / contract。 |
| 2 | 设计独立 checker 和 wrong-result-type 失败模型。 | schema 完成。 | checker contract。 | TEST-PLAN CONTRACT-RF-02。 |
| 3 | 设计 process-only cascade writer 与 forbidden target guard。 | FEAT-PG-003 project objects 明确。 | cascade contract。 | TEST-PLAN SEC-RF-02。 |
| 4 | 设计 Gate Ledger `roadmap_refresh` event。 | ledger event contract 明确。 | event payload contract。 | TEST-PLAN INTEG-RF-02。 |
| 5 | 设计 stale/follow-up 输出到 FEAT-PG-007/008 的消费契约。 | result schema 完成。 | downstream contract。 | TEST-PLAN INTEG-RF-03。 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR-F-S01 roadmap refresh result schema and checker | `process/docs/features/roadmap-refresh-governance/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | data-model / checker / rollback | result schema、decision required fields、checker CLI、error codes。 | Feature DESIGN / TEST-PLAN / TASKS |
| CR-F-S02 process-only cascade and Gate Ledger event | `process/docs/features/roadmap-refresh-governance/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | security / runtime_authorization / event-contract | process boundary guard、event append semantics、release repo stale output。 | Feature DESIGN / TEST-PLAN / TASKS |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| story-manager | CP4 前 | Story 拆分建议、full-lld policy、output paths。 | Story 卡片包含 `feature_design_refs` 和 `lld_policy`。 | 缺失则 CP4 FAIL。 |
| lld-designer | CP5 前 | 本 DESIGN、TEST-PLAN、TASKS、HLD/ADR。 | CR-F-S01/02 full LLD。 | 若 result/checker 字段争议，回到 CP5 clarification。 |
| meta-qa | CP7 / CP8 | TEST-PLAN、checker contract、security rules。 | 验证报告追溯 result/checker/cascade/event。 | 未自动化的 semantic stale 抽样需人工验收。 |
| FEAT-PG-007 / FEAT-PG-008 | Story LLD / 实现前 | `stale_items`、`follow_up_candidates`、result refs。 | FU-RF tracking 和 stale-check 消费稳定字段。 | 字段缺失则 refresh result checker FAIL。 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| R-RF-01 | 独立 checker 增加维护成本。 | CLI UX 变复杂。 | 可以在上层命令聚合检查，但 schema 不合并。 | 保持专用 checker，聚合入口仅做转发。 |
| R-RF-02 | process-only cascade 被误实现为跨仓写。 | 未授权修改发布库。 | forbidden target guard、security tests、manual review。 | 回滚 process result，转 FU-RF/CR。 |
| R-RF-03 | Gate Ledger event 字段与现有 ledger contract 不兼容。 | refresh 无法进入审计链。 | LLD 前检查 ledger event schema，先 fixture 后实现。 | result 保留 `event_append_failed`，不伪造成功。 |
| R-RF-04 | `UPDATED_WITH_DOC_IMPACTS` 语义不清导致误报。 | 下游 FU-RF 噪音。 | decision-specific examples + manual acceptance。 | 降级为 stale item，等待人工确认。 |
