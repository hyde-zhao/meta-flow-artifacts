---
status: draft
version: "1.0"
feature_id: "FEAT-PG-009"
feature_name: "Quant-lab Migration Readiness"
source_blueprint: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
source_adr: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
related_stories:
  - "CR-H-S01 quant-lab migration dry-run and reports"
lld_policy_summary: "full-lld required for migration dry-run, release repository read-only boundary, process-side state cleanup, rollback, and security evidence."
confirmed_by: ""
confirmed_at: ""
---

# Feature Design: Quant-lab Migration Readiness

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 定义 quant-lab 作为真实样本迁移的只读发布库边界、dry-run、过程侧状态清理、报告与回退策略 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature 目标 | 在 P0/P1 治理能力完成后，用 quant-lab 作为真实样本验证 state cleanup、capability registry、impact migration 和 stale report。 |
| 推荐方案 | 对 quant-lab 发布库只读；迁移命令默认 dry-run；只清理过程侧状态和 artifact；不授权修改 quant-lab 代码、tests、正式 docs 或发布文档。 |
| 关键取舍 | 放弃一次性自动修复发布库，换取授权边界清楚、迁移可预演、回滚可审计。 |
| 下游 Story | CR-H-S01 |
| LLD 策略 | full-lld，因为涉及真实样本迁移、安全边界、dry-run、process-side write、rollback 和报告证据。 |

## 背景与问题

| 问题 ID | 背景 | 触发场景 | 影响 | 若不设计的风险 |
|---|---|---|---|---|
| P-QL-01 | HLD 将 quant-lab migration 作为 P2 真实样本验证，但明确不自动修改发布库。 | P0/P1 能力完成后，对 quant-lab 过程态执行迁移准备。 | 需要 dry-run、read-only release boundary 和 migration report。 | 样本迁移越权写代码/tests/docs，造成审计和回滚风险。 |
| P-QL-02 | quant-lab 可能存在 current state 膨胀、capability refs 和 impact_surface 漂移。 | 迁移检查命中历史状态或 CR impact。 | 需要过程侧状态清理和 registry-backed migration。 | 样本问题绕过通用机制，污染机制边界。 |
| P-QL-03 | 发布库正式文档陈旧可能需要后续修复。 | stale-check 发现 TEST-STRATEGY / release docs 与 roadmap 不一致。 | 需要输出 stale report 和 FU-RF/正式 CR 候选。 | 迁移阶段直接改正式 docs，绕过 CP / CR。 |

## 上游依据与输入

| 来源 | 路径 / ID | 被本设计消费的内容 |
|---|---|---|
| Blueprint | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-BLUEPRINT.md` | FEAT-PG-009 负责 quant-lab process-side migration reports/project state objects；禁止改发布库正式代码或文档。 |
| Domain Map | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DOMAIN-MAP.md` | OBJ-PG-016 Quant-lab Migration Report、SM-PG-005、RULE-PG-012。 |
| Dependency Map | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-DEPENDENCY-MAP.md` | FEAT-PG-009 依赖 FEAT-PG-001/004/005/006/008；FD-PG-008 禁止与机制实现 CR 混交。 |
| HLD | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md` | UC-PG-007、SIM-PG-003、HLD §18 quant-lab dry-run 输入/输出/回滚与授权边界。 |
| ADR | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md` | ADR-PG-003 发布库只输出 stale/follow-up；ADR-PG-004 registry refs。 |
| Feature Matrix | `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` | FEAT-PG-009 required；CR-H-S01 full-lld。 |

## 目标与非目标

| 类型 | 内容 | 来源 |
|---|---|---|
| Goal | 设计 quant-lab migration dry-run：读取发布库必要摘要/refs，生成 process-side migration report，不写发布库。 | HLD §17 / RULE-PG-012 |
| Goal | 过程侧清理 current state、project state refs、capability refs、impact migration report 和 stale report。 | UC-PG-007 |
| Goal | 使用 FEAT-PG-004 registry 和 FEAT-PG-005 impact migration，不允许迁移凭空创造 capability ID。 | ADR-PG-004 / FD-PG-004 |
| Goal | 输出 stale report 和 FU-RF/正式 CR 候选，用于后续授权修复。 | ADR-PG-003 / FEAT-PG-007/008 |
| Non-Goal | 不修改 quant-lab 发布库代码、tests、正式 docs、质量文档或发布文档。 | RULE-PG-012 |
| Non-Goal | 不读取或写入凭据、账户、token、cookie、私钥。 | CR-037 authz policy |
| Non-Goal | 不把 quant-lab 样本特例反向写入通用机制设计。 | FD-PG-008 |
| Non-Goal | 本轮 feature design 不读取或修改 `process/quant-lab/**`。 | 用户约束 |

## Feature 边界与相邻对象

| 对象 | 本 Feature 负责 | 不负责 | 相邻 Feature / 模块 | 边界判定依据 |
|---|---|---|---|---|
| Migration Dry-run | 预演 process-side 迁移、输出 planned changes 和 blocked findings。 | 实际修改发布库。 | FEAT-PG-001/004/005/006/008 | SM-PG-005 |
| Process-side State Cleanup | 清理过程侧 current/project state refs 和 migration evidence。 | 发布库 code/tests/docs cleanup。 | FEAT-PG-001/003 | RULE-PG-012 |
| Capability / Impact Migration | 使用 registry resolver 和 impact migration report。 | 自动注册自由字符串 capability。 | FEAT-PG-004/005 | ADR-PG-004 |
| Stale Report | 只读发布库后输出 stale findings/FU-RF 候选。 | 自动修复 stale docs。 | FEAT-PG-006/007/008 | ADR-PG-003 |

## 现有代码位置

| 区域 | 路径 | 当前职责 | 变更方式 |
|---|---|---|---|
| Migration CLI | 后续 LLD 定位 meta-flow migration command。 | 执行 dry-run / report。 | create/modify in meta-flow only。 |
| Process artifact state | process-side state/project/changes/report 路径。 | 保存迁移报告和过程侧状态。 | write process-side only。 |
| quant-lab release repo | 目标项目代码、tests、正式 docs、release docs。 | 真实样本只读来源。 | read-only; no write。 |
| Registry / impact checker | FEAT-PG-004/005 产物。 | 解析 capability / impact migration。 | read / call only。 |

## 现状分析

| 维度 | 当前状态 | 缺口 | 约束 |
|---|---|---|---|
| 数据 | Domain Map 定义 Quant-lab Migration Report。 | 缺少 report 字段、dry-run diff 表达和 blocked finding。 | 不复制发布库长正文。 |
| 接口 | HLD 确认 P2 在 W1-W4 后执行。 | 缺少 migration command dry-run/apply 边界。 | 当前不授权实现或发布库写。 |
| 测试 | Feature Matrix 要求 full-lld。 | 缺少 read-only guard、dry-run、process-side rollback 测试。 | 不读取/修改 `process/quant-lab/**`。 |
| 运维 / 发布 | quant-lab 发布库正式变更需要独立授权。 | 需要把发布库修复转 stale/FU-RF/CR。 | NO_PRODUCTION_WRITE / NO_TRADING。 |

## 推荐方案

| 设计点 | 推荐做法 | 理由 | 代价 |
|---|---|---|---|
| 默认模式 | migration command 默认 `--dry-run`，显式 apply 也只作用于 process-side artifacts。 | 先证明迁移计划，不做不可逆写入。 | 需要 report 和 fixture。 |
| 发布库边界 | release repo read-only；任何 code/tests/formal docs change 都转 stale/FU-RF/正式 CR。 | 满足用户授权和 ADR-PG-003。 | 修复链路更长。 |
| Process-side cleanup | 清理 current state refs、project state refs、impact migration report、capability refs、stale report。 | 验证 P0/P1 机制在真实样本可用。 | 依赖前置 Feature 完成。 |
| Registry usage | capability/feature refs 必须解析 registry；缺失输出 blocked finding。 | 防止样本特例污染 registry。 | 需要先补 registry 或 CR。 |
| 回退 | process-side apply 记录 before/after hash 和 rollback refs。 | 可审计恢复。 | 需要保存 rollback metadata。 |

## 方案对比与决策记录

| Decision ID | 方案 | Pros | Cons | Impact Surface | Recommendation | When to switch |
|---|---|---|---|---|---|---|
| DQ-QL-001 | Option A：发布库只读 + process-side dry-run/apply | 授权边界清楚；回滚范围可控；样本验证不越权。 | 不能一次性修复发布库正式问题。 | migration、security、project-governance | 推荐 | 若要改发布库，另开 CR + human gate。 |
| DQ-QL-001 | Option B：迁移时自动改发布库 | 修复快。 | 未授权生产写；跨仓回滚风险高。 | release、quality、security | 备选，不采用 | 仅在独立发布库写授权和回滚设计批准后切换。 |
| DQ-QL-002 | Option A：先 dry-run 后 process-side apply | 可审计、可人工确认。 | 多一步人工检查。 | migration、quality | 推荐 | 对低风险 fixture 可保持 dry-run only。 |
| DQ-QL-002 | Option B：直接 process-side apply | 快。 | 容易把样本误判落盘。 | process-state | 备选 | 仅用于已验证 fixture，不用于真实样本默认路径。 |

## 模块变更

| Module | 变更 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| quant-lab migration command | 执行 dry-run、生成 planned changes、可选 process-side apply。 | process refs、release repo readonly refs、registry refs | migration report、stale report、rollback refs | release write requested -> FAIL。 |
| release repo read-only adapter | 读取必要摘要和 refs。 | release repo path / refs | observed summary | credential request / write action -> FAIL。 |
| process-side cleanup writer | 写入过程侧 state/project/migration evidence。 | planned process changes | updated process artifacts | target outside process -> reject。 |
| registry migration adapter | 调用 capability/feature resolver。 | capability strings / refs | resolved refs or blocked findings | unresolved -> blocked finding，不创造 ID。 |
| impact migration adapter | 调用 normalized impact migration。 | legacy impact records | migration report | unknown surface -> report-only / blocked by mode。 |
| stale report adapter | 调用 stale-check / follow-up contract。 | observed release refs | stale findings / FU-RF candidates | tracking unavailable -> keep stale report。 |

## 数据模型与状态

| Object | Owner | 新增 / 修改字段 | 状态变化 | 兼容性 |
|---|---|---|---|---|
| Quant-lab Migration Report | FEAT-PG-009 | `schema_version`、`run_id`、`mode=dry_run/process_apply`、`source_refs`、`planned_process_changes`、`release_repo_readonly_refs`、`blocked_findings`、`stale_report_ref`、`rollback_refs`。 | planned -> dry-run -> process-side-updated -> stale-reported -> complete / blocked | 不改变发布库。 |
| Process-side Change | FEAT-PG-009 | `target_ref`、`operation`、`before_hash`、`after_hash`、`rollback_ref`。 | planned / applied / rolled_back | target 必须在 process-side boundary。 |
| Release Readonly Observation | FEAT-PG-009 | `object_ref`、`observed_summary`、`hash`、`stale_rule_refs`。 | observed | 不保存长正文，不写回。 |
| Blocked Finding | FEAT-PG-009 | `finding_id`、`reason`、`required_action`、`owner`、`source_ref`。 | open / resolved / converted_to_cr | 可转 FU-RF 或正式 CR。 |

## API / 接口设计

| Interface ID | 调用方 | 被调用方 | 输入契约 | 输出契约 | 错误模型 |
|---|---|---|---|---|---|
| IF-QL-01 | host-orchestrator / CLI | migration dry-run | project/process refs、release_repo_ref readonly、registry refs、mode=dry-run | migration report path、blocked findings | release write request -> FAIL。 |
| IF-QL-02 | migration command | process-side writer | planned_process_changes with process targets | applied refs / rollback refs | target outside process -> reject。 |
| IF-QL-03 | migration command | registry resolver | candidate capability/feature refs | resolved refs / unresolved findings | unresolved -> blocked finding。 |
| IF-QL-04 | migration command | impact migration | legacy impact records | normalized report | unknown surface -> report finding。 |
| IF-QL-05 | migration command | stale-check / FU-RF tracking | release readonly observations | stale report / follow-up candidates | tracking unavailable -> stale report only。 |

## 关键流程

| Flow ID | 触发条件 | 主流程 | 异常流程 | 输出 / 状态变化 | 观测点 |
|---|---|---|---|---|---|
| FLOW-QL-01 | W1-W4 能力完成后，准备 quant-lab 迁移。 | 收集 process refs -> 只读 release refs -> registry/impact dry-run -> stale-check -> 生成 migration report。 | 发现 release write need -> blocked finding / FU-RF，不写 release repo。 | dry-run report。 | blocked_count、planned_process_change_count。 |
| FLOW-QL-02 | dry-run 人工确认后执行 process-side apply。 | 校验 target boundary -> 写 process-side artifacts -> 记录 before/after hash 和 rollback refs。 | target outside process -> reject。 | process-side-updated report。 | rollback_refs、applied_count。 |
| FLOW-QL-03 | capability unresolved。 | 输出 blocked finding，不自动注册 capability。 | 需要补 registry 或正式 CR。 | migration blocked / partial。 | unresolved_capability_count。 |
| FLOW-QL-04 | stale formal docs detected。 | 生成 stale report 和 FU-RF/CR candidate。 | 不修改正式 docs。 | stale-reported。 | stale_count、candidate_count。 |

## 人机协作与确认点

| 确认点 | 触发条件 | 需要谁确认 | 推荐方案 | 备选方案 | 不授权项 |
|---|---|---|---|---|---|
| DQ-QL-001 | dry-run 结果准备 apply 到 process-side。 | user / host-orchestrator | 仅 apply process-side changes。 | 保持 dry-run only。 | 不授权发布库写。 |
| DQ-QL-002 | migration 发现发布库正式文档需改。 | user / PO | 生成 FU-RF 或正式 CR。 | waive with reason。 | 不在 migration 内改 docs/tests/code。 |
| DQ-QL-003 | unresolved capability 出现。 | meta-se / user | 补 registry 或创建 blocked follow-up。 | 临时跳过该 capability。 | 不自动创造 capability ID。 |

## 异常、失败与降级策略

| Failure ID | 失败条件 | 系统行为 | 用户可见影响 | 恢复 / 回退 | 测试入口 |
|---|---|---|---|---|---|
| F-QL-01 | migration 尝试写 quant-lab release repo code/tests/docs。 | FAIL security guard。 | 迁移阻断。 | 转 FU-RF/正式 CR。 | TEST-PLAN SEC-QL-01 |
| F-QL-02 | dry-run 发现 process-side target 越界。 | blocked finding，不 apply。 | 需要修正 target mapping。 | 修正 mapping 后重跑 dry-run。 | TEST-PLAN SEC-QL-02 |
| F-QL-03 | capability unresolved。 | blocked finding，不创造 ID。 | migration partial/blocked。 | 补 registry 后重跑。 | TEST-PLAN INTEG-QL-03 |
| F-QL-04 | process-side apply 后发现错误。 | 使用 rollback_refs 恢复 process-side artifacts。 | 发布库不受影响。 | rollback process-side only。 | TEST-PLAN INTEG-QL-04 |
| F-QL-05 | stale-check 或 FU-RF tracking 不可用。 | 保留 stale report，不丢失 evidence。 | follow-up 自动化延后。 | 待 FEAT-PG-007/008 可用后转换。 | TEST-PLAN INTEG-QL-05 |

## 权限与安全

| Rule ID | 规则 | 触发条件 | 失败行为 | 测试入口 |
|---|---|---|---|---|
| SEC-QL-01 | quant-lab 发布库只读，不修改代码/tests/正式 docs/发布文档。 | planned change target 指向 release repo。 | FAIL / blocked finding。 | TEST-PLAN SEC-QL-01 |
| SEC-QL-02 | migration 默认 dry-run，不做 process-side apply，除非后续 human gate 明确授权。 | command without apply authorization。 | only report。 | TEST-PLAN UNIT-QL-01 |
| SEC-QL-03 | process-side cleanup 只能写过程归档库状态和迁移证据。 | target outside process boundary。 | reject。 | TEST-PLAN SEC-QL-02 |
| SEC-QL-04 | 不读取凭据、账户、token、cookie、私钥。 | adapter 请求 credential path/env。 | FAIL。 | TEST-PLAN SEC-QL-03 |
| SEC-QL-05 | 不授权 live、publish、production write、trading。 | runtime/publish flag requested。 | FAIL。 | TEST-PLAN SEC-QL-04 |

## 测试与验收策略

| 验收对象 | 测试层级 | 覆盖场景 | 自动化方式 | 未自动化原因 / 手工入口 |
|---|---|---|---|---|
| Dry-run report | unit / contract | planned changes、blocked findings、readonly observations。 | fixture validation。 | 无。 |
| Release read-only guard | security / integration | release target write blocked。 | temp fixture with forbidden targets。 | 不对真实发布库写入。 |
| Process-side cleanup | integration | process target apply + rollback refs。 | temp process fixture。 | 真实样本 apply 需人工确认。 |
| Registry / impact integration | integration | unresolved capability、unknown impact surface。 | registry / impact fixtures。 | 无。 |
| Stale report / FU-RF | integration / manual | stale formal docs -> FU-RF candidate。 | fixture + manual review。 | 语义陈旧需人工抽样。 |

## 实现顺序

| Step | 内容 | 前置条件 | 输出 | 验证入口 |
|---|---|---|---|---|
| 1 | 定义 dry-run report schema 和 release readonly observation。 | FEAT-PG-009 CP5 设计。 | report schema。 | TEST-PLAN CONTRACT-QL-01。 |
| 2 | 定义 release repo read-only adapter 和 forbidden write guard。 | Step 1。 | security guard。 | TEST-PLAN SEC-QL-01。 |
| 3 | 定义 process-side cleanup planned changes 和 rollback refs。 | FEAT-PG-001/003 可用。 | process-side apply contract。 | TEST-PLAN INTEG-QL-04。 |
| 4 | 定义 registry / impact migration adapters。 | FEAT-PG-004/005 可用。 | integration contract。 | TEST-PLAN INTEG-QL-03。 |
| 5 | 定义 stale report 和 FU-RF/CR candidate 输出。 | FEAT-PG-006/007/008 可用。 | stale migration evidence。 | TEST-PLAN INTEG-QL-05。 |

## Story 拆分建议与 LLD 策略

| Story ID | feature_design_refs | lld_policy.required_level | 触发原因 | 必须进一步设计的问题 | 可用设计证据 |
|---|---|---|---|---|---|
| CR-H-S01 quant-lab migration dry-run and reports | `process/docs/features/quant-lab-migration-readiness/DESIGN.md`、`TEST-PLAN.md`、`TASKS.md` | full-lld | migration / security / external-project-boundary | dry-run schema、read-only adapter、process-side cleanup、rollback、registry/impact/stale integration。 | Feature DESIGN / TEST-PLAN / TASKS |

## 下游消费契约

| 消费方 | 消费时机 | 输入契约 | 输出 / 状态要求 | 降级策略 |
|---|---|---|---|---|
| story-manager | CP4 前 | CR-H-S01 full-lld policy and refs。 | Story 卡片含 feature_design_refs。 | 缺失则 CP4 FAIL。 |
| lld-designer | CP5 前 | 本 DESIGN、TEST-PLAN、TASKS、前置 Feature contracts。 | full LLD。 | 前置 Feature 未完成则标 blocker。 |
| meta-qa | CP7 / CP8 | dry-run report、security guard、rollback evidence。 | verification report。 | 真实样本只读，人工确认 release repo 未写。 |
| FEAT-PG-004/005/006/007/008 | LLD / 实现前 | registry / impact / refresh / stale / follow-up contracts。 | migration adapters consume stable outputs。 | 上游缺失时 migration blocked。 |

## 风险与回退

| Risk ID | 风险 | 影响 | 缓解 | 回退 |
|---|---|---|---|---|
| R-QL-01 | 误写 quant-lab 发布库。 | 未授权生产写和正式产物破坏。 | read-only guard、dry-run default、manual review。 | 立即停止，恢复发布库，创建 incident / CR。 |
| R-QL-02 | 样本特例污染通用 registry。 | capability namespace 漂移。 | unresolved 输出 blocked，不自动注册。 | 回退 registry change，补正式决策。 |
| R-QL-03 | process-side cleanup 误删必要状态。 | 过程态恢复困难。 | before/after hash、rollback refs、dry-run。 | 使用 rollback refs 恢复。 |
| R-QL-04 | 前置 Feature 未完成即迁移。 | 迁移报告不可信。 | W5 依赖 W1-W4 验收。 | 保持 dry-run blocked，等待前置完成。 |
| R-QL-05 | 发布库 stale report 被当成已修复。 | 正式 docs 继续陈旧。 | FU-RF/CR candidate 状态明确。 | 保持 stale finding open，直到正式 CR 关闭。 |
