---
checkpoint_id: "CP2-CR-037-REQUIREMENTS-BASELINE"
checkpoint_name: "CR-037 Requirements / Scenario / Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-02T17:27:19+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-02T17:45:07+08:00"
auto_check_result: "process/checks/CP2-CR-037-REQUIREMENTS-BASELINE.result.json"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/context/CP2-CR-037-CONTEXT.yaml"
    - "process/changes/CR-037.md"
    - "process/docs/product/USE-CASES.md"
    - "process/docs/product/REQUIREMENTS.md"
    - "process/docs/product/SCENARIOS.yaml"
    - "process/docs/product/TEST-MATRIX.md"
    - "process/docs/product/STORY-MAP.md"
    - "process/docs/product/MVP-SCOPE.md"
    - "process/discussions/CP2-CR-037-SCENARIO-DISCUSSION-LOG.md"
    - "process/checks/CP2-CR-037-DISCUSSION-CHECKPOINT.json"
---

# CP2 CR-037 Requirements / Scenario / Scope Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR-037-REQUIREMENTS-BASELINE.result.json` | PASS | 0 | CR-037 产品 / 场景 / 范围基线可进入人工确认；不授权实现。 |
| `process/checks/CP2-CR-037-DISCUSSION-CHECKPOINT.json` | completed | 0 | CR-037 Scenario Gray Areas 和 3 条用户可见确认已记录。 |
| `process/checks/CP0-CR-037-BOOTSTRAP.result.json` | PASS | 0 | CR-037 已激活，CP0 通过，下一路由为 human_gate。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-037 的产品 / 场景 / 范围基线，允许进入 CP3 蓝图 / HLD 人工确认路径。 |
| 推荐动作 | `approve`；当前 UC、REQ、SCENARIOS、TEST-MATRIX、Story Map 和 MVP Scope 已自洽，CP2 自动预检无 blocker。 |
| approve 后会发生什么 | `CR-037` 可从 requirement-clarification 推进到 CP3 solution-design / HLD 人工确认；后续仍需 CP3 和 CP5 才能进入实现。 |
| approve 不授权什么 | 不授权实现、CP5、runtime、生产写入、publish、live、真实交易、读取凭据、修改 quant-lab 发布库，也不关闭、恢复或完成 `CR-036`。 |
| 不确认会阻塞什么 | 阻塞 CR-037 进入 CP3；P0 state enforcement、P1 project governance、P2 quant-lab migration 均不得开始实现。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR-037-CONTEXT.yaml` |
| capsule 状态 | `ready` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | `0 次；CP2 brief 只引用正式基线和检查结果路径，不复制长文档正文` |
| 缺失 / waived 理由 | `RELEASE-SLICES.md` 与 `BACKLOG.md` 未单独生成；release slices 已由 `STORY-MAP.md` 承载，deferred backlog 已由 `MVP-SCOPE.md` 承载，CP2 作为 N/A-with-reason 处理。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | n/a | 0 | 0 | 当前 v2 轻量状态不存重型 pending queue；本轮从 CR、summary、discussion 和产品基线聚合。 |
| 委托 Agent 交还摘要 | `process/state/AGENT-DISPATCH-LEDGER.ndjson` / `D-CR037-META-PM-BASELINE` | scanned | 1 | 0 | baseline 已由 meta-pm 产出并在 CP0 证据中引用；本轮无新增 blocker。 |
| 自动预检结果 | `process/checks/CP2-CR-037-REQUIREMENTS-BASELINE.result.json` | scanned | 1 | 0 | 自动预检 PASS；无 blocker，下一步为 human_gate。 |
| discussion log / checkpoint | `process/discussions/CP2-CR-037-SCENARIO-DISCUSSION-LOG.md` / `process/checks/CP2-CR-037-DISCUSSION-CHECKPOINT.json` | scanned | 5 | 5 | 5 个 SGA 已收敛；scope、CR-036 边界、不授权项、registry / roadmap / P2 边界纳入 DQ。 |
| 下游正式产物 | `USE-CASES.md` / `REQUIREMENTS.md` / `SCENARIOS.yaml` / `TEST-MATRIX.md` / `STORY-MAP.md` / `MVP-SCOPE.md` | scanned | 6 | 4 | 追溯链已闭合；allowlist、second-system guardrail、roadmap refresh、capability registry、P2 migration 边界纳入 DQ；release slices/backlog 分别由 Story Map / MVP Scope 承载，N/A-with-reason。 |
| 用户显式选择题 | 当前对话 / `process/changes/CR-037.md` | scanned | 3 | 3 | 用户已批准方案建档、暂停 CR-036 并激活 CR-037、推进 CP2；本门禁仍需确认 CP2 基线和高风险边界。 |

> 发起人工确认前已扫描适用来源。当前纳入 7 个待人工决策项，覆盖 CP2 范围、CR-036 风险边界、不授权边界、second-system guardrail、roadmap 发布库边界、registry 引用边界和 P2 quant-lab 迁移边界。

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 2 | `CP2-CR037-DQ-01`、`CP2-CR037-DQ-07` 需要用户确认产品基线和 P2 quant-lab 迁移是否保留在本 CR 范围。 |
| 高风险策略确认 | 5 | `CP2-CR037-DQ-02` 至 `CP2-CR037-DQ-06` 明确 CR-036、授权、second-system、roadmap 发布库和 registry 边界。 |
| agent 默认处理 | 2 | release slices/backlog 单独文件 N/A-with-reason；后续 CP3 由 meta-se 使用现有蓝图 / HLD 基线。 |
| 仅审计记录 | 4 | CP0 PASS、CP2 context ready、dispatch evidence、状态语义残留扫描已闭合。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-CR037-DQ-01 | scope | 是否批准 CR-037 产品 / 场景 / 范围基线，并允许进入 CP3 蓝图 / HLD 人工确认路径？ | 批准当前基线，进入 CP3；实现仍需 CP3 / CP5。 | 备选 A：要求修改指定 UC / REQ / Scenario / Scope 后重发 CP2；备选 B：reject 并暂停 CR-037。 | 推荐方案保持八轮评审收敛结果并推进设计确认；备选 A 更谨慎但延迟 P0 state enforcement；备选 B 停止整改。 | 影响 CR-037 后续全部 P0/P0.5/P1/P2 切片；若基线错误会传导到 HLD 和 Story。 | 用户回复 `修改: <具体修改点>` 时回到 requirement-clarification 修订基线；回复 `reject` 时 CR-037 停止推进。 |
| CP2-CR037-DQ-02 | risk_acceptance | 是否接受 `CR-036` 仍 blocked / unfinished、且冲突检查仍会报告 CR-037 与 CR-036 在 `changes` surface 重叠？ | 接受该风险边界：CR-036 保持 blocked，不视为完成；CP2 brief 保留冲突说明。 | 备选 A：先恢复并完成 / 关闭 CR-036 后再继续 CR-037；备选 B：把 CR-036 显式 supersede 后再推进。 | 推荐方案符合用户已授权的暂停策略，能继续处理更紧急的状态治理；备选 A 最干净但会延迟 CR-037；备选 B 需要额外 CR-036 关闭审查。 | 冲突检查预期失败会继续存在；若被误读，可能把 CR-036 当作已完成。 | 若后续 CP3 / CP5 需要无冲突状态，先恢复 CR-036 或单独执行关闭 / supersede 审查。 |
| CP2-CR037-DQ-03 | security | 是否确认 CP2 approve 不授权实现、CP5、runtime、生产写入、publish、live、真实交易、读取凭据或 quant-lab 发布库修改？ | 确认不授权边界；CP2 只批准产品基线，任何高风险动作需后续门禁或独立授权。 | 备选 A：在 CP2 中额外授权有限实现预研；备选 B：要求新增 runtime / publish 授权门禁后再继续。 | 推荐方案最小权限且与 meta-flow 门禁一致；备选 A 会越过 CP3/CP5；备选 B 对当前无 runtime 需求过重。 | 防止 CP2 被误解为实现或生产授权；保护 quant-lab 发布库和凭据边界。 | 若用户需要 runtime 或发布库写入，必须创建独立授权项或后续 CR，并补安全 / 回滚 / 验证证据。 |
| CP2-CR037-DQ-04 | implementation | 是否确认 P0 state enforcement 主路线为 allowlist + field budget + audit/enforce 灰度，并阻断新建第二套机制？ | 接受 allowlist + budget + audit/enforce；黑名单 strip、hot/warm/cold 新术语、PROJECT-LEDGER、roadmap_impact、自由 capability 命名空间均不作为主路线。 | 备选 A：先 audit-only，延后 enforce；备选 B：只做检查报告，不阻断写入。 | 推荐方案能根治未知字段和重型状态污染；备选 A 误伤风险低但污染继续存在；备选 B 最保守但不能形成真实门禁。 | allowlist 可能误伤存量合理字段；audit/enforce 灰度用于降低该风险。 | 若 audit 阶段发现合理字段缺失，先补 schema / budget 再切 enforce；若误伤严重，回退到 audit-only。 |
| CP2-CR037-DQ-05 | runtime_authorization | 是否确认 roadmap refresh cascade 只自动写过程归档库机器状态，quant-lab 发布库只输出 stale finding / follow-up？ | 确认只自动写过程归档库；发布库影响进入 must_check、stale_items 和 FU-RF。 | 备选 A：完全禁用 cascade，只生成报告；备选 B：未来在独立 CR 和明确授权下允许发布库写入。 | 推荐方案兼顾可追踪和最小授权；备选 A 最安全但自动化价值低；备选 B 需要额外事务 / 回滚设计。 | 避免把 `UPDATED_WITH_DOC_IMPACTS` 误解成发布库写入授权。 | 若未来需要发布库自动写入，必须新建授权 CR，补回滚、测试和跨仓一致性策略。 |
| CP2-CR037-DQ-06 | implementation | 是否确认 feature_refs / capability_refs 必须引用标准 registry，缺失 ID 不得由迁移过程自动创造？ | 确认 registry refs 只引用真实已注册 ID；缺失时 blocked finding 或 FU-RF。 | 备选 A：允许生成 proposed registry entries，但需人工确认后入库；备选 B：短期允许自由字符串，仅 warning。 | 推荐方案避免第二套 capability 命名空间；备选 A 迁移更顺滑但需额外审核；备选 B 成本低但会延续漂移。 | 过严可能增加 quant-lab 迁移阻力；过松会让冲突检测和 capability claims 继续失真。 | 若标准 registry 无法承载真实能力，先扩展 registry schema，再迁移引用。 |
| CP2-CR037-DQ-07 | scope | 是否确认 P2 quant-lab migration 是验证切片，不授权发布库代码 / 测试 / 正式文档修改？ | 确认 P2 只做过程侧状态清理、能力归一、impact migration、stale report 和 FU-RF；发布库改动后续另行授权。 | 备选 A：暂缓 P2，只完成 P0/P1 机制；备选 B：将 quant-lab 迁移拆成独立 CR。 | 推荐方案用真实项目验证治理机制但保持写入边界；备选 A 降低范围风险但缺真实样本；备选 B 审计更干净但增加 CR 成本。 | P2 过早绑定真实项目可能扩大范围；不做 P2 则缺少迁移验证闭环。 | 若 P1 机制未稳定或 quant-lab 发布库需要修改，暂停 P2 并另起 CR / 授权门禁。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受以上 7 项推荐方案，允许 CR-037 进入 CP3 蓝图 / HLD 人工确认。 |
| 备选方案 | 对任一 DQ 可回复 `修改: CP2-CR037-DQ-xx <具体修改点>`；也可 `reject` 暂停 CR-037。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全 / 权限、交付影响。 |
| 优劣分析 | 已在待人工决策清单逐项列出。 |
| 风险与回退 | 风险等级 medium；回退到 requirement-clarification 修订产品基线；不授权 runtime / quant-lab 发布库写入。 |
| 用户需决策事项 | `CP2-CR037-DQ-01`、`CP2-CR037-DQ-02`、`CP2-CR037-DQ-03`、`CP2-CR037-DQ-04`、`CP2-CR037-DQ-05`、`CP2-CR037-DQ-06`、`CP2-CR037-DQ-07`。 |

### CP2 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 用户真实意图 | 用 meta-flow 自身整改能力解决长期项目治理弱、状态膨胀、token 读取成本高、roadmap / phase 缺失和 quant-lab 迁移治理问题。 |
| 场景覆盖 | `UC-PG-001..007` 覆盖 state enforcement、refs-only project state、project scale、registry refs、impact surface、roadmap refresh、quant-lab migration。 |
| 认知盲区补充 | 明确分离读取分层、长期对象模型和规模治理；避免再造第二套 context / impact / capability / ledger / result 机制。 |
| Scenario Gray Areas 处理结果 | `SGA-CR037-01..05` 均已收敛，见 `process/discussions/CP2-CR-037-SCENARIO-DISCUSSION-LOG.md`。 |
| Deferred Ideas | 跨仓自动修改 quant-lab 发布库、regulated 独立档、更细粒度 scale 延后，不进入当前 CP2 范围。 |
| 用户选择影响 | approve 后进入 CP3；不 approve 则 P0 state enforcement 与后续治理实现均阻塞。 |
| 回退方式 | `修改: <具体修改点>` 回到产品基线修订；`reject` 停止 CR-037；需要处理 CR-036 时恢复 / 关闭 / supersede CR-036。 |
| discussion log / checkpoint | `process/discussions/CP2-CR-037-SCENARIO-DISCUSSION-LOG.md`；`process/checks/CP2-CR-037-DISCUSSION-CHECKPOINT.json`。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP0 已通过 | 待审查 | `process/checks/CP0-CR-037-BOOTSTRAP.result.json` |  |
| CR-037 是当前 active CR | 待审查 | `process/state/STATE.current.json` / `process/changes/CR-037.md` |  |
| CR-036 暂停边界明确 | 待审查 | `process/changes/CR-036.md` / `process/changes/CR-037.md` |  |
| 产品基线存在 | 待审查 | `process/docs/product/USE-CASES.md` / `process/docs/product/REQUIREMENTS.md` |  |
| 工程验证场景存在 | 待审查 | `process/docs/product/SCENARIOS.yaml` / `process/docs/product/TEST-MATRIX.md` |  |
| 产品规划输入存在 | 待审查 | `process/docs/product/STORY-MAP.md` / `process/docs/product/MVP-SCOPE.md` | `RELEASE-SLICES.md` / `BACKLOG.md` 为 N/A-with-reason。 |
| 场景讨论证据存在 | 待审查 | `process/discussions/CP2-CR-037-SCENARIO-DISCUSSION-LOG.md` / `process/checks/CP2-CR-037-DISCUSSION-CHECKPOINT.json` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | UC-PG-001..007 是否完整表达本轮整改用户目标 | 待审查 | `process/docs/product/USE-CASES.md` |  |
| 2 | REQ-PG-* 是否覆盖 P0/P0.5/P1/P2 切片 | 待审查 | `process/docs/product/REQUIREMENTS.md` |  |
| 3 | SCENARIOS / TEST-MATRIX 是否闭合需求、Story 和测试状态追溯 | 待审查 | `process/docs/product/SCENARIOS.yaml` / `process/docs/product/TEST-MATRIX.md` |  |
| 4 | Story Map 是否表达正确依赖顺序 | 待审查 | `process/docs/product/STORY-MAP.md` |  |
| 5 | MVP Scope 是否明确 In Scope / Out of Scope / Deferred | 待审查 | `process/docs/product/MVP-SCOPE.md` |  |
| 6 | 不造第二套机制约束是否进入产品基线 | 待审查 | `ST-PG-013` / `IN-PG-011` |  |
| 7 | CR-036 暂停但未完成的边界是否清楚 | 待审查 | `CP2-CR037-DQ-02` |  |
| 8 | CP2 不授权实现和高风险操作是否清楚 | 待审查 | `CP2-CR037-DQ-03` |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 产品 / 场景 / 范围基线被人工确认 | 待审查 | 本 checkpoint 人工审查结果 |  |
| 待人工决策项全部有结论 | 待审查 | `CP2-CR037-DQ-01..07` |  |
| 无 CP2 blocker | 待审查 | `process/checks/CP2-CR-037-REQUIREMENTS-BASELINE.result.json` |  |
| 可进入 CP3 | 待审查 | 本 checkpoint approved 后 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 context capsule | `process/context/CP2-CR-037-CONTEXT.yaml` | 待审查 |  |
| CP2 result JSON | `process/checks/CP2-CR-037-REQUIREMENTS-BASELINE.result.json` | 待审查 |  |
| CP2 discussion log | `process/discussions/CP2-CR-037-SCENARIO-DISCUSSION-LOG.md` | 待审查 |  |
| CP2 discussion checkpoint | `process/checks/CP2-CR-037-DISCUSSION-CHECKPOINT.json` | 待审查 |  |
| Use Cases | `process/docs/product/USE-CASES.md` | 待审查 |  |
| Requirements | `process/docs/product/REQUIREMENTS.md` | 待审查 |  |
| Scenarios | `process/docs/product/SCENARIOS.yaml` | 待审查 |  |
| Test Matrix | `process/docs/product/TEST-MATRIX.md` | 待审查 |  |
| Story Map | `process/docs/product/STORY-MAP.md` | 待审查 |  |
| MVP Scope | `process/docs/product/MVP-SCOPE.md` | 待审查 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-02T17:45:07+08:00
- 修改意见：用户确认 `CP2-CR037-DQ-01` 至 `CP2-CR037-DQ-07` 全部按推荐方案处理。
- 风险接受项：接受 `CR-036` 保持 `blocked / unfinished`，冲突检查预期失败继续显式暴露；确认 CP2 不授权实现、CP5、runtime、production write、publish、live、凭据读取或 quant-lab 发布库修改。
