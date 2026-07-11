---
checkpoint_id: "CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE"
checkpoint_name: "CR163 Requirement, Scenario and Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-11T10:45:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-11T10:52:17+08:00"
auto_check_result: "process/checks/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.result.json"
decision_brief_profile: "compact"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts: ["docs/product/USE-CASES.md", "docs/product/REQUIREMENTS.md", "docs/product/SCENARIOS.yaml", "docs/product/TEST-MATRIX.md", "docs/product/STORY-MAP.md", "docs/product/MVP-SCOPE.md", "docs/product/RELEASE-SLICES.md", "docs/product/BACKLOG.md"]
---

# CP2 CR163 Requirement, Scenario and Scope Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.result.json` | PASS | 0 | 15 项检查全部 PASS，waiver=0。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 把 trial_count 从事后声明升级为未来研究运行中自动记录、可封存、可校验、可审计的 experiment-family raw lineage 事实。 |
| 推荐动作 | `approve`：确认 CR163 产品基线和五 Story 目标，进入 CP3 架构设计。 |
| approve 后会发生什么 | 主编排器真实拉起 `meta-se-critical`；冻结 family/trial/attempt/selection contract、append-only storage、seal/supersession、两条 producer chain 集成、现有 admission consumer 接口和五 Story 正式拆解输入。 |
| approve 不授权什么 | 不授权代码实现、真实 lake/NAS/provider/credential、simulation/paper/live/broker/trading、外部框架、Git remote write、publish/release；不授权 effective-trial/FDR/PBO/DSR/OOS/TCA/capacity 计算或 CR155 lineage 回填。 |
| 不确认会阻塞什么 | CP3/HLD、正式 Story/DAG、CP5 设计证据和后续实现均保持阻断。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。 |
| 全文档读取扩展 | 2 次：SGQ discussion log 与 meta-pm return summary 的 CP2 human audit，均已写入 READ-EXPANSION-LEDGER。 |
| 缺失 / waived 理由 | 无；waiver=0。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` | scanned | 1 | 0 | SGQ 阻塞已由用户回答，转 resolved-by-user。 |
| 委托 Agent 交还摘要 | `process/handoffs/CR163-CP2-META-PM-RETURN-SUMMARY.md` | scanned | 4 | 0 | SGQ-001..004 全部 confirmed A；纳入审计摘要。 |
| 自动预检结果 | `process/checks/CP1-*` / `CP2-*` | scanned | 0 | 0 | CP1/CP2 precheck 均 PASS。 |
| discussion log / checkpoint | `process/discussions/CP2-CR163-*` / `process/checks/CP2-CR163-DISCUSSION-CHECKPOINT.json` | scanned | 4 | 0 | 4/4 resolved-by-user，remaining=0。 |
| 下游正式产物 | HLD / LLD / REVIEW / FIXES | n/a | 0 | 0 | CP2 前尚未生成，按 route 正常。 |
| 用户显式选择题 | 用户阶段目标评估 | scanned | 4 | 0 | 明确确认 inventory/count/effective boundary/seal 四项推荐。 |
| CP2 基线整体批准 | 当前 checkpoint | scanned | 3 | 3 | 以下 DQ 需要正式 gate approve。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 3 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 0 | 本轮无 runtime/credential/external-write 授权。 |
| agent 默认处理 | 2 | S03 文案归一化；CP3 冻结最终对象名与 artifact path。 |
| 仅审计记录 | 4 | SGQ-CR163-001..004 confirmed A。 |

### 已解决 Scenario Gray Areas

| SGQ | 用户结论 | 基线化结果 |
|---|---|---|
| SGQ-CR163-001 | A | 2 条去重 producer chains / 4 mappings；S03 4/4 覆盖。 |
| SGQ-CR163-002 | A | retry=attempt；seed/参数差异=trial；failed/cancelled/excluded 保留并计数。 |
| SGQ-CR163-003 | A | effective count 维持 typed_unavailable，C1 只 raw-input-ready。 |
| SGQ-CR163-004 | A | sealed immutable；correction 只能 supersession。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR163-001 | scope | 是否批准 CR163 以 2 条 producer chains / 4 mappings、5 个 Story 目标覆盖 family lifecycle、automatic lineage、seal/validator 与现有 gate integration？ | 批准当前基线；S03 单 Story 覆盖全部 4 mappings。 | 修改范围后重跑 CP1/CP2；或拒绝并将 CR163 退回 candidate。 | 推荐方案覆盖已确认绕过点且不增加 Story；缩减会留下 lineage gap，扩大则混淆 producer/consumer。 | 决定 CP3 架构边界、Story owner 和 100% coverage 分母。 | 发现新真实 candidate producer 时回退 CP2 或创建增量 CR；当前拒绝则关闭/退回候选。 |
| DQ-CP2-CR163-002 | scope | 是否批准 raw-lineage claim ceiling：完整 instrumented future run 才可 present，uninstrumented 为 typed_unavailable，invalid/tampered 为 blocked；C1 仍不可计算？ | 批准；effective count/ref/method 继续 unavailable/empty，CR155 保持 blocked。 | 将统计计算并入本 CR；或维持所有路径 typed_unavailable。 | 推荐方案实现事实源而不制造统计过度声明；合并统计会扩大方法和验证风险；全 unavailable 无法达成 CR163 价值。 | 决定 admission consumer 语义和后续 FU-002 边界。 | 若统计方法后续获批，另由 FU-002 扩展；不得在 CR163 静默切换。 |
| DQ-CP2-CR163-003 | security | 是否批准 deny-default 权限与 standard 路由：CP2 后只进入设计，任何实现/真实数据/runtime/外部写入仍受后续门禁与独立授权约束？ | 批准 architecture-major CP3→CP4→CP5→CP6→独立 CP7→CP8 路由。 | 暂停在 CP2；或拆出独立 runtime-authorization CR 后再评估。 | 推荐方案允许离线 contract/fixture 设计继续且不放宽权限；暂停无风险但阻塞价值；混入 runtime 授权会扩大审计边界。 | 决定是否可启动 meta-se-critical；不会授权实现或运行。 | 任何 credential/real lake/NAS/provider/broker/trading/publish 需求出现时立即 BLOCKED 并另行授权。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 DQ-CP2-CR163-001..003 的推荐方案并进入 CP3。 |
| 备选方案 | `修改: <决策 ID + 修改点>` 后回到 requirement-clarification；或 `reject` 退回/关闭 CR163。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、安全权限、交付与后续统计边界。 |
| 优劣分析 | 当前推荐保持五 Story 最小闭环；缩减留下 instrumentation gap，扩大到统计/runtime 则破坏 CR 边界。 |
| 风险与回退 | CP3 仅设计；scope 变化回 CP2；任何权限扩大立即 BLOCKED。 |
| 用户需决策事项 | `DQ-CP2-CR163-001`、`DQ-CP2-CR163-002`、`DQ-CP2-CR163-003`。 |

### CP2 追加摘要

- 用户真实意图：系统自动记录 raw trial lineage，替代人工后验 trial_count。
- 场景覆盖：12 个 P0，覆盖声明、记录、retry/seed/duplicate、fail/cancel/exclude、seal、supersession、tamper、permission、missing instrumentation、CR155 regression。
- 认知盲区补充：仓库有两条实际 producer chains，不是单一 Stage 3；因此采用 4 mappings / 2 chains。
- Deferred Ideas：effective-trial/statistical producer、historical backfill、real ML/event runner instrumentation。
- 用户选择影响：五 Story 数量不变；S03 扩写为两条 chains 全覆盖；present 条件和 C1 ceiling 更精确。
- 回退方式：scope 变化回 CP2；架构问题回 CP3；CP5 前不允许实现。
- discussion log：`process/discussions/CP2-CR163-SCENARIO-DISCUSSION-LOG.md`。
- discussion checkpoint：`process/checks/CP2-CR163-DISCUSSION-CHECKPOINT.json`。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP1/CP2 自动预检通过 | approved | CP1/CP2 result JSON | 自动预检 PASS。 |
| SGQ 4/4 confirmed | approved | Discussion checkpoint | 用户确认全部 A。 |
| Capsule ready | approved | CP2 context | compact capsule-first。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 批准两条 producer chains / 4 mappings 与五 Story 目标 | approved | DQ-CP2-CR163-001 | 接受推荐方案。 |
| 2 | 批准 availability/C1 claim ceiling 与 CR155 negative regression | approved | DQ-CP2-CR163-002 | 接受推荐方案。 |
| 3 | 批准 deny-default 权限与 architecture-major 路由 | approved | DQ-CP2-CR163-003 | 接受推荐方案；不授权实现或运行。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 产品基线可作为 CP3 输入 | approved | 本 checkpoint | 可进入 HLD/ADR 设计。 |
| 用户明确批准/修改/拒绝 | approved | 人工审查结果 | 用户回复 approve。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 产品基线 | `docs/product/*` CR163 sections | approved | 作为 CP3 输入。 |
| CP2 result | `process/checks/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-SCOPE.result.json` | approved | PASS。 |
| CP2 capsule | `process/context/CP2-CR163-TRIAL-LINEAGE-INSTRUMENTATION-CONTEXT.yaml` | approved | ready。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-11T10:52:17+08:00
- 修改意见：无；接受 DQ-CP2-CR163-001..003 推荐方案并继续推进到下一人工门禁。
- 风险接受项：无；本 CP2 不接受或授权 runtime/data/credential/external-write 风险。
