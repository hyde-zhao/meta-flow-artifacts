---
status: draft
version: "1.0"
feature_id: "FEAT-PG-008"
feature_name: "Project Stale Check"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR-G-S02 project stale-check"
lld_policy_summary: "full-lld required for cross-object semantic rules, CLI/checker contract, stale finding schema, and no-auto-fix security boundary."
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Project Stale Check

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 定义项目级跨对象语义陈旧检查、stale finding 输出、FU-RF 集成和禁止自动修复正式文档边界 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 检测 project state、roadmap、HLD、测试策略和 release docs refs 之间的语义陈旧，输出可追踪 stale finding。 |
| 推荐方案 | 新增 project stale-check checker/CLI，产出 stale finding result；只报告和生成 FU-RF 候选，不自动修改 HLD、TEST-STRATEGY、release docs 或发布库。 |
| 关键取舍 | 增加语义检查规则维护成本，换取长期项目 roadmap 和正式文档之间的可审计一致性。 |
| 下游 Story | CR-G-S02 |
| LLD 策略 | full-lld，因为涉及跨对象语义、CLI 命名、result schema、follow-up 集成和安全失败路径。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-ST-01 | HLD 要求 stale-check 检测 roadmap 与 HLD/测试/发布文档的跨对象陈旧。 | ROADMAP 声明项目阶段变化，但 TEST-STRATEGY 或 release docs 仍停留旧语义。 | 需要语义规则和 finding 输出。 | 长期项目依赖人工记忆发现陈旧，迁移时遗漏。 |
| P-ST-02 | stale-check 不能重复 cr-tracking 结构一致性检查。 | CR tracking 已能发现 CR 状态/编号问题。 | 需要明确只做跨对象语义检测。 | checker 职责重叠，噪音过大。 |
| P-ST-03 | stale-check 发现正式文档陈旧，但当前未授权自动修复。 | quant-lab 或 meta-flow release docs 被命中。 | 需要只报告、转 FU-RF/CR。 | 未经 CR / gate 修改正式产物。 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| Blueprint | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md` | FEAT-PG-008 负责 stale-check result/findings；不重复 cr-tracking。 |
| Domain Map | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DOMAIN-MAP.md` | OBJ-PG-015 Stale Finding、RULE-PG-011 stale-check 边界。 |
| Dependency Map | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DEPENDENCY-MAP.md` | FEAT-PG-006 -> FEAT-PG-008 read/event；FD-PG-007 禁止自动修复正式文档。 |
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | SIM-PG-002、HLD §18 下沉 stale-check 规则和 follow-up 生成策略。 |
| ADR | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md` | ADR-PG-003/006 约束发布库只读和 follow-up tracking。 |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-008 required；CR-G-S02 full-lld。 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 定义 stale finding schema：stale object、expected semantic、observed semantic、severity、source refs、recommended follow-up。 | Domain OBJ-PG-015 |
| Goal | 设计 stale-check CLI/checker 的输入、输出、错误模型和 result summary。 | HLD §17 |
| Goal | 集成 FEAT-PG-006 refresh result 和 FEAT-PG-007 follow-up candidate。 | Dependency FLOW-PG-003 |
| Goal | 明确 stale-check 只报告，不自动修改正式 HLD / TEST-STRATEGY / release docs。 | FD-PG-007 |
| Non-Goal | 不检查 CR tracking 的结构一致性、编号合法性或 CR lifecycle 完整性。 | RULE-PG-011 |
| Non-Goal | 不自动修复陈旧文档，不绕过 CR 或 human gate。 | FD-PG-007 |
| Non-Goal | 不读取或修改 `process/quant-lab/**` 作为本设计任务输入。 | 本轮用户约束 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| Stale Finding | schema、severity、source refs、recommended follow-up。 | follow-up 生命周期管理。 | FEAT-PG-007 | Domain OBJ-PG-015 |
| Project Stale Checker | 跨对象语义陈旧检测。 | CR tracking 结构校验。 | CR tracking / state-router | RULE-PG-011 |
| Release / Formal Docs | 只读观察和 stale finding。 | 自动修改 HLD、TEST-STRATEGY、release docs。 | FEAT-PG-009 / release-readiness | FD-PG-007 |
| Roadmap Refresh Result | 消费 refresh result 作为 stale 输入之一。 | 生成 refresh result。 | FEAT-PG-006 | Dependency Map |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| CLI / checker | 后续 LLD 定位 `meta_flow` 项目命令与 checker 文件。 | 运行项目检查。 | create/modify stale-check checker。 |
| Project objects | `process/state/PROJECT.current.json`、`process/project/ROADMAP.yaml` 等。 | project state / roadmap refs。 | read-only for check。 |
| Design / quality refs | HLD、TEST-STRATEGY、release docs refs。 | 正式设计 / 质量 / 发布文档。 | read-only; no auto-fix。 |
| FU-RF tracking | FEAT-PG-007 tracking store/template。 | follow-up 候选生命周期。 | write candidate through contract。 |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| 数据 | Domain Map 定义 Stale Finding 概念。 | 缺少字段级 schema 和 severity 规则。 | 不复制长文档正文，只保留 refs 和摘要。 |
| 接口 | HLD 给出命令名备选。 | `meta-flow project stale-check` vs `meta-flow check project-stale` 未最终确定。 | LLD 前确认命令名。 |
| 测试 | Feature Matrix 要求 full-lld。 | 缺少跨对象 fixture 和 false positive 控制。 | 不读取/修改 process/quant-lab。 |
| 运维 / 发布 | stale-check 可能命中 release docs。 | 需要 no-auto-fix guard。 | 只输出 finding / FU-RF。 |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| CLI 命名 | 默认 `meta-flow project stale-check`，备选 `meta-flow check project-stale`。 | 与 project 子域一致，HLD 推荐项。 | 最终命名需 LLD 前确认。 |
| Stale Finding | 输出 `finding_id`、`object_ref`、`expected_semantic`、`observed_semantic`、`source_refs`、`severity`、`recommended_route`、`follow_up_candidate_ref`。 | 足够定位陈旧原因和后续处理。 | 需要语义摘要约束。 |
| 规则集 | 第一批只覆盖 roadmap/project phase、HLD/test/release refs、refresh result doc impacts。 | 避免一开始规则过宽。 | 后续规则需版本化。 |
| Guard | 检查器禁止自动修复正式文档。 | 满足安全边界。 | 需要手工或后续 CR 修复。 |
| Follow-up | 严重陈旧项生成 FU-RF/SP-RF/RA-RF 候选。 | 让发现能进入跟踪闭环。 | 依赖 FEAT-PG-007。 |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| DQ-ST-001 | Option A：project stale-check 独立 checker | 边界清楚；不混入 cr-tracking；可针对语义规则测试。 | 新增 checker。 | project-governance、roadmap、quality | 推荐 | 如果只剩结构检查，可合并到已有 check，但当前不适用。 |
| DQ-ST-001 | Option B：扩展 cr-tracking check | 少一个命令。 | CR 生命周期与项目语义陈旧混淆。 | changes、roadmap | 备选 | 仅在 stale rules 全部降级为 CR tracking 结构规则时考虑。 |
| DQ-ST-002 | Option A：只报告 + follow-up | 安全，不越权修改正式文档。 | 修复需要后续 CR。 | security、design、release | 推荐 | 获得独立 CR 和 human gate 后再实现修复。 |
| DQ-ST-002 | Option B：自动修复文档 | 自动化强。 | 未授权修改正式产物，风险高。 | design、quality、release | 备选，不采用 | 未来需单独 CR。 |

## 模块变更

| Module | 变更 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| stale-check CLI | 读取 project/roadmap/design/quality refs，运行规则集。 | project refs、roadmap refs、optional refresh result refs | stale-check result / findings | 输入缺失 -> BLOCKED finding。 |
| stale rule engine | 比较 expected vs observed semantic。 | normalized refs and summaries | stale findings | 规则无法判定 -> WARN/needs-human-review。 |
| finding reporter | 输出机器结果和人类摘要。 | findings | result / summary | finding schema invalid -> checker FAIL。 |
| FU-RF adapter | 将 actionable finding 转候选。 | stale finding | FU-RF/SP-RF/RA-RF candidate | FEAT-PG-007 不可用 -> 保持 stale finding。 |
| no-auto-fix guard | 防止对正式文档执行写入。 | target refs | security PASS/FAIL | 检测到 write action -> FAIL。 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| Stale Check Result | FEAT-PG-008 | `schema_version`、`result_id`、`source_refs`、`finding_count`、`findings`、`follow_up_candidate_refs`、`errors`。 | pass / warn / blocked / failed | 新 result 类型，不影响 CP result。 |
| Stale Finding | FEAT-PG-008 | `finding_id`、`object_ref`、`expected_semantic`、`observed_semantic`、`severity`、`source_refs`、`recommended_route`、`status`。 | open -> converted / waived | 可被 FEAT-PG-007 消费。 |
| Stale Rule | FEAT-PG-008 | `rule_id`、`inputs`、`expected`、`observed_resolver`、`severity_default`。 | active / disabled | 规则版本化，降低误报。 |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| IF-ST-01 | host-orchestrator / CLI | project stale-check | project_ref、roadmap_ref、optional hld/test/release refs、optional refresh_result_ref | stale-check result path、summary path | missing critical refs -> blocked finding。 |
| IF-ST-02 | stale-check | FU-RF tracking | actionable finding with recommended_route | candidate ref or none | tracking unavailable -> keep finding。 |
| IF-ST-03 | QA / CP precheck | stale-check result checker | result path | PASS/WARN/FAIL | invalid finding schema -> FAIL。 |
| IF-ST-04 | stale-check | formal docs / release docs refs | read-only path refs and summaries | observed semantic | write requested -> FAIL security guard。 |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| FLOW-ST-01 | roadmap refresh 后或 CP5/CP7 前运行 stale-check。 | 收集 refs -> 运行规则 -> 输出 findings -> 生成 follow-up candidates。 | refs 缺失时输出 BLOCKED finding，不读取全文默认外对象。 | stale-check result。 | finding_count、blocked_count。 |
| FLOW-ST-02 | 检测到正式文档陈旧。 | 记录 expected/observed -> recommended_route=FU-RF 或 formal CR。 | 不自动修改正式文档。 | finding status=open。 | no-auto-fix guard。 |
| FLOW-ST-03 | 规则无法确定 observed semantic。 | 输出 needs-human-review finding。 | 不猜测、不改文档。 | severity=warning 或 blocked。 | manual review refs。 |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| DQ-ST-001 | LLD 前确定命令名。 | user / host-orchestrator | `meta-flow project stale-check`。 | `meta-flow check project-stale`。 | 不授权实现。 |
| DQ-ST-002 | Finding 建议修改正式文档。 | user / PO | 转 FU-RF 或正式 CR。 | waive 并记录理由。 | stale-check 不自动修复。 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| F-ST-01 | 缺少 project_ref 或 roadmap_ref。 | result includes blocked finding。 | stale-check 不判定 PASS。 | 补 project objects 后重跑。 | TEST-PLAN SEC-ST-03 |
| F-ST-02 | 规则误把 CR tracking 结构问题当 stale。 | checker 应拒绝或标记 rule boundary violation。 | 降低噪音。 | 交给 CR tracking。 | TEST-PLAN UNIT-ST-04 |
| F-ST-03 | 尝试自动修改 HLD/TEST/release docs。 | no-auto-fix guard FAIL。 | 阻断写入。 | 转 FU-RF/CR。 | TEST-PLAN SEC-ST-01 |
| F-ST-04 | FU-RF tracking 不可用。 | 保留 stale finding，不丢失。 | 后续无法自动进入 follow-up query。 | 待 FEAT-PG-007 可用后转换。 | TEST-PLAN INTEG-ST-03 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| SEC-ST-01 | stale-check 只读正式 docs / release docs refs。 | checker 产生 write action。 | FAIL。 | TEST-PLAN SEC-ST-01 |
| SEC-ST-02 | stale-check 不重复 cr-tracking 结构一致性。 | rule target 是 CR 编号/状态结构。 | boundary violation。 | TEST-PLAN UNIT-ST-04 |
| SEC-ST-03 | 缺少关键 refs 时不猜测。 | project/roadmap refs missing。 | blocked finding。 | TEST-PLAN SEC-ST-03 |
| SEC-ST-04 | 不读取或修改 `process/quant-lab/**` 作为本设计任务输入。 | 本轮 feature design 编写。 | 不访问该路径。 | manual review。 |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| Stale rule engine | unit | phase mismatch、roadmap/test strategy mismatch、doc impact mismatch。 | rule fixtures。 | 语义期望需人工抽样。 |
| Stale result schema | contract | result/finding required fields。 | schema fixtures。 | 无。 |
| No-auto-fix guard | security | write action to formal docs blocked。 | security fixture。 | 无。 |
| FU-RF integration | integration | actionable finding -> follow-up candidate。 | fixture with FEAT-PG-007 contract。 | 无。 |
| Command UX | manual | 命令名和输出摘要可理解。 | CP5 / manual review。 | 命名需人工确认。 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 确认 stale-check CLI 命名和输入 refs。 | Q-PG-003 resolution。 | CLI contract。 | MAN-ST-01。 |
| 2 | 定义 stale-check result / finding schema。 | Step 1。 | schema contract。 | CONTRACT-ST-01。 |
| 3 | 定义首批跨对象语义规则。 | project/roadmap refs 明确。 | rule set。 | UNIT-ST-01..04。 |
| 4 | 定义 no-auto-fix guard 和 forbidden write tests。 | formal docs refs 明确。 | security guard。 | SEC-ST-01。 |
| 5 | 定义 FU-RF adapter 和 degraded behavior。 | FEAT-PG-007 contract。 | adapter contract。 | INTEG-ST-03。 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR-G-S02 project stale-check | `process/docs/features/project-stale-check/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | cross-object-semantics / follow_up_tracking / security | CLI 命名、result schema、rule engine、no-auto-fix guard、FU-RF integration。 | Feature DESIGN / TEST-PLAN / TASKS |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| roadmap refresh | refresh 后 | stale-check 可消费 refresh result refs。 | stale findings / follow-up candidates。 | refresh result 缺失时只跑 project refs 规则。 |
| FU-RF tracking | finding actionable 时 | finding with recommended_route。 | candidate ref。 | tracking unavailable 时保留 finding。 |
| story-manager | CP4 前 | full-lld policy 和 Story refs。 | CR-G-S02 Story card。 | 缺 Feature refs 则 CP4 FAIL。 |
| meta-qa | CP7 / CP8 | TEST-PLAN、rule fixtures、manual semantic checklist。 | verification report。 | 语义无法自动判定时人工抽样。 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| R-ST-01 | stale rules 误报过多。 | 用户忽略 findings。 | 首批规则保守，severity 分级，人工抽样。 | 将规则降级为 warning 或 disabled。 |
| R-ST-02 | stale-check 与 cr-tracking 重叠。 | checker 噪音和职责冲突。 | 明确 RULE-PG-011 和 boundary tests。 | 将结构检查移回 cr-tracking。 |
| R-ST-03 | 自动修复正式文档越权。 | 未授权修改长期产物。 | no-auto-fix guard。 | 回滚写入，改为 FU-RF/CR。 |
| R-ST-04 | 命令命名未确认。 | LLD / docs 不一致。 | CP5 前确认 Q-PG-003。 | 采用默认 `meta-flow project stale-check` 并记录切换条件。 |
