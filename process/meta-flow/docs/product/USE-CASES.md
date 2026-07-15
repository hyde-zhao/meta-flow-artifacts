---
status: baseline
version: "1.4"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
source_plan: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-STATE-ENFORCEMENT-IMPLEMENTATION-PLAN-2026-07-02.md"
baseline_note: "正式产品基线；CR-047 已通过 CP2，正在进行 workflow truth 与交付治理的 CP3 架构设计；CR-046 保持 closed / READY_WITH_RISK，历史时序不得改写；当前未授权真实运行、凭据、生产写入、发布、交易或仓库推送。"
engagement_mode: meta-self-dev
scenario_subject_type: implementation-carrier
scenario_subject_id: "meta-flow"
target_artifact_type: workflow
governance_mode: review-gated
review_policy: strict
delivery_routing:
  mode: meta-flow-delivery
  output_root: "process/docs/product"
  source: meta-self-dev
active_change_ref: "CR-047"
total_use_cases: 19
---

# Meta Flow 项目治理与状态强制用户场景

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活、CR-036 暂停未完成和 CP2 pending 不授权实现的状态语义 | 小范围状态语义同步 |
| 1.2 | 2026-07-11 | meta-pm | 为 CR-046 增量加入可信时序、平台调度证明、checker 重放、token telemetry 与 append-only 历史迁移场景；保留 UC-PG-001..007 | 原文档增量更新 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：显式加入 compaction 语义保持、通用 post-close correction、机器生成 audit report、null-provenance dogfooding 与 session-observed/repository-unverifiable dispatch 披露；保留全部 UC ID | 原文档增量更新 |
| 1.4 | 2026-07-13 | meta-pm | 为 CR-047 增量加入跨设备 workflow truth、canonical CR tracking、artifact/docs 路由、Doctor、clean-clone guardrail、Ruff、非交互安装与 CR-046 状态收敛场景；保留全部既有 UC ID | 原文档增量更新 |
| 1.0 | 2026-07-02 | meta-pm | 基于已批准实施计划建立产品侧场景基线 | 初始化长期可追踪产品基线 |

## 用户画像

| 画像 ID | 角色名称 | 典型背景 | 核心诉求 | 技术水平 |
|---|---|---|---|---|
| P-01 | Host Orchestrator 维护者 | 维护 meta-flow 状态机、检查点、CR 和交付规则 | 防止轻量状态膨胀，确保推进流程可审计、可恢复 | 高级 |
| P-02 | 功能 Agent / Skill 作者 | 编写或维护 meta-flow agent、skill、规则和 CLI | 清楚知道哪些状态可写、如何写、写错时如何被拦截 | 中高级 |
| P-03 | 项目迁移执行者 | 将 quant-lab 等长期项目迁移到新的治理机制 | 在不污染发布库的前提下完成状态清理、路线图刷新和能力引用归一 | 高级 |
| P-04 | 审批者 / Reviewer | 审批高风险流程、迁移计划和治理变更 | 快速看到自动更新范围、人工决策项、风险和回退条件 | 中高级 |
| P-05 | 工作流审计者 | 复核 checkpoint、gate、dispatch、state 与 ledger 的一致性 | 以机器证据重建实际发生顺序，并区分平台证明、仓库证明与不可用证据 | 高级 |
| P-06 | 证据生产者 / 成本分析者 | 维护自动检查、dispatch producer、context/read 记录与成本报告 | 让每次 attempt、checker 身份、输入摘要和 token 用量具有一致且不可伪造的语义 | 高级 |

## 成功指标

| 指标 ID | 指标名称 | 度量方式 | 目标值 |
|---|---|---|---|
| SM-01 | Current state 瘦身合规 | `STATE.current.json` schema / size / unknown field 检查 | audit 阶段可报告，enforce 阶段可阻断非法写入 |
| SM-02 | 项目级治理对象可追踪 | `PROJECT.current.json`、`PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml` 引用完整性检查 | 关键 refs 完整，`PROJECT.current.json` 不超过 16KB |
| SM-03 | 影响面语义归一 | 新 CR 的 `impact_surface` 只包含治理面枚举，路径进入 `affected_paths` | 新 CR enforce 阶段 unknown surface 为 0 |
| SM-04 | Roadmap refresh 边界清晰 | refresh result 中自动写入项、must_check、stale_items、follow_up_candidates 可区分 | 不自动修改 quant-lab 发布库 |
| SM-05 | quant-lab 迁移可验证 | 迁移后 state check、capability check、feature check、capability-claims check 结果 | P2 迁移报告可复现、失败项可追踪 |
| SM-06 | 时序负例阻断 | chronology negative fixtures 的 checker 结果 | 非法 gate / dispatch / result / state 顺序 100% 被拒绝 |
| SM-07 | Evidence attempt 完整性 | execution/check attempt 的 terminal status、supersession 与 final correlation 覆盖率 | 适用 attempt 覆盖率 100%，final result 只引用最终 terminal attempt |
| SM-08 | Telemetry 诚实性 | usage `measurement_status` 与 measured/proxy 字段检查 | 适用记录覆盖率 100%；不可用写 `unavailable`，估算值不得标为 measured |
| SM-09 | 历史重放可复现 | checker identity、schema/policy hash、as-executed/current-replay 报告 | quant-lab CR-163 current replay 23/23 PASS；lineage 业务源码 diff 为 0 |
| SM-10 | Compaction 语义保持 | compact/restore 前后关系图和 terminal selection 对比 | event/dispatch/attempt/run 标识不混淆；attempt、correction、workflow-health refs 语义差异为 0 |
| SM-11 | Post-close correction 可审计 | versioned correction schema、允许字段、supersedes closure 与 audit trail 检查 | 非法历史改写 100% 拒绝；合法 correction 100% 可独立重放 |
| SM-12 | Audit report 统计可信 | 机器生成报告与已知 fixture 逐维对账 | event rows、attempts、threads、terminal outcomes 与 measured/proxy/unavailable token 指标 100% 正确，且携带 checker provenance/input hashes |
| SM-WT-01 | 跨设备 workflow truth 一致 | 两台 clean clone 分别 link 同一版本 artifact 后比对 state/current/CR index | `active_change`、phase、pending gate、canonical CR 状态差异为 0 |
| SM-WT-02 | Canonical CR tracking 可执行 | `meta-flow check cr-tracking --project-root .` | 退出码 0；canonical 目录无 legacy YAML 干扰；CR-033 仅以 candidate 身份可追踪 |
| SM-WT-03 | Workspace / docs 路由确定 | clean clone + `workspace link` + `workspace check` | `process_link_health=ok`；内部产品文档只有 1 个 canonical artifact 路径；根 `docs/` 继续只承载公开入口 |
| SM-WT-04 | Doctor 阻断归零 | `meta-flow doctor all --project-root .` | 退出码 0；阻断错误为 0；warning 单独披露且不冒充 blocker |
| SM-WT-05 | 静态质量门闭合 | `ruff check .` 与既有全量 pytest | Ruff error=0；不少于 377 个测试及 70 个 subtests 全部通过 |
| SM-WT-06 | Clean-clone guardrail 自洽 | `git archive HEAD` 等价 clean tree 运行 guardrail | 不依赖 ignored 根规则或本机 cache；guardrail 退出码 0 |
| SM-WT-07 | 非交互安装入口可复现 | Codex / Claude / Qoder project-scope full dry-run | 3/3 命令显式提供 `--project-dir` 且退出码 0 |
| SM-WT-08 | CR-046 状态收敛诚实 | 对产品矩阵、CR、CP7 与 release 状态做交叉检查 | 7/7 Story 显示 implemented + `PASS_WITH_RISK`；平台 receipt / 独立 QA 不被写成已具备 |

## 明确排除

- 不新增第二套上下文治理、影响分析、capability 命名、CP result 或 ledger 体系。
- 不声明或实现跨仓原子事务。
- 不把 roadmap refresh 自动写入 quant-lab 发布库代码、测试或正式文档。
- 不直接修改 `process/policies/GATE-PROFILES.json` 来表达 project scale。
- 不让 `capability_refs` 或 `feature_refs` 成为自由字符串命名空间。
- 不把 `PROJECT.current.json` 设计成新巨型状态文件。
- 不伪造历史 platform receipt、签名、token telemetry 或 checker identity。
- 不修改 quant-lab lineage contract、recorder、producer、consumer 或 admission 业务实现。
- 不让 CP2 approval 隐式授权 credentials、runtime、production write、publish、交易、commit 或 push。

## Scenario Gray Areas

本轮输入来自已批准实施计划，灰区已在设计评审中收敛；产品基线保留以下场景取舍，供后续 CP2 / CR 审批追踪。

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 用户选择 | 状态 |
|---|---|---|---|---|---|
| SGA-01 | `STATE.current.json` 使用黑名单还是 allowlist | 决定能否阻止下一个自造字段继续污染轻量状态 | 范围 / 验证 / 后续门控 | allowlist schema + 字段预算 | resolved |
| SGA-02 | Roadmap refresh 是否跨仓自动更新 | 决定 quant-lab 发布库是否被自动改写以及回滚边界 | 范围 / 运行风险 / 交付出口 | 只自动更新过程归档库，发布库只输出 follow-up | resolved |
| SGA-03 | capability / feature 引用是否允许自由字符串 | 决定迁移后冲突检测和能力追踪是否可信 | 验证 / 维护成本 / 数据治理 | 必须引用标准 registry | resolved |
| SGA-04 | 项目规模是否引入更多档位 | 影响 gate profile、状态复杂度和审批负担 | 范围 / 复杂度 / 后续门控 | 使用 `lite / standard / full` 三档 | resolved |
| SGA-05 | 平台没有可验证 receipt 时是否推断为已证明 | 决定 dispatch provenance 是否可信 | 安全 / 审计 / 降级 | 明确记录 `unavailable`，保留 session-observed 与 repository-verifiable 分层，不合成 receipt | resolved |
| SGA-06 | token telemetry 缺失时能否用估算替代实测 | 决定成本报告是否误导审批与优化 | 度量 / 审计 / 用户信任 | `measured`、`estimated`、`unavailable` 分离；估算仅作 proxy | resolved |
| SGA-07 | quant-lab CR-163 历史证据如何修正 | 决定迁移能否保留事实链和可追溯性 | 迁移 / 回滚 / 验证 | append-only correction / supersession / migration events；不改写原事件 | resolved |
| SGA-08 | checker 重放只看当前结论还是保留执行时结论 | 决定历史证据能否解释 checker 演进 | 兼容性 / 审计 / 发布 | 同时记录 checker version/commit/schema-policy hash 与 as-executed/current-replay 双口径 | resolved |
| SGA-WT-01 | 根 `AGENTS.md` 的 canonical source 是 tracked 根文件，还是 tracked `delivery/rules/AGENTS.md` + generated wrapper | 决定 clean clone 能否独立通过 guardrail，以及个人配置是否会混入共享规则 | 交付出口 / 维护成本 / clean-clone 验证 | tracked `delivery/rules/AGENTS.md` + generated root wrapper | resolved-by-user / CP2-DQ-01 |
| SGA-WT-02 | Doctor 的绿色是否要求零 warning，还是只要求零 blocking error 并完整披露 warning | 决定历史兼容提示、空 ledger 等 warning 是否阻断发布 | 验证 / 门控 / 发布结论 | blocking error=0；warning 计数与披露 | resolved-by-user / CP2-DQ-02 |
| SGA-WT-03 | 历史超预算证据采用 compact summary + archive/hash，还是原位压缩 | 决定历史不可变性、读取预算和恢复成本 | 审计 / 维护 / 回退 | active/default-read 严格预算；closed 历史用 summary/index/hash/correction/archive，不改写 | resolved-by-user / CP2-DQ-03 |
| SGA-WT-04 | 平台 receipt、独立 QA 与真实 pilot 缺失是否纳入本轮强行“补绿” | 决定是否会把不可用平台事实伪装为仓库证明 | 安全 / 权限 / 发布 | 固定为不伪造；保留 `READY_WITH_RISK` / follow-up，真实运行需独立授权 | resolved |

## Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-01 | 更丰富的项目规模矩阵或 `regulated` 独立档 | 设计评审否决项 | 当前可复用 runtime-high-risk、authz、evidence、human gate | 真实受监管项目出现现有 gate 无法表达的合规缺口 |
| DEF-02 | 跨仓事务式 roadmap refresh | 设计评审否决项 | 跨仓原子性成本高且回滚边界不清 | 未来有可靠跨仓事务协调器和明确授权 |
| DEF-03 | 长期消费 Markdown register 或 Python 常量作为 capability registry | P1.2a 备选 | 不利于稳定引用和 checker 实现 | 标准 YAML registry 无法覆盖某类能力状态时重新评审 |
| DEF-04 | 复用 CP result checker 校验 roadmap refresh | P1.4 备选 | result 语义不同，复用会污染 CP 检查模型 | roadmap refresh 与 CP result 出现大量共享字段和统一生命周期时 |
| DEF-EI-001 | 跨平台统一加密签名 receipt | SGA-05 | 平台能力并不一致，本轮只能诚实表达 receipt 可用性和证据层级 | 所有目标平台提供稳定可验证签名契约后另行立项 |
| DEF-EI-002 | 用估算 token 建立计费或配额门禁 | SGA-06 | 估算不是平台实测，不能成为强制成本决策依据 | 平台 telemetry 覆盖稳定且误差模型经独立验证后重新评审 |
| DEF-WT-001 | 平台签发的 custom-agent / model / profile receipt 与独立 runtime attestation | CR-045/046 遗留风险、SGA-WT-04 | 当前平台 surface 不提供仓库可验证 receipt，本 CR 不授权真实 runtime 或 SaaS | 平台公开稳定 receipt contract，且用户另行授权真实运行验证 |
| DEF-WT-002 | CR-033 MF-018 runtime trace / SaaS 真实验证 | follow-up candidate | 缺 C0 runtime target、凭据边界和独立运行授权 | 用户正式启动新 CR，提供 runtime target 与最小权限授权 |

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

### UC-EI-001：重建可信的 gate 与结果时序

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-01 Host Orchestrator 维护者 |
| 触发条件 | 自动检查、条件式批准、人工门、state finalization 或 ledger 事件需要被审计或重放 |
| 输入 | checkpoint result、human checkpoint、gate/state/ledger 事件、check attempt 与输入 artifact hash |
| 处理逻辑 | 校验事件因果顺序、条件式批准前置、check attempt、supersession、最终 dispatch correlation 和跨真相源一致性；compaction/restore 还必须保持 event/dispatch/attempt/run 标识、terminal selection、correction chain 与 workflow-health refs；非法顺序或语义丢失 fail-closed |
| 输出/结果 | 可机器重建的有效时序与 provenance-bearing 机器 audit report，或带对象 ID、冲突字段和路由建议的阻断 finding；报告分开统计 event rows、attempts、threads、terminal outcomes 与 measured/proxy/unavailable token 指标 |
| 前置条件 | 事件时间、attempt ID、correlation refs 与 checker contract 可用 |
| 排除情况 | 不以文件修改时间代替事件因果；不回填伪造的历史批准时间 |

### UC-EI-002：证明平台调度与 attempt 生命周期

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-06 证据生产者 / 成本分析者 |
| 触发条件 | 功能 agent 被 spawn/resume/send_input，发生 retry、supersession 或 terminal closure |
| 输入 | dispatch event、platform receipt（若平台提供）、agent/thread ID、attempt 状态和 supersession refs |
| 处理逻辑 | 区分 platform-attested、session-observed、repository-verifiable 与 unavailable；要求每个 attempt 最终 closed，并让 retry 指向被替代 attempt；仅在当前 session 观察到 agent/thread/tool 且仓库无平台 receipt 时必须披露 `session-observed/repository-unverifiable` |
| 输出/结果 | 可追踪到终态的 dispatch chain；缺 receipt 时显式 `unavailable` 或 `session-observed/repository-unverifiable`，不得升级为 repository-verified/platform-attested |
| 前置条件 | producer 能写 attempt/correlation 字段；平台 receipt 能力按实际报告 |
| 排除情况 | 不合成平台签名或 receipt；handoff 文件本身不等于真实调度 |

### UC-EI-003：用已识别 checker 双口径重放证据

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-03 项目迁移执行者 |
| 触发条件 | 历史 CP/CR 证据需要解释执行时结果，或用当前 checker 评估兼容性 |
| 输入 | evidence bundle、checker version/commit、schema/policy hash、执行时结论与当前 checker |
| 处理逻辑 | 保留 as-executed 结论和 checker identity，再运行 current-replay；对差异使用通用、版本化、append-only post-close correction lifecycle，限制允许修正字段和范围，并记录 author/reason/evidence/supersedes 与独立 audit trail；CR-046 CP1/CP2 原始 null-provenance results 保留为 dogfooding 输入，不静默改写 |
| 输出/结果 | 同时包含 as-executed/current-replay、checker provenance、输入 hash 和差异分类的 replay report；null provenance 在 strict profile 下失败或明确标为 legacy/unavailable；合法 correction 可独立校验 |
| 前置条件 | 历史证据可读；无法识别的 checker 字段可明确标为 unavailable |
| 排除情况 | 不以当前 PASS 改写历史 FAIL/PARTIAL；不让 legacy YAML 干扰 canonical JSON checker |

### UC-EI-004：诚实度量工作流 token 成本

| 字段 | 内容 |
|---|---|
| 使用角色 | P-06 证据生产者 / 成本分析者、P-04 审批者 / Reviewer |
| 触发条件 | agent turn、checker run、context expansion 或阶段交接需要报告成本 |
| 输入 | 平台报告 token usage（若有）、measurement status、proxy estimate、阶段/CR/agent attribution |
| 处理逻辑 | 将 measured、estimated/proxy 和 unavailable 分栏；只把平台报告值计为 measured，并按 CR/阶段/agent/attempt 聚合；机器 audit report 不得以 dispatch event rows 代替 attempt/thread 数或推断 token 占比 |
| 输出/结果 | 可比较且不会把估算冒充实测的 workflow cost report，以及带 checker provenance/input hashes 的机器 audit report |
| 前置条件 | telemetry 字段和 attribution contract 已定义 |
| 排除情况 | 不从文本长度反推“实测 token”；不因 telemetry 缺失阻止如实记录 unavailable |

### UC-EI-005：以 append-only 方式迁移 quant-lab CR-163 证据

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-05 工作流审计者 |
| 触发条件 | Meta Flow 的 evidence-integrity contract 已实现并进入 CR-163 验收试点 |
| 输入 | quant-lab CR-163 既有 process evidence、migration manifest、当前 checker 和 immutable business-code boundary |
| 处理逻辑 | 先消费通用 post-close correction lifecycle，再仅追加 migration/correction/supersession events 与 replay fixture，校验 23 个目标证据，并检查 lineage 业务源码 diff 为 0；pilot 不得自创专用 correction 语义 |
| 输出/结果 | 23/23 current replay PASS 的可重放试点证据，或逐项阻断 finding；原历史行保持不变 |
| 前置条件 | 相应 Story 已通过设计门且迁移路径获得独立授权 |
| 排除情况 | 不修改或重做 quant-lab lineage 业务实现；不访问 credentials、runtime 或 production data |

### UC-WT-001：让 state、current 与 canonical CR index 收敛到同一事实

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-05 工作流审计者 |
| 触发条件 | clone、link、恢复工作流或执行 CR/state 检查 |
| 输入 | `STATE.current.json`、`CURRENT.json`、canonical `CR-INDEX.json`、正式 CR 生命周期状态 |
| 处理逻辑 | 以 State v2 与 JSON index 为机器入口，检查 active CR 存在且未关闭、CURRENT refs 一致、candidate 不被当作 active，并拒绝 legacy YAML 干扰 |
| 输出/结果 | 单一可执行 workflow truth；已关闭 CR 不可能继续作为 active，CR-033 保持 candidate |
| 前置条件 | `process` 路由健康，canonical JSON index 可读 |
| 排除情况 | 不用 Markdown summary 覆盖机器真相；不把 candidate 自动激活 |

### UC-WT-002：在 clean clone 中确定性建立 process 与文档路由

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-03 项目迁移执行者 |
| 触发条件 | 同时 clone `meta-flow` 与 `meta-flow-artifacts` 后首次初始化工作区 |
| 输入 | project root、artifact root、project name、`.meta-flow-process.yaml` |
| 处理逻辑 | 从源码项目执行一次 `workspace link`，使用锚点 + 相对路径记录路由；内部产品/设计/质量文档写入 artifact process 路径，根 `docs/` 保持公开入口 |
| 输出/结果 | `process_link_health=ok`，不同设备看到同一版本的过程真相源，不创建第二套 `docs/product` |
| 前置条件 | 两个仓库存在且 artifact 仓包含 `process/meta-flow` |
| 排除情况 | 不在 artifact root 上创建自指链接；不处理 prelink backup |

### UC-WT-003：让 Doctor 结果区分阻断、warning 与历史审计事实

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 发布前、CP8 前或跨设备恢复后执行 `doctor all` |
| 输入 | artifact budgets、QUALITY-MODEL、read-expansion ledger、Story/Run ledgers、历史 CP result |
| 处理逻辑 | 修复 active budget 与 policy 路径错误；历史证据通过 append-only correction、compact summary、index/hash 或 archive 处理；warning 单列，不把可接受 legacy 状态误报成 blocker |
| 输出/结果 | Doctor 无阻断错误，warning 数量与原因可审计，历史机器结果不被静默改写 |
| 前置条件 | budget / retention / quality policy 可读 |
| 排除情况 | 不截断或原位美化历史 CP result；不伪造缺失授权或 run 时间 |

### UC-WT-004：让 clean-clone guardrail 不依赖 ignored 本机文件

| 字段 | 内容 |
|---|---|
| 使用角色 | P-02 Agent / Skill 作者、P-04 审批者 / Reviewer |
| 触发条件 | clean tree、CI 或发布前运行 delivery guardrail |
| 输入 | tracked canonical rules、generated root wrapper 契约、Git tracked files、cache policy |
| 处理逻辑 | 按 CP2 选择统一规则 source；guardrail 在 clean clone 可独立发现 canonical source，并区分 tracked cache 与 ignored 本机 cache |
| 输出/结果 | `git archive HEAD` 等价环境 guardrail 退出码 0；个人配置不污染共享规则 |
| 前置条件 | canonical-source 决策已批准 |
| 排除情况 | 不把本机个人 Memory Policy 提交为共享规范；不以 ignored 文件作为唯一必需输入 |

### UC-WT-005：把 Ruff 与回归测试纳入确定性质量门

| 字段 | 内容 |
|---|---|
| 使用角色 | P-02 Agent / Skill 作者、P-04 审批者 / Reviewer |
| 触发条件 | 代码变更、发布前检查或 CI |
| 输入 | Python 源码、Ruff 配置、既有 pytest / subtest suite |
| 处理逻辑 | 安全自动修复可机械处理项，人工审查 B/F 语义项；随后运行 Ruff 和完整回归，并把命令纳入发布前组合门 |
| 输出/结果 | Ruff error=0；不少于 377 tests + 70 subtests 通过；失败会阻断发布就绪 |
| 前置条件 | 使用 `uv run --python 3.11` |
| 排除情况 | 不用测试通过替代 lint；不在产品阶段规定具体代码 diff |

### UC-WT-006：为 Agent/CI 提供完整的非交互安装与 cache preflight

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、自动化调用方 |
| 触发条件 | CI、Agent 或非 TTY shell 按 README 执行 project-scope 安装 dry-run |
| 输入 | 平台、component、scope、显式 `--project-dir`、cache policy |
| 处理逻辑 | README 为 Codex/Claude/Qoder 提供等价非交互命令；preflight 对 tracked cache 阻断，对 ignored cache 按批准策略清理或 warning |
| 输出/结果 | 3/3 平台 dry-run 退出码 0；测试后不会因 ignored cache 形成不可解释的永久红灯 |
| 前置条件 | 安装器主体可用，目标目录可读 |
| 排除情况 | 不执行真实用户级安装、外部发布或凭据操作 |

### UC-WT-007：收敛 CR-046 产品状态但保留恢复时序与风险

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-04 审批者 / Reviewer |
| 触发条件 | 产品矩阵、Feature matrix、CR index、CP7 与 release 状态出现陈旧或冲突 |
| 输入 | CR-046 formal CR/index、7 个 Story/LLD/CP6/CP7、产品矩阵与 `READY_WITH_RISK` 风险 |
| 处理逻辑 | 将 current automation 和 closure 状态回写为已实现/已验证带风险，保留 recovered/post-hoc 语义和原始 hash；平台 receipt、独立 QA、token telemetry 与真实 pilot 继续明确 unavailable/未授权 |
| 输出/结果 | 7/7 Story 状态可追踪，产品矩阵不再写 CP2 pending，发布结论仍为 `READY_WITH_RISK` |
| 前置条件 | 正式 CR-046 与 CP7 result 可读 |
| 排除情况 | 不倒填事前批准，不把 fixture rejection 写成真实平台 receipt，不重新打开历史时序 |

## 附录：覆盖自检表

| 维度 ID | 维度名称 | 状态 | 涉及场景 | 备注 |
|---|---|---|---|---|
| D1 | 用户维度 | 已补充 | UC-PG-001..007, UC-EI-001..005 | 新增审计者、证据生产者与成本分析者 |
| D2 | 任务维度 | 已补充 | UC-PG-001..007, UC-EI-001..005 | 补充时序、调度证明、重放、telemetry 与 append-only 迁移 |
| D3 | 动机维度 | 已补充 | UC-EI-001..005 | 可信审计、可复现重放、诚实成本和历史不可变 |
| D4 | 时间维度 | 已补充 | UC-EI-001, UC-EI-002, UC-EI-003, UC-EI-005 | 覆盖条件式批准、retry/supersession、as-executed/current-replay 与历史修正 |
| D5 | 环境维度 | 已补充 | UC-EI-002, UC-EI-005 | 区分平台 session、仓库证据与 quant-lab pilot |
| D6 | 方式维度 | 已补充 | UC-EI-001..005 | checker、ledger、receipt、hash、telemetry 与 migration manifest |
| D7 | 异常维度 | 已补充 | UC-EI-001..005 | 非法时序、缺 receipt/telemetry、retry、replay drift、历史不可变性 |
| D8 | 集成维度 | 已补充 | UC-EI-001..005 | 与 CP result、gate/state/dispatch/read ledger、checker 和 CR-163 衔接 |
| D1-WT | 用户维度 | 已补充 | UC-WT-001..007 | 覆盖维护者、审计者、迁移执行者、Reviewer 与非交互调用方 |
| D2-WT | 任务维度 | 已补充 | UC-WT-001..007 | 覆盖 clone/link、状态发现、CR tracking、Doctor、guardrail、lint、安装和历史收敛 |
| D3-WT | 动机维度 | 已补充 | UC-WT-001..007 | 单一 workflow truth、确定性质量门与诚实风险披露 |
| D4-WT | 时间维度 | 已补充 | UC-WT-001, UC-WT-003, UC-WT-007 | 覆盖首次 clone、发布前、closed CR 事后收敛与回退 |
| D5-WT | 环境维度 | 已补充 | UC-WT-002, UC-WT-004, UC-WT-006 | 覆盖不同设备、clean clone、CI/Agent 非 TTY 与本机 cache |
| D6-WT | 方式维度 | 已补充 | UC-WT-001..007 | 通过 JSON truth、symlink metadata、Doctor、Ruff、guardrail、dry-run 和 append-only correction 验证 |
| D7-WT | 异常维度 | 已补充 | UC-WT-001..007 | 覆盖 closed active CR、legacy index、断链、超预算、missing rule、lint error、缺 project-dir 和 receipt unavailable |
| D8-WT | 集成维度 | 已补充 | UC-WT-001..007 | 与 state/current/CR index、artifact repo、quality policy、README、CI 与 CR-046 evidence 链衔接 |
