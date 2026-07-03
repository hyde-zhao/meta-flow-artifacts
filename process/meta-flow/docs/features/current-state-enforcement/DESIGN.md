---
status: draft
version: "1.0"
feature_id: "FEAT-PG-001"
feature_name: "Current State Enforcement"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR037-S01-current-state-schema-and-budgets"
  - "CR037-S02-controlled-update-api-and-writer-refactor"
  - "CR037-S03-agent-contract-and-guardrail-sync"
lld_policy_summary: "schema / writer / gate tests require full-lld; agent contract sync may use technical-note if no code path changes"
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Current State Enforcement

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | host-orchestrator | CP3 approved 后生成 current state enforcement 的 per-feature 设计基线 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 让 `STATE.current.json` 从“可被任意字段污染的默认入口”变成 allowlist、field budget 和受控 writer 共同约束的轻量机器状态。 |
| 推荐方案 | 以 `default_current_state()` 字段集为 allowlist 基线，补显式 optional keys；`write_current_state()` 与新增 `update_current_state()` 共用校验；`state check` 同步执行 unknown field、disallowed field 和 size budget 检查。 |
| 关键取舍 | 优先阻止新污染，历史项目先 audit 后 enforce；不把重型状态迁入 current state。 |
| 下游 Story | `CR037-S01`、`CR037-S02`、`CR037-S03` |
| LLD 策略 | schema、writer/update API、内部直写收敛和 gate tests 使用 full-lld；纯文档契约同步可 technical-note。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-01 | `write_current_state()` 直接写盘，`check_current_state()` 不在写入路径 | agent 或内部代码写入未知字段 | 默认读取入口膨胀，token 成本上升 | 新字段名绕过黑名单，污染复发 |
| P-02 | `cr_lifecycle._update_current_active_change()` 存在内部直写路径 | CR 切换或 bootstrap 更新 active change | 绕过 writer 校验 | allowlist 只在部分路径生效 |
| P-03 | agent 契约缺少合法写入口说明 | agent 直接编辑 `STATE.current.json` | 约束无法不可绕过 | CP/CI 之外仍能污染状态 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | ADR-PG-001、P0 enforcement-first、state/project state 分层 |
| ADR | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md` | `STATE.current.json` 使用 allowlist schema + field budgets，audit -> enforce |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-001 required，full-lld |
| Product Baseline | `process/docs/product/REQUIREMENTS.md` / `SCENARIOS.yaml` | UC-PG-001、REQ-PG-001..005、TC-PG-001..004、TC-PG-018 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | unknown top-level keys 在 audit 阶段 WARN，在 enforce 阶段 ERROR | ADR-PG-001 |
| Goal | field budgets 覆盖 `source_refs`、`open_risks`、`authz_policy_refs`、`next_action` 等合法但可膨胀字段 | CP2 DQ-04 |
| Goal | 内部 active change 更新和外部 writer 统一走受控 API | HLD §18 |
| Non-Goal | 不在本 Feature 设计 `PROJECT.current.json` 长期对象 schema | FEAT-PG-003 |
| Non-Goal | 不执行 quant-lab 数据清理，只为 P2 migration 提供检查能力 | FEAT-PG-009 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| `STATE.current.json` | allowlist、budget、write/update 校验、state check | project roadmap / milestone 内容 | FEAT-PG-003 | current state 只保存轻量运行态 |
| `process/STATE.md` | 渲染后保持人类摘要一致 | 作为机器默认入口 | state-router | AGENT-SKILL-CONTRACT slimming |
| agent 写契约 | 禁止直编和重型字段落点 | 具体每个 skill 的业务输出 | delivery rules / state-router | CP2 DQ-03 |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| State writer/checker | `meta_flow/state/current.py` | default state、write、render、check | modify |
| CR lifecycle | `meta_flow/workflow/cr_lifecycle.py` | active change 更新和 CR summary | modify |
| CLI | `meta_flow/cli.py` | `meta-flow state ...` | modify |
| Tests | `tests/test_state_v2.py` 等 | state v2 回归 | modify / create |
| Agent contract | `delivery/rules/AGENT-SKILL-CONTRACT.md`、state-router skill | agent 读写规则 | modify |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| 字段治理 | `ALLOWED_CURRENT_KEYS = default_current_state.keys() + EXPLICIT_OPTIONAL_CURRENT_KEYS` | 黑名单挡不住自造字段 | 需要迁移或 audit 存量 unknown 字段 |
| 写入口 | `write_current_state()` 和 `update_current_state()` 都先 validate | 统一外部和内部写路径 | 改动现有调用点 |
| patch 语义 | `update_current_state(project_root, patch)` 使用 deep-merge，patch key 也必须 allowlist | 避免读全量改全量、减少并发覆盖 | 删除语义需在 LLD 明确 |
| field budget | 列表字段执行 max_items / max_item_bytes / max_total_bytes；标量字段执行 max_bytes | 防合法字段承载重型正文 | 需要错误信息和测试 fixture |
| gate 接入 | CP/CI 前运行 `meta-flow state check` | 阻断 agent 直编绕过 writer | 需要文档和 gate profile 更新 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| CurrentState schema | `meta_flow/state/current.py` | allowlist、explicit optional keys、field budgets | audit -> enforce | audit 阶段对历史 unknown WARN |
| `project_state_ref` | FEAT-PG-003 生产，FEAT-PG-001 放行 | optional allowlist key | current state 指向 project state | P0 提前纳入 allowlist |
| `pending_checklist_path` | state migration 已有 | 保留合法字段 | human gate 可恢复 | 必须在 allowlist 中 |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| IF-CS-01 | state CLI / internal modules | `write_current_state()` | 完整 state dict | 写入前 validate，失败不落盘 | `StateValidationError` |
| IF-CS-02 | CR lifecycle / state-router | `update_current_state()` | allowlist patch，deep-merge | 返回更新后 state，落盘并可 render | unknown key / budget exceeded |
| IF-CS-03 | CP/CI | `meta-flow state check` | project root | OK / FAIL / WARN list | enforce 下 FAIL 阻断 |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| FLOW-CS-01 | 写入 current state | 构造 state -> validate keys -> validate budgets -> atomic write | unknown key / budget fail 时拒绝写入 | `STATE.current.json` 保持轻量 | state check output |
| FLOW-CS-02 | 内部更新 active change | `update_current_state()` deep-merge `active_change/current_phase/next_action` | patch key unknown 时失败并记录检查结果 | active CR 与 state 一致 | CR tracking check |
| FLOW-CS-03 | agent 直编后 gate 拦截 | CP 前运行 state check | 发现 unknown / disallowed / oversized 时 FAIL | 阻断推进，要求清理 | CP result / state check |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| DQ-CS-01 | audit unknown 字段数量高 | host-orchestrator / user | 扩展 explicit optional keys 或迁移字段 | 暂缓 enforce | 不允许直接放宽为自由字段 |
| DQ-CS-02 | 存量项目依赖 unknown 字段 | user / meta-se | 迁移到 STATE.md、ledger 或 PROJECT.current refs | 保持 audit warning | 不把重型字段放回 current state |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| F-CS-01 | unknown top-level key | audit WARN / enforce ERROR | state check 不通过 | 迁移字段或加白名单理由 | TEST-CS-UNKNOWN |
| F-CS-02 | list budget exceeded | 拒绝写入或检查失败 | 需压缩为 refs | 截断不得静默，改为 ledger/ref | TEST-CS-BUDGET |
| F-CS-03 | 内部调用绕过 writer | 测试失败 | CR bootstrap 不可推进 | 改调用 `update_current_state()` | TEST-CS-WRITER |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| SEC-CS-01 | agent 禁止直接写未知 current state 字段 | agent contract / CP gate | state check FAIL | contract guardrail test |
| SEC-CS-02 | current state 不保存凭据、账户、token、cookie 或私钥 | writer/checker | FAIL | secret fixture |
| SEC-CS-03 | 重型状态必须进入 ledger / result / project state refs | oversized field | FAIL / WARN | budget test |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| allowlist unknown key | unit | TC-PG-001、TC-PG-018 | pytest + `meta-flow state check` | N/A |
| internal writer convergence | integration | TC-PG-002 | CR lifecycle fixture | N/A |
| field budgets | unit | TC-PG-003 | budget fixture | N/A |
| agent contract wording | static / manual | TC-PG-004 | rg guardrail + CP review | 语义需人工审查 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 定义 allowlist、optional keys、budget config | CP3 approved | state schema constants | unit tests |
| 2 | 实现 validate + writer/update API | Step 1 | current.py API | pytest |
| 3 | 收敛内部 direct write 调用 | Step 2 | cr_lifecycle 调用更新 | integration test |
| 4 | 更新 state CLI / gate / contract | Step 2 | CLI 和文档契约 | state check + guardrail |
| 5 | 增加 audit/enforce fixtures | Step 1-4 | tests | pytest |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR037-S01-current-state-schema-and-budgets | `process/docs/features/current-state-enforcement/DESIGN.md` | full-lld | data / security / migration | allowlist set、budget table、audit/enforce切换 | Story LLD |
| CR037-S02-controlled-update-api-and-writer-refactor | 同上 | full-lld | cross-module / rollback | deep-merge、删除语义、内部调用点 | Story LLD |
| CR037-S03-agent-contract-and-guardrail-sync | 同上 | technical-note | docs-contract / guardrail | 哪些 contract 文件同步 | Story 技术说明 |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| story-manager | CP4 前 | Story 拆分建议和 lld_policy | Story 卡片引用本 DESIGN / TEST-PLAN / TASKS | 缺失则 CP4 FAIL |
| lld-designer | CP5 前 | API、budget、失败路径 | full-lld / technical-note | unknown 语义不清则 clarification queue |
| meta-qa | CP7 / CP8 | TEST-PLAN、state fixture | TEST-REPORT / REVIEW 追溯 | 缺测试 fixture 则 CP7 BLOCKED |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| R-CS-01 | allowlist 误伤存量合法字段 | 存量项目推进受阻 | audit 阶段 WARN，收集字段后 enforce | 回到 audit，不回退自由字段 |
| R-CS-02 | `PROJECT.current.json` ref 未提前放行 | P1 无法写 project state ref | P0 optional key 预置 | 增加 explicit optional key 并补测试 |
| R-CS-03 | agent 仍直编文件 | enforcement 被绕过 | CP gate + contract guardrail | state check 阻断推进 |
