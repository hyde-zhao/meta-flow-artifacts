---
status: draft
version: "1.0"
feature_id: "FEAT-PG-003"
feature_name: "Project State Governance"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR-C-S01 project scaffold and PROJECT.current"
  - "CR-C-S02 PROJECT-SCALE and roadmap objects"
lld_policy_summary: "full-lld for all CR-C stories because project state introduces new persistent objects, schema budgets, workspace scaffold, and cross-module governance contracts."
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Project State Governance

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 基于 CR-037 CP3 已批准推荐方案生成 FEAT-PG-003 长期 baseline |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 建立独立项目级治理状态，使 `STATE.current.json` 只保存 `project_state_ref`，避免默认机器入口承载 roadmap、milestone 和规模策略等重型状态。 |
| 推荐方案 | 新增 `process/project/` 对象族：`PROJECT.current.json`、`PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml`，由 project-state checker 校验，并通过 current-state 受控写入口引用。 |
| 关键取舍 | 增加一组 schema/checker 和 scaffold 成本，换取 current state 瘦身、项目治理对象可审计、后续 roadmap refresh 可依赖稳定对象。 |
| 下游 Story | CR-C-S01、CR-C-S02 |
| LLD 策略 | 两个 Story 均为 `full-lld`，因为涉及数据模型、workspace scaffold、跨模块契约和 gate profile bias。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-PSG-01 | HLD 将 current state 定位为轻量默认机器入口。 | 需要保存项目 roadmap、milestone、scale、active CR 摘要时。 | 默认入口可能再次膨胀，破坏 `STATE.current.json` 字段预算。 | agent 继续把项目长期状态塞入 current state，P0 enforcement 失效。 |
| P-PSG-02 | ADR-PG-002 确认 project state 独立于 current state。 | Host Orchestrator 或 checker 需要项目级状态引用。 | 需要明确对象归属、schema、预算和引用方式。 | `PROJECT.current.json` 成为新的巨型状态，缺少预算和失败路径。 |
| P-PSG-03 | HLD 要求 project scale 只提供 gate profile 默认偏好。 | 项目规模影响 CP 门禁默认建议。 | 容易被误用为单次 CR 授权或绕过人工确认。 | gate profile policy 被隐式覆盖，产生授权边界风险。 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | ADR-PG-002、UC-PG-002、UC-PG-003、§17 FEAT-PG-003 required、§18 project object schema 下沉要求。 |
| ADR | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md` | `PROJECT.current.json` 独立存在，`STATE.current.json` 只保存 `project_state_ref`。 |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-003 required；CR-C-S01/CR-C-S02 均建议 `full-lld`。 |
| CP3 决策 | CR-037 已批准的推荐方案 | 采用 enforcement-first 分层治理，project governance 位于 current-state enforcement 之后。 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 创建可校验的 `process/project/` scaffold，支持项目级状态、规模、roadmap 和 milestone。 | HLD §4 / §15 |
| Goal | `PROJECT.current.json` 总量预算不超过 16KB，且 current state 只保存 `project_state_ref`。 | HLD §1 成功标准 |
| Goal | project scale 只影响 gate profile 默认偏好，不直接修改 `process/policies/GATE-PROFILES.json`。 | HLD §6 UC-PG-003 |
| Goal | 对无效 project object、超预算、断引用输出明确 FAIL/WARN。 | HLD §12 可验证性 |
| Non-Goal | 不实现 roadmap refresh cascade、stale-check 或 quant-lab migration。 | HLD §15 W4/W5 |
| Non-Goal | 不新增第二套 context/result/ledger 体系。 | HLD §1 非目标 |
| Non-Goal | 不授权自动修改发布库。 | ADR-PG-003 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| `PROJECT.current.json` | schema、预算、字段语义、与 current state 的 refs-only 关系。 | current state allowlist 细节。 | FEAT-PG-001 Current State Enforcement | ADR-PG-002 |
| `PROJECT-SCALE.yaml` | 项目规模、复杂度和 gate profile bias 的只读建议。 | 直接改写 gate policy 或替用户批准风险。 | checkpoint-manager / gate profile policy | HLD UC-PG-003 |
| `ROADMAP.yaml` / `MILESTONES.yaml` | project-level baseline 对象和引用约束。 | refresh decision、stale 规则、Gate Ledger event。 | FEAT-PG-006 / FEAT-PG-008 | HLD §18 |
| workspace scaffold | 目录和初始文件创建、幂等检查、dry-run 预览。 | 跨仓迁移和发布库写入。 | package/workspace setup | HLD §9 |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| project artifacts | `process/project/` | 尚未由本 Feature 建立标准对象族。 | create scaffold and schema-owned files |
| state reference | `process/state/STATE.current.json` | 默认机器状态入口。 | modify through FEAT-PG-001 controlled writer only |
| checker | `meta_flow/checks/` 或等价 checker 模块 | 承载自动检查入口。 | create project-state checker |
| CLI | `meta_flow/cli` 或等价命令入口 | 暴露项目治理操作。 | add scaffold/check subcommands in Story LLD |
| tests | `tests/` | 覆盖 schema、CLI、checker。 | create fixtures and regression tests |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| 数据 | HLD 已定义 project state 独立原则，但未下沉字段。 | 缺少 `PROJECT.current.json`、scale、roadmap、milestone schema。 | current state 只能保存 ref，project current 预算 16KB。 |
| 接口 | 缺少统一 scaffold/check/update 入口。 | 需要命令或库 API 创建和校验 project objects。 | 写 current state ref 必须走受控 writer。 |
| 测试 | 当前只有 HLD 级成功标准。 | 需要 unit / integration / contract / manual 验收。 | 不触发 CP5，不实现测试代码。 |
| 运维 / 发布 | project objects 是 process artifact，不是发布库内容。 | 需要明确断链、超预算和不兼容字段失败路径。 | 不自动修改发布库。 |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| 对象分层 | `STATE.current.json` 只保存 `project_state_ref`，具体项目治理信息进入 `process/project/`。 | 保持默认机器入口瘦身，并让项目级状态独立演进。 | 需要 checker 跟踪 ref 完整性。 |
| project current | `PROJECT.current.json` 保存项目摘要、scale ref、roadmap ref、milestone ref、active governance refs，预算 16KB。 | 提供机器可读项目状态，但避免完整历史。 | 字段必须严控，长文本需外链。 |
| scaffold | 提供 dry-run + apply 两阶段 scaffold。 | 降低误写目录和覆盖用户文件风险。 | CLI 设计略复杂。 |
| scale bias | `PROJECT-SCALE.yaml` 只声明默认 gate profile bias 和理由。 | 避免把项目规模误解为授权。 | 需要文档和 checker 明确“不授权项”。 |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| PSG-DQ-001 | 独立 `PROJECT.current.json` + refs-only current state | current state 瘦身，项目对象可校验，可被 refresh/stale 复用。 | 新增对象和 checker。 | state / project-governance / roadmap | 推荐 | 若 project current 超预算，继续拆外链，不回退到 current state。 |
| PSG-DQ-001 | project fields 直接进入 `STATE.current.json` | 实现快，读取路径少。 | 破坏 P0 allowlist 与 budget，不符合 ADR-PG-002。 | state / agent contract | 备选但不采用 | 仅在项目治理被完全取消时才可考虑。 |
| PSG-DQ-002 | scale 作为 gate bias | 保留人工门禁，风险低。 | 需要调用方理解 bias 不是授权。 | gate / checkpoint | 推荐 | 若未来要自动切换 gate profile，必须新 CR + human gate。 |
| PSG-DQ-002 | scale 直接改 gate profile | 自动化强。 | 隐式授权风险高。 | security / runtime_authorization | 备选但不采用 | 仅在 CP gate policy 明确授权后可切换。 |

## 模块变更

| Module | 变更 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| project scaffold | 新增幂等 scaffold 创建逻辑。 | workspace root、project id、dry-run/apply mode。 | `process/project/` 初始对象。 | 目录已存在且字段冲突时 fail，不覆盖用户改动。 |
| project-state schema | 新增 `PROJECT.current.json` schema 和预算规则。 | candidate project current。 | PASS/WARN/ERROR findings。 | unknown key、over-budget、断 ref 时 fail。 |
| project-scale schema | 新增 scale profile 和 gate bias 规则。 | `PROJECT-SCALE.yaml`。 | scale validation result。 | 直接声明授权或修改 gate profile 时 fail。 |
| roadmap/milestone schema | 定义 roadmap/milestone baseline 最小字段。 | `ROADMAP.yaml`、`MILESTONES.yaml`。 | validated project planning objects。 | milestone 无 roadmap ref、状态非法或 ID 重复时 fail。 |
| current-state integration | 通过 FEAT-PG-001 writer 写入 `project_state_ref`。 | validated project current path。 | updated `STATE.current.json` ref。 | 未经 writer 或 ref 指向无效对象时 fail。 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| `PROJECT.current.json` | project-state-governance | `project_id`、`project_name`、`project_uid`、`scale_ref`、`roadmap_ref`、`milestones_ref`、`active_governance_refs`、`updated_at`、`source_refs`。 | new -> valid -> referenced_by_current_state。 | 长文本、历史、完整 roadmap 不得内嵌；通过 refs 外链。 |
| `PROJECT-SCALE.yaml` | project-state-governance | `scale_level`、`scale_reason`、`gate_profile_bias`、`review_cadence_bias`、`not_authorized`。 | draft -> baseline。 | 不直接写 `GATE-PROFILES.json`。 |
| `ROADMAP.yaml` | project-state-governance | `roadmap_id`、`horizon`、`items[]`、`milestone_refs`、`source_refs`。 | baseline only；refresh 由 FEAT-PG-006 更新。 | 本 Feature 只定义初始结构。 |
| `MILESTONES.yaml` | project-state-governance | `milestone_id`、`title`、`target_window`、`status`、`roadmap_item_refs`。 | baseline only。 | 状态枚举由 checker 校验。 |
| `STATE.current.json` | FEAT-PG-001 | `project_state_ref`。 | ref absent -> ref present。 | 写入必须通过 controlled writer。 |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| PSG-IF-01 | Host Orchestrator / CLI | project scaffold service | project root、project id/name、mode=`dry-run|apply`。 | scaffold plan 或 created file list。 | `E_PROJECT_EXISTS_CONFLICT`、`E_INVALID_PROJECT_ID`、`E_WRITE_NOT_AUTHORIZED`。 |
| PSG-IF-02 | checker / CP4 precheck | project-state validator | project object paths、schema version。 | finding list with severity。 | `E_PROJECT_STATE_MISSING`、`E_PROJECT_STATE_OVER_BUDGET`、`E_PROJECT_REF_BROKEN`。 |
| PSG-IF-03 | project scaffold service | current-state writer | `project_state_ref` patch、actor、reason。 | updated current state。 | 透传 FEAT-PG-001 writer errors。 |
| PSG-IF-04 | roadmap refresh / stale-check | project object reader | refs from `PROJECT.current.json`。 | typed project state snapshot。 | `E_PROJECT_OBJECT_INVALID`、`E_PROJECT_OBJECT_STALE`。 |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| PSG-FLOW-01 | 初始化项目治理对象。 | dry-run 生成 plan -> 用户/调用方确认 apply -> 创建 project objects -> 校验 -> 写 `project_state_ref`。 | 任一对象冲突或校验失败时不写 current state ref。 | `process/project/` baseline + current ref。 | scaffold plan、checker result。 |
| PSG-FLOW-02 | 更新 project scale bias。 | 修改 `PROJECT-SCALE.yaml` -> checker 校验枚举和不授权声明 -> project current 更新时间。 | 发现 gate profile 直接覆盖则 fail。 | scale bias 可被 CP gate 作为默认建议读取。 | project-state check。 |
| PSG-FLOW-03 | roadmap/milestone baseline 被下游读取。 | reader 解析 `PROJECT.current.json` refs -> 加载 roadmap/milestones -> 返回 typed snapshot。 | 断 ref、ID 重复、状态非法时返回 blocked finding。 | 下游 refresh/stale 使用稳定输入。 | checker findings。 |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| PSG-HG-001 | project scale 需要影响默认 gate profile bias。 | CP5 / 后续实现前由用户或 host-orchestrator 确认。 | scale 只作为 bias，不自动批准风险。 | 不暴露 scale bias，只记录 scale。 | 不授权修改 gate policy，不授权跳过人工 CP。 |
| PSG-HG-002 | scaffold 将覆盖已存在 project object。 | 执行前。 | fail 并要求显式 merge。 | 生成 `.candidate` 文件供人工合并。 | 不授权静默覆盖。 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| PSG-F-01 | `PROJECT.current.json` 超过 16KB。 | checker ERROR，拒绝写入 current ref。 | project governance 初始化或更新失败。 | 将长文本迁移到独立文档，只保留 ref。 | TEST-PLAN PSG-SEC-02 / PSG-INT-02 |
| PSG-F-02 | `project_state_ref` 指向缺失文件。 | current/project checker FAIL。 | 后续 refresh/stale blocked。 | 重新 scaffold 或修正 ref。 | TEST-PLAN PSG-CON-01 |
| PSG-F-03 | scaffold 目标文件已存在且内容冲突。 | apply 中止，不覆盖。 | 需要人工合并。 | 使用 dry-run plan 或 candidate 文件重试。 | TEST-PLAN PSG-SEC-01 |
| PSG-F-04 | scale 试图直接授权 gate profile。 | checker FAIL。 | gate bias 不生效。 | 改为建议字段，风险授权走 CP gate。 | TEST-PLAN PSG-SEC-03 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| PSG-SEC-01 | scaffold 不得静默覆盖已有用户文件。 | apply mode 且目标存在。 | fail with conflict finding。 | TEST-PLAN PSG-SEC-01 |
| PSG-SEC-02 | `PROJECT.current.json` 不得内嵌长历史、完整 HLD、完整 transcript 或 credential。 | checker 发现 forbidden fields 或超预算。 | ERROR。 | TEST-PLAN PSG-SEC-02 |
| PSG-SEC-03 | project scale 不得替代人工授权。 | scale file 包含 direct authorization / skip gate 语义。 | ERROR。 | TEST-PLAN PSG-SEC-03 |
| PSG-SEC-04 | current state ref 写入必须走受控 writer。 | 需要更新 `project_state_ref`。 | 未经 writer 的写入在 guardrail/checker 中失败。 | TEST-PLAN PSG-CON-02 |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| project object schema | unit | valid / invalid / over-budget project current。 | schema validator tests。 | 无。 |
| scaffold apply | integration | dry-run 不写入、apply 创建、冲突不覆盖。 | tmp workspace integration tests。 | 无。 |
| current ref contract | contract | `STATE.current.json` only stores `project_state_ref`。 | checker + writer contract tests。 | 无。 |
| scale gate bias | security / contract | scale 只能提供 bias，不得授权。 | negative fixtures。 | CP5 人工确认 bias 文案。 |
| baseline usability | manual | reviewer 能从 project current 找到 roadmap/milestones。 | 手工审查路径和样例。 | 需要人确认语义可读性。 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 冻结 project object schema 和预算。 | CP5 前 LLD。 | schema / fixtures plan。 | TEST-PLAN unit cases。 |
| 2 | 设计 scaffold dry-run/apply 行为。 | schema 稳定。 | scaffold API / CLI design。 | integration cases。 |
| 3 | 接入 current-state writer 的 `project_state_ref`。 | FEAT-PG-001 可用。 | writer contract。 | contract tests。 |
| 4 | 建立 project-state checker。 | schema + scaffold。 | finding model。 | checker tests。 |
| 5 | 补充人工验收和文档契约。 | checker pass。 | Story card refs / CP5 evidence。 | manual checklist。 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR-C-S01 project scaffold and PROJECT.current | `process/docs/features/project-state-governance/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | data-model / workspace-scaffold / security | scaffold overwrite semantics、project current schema、current ref write path。 | 本 Feature 设计 + Story LLD。 |
| CR-C-S02 PROJECT-SCALE and roadmap objects | `process/docs/features/project-state-governance/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | data-model / cross-module-contract | scale bias fields、roadmap/milestone schema、downstream reader contract。 | 本 Feature 设计 + Story LLD。 |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| story-manager | CP4 前 | Story 拆分建议、feature refs、`full-lld` 判定。 | Story 卡片必须引用本设计包。 | 缺失则 CP4 FAIL。 |
| lld-designer | CP5 前 | 本 DESIGN / TEST-PLAN / TASKS。 | CR-C-S01/CR-C-S02 输出 full LLD。 | 若 FEAT-PG-001 未就绪，LLD 标记依赖 blocked。 |
| roadmap refresh | FEAT-PG-006 实现前 | typed project state snapshot。 | 只读取 validated refs。 | project state invalid 时 refresh blocked。 |
| meta-qa | CP7 / CP8 | TEST-PLAN 和 failure cases。 | 验证 project object、security failure、manual acceptance。 | 未自动化项必须列入人工验收。 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| PSG-R-01 | `PROJECT.current.json` 变成新巨型状态。 | default state 瘦身目标失败。 | 16KB budget、refs-only、forbidden fields。 | 拆分长内容到 roadmap/milestone/detail docs，只保留 refs。 |
| PSG-R-02 | project scale 被误解为授权。 | CP gate 和 runtime authorization 被绕过。 | checker 禁止授权语义，文档明确不授权项。 | 移除 gate bias 字段，仅保留 scale observation。 |
| PSG-R-03 | scaffold 覆盖用户文件。 | 数据丢失。 | dry-run 默认、conflict fail。 | 从备份或 VCS 恢复，重新人工合并。 |
| PSG-R-04 | 下游 refresh/stale 依赖未稳定 schema。 | W4 返工。 | 在 CR-C full LLD 冻结 reader contract。 | refresh/stale LLD 等待 schema baseline。 |
