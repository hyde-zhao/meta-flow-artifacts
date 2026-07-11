---
checkpoint_id: "CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-HLD-REVIEW"
checkpoint_name: "CR163 Trial Lineage Instrumentation HLD Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-11T11:45:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-11T11:18:50+08:00"
auto_check_result: "process/checks/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-HLD-CONSISTENCY.result.json"
decision_brief_profile: "compact"
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "docs/design/BLUEPRINT-TRIAL-LINEAGE-INSTRUMENTATION.md"
    - "docs/design/DOMAIN-MAP-TRIAL-LINEAGE-INSTRUMENTATION.md"
    - "docs/design/DEPENDENCY-MAP-TRIAL-LINEAGE-INSTRUMENTATION.md"
    - "docs/design/HLD-TRIAL-LINEAGE-INSTRUMENTATION.md"
    - "docs/design/ARCHITECTURE-DECISION-TRIAL-LINEAGE-INSTRUMENTATION.md"
---

# CP3 CR163 Trial Lineage Instrumentation HLD 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-HLD-CONSISTENCY.result.json` | PASS | 0 | Blueprint/Domain/Dependency/HLD/ADR、8/8 requirements、12/12 scenarios、5/5 Story outcomes 一致；waiver=0。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 冻结 CR163 的公共对象、生命周期 API、append-only 事件、canonical seal/supersession、两条 producer chain 接入和 existing admission consumer 边界，使后续五 Story 可进入正式规划和全量 LLD。 |
| 推荐动作 | `approve`：接受 session façade + typed commands、local canonical JSON/JSONL、immutable SHA-256 seal、manual-count reconciliation-only 和五 Story CP4 输入。 |
| approve 后会发生什么 | 主编排器继续使用同一 `meta-se` 角色完成 CP4 正式 Story/DAG/文件 owner/Feature design matrix；CP4 自动 PASS 后调度 meta-dev 产出全部五 Story 设计证据，并在 CP5 required human gate 停止。 |
| approve 不授权什么 | 不授权代码/测试实现、真实 lake/NAS/provider/credential、simulation/paper/live/broker/trading、外部 registry/framework、Git remote write、publish/release；不授权 effective-trial 或其他统计计算、CR155 backfill。 |
| 不确认会阻塞什么 | CP4 Story 规划、CP5 全量设计证据、CP6 实现与 CP7 独立验证均保持阻断。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅展开已批准的产品基线、CR161 parent design baseline 和命中的 producer/consumer contracts。 |
| 全文档读取扩展 | meta-se 在 discussion checkpoint 记录三组 architecture-traceability/parent-conflict/public-contract 展开；Host 另登记 1 次 discussion human audit。 |
| 缺失 / waived 理由 | 无；waiver=0。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` | scanned | 0 | 0 | CP2 已 approved；无遗留 scope/authz 阻断。 |
| 委托 Agent 交还摘要 | `process/handoffs/CR163-CP3-META-SE-RETURN-SUMMARY.md` | scanned | 4 | 4 | DQ-CP3-CR163-001..004 纳入本门。 |
| 自动预检结果 | `process/checks/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-HLD-CONSISTENCY.result.json` | scanned | 4 | 4 | 与 return summary 一致。 |
| discussion log / checkpoint | `process/discussions/CP3-CR163-HLD-DISCUSSION-LOG.md` / `process/checks/CP3-CR163-DISCUSSION-CHECKPOINT.json` | scanned | 4 | 4 | 四个 Architecture Gray Areas 已形成推荐方案；无 blocking batch。 |
| 下游正式产物 | Blueprint / Domain / Dependency / HLD / ADR | scanned | 4 | 4 | Decision Register 与本 Brief 一致。 |
| 用户显式选择题 | CP2 approved baseline | scanned | 0 | 0 | 2 chains/4 mappings、count/effective/seal 已是上游确认，不重复询问。 |
| LLD clarification queue | N/A | n/a | 0 | 0 | CP5 前尚未启动 LLD。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | 进入待人工决策清单。 |
| 高风险策略确认 | 1 | DQ-004 确认 design-only 授权边界，不扩大权限。 |
| agent 默认处理 | 3 | exact schema field types、错误码和文件清单留给 Feature design/LLD。 |
| 仅审计记录 | 5 | 8/8 REQ、12/12 scenarios、5 simulations、5 Stories、4/4 mappings。 |

### 推荐架构摘要

```text
Producer orchestration
    -> FamilyLineageSession façade
        -> typed append-only event commands
            -> create-only canonical FamilySpec JSON
            -> append-only Event JSONL
            -> immutable versioned Manifest JSON
            -> independent Validation JSON
                -> validated projection
                    -> existing CR151 / CR154 / admission package consumers
```

核心语义：

- 六个持久化对象：`ExperimentFamilySpec`、`ExperimentTrial`、`TrialAttempt`、`TrialSelection`、`ExperimentFamilyManifest`、`FamilyLineageValidationResult`。
- `FamilyLineageSession` 是应用 façade，不是第七个持久化对象。
- `ExperimentManifest` / `BacktestRunSpec` 仍是单次 run contract，通过 `run_id`、`experiment_id` 和 refs 连接。
- `raw_trial_count=count(distinct stable_trial_id)`；seed/参数不同是 Trial，retry 是 Attempt；failed/cancelled/excluded/带原因 never-started trial 保留并计数。
- canonical JSON 排除 path/mtime/current clock 等 volatile 字段；SHA-256 seal 绑定 immutable version。
- 修正只允许 supersession；full chain 是真相，latest pointer 只能是可重建 cache。
- manual legacy `trial_count` 仅 reconciliation：无 native sealed ref 为 `typed_unavailable`，不一致为 `blocked`，永远不能单独产生 `present`。
- consumer 只读取 validator projection，不创建新 gate，不得改善更差的 admission 状态。
- C1 仅 raw-input-ready；effective count 继续 unavailable；CR155 保持 blocked。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR163-001 | architecture | 是否批准 `FamilyLineageSession` lifecycle façade，内部使用 typed append-only event commands？ | 批准 session + commands；producer 易接入，底层事件仍完整可审计。 | 纯 command/fold API；或 close-time snapshot session。 | 推荐兼顾 ergonomics 与 audit；纯 commands 更显式但增加 producer 复杂度；snapshot 简单但隐藏中途失败并丢事件事实。 | 决定 public producer API、S01/S02 contract 与两条 chain 接入方式。 | 出现跨语言/跨进程 transport 需求时暴露 command DTO；不得退回 snapshot-only。 |
| DQ-CP3-CR163-002 | architecture | 是否批准 repo-local canonical JSON/JSONL、SHA-256 immutable version seal 和 append-only full supersession chain？ | 批准 create-only spec + append-only events + immutable manifest/validation；full chain 为真相。 | 单 snapshot JSON；SQLite；外部 registry/MLflow。 | 推荐最小、确定、易 fixture/审计；snapshot 丢历史；DB/registry 并发更强但引入迁移/服务/依赖。 | 决定 storage、hash、tamper、recovery、rollback 和 S02 范围。 | concurrent writer conflict>0 或规模超过文件模型时另起 storage/migration CR；不得在当前 CR 静默切换。 |
| DQ-CP3-CR163-003 | architecture | 是否批准 manual legacy count 仅作 reconciliation，并只复用现有 CR151/CR154/admission consumers？ | 批准：无 native sealed ref=`typed_unavailable`；mismatch/invalid=`blocked`；consumer 只取 validator projection；不建新 gate。 | 接受手填 count 产生 present；完全忽略 manual count；创建 CR163 平行 gate。 | 推荐消除后验 bypass 并保持现有 policy 单一；接受 manual present 违背核心目标；忽略会失去迁移诊断；新 gate 重复政策。 | 决定 S04 接口、legacy compatibility、fail-closed 和 admission 状态合并。 | 未来 inferred/backfill 需要独立 provenance policy/CR；不能由 CR163 自动启用。 |
| DQ-CP3-CR163-004 | security | 是否批准冻结的六对象、五 Story CP4 输入与 design-only 授权边界？ | 批准 HLD/ADR 并进入 CP4；S01-S05 不变，S03 4/4 mappings；实现仍等 CP5。 | 请求架构修改；暂停 CR163。 | 推荐形成完整 CP4/CP5 输入且不扩大权限；修改可降低后续返工但需重跑 CP3；暂停最安全但阻塞价值。 | 决定是否正式拆 Story/DAG/owner/LLD policy；不会授权源码实现。 | 任何 runtime/data/credential/statistical/external-write 需要立即 BLOCKED 并走独立门禁/CR。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 DQ-CP3-CR163-001..004，并连续推进 CP4、LLD 批次直到 CP5 人工门。 |
| 备选方案 | `修改: <DQ ID + 修改点>` 后退回 meta-se-critical；或 `reject` 回退 CP2/暂停 CR163。 |
| 影响维度 | API ergonomics、审计完整性、确定性、恢复、legacy compatibility、文件 owner、可验证性、安全权限。 |
| 优劣分析 | 推荐方案在不引入数据库/registry/runtime 的情况下完成可审计最小闭环；各备选的切换条件已写入 ADR。 |
| 风险与回退 | CP3 只批准设计；CP4/CP5 前仍无实现；权限扩大立即阻断。 |
| 用户需决策事项 | `DQ-CP3-CR163-001`、`DQ-CP3-CR163-002`、`DQ-CP3-CR163-003`、`DQ-CP3-CR163-004`。 |

### CP3 追加摘要

- 候选架构：A Local Lifecycle Ledger（推荐）；B Pure Event Commands；C SQLite Registry。
- 推荐优化：producer 易接入、事件完整、确定性强、无需新服务。
- 推荐牺牲：当前假设 single writer；并发/大规模时需独立 storage ADR/CR。
- Use Case→Architecture Traceability：8/8 requirements、12/12 scenarios。
- 场景模拟：5 个关键模拟全部 PASS，包括 happy seal/present、retry/failure count、tamper、supersession、CR155 regression。
- 未决风险：无 blocking；并发规模、inferred backfill、真实 ML/event runner 均有 switch condition。
- discussion log：`process/discussions/CP3-CR163-HLD-DISCUSSION-LOG.md`。
- discussion checkpoint：`process/checks/CP3-CR163-DISCUSSION-CHECKPOINT.json`。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 已批准 | 通过 | CP2 checkpoint / gate ledger | CP2 approval 有效。 |
| Blueprint/HLD/ADR 齐备 | 通过 | 五份 design docs | 用户接受其作为 CP4 输入。 |
| CP3 自动预检 PASS | 通过 | CP3 result JSON | 机器预检 PASS、blocker=0、waiver=0。 |
| Blocking DQ=0 | 通过 | Discussion checkpoint | 四项 DQ 均由本次 approve 接受推荐方案。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 批准 session façade + typed commands | 通过 | DQ-CP3-CR163-001 | 接受推荐方案。 |
| 2 | 批准 JSON/JSONL + SHA-256 seal + supersession | 通过 | DQ-CP3-CR163-002 | 接受推荐方案。 |
| 3 | 批准 reconciliation-only + existing consumers | 通过 | DQ-CP3-CR163-003 | 接受推荐方案。 |
| 4 | 批准六对象、五 Story 输入与 design-only 边界 | 通过 | DQ-CP3-CR163-004 | 接受推荐方案；仍不授权实现或 runtime。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| HLD/ADR 可作为 CP4 输入 | 通过 | 本 checkpoint | 允许进入 Story/Feature 规划。 |
| 用户明确批准/修改/拒绝 | 通过 | 人工审查结果 | 用户回复 `approve`。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| Blueprint/Domain/Dependency | `docs/design/*-TRIAL-LINEAGE-INSTRUMENTATION.md` | 通过 | 已批准。 |
| HLD | `docs/design/HLD-TRIAL-LINEAGE-INSTRUMENTATION.md` | 通过 | 已批准。 |
| ADR | `docs/design/ARCHITECTURE-DECISION-TRIAL-LINEAGE-INSTRUMENTATION.md` | 通过 | 已批准四项推荐架构决策。 |
| CP3 result | `process/checks/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-HLD-CONSISTENCY.result.json` | 通过 | PASS。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-11T11:18:50+08:00
- 修改意见：无；接受 `DQ-CP3-CR163-001`、`DQ-CP3-CR163-002`、`DQ-CP3-CR163-003`、`DQ-CP3-CR163-004` 的推荐方案，并按 route plan 自动推进至 CP5 人工门禁。
- 风险接受项：无；本 CP3 不接受或授权 runtime/data/credential/statistical/external-write 风险。
