---
checkpoint_id: "CP5-CR152-ML-STRATEGY-E2E-LLD-BATCH"
checkpoint_name: "CR152 ML Strategy E2E LLD Batch Review"
type: "batch_auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-02T10:37:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-02T11:24:55+08:00"
auto_check_result: "process/checks/CP5-CR152-ML-STRATEGY-E2E-LLD-BATCH.result.json"
target:
  phase: "story-planning"
  change_id: "CR-152"
  artifacts:
    - "process/stories/CR152-S01-pit-feature-label-contracts-LLD.md"
    - "process/stories/CR152-S02-purged-embargo-cv-fixture-contract-LLD.md"
    - "process/stories/CR152-S03-training-model-prediction-metadata-LLD.md"
    - "process/stories/CR152-S04-ml-admission-gate-adapter-LLD.md"
    - "process/stories/CR152-S05-static-evidence-release-wording.md"
---

# CP5 CR152 ML Strategy E2E LLD Batch Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP4-CR152-STORY-DAG-PARALLEL-SAFETY.result.json` | PASS | 0 | 5 Story / 5 Wave / DAG / file owner / no-registry-write boundary 已通过。 |
| `process/checks/CP5-CR152-ML-STRATEGY-E2E-LLD-BATCH.result.json` | PASS | 0 | 4 份 full LLD + 1 份 technical-note ready-for-review；blocking clarification=0。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR152 的 5 个 Story 设计证据可作为后续本地/static/fixture 实现输入。 |
| 推荐动作 | `approve`；S01-S04 full LLD、S05 technical-note、CP4 result、CP5 result 和 context 均已生成，阻断项 0。 |
| approve 后会发生什么 | CR152 可进入受控 story-execution：先实现 PIT/label contracts，再实现 purged/embargo CV fixture，再实现 training/model/prediction metadata，再实现 ML gate adapter，最后收口 static-only evidence wording。 |
| approve 不授权什么 | 不授权真实训练、真实数据验证、model registry/store/catalog 写入、lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote 操作。 |
| 不确认会阻塞什么 | 阻塞 CR152 进入实现；ML Strategy E2E first-wave foundation 无法进入 CP6。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR152-ML-STRATEGY-E2E-CONTEXT.yaml` |
| capsule 状态 | ready-for-design-evidence |
| read_profile | compact |
| 默认读取策略 | 先读 CP5 context、CP5 result、CP4 result、Story backlog、Story status 和 Development Plan；仅审查具体 Story 时展开对应 LLD / technical-note。 |
| 全文档读取扩展 | S01-S04 full LLD 已生成；S05 为 technical-note，直接在 Story 卡内审查。 |
| 缺失 / waived 理由 | N/A；CP5 context、CP4 result、CP5 result、4 份 full LLD、1 份 technical-note 均已生成。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | 进入下方待人工决策清单 |
| 高风险策略确认 | 1 | DQ-CP5-CR152-004 明确 approve 不授权真实操作 |
| agent 默认处理 | 2 | CP6 源码实现时读取现有源码后选择最小兼容 placement；若 adapter 字段映射歧义出现则回退设计 |
| 仅审计记录 | 3 | CP4 PASS、CP5 auto PASS、host-orchestrator inline-fallback for CP5 design evidence |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` | scanned | 0 | 0 | 无旧 pending human decision。 |
| CP4 自动预检 | `process/checks/CP4-CR152-STORY-DAG-PARALLEL-SAFETY.result.json` | scanned | 2 | 2 | implementation sequence、CP5 attention items。 |
| CP5 自动预检 | `process/checks/CP5-CR152-ML-STRATEGY-E2E-LLD-BATCH.result.json` | scanned | 4 | 4 | LLD batch、sequence、triple_barrier enforcement、security boundary。 |
| Story LLD / technical-note | `process/stories/CR152-*` | scanned | 2 | 1 | triple_barrier resolved as decision item；inline-fallback is audit record unless user requests meta-se dispatch。 |
| HLD / ADR | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md`、`process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | scanned | 0 | 0 | CP3 已 approved。 |

### 待人工决策清单

本轮待人工决策项：4。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR152-001 | implementation | 是否确认 S01-S04 full LLD + S05 technical-note 作为后续实现输入？ | approve 全批次设计证据，进入本地/static/fixture 实现。 | A. 修改指定 Story LLD；B. 拆成 S01/S02 与 S03/S04/S05 两个 CP5 子批次。 | 推荐方案保持 first-wave 链路一致；修改可降低局部风险但延迟；拆批降低单次审查但增加跨契约漂移。 | 影响 dev-ready 队列、Wave 调度和 CP6/CP7 验证入口。 | 若用户指出具体 Story 或章节问题，回退对应设计证据；若批次过大，拆 CP5 子批次。 |
| DQ-CP5-CR152-002 | implementation | 是否接受当前实现顺序和文件 owner？ | 接受 S01 -> S02 -> S03 -> S04 -> S05；S01/S02 共享 research production contracts 串行，S03 metadata anchors，S04 adapter，S05 wording。 | A. 合并 S01/S02；B. 先做 S01/S02，S03-S05 后置。 | 推荐方案降低共享文件冲突并保持 E2E 闭环；合并增大单 Story；后置会延迟 admission chain。 | 影响 implementation wave、merge owner 和回归范围。 | 若 CP6 发现共享文件冲突，保持严格串行；若 linkage 风险变大，S04 另起 follow-up。 |
| DQ-CP5-CR152-003 | implementation | 是否接受 active `triple_barrier` / `meta_label` first-wave exact enforcement？ | 选中 `triple_barrier` 或 `meta_label` 时返回 `BLOCKED`，因为算法仅预留 slot，未实现。 | A. 返回 `NEEDS_REVIEW`；B. 本轮实现 triple-barrier 算法；C. 移除扩展 slots。 | 推荐方案消除 HLD 模糊且不扩大范围；A 会留下灰区；B 扩大 first wave；C 破坏未来 ML-3 收敛。 | 影响 S01 validator、S04 aggregate status 和 CP7 release wording。 | Future CR 实现算法后可切换为 supported。 |
| DQ-CP5-CR152-004 | security | CP5 approve 是否授权任何真实训练、真实数据、registry/store/catalog 写入或 runtime 操作？ | 不授权；仅授权本地/static/fixture 源码实现和测试。 | A. 另起 runtime/storage authorization gate；B. 暂停 CR152 等真实数据验证。 | 推荐方案与 CP2/CP3 一致，能推进框架补齐；runtime/storage gate 风险高且非本轮目标；暂停会阻塞 first wave。 | 影响 CP6/CP7 验证模式和 release wording。 | 任一真实操作前必须另起 gate，限定 action scope、运行窗口、脱敏、回滚和审计。 |

| 字段 | 内容 |
|---|---|
| 用户需决策事项 | DQ-CP5-CR152-001、DQ-CP5-CR152-002、DQ-CP5-CR152-003、DQ-CP5-CR152-004 |
| 推荐决策 | `approve`；接受 4 项推荐方案，进入本地/static/fixture CP6 implementation。 |
| 备选方案 | `修改: <具体修改点>` 指明决策 ID 和修改内容；或 `reject` 退回 CP5 设计证据。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、安全权限和交付节奏。 |
| 风险与回退 | 若实现时发现 source contract 或 adapter 字段映射冲突，回退对应 Story LLD 或调度 meta-se；任一真实操作必须另起 runtime/storage authorization gate。 |

### CP6 执行关注项

| 关注项 | CP6 要求 | 阻断 / 回退条件 |
|---|---|---|
| S03 `research_production_contracts.py` bridge | S03 默认只读检查该文件；优先在 `engine/training_snapshot_contract.py` / `engine/research_manifest.py` 放置 metadata companion。 | 若确需修改 `engine/research_production_contracts.py`，先停止并记录 CP5 / merge-owner 确认。 |
| S04 `strategy_admission_package.py` linkage | 修改前必须源码检查 `StrategyAdmissionPackage` 是否能安全扩展 `gate_present` / `gate_required` / `gate_status` / `gate_ref` / `blocked_reasons`。 | 若安全扩展不明显，停止实现并 reopen design 或 dispatch meta-se。 |
| S05 feature refs | `docs/features/factor-research-loop/TEST-PLAN.md` / `TASKS.md` 只作为 CP7/CP8 文案对齐参考。 | 若 S05 文案与 Feature TEST-PLAN 冲突，以 CR152 fixture-only / no-real-op 边界为准回退文案。 |

### 不授权项

如果你回复 `approve`，表示你接受 CR152 的设计证据并允许后续本地/static/fixture 实现，不表示授权以下操作：

| 不授权项 | 当前状态 |
|---|---|
| `.env`、token、secret、账号、session、credential 读取 | not-authorized |
| 真实训练、真实数据验证、模型调参 | not-authorized |
| model registry write / publish / promote / upload / set_current | not-authorized |
| feature store / label store / model store / prediction store write | not-authorized |
| catalog pointer mutation、Git remote write | not-authorized |
| 真实 lake / NAS 读写、sync、chmod/chgrp、metadata normalization | not-authorized |
| provider fetch | not-authorized |
| QMT / MiniQMT / xtquant runtime、simulation、paper、live、trading | not-authorized |
| broker/account/market/order/fill 查询、submit/cancel/buy/sell | not-authorized |
| external framework clone/install/run | not-authorized |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP4 自动预检通过 | pending_review | `process/checks/CP4-CR152-STORY-DAG-PARALLEL-SAFETY.result.json` | PASS，阻断项 0 |
| 全部目标 Story 设计证据已生成 | pending_review | 4 份 LLD + 1 份 technical-note | ready-for-review |
| CP5 自动预检通过 | pending_review | `process/checks/CP5-CR152-ML-STRATEGY-E2E-LLD-BATCH.result.json` | PASS，阻断项 0 |
| CP5 context capsule 已生成 | pending_review | `process/context/CP5-CR152-ML-STRATEGY-E2E-CONTEXT.yaml` | ready |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | 设计证据覆盖 Story AC | pending_review | LLD / technical-note §2 / §10 / §14 |  |
| 2 | 与 HLD / ADR / Feature Design 一致 | pending_review | LLD §0、Feature Matrix |  |
| 3 | 文件影响范围明确 | pending_review | LLD §4 / §11、Development Plan |  |
| 4 | 接口契约完整 | pending_review | LLD §6 |  |
| 5 | 数据结构明确 | pending_review | LLD §5 |  |
| 6 | 控制流 / 失败路径明确 | pending_review | LLD §7 / §8 / §12、S05 technical-note |  |
| 7 | `triple_barrier` first-wave enforcement 明确 | pending_review | S01 LLD §5、S04 LLD §8 |  |
| 8 | metadata vs registry write 边界明确 | pending_review | S03 LLD §8 |  |
| 9 | adapter / inline-fallback 条件明确 | pending_review | S04 LLD §8 / §12 |  |
| 10 | 安全设计明确 | pending_review | 不授权项、LLD §9 |  |
| 11 | 可测试性明确 | pending_review | LLD §10、Feature TEST-PLAN |  |
| 12 | dev_gate 可计算 | pending_review | Story cards、CP5 result |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 全部自动预检 PASS | pending_review | `process/checks/CP5-CR152-*` | blockers=0 |
| 用户明确 approve / 修改 / reject | pending_review | 当前对话 |  |
| CP5 approve 不授权真实操作 | pending_review | 不授权项 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| S01 LLD | `process/stories/CR152-S01-pit-feature-label-contracts-LLD.md` | pending_review |  |
| S02 LLD | `process/stories/CR152-S02-purged-embargo-cv-fixture-contract-LLD.md` | pending_review |  |
| S03 LLD | `process/stories/CR152-S03-training-model-prediction-metadata-LLD.md` | pending_review |  |
| S04 LLD | `process/stories/CR152-S04-ml-admission-gate-adapter-LLD.md` | pending_review |  |
| S05 technical-note | `process/stories/CR152-S05-static-evidence-release-wording.md#技术说明` | pending_review |  |
| CP5 context | `process/context/CP5-CR152-ML-STRATEGY-E2E-CONTEXT.yaml` | pending_review |  |
| CP5 batch result | `process/checks/CP5-CR152-ML-STRATEGY-E2E-LLD-BATCH.result.json` | pending_review |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-02T11:24:55+08:00
- 已接受决策项：`DQ-CP5-CR152-001`、`DQ-CP5-CR152-002`、`DQ-CP5-CR152-003`、`DQ-CP5-CR152-004`
- 修改意见：N/A
- 风险接受项：CP5 approve 仅授权本地/static/fixture 源码实现和测试；真实训练、真实数据验证、registry/store/catalog 写入、runtime、lake/NAS/provider/QMT/broker/credential/external framework/Git remote 继续不授权。
