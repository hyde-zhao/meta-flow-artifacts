---
discussion_id: "CP3-CR158-HLD-DISCUSSION"
cr_id: "CR-158"
phase: "solution-design"
status: "summarized-for-cp3"
created_at: "2026-07-05T17:20:00+08:00"
owner: "host-orchestrator"
---

# CP3 CR158 HLD Discussion Log

## Architecture Gray Areas

| ID | 问题 | 影响面 | 推荐方案 | 状态 |
|---|---|---|---|---|
| AGA-CR158-01 | shared core 厚度 | schema / Story split / validation | thin shared core + typed extensions | included-in-HLD |
| AGA-CR158-02 | evidence extension 归属 | evidence index / handoff / CP7 | adapter owns shape, evidence index stores refs-only | included-in-HLD |
| AGA-CR158-03 | no-runtime enforcement 层级 | safety / CP7 / release wording | counter report + FEAT-07 guard | included-in-HLD |
| AGA-CR158-04 | 是否拆分 HLD | governance / review / Story planning | single HLD, split only if CP5 proves shared core invalid | included-in-HLD |

## Advisor Summary

| Lane | 关注点 | 结论 |
|---|---|---|
| lane-product | 用户希望把两个 deferred item 合并推进，避免两个 CR 重复门禁。 | 支持单 CR 和统一 adapter core，但必须保持 event/ML 差异可见。 |
| lane-architecture | 模块边界、schema 演进和 ADR。 | 推荐 thin core + typed extensions。 |
| lane-quality | no-runtime、refs-only evidence、negative scenarios。 | 推荐 adapter counter report + FEAT-07 guard。 |
| lane-docs | release wording 和误用防护。 | CP3/CP8 必须复述 not-authorized list。 |

## Deferred / Open

| ID | 内容 | 状态 | 触发条件 |
|---|---|---|---|
| DEF-CP3-CR158-01 | 如果 CP5 证明 shared core 过度泛化，拆分 event / ML 子 CR。 | non-blocking-open | CP5 design evidence fails extension isolation. |
| DEF-CP3-CR158-02 | 真实 feed / training / registry / runtime proof。 | not-authorized | Separate runtime authorization CR. |
