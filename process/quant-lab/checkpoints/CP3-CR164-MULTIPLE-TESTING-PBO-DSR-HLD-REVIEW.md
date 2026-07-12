---
checkpoint_id: "CP3-CR164-MULTIPLE-TESTING-PBO-DSR-HLD-REVIEW"
checkpoint_name: "CR164 Multiple-Testing PBO DSR HLD Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-12T20:12:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-12T20:20:00+08:00"
auto_check_result: "process/checks/CP3-CR164-MULTIPLE-TESTING-PBO-DSR-HLD-CONSISTENCY.result.json"
decision_brief_profile: "compact"
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "docs/design/BLUEPRINT-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md"
    - "docs/design/DOMAIN-MAP-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md"
    - "docs/design/DEPENDENCY-MAP-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md"
    - "docs/design/HLD-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md"
    - "docs/design/ARCHITECTURE-DECISION-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md"
---

# CP3 CR164 Multiple-Testing / PBO / DSR HLD 人工审查

## 自动预检摘要

| 预检 | 结论 | 阻断 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR164-MULTIPLE-TESTING-PBO-DSR-HLD-CONSISTENCY.result.json` | PASS | 0 | 12/12 checks PASS，9/9 requirements、13/13 scenarios、10/10 QAC，waiver=0。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 将 CR163 sealed lineage 与 CR161 typed evidence contract 连接成 BH/WRC-SPA/PBO-CSCV/DSR 可计算 evidence pipeline，同时保持 fail-closed 与 claim ceiling。 |
| 推荐动作 | `approve`：接受四项架构决策，继续以内联方式推进 CP4 与全量设计证据准备，直到 CP5 人工门禁。 |
| approve 后会发生什么 | Host Orchestrator 不拉起子 Agent，按五个 outcome 输入完成正式 Story/DAG/owner/Feature design/LLD 准备；CP4 自动检查通过后在 CP5 停止。 |
| approve 不授权什么 | 不授权子 Agent、源代码/测试实现、真实统计/研究运行、生产数据/凭据/NAS/provider/external framework、broker/trading、deploy/publish/Git remote write。 |
| 不确认会阻塞什么 | CP4 Story planning、CP5 全量设计、CP6 实现与 CP7 独立验证。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR164-MULTIPLE-TESTING-PBO-DSR-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | capsule-first；仅在 source contract 事实不足时用 `rg` 窄读命中的五个模块。 |
| 全文档读取 | 0 次批量全文扩展；仅按命中位置读取 CR164 product sections 与五个 source contracts。 |
| 缺失 / waived 理由 | 无；waiver=0。 |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---:|---:|---:|---|
| STATE / CP2 gate | scanned | 4 | 0 | CP2 DQ 4/4 已批准，不重复询问。 |
| CP3 discussion/checkpoint | scanned | 4 | 4 | architecture/security；四个 AGA 收敛为 DQ-001..004。 |
| CP3 result | scanned | 4 | 4 | architecture/security；与 discussion 和 ADR 一致。 |
| Blueprint/Domain/Dependency/HLD/ADR | scanned | 4 | 4 | architecture/security；Decision Register 一致。 |
| LLD clarification queue | n/a | 0 | 0 | CP5 前尚未启动。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | 进入下方待人工决策清单。 |
| 高风险策略确认 | 1 | DQ-004 保持 no-subagent/design-only 权限边界。 |
| agent 默认处理 | 3 | 精确 field types、repo paths、golden fixture values 留给 CP4/CP5，不能改变本 HLD。 |
| 仅审计记录 | 6 | 9/9 requirements、13/13 scenarios、10/10 QAC、6 simulations、5 Stories、4 Waves。 |

### 推荐架构

```text
CR163 sealed lineage + explicit method inputs
  -> input normalizer/validator
    -> BH | WRC/SPA | PBO/CSCV | DSR(raw-count)
      -> typed MethodEvidence refs
        -> conservative aggregator
          -> existing CR151 / CR154 / admission-package projections
```

核心冻结项：

- 统一 method-neutral input/evidence envelope；四个 pure calculator；aggregator 不重算。
- WRC/SPA 使用 stationary bootstrap，MVP `block_length_mode=fixed_window`；window、seed、replications、benchmark/null 与 hashes 全量入 provenance。
- DSR 强制 `dsr_input_method=raw_trial_count`；effective count/ref/method 继续 typed_unavailable/空，raw 不得 alias effective。
- 聚合表固定 `BLOCKED > FAIL > TYPED_UNAVAILABLE > PASS`；仅 mandatory set 全 PASS 才能 claim PASS。
- 只复用既有 consumers；UC-58 实现，UC-59/60 compatibility-only；不创建平行 gate。
- 五 Story、四 Wave：S01；S02/S03 并行；S04；S05。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR164-001 | architecture | 是否批准统一 envelope、四 pure calculators、sealed identity binding 与 deterministic artifact contract？ | 批准 ADR-001/002/007。 | 单体 calculator；dynamic plugin/store。 | 推荐最可测且无 runtime discovery；单体更少文件但耦合；plugin 扩展强但治理重。 | 决定 S01-S03 module/schema/test boundary。 | 方法生态显著扩大时另起 ADR 评审 plugin。 |
| DQ-CP3-CR164-002 | architecture | 是否批准 fixed-window stationary bootstrap 与 raw-count DSR non-alias？ | 批准 ADR-003/004。 | 经独立验证的 automatic selector；暂停相应方法。 | 固定 window 复跑最明确；automatic 易用但必须冻结算法/version/fixtures。 | 决定 WRC/SPA 与 DSR 统计 provenance；raw→effective 会 overclaim。 | automatic 通过独立设计/验证后切换；不能静默 alias。 |
| DQ-CP3-CR164-003 | architecture | 是否批准 severity lattice 与 existing-consumer-only 集成？ | 批准 ADR-005/006。 | 回 CP2 改 mandatory scope；暂停 projection。 | 推荐严格 no-OR-pass；多数票/OR 提高表面可用率但会制造 false admission。 | 决定 final claim 和三 consumer behavior。 | mandatory scope 变化必须回 CP2；不能在 CP4 放宽。 |
| DQ-CP3-CR164-004 | security | 是否批准五 Story CP4 输入并保持 no-subagent/design-only？ | 批准 ADR-008，继续 inline。 | 修改设计；暂停 CR164。 | 推荐完成下游设计且不扩大权限；暂停风险最低但阻塞价值。 | 决定能否进入 CP4/CP5；不授权实现。 | 任何 runtime/data/external/write 需求立即 BLOCKED。 |

| 字段 | 内容 |
|---|---|
| 用户需决策事项 | `DQ-CP3-CR164-001`、`DQ-CP3-CR164-002`、`DQ-CP3-CR164-003`、`DQ-CP3-CR164-004`。 |

### 授权说明

如果你回复 `approve`，表示批准 DQ-CP3-CR164-001..004 的推荐架构，并允许继续以内联方式准备 CP4/CP5；不表示授权源码/测试实现或真实运行。不授权项包括子 Agent、真实数据、凭据、NAS/provider、外部 framework、broker/trading、publish 和 Git remote write。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP2 approved | 通过 | CP2 checkpoint + gate ledger |
| CP3 capsule ready | 通过 | CP3 context |
| Design package complete | 通过 | Blueprint/Domain/Dependency/HLD/ADR |
| CP3 result PASS | 通过 | CP3 result JSON |

## Checklist

| # | 检查项 | 审查结果 | 证据 |
|---|---|---|---|
| 1 | 批准 envelope / calculators / sealed binding / deterministic contract | 通过 | DQ-001 |
| 2 | 批准 fixed-window WRC/SPA 与 raw-count DSR non-alias | 通过 | DQ-002 |
| 3 | 批准 severity lattice 与 existing consumers only | 通过 | DQ-003 |
| 4 | 批准五 Story 输入与 no-subagent/design-only 边界 | 通过 | DQ-004 |

## Exit Criteria

| 条目 | 审查结果 | 说明 |
|---|---|---|
| HLD/ADR 可作为 CP4 输入 | 通过 | 允许 Story planning，不允许实现。 |
| 用户明确 approve / 修改 / reject | 通过 | 用户回复 `approve`。 |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| Blueprint/Domain/Dependency | `docs/design/*-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md` | 待审查 |
| HLD | `docs/design/HLD-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md` | 待审查 |
| ADR | `docs/design/ARCHITECTURE-DECISION-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.md` | 待审查 |
| CP3 result | `process/checks/CP3-CR164-MULTIPLE-TESTING-PBO-DSR-HLD-CONSISTENCY.result.json` | PASS / 待人工确认 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-12T20:20:00+08:00
- 修改意见：接受 `DQ-CP3-CR164-001..004`；继续 no-subagent inline 推进至 CP5。
- 风险接受项：无；本 CP3 不接受或授权 runtime/data/credential/external-write 风险。
