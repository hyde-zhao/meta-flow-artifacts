---
status: cp3-approved
version: "1.0"
cr_ref: "CR-037"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
confirmed_by: "user"
confirmed_at: "2026-07-02T19:43:16+08:00"
---

# Meta Flow Project Governance Feature Design Matrix

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 基于 HLD 判定 Feature 级设计适用性、产物路径和 lld_policy |
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 CP3 pending 状态；将 CR-A..CR-H 命名映射决策纳入 CP3 DQ |
| 1.2 | 2026-07-03 | host-orchestrator | 同步 CP3 approved / CP5 pending 状态；确认 per-feature 设计已生成，下一步为 Story LLD 批次 |

## 适用性判定规则

| 维度 | 需要 Feature 设计的触发条件 | 可豁免条件 |
|---|---|---|
| 数据与状态 | 新增 / 修改核心状态对象、schema、migration、兼容策略 | 只读文档或无状态配置 |
| 接口与依赖 | 跨模块 API、CLI、checker、resolver 或 ledger/event 契约 | 单文件低风险文案改动 |
| 权限与安全 | 默认状态入口、发布库写边界、runtime authorization、审计 | 无权限变化且不触及 process / release 边界 |
| 运行与可靠性 | audit/enforce、dry-run、回滚、blocked finding、stale report | 无运行时风险扩展 |
| 多 Story 复用 | 多个 Story 共享同一 schema / checker / registry / migration 契约 | 单 Story 可由技术说明覆盖 |

## Feature 设计矩阵

| Feature ID | Feature / Epic | 来源 | 适用性 | 判定理由 | 需要产物 | 关联 Story / CR 切片建议 | 建议 lld_policy | 重访条件 |
|---|---|---|---|---|---|---|---|---|
| FEAT-PG-001 | Current State Enforcement | BP §3 / HLD §17 / ADR-PG-001 | required | state schema、writer API、agent 写契约和门禁风险高 | `process/docs/features/current-state-enforcement/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-A；Story: schema、writer/update API、contract sync、gate tests | full-lld | allowlist、预算或写入口变化 |
| FEAT-PG-002 | Ledger Compaction | BP §3 / HLD §15 | required | retention / archive / index 影响审计和恢复 | `process/docs/features/ledger-compaction/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-B；Story: compact policy、CLI、archive index、tests | full-lld | ledger 类型或 retention policy 扩展 |
| FEAT-PG-003 | Project State Governance | BP §3 / HLD §17 / ADR-PG-002 | required | 新增 project objects、workspace scaffold、state ref 边界 | `process/docs/features/project-state-governance/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-C；Story: scaffold、PROJECT.current、PROJECT-SCALE、ROADMAP/MILESTONES | full-lld | project scale 或 gate bias 规则变化 |
| FEAT-PG-004 | Capability / Feature Registry | BP §3 / HLD §17 / ADR-PG-004 | required | registry 是 migration 和 impact refs 的共享契约 | `process/docs/features/capability-feature-registry/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-D；Story: registry schema、resolver、capability check、quant-lab source mapping | full-lld | registry schema、source priority 或 ID policy 变化 |
| FEAT-PG-005 | Impact Surface Normalization | BP §3 / HLD §17 | required | CR schema migration、audit/enforce、blocked finding | `process/docs/features/impact-surface-normalization/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-E；Story: field split、migration report、new CR audit/enforce | full-lld | impact enum 或 CR index schema 变化 |
| FEAT-PG-006 | Roadmap Refresh Governance | BP §3 / HLD §17 / ADR-PG-003 / ADR-PG-005 | required | 独立 result、checker、cascade、Gate Ledger 和安全边界 | `process/docs/features/roadmap-refresh-governance/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-F；Story: result schema、checker、process-only cascade、ledger event | full-lld | refresh decision、自动写入边界或 ledger event 字段变化 |
| FEAT-PG-007 | FU-RF Candidate Support | BP §3 / HLD §17 / ADR-PG-006 | required | CR tracking regex、模板、状态查询影响 follow-up 生命周期 | `process/docs/features/roadmap-follow-up-tracking/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-G；Story: regex、templates、state-router/change-impact docs、tests | technical-note 或 full-lld | 若影响多个 tracking 模块或 release context guardrail，升为 full-lld |
| FEAT-PG-008 | Project Stale Check | BP §3 / HLD §17 | required | 跨对象语义检查，需避免与 cr-tracking 重叠 | `process/docs/features/project-stale-check/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-G 或独立 CR；Story: stale rules、CLI、report、FU-RF integration | full-lld | stale rule set 扩展到 release readiness 或 runtime 阶段 |
| FEAT-PG-009 | Quant-lab Migration Readiness | BP §3 / HLD §17 | required | 真实样本迁移、发布库只读边界、dry-run 和回滚 | `process/docs/features/quant-lab-migration-readiness/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | CR-H；Story: current state cleanup、capability registry migration、impact migration、stale report | full-lld | quant-lab 发布库写授权或 migration scope 变化 |

## Story 下游消费表

| Story / Slice ID | feature_design_refs | lld_policy.required_level | trigger_reasons | 设计证据 | CP5 审查方式 |
|---|---|---|---|---|---|
| CR037-S01 current-state schema and budgets（alias: CR-A-S01） | current-state-enforcement DESIGN / TEST-PLAN / TASKS | full-lld | data-model / security / shared-story-boundary | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S02 controlled update API and writer refactor（alias: CR-A-S02） | current-state-enforcement DESIGN / TEST-PLAN / TASKS | full-lld | cross-module-contract / rollback | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S03 agent contract and guardrail sync（alias: CR-A-S03） | current-state-enforcement DESIGN / TEST-PLAN / TASKS | technical-note | docs-contract / guardrail | Story 技术说明 | CP5 自动预检 + 批量人工确认 |
| CR037-S04 ledger compaction policy and CLI（alias: CR-B-S01） | ledger-compaction DESIGN / TEST-PLAN / TASKS | full-lld | data-model / migration / rollback | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S05 project scaffold and PROJECT.current（alias: CR-C-S01） | project-state-governance DESIGN / TEST-PLAN / TASKS | full-lld | data-model / workspace-scaffold | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S06 PROJECT-SCALE and roadmap objects（alias: CR-C-S02） | project-state-governance DESIGN / TEST-PLAN / TASKS | full-lld | data-model / cross-module-contract | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S07 feature/capability registry and resolver（alias: CR-D-S01） | capability-feature-registry DESIGN / TEST-PLAN / TASKS | full-lld | data-model / external-interface | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S08 impact surface field split and migration report（alias: CR-E-S01） | impact-surface-normalization DESIGN / TEST-PLAN / TASKS | full-lld | migration / data-model / compatibility | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S09 roadmap refresh result schema and checker（alias: CR-F-S01） | roadmap-refresh-governance DESIGN / TEST-PLAN / TASKS | full-lld | data-model / checker / rollback | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S10 process-only cascade and Gate Ledger event（alias: CR-F-S02） | roadmap-refresh-governance DESIGN / TEST-PLAN / TASKS | full-lld | security / runtime_authorization / event-contract | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S11 FU-RF tracking support（alias: CR-G-S01） | roadmap-follow-up-tracking DESIGN / TEST-PLAN / TASKS | technical-note | cross-module-contract / low-runtime-risk | Story 技术说明；若 touch 面扩大则 full-lld | CP5 自动预检 + 批量人工确认 |
| CR037-S12 project stale-check（alias: CR-G-S02） | project-stale-check DESIGN / TEST-PLAN / TASKS | full-lld | cross-object-semantics / follow_up_tracking | Story LLD | CP5 自动预检 + 批量人工确认 |
| CR037-S13 quant-lab migration dry-run and reports（alias: CR-H-S01） | quant-lab-migration-readiness DESIGN / TEST-PLAN / TASKS | full-lld | migration / security / external-project-boundary | Story LLD | CP5 自动预检 + 批量人工确认 |

## 提前确认的关键决策

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 | 状态 |
|---|---|---|---|---|---|---|---|---|
| DQ-FD-001 | architecture | 后续 CR 命名是否沿用 CR-A..CR-H 还是映射成正式 CR-037+ | 映射为正式 CR 编号，并保留 CR-A..CR-H 作为 slice alias | 直接使用 CR-A..CR-H | 正式编号便于 CR index；alias 保持计划可读 | 影响 Story / evidence 命名 | 已由 `CP3-CR037-DQ-07` 确认：正式 Story / evidence 使用 `CR037-Sxx`，CR-A..CR-H 仅作 alias | resolved-cp3 |
| DQ-FD-002 | implementation | FEAT-PG-007 是否需要 full-lld | 初始 technical-note，若触及多个 tracking 模块或 release guardrail 升级 full-lld | 直接 full-lld | 推荐方案减少低风险设计成本；备选更稳但成本高 | 影响 CP5 设计证据量 | 文件影响超过 3 个模块或新增 schema 时升级 | open |
| DQ-FD-003 | implementation | ledger compact 命令名 | `meta-flow ledger compact` | `meta-flow event compact` | 推荐方案语义更贴近对象；备选覆盖所有 event ledger | 影响 CLI UX 和文档 | FEAT-PG-002 LLD 前确认 | open |
| DQ-FD-004 | implementation | stale-check 命令名 | `meta-flow project stale-check` | `meta-flow check project-stale` | 推荐方案体现 project 子域；备选和 checker 命名统一 | 影响 CLI UX | FEAT-PG-008 LLD 前确认 | open |

## 豁免与 N/A 说明

| Feature ID | 豁免 / N/A 原因 | 影响范围 | 风险接受 | 重访条件 | 责任方 |
|---|---|---|---|---|---|
| N/A | 当前 9 个 Feature 均命中 required；无 waived / N/A | 无 | not-needed | 若后续 CR 缩小为单文件文档同步，可对该 Story 使用 technical-note | meta-se / host-orchestrator |

## Feature 设计证据策略

当前已存在的 `process/docs/features/meta-flow-project-governance-state-enforcement/` 是 CR-037 的 umbrella 设计包，用于记录整体实施策略、测试计划和任务基线。它不能替代下表 9 个 required Feature 的 per-feature `DESIGN.md` / `TEST-PLAN.md` / `TASKS.md`。

CR-037 已通过 CP3，HLD / ADR frontmatter 已标记 `confirmed: true`；per-feature 设计正文已按矩阵生成。当前处于 `story-planning / CP5 pending`，进入实现前必须满足：

- FEAT-PG-001、002、003、004、005、006、008、009 已生成 full-lld 级 Feature 设计证据。
- FEAT-PG-007 至少生成 technical-note 级证据；若触及多个 tracking 模块或 release guardrail，则升级 full-lld。
- 每个 Story 的 `feature_design_refs` 指向对应 per-feature 路径，而不是 umbrella 目录。

## 自检

| 检查项 | 结果 | 证据 |
|---|---|---|
| 所有 Feature / Epic 均已判定 | PASS | FEAT-PG-001..009 |
| required Feature 均有产物计划 | PASS | 本矩阵列出 FEAT-PG-001..009 的 DESIGN / TEST-PLAN / TASKS 目标路径 |
| required Feature 设计正文已生成 | PASS | `process/docs/features/current-state-enforcement/`、`ledger-compaction/`、`project-state-governance/`、`capability-feature-registry/`、`impact-surface-normalization/`、`roadmap-refresh-governance/`、`roadmap-follow-up-tracking/`、`project-stale-check/`、`quant-lab-migration-readiness/` |
| 每个建议 Story 均有 feature_design_refs 与 lld_policy | PASS | Story 下游消费表 |
| 提前确认的关键决策已列出 | PASS | DQ-FD-001..004 |
