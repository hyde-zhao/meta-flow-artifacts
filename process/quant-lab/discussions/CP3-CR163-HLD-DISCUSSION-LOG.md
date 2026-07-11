---
status: completed-for-draft
cr_id: "CR-163"
checkpoint: "CP3"
owner: "meta-se-critical"
created_at: "2026-07-11T11:35:00+08:00"
formal_gate_opened: false
---

# CP3 CR163 Architecture Gray Areas and Advisor Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-11 | meta-se-critical | 根据 CP2 confirmed decisions、产品基线、CR161 baseline 与现有源码契约形成 HLD 方案输入。 |

## Context and Interaction Boundary

- Capsule-first：`process/context/CP3-CR163-TRIAL-LINEAGE-INSTRUMENTATION-DESIGN-CONTEXT.yaml`。
- 用户问题权限：本 agent 不直接询问；阻断问题须批量交 host。本轮无阻断 DQ。
- 讨论性质：方案形成输入，不是 CP3 人工批准；formal gate 未开启。
- Reviewer lanes：未额外调度，不声称虚构的 lane 结论；以下 advisor 视角由 meta-se-critical 基于已批准基线综合。

## Architecture Gray Areas

| ID | Source | Question | Why architecture-changing | Status |
|---|---|---|---|---|
| AGA-CR163-01 | REQ-001/002, producer facts | event command 或 lifecycle session | 改变 public API、producer coupling、漏记风险 | selected: hybrid |
| AGA-CR163-02 | REQ-002/004/007, local-only constraint | snapshot / JSONL / DB | 改变 append-only proof、并发、migration | selected: JSON/JSONL |
| AGA-CR163-03 | SGQ-004, NFR-002/004 | canonical seal/supersession pointer owner | 改变跨运行确定性、tamper/recovery | selected: full chain truth |
| AGA-CR163-04 | existing manual trial_count consumers | legacy count fail-closed 或兼容 present | 决定是否仍保留后验绕过 | selected: reconciliation-only |

## Table-first Advisor Input

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Session façade + internal event commands | producer 易用、状态前置校验、逐事件审计 | 维护 façade+schema | API/modules/2 chains/validation | recommended | local Python producers；跨语言时公开 command transport |
| Pure event commands | 最透明、易重放 | producer 负担高、易漏顺序 | producer/maintenance/testing | conditional | producer 多样或消息 transport 出现 |
| Snapshot session | 最少调用 | 无法证明 retry/failure append-only | integrity/audit | reject | 不满足 CR163 |
| Canonical JSON/JSONL | 无服务依赖、diff/fixture 友好 | concurrency 有限 | storage/security/rollback | recommended | single writer/local scope |
| SQLite/registry | transaction/query/concurrency | migration/runtime/ops 成本高 | deploy/security/maintenance | defer | concurrent writers 或规模阈值触发新 CR |
| Legacy count reconciliation-only | 消除 bypass、保留冲突诊断 | 历史路径仍 unavailable | integration/risk | recommended | native sealed lineage 是 present 唯一来源 |
| Accept manual count as present | compatibility 最宽 | 直接破坏 pre-search/audit 目标 | admission/security | reject | 不切换；需独立 inferred-backfill policy |

## Advisor Perspective Summary

| Perspective | Formation input | Adopted result |
|---|---|---|
| product | 未来 native run 可合法 present，但 C1仍 non-computable | raw-lineage-only projection；effective unavailable |
| architecture | unique truth owner、no parallel gate、single-run separation | lineage core + adapters；refs link ExperimentManifest |
| quality | 12 P0、deterministic 10/10、5/5 negatives、CR155 | validator binds target hash；chain/tamper/count fail closed |
| docs/maintainability | final names、ownership、switch conditions 可读 | six objects、four ADR decision items、Gotchas |

## Repository Facts That Shaped the Design

- Chain A：`run_stage3_mature_multifactor_research` orchestration 直接调用 `build_strategy_candidate`，后续形成 admission package/manifest；wrapper 与 hook 是同一 chain。
- Chain B：`run_strategy_research` 调用 `build_strategy_candidates`、refine、package builder；wrapper 与 hook 是同一 chain。
- `ExperimentManifest` 以 `run_id` 为单次 run 主键；不能承载跨 trial family owner。
- CR151 当前 overfit report 接收 `trial_count`；CR154 reliability artifact 接收 raw/effective/provenance；package 已有 attach existing gate 的 status-worsening 模式。因此引入 validator projection，而非新 gate。

## Decision Outcome for HLD Draft

1. Public application contract：`FamilyLineageSession` façade。
2. Persistence contract：typed event commands，create-only spec JSON + append-only events JSONL。
3. Seal：schema-versioned restricted canonical JSON + SHA-256；immutable versioned manifest。
4. Supersession：prior ref/hash/reason 全链；latest pointer 非真相。
5. Consumer：validated projection only；manual count reconciliation-only。
6. Inventory/Stories：2 chains / 4 mappings；S03 4/4；S01-S05 总数不变。

## Deferred Ideas

| ID | Deferred | Reason | Restart condition |
|---|---|---|---|
| DAI-CR163-01 | effective/statistical producer | scope/方法未授权 | independent CR + method/validation approval |
| DAI-CR163-02 | historical lineage backfill | native provenance不能推断 | audit/data authorization + inferred provenance policy |
| DAI-CR163-03 | real ML/event runner | no authorized runner | real runner + runtime/data gate |
| DAI-CR163-04 | SQLite/registry backend | current local sequential scope does not need it | concurrent writer/scale threshold + migration CR |

## Blocking DQ Batch

`none`。以下 `DQ-CP3-CR163-001..004` 是 formal CP3 confirmation items，不是阻断本 agent 完成设计的缺失信息；由 host 汇入 Decision Brief。

## Not Authorized

source/tests/DEVELOPMENT-PLAN/Story/LLD/quality/release/runtime/data/credentials/external write/statistical computation/historical backfill/Git remote 均未修改或授权。

