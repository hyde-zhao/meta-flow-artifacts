---
cr_id: "CR-030"
cr_type: "product-scope"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "READY_WITH_RISK"
gate_status: "cp8-approved"
gate_profile: "architecture-major"
route_plan_ref: "process/checks/CP0-CR030.route-plan.json"
cr_trait_uses_existing_evidence_only: false
cr_trait_has_new_design: true
cr_trait_has_new_implementation: true
cr_trait_has_new_verification: true
cr_trait_requires_architecture_review: true
cr_trait_requires_story_decomposition: true
cr_trait_requires_subagent_dispatch: true
cr_trait_verification_waiver_reason: ""
cr_trait_verification_waiver_ref: ""
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
rollback_to: "requirement-clarification"
approval_result: "approved"
created_at: "2026-07-15T07:00:27+00:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-07-15T09:50:00+00:00"
source: "user"
goal_statement: "为 ptm-tse 增加仅面向恢复后现网问题的逆向分析能力，以可审计的 RCA、引入点/流出点分析、CA/PA 和有效性验证驱动跨 Agent 的持续改进。"
user_goal_impact: "让测试架构师能受控获取、保存和分析现网问题单，向测试经理、测试设计负责人和质量负责人提供可验证的总结与改进措施，同时不代替人工做生产修复决策。"
approval_focus: "scope|architecture|security|runtime_authorization|implementation|risk"
decision_burden: "high"
approve_effect: "CP2 批准后将进入 HLD/蓝图设计；CP3、CP5 批准后才会实施 ptm-tse Agent、专属 Skills、模板和集成契约。"
reject_effect: "阻塞 ptm-tse 逆向分析能力的设计与实施；现有 Agent 基线和其他已交付 Agent 不受影响。"
not_authorized_by_approve:
  - "读取凭据、密钥、账号或会话"
  - "访问、修改或执行现网设备、配置、策略、工单或外部系统；CP2 批准前也不读取用户指定 ITR 接口"
  - "自动执行缓解、修复、发布、TAC 开单或生产写入"
  - "内部问题逆向分析（当前仅支持现网问题恢复后分析）"
product_baseline_refresh_required: true
required_phase: "requirement-clarification"
required_agent: "meta-pm"
required_gate: "CP2"
block_story_decomposition_until: "CP2-approved"
affected_product_docs:
  - "docs/product/USE-CASES.md"
  - "docs/product/REQUIREMENTS.md"
  - "docs/product/SCENARIOS.yaml"
  - "docs/product/TEST-MATRIX.md"
  - "docs/product/STORY-MAP.md"
  - "docs/product/MVP-SCOPE.md"
  - "docs/design/BLUEPRINT.md"
  - "docs/design/HLD.md"
  - "agents/ptm-tse.md"
affected_use_cases: []
conflict_keys:
  - "ptm-tse"
  - "reverse-analysis"
  - "continuous-improvement"
  - "agent-skill-contract"
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_PRODUCTION_WRITE"
  - "NO_EXTERNAL_PUBLISH"
risk_refs:
  - "RISK-CR030-AUTONOMOUS-REMEDIATION"
  - "RISK-CR030-UNSUPPORTED-INTERNAL-INCIDENT"
---

# CR-030 — ptm-tse 现网问题逆向分析能力

## 变更描述

基于用户提供的《逆向分析实施文档》与《融合 TAC、SRE、RCA、SOC、DMAIC 与缺陷预防的防火墙测试加 AI Agent 持续改进方法论》，为尚处于规划阶段的 `ptm-tse`（测试架构师 / 技术 Owner）新增逆向分析能力。

本能力的触发点是**现网问题已经恢复或完成安全缓解之后**。它把一次问题反向展开为：根因分析、问题引入点、流出控制失效、永久纠正措施（CA）、预防措施（PA）、行动项、有效性验证、复发监控与跨 Agent 改进输入。RCA 是其中一个子步骤，不能代替完整闭环。

## 目标影响摘要

| 字段 | 内容 |
|---|---|
| 整体目标 | 将现网问题的经验沉淀为可追溯、可验证、可分派的持续改进输入。 |
| 用户目标影响 | 降低同类问题再次流入现网的概率，并让测试策略、用例、执行反馈和工具改进有共同证据链。 |
| 为什么独立推进 | 新能力横跨 `ptm-tm`、`ptm-tde`、`ptm-te`、`ptm-tae` 的消费契约，涉及新 Agent/Skill、数据对象、门禁和多 Story 实施，不能作为单一文档或现有 Agent 的局部任务处理。 |
| approve 后会发生什么 | 仅按当前门禁推进到下一阶段；不会授权真实运行、生产修改或外部写入。 |
| 不确认会阻塞什么 | 阻塞设计、Story 拆分与实现；现有六 Agent 基线保持不变。 |

## 范围与边界

### In scope

- 仅支持恢复后现网问题的逆向分析；输入为经人工脱敏、授权提供的问题事实和证据索引。
- 在 CP2 新增授权通过后，受控读取用户指定的 ITR 只读问题单接口；保存可追溯的问题单快照/规范化记录，支持按单和批量分析、总结与人工确认的改进措施输出。
- 三段式工作流：问题分析（RCA、引入点、流出点）→ 改进分析（CA、PA、横向展开）→ 闭环跟踪（Owner、期限、有效性、复发监控、关闭条件）。
- `ptm-tse` 负责生成分析和改进建议，并将经确认的测试/执行/自动化/质量改进输入交给相应下游 Agent。
- 新增 `reverse-analysis-skill`、`improvement-tracker-skill` 的候选能力边界、模板和输入/输出契约。
- 与现有逆向分析参考方法对应的、可静态验证的 Agent/Skill/文档产物。

### Out of scope / 不授权范围

- 自动检测、事件宣告、实时事故指挥、恢复或生产缓解动作。
- 除用户指定 ITR 只读问题单接口外，直接读取日志平台、设备、抓包、其他工单系统、TAC、知识库或任何外部系统。
- 在 CP2 批准前调用 ITR 接口；读取凭据、推断认证头，或向 ITR 及任何外部系统写入。
- 读取凭据、秘密、会话、账号，或执行设备/策略/配置/工单/发布/外部写入。
- 自动判定根因为事实、自动关闭行动项，或越过人工确认执行 CA/PA。
- 内部问题逆向分析；该能力保留为后续候选项，需独立 CR 重新评估。

## 参考基线与文档处理决策

| 对象 | 处理方式 | 旧基线保留方式 | 修订记录 / 证据 | 当前状态 |
|---|---|---|---|---|
| `agents/ptm-tse.md` | 原文档更新 | 保留原测试架构师职责和评审链；追加逆向分析职责/契约 | `## 修订记录` | pending |
| `docs/ptm-tse/README.md` | 原文档更新 | 保留 Step 1-4 演进基线；追加新能力说明与链接 | `## 修订记录` | pending |
| `docs/ptm-team-blueprint.md` | 原文档更新（用户已开始草案） | 仅增量吸收经 CP2/CP3 确认的内容；不覆盖用户现有改动 | `## 修订记录` | pending |
| `skills/reverse-analysis-skill/` | 新增 | N/A | Skill 内修订记录 | pending |
| `skills/improvement-tracker-skill/` | 新增 | N/A | Skill 内修订记录 | pending |
| `docs/product/*` | 新增 / 原文档更新（待基线确认） | 不删除既有 ptm-tde 基线；通过 CR-030 追加并建立映射 | `## 修订记录` | pending |
| 外部参考目录 | 只读参考 | 不复制原文，不写回 | 来源清单与引用摘要 | confirmed |

**来源目录（只读）**：`/home/hyde/projects/llm-wiki/llm-wiki/work/studies/ptm-team/sources/ptm-tse/`。

## 五维度影响分析

| 维度 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|
| 需求 / 场景 | 现网恢复后的分析、纠正、预防与闭环场景 | 新增 | 由 `meta-pm` 产出增量 USE-CASES、REQUIREMENTS、SCENARIOS、TEST-MATRIX 与范围基线。 |
| 设计 | `ptm-tse`、两个候选 Skill、跨 Agent 改进输入、数据归属与人机门禁 | 新增架构契约 | 由 `meta-se` 进行蓝图/HLD/ADR 设计，明确调用方向、时机、输入输出和降级。 |
| Story / 实现 | Agent 提示词、Skills、模板、文档、可能的校验脚本 | 多 Story | CP3 后拆 Story，CP5 前由 `meta-dev` 形成 LLD 或技术说明。 |
| 安全 / 权限 | ITR 问题单可能含敏感内容；新增 HTTP 外部读取和本地保存 | 高风险边界变更 | REV-02 仅提议固定来源、GET-only、无凭据、无写入的受控读取；数据分类、存储位置、保留与脱敏须经 CP2 决策。 |
| 交付 / 验证 | Agent/Skill 发现、文档、契约和静态评估 | 新增 | `meta-qa` 制定静态/fixture/dry-run 验证，验证覆盖和禁止能力声明。 |

## 初始风险与回退

| 风险 | 等级 | 缓解 | 回退 / 切换条件 |
|---|---|---|---|
| Agent 把建议描述成可自动执行的生产修复 | High | Agent/Skill 合同强制建议模式、人工门禁、禁用 runtime 工具 | 回退到分析报告草案；另建 runtime 授权 CR。 |
| 事件证据包含敏感信息 | High | 只接收脱敏摘要/索引，禁止凭据和外部读取 | 停止该案例处理，要求提供合规脱敏输入。 |
| RCA 与“逆向分析”职责重叠或遗漏 | Medium | 在 HLD 定义三段式边界和数据对象，要求可证伪证据 | 回退到 CP3，补充职责矩阵/ADR。 |
| 现有 `ptm-tm/tde/te/tae` 契约不清 | Medium | CP2/CP3 显式定义上游来源、下游消费、拒绝与降级路径 | 将无法收敛部分转 Spike/后续 CR。 |
| ITR 接口为 HTTP 且响应字段/数据分类未知 | High | 固定 allowlist、GET-only、无凭据、响应 schema 校验、最小化保存、人工审查 | CP2 未批准或 schema/数据分类不满足时退回人工提供的脱敏摘要。 |

## 路由与检查点索引

| CP | 状态 | 机器结果 ref | 人工门禁 ref | Context ref | 说明 |
|---|---|---|---|---|---|
| CP0 | pass | `process/checks/CP0-CR030.result.json` | N/A | `process/context/CP0-CR030.context.json` | 受理、路由与冲突预检已通过。 |
| CP1 | pending | `process/checks/CP1-CR030-REV02.result.json` | N/A | `process/context/CP2-REQUIREMENT-CONTEXT.yaml` | REV-02 改变了数据来源、保存和用户旅程，须重跑。 |
| CP2 | approved | `process/checks/CP2-CR030-REV02.result.json` | `process/checkpoints/CP2-CR030-REQUIREMENTS-BASELINE.md` | `process/context/CP2-REQUIREMENT-CONTEXT.yaml` | 用户批准 ITR 读取、数据保存、分析输出和安全边界；schema 探测 HTTP 200。 |
| CP3 | changes_requested | N/A | `process/checkpoints/CP3-CR030-HLD-REVIEW.md` | N/A | 等待 REV-02 CP2 后刷新蓝图/HLD。 |
| CP4 | pending | `process/checks/CP4-CR030.result.json` | N/A | Story planning context | 自动 Story/DAG 预检。 |
| CP5 | pending | `process/checks/CP5-CR030.result.json` | `process/checkpoints/CP5-CR030-ALL-STORIES-LLD.md` | `process/context/CP5-LLD-CONTEXT.yaml` | 全量 Story 设计证据确认。 |
| CP6 | pending | Story CP6 results | N/A | Story packets | 实现完成。 |
| CP7 | pending | Story CP7 results | N/A | Story packets | 静态/fixture/dry-run 验证。 |
| CP8 | pending | `process/checks/CP8-CR030.result.json` | `process/checkpoints/CP8-CR030-DELIVERY-READINESS.md` | `process/context/CP8-DELIVERY-CONTEXT.yaml` | 交付就绪。 |

## 进入条件

- [x] 用户明确提出新增能力及参考资料目录。
- [x] 现有 `ptm-tse` 基线和项目蓝图已识别。
- [x] 过程目录路由健康，且状态已迁移到 state v2。
- [x] 不授权范围已声明。
- [x] CP0 route plan、冲突检查和受理结果已固化。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.0 | 2026-07-15 | host-orchestrator | 创建 CR-030：将 ptm-tse 逆向分析作为产品范围/架构/多 Story 变更立项，明确只支持恢复后现网问题与禁止能力边界。 |
| v1.1 | 2026-07-15 | host-orchestrator | 根据用户反馈启动 REV-01：回退至 requirement-clarification，将流程中心的场景改写为用户中心场景；CP2/CP3 重新确认。 |
| v1.2 | 2026-07-15 | host-orchestrator | 根据用户新增的 ITR 问题单读取、保存、分析总结与改进措施要求启动 REV-02；外部读取和持久化边界待新的 CP2 决策。 |
| v1.3 | 2026-07-16 | host-orchestrator | CP8 已批准：CR-030 以 `READY_WITH_RISK` 交付；静态交付不构成真实运行、凭据访问、外部写入或自动 reviewer 动作授权。 |

## REV-01 用户中心场景修订

- 影响分析：`process/changes/CR-030-USER-CENTERED-SCENARIO-IMPACT-2026-07-15.md`
- 回退阶段：`requirement-clarification`
- 已失效门禁：CP1、CP2、CP3；旧结果仅保留审计，不得作为当前设计依据。

## REV-02 ITR 问题单摄取、保存与分析修订

- 影响分析：`process/changes/CR-030-EXTERNAL-ITR-INGESTION-AND-ANALYSIS-IMPACT-2026-07-15.md`
- 回退阶段：`requirement-clarification`
- 待重新确认的边界：用户指定 ITR 接口的 GET-only 读取、问题单保存策略、响应 schema、脱敏/保留、分析总结与改进措施的人机确认链。
- 旧的“完全不读取外部系统”范围只保留为 REV-01 历史基线；本 CR 不因此获得其他系统访问、凭据读取或任何写入权限。
