---
status: draft
version: "0.2"
complexity: "standard"
selected_option: "guided-cli-regression-workflow"
confirmed: false
confirmed_by: ""
confirmed_at: ""
design_scope: "ptm-te execution and issue-regression capability"
formal_hld: false
---

# ptm-te 方案设计草案

> 本文是 ptm-te “物理用例执行、执行失败定位、禅道问题单回归”能力的方案设计草案。由于全局 `docs/product/USE-CASES.md` 与 `docs/product/REQUIREMENTS.md` 尚未形成 confirmed 基线，本文不作为正式 CP3 HLD；进入 Story 拆解前必须并入正式产品 / 设计基线并完成门禁确认。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-25 | Codex / Host Orchestrator | 初始方案草案，覆盖禅道 CLI 读取、问题单分析、人工逐步确认、回归路径规划、真实执行、回滚和回写 |
| 0.2 | 2026-06-25 | Codex / Host Orchestrator | 补充物理用例执行和执行失败定位 / 调试能力，更新模块职责、流程与追溯矩阵 |

## 1. 问题定义

### 问题陈述

测试工程师在执行物理用例时，需要解析 Markdown 表格用例、准备物理组网、选择 ptm-atomic 原子操作、执行配置 / 流量 / Check 步骤、判定结果，并在失败时定位原因。当前这些动作依赖人工经验，容易出现步骤解释不一致、组网绑定不清、执行证据缺失、失败原因难以复盘等问题。

测试工程师在处理禅道问题单回归时，还需要从 bug 标题、重现步骤、开发定位分析和开发修改自测中理解问题背景，再手工规划回归路径、准备组网、选择原子操作、执行验证并填写测试回归记录。该过程高度自由，依赖测试经验，且存在信息遗漏、扩展测试点遗漏、执行记录不可复盘和回归结果难以被 ptm-tae / ptm-qa / ptm-tm / ptm-tse 消费的问题。

ptm-te 需要成为测试工程师的执行协作体：既能执行物理用例，也能在执行失败后辅助定位，还能读取禅道问题单并完成回归规划、真实执行、回滚和禅道回写。

### 核心价值

- 将禅道问题单中的非结构化信息转为可确认、可执行、可审计的回归计划。
- 将 Markdown 表格物理用例转为可确认、可执行、可回滚的 ptm-atomic op path。
- 将执行失败从“只有失败结论”提升为“有证据链、候选原因和下一步动作”的定位报告。
- 保持测试工程师对高自由度回归任务的控制权，每一步均可确认、修改或停止。
- 将回归过程沉淀为人工可读报告和机械可读事件日志，支撑 ptm-tae 自动化沉淀和 ptm-qa / ptm-tm / ptm-tse 后续分析。
- 在 ptm-atomic 能力缺失时，形成可开发、可验证、可提 PR 的能力补齐路径。

### 目标

| 优先级 | 目标 | 度量方式 |
|---|---|---|
| P0 | 解析并执行 Markdown 表格物理用例 | 15 个核心字段可解析或明确缺失，目标用例可生成执行计划 |
| P0 | 执行失败后形成定位报告 | 配置、流量、日志、环境、工具调用 5 类证据至少覆盖 4 类或说明缺失原因 |
| P0 | 读取并结构化理解单个禅道 bug | 6 个核心字段全部展示或明确缺失 |
| P0 | 默认按问题单重现步骤形成回归 ptm-atomic op path | 每个重现步骤都有 op path、manual step 或 gap 归类 |
| P0 | 每一步人工确认 | 关键节点确认记录覆盖率 100% |
| P0 | 真实执行前完成 rollback-readiness | 所有状态变更 op 具备 rollback_strategy 或阻断执行 |
| P0 | 回写测试回归记录 | 回归完成后生成并写入禅道测试回归字段 / 记录 |
| P1 | 从开发定位分析和开发修改自测中提取扩展测试点 | 每个扩展点具备来源字段、理由、人工确认状态 |
| P1 | 输出双层日志 | 操作日志和详细事件日志均生成 |

### 成功标准

- [ ] 对单个 bug id，可通过禅道读取 CLI 获取并展示 `id / bug 标题 / 重现步骤 / 开发定位分析 / 开发修改自测 / 测试回归` 6 个核心字段。
- [ ] 对单条 Markdown 表格物理用例，可解析元信息、组网、预置条件、测试步骤和预期结果，并生成 ptm-atomic op path 草案。
- [ ] 对一次执行失败，可输出候选原因、证据链、补充定位动作和后续处理建议。
- [ ] 问题理解、组网识别、默认回归路径、扩展测试点、真实执行、回写内容 6 类节点均有人工确认记录。
- [ ] 默认回归路径按问题单重现步骤生成，且每个步骤落到 `ptm-atomic op / manual step / gap` 三类之一。
- [ ] 真实执行前，100% 状态变更 op 通过 rollback-readiness 或被阻断并转入 ptm-atomic 能力补齐。
- [ ] 回归完成后输出 1 份人工操作日志、1 份机械详细事件日志和 1 条禅道回归记录草案 / 回写结果。

### 约束

| 类型 | 约束内容 |
|---|---|
| 外部系统 | 禅道读取与回写初步定位为 CLI 工具模式 |
| 人工交互 | 问题单回归高度自由，每一步必须人工确认 |
| 执行范围 | 默认按重现步骤回归；扩展测试必须用户确认后执行 |
| 回滚 | 采用 PTM-TE-RB-01~06；缺少 rollback_strategy 的状态变更 op 不得真实执行 |
| 组网 | 物理组网绘制遵循 `resource/network-topology` 组网集合格式，不另行设计独立 YAML |
| ptm-atomic | 能力缺失时，ptm-te 需要本地修改 ptm-atomic，验证后推送 PR 分支，由人工发起 merge |
| 定位 | ptm-te 只能输出候选原因和证据链，最终缺陷归因必须人工确认 |

### 非目标（Out of Scope）

- 不自动变更禅道 bug 状态。
- 不在未经人工确认时执行扩展测试。
- 不在本草案中设计全新的 topology schema。
- 不支持多 bug 批量调度；本阶段先覆盖单 bug 对话式回归。
- 不绕过 ptm-atomic 回滚契约执行真实环境状态变更。
- 不把执行失败的候选原因直接升级为正式缺陷结论。

## 2. 架构灰区与方案形成记录

### Architecture Gray Areas

| 灰区 ID | 关键问题 | 为什么会影响架构 | 影响面 | 推荐讨论顺序 | canonical refs | 状态 |
|---|---|---|---|---|---|---|
| AGA-TE-IR-01 | 禅道集成采用 CLI 工具还是多输入适配 | 决定输入模块、凭据边界、测试方式 | 模块 / 安全 / 验证 | 1 | PTM-TE-Q16 | resolved：CLI 工具模式 |
| AGA-TE-IR-02 | 人工确认粒度 | 决定工作流状态机和交互模型 | 交互 / 风险 / 执行效率 | 2 | PTM-TE-Q18 | resolved：每一步确认 |
| AGA-TE-IR-03 | 扩展测试点是否自动执行 | 决定回归范围策略和执行授权 | 范围 / 验证 / 风险 | 3 | PTM-TE-Q19 | resolved：需确认后执行 |
| AGA-TE-IR-04 | 禅道回写是否包含状态流转 | 状态变更权限高于记录回写 | 权限 / 审计 / 外部系统 | 4 | PTM-TE-Q20 | deferred：本阶段只回写回归记录 |
| AGA-TE-EX-01 | 是否必须覆盖物理用例执行 | 决定 ptm-te 是主执行 Agent 还是只做问题单回归工具 | 范围 / 模块 / 验证 | 0 | PTM-TE-Q21 | resolved：必须覆盖 |
| AGA-TE-EX-02 | 是否必须覆盖执行失败定位 | 决定是否需要诊断模块和证据链模型 | 模块 / 质量 / 下游消费 | 0 | PTM-TE-Q21 | resolved：必须覆盖 |

### Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 引导式 CLI 回归工作流 | 与用户确认的 CLI 模式一致；每一步可审计；容易 mock 禅道工具；风险可控 | 交互轮次多；执行效率低于自动化批处理 | 模块 / 交互 / 安全 / 验证 | 推荐 | 当前问题单回归高度自由且每一步需确认；若后续批量回归成熟，可扩展后台模式 |
| B. 禅道直连 API + 自动规划 | 自动化程度高，后续批量处理更方便 | 凭据和权限复杂；容易绕过人工判断；早期验证成本高 | 安全 / 外部系统 / 验证 / 维护 | 不推荐当前阶段 | 只有当 CLI 工具无法满足读取/回写，且权限模型明确时切换 |
| C. 离线导入 + 仅生成回归计划 | 风险最低；不依赖禅道权限；适合验证项目早期 | 不能完成回写；不满足用户要求的真实回归闭环 | 范围 / 价值 / 验证 | 作为 fallback | 禅道 CLI 不可用或验证项目无访问权限时使用 |

**推荐方案**：A. 引导式 CLI 回归工作流。

理由：它符合用户确认的禅道 CLI 模式和“每一步人工确认”约束，同时可以复用 ptm-te 的真实执行、回滚栈、双层日志和 ptm-atomic 缺口开发机制。

## 3. 推荐方案总览

**复杂度模式**：`standard`

| 判定维度 | 依据 | 结论 |
|---|---|---|
| 需求规模 | 覆盖读取、分析、规划、执行、回滚、回写 | standard |
| 角色数量 | 测试工程师 + ptm-te + 禅道 + ptm-atomic + 日志消费者 | standard |
| 状态流转 | 每一步人工确认，存在执行 / 回滚 / gap / PR 分支 | standard |
| 平台适配 | CLI 工具、ptm-atomic、本地 git PR 流程 | standard |
| Story 拆解 | 可拆为 4~5 个 Story | standard |

**系统核心思路**：

ptm-te 有两类入口：物理用例入口和禅道问题单入口。物理用例入口将 Markdown 表格用例解析为可执行计划；禅道问题单入口调用禅道读取 CLI 获取问题单，生成结构化问题理解并逐步与测试工程师确认。两类入口最终都收敛到统一的 topology binding、ptm-atomic op path、rollback-readiness、真实执行、失败定位、双层日志和外部回写 / 报告能力。

**关键架构风格**：引导式状态机 + 管道过滤 + 事件日志。

## 4. 模块职责

| 模块 | 职责 | 输入 | 输出 | 失败行为 |
|---|---|---|---|---|
| Physical Case Parser | 解析 Markdown 表格物理用例 | 物理用例 Markdown | 结构化 physical case record | 字段缺失则标记 missing，不脑补 |
| Feature Skill Router | 根据目录、关键词、op 引用选择特性 Skill | physical case / bug context | feature skill candidates | 无匹配则进入人工选择或 gap |
| Feature Skill Pack | 解释特性语义、提取参数、生成特性级 op 映射和扩展建议 | 结构化用例 / 问题单上下文 | 局部 op mapping、risk notes、gaps | 输出 confidence，不直接执行 |
| ZenTao CLI Adapter | 调用禅道读取 / 回写工具 | bug id、CLI 配置 | 原始问题单、回写结果 | 工具不可用则阻断并提示改用 fallback |
| Bug Parser | 解析核心字段 | 原始问题单 | 结构化 bug record | 缺字段则标记 missing，不脑补 |
| Bug Understanding Dialog | 与测试工程师确认问题理解 | bug record、候选理解 | 确认记录、修正内容 | 未确认不得进入规划 |
| Topology Reasoner | 从问题单和组网规范识别逻辑 / 物理组网 | 重现步骤、开发定位分析、组网集合 | 组网映射草案、物理组网绘图输入 | 组网不清则进入人工确认 |
| Execution Planner | 合并物理用例 / 回归计划 / 特性 Skill 输出，形成统一执行计划 | physical case record、regression plan、feature mappings | execution plan | 冲突时进入人工确认 |
| Regression Planner | 生成默认回归计划和扩展测试候选 | 重现步骤、开发定位分析、自测信息 | 默认计划、扩展候选 | 扩展候选未确认不得执行 |
| Atomic Path Builder | 将步骤转成 ptm-atomic op path | 回归计划、ptm-atomic 索引 | op path、manual steps、gaps | gap 进入 ptm-atomic 能力补齐 |
| Rollback Readiness Checker | 检查 RB-01~06 | op path、ptm-atomic schema | rollback stack plan | 不通过则阻断真实执行 |
| Execution Engine | 调用 ptm-atomic 执行 | 已确认 op path、topology binding | 执行结果、证据 | 失败时触发逆序回滚 |
| Failure Diagnoser | 聚合失败证据，输出候选原因和补充定位动作 | execution events、evidence、expected result | diagnosis report、diagnostic op path | 证据不足时标记 unknown 并请求人工确认 |
| Log Writer | 生成操作日志和详细事件日志 | run context、events、evidence | Markdown 报告、JSONL/NDJSON 日志 | 写入失败阻断回写 |
| Regression Record Writer | 生成并回写禅道测试回归记录 | 执行报告、人工确认 | 禅道回归记录 | 回写失败保留本地报告和重试建议 |

## 5. 关键数据对象

### 5.1 Bug Record

```yaml
bug:
  id: "BUG-12345"
  title: ""
  reproduce_steps: ""
  dev_root_cause_analysis: ""
  dev_change_self_test: ""
  test_regression: ""
  raw_source_ref: "zentao://bug/BUG-12345"
  missing_fields: []
```

### 5.2 Physical Case Record

```yaml
physical_case:
  case_id: "PC-MATCH-5T-001-01"
  title: ""
  directory:
    level3: ""
    level4: ""
    level5: ""
  priority: "P0 | P1 | P2 | P3"
  topology_description: ""
  topology_constraints: ""
  preconditions: ""
  test_steps: []
  expected_results: []
  keywords: []
  test_type: ""
  automation_flag: "yes | no"
  missing_fields: []
```

### 5.3 Confirmation Record

```yaml
confirmation:
  id: "CONF-001"
  bug_id: "BUG-12345"
  stage: "bug_understanding | topology | default_plan | extension_point | execution_step | regression_record"
  prompt: ""
  proposed_decision: ""
  user_response: "approve | modify | reject"
  resolved_content: ""
  confirmed_at: "2026-06-25T00:00:00+08:00"
```

### 5.4 Regression Plan

```yaml
regression_plan:
  bug_id: "BUG-12345"
  strategy: "reproduce-steps-first"
  default_path:
    - step_id: "S1"
      source: "reproduce_steps"
      intent: ""
      action_type: "ptm_atomic | manual | gap"
      op_refs: []
      expected_result: ""
      confirmation_ref: "CONF-..."
  extension_candidates:
    - candidate_id: "EXT-001"
      source_field: "dev_root_cause_analysis | dev_change_self_test"
      reason: ""
      risk_area: ""
      status: "pending | approved | rejected"
```

### 5.5 Atomic Operation Path

```yaml
atomic_path:
  bug_id: "BUG-12345"
  topology_binding_ref: "TOPO-BIND-001"
  steps:
    - step_id: "S1"
      op_id: "fw_config_policy_route"
      inputs: {}
      expected: {}
      rollback_strategy:
        type: "inverse_op"
        op_id: "fw_delete_policy_route"
      readiness: "pass | blocked"
```

### 5.6 Diagnosis Report

```yaml
diagnosis_report:
  run_id: "RUN-..."
  failed_step_id: "S3"
  failure_type: "config_failed | traffic_mismatch | log_mismatch | env_unreachable | ptm_atomic_gap | unknown"
  evidence_refs:
    config: []
    traffic: []
    logs: []
    topology: []
    atomic_calls: []
  candidate_causes:
    - cause_id: "CAUSE-001"
      category: "product_defect | environment_issue | case_issue | ptm_atomic_issue | unknown"
      confidence: "high | medium | low"
      evidence_refs: []
      recommended_next_action: ""
  user_confirmation_ref: "CONF-..."
```

### 5.7 Regression Record

```yaml
regression_record:
  bug_id: "BUG-12345"
  conclusion: "pass | fail | blocked | pass_with_risk"
  default_regression_result: ""
  extension_test_result: ""
  evidence_refs: []
  rollback_result: ""
  ptm_atomic_gap_refs: []
  pr_refs: []
  operator_confirmation_refs: []
  writeback_target: "zentao.test_regression"
```

## 6. 核心流程

### 6.1 物理用例执行流程

```text
Markdown physical case
  -> Physical Case Parser 解析表格行
  -> Feature Skill Router 选择特性 Skill
  -> Feature Skill Pack 输出步骤语义和局部 op mapping
  -> Topology Reasoner 绘制 / 绑定物理组网
  -> Execution Planner 合并执行计划
  -> Atomic Path Builder 生成 ptm-atomic op path
  -> rollback-readiness 检查
  -> 人工确认执行计划
  -> Execution Engine 逐步执行
  -> 失败则进入 Failure Diagnoser
  -> 必要时逆序回滚
  -> Log Writer 输出操作日志和详细事件日志
```

### 6.2 问题单回归流程

```text
bug id
  -> ZenTao CLI Adapter 读取问题单
  -> Bug Parser 提取核心字段
  -> 人工确认问题理解
  -> Topology Reasoner 识别组网 / 绘制物理组网
  -> 人工确认组网与复现路径
  -> Regression Planner 生成默认回归计划
  -> 人工确认默认计划
  -> 从开发定位分析 / 自测中提取扩展测试点
  -> 人工逐项确认扩展测试点
  -> Atomic Path Builder 生成 ptm-atomic op path
  -> rollback-readiness 检查
  -> 人工逐步确认并执行
  -> 失败则逆序回滚
  -> 生成操作日志 + 详细事件日志
  -> 人工确认禅道回归记录
  -> ZenTao CLI Adapter 回写测试回归记录
```

### 6.3 失败定位流程

```text
execution step failed
  -> Failure Diagnoser 分类失败类型
  -> 聚合配置 / 流量 / 日志 / 环境 / ptm-atomic 调用证据
  -> 判断是否需要补充只读诊断动作
  -> 人工确认补充定位 op path
  -> 执行诊断动作
  -> 输出候选原因和证据链
  -> 人工确认后选择：回滚 / 重试 / 记录 gap / 转问题单回归输入
```

### 6.4 人工确认门禁

| Gate | 确认内容 | 未确认行为 |
|---|---|---|
| IR-G0 | 物理用例解析结果和目标用例选择 | 不进入执行计划 |
| IR-G1 | 问题单字段和问题理解 | 不进入组网分析 |
| IR-G2 | 问题组网和重现路径 | 不生成执行计划 |
| IR-G3 | 默认回归计划 | 不生成 op path |
| IR-G4 | 每个扩展测试点 | 未确认项不执行 |
| IR-G5 | ptm-atomic gap 处理方式 | 不开发 / 不 mock / 不执行相关路径 |
| IR-G6 | 每个真实执行步骤 | 不调用 ptm-atomic |
| IR-G7 | 失败定位动作和回滚动作 | 未确认则不执行额外诊断；若环境风险可控则执行预先批准的回滚，否则等待人工确认 |
| IR-G8 | 禅道回归记录内容 | 不回写禅道 |

### 6.5 ptm-atomic 缺口分支

```text
op path 生成失败或 readiness blocked
  -> 分类 gap: missing_op / missing_rollback / missing_query / missing_verify / parameter_unknown
  -> 生成 ptm-atomic 开发任务草案
  -> 人工确认是否进入开发
  -> 拉取 ptm-atomic master
  -> 创建开发分支
  -> 实现 op / rollback / query / verify 能力
  -> 运行测试
  -> 推送分支
  -> 输出验证报告 + 修改影响分析
  -> 人工发起 merge 请求
  -> 回到 ptm-te 回归计划
```

## 7. 日志与报告

### 7.1 操作日志（人工可读）

| 字段 | 说明 |
|---|---|
| bug id / 标题 | 问题单身份 |
| 问题理解 | ptm-te 复述并经确认的理解 |
| 回归范围 | 默认重现步骤和已批准扩展测试 |
| 组网 | 使用的物理组网、逻辑组网映射、关键端口 |
| 步骤结果 | 每一步动作、结果、证据 |
| 回滚结果 | 是否触发回滚、回滚步骤、失败项 |
| 结论 | pass / fail / blocked / pass_with_risk |
| 禅道回写 | 回写内容和结果 |

### 7.2 详细事件日志（机械可读）

事件类型建议：

| event | 说明 |
|---|---|
| `bug_loaded` | 禅道问题单读取完成 |
| `physical_case_loaded` | 物理用例读取完成 |
| `field_missing` | 核心字段缺失 |
| `confirmation_requested` / `confirmation_resolved` | 人工确认请求与结果 |
| `feature_skill_selected` | 特性 Skill 路由完成 |
| `topology_inferred` | 组网识别或绘制完成 |
| `execution_plan_created` | 物理用例执行计划生成 |
| `default_plan_created` | 默认回归计划生成 |
| `extension_candidate_created` | 扩展测试候选生成 |
| `atomic_path_created` | 原子操作路径生成 |
| `readiness_blocked` | rollback-readiness 阻断 |
| `op_started` / `op_succeeded` / `op_failed` | ptm-atomic 执行事件 |
| `diagnosis_started` / `diagnosis_completed` | 失败定位事件 |
| `diagnostic_op_started` / `diagnostic_op_completed` | 补充诊断动作事件 |
| `rollback_started` / `rollback_op_succeeded` / `rollback_op_failed` | 回滚事件 |
| `zentao_writeback_prepared` / `zentao_writeback_done` | 回写事件 |

## 8. Use Case → Architecture Traceability

| Use Case | 支撑模块 / 组件 | 关键流程 | 异常 / 失败路径 | 验证方式 |
|---|---|---|---|---|
| UC-TE-EX-01 | Physical Case Parser, Feature Skill Router, Execution Planner, Atomic Path Builder, Execution Engine | 物理用例解析、计划、执行、日志 | 字段缺失、组网不清、op gap、readiness blocked | Markdown fixture + mock ptm-atomic |
| UC-TE-EX-02 | Failure Diagnoser, Execution Engine, Log Writer | 失败分类、证据聚合、补充诊断、候选原因 | 证据不足、诊断 op 缺失、回滚失败 | failure fixture + diagnostic event log |
| UC-TE-IR-01 | ZenTao CLI Adapter, Bug Parser, Dialog | 读取问题单并确认理解 | CLI 不可用、字段缺失、用户修正理解 | mock CLI fixture + 人工确认记录 |
| UC-TE-IR-02 | Topology Reasoner, Dialog | 识别组网与复现路径 | 组网不清、物理资源不匹配 | topology fixture + 确认记录 |
| UC-TE-IR-03 | Regression Planner, Atomic Path Builder, Readiness Checker | 默认重现步骤转 op path | op 缺失、rollback 缺失、参数不明 | plan fixture + readiness check |
| UC-TE-IR-04 | Regression Planner, Dialog | 扩展测试点确认 | 扩展点来源不清、用户拒绝 | 扩展候选 fixture + 确认记录 |
| UC-TE-IR-05 | Execution Engine, Log Writer, ZenTao CLI Adapter | 执行、回滚、记录、回写 | 执行失败、回滚失败、回写失败 | mock ptm-atomic + 回写 dry-run |

## 9. 风险与控制

| 风险 | 影响 | 控制 |
|---|---|---|
| 物理用例字段缺失或格式漂移 | 解析失败或执行计划错误 | 字段缺失显式标记；用例行需人工确认 |
| 禅道字段不稳定 | 解析失败或误判 | 字段缺失显式标记；不脑补；保留原文 |
| 每一步确认导致交互成本高 | 回归效率下降 | 保留明确 stage 和确认记录，后续可在用户授权后批量确认低风险步骤 |
| 扩展测试点过多 | 回归范围膨胀 | 默认只执行重现步骤；扩展点逐项确认 |
| ptm-atomic 缺少反操作 | 真实执行风险 | rollback-readiness 阻断，进入 ptm-atomic 能力补齐 |
| 定位报告误判为产品缺陷 | 误导修复方向 | 只输出候选原因和证据链，最终归因由人工确认 |
| 禅道回写误写 | 外部系统污染 | 回写前生成草案并人工确认；不自动流转状态 |
| 开发定位分析质量不足 | 扩展测试建议不可靠 | 标注 fact_status，来源不清则不执行扩展 |

## 10. 分阶段落地建议

| Phase | 范围 | 退出条件 |
|---|---|---|
| P0 | 物理用例 Markdown 解析、特性 Skill 路由、执行计划草案 | 可从物理用例 fixture 生成 execution plan |
| P1 | 禅道 CLI mock 读取、字段解析、问题理解确认 | 可从 fixture bug 生成结构化 bug record 和确认记录 |
| P2 | 默认重现步骤回归规划、op path 生成、readiness check | 可把重现步骤映射为 op/manual/gap 并阻断无回滚 op |
| P3 | 扩展测试点提取、失败定位与人工确认 | 可从开发定位分析 / 自测中生成扩展候选；可从失败 fixture 生成定位报告 |
| P4 | 真实执行与双层日志 | 可在验证项目中执行 mock / controlled ptm-atomic path 并生成日志 |
| P5 | 禅道回写和 ptm-atomic 缺口 PR 流程 | 可回写测试回归记录；可对缺失 op 走 ptm-atomic 分支 |

## 11. ADR 候选

| ADR ID | 决策 | 推荐 | 备选 | 理由 |
|---|---|---|---|---|
| ADR-TE-IR-01 | 禅道集成形态 | CLI Adapter | API Adapter / 离线导入 | 与用户确认一致，降低凭据和权限复杂度 |
| ADR-TE-IR-02 | 回归策略 | 默认按重现步骤，扩展测试确认后执行 | 自动执行所有扩展 / 只输出计划 | 保持可控性，符合“每一步确认” |
| ADR-TE-IR-03 | 确认模型 | 每一步人工确认 | 仅关键门禁确认 | 用户明确要求问题单回归每一步确认 |
| ADR-TE-IR-04 | 回写策略 | 只回写测试回归记录 | 自动修改 bug 状态 | 当前未授权状态流转，记录回写风险更低 |
| ADR-TE-EX-01 | ptm-te 组织形态 | 单 Agent + 多特性 Skill Pack | 多特性多 Agent | 统一执行状态、回滚栈、日志和禅道回写，特性知识通过 Skill 扩展 |
| ADR-TE-EX-02 | 定位结论策略 | 输出候选原因 + 证据链，人工确认最终归因 | Agent 直接判定产品缺陷 | 避免误归因，保留测试工程师控制权 |

## 12. 自审记录

| 检查项 | 结果 | 说明 |
|---|---|---|
| Use Case 覆盖 | PASS | 7 个 UC 均有架构支撑，包含物理用例执行和失败定位 |
| 候选方案比较 | PASS | 比较 CLI / API / 离线导入三种方案 |
| 人工确认门禁 | PASS | IR-G1~IR-G8 覆盖关键节点 |
| 回滚契约 | PASS | 复用 PTM-TE-RB-01~06 |
| 外部系统权限 | WARN | 禅道状态流转未授权，本阶段仅回写回归记录 |
| 正式 HLD 前置 | BLOCKED | 全局 USE-CASES / REQUIREMENTS 尚未 confirmed，本文不能作为 CP3 HLD |
