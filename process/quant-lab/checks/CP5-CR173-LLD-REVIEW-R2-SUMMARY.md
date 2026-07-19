---
artifact: "CP5-CR173-LLD-REVIEW-R2-SUMMARY"
round: 2
status: final
decision: revise
blocking_count: 0
required_count: 2
optional_count: 1
cr_id: "CR-173"
review_scope: "design-only-remediation-recheck"
---

# Review Summary

## 1. 输入清单

- findings_files:
  - `process/checks/CP5-CR173-LLD-REVIEW-R2-FINDINGS.md`
- prior_review:
  - `process/checks/CP5-CR173-LLD-REVIEW-FINDINGS.md`
  - `process/checks/CP5-CR173-LLD-REVIEW-SUMMARY.md`
- remediation_return:
  - `process/handoffs/CR173-CP5-META-SE-DESIGN-REMEDIATION-RETURN-SUMMARY.md`
- authoritative_baseline:
  - `docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md` v1.1
  - `docs/design/DOMAIN-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md` v1.1
  - `docs/design/ARCHITECTURE-DECISION-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md` v1.1
  - `docs/features/effective-trial-offline-estimator/DESIGN.md` v0.2
  - `docs/features/effective-trial-offline-estimator/TEST-PLAN.md` v0.2
  - `docs/features/effective-trial-offline-estimator/TASKS.md` v0.2
- story_and_lld:
  - 三张 CR173 Story 卡
  - S01/S02/S03 LLD v1.2

## 2. 严重度汇总

| Severity | Count | Owner |
|----------|-------|-------|
| 严重 | 0 | `host-orchestrator` |
| 一般 | 2 | `meta-dev/S03 owner + QA/verification owner` |
| 轻微 | 1 | `authority design/Story owner` |

## 3. 决策

- decision: `revise`
- rationale: 权威整改已正确关闭 F03/F04 和 public-call lane 根因，并把 stable computation ref、七项 attempt basis、external audit owner/linkage/N/A lifecycle写入 HLD/Domain/ADR/Feature/S01；但 S03 LLD v1.2 仍使用与权威合同不同的四组件 presence bitmap，并把含 actual outcome/failure ID 的完整 basis 放在 estimator 之前构造。另有“10 类 forbidden counter”与九项实际枚举的量化不一致。required_count=2，因此本轮不能 proceed。
- next_checkpoint: `no CP5; revise R2 required findings and perform another design-only independent recheck`

### CP3 Advisor Summary（适用时填写）

> 本轮为 CP5 remediation recheck，不是 CP3 advisor formation；下表仅保留模板结构。

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| N/A | N/A | N/A | CP5 design-only remediation recheck | N/A | N/A |

| 输入类型 | 来源 lane | 进入方案形成 | 进入 HLD 后评审 | 处理结果 |
|---|---|---|---|---|
| review_findings | lane-quality / meta-qa-critical | no | yes | revise |

## 4. Decision Brief 输入

| 字段 | 内容 |
|---|---|
| 推荐决策 | `修改`。先修正 S03 的三位 presence bitmap 与 outcome 后置 basis 流程，再修正 authorization operation-class cardinality；required/blocking 归零前不发起 CP5。 |
| 备选方案 | 主选：只修 S03 LLD/Story/测试设计的 basis 消费与计数清单，保持 HLD/Domain/ADR/Feature 已冻结的 stable identity + external audit 方案。治理备选：若无法在当前轮消除 schema/计数歧义，则保持 estimator typed_unavailable并暂停 CP5。不得回退为逐执行 computation ref、float/tolerance、raw count 或跳过 12/12 regression。 |
| 影响维度 | canonical basis/hash interoperability、F01-F08 oracle独立性、S03 verification可信度、authorization counter可计算性、CP5审计新鲜度。 |
| 优劣分析 | 主选改动小，不影响公式、七字段、公有边界或 Story DAG；后置 basis 可避免把 expected outcome 预灌给被测对象。计数清单修正只澄清已有 deny-default controls。暂停方案安全但延后交付。 |
| 风险与回退 | 修正前的主要风险是 S01/S03 computation ref不一致、自证 oracle与 CP7 9/10 或重复计数。回退点仍是 standalone estimator unavailable，current public C1/Gate1/DSR/admission 不变；任何真实/runtime/public write继续为0。 |
| 用户需决策事项 | 无需用户扩大范围或授权；这是设计内部一致性返工。下一次 review required=0 后，Host 才可准备 CP5 Decision Brief。 |

## 5. 后续动作

1. meta-dev/S03 owner处理 F-CR173-CP5-R2-001，严格复用三位 bitmap，并将 basis finalize 移到 actual outcome 产生之后。
2. QA/verification owner处理 F-CR173-CP5-R2-002，形成唯一 operation-class inventory及与 public 六计数的非重复映射。
3. authority design/Story owner处理 optional freshness，刷新 v1.2 resync 与 R2 状态，同时保持 `confirmed=false` 和实现锁定。
4. 重新执行 design-only 独立审查和 review-artifact validator；只有 blocking=0 且 required=0 才可 `proceed`。
5. 不运行 native/public tests、不实现任何资产、不修改 STATE/checkpoint/ledger、不发起 CP5，直至人工门禁由 Host 正式准备。

