---
status: baseline
version: "1.0"
cr_ref: "CR-037"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_plan: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-STATE-ENFORCEMENT-IMPLEMENTATION-PLAN-2026-07-02.md"
---

# Meta Flow Project Governance Domain Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 定义项目治理整改的核心术语、领域对象、状态机和规则 |

## 1. 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| Current State | 机器默认读取的轻量运行态入口，路径为 `process/state/STATE.current.json` | P0 | 只能保存 allowlist 字段、短 refs 和小预算字段 |
| Project State | 项目级轻量状态对象，路径为 `process/state/PROJECT.current.json` | P1 | 由 `STATE.current.json.project_state_ref` 引用 |
| Project Scale | 项目规模分档：`lite` / `standard` / `full` | P1 | 只记录 gate profile bias，不直接修改 `GATE-PROFILES.json` |
| Roadmap Refresh | 根据项目定位、CR 或 milestone 变化刷新过程侧 roadmap / project state 的机制 | P1.4 / P1.5 | 不复用 CP result checker |
| Process Artifact Repository | 保存 process、CR、checkpoint、ledger、context、运行态证据的过程归档空间 | §3 仓库边界 | roadmap refresh 可自动写入 |
| Release Repository | quant-lab 等目标项目的代码、正式文档、测试和发布对象所在空间 | §3 仓库边界 | roadmap refresh 不自动修改 |
| Impact Surface | 有限治理面枚举，用于冲突检测和影响分类 | P1.3 | 不再承载路径、模块或文件名 |
| Affected Paths | 文件、目录、模块路径影响集合 | P1.3 | 从 `impact_surface` 中拆出 |
| Feature Refs | 指向 `FEATURE-REGISTRY.yaml` 中已注册 Feature ID 的引用 | P1.2a | 不允许自由字符串 |
| Capability Refs | 指向 `CAPABILITY-STATUS.yaml` 中已注册 capability id 的引用 | P1.2a | 不允许 migration 凭空创造 |
| FU-RF | Roadmap refresh follow-up 候选项编号前缀 | P1.6 | 不写入 `RELEASE-CONTEXT` |
| Stale Item | 发布库或设计对象与当前 project state / roadmap 语义不一致的条目 | P1.7 | 只报告，不自动修复正式文档 |

## 2. 领域对象

| Object ID | 对象 | Owner Feature | 关键字段 / 属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-PG-001 | `STATE.current.json` | FEAT-PG-001 | `schema_version`、`project_id`、`current_phase`、`active_change`、`pending_gate`、`next_action`、`project_state_ref`、refs | audit / enforce compliant / non-compliant | P0 allowlist、字段预算 |
| OBJ-PG-002 | Current State Schema | FEAT-PG-001 | required keys、optional keys、field budgets、unknown field policy | audit / enforce | P0 §5.2-5.4 |
| OBJ-PG-003 | State Patch | FEAT-PG-001 | patch、actor、reason、deep-merge semantics | accepted / rejected | P0 §5.5 |
| OBJ-PG-004 | Ledger Compaction Policy | FEAT-PG-002 | `keep_latest_n_events`、`keep_latest_n_cr`、`window_days`、`archive_rule`、`index_fields` | draft / active | P0 后段 |
| OBJ-PG-005 | `PROJECT.current.json` | FEAT-PG-003 | `project_positioning`、`project_scale`、`current_project_phase`、objective/milestone refs、roadmap/milestone refs、health refs | active / stale / invalid | P1 §7.2 |
| OBJ-PG-006 | `PROJECT-SCALE.yaml` | FEAT-PG-003 | `project_scale`、`gate_profiles_ref`、`default_gate_profile_bias`、reason | active / invalid | P1 §7.3 |
| OBJ-PG-007 | `ROADMAP.yaml` | FEAT-PG-003 | roadmap items、milestones、objective refs | current / stale | P1 §7.4 |
| OBJ-PG-008 | `MILESTONES.yaml` | FEAT-PG-003 | milestone id、status、objective refs | current / stale | P1 §7.4 |
| OBJ-PG-009 | Feature Registry | FEAT-PG-004 | registered `feature_id`、name、status、owner | valid / missing / invalid | P1.2a |
| OBJ-PG-010 | Capability Registry | FEAT-PG-004 | registered capability id、status、source refs | valid / missing / invalid | P1.2a |
| OBJ-PG-011 | CR Impact Record | FEAT-PG-005 | `impact_surface`、`affected_paths`、`feature_refs`、`capability_refs` | legacy / audit-warn / enforce-error / normalized | P1.3 |
| OBJ-PG-012 | Roadmap Refresh Result | FEAT-PG-006 | `decision`、source、machine_updates、must_check、stale_items、follow_up_candidates、event_refs | NO_CHANGE / UPDATED / UPDATED_WITH_DOC_IMPACTS / BLOCKED / FAILED | P1.4 |
| OBJ-PG-013 | Roadmap Refresh Event | FEAT-PG-006 | `event_id`、`event_type`、gate、status、result_ref | appended / rejected | P1.5 |
| OBJ-PG-014 | FU-RF Candidate | FEAT-PG-007 | `FU-RFxxx-yyy`、source refresh、impact、recommended CR route | open / accepted / closed | P1.6 |
| OBJ-PG-015 | Stale Finding | FEAT-PG-008 | stale object, expected semantic, observed semantic, follow-up candidate | open / converted / waived | P1.7 |
| OBJ-PG-016 | Quant-lab Migration Report | FEAT-PG-009 | current state cleanup, capability normalization, impact migration, stale report, FU-RF refs | draft / passed / blocked | P2 |

## 3. 状态机

| State Machine ID | 对象 | 状态 | 合法转换 | 非法转换处理 |
|---|---|---|---|---|
| SM-PG-001 | Current State Enforcement | audit -> enforce | audit 阶段 unknown fields=WARN；enforce 阶段 unknown fields=ERROR | enforce 中出现 unknown / over-budget / forbidden keys 时拒绝写入并阻断 gate |
| SM-PG-002 | State Patch Lifecycle | proposed -> validated -> written -> checked | patch key 属于 allowlist，deep-merge 后完整 state 校验通过 | unknown key、预算超限、required key 删除均 rejected |
| SM-PG-003 | CR Impact Migration | legacy -> report-only -> new-cr-audit -> new-cr-enforce | 历史 CR 静默，migration report 输出；新 CR 先 WARN 后 ERROR | 无法映射的 capability / feature refs 进入 blocked finding |
| SM-PG-004 | Roadmap Refresh | triggered -> evaluated -> updated / updated_with_doc_impacts / no_change / blocked / failed | 过程归档库更新成功后追加 Gate Ledger event | 发布库需要变更时不得自动写入，只生成 stale / follow-up |
| SM-PG-005 | Quant-lab Migration | planned -> dry-run -> process-side-updated -> stale-reported -> complete / blocked | 新能力通过后对 quant-lab process artifacts 迁移 | 发现发布库修改需求时转 FU-RF / 正式 CR，不在迁移内直接修改 |

## 4. 业务规则

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-PG-001 | `STATE.current.json` 顶层 key 必须全部属于 allowlist | FEAT-PG-001 | P0 state writer、agent contract | state check、writer tests |
| RULE-PG-002 | 合法字段也必须受字段预算约束，重型状态不得迁入 current state | FEAT-PG-001 | P0 default entry slimming | state check、field budget tests |
| RULE-PG-003 | agent / skill 不得直接编辑 `STATE.current.json` | FEAT-PG-001 | 所有 agent contract | guardrail、contract docs check |
| RULE-PG-004 | `PROJECT.current.json` 只保存 refs 和短字段，总预算建议不超过 16KB | FEAT-PG-003 | P1 project governance | project state checker |
| RULE-PG-005 | `project_scale` 只产生 gate profile bias，不直接修改 `GATE-PROFILES.json` | FEAT-PG-003 | P1 project scale | project scale check |
| RULE-PG-006 | `feature_refs` 和 `capability_refs` 必须引用标准 registry | FEAT-PG-004 | P1.2a / P1.3 / P2 | `meta-flow capability check`、`feature check` |
| RULE-PG-007 | `impact_surface` 只能保存有限治理面枚举，路径必须放入 `affected_paths` | FEAT-PG-005 | P1.3 CR impact | impact migration check |
| RULE-PG-008 | Roadmap refresh 不复用 CP result checker | FEAT-PG-006 | P1.4 | `meta-flow check roadmap-refresh` |
| RULE-PG-009 | Roadmap refresh 自动写入范围只覆盖过程归档库 | FEAT-PG-006 | P1.5 / P2 | refresh result checker、Gate Ledger check |
| RULE-PG-010 | FU-RF candidate 不写入 `RELEASE-CONTEXT` | FEAT-PG-007 | P1.6 | cr_tracking tests、release context guardrail |
| RULE-PG-011 | stale-check 只检测跨对象语义陈旧，不重复 cr-tracking 结构一致性 | FEAT-PG-008 | P1.7 | stale-check tests |
| RULE-PG-012 | quant-lab migration 不自动修改发布库代码、测试或正式文档 | FEAT-PG-009 | P2 | migration dry-run / stale report |
