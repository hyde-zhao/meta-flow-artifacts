---
status: baseline
version: "1.0"
created_at: "2026-07-02"
owner: "host-orchestrator"
cr_ref: "CR-037"
source_scenarios: "process/docs/product/SCENARIOS.yaml"
source_requirements: "process/docs/product/REQUIREMENTS.md"
source_story_map: "process/docs/product/STORY-MAP.md"
---

# Meta Flow 项目治理与状态强制测试覆盖矩阵

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活但 CP2 pending，自动化 planned 原因改为尚未通过 CP2/CP3/CP5。 |
| 1.0 | 2026-07-02 | host-orchestrator | 补齐 CR-037 工程验证场景覆盖矩阵，覆盖 UC-PG-001..007、REQ-PG-001..022 与 ST-PG-001..013。 |

## 覆盖统计

| 指标 | 数值 | 说明 |
|---|---:|---|
| 场景总数 | 18 | `TC-PG-001..018` |
| 已映射 Story 的场景 | 18 | 每个 TC 至少回链一个 ST-PG |
| 已映射需求的场景 | 18 | 每个 TC 至少回链一个 REQ-PG |
| 自动化状态 | 0 implemented / 18 planned | 当前 CR-037 active 但 CP2 pending，尚未通过 CP2/CP3/CP5，未授权实现 |
| 覆盖结论 | BASELINE_READY_GATE_PENDING | 可作为 CP1/CP2 输入；不能作为 CP7 执行证据 |

## 场景覆盖矩阵

| TC ID | 类型 | 优先级 | 覆盖 UC | 覆盖需求 | 覆盖 Story | 测试层级 | 自动化状态 | 手工验收状态 | 未覆盖 / 延后原因 |
|---|---|---|---|---|---|---|---|---|---|
| TC-PG-001 | positive | HIGH | UC-PG-001 | REQ-PG-001, REQ-PG-002, REQ-PG-003 | ST-PG-001, ST-PG-002 | unit | planned | n/a | CR-037 active 但 CP2/CP3/CP5 未通过，待 P0 实现授权 |
| TC-PG-002 | negative | HIGH | UC-PG-001 | REQ-PG-001, REQ-PG-NF003 | ST-PG-001 | unit | planned | n/a | CR-037 active 但 CP2/CP3/CP5 未通过，待 P0 实现授权 |
| TC-PG-003 | boundary | HIGH | UC-PG-001 | REQ-PG-002, REQ-PG-NF001 | ST-PG-001 | unit | planned | n/a | CR-037 active 但 CP2/CP3/CP5 未通过，待 P0 实现授权 |
| TC-PG-004 | precheck | HIGH | UC-PG-001 | REQ-PG-004, REQ-PG-005, REQ-PG-NF001 | ST-PG-002, ST-PG-003 | static-check | planned | n/a | CR-037 active 但 CP2/CP3/CP5 未通过，待 P0 实现授权 |
| TC-PG-005 | positive | HIGH | UC-PG-002 | REQ-PG-006, REQ-PG-007, REQ-PG-008 | ST-PG-004 | integration | planned | n/a | P1 依赖 P0 |
| TC-PG-006 | negative | HIGH | UC-PG-002 | REQ-PG-008, REQ-PG-NF001 | ST-PG-004 | unit | planned | n/a | P1 依赖 P0 |
| TC-PG-007 | positive | MEDIUM | UC-PG-003 | REQ-PG-009, REQ-PG-C003 | ST-PG-005 | integration | planned | n/a | P1 依赖 P0 |
| TC-PG-008 | positive | HIGH | UC-PG-004 | REQ-PG-010, REQ-PG-011 | ST-PG-006 | checker | planned | n/a | P1 registry checker 待实现 |
| TC-PG-009 | negative | HIGH | UC-PG-004 | REQ-PG-011, REQ-PG-C004 | ST-PG-006 | checker | planned | n/a | P1 registry checker 待实现 |
| TC-PG-010 | positive | HIGH | UC-PG-005 | REQ-PG-012, REQ-PG-013 | ST-PG-007 | migration-dry-run | planned | n/a | P1 impact migration 待实现 |
| TC-PG-011 | negative | HIGH | UC-PG-005 | REQ-PG-012, REQ-PG-NF002 | ST-PG-007 | checker | planned | n/a | P1 impact migration 待实现 |
| TC-PG-012 | positive | HIGH | UC-PG-006 | REQ-PG-014, REQ-PG-015, REQ-PG-017 | ST-PG-008, ST-PG-009 | checker | planned | n/a | P1 roadmap-refresh checker 待实现 |
| TC-PG-013 | permission | HIGH | UC-PG-006 | REQ-PG-016, REQ-PG-C002 | ST-PG-009 | integration | planned | n/a | P1 cascade 待实现；发布库写入禁止 |
| TC-PG-014 | positive | MEDIUM | UC-PG-006 | REQ-PG-018, REQ-PG-019 | ST-PG-010 | checker | planned | n/a | P1 FU-RF/stale-check 待实现 |
| TC-PG-015 | positive | HIGH | UC-PG-007 | REQ-PG-020, REQ-PG-021, REQ-PG-022, REQ-PG-NF004 | ST-PG-011 | migration-dry-run | planned | n/a | P2 依赖 P0/P1 |
| TC-PG-016 | negative | HIGH | UC-PG-007 | REQ-PG-021, REQ-PG-C004 | ST-PG-011 | migration-dry-run | planned | n/a | P2 依赖 registry readiness |
| TC-PG-017 | negative | MEDIUM | UC-PG-001 | REQ-PG-C005 | ST-PG-012 | cli-contract-check | planned | n/a | P0.5 ledger compaction 设计待展开 |
| TC-PG-018 | negative | HIGH | UC-PG-001, UC-PG-004, UC-PG-005, UC-PG-006 | REQ-PG-C001 | ST-PG-013 | design-guardrail-review | planned | n/a | CR-037 active 但 CP2/CP3/CP5 未通过，待设计/实现 diff 审查授权 |

## Requirement 覆盖索引

| Requirement | 覆盖 TC |
|---|---|
| REQ-PG-001 | TC-PG-001, TC-PG-002 |
| REQ-PG-002 | TC-PG-001, TC-PG-003 |
| REQ-PG-003 | TC-PG-001 |
| REQ-PG-004 | TC-PG-004 |
| REQ-PG-005 | TC-PG-004 |
| REQ-PG-006 | TC-PG-005 |
| REQ-PG-007 | TC-PG-005 |
| REQ-PG-008 | TC-PG-005, TC-PG-006 |
| REQ-PG-009 | TC-PG-007 |
| REQ-PG-010 | TC-PG-008 |
| REQ-PG-011 | TC-PG-008, TC-PG-009 |
| REQ-PG-012 | TC-PG-010, TC-PG-011 |
| REQ-PG-013 | TC-PG-010 |
| REQ-PG-014 | TC-PG-012 |
| REQ-PG-015 | TC-PG-012 |
| REQ-PG-016 | TC-PG-013 |
| REQ-PG-017 | TC-PG-012 |
| REQ-PG-018 | TC-PG-014 |
| REQ-PG-019 | TC-PG-014 |
| REQ-PG-020 | TC-PG-015 |
| REQ-PG-021 | TC-PG-015, TC-PG-016 |
| REQ-PG-022 | TC-PG-015 |
| REQ-PG-C001 | TC-PG-018 |
| REQ-PG-C002 | TC-PG-013 |
| REQ-PG-C003 | TC-PG-007 |
| REQ-PG-C004 | TC-PG-009, TC-PG-016 |
| REQ-PG-C005 | TC-PG-017 |
| REQ-PG-NF001 | TC-PG-003, TC-PG-004, TC-PG-006 |
| REQ-PG-NF002 | TC-PG-011 |
| REQ-PG-NF003 | TC-PG-002 |
| REQ-PG-NF004 | TC-PG-015 |

## 当前缺口

| 缺口 | 状态 | 处理 |
|---|---|---|
| 自动化测试均未实现 | EXPECTED | CR-037 active 但 CP2 pending，尚未通过 CP2/CP3/CP5，未授权实现 |
| CP7 执行证据不存在 | EXPECTED | 当前只是 baseline，不是 verification report |
