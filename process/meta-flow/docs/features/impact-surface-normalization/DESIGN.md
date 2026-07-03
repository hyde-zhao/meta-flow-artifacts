---
status: draft
version: "1.0"
feature_id: "FEAT-PG-005"
feature_name: "Impact Surface Normalization"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR-E-S01 impact surface field split and migration report"
lld_policy_summary: "full-lld because this feature changes CR schema semantics, migration reporting, audit/enforce behavior, and dependency on registry-backed refs."
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Impact Surface Normalization

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 基于 CR-037 CP3 已批准推荐方案生成 FEAT-PG-005 长期 baseline |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 将 CR `impact_surface` 从混合自由文本归一为有限治理面枚举、`affected_paths`、`feature_refs`、`capability_refs` 四类语义字段。 |
| 推荐方案 | 新 CR 使用 normalized schema，历史 CR 不在普通 `cr check` 中刷屏，通过 migration report 聚合旧漂移；unknown surface 在 audit 阶段 WARN、enforce 阶段 ERROR。 |
| 关键取舍 | 增加 migration/report 和兼容逻辑，换取影响面可分析、registry refs 可追踪、历史噪音可控。 |
| 下游 Story | CR-E-S01 |
| LLD 策略 | `full-lld`，因为涉及数据模型迁移、兼容策略、checker 行为和安全失败路径。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-ISN-01 | HLD 成功标准要求 `impact_surface` 拆成有限治理面、路径、feature refs、capability refs。 | CR 创建、CR audit/enforce、migration report。 | 混合字段难以校验和聚合。 | 路径、能力、模块、风险语义混杂，影响分析不可重复。 |
| P-ISN-02 | HLD R-PG-005 指出历史漂移会造成噪音。 | 普通 `cr check` 扫描历史 CR。 | 旧数据可能产生大量 FAIL。 | 检查不可用，团队关闭 enforcement。 |
| P-ISN-03 | FEAT-PG-004 registry 是 refs 真相源。 | `feature_refs` / `capability_refs` 写入。 | 需要 resolver contract。 | unknown capability 被当作合法影响面。 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | UC-PG-005、R-PG-005、§17 FEAT-PG-005 required、§18 impact migration 下沉要求。 |
| ADR | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md` | ADR-PG-004 registry-backed refs。 |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-005 required；CR-E-S01 `full-lld`。 |
| CP3 决策 | CR-037 已批准的推荐方案 | registry-backed refs 和 impact normalization 均为 W3 前置能力。 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 定义 normalized impact schema：`impact_surfaces[]`、`affected_paths[]`、`feature_refs[]`、`capability_refs[]`。 | HLD 成功标准 |
| Goal | 建立有限治理面枚举，至少覆盖 HLD 要求的 14 类治理面。 | HLD 成功标准 |
| Goal | 新 CR audit WARN / enforce ERROR unknown surface；历史 CR 进入 migration report。 | HLD R-PG-005 |
| Goal | `feature_refs` / `capability_refs` 必须通过 FEAT-PG-004 resolver。 | ADR-PG-004 |
| Non-Goal | 不修改 registry schema 本身。 | FEAT-PG-004 |
| Non-Goal | 不自动改写所有历史 CR 正文。 | HLD 兼容性约束 |
| Non-Goal | 不读取或修改外部项目发布库。 | ADR-PG-003 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| CR impact schema | normalized fields、枚举、兼容规则。 | CR 生命周期和审批逻辑。 | change-impact-analysis / issue-routing | HLD UC-PG-005 |
| impact migration report | 历史漂移扫描、分类、blocked finding。 | 自动重写旧 CR。 | checkpoint-manager / host-orchestrator | HLD R-PG-005 |
| registry refs | 调 resolver 校验 feature/capability refs。 | registry canonical ID 创建。 | FEAT-PG-004 | ADR-PG-004 |
| audit/enforce mode | 新 CR unknown surface WARN/ERROR 策略。 | 全局 gate profile 修改。 | CP/checker framework | HLD §12 |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| CR objects | `process/changes/` | 承载 CR 文档和索引输入。 | update schema consumer in Story LLD |
| CR index / checker | `meta_flow/` change/check modules | 校验 CR 字段和影响面。 | add normalized impact checker |
| migration report | `process/checks/` 或 `process/reports/` | 承载自动检查和迁移摘要。 | create impact migration report output |
| registry resolver | FEAT-PG-004 module | 解析 feature/capability refs。 | consume resolver API |
| tests | `tests/` | 覆盖 schema、migration、audit/enforce。 | add fixtures |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| 数据 | `impact_surface` 语义混合。 | 缺少 normalized schema 和 migration report。 | 历史 CR 不应在普通 check 中刷屏。 |
| 接口 | CR checker 未区分治理面、路径和 refs。 | 需要 mode-aware checker 与 resolver contract。 | `feature_refs` / `capability_refs` 必须 canonical。 |
| 测试 | 缺少 legacy/new CR 对比 fixture。 | 需要 audit/enforce、migration、security negative cases。 | 不执行实现。 |
| 运维 / 发布 | migration 是过程侧报告。 | 需要 blocked finding 和人工回修入口。 | 不跨仓写发布库。 |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| 字段拆分 | `impact_surfaces[]` 表示有限治理面；`affected_paths[]` 表示文件路径；`feature_refs[]` / `capability_refs[]` 表示 registry-backed refs。 | 消除混合语义。 | 需要迁移兼容。 |
| 历史处理 | 历史 CR 由 migration report 聚合，不在普通 `cr check` 中大量 FAIL。 | 降低噪音，保留可追踪性。 | 需要单独 report 命令/入口。 |
| 新 CR 策略 | audit WARN、enforce ERROR unknown surface / unresolved refs。 | 支持灰度。 | 需要 mode 参数。 |
| 路径安全 | `affected_paths` 只做路径影响声明，不等同授权写入。 | 避免 impact 字段被误用为权限。 | 需要安全测试和文档说明。 |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| ISN-DQ-001 | 四字段拆分 | 语义清晰，可校验，可统计。 | 迁移成本高。 | changes / registry / checker | 推荐 | 若某类字段过度膨胀，拆子对象但不回退混合字段。 |
| ISN-DQ-001 | 保持单字段，加命名约定 | 改动小。 | 无法可靠校验路径和 capability。 | changes | 备选但不采用 | 仅适合一次性文档，不适合 CR-037。 |
| ISN-DQ-002 | 历史走 migration report | 不刷屏，保留审计。 | 需要单独报告入口。 | migration / audit | 推荐 | 若历史数据很少，可人工迁移后关闭 report。 |
| ISN-DQ-002 | 普通 check 直接 FAIL 所有旧 CR | 简单严格。 | 噪音大，影响日常检查。 | checker UX | 备选但不采用 | 仅在旧 CR 已完成清理后可切换。 |

## 模块变更

| Module | 变更 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| impact schema | 新增 normalized fields 和枚举。 | CR object。 | normalized impact model。 | unknown surface / invalid path / unresolved refs。 |
| CR checker | 支持 audit/enforce mode。 | new CR、mode。 | WARN/ERROR findings。 | enforce unknown -> ERROR。 |
| migration scanner | 扫描 legacy `impact_surface`。 | CR index / CR summaries。 | migration report。 | 无法分类 -> blocked finding。 |
| registry adapter | 调 FEAT-PG-004 resolver。 | feature/capability refs。 | canonical refs 或 blocked finding。 | registry invalid -> blocked。 |
| report writer | 输出 migration report。 | scanner findings。 | report JSON/Markdown summary。 | report path 不可写 -> fail without modifying CR。 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| CR impact block | impact-surface-normalization | `impact_surfaces[]`、`affected_paths[]`、`feature_refs[]`、`capability_refs[]`。 | legacy -> normalized。 | legacy `impact_surface` 可读但新 CR 不推荐写。 |
| Impact surface enum | impact-surface-normalization | 至少包含 `state`、`context`、`project-governance`、`roadmap`、`changes`、`registry`、`quality`、`release`、`security`、`runtime-authorization`、`docs`、`agent-contract`、`workflow`、`migration`。 | baseline -> extend via CR。 | unknown audit WARN / enforce ERROR。 |
| Migration report | impact-surface-normalization | `source_cr`、`legacy_value`、`suggested_surfaces`、`suggested_paths`、`suggested_feature_refs`、`suggested_capability_refs`、`finding_severity`。 | generated -> reviewed -> applied by future CR。 | 报告不等同自动修改。 |
| Checker finding | impact-surface-normalization | `field`、`mode`、`severity`、`message`、`recovery_hint`。 | WARN/ERROR/BLOCKED。 | 与 CP result 可聚合但不复用 CP schema。 |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| ISN-IF-01 | CR checker | impact schema validator | CR object、mode=`audit|enforce`。 | findings + normalized model。 | `E_UNKNOWN_IMPACT_SURFACE`、`E_INVALID_AFFECTED_PATH`。 |
| ISN-IF-02 | impact validator | FEAT-PG-004 resolver | `feature_refs`、`capability_refs`。 | resolver result。 | `E_REF_UNRESOLVED`、`E_REF_DEPRECATED`。 |
| ISN-IF-03 | migration command/checker | migration scanner | CR index、legacy field policy、registry snapshot。 | migration report。 | `E_LEGACY_IMPACT_UNCLASSIFIED`、`E_REGISTRY_BLOCKED`。 |
| ISN-IF-04 | host-orchestrator / reviewer | migration report reader | report path。 | summary counts and blocked findings。 | `E_REPORT_MISSING`。 |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| ISN-FLOW-01 | 新 CR audit。 | parse normalized fields -> enum validate -> resolver validate refs -> WARN findings。 | unknown surface WARN，unresolved refs WARN/BLOCKED 取决于 policy。 | audit result。 | CR checker output。 |
| ISN-FLOW-02 | 新 CR enforce。 | parse -> validate -> resolver -> ERROR on unknown/unresolved。 | ERROR 阻止通过。 | enforce result。 | CP/checker result。 |
| ISN-FLOW-03 | 历史 migration report。 | scan legacy -> classify -> resolver -> write report。 | unclassified 输出 blocked finding，不改源 CR。 | migration report。 | report summary counts。 |
| ISN-FLOW-04 | 人工回修。 | reviewer 审查 report -> 选择是否创建后续 CR 更新旧对象。 | 不确认则保留 report。 | follow-up decision。 | manual review。 |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| ISN-HG-001 | legacy value 无法自动分类。 | SE / user。 | 标记 blocked finding，人工补 surface/path/ref。 | 暂时保留 legacy，不进入 enforce。 | 不授权模型猜测 canonical refs。 |
| ISN-HG-002 | 需要新增 impact surface enum。 | user / architecture owner。 | 走 CR 扩展枚举。 | 映射到 existing surface 并记录 rationale。 | 不授权自由新增 enum。 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| ISN-F-01 | 新 CR 包含 unknown impact surface。 | audit WARN；enforce ERROR。 | CR check 不通过或提示整改。 | 改为已知 enum 或发 CR 扩展 enum。 | TEST-PLAN ISN-SEC-01 |
| ISN-F-02 | `feature_refs` / `capability_refs` unresolved。 | 输出 blocked finding，不写 normalized refs。 | migration 或 CR check blocked。 | 补 registry 或移除 ref。 | TEST-PLAN ISN-CON-01 |
| ISN-F-03 | legacy `impact_surface` 无法分类。 | migration report 标记 blocked。 | 历史迁移不能自动完成。 | 人工分类后重跑。 | TEST-PLAN ISN-INT-03 |
| ISN-F-04 | `affected_paths` 被误认为写授权。 | checker/文档明确 impact 只表影响。 | 需要另走 runtime authorization。 | 移除授权语义，转人工 gate。 | TEST-PLAN ISN-SEC-03 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| ISN-SEC-01 | unknown impact surface 在 enforce 下必须失败。 | mode=`enforce`。 | ERROR。 | TEST-PLAN ISN-SEC-01 |
| ISN-SEC-02 | `feature_refs` / `capability_refs` 必须 resolver resolved。 | refs present。 | unresolved blocked。 | TEST-PLAN ISN-CON-01 |
| ISN-SEC-03 | `affected_paths` 不是写授权，不得包含 approval 语义。 | CR impact block。 | ERROR 或 manual review finding。 | TEST-PLAN ISN-SEC-03 |
| ISN-SEC-04 | migration report 不得自动重写历史 CR。 | migration scanner 完成。 | only report output。 | TEST-PLAN ISN-SEC-04 |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| normalized schema | unit | valid fields、unknown enum、invalid path。 | schema/checker tests。 | 无。 |
| audit/enforce behavior | unit / integration | audit WARN vs enforce ERROR。 | mode tests。 | 无。 |
| registry refs | contract | resolved/unresolved/deprecated refs。 | FEAT-PG-004 resolver contract tests。 | 无。 |
| migration report | integration | legacy value classification、blocked finding、no source rewrite。 | fixture-based report tests。 | 无。 |
| enum expansion | manual | 是否需要新增治理面。 | architecture review。 | 需要用户/架构确认。 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 冻结 normalized field schema 和 impact surface enum。 | CP5 前 LLD。 | schema / enum list。 | ISN-UNIT-01..03。 |
| 2 | 接入 FEAT-PG-004 resolver contract。 | registry resolver API 稳定。 | refs validation adapter。 | ISN-CON-01。 |
| 3 | 实现 new CR audit/enforce checker。 | schema + resolver。 | checker result。 | ISN-INT-01..02。 |
| 4 | 实现 legacy migration scanner/report。 | CR index input。 | migration report。 | ISN-INT-03、ISN-SEC-04。 |
| 5 | 补充人工回修和 enum 扩展流程。 | report output stable。 | manual review checklist。 | ISN-MAN-01..03。 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR-E-S01 impact surface field split and migration report | `process/docs/features/impact-surface-normalization/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | migration / data-model / compatibility / security | enum 列表、legacy classification、report format、audit/enforce policy、resolver contract。 | 本 Feature 设计 + Story LLD。 |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| change-impact-analysis | CR 创建/更新时 | normalized impact fields。 | CR 影响面可审计。 | legacy 字段只读并进入 migration report。 |
| checkpoint-manager / CP checks | CP 自动预检时 | checker findings。 | audit/enforce 结果可汇总。 | historical drift 不刷屏。 |
| FEAT-PG-006 / FEAT-PG-008 | roadmap/stale 消费影响面时 | canonical surfaces and refs。 | stale/follow-up 可按治理面聚合。 | unresolved refs blocked。 |
| meta-qa | CP7 / CP8 | TEST-PLAN、migration report。 | 验证 no source rewrite、安全失败路径。 | 手工审查 unclassified legacy。 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| ISN-R-01 | 历史漂移太多，migration report blocked 项过多。 | 延迟 enforce。 | audit 阶段保留，分批人工分类。 | 延长 audit，只对新 CR enforce。 |
| ISN-R-02 | impact surface enum 过窄。 | 合法影响面无法表达。 | enum 扩展走 CR，manual review。 | 暂时映射到 nearest surface 并记录 rationale。 |
| ISN-R-03 | registry resolver 不稳定。 | `feature_refs` / `capability_refs` 无法落地。 | CR-E 依赖 CR-D，contract tests。 | 先只启用 surfaces/path audit，不 enforce refs。 |
| ISN-R-04 | `affected_paths` 被误作授权。 | 越权写风险。 | 明确权限规则和 negative tests。 | 移除授权语义，要求 runtime_authorization 决策。 |
