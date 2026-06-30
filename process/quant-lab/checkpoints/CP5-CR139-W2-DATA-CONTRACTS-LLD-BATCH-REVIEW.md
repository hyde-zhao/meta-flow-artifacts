---
checkpoint_id: "CP5"
checkpoint_name: "CR139 W2 Data Contracts LLD Batch Preparation"
type: "manual-preparation-brief"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-28T23:14:15+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-29T07:10:07+08:00"
auto_check_result: "process/checks/CP5-CR139-W2-DATA-CONTRACTS-LLD-BATCH.result.json"
target:
  phase: "story-planning"
  cr_id: "CR-139"
  batch_id: "CR139-W2-DATA-CONTRACTS"
  artifacts:
    - "process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml"
    - "process/checks/CP5-CR139-W2-DATA-CONTRACTS-DESIGN-EVIDENCE-CONVERGENCE-PLAN.md"
    - "process/stories/CR139-S33-catalog-manifest-source-of-truth.md"
implementation_allowed: false
runtime_allowed: false
auto_final_authorization: false
approval_scope: "preparation-gate-only; design-evidence-writing-only"
---

# CP5-W2-DATA-CONTRACTS 批次准备 Decision Brief

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR139-W2-DATA-CONTRACTS-LLD-BATCH.result.json` | BLOCKED | 1 | 批次选择和收敛计划已准备；正式 CP5 approval 被阻断，直到 S09 LLD 和八个 technical-note 实际完成并通过 per-story precheck |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 启动 CR139 后续 CP5 批次准备，默认选择 `CR139-W2-DATA-CONTRACTS`，冻结 S09/S14/S22/S30/S31/S32/S33/S34/S39 的设计证据收敛范围、顺序、S33 验收补充和不授权边界。 |
| 推荐动作 | `approve` 本批次准备方案：先收敛 `CR139-W2-DATA-CONTRACTS`，不先启动 ML/read consistency 或 config/audit 批次。 |
| approve 后会发生什么 | meta-dev 或经用户批准的 inline-fallback 进入设计证据写作：S09 产出 full LLD；S14/S22/S30/S31/S32/S33/S34/S39 产出 technical-note；S33 必须包含 `catalog.py` CR014/CR018 命名清理验收补充。完成后重新生成正式 CP5 自动预检和 Decision Brief。 |
| approve 不授权什么 | 不授权实现；不授权 runtime、真实 lake 写入、provider catalog 写入、provider-lake-catalog 写入、published pointer advance execution、物理分区迁移执行、凭据读取、NAS/QMT/trading 或 Git remote write。 |
| 不确认会阻塞什么 | 阻塞 CR139 Wave2/W3 的设计证据收敛；任何 W2/W3 Story 都不得进入 `dev-ready` 或实现。 |
| 自动终验授权 | `auto_final_authorization: false`；本准备门禁不授权终验、实现、runtime、真实 lake 写入、provider catalog 写入、published pointer advance execution 或物理分区迁移执行。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml` |
| capsule 状态 | ready-for-design-evidence-convergence |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；只读 batch 必需的 STATE.current、Wave1 follow-up context、STORY-STATUS、DEVELOPMENT-PLAN 和 STORY-BACKLOG |
| 全文档读取扩展 | 0 次；未读取完整 HLD、完整 LLD 集、完整验证报告或完整 git diff |
| 缺失 / waived 理由 | 不适用；本轮是批次准备，不是正式 CP5 approval |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE.current | `process/state/STATE.current.json` | scanned | 1 | 1 | next_action 已指定默认批次和禁止项 |
| Follow-up capsule | `process/context/CR139-WAVE1-COMPLETE-FOLLOWUP-CONTEXT.yaml` | scanned | 1 | 1 | 推荐首批 `CR139-W2-DATA-CONTRACTS` |
| Follow-up CP5 context | `process/context/CR139-FOLLOWUP-CP5-IMPLEMENTATION-CONTEXT.yaml` | scanned | 3 | 3 | CR139-FU-DQ-01/02/03 转入本 Brief |
| Story status | `process/STORY-STATUS-CR139.md` | scanned | 1 | 0 | W2/W3 全部 pending-cp5；作为阻断证据，不需用户另决策 |
| Development plan | `process/DEVELOPMENT-PLAN-CR139.yaml` | scanned | 1 | 0 | merge_order 已有，agent 默认遵守 |
| Story cards | S09/S14/S22/S30/S31/S32/S33/S34/S39 | scanned | 1 | 1 | S33 验收补充需要用户确认纳入 |
| `catalog.py` symbol scan | `market_data/catalog.py` | scanned | 1 | 1 | CR014/CR018 命名清理归属 S33 |
| 用户显式约束 | 当前请求 | scanned | 1 | 1 | runtime / real write / provider catalog / pointer advance / migration 禁止项 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 2 | 确认批次范围；确认 S33 接收 CR014/CR018 命名清理验收补充 |
| 高风险策略确认 | 1 | 明确本轮不授权 runtime、真实 lake 写入、provider catalog 写入、pointer advance execution、物理分区迁移 |
| agent 默认处理 | 3 | 按 merge_order 写设计证据；S39 不承担命名清理主体；无阻断 LCQ 时继续收敛 |
| 仅审计记录 | 2 | Wave1 verified；W2/W3 仍 pending-cp5，不得 dev-ready |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR139-W2-DATA-01 | implementation | 是否按默认启动 `CR139-W2-DATA-CONTRACTS` 作为 CR139 后续首个 CP5 设计证据批次？背景：Wave1 已 verified，后续 ML/read/config/audit 工作都依赖 schema、run-id、catalog/manifest、incremental/pointer 等底层契约先收敛。 | 启动 `CR139-W2-DATA-CONTRACTS`，覆盖 S09/S14/S22/S30/S31/S32/S33/S34/S39。 | A. 先启动 ML/read consistency 批次；B. 只启动 S33/S39 catalog 子批次。 | 推荐方案减少后续返工，先冻结共享契约；A 可能在 schema/run-id/catalog 未定时放大返工；B 范围更小但会延迟 S09/S14/S22 等共同契约。 | 影响后续 CP5 设计顺序、文件 merge_order、Story 可实现性；风险是本批覆盖 9 个 Story，需要严格区分准备与正式 CP5。 | 若 S09 full LLD 复杂度过高，可拆出 S09 单独 CP5 子批；若 catalog 命名风险过大，可把 S33/S39 子批单独提前。 |
| DQ-CP5-CR139-W2-DATA-02 | implementation | 是否把 `catalog.py` 中 CR014/CR018 命名清理作为 S33 验收补充？背景：`catalog.py` 仍存在 `CR014_CATALOG_POINTER_REQUIRED_FIELDS`、`CR018_*`、`build_cr018_*()` 和 `_cr018_*()` 等历史 CR 命名。 | 纳入 S33：新增领域命名 API / constants，保留旧 CR014/CR018 alias 和兼容测试；S39 只负责 `triggered_by_cr` / `run_lineage`。 | A. 保持 CR014/CR018 名称不动，只记录历史来源；B. 直接删除或破坏性重命名旧 API。 | 推荐方案提高领域语义并保留兼容；A 最稳但继续扩散 CR 命名；B 清理最彻底但破坏测试和历史合同风险高。 | 影响 `market_data/catalog.py` public API、测试、历史合同兼容和 S39 边界；风险是命名迁移若无 alias 会导致导入破坏。 | 若发现旧 API 被外部调用广泛依赖，则先只新增领域 API 和 docs，不 deprecate；若测试可完全迁移，再计划后续 CR 删除 alias。 |
| DQ-CP5-CR139-W2-DATA-03 | runtime_authorization | 本轮是否授权 runtime、真实 lake 写入、provider catalog 写入、published pointer advance execution 或物理分区迁移执行？背景：S14/S33/S39/S08 相关主题容易被误读为可执行真实写入或 pointer/migration。 | 本轮不授权任何上述操作；仅允许设计证据收敛。 | A. 只授权 fixture/dry-run 验证；B. 后续单独授权 scoped real operation。 | 推荐方案符合当前 CP5 准备门，不引入真实数据风险；A 可用于后续 CP6/CP7 fixture；B 只能在独立 runtime gate 明确 action scope、窗口、回滚和审计后执行。 | 安全 / 数据一致性 / 审计风险高；误授权会影响真实 lake、catalog pointer、物理分区和 provider catalog。 | 如用户明确需要真实执行，停止当前准备流程，创建独立 `runtime_authorization` 决策项并限定 scope。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 `CR139-W2-DATA-CONTRACTS` 作为下一批设计证据收敛范围；接受 S33 验收补充；确认本轮不授权 runtime / real write / provider catalog / pointer advance / physical migration。 |
| 备选方案 | 见 DQ-01 至 DQ-03；可选择先做 ML/read consistency、只做 catalog 子批、保持 CR 命名不动，或另起 runtime gate。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全 / 权限、交付影响。 |
| 优劣分析 | 推荐方案优先稳定底层数据契约，降低后续返工；代价是先处理 9 个 Story 的设计证据，正式 CP5 approval 仍需等待证据完成。 |
| 风险与回退 | 风险等级 HIGH：数据契约、catalog/pointer/migration 容易被误授权；回退方式是缩小批次或拆分 S09/S33/S39 子批。 |
| 用户需决策事项 | DQ-CP5-CR139-W2-DATA-01、DQ-CP5-CR139-W2-DATA-02、DQ-CP5-CR139-W2-DATA-03。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| Wave1 verified | 待审查 | `process/STORY-STATUS-CR139.md` | S01-S08 verified |
| 后续首批推荐存在 | 待审查 | `process/context/CR139-WAVE1-COMPLETE-FOLLOWUP-CONTEXT.yaml` | 推荐 `CR139-W2-DATA-CONTRACTS` |
| 批次上下文已生成 | 待审查 | `process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml` | read_profile=compact |
| 设计证据尚未完成 | 待审查 | `process/checks/CP5-CR139-W2-DATA-CONTRACTS-LLD-BATCH.result.json` | 正式 CP5 approval 阻断，需先写证据 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 批次范围是否覆盖用户指定 S09/S14/S22/S30/S31/S32/S33/S34/S39 | 待审查 | 本 Brief；context capsule |  |
| 2 | 设计证据类型分布是否合理（S09 full-lld；其余 technical-note） | 待审查 | `process/STORY-BACKLOG-CR139.md`；`process/STORY-STATUS-CR139.md` |  |
| 3 | merge_order 是否明确 | 待审查 | 本 Brief；`process/DEVELOPMENT-PLAN-CR139.yaml` |  |
| 4 | S33 CR014/CR018 命名清理验收补充是否纳入 | 待审查 | `process/stories/CR139-S33-catalog-manifest-source-of-truth.md` |  |
| 5 | approve 不授权项是否完整列出 | 待审查 | 本 Brief；context capsule |  |
| 6 | 正式 CP5 approval 是否被正确阻断直到证据完成 | 待审查 | result JSON decision=BLOCKED |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认批次准备方案 | 待审查 | 本 Brief DQ-01 |  |
| 用户确认 S33 验收补充 | 待审查 | 本 Brief DQ-02 |  |
| 用户确认本轮不授权 runtime / real write / pointer / migration | 待审查 | 本 Brief DQ-03 |  |
| meta-dev 可进入设计证据写作 | 待审查 | approve 后生效 | 仍不得实现 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 W2 context capsule | `process/context/CP5-CR139-W2-DATA-CONTRACTS-LLD-CONTEXT.yaml` | 待审查 |  |
| 设计证据收敛计划 | `process/checks/CP5-CR139-W2-DATA-CONTRACTS-DESIGN-EVIDENCE-CONVERGENCE-PLAN.md` | 待审查 |  |
| 准备态 result JSON | `process/checks/CP5-CR139-W2-DATA-CONTRACTS-LLD-BATCH.result.json` | 待审查 |  |
| S33 验收补充 | `process/stories/CR139-S33-catalog-manifest-source-of-truth.md` | 待审查 |  |
| 人工门禁发起消息 | `process/checks/CP5-CR139-W2-DATA-CONTRACTS-HUMAN-GATE-LAUNCH-MESSAGE.md` | 待审查 |  |

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-29T07:10:07+08:00
- 修改意见：按评审意见修正 human-gate checker 可校验性后批准继续推进。
- 风险接受项：接受 DQ-CP5-CR139-W2-DATA-01/02/03 的推荐方案；批准范围仅限进入 `CR139-W2-DATA-CONTRACTS` 设计证据写作。正式 CP5 approval、实现、runtime、真实 lake 写入、provider catalog 写入、published pointer advance execution、物理分区迁移执行、凭据读取、NAS/QMT/trading 和 Git remote write 均未授权。

> 说明：若本准备 Brief 被 approved，只表示允许进入本批次设计证据写作；不表示正式 CP5 设计证据已经通过，也不授权任何实现或 runtime 操作。
