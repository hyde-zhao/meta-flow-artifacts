---
title: "CR166 Fixes"
status: "CLOSED"
version: "1.1"
cr_id: "CR-166"
updated_at: "2026-07-13T15:19:46+08:00"
---

# CR166 回修记录

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-13 | host-orchestrator inline | 记录治理登记和两项 CP7 语义回修的闭环证据。 |
| 1.1 | 2026-07-13 | host-orchestrator inline | 记录 CP8 批准后关闭态 artifact-hygiene 回修。 |

| ID | 状态 | 修复 | 验证 |
|---|---|---|---|
| GR-CR166-001 | CLOSED | 登记 7 个新 test provenance；归档 5 份专题设计；分类 CR166 active assets。 | 25 guardrail tests + final full suite PASS |
| QR-CR166-001 | CLOSED | component 保存 `leakage_unit/purge_applied/embargo_applied`，CR151 legacy projection 使用真实 day 值。 | `plan.embargo_days == 1` |
| QR-CR166-002 | CLOSED | 增加 non-present structural validation；typed-unavailable 原样投影但 package 仍 blocked；重算 metric decision 与 aggregate outcome。 | typed-unavailable + semantic tamper tests |
| GR-CR166-002 | CLOSED | active-only CR166 资产分类改为支持关闭态 `closed_cr166_asset`，新增关闭态回归测试。 | unclassified=0；42 guardrail tests；1987 full suite |

开放问题 0；waiver 0；未创建后续缺陷工单。真实数据/runtime 验证属于明确 Out of Scope，不作为回修项。
