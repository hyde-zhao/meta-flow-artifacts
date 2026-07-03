---
status: draft
version: "1.0"
feature_id: "FEAT-PG-007"
feature_name: "Roadmap Follow-up Tracking"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR-G-S01 FU-RF tracking support"
lld_policy_summary: "technical-note by default; upgrade to full-lld if tracking touches multiple modules, adds schema, or changes release context guardrails."
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Roadmap Follow-up Tracking

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 定义 FU-RF / SP-RF / RA-RF roadmap follow-up candidate 的 tracking 边界，明确不进入 RELEASE-CONTEXT |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 让 roadmap refresh 产生的后续事项进入 CR tracking / follow-up 生命周期，而不是混入 release readiness 上下文。 |
| 推荐方案 | 使用 `FU-RF`、`SP-RF`、`RA-RF` 作为 roadmap refresh follow-up 编号族，扩展 tracking regex、模板和状态查询；禁止写入 `RELEASE-CONTEXT`。 |
| 关键取舍 | 保持 release context 语义纯净，代价是 CR tracking 需要识别新的 follow-up 前缀。 |
| 下游 Story | CR-G-S01 |
| LLD 策略 | 默认 technical-note；若改动跨多个 tracking 模块、引入 schema 或 release context guardrail，则升级 full-lld。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-FU-01 | ADR-PG-006 已确认 roadmap follow-up 使用 FU-RF / SP-RF / RA-RF，不写 RELEASE-CONTEXT。 | refresh 输出 stale/follow_up_candidates。 | tracking regex、模板和状态查询需识别新编号。 | follow-up 消失在散落 result 中，或误入 release context。 |
| P-FU-02 | Roadmap follow-up 不是 release readiness evidence。 | 发布库文档陈旧、project state 与 release docs 不一致。 | 需要区分项目治理 follow-up 与发布上下文。 | RELEASE-CONTEXT 被项目治理待办污染。 |
| P-FU-03 | Feature Matrix 将 FEAT-PG-007 判为 required 但低风险可 technical-note。 | CP5 前 Story 卡片需要 feature refs 和 lld_policy。 | 需要清楚升级 full-lld 的触发条件。 | 低估多模块 tracking 改动，CP5 证据不足。 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| Blueprint | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md` | FEAT-PG-007 负责 FU-RF / SP-RF / RA-RF 编号、tracking regex、模板、状态查询。 |
| Domain Map | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DOMAIN-MAP.md` | OBJ-PG-014 FU-RF Candidate、RULE-PG-010 不写 RELEASE-CONTEXT。 |
| Dependency Map | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DEPENDENCY-MAP.md` | FEAT-PG-006 -> FEAT-PG-007 write allowed；FD-PG-006 禁止写 RELEASE-CONTEXT。 |
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | ADR-PG-006、HLD §17 technical-note 或 full-lld 条件。 |
| ADR | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md` | ADR-PG-006 roadmap follow-up tracking 边界。 |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-007 required；CR-G-S01 technical-note，触面扩大则 full-lld。 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 定义 `FU-RF`、`SP-RF`、`RA-RF` 编号族和最小字段契约。 | ADR-PG-006 / Domain OBJ-PG-014 |
| Goal | 更新 CR tracking regex、模板、状态查询逻辑，使 roadmap refresh follow-up 可被发现、列出和关闭。 | Blueprint FEAT-PG-007 |
| Goal | 提供 release context guardrail，证明 FU-RF/SP-RF/RA-RF 不进入 `RELEASE-CONTEXT`。 | RULE-PG-010 |
| Non-Goal | 不把 roadmap follow-up 作为 release readiness evidence 或 release risk。 | ADR-PG-006 |
| Non-Goal | 不自动创建正式 CR，也不自动修改发布库文档。 | ADR-PG-003 |
| Non-Goal | 不重新定义 ROADMAP-REFRESH result schema；只消费 FEAT-PG-006 输出。 | Dependency Map |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| FU-RF Candidate | 编号、字段、模板、状态生命周期。 | stale-check 规则本身。 | FEAT-PG-008 | Domain OBJ-PG-014 |
| SP-RF / RA-RF Candidate | 对 Spike / Risk Acceptance 类 roadmap follow-up 给出编号族。 | 批准风险接受或执行 Spike。 | host-orchestrator / CR tracking | ADR-PG-006 |
| CR Tracking | regex、query、status summary、模板入口。 | ROADMAP-REFRESH result 生成。 | FEAT-PG-006 | Blueprint FEAT-PG-007 |
| RELEASE-CONTEXT | 提供禁止写入规则和测试。 | release readiness 正常证据管理。 | release-readiness skill | FD-PG-006 |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| CR tracking | 后续 LLD 定位 `meta_flow` 中 CR / follow-up tracking 模块。 | 识别 CR / follow-up 状态。 | modify regex/query/template。 |
| Templates | `process/changes/` 相关模板或 skill 模板。 | 生成 follow-up / CR 记录。 | add FU-RF/SP-RF/RA-RF template section。 |
| Release context | `process/release/RELEASE-CONTEXT.yaml` 及 release-readiness 输入。 | 发布上下文证据。 | no write；增加 guardrail / tests。 |
| Roadmap refresh result | FEAT-PG-006 输出路径。 | 提供 follow_up_candidates。 | read-only consumer。 |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| 数据 | HLD 定义 FU-RF / SP-RF / RA-RF 方向。 | 缺少字段级 tracking entry。 | 不能写 RELEASE-CONTEXT。 |
| 接口 | roadmap refresh 将输出 follow_up_candidates。 | 缺少 CR tracking 消费契约。 | 不自动创建正式 CR。 |
| 测试 | Feature Matrix 要求至少 technical-note。 | 缺少 regex、template、release-context guardrail 测试。 | 触面扩大升级 full-lld。 |
| 运维 / 发布 | release context 是发布准备输入。 | 需要证明 roadmap follow-up 不污染 release context。 | release readiness 不应消费 FU-RF 作为 release evidence。 |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| 编号族 | `FU-RF` 表示 follow-up，`SP-RF` 表示 Spike，`RA-RF` 表示 Risk Acceptance。 | 将 roadmap refresh 后续事项与普通 CR / release context 区分。 | tracking regex 需要扩展。 |
| 最小字段 | `id`、`source_result_ref`、`source_stale_item_ref`、`type`、`status`、`recommended_route`、`impact`、`owner`、`created_at`。 | 足够追踪来源和后续路径，不复制 result 全文。 | 需要模板校验。 |
| 状态 | `open`、`accepted`、`converted_to_cr`、`waived`、`closed`。 | 支持 follow-up 生命周期闭环。 | 需要状态查询更新。 |
| RELEASE-CONTEXT guardrail | 明确禁止 FU-RF/SP-RF/RA-RF 写入 release context；release context 只可引用已转换的正式 CR 或 release evidence。 | 保持发布上下文语义纯净。 | 需要新增 guardrail 测试。 |
| LLD 分级 | CR-G-S01 默认 technical-note；跨 3+ tracking 模块、新增 schema 或 guardrail 变更时 full-lld。 | 控制设计成本，同时保留升级条件。 | CP5 需检查触发条件。 |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| DQ-FU-001 | Option A：FU-RF/SP-RF/RA-RF 独立编号族 | 语义清楚；可被 CR tracking 查询；release context 不污染。 | 需要扩展 regex 和模板。 | follow_up_tracking、changes、roadmap | 推荐 | 保持；若前缀过多，可在 UI 聚合。 |
| DQ-FU-001 | Option B：写普通 follow-up 或 release context | 查询路径少。 | 项目治理待办与发布证据混淆。 | release、changes | 备选，不采用 | 仅当 roadmap refresh 取消独立 follow-up 时考虑。 |
| DQ-FU-002 | Option A：technical-note 默认 | 成本低，适合 regex/template 小改。 | 容易低估多模块影响。 | story-planning | 推荐 | 触及多个 tracking 模块或 release guardrail 时升级 full-lld。 |
| DQ-FU-002 | Option B：直接 full-lld | 设计更稳。 | 对低风险改动成本偏高。 | CP5 design evidence | 备选 | 若实现面超过 3 个模块，自动切换。 |

## 模块变更

| Module | 变更 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| tracking regex | 识别 `FU-RF`、`SP-RF`、`RA-RF`。 | candidate id / text refs | matched candidate | 格式不合法 -> tracking FAIL。 |
| follow-up template | 增加 roadmap follow-up entry 模板。 | source_result_ref、stale item | candidate record | 缺 source refs -> invalid candidate。 |
| status query | 列出 open/accepted/converted/waived/closed candidates。 | tracking store / process refs | status summary | unknown status -> WARN/FAIL。 |
| release context guardrail | 检测 RELEASE-CONTEXT 是否包含 FU-RF/SP-RF/RA-RF。 | release context path | PASS/FAIL | 命中前缀 -> FAIL。 |
| roadmap refresh consumer | 从 result `follow_up_candidates` 读取候选项。 | ROADMAP-REFRESH result | tracking entries | result 不合法 -> 不写 tracking。 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| Roadmap Follow-up Candidate | FEAT-PG-007 | `id`、`prefix`、`type`、`status`、`source_result_ref`、`source_stale_item_ref`、`recommended_route`、`impact`、`owner`。 | open -> accepted -> converted_to_cr / waived / closed | 新增对象，不改变 release context schema。 |
| Prefix Regex | FEAT-PG-007 | `FU-RF[0-9]+-*`、`SP-RF[0-9]+-*`、`RA-RF[0-9]+-*` 的最终格式由 LLD 固化。 | active | 兼容普通 CR 编号，不覆盖 CR-037。 |
| Release Context Guardrail | FEAT-PG-007 / release-readiness consumer | forbidden refs: FU-RF/SP-RF/RA-RF | pass / fail | release context 只能引用转换后的正式 CR 或 release evidence。 |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| IF-FU-01 | roadmap refresh | follow-up tracking | `follow_up_candidates[*]` with prefix, source_result_ref, recommended_route | candidate entry refs | invalid prefix / missing source -> FAIL。 |
| IF-FU-02 | host-orchestrator / CLI | status query | candidate id or project scope | grouped candidate status | unknown status -> WARN or FAIL by mode。 |
| IF-FU-03 | release-readiness guardrail | release context checker | RELEASE-CONTEXT refs | PASS if no FU-RF/SP-RF/RA-RF | forbidden prefix -> FAIL。 |
| IF-FU-04 | story-manager | Feature design evidence | lld_policy trigger conditions | technical-note or full-lld decision | touch-surface expansion -> require full-lld。 |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| FLOW-FU-01 | ROADMAP-REFRESH result 包含 follow_up_candidates。 | 校验 prefix/source refs -> 写 tracking entry -> status=open -> 状态查询可见。 | prefix 不合法或 source 缺失时拒绝写入。 | FU-RF/SP-RF/RA-RF candidate。 | candidate count、invalid count。 |
| FLOW-FU-02 | 用户接受 follow-up。 | candidate open -> accepted -> converted_to_cr 或 closed。 | 需要正式变更时不直接修改发布库，创建/路由 CR。 | status summary 更新。 | conversion ref。 |
| FLOW-FU-03 | release readiness 读取 RELEASE-CONTEXT。 | guardrail 扫描 forbidden prefixes。 | 命中 FU-RF/SP-RF/RA-RF 则 FAIL。 | release context 保持纯净。 | guardrail finding。 |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| DQ-FU-001 | CP5 审查 CR-G-S01 设计策略。 | host-orchestrator / user | technical-note 默认，触面扩大即 full-lld。 | 直接 full-lld。 | 不授权实现或 CP5 通过。 |
| DQ-FU-002 | FU-RF 需要转正式 CR。 | user / host-orchestrator | 创建正式 CR 并进入对应门控。 | waive 并记录理由。 | 不自动修改发布库。 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| F-FU-01 | FU-RF/SP-RF/RA-RF 缺少 source_result_ref。 | tracking entry invalid。 | follow-up 不进入状态查询。 | 修复 source refs 后重试。 | TEST-PLAN CONTRACT-FU-01 |
| F-FU-02 | RELEASE-CONTEXT 包含 FU-RF/SP-RF/RA-RF。 | guardrail FAIL。 | release readiness 阻断或要求清理。 | 移出 release context，保留 tracking entry。 | TEST-PLAN SEC-FU-01 |
| F-FU-03 | 改动触及多个 tracking 模块但仍只写 technical-note。 | CP5 预检应要求升级 full-lld。 | 设计证据不足。 | 升级 full-lld。 | TEST-PLAN MAN-FU-03 |
| F-FU-04 | follow-up 被错误转换为发布风险。 | 状态查询语义混乱。 | release context 污染。 | 转回 FU-RF tracking，必要时创建正式 CR。 | TEST-PLAN INTEG-FU-03 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| SEC-FU-01 | FU-RF/SP-RF/RA-RF 不得写入 `RELEASE-CONTEXT`。 | release context 包含该前缀。 | guardrail FAIL。 | TEST-PLAN SEC-FU-01 |
| SEC-FU-02 | Follow-up tracking 不自动修改发布库。 | candidate recommended_route 指向 release docs/code/tests。 | 只能转正式 CR 或 stale item。 | TEST-PLAN SEC-FU-02 |
| SEC-FU-03 | Candidate 必须保留来源 result/stale item refs。 | tracking entry 缺来源。 | invalid candidate。 | TEST-PLAN CONTRACT-FU-01 |
| SEC-FU-04 | Risk acceptance 类 RA-RF 不等于批准风险。 | RA-RF created。 | 仍需 human gate 或正式 risk decision。 | TEST-PLAN MAN-FU-02 |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| Prefix regex | unit | FU-RF/SP-RF/RA-RF 正反例。 | regex tests。 | 无。 |
| Template fields | contract | source refs、status、recommended route。 | fixture validation。 | 无。 |
| Status query | integration | open -> converted_to_cr / waived / closed。 | fixture tracking store。 | 无。 |
| RELEASE-CONTEXT guardrail | security / integration | forbidden prefix 命中 FAIL。 | release context fixture。 | 无。 |
| LLD upgrade decision | manual | touch 面扩大是否升级 full-lld。 | CP5 checklist。 | 需要人工审查文件影响面。 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 固化 prefix regex 和 candidate 最小字段。 | ADR-PG-006 已批准。 | tracking contract。 | TEST-PLAN UNIT-FU-01。 |
| 2 | 设计 template 和 status query 更新。 | Step 1。 | template / query design。 | TEST-PLAN INTEG-FU-01。 |
| 3 | 设计 RELEASE-CONTEXT guardrail。 | Step 1。 | guardrail contract。 | TEST-PLAN SEC-FU-01。 |
| 4 | 设计 roadmap refresh result 消费契约。 | FEAT-PG-006 result schema。 | consumer contract。 | TEST-PLAN CONTRACT-FU-01。 |
| 5 | CP5 前判定 CR-G-S01 是否保持 technical-note 或升级 full-lld。 | 文件影响面确认。 | lld_policy decision。 | MAN-FU-03。 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR-G-S01 FU-RF tracking support | `process/docs/features/roadmap-follow-up-tracking/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | technical-note by default; full-lld if expanded | cross-module-contract / low-runtime-risk / release-context-guardrail | prefix regex、template fields、status lifecycle、release-context guardrail。 | Feature DESIGN / TEST-PLAN / TASKS，Story 技术说明或 full LLD。 |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| roadmap refresh | refresh result 生成后 | `follow_up_candidates` 使用 FU-RF/SP-RF/RA-RF 契约。 | tracking entries 可查询。 | invalid candidate 留在 result errors，不写 tracking。 |
| release-readiness | CP8 / release context 生成前 | RELEASE-CONTEXT 不含 FU-RF/SP-RF/RA-RF。 | guardrail PASS。 | 命中则移出 release context，转 tracking。 |
| story-manager | CP4 前 | Story refs 和 lld_policy。 | CR-G-S01 标记 technical-note / upgrade condition。 | 触面扩大时 CP5 要求 full-lld。 |
| lld-designer | CP5 前 | 本 DESIGN、TEST-PLAN、TASKS。 | Story 技术说明或 full LLD。 | 证据不足时 clarification。 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| R-FU-01 | FU-RF/SP-RF/RA-RF 前缀过多，用户难以检索。 | follow-up UX 下降。 | 状态查询按 roadmap-refresh 分组聚合。 | 保留前缀，优化展示层。 |
| R-FU-02 | FU-RF 被误写入 RELEASE-CONTEXT。 | 发布上下文污染。 | release context guardrail。 | 清理 release context，保留 tracking entry。 |
| R-FU-03 | technical-note 低估影响面。 | CP5 设计证据不足。 | 明确 full-lld 升级条件。 | 升级 full-lld。 |
| R-FU-04 | RA-RF 被误认为风险已批准。 | 风险接受绕过 human gate。 | RA-RF 只是候选，必须转正式 decision。 | 撤销 RA-RF 状态，补人工决策。 |
