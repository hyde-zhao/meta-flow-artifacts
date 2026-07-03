---
status: baseline
version: "1.0"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
source_plan: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-STATE-ENFORCEMENT-IMPLEMENTATION-PLAN-2026-07-02.md"
baseline_note: "正式产品基线；CR-037 已按用户授权激活，CR-036 仍 blocked / unfinished，不可视为完成或关闭；CR-037 仍为 CP2 pending，未授权实现、CP5、真实运行、生产写入或 quant-lab 发布库修改。"
engagement_mode: meta-self-dev
scenario_subject_type: implementation-carrier
scenario_subject_id: "meta-flow-project-governance-state-enforcement"
target_artifact_type: workflow
governance_mode: review-gated
review_policy: strict
delivery_routing:
  mode: meta-flow-delivery
  output_root: "process/docs/product"
  source: meta-self-dev
total_use_cases: 7
---

# Meta Flow 项目治理与状态强制用户场景

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活、CR-036 暂停未完成和 CP2 pending 不授权实现的状态语义 | 小范围状态语义同步 |
| 1.0 | 2026-07-02 | meta-pm | 基于已批准实施计划建立产品侧场景基线 | 初始化长期可追踪产品基线 |

## 用户画像

| 画像 ID | 角色名称 | 典型背景 | 核心诉求 | 技术水平 |
|---|---|---|---|---|
| P-01 | Host Orchestrator 维护者 | 维护 meta-flow 状态机、检查点、CR 和交付规则 | 防止轻量状态膨胀，确保推进流程可审计、可恢复 | 高级 |
| P-02 | 功能 Agent / Skill 作者 | 编写或维护 meta-flow agent、skill、规则和 CLI | 清楚知道哪些状态可写、如何写、写错时如何被拦截 | 中高级 |
| P-03 | 项目迁移执行者 | 将 quant-lab 等长期项目迁移到新的治理机制 | 在不污染发布库的前提下完成状态清理、路线图刷新和能力引用归一 | 高级 |
| P-04 | 审批者 / Reviewer | 审批高风险流程、迁移计划和治理变更 | 快速看到自动更新范围、人工决策项、风险和回退条件 | 中高级 |

## 成功指标

| 指标 ID | 指标名称 | 度量方式 | 目标值 |
|---|---|---|---|
| SM-01 | Current state 瘦身合规 | `STATE.current.json` schema / size / unknown field 检查 | audit 阶段可报告，enforce 阶段可阻断非法写入 |
| SM-02 | 项目级治理对象可追踪 | `PROJECT.current.json`、`PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml` 引用完整性检查 | 关键 refs 完整，`PROJECT.current.json` 不超过 16KB |
| SM-03 | 影响面语义归一 | 新 CR 的 `impact_surface` 只包含治理面枚举，路径进入 `affected_paths` | 新 CR enforce 阶段 unknown surface 为 0 |
| SM-04 | Roadmap refresh 边界清晰 | refresh result 中自动写入项、must_check、stale_items、follow_up_candidates 可区分 | 不自动修改 quant-lab 发布库 |
| SM-05 | quant-lab 迁移可验证 | 迁移后 state check、capability check、feature check、capability-claims check 结果 | P2 迁移报告可复现、失败项可追踪 |

## 明确排除

- 不新增第二套上下文治理、影响分析、capability 命名、CP result 或 ledger 体系。
- 不声明或实现跨仓原子事务。
- 不把 roadmap refresh 自动写入 quant-lab 发布库代码、测试或正式文档。
- 不直接修改 `process/policies/GATE-PROFILES.json` 来表达 project scale。
- 不让 `capability_refs` 或 `feature_refs` 成为自由字符串命名空间。
- 不把 `PROJECT.current.json` 设计成新巨型状态文件。

## Scenario Gray Areas

本轮输入来自已批准实施计划，灰区已在设计评审中收敛；产品基线保留以下场景取舍，供后续 CP2 / CR 审批追踪。

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 用户选择 | 状态 |
|---|---|---|---|---|---|
| SGA-01 | `STATE.current.json` 使用黑名单还是 allowlist | 决定能否阻止下一个自造字段继续污染轻量状态 | 范围 / 验证 / 后续门控 | allowlist schema + 字段预算 | resolved |
| SGA-02 | Roadmap refresh 是否跨仓自动更新 | 决定 quant-lab 发布库是否被自动改写以及回滚边界 | 范围 / 运行风险 / 交付出口 | 只自动更新过程归档库，发布库只输出 follow-up | resolved |
| SGA-03 | capability / feature 引用是否允许自由字符串 | 决定迁移后冲突检测和能力追踪是否可信 | 验证 / 维护成本 / 数据治理 | 必须引用标准 registry | resolved |
| SGA-04 | 项目规模是否引入更多档位 | 影响 gate profile、状态复杂度和审批负担 | 范围 / 复杂度 / 后续门控 | 使用 `lite / standard / full` 三档 | resolved |

## Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-01 | 更丰富的项目规模矩阵或 `regulated` 独立档 | 设计评审否决项 | 当前可复用 runtime-high-risk、authz、evidence、human gate | 真实受监管项目出现现有 gate 无法表达的合规缺口 |
| DEF-02 | 跨仓事务式 roadmap refresh | 设计评审否决项 | 跨仓原子性成本高且回滚边界不清 | 未来有可靠跨仓事务协调器和明确授权 |
| DEF-03 | 长期消费 Markdown register 或 Python 常量作为 capability registry | P1.2a 备选 | 不利于稳定引用和 checker 实现 | 标准 YAML registry 无法覆盖某类能力状态时重新评审 |
| DEF-04 | 复用 CP result checker 校验 roadmap refresh | P1.4 备选 | result 语义不同，复用会污染 CP 检查模型 | roadmap refresh 与 CP result 出现大量共享字段和统一生命周期时 |

## 使用场景列表

### UC-PG-001：拒绝污染 `STATE.current.json` 的状态写入

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-02 功能 Agent / Skill 作者 |
| 触发条件 | Agent、Skill、CLI 或内部流程准备更新 `process/state/STATE.current.json` |
| 输入 | 待写入 patch、actor、reason、当前 `STATE.current.json` |
| 处理逻辑 | 系统按 allowlist 校验顶层字段，按字段预算校验大小和结构；audit 阶段输出 WARN，enforce 阶段拒绝 unknown / 超预算 / required key 缺失写入 |
| 输出/结果 | 合法写入成功；非法写入产生可追踪错误和检查结果 |
| 前置条件 | allowlist schema、field budgets、受控 update API 已存在 |
| 排除情况 | 不迁移重型状态到 `STATE.current.json`；不允许 agent 直接手工编辑 current state |

### UC-PG-002：以 refs-only 项目状态表达长期项目治理

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 工作流需要表达项目定位、规模、当前项目阶段、活跃目标和 roadmap refs |
| 输入 | 项目 ID、项目定位短字段、project scale、roadmap / milestone / health / deferred index refs |
| 处理逻辑 | 系统在 `STATE.current.json` 仅保留 `project_state_ref`，在 `PROJECT.current.json` 保存短字段和 refs，并校验预算 |
| 输出/结果 | 项目级状态可被下游读取，但不会膨胀 current state |
| 前置条件 | `process/project/` scaffold、`PROJECT.current.json` schema 和预算存在 |
| 排除情况 | 不在 `PROJECT.current.json` 保存 roadmap 全文、deferred 详情或长历史 |

### UC-PG-003：按项目规模提供 gate profile 默认偏好

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 新建或迁移长期项目，需要表达 `lite / standard / full` 项目规模 |
| 输入 | 项目规模、规模原因、gate profile bias refs |
| 处理逻辑 | 系统写入 `PROJECT-SCALE.yaml`，记录默认 gate profile bias 和原因，但不修改 `GATE-PROFILES.json` |
| 输出/结果 | 后续 CR / gate 可读取规模偏好，并保留人工审查边界 |
| 前置条件 | `process/project/PROJECT-SCALE.yaml` 模板和检查逻辑存在 |
| 排除情况 | 不新增五档规模矩阵；不把 project scale 当成自动授权 |

### UC-PG-004：归一 capability / feature 引用来源

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-01 Host Orchestrator 维护者 |
| 触发条件 | CR、roadmap refresh 或迁移流程声明 `capability_refs` / `feature_refs` |
| 输入 | 待引用 capability ID、feature ID、标准 registry |
| 处理逻辑 | 系统校验 capability ID 必须存在于 `CAPABILITY-STATUS.yaml`，feature ID 必须存在于 `FEATURE-REGISTRY.yaml`；缺失时输出 blocked finding 或 FU-RF follow-up |
| 输出/结果 | 引用可解析；无法解析的迁移项不会被自动创造 ID |
| 前置条件 | 标准 capability / feature registry 路径和 checker 已定义 |
| 排除情况 | 不把示例值或历史自由字符串直接升级为正式 ID |

### UC-PG-005：拆分治理影响面、路径和能力引用

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-03 项目迁移执行者 |
| 触发条件 | 新 CR 或迁移报告需要描述影响范围 |
| 输入 | 原始 impact surface 值、文件路径、模块路径、feature / capability 线索 |
| 处理逻辑 | 系统将有限治理面写入 `impact_surface`，路径写入 `affected_paths`，可解析能力写入 `feature_refs` / `capability_refs`；历史漂移进入 migration report |
| 输出/结果 | 冲突检测能基于语义 surface 工作，路径和能力引用各自可检查 |
| 前置条件 | impact surface 枚举、迁移报告格式、新 CR audit / enforce 策略存在 |
| 排除情况 | 不在普通 `cr check` 中对历史 CR 刷屏；不自动创造 feature / capability ID |

### UC-PG-006：执行受边界约束的 roadmap refresh cascade

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 项目定位、milestone、CR 完成或 roadmap 输入发生变化 |
| 输入 | source CR、refresh trigger、当前 project state、roadmap / milestone 对象、发布库 stale 线索 |
| 处理逻辑 | 系统生成独立 ROADMAP-REFRESH result；只自动更新过程归档库机器状态、project 对象、CR index 和 gate ledger；发布库变更只列入 must_check、stale_items、follow_up_candidates |
| 输出/结果 | refresh decision 为 `NO_CHANGE / UPDATED / UPDATED_WITH_DOC_IMPACTS / BLOCKED / FAILED` 之一，并可被 checker 校验 |
| 前置条件 | ROADMAP-REFRESH schema、checker、GATE-LEDGER 事件写入规则存在 |
| 排除情况 | 不复用 CP result checker；不自动修改 quant-lab 发布库 |

### UC-PG-007：迁移 quant-lab 到项目治理基线

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-04 审批者 / Reviewer |
| 触发条件 | P0 / P1 机制实现后，开始真实样本迁移 |
| 输入 | quant-lab 当前 `STATE.current.json`、capability 来源、历史 CR impact surface、roadmap / milestone 线索 |
| 处理逻辑 | 系统清理 current state，归一 capability registry，迁移历史 impact surface，生成 project 状态对象，输出 stale report 和 FU-RF 候选 |
| 输出/结果 | quant-lab 通过 state / capability / feature / capability-claims 检查，发布库只收到 follow-up |
| 前置条件 | P0 / P1 机制和 checker 已可用 |
| 排除情况 | 不与能力实现 CR 混在一起；不自动修改 quant-lab 发布库正式代码或文档 |

## 附录：覆盖自检表

| 维度 ID | 维度名称 | 状态 | 涉及场景 | 备注 |
|---|---|---|---|---|
| D1 | 用户维度 | 已覆盖 | UC-PG-001..007 | 覆盖维护者、作者、迁移者、审批者 |
| D2 | 任务维度 | 已覆盖 | UC-PG-001..007 | 覆盖状态写入、项目治理、引用归一、refresh、迁移 |
| D3 | 动机维度 | 已覆盖 | UC-PG-001..007 | 防污染、防漂移、可审计、可迁移 |
| D4 | 时间维度 | 已覆盖 | UC-PG-001, UC-PG-006, UC-PG-007 | 覆盖 audit -> enforce、P0 -> P1 -> P2 |
| D5 | 环境维度 | 已覆盖 | UC-PG-006, UC-PG-007 | 区分过程归档库与 quant-lab 发布库 |
| D6 | 方式维度 | 已覆盖 | UC-PG-001, UC-PG-006 | 受控 API / CLI / checker / result |
| D7 | 异常维度 | 已覆盖 | UC-PG-001, UC-PG-004, UC-PG-006 | unknown field、缺失 registry、refresh blocked |
| D8 | 集成维度 | 已覆盖 | UC-PG-001..007 | 与 CR、ledger、state-router、agent contract、checker 衔接 |
