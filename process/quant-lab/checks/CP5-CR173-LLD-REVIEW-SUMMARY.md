---
artifact: "CP5-CR173-LLD-REVIEW-SUMMARY"
round: 1
status: final
decision: revise
blocking_count: 0
required_count: 3
optional_count: 1
cr_id: "CR-173"
review_scope: "design-only"
---

# Review Summary

## 1. 输入清单

- findings_files:
  - `process/checks/CP5-CR173-LLD-REVIEW-FINDINGS.md`
- reviewed_design_evidence:
  - `process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md`
  - `process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md`
  - `process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md`
- baseline_and_trace_inputs:
  - `process/context/CP5-CR173.context.json`
  - `process/checks/CP4-CR173-STORY-DAG-PARALLEL-SAFETY.result.json`
  - `docs/features/effective-trial-offline-estimator/DESIGN.md`
  - `docs/features/effective-trial-offline-estimator/TEST-PLAN.md`
  - `docs/features/effective-trial-offline-estimator/TASKS.md`
  - `docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md`
  - `docs/design/DOMAIN-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md`
  - `docs/design/ARCHITECTURE-DECISION-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md`
  - `docs/design/DEPENDENCY-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md`

## 2. 严重度汇总

| Severity | Count | Owner |
|----------|-------|-------|
| 严重 | 0 | `host-orchestrator` |
| 一般 | 3 | `architecture/methodology owner + contract/numeric owner + QA/verification owner` |
| 轻微 | 1 | `Story/Feature owner` |

## 3. 决策

- decision: `revise`
- rationale: exact PSD 算法、两项重点 oracle、participation-ratio、双 invariant、一次 half-even、8+12 精确 inventory、文件所有权和 estimator-only deny-default 大框架均可实现；但 `effective_trial_computation_ref` 被 LLD 从权威基线的 append-only computation/attempt ref 重解释为 stable identity，并新增外置 attempt audit，尚未回写 HLD/Domain Map/ADR/Feature。另有 nonfinite 的 F03/F04 精确 reason 冲突，以及 public-call zero counter 与 12/12 read-only regression 调用的作用域冲突。三项均直接影响可实现、可验证的 CP5 合同，因此当前不得 proceed。
- next_checkpoint: `CP5 only after revised design evidence is independently rechecked and required_count=0`

### CP3 Advisor Summary（适用时填写）

> 本轮为 CP5 post-design review，不是 CP3 advisor formation；下表仅保留模板结构。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| N/A | N/A | N/A | CP5 design-only review | N/A | N/A |

| 输入类型 | 来源 lane | 进入方案形成 | 进入 HLD 后评审 | 处理结果 |
|---|---|---|---|---|
| review_findings | lane-quality / meta-qa-critical | no | yes | revise |

## 4. Decision Brief 输入

| 字段 | 内容 |
|---|---|
| 推荐决策 | `修改`。先返工 F-001..003，再由独立 review 确认 required=0；当前不发起 CP5。 |
| 备选方案 | 主选：保留 stable content-addressed `effective_trial_computation_ref`，把逐执行 append-only audit 显式建模为外置对象并回写全部权威设计。备选：第七字段逐执行唯一，但需同步修改 HLD 的 3/3 computation-ref/evidence-hash 稳定条件、canonical hash 合同与测试 oracle，影响更大。治理备选：若 owner 暂不能决定，保持 estimator typed_unavailable 并暂停 CP5。 |
| 影响维度 | identity/schema、canonical hash、failure recovery、F03/F04 reason、CP7 test instrumentation、public boundary、未来 persistence/projection migration、Story 生命周期审计。 |
| 优劣分析 | 主选两层 identity/audit 兼顾 deterministic result 与逐执行审计，改动集中在设计回写和明确 owner；备选单一逐执行 ref 更接近 Domain Map 字面，但会破坏七字段 canonical evidence 3/3 唯一，迁移面更大；暂停方案风险最低但延后交付。 |
| 风险与回退 | 不修正会产生 silent contract drift、reason/hash 不稳定或 CP7 必然假失败。回退点是保持 CR173 standalone estimator unavailable，public C1/Gate1/DSR/admission 继续现状；不得以 raw count、float/tolerance、修改 existing expected 或 public adapter 降级。 |
| 用户需决策事项 | 本轮无需用户扩大授权；需由设计 owner先完成三项 required 修订。若两层 identity/audit 被采纳，CP5 Decision Brief 应明确这是对已批准基线的显式澄清并列出回写文档。 |

## 5. 后续动作

1. architecture/methodology owner 处理 F-CR173-CP5-001：统一 HLD、Domain Map、ADR、Feature、S01/S03 的 identity/audit 语义与任务映射。
2. contract/numeric owner 处理 F-CR173-CP5-002：冻结 nonfinite 的唯一可达 failure reason，并同步 golden/failure tables 与 method hash descriptor。
3. QA/verification owner 处理 F-CR173-CP5-003：拆分 CR173 integration-call zero 与 12/12 read-only regression invocation 计数。
4. Story/Feature owner处理 optional freshness，刷新 `planned-pending-cp4`/“LLD=0”历史陈述。
5. 仅在上述 required 修订完成后重新做 design-only 独立审查；validator 通过且 required/blocking 均为 0 时，Host Orchestrator 才可准备 CP5。

