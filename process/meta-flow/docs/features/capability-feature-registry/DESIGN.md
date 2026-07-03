---
status: draft
version: "1.0"
feature_id: "FEAT-PG-004"
feature_name: "Capability / Feature Registry"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR-D-S01 feature/capability registry and resolver"
lld_policy_summary: "full-lld because registry IDs become shared contracts for impact normalization, migration reports, roadmap refresh, stale-check, and external sample mapping."
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Capability / Feature Registry

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 基于 CR-037 CP3 已批准推荐方案生成 FEAT-PG-004 长期 baseline |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 建立机器可校验的 feature/capability registry，禁止 `feature_refs` / `capability_refs` 使用自由字符串。 |
| 推荐方案 | 使用 YAML registry 作为真相源，提供 resolver/checker，将 unresolved refs 输出为 blocked finding；不从 Markdown、Python 常量或迁移样本中隐式创造 ID。 |
| 关键取舍 | 需要初始化和维护 registry，但换取 migration、impact normalization 和 roadmap refresh 的稳定引用。 |
| 下游 Story | CR-D-S01 |
| LLD 策略 | `full-lld`，因为这是多 Feature 共享数据契约和 resolver 接口。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-CFR-01 | HLD AGA-PG-004 已确认 refs 必须引用标准 registry。 | impact migration、roadmap refresh、stale-check 需要解析能力/Feature。 | 自由字符串会导致冲突、重复和不可追踪。 | migration 输出不可信，后续 checker 只能做弱匹配。 |
| P-CFR-02 | ADR-PG-004 明确不长期消费 Markdown/Python 常量。 | 需要跨模块共享 ID 和状态。 | 需要定义 registry schema、source priority 和 resolver 错误模型。 | 每个模块自建 ID 口径，出现 contract drift。 |
| P-CFR-03 | 真实样本映射必须安全处理未注册能力。 | migration 遇到未知 capability。 | 不能自动创造 ID 或写发布库。 | 未确认能力被写入正式状态，后续审计失真。 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | UC-PG-004、SIM-PG-003、R-PG-003、§17 FEAT-PG-004 required。 |
| ADR | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md` | ADR-PG-004：refs 必须引用 YAML registry，不退回自由字符串。 |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-004 required；CR-D-S01 `full-lld`。 |
| CP3 决策 | CR-037 已批准的推荐方案 | registry-backed refs 位于 impact normalization 和迁移之前。 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 定义 feature registry 和 capability registry 的 YAML schema、ID policy、状态字段和 owner 字段。 | ADR-PG-004 |
| Goal | 提供 resolver，将 refs 解析为 resolved / unresolved / deprecated / conflict findings。 | HLD SIM-PG-003 |
| Goal | 支持 impact normalization、roadmap refresh、stale-check、migration readiness 读取同一 registry。 | HLD §9 |
| Goal | 对未知 refs 输出 blocked finding，不自动创造 ID。 | HLD §6 UC-PG-004 |
| Non-Goal | 不实现具体 impact field split。 | FEAT-PG-005 |
| Non-Goal | 不执行真实项目迁移，也不读取或修改外部项目内容。 | FEAT-PG-009 |
| Non-Goal | 不把 registry 存为 Markdown 表格或 Python 常量真相源。 | ADR-PG-004 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| feature registry | Feature ID、名称、状态、owner、source refs、aliases、deprecation。 | Story backlog 生成。 | story-manager / Feature Matrix | ADR-PG-004 |
| capability registry | Capability ID、领域、状态、source refs、映射说明。 | capability 的业务实现。 | FEAT-PG-005 / FEAT-PG-009 | HLD UC-PG-004 |
| resolver | ref 解析、重复检测、unknown/deprecated finding。 | 自动修正未知 refs。 | impact migration / roadmap refresh | HLD SIM-PG-003 |
| registry checker | schema 和引用完整性。 | CP result checker。 | checkpoint-manager | HLD §10 |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| registry data | `process/project/` 或 `process/registry/` 下的 YAML registry | 尚未冻结机器真相源。 | create registry YAML files in Story LLD |
| resolver | `meta_flow/` checker/resolver 模块 | 尚未提供共享 ref resolver。 | create shared resolver library |
| CLI / checker | `meta_flow/checks/` 或等价命令入口 | 校验 registry 和 refs。 | add registry check command |
| tests | `tests/` | 覆盖 schema、resolver、contract。 | add positive/negative fixtures |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| 数据 | HLD 和 Feature Matrix 只有 registry 需求。 | 缺少 YAML schema、ID 命名策略、状态流。 | registry 是机器真相源。 |
| 接口 | 下游 feature_refs/capability_refs 没有统一解析入口。 | 需要 resolver API 和 finding 模型。 | unknown refs blocked，不自动创建 ID。 |
| 测试 | 需要覆盖 unresolved / deprecated / duplicate。 | 缺少 fixtures。 | 不读取 `process/quant-lab/**`。 |
| 运维 / 发布 | registry 是过程治理对象。 | 需要兼容 aliases 和 deprecation。 | 不跨仓写发布库。 |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| Registry 格式 | YAML registry，区分 `features[]` 和 `capabilities[]`。 | 机器可解析、可 review、适合 process artifact。 | 需要 schema 和格式规范。 |
| ID policy | `FEAT-*` / `CAP-*` 稳定 ID，允许 aliases 但 canonical ID 唯一。 | 支持重命名和迁移。 | resolver 需要处理 alias/deprecated。 |
| Resolver 行为 | 返回 typed result：resolved、unresolved、deprecated、duplicate。 | 下游可以用统一错误模型。 | 需要统一 finding severity。 |
| Source priority | registry > explicit alias > migration hint；不得从样本自由创造 ID。 | 避免样本污染 canonical registry。 | 初始化 registry 时需要人工补缺。 |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| CFR-DQ-001 | YAML registry 为真相源 | 可机器校验、review 友好、适合迁移。 | 需要维护 schema。 | registry / migration / impact / roadmap | 推荐 | 若 registry 规模极大，再评估拆分文件，不改变 YAML 真相源原则。 |
| CFR-DQ-001 | Markdown 表格为真相源 | 人类可读。 | 机器解析脆弱，容易漂移。 | docs / migration | 备选但不采用 | 仅作为渲染摘要，不作真相源。 |
| CFR-DQ-002 | unknown refs blocked | 防止自由创造 ID。 | 迁移初期会产生较多 blocked finding。 | migration / checker | 推荐 | 若 blocked 过多，先扩 registry 或写 aliases，不放宽为自由字符串。 |
| CFR-DQ-002 | unknown refs 自动创建候选 ID | 快速迁移。 | 可信度低，审计差。 | registry / audit | 备选但不采用 | 仅允许生成 candidate report，不写 canonical registry。 |

## 模块变更

| Module | 变更 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| registry schema | 新增 feature/capability YAML schema。 | registry files。 | schema validation result。 | duplicate ID、非法状态、缺 owner/source refs 时 fail。 |
| registry resolver | 新增 shared resolver API。 | `feature_refs`、`capability_refs`、registry snapshot。 | resolved refs + findings。 | unknown/deprecated/conflict 返回 blocked 或 warning。 |
| registry checker | 新增 registry 自检和 ref 检查。 | registry + consumer refs。 | PASS/WARN/ERROR。 | source priority 冲突时 fail。 |
| migration integration | 为 impact migration 提供解析服务。 | legacy impact hints。 | canonical refs 或 blocked finding。 | 不自动创建 ID。 |
| roadmap/stale integration | 为 refresh/stale 提供 stable IDs。 | roadmap items、stale candidates。 | canonical refs。 | registry invalid 时下游 blocked。 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| `FEATURE-REGISTRY.yaml` | capability-feature-registry | `features[].id`、`name`、`status`、`owner`、`source_refs`、`aliases`、`depends_on`、`deprecated_by`。 | proposed -> active -> deprecated。 | aliases 解析到 canonical ID；deprecated 输出 warning/error 取决于模式。 |
| `CAPABILITY-REGISTRY.yaml` | capability-feature-registry | `capabilities[].id`、`name`、`domain`、`status`、`owner`、`source_refs`、`feature_refs`、`aliases`。 | proposed -> active -> deprecated。 | capability 可映射多个 feature，但必须引用已注册 feature。 |
| Resolver result | capability-feature-registry | `input_ref`、`canonical_id`、`kind`、`status`、`severity`、`message`、`source`。 | resolved / unresolved / deprecated / conflict。 | 下游统一消费 finding 模型。 |
| Consumer refs | FEAT-PG-005 / 006 / 008 / 009 | `feature_refs[]`、`capability_refs[]`。 | free text -> canonical refs。 | 历史对象通过 migration report 处理。 |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| CFR-IF-01 | checker / migration / refresh | registry loader | registry file paths、schema version。 | immutable registry snapshot。 | `E_REGISTRY_MISSING`、`E_REGISTRY_SCHEMA_INVALID`。 |
| CFR-IF-02 | impact migration | resolver | refs or legacy hints、kind=`feature|capability`、mode=`audit|enforce`。 | resolver result list。 | `E_REF_UNRESOLVED`、`E_REF_DEPRECATED`、`E_REF_CONFLICT`。 |
| CFR-IF-03 | registry checker | registry schema | registry YAML。 | PASS/WARN/ERROR findings。 | duplicate ID / alias collision / invalid source refs。 |
| CFR-IF-04 | roadmap/stale checker | resolver | roadmap/stale candidate refs。 | canonical refs or blocked findings。 | unresolved refs block dependent check。 |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| CFR-FLOW-01 | registry check 执行。 | load registry -> schema validate -> ID/alias uniqueness -> source refs validate。 | registry invalid 时所有 consumer checks blocked。 | registry PASS/WARN/ERROR。 | registry check result。 |
| CFR-FLOW-02 | impact migration 解析 refs。 | legacy hint -> resolver -> canonical ref -> migration report。 | unresolved 输出 blocked finding，不写 canonical field。 | resolved refs 或 blocked finding。 | migration report。 |
| CFR-FLOW-03 | 下游 checker 消费 registry。 | consumer refs -> resolver -> typed result -> downstream validation。 | deprecated 按模式 WARN/ERROR。 | stable refs for downstream。 | checker findings。 |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| CFR-HG-001 | 初始化 registry 时出现未归属 capability。 | 用户 / SE / domain owner。 | 写入 candidate report，人工确认后加入 registry。 | 暂时 blocking，不加入。 | 不授权自动创建 canonical ID。 |
| CFR-HG-002 | alias 冲突指向多个 canonical ID。 | SE / implementation owner。 | fail 并人工合并或改名。 | 临时禁用 alias。 | 不授权 resolver 猜测。 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| CFR-F-01 | registry 文件缺失或 schema invalid。 | registry checker ERROR，下游 resolver blocked。 | impact/refresh/stale 无法继续。 | 创建或修复 registry。 | TEST-PLAN CFR-SEC-01 / CFR-INT-01 |
| CFR-F-02 | ref 未注册。 | resolver 返回 unresolved blocked finding。 | migration 不写 canonical refs。 | 人工确认并补 registry 或记录 follow-up。 | TEST-PLAN CFR-UNIT-03 |
| CFR-F-03 | alias 冲突。 | checker ERROR。 | 下游解析停止。 | 修改 alias 或 canonical ID。 | TEST-PLAN CFR-UNIT-04 |
| CFR-F-04 | deprecated ref 被新对象使用。 | audit WARN / enforce ERROR。 | 新 CR 或新 roadmap item 被阻断。 | 改用 `deprecated_by` 指向的新 ID。 | TEST-PLAN CFR-CON-02 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| CFR-SEC-01 | resolver 不得从自由字符串自动创建 canonical ID。 | unresolved input。 | blocked finding。 | TEST-PLAN CFR-SEC-01 |
| CFR-SEC-02 | registry 不得包含 credential、private token 或生产数据样本。 | schema/checker 扫描 source refs / notes。 | ERROR。 | TEST-PLAN CFR-SEC-02 |
| CFR-SEC-03 | registry source priority 固定为 canonical registry 优先。 | migration hint 与 registry 冲突。 | registry wins，输出 conflict finding。 | TEST-PLAN CFR-SEC-03 |
| CFR-SEC-04 | 外部样本只能作为 hint，不得直接写 registry。 | migration readiness 消费样本。 | 生成 candidate report，不写 canonical。 | TEST-PLAN CFR-SEC-04 |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| registry schema | unit | valid、missing fields、duplicate ID、alias collision。 | schema unit tests。 | 无。 |
| resolver | unit | resolved、unresolved、deprecated、conflict。 | resolver fixture tests。 | 无。 |
| downstream contract | integration / contract | impact migration 和 roadmap/stale 通过同一 resolver。 | contract tests with synthetic consumers。 | 无。 |
| security failure | security | free string auto-create、credential-like field、source conflict。 | negative fixtures。 | 无。 |
| initial registry review | manual | ID 命名、owner、source refs 是否可维护。 | 人工审查 registry diff。 | 语义归属需要人确认。 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 冻结 registry schema、ID policy、状态枚举和 alias 规则。 | CP5 前 LLD。 | schema / docs。 | CFR-UNIT-01..04。 |
| 2 | 实现 registry loader/checker。 | schema 稳定。 | loader、checker result。 | CFR-INT-01。 |
| 3 | 实现 resolver API 和 finding 模型。 | loader/checker。 | resolver library。 | CFR-UNIT-02..05。 |
| 4 | 接入 impact normalization contract。 | FEAT-PG-005 LLD。 | consumer contract tests。 | CFR-CON-01。 |
| 5 | 接入 roadmap/stale/migration consumer contract。 | downstream LLD。 | shared resolver consumption。 | CFR-CON-02..03。 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR-D-S01 feature/capability registry and resolver | `process/docs/features/capability-feature-registry/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | data-model / external-interface / shared-story-boundary | YAML schema、resolver API、finding severity、consumer contract。 | 本 Feature 设计 + Story LLD。 |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| FEAT-PG-005 impact normalization | CR-E LLD / implementation | resolver API、canonical refs、finding model。 | normalized `feature_refs` / `capability_refs` or blocked finding。 | registry invalid 时 migration report blocked。 |
| FEAT-PG-006 roadmap refresh | W4 LLD / implementation | registry snapshot and resolver。 | roadmap result refs are canonical。 | unresolved refs become stale/follow-up candidates。 |
| FEAT-PG-008 stale-check | W4 LLD / implementation | resolver results。 | stale findings reference canonical IDs。 | checker blocked on invalid registry。 |
| FEAT-PG-009 migration readiness | W5 dry-run | candidate mapping inputs。 | candidate report，不自动写 registry。 | 人工补 registry 后重跑。 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| CFR-R-01 | registry 初始化不完整，导致 blocked findings 多。 | migration 和 impact normalization 进度变慢。 | candidate report + aliases + 分批补 registry。 | 延长 audit，不进入 enforce。 |
| CFR-R-02 | ID 命名随意，长期不可维护。 | 下游 refs 难追踪。 | ID policy、owner/source refs、manual review。 | deprecated + `deprecated_by`，不复用旧 ID。 |
| CFR-R-03 | resolver 被下游绕过。 | refs 再次自由化。 | contract tests、checker enforce。 | CP4/CP5 失败并回修 consumer。 |
| CFR-R-04 | registry 泄露敏感信息。 | 安全事故。 | 禁止 credential / 生产样本字段，negative fixtures。 | 删除敏感字段，轮换凭据按安全流程处理。 |
