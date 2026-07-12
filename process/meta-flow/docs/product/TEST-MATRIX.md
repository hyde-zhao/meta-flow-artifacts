---
status: baseline
version: "1.3"
created_at: "2026-07-02"
owner: "host-orchestrator"
cr_ref: "CR-037"
active_change_ref: "CR-046"
source_scenarios: "process/docs/product/SCENARIOS.yaml"
source_requirements: "process/docs/product/REQUIREMENTS.md"
source_story_map: "process/docs/product/STORY-MAP.md"
---

# Meta Flow 项目治理与状态强制测试覆盖矩阵

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活但 CP2 pending，自动化 planned 原因改为尚未通过 CP2/CP3/CP5。 |
| 1.2 | 2026-07-11 | meta-pm | 为 CR-046 增量追加 TC-EI-001..013、REQ-EI 与 ST-EI 回链；保留既有矩阵行。 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：追加 TC-EI-014..018 与 REQ-EI-019..023 回链，保留全部既有 TC/REQ/ST ID。 |
| 1.0 | 2026-07-02 | host-orchestrator | 补齐 CR-037 工程验证场景覆盖矩阵，覆盖 UC-PG-001..007、REQ-PG-001..022 与 ST-PG-001..013。 |

## 覆盖统计

| 指标 | 数值 | 说明 |
|---|---:|---|
| 场景总数 | 36 | `TC-PG-001..018` + `TC-EI-001..018` |
| 已映射 Story 的场景 | 36 | 每个 TC 至少回链一个 ST-PG 或 ST-EI |
| 已映射需求的场景 | 36 | 每个 TC 至少回链一个 REQ-PG 或 REQ-EI |
| 自动化状态 | 0 implemented / 36 planned | CR-046 仍待 CP2；未授权实现、runtime 或 quant-lab business-code 修改 |
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
| TC-EI-001 | positive | HIGH | UC-EI-001 | REQ-EI-001, REQ-EI-002 | ST-EI-001 | chronology-fixture | planned | n/a | CR-046 CP2 pending |
| TC-EI-002 | negative | HIGH | UC-EI-001 | REQ-EI-001, REQ-EI-002, REQ-EI-NF001 | ST-EI-001 | negative-fixture-suite | planned | n/a | CR-046 CP2 pending |
| TC-EI-003 | positive | HIGH | UC-EI-002 | REQ-EI-003, REQ-EI-005 | ST-EI-002 | dispatch-contract-test | planned | n/a | CR-046 CP2 pending |
| TC-EI-004 | negative | HIGH | UC-EI-002 | REQ-EI-005, REQ-EI-C001 | ST-EI-002 | dispatch-degradation-fixture | planned | n/a | 缺 receipt 的合法结果是 unavailable |
| TC-EI-005 | failure-recovery | HIGH | UC-EI-002 | REQ-EI-003, REQ-EI-004 | ST-EI-002 | retry-recovery-fixture | planned | n/a | CR-046 CP2 pending |
| TC-EI-006 | positive | HIGH | UC-EI-001, UC-EI-002 | REQ-EI-006, REQ-EI-007 | ST-EI-003 | cp-result-correlation-test | planned | n/a | CR-046 CP2 pending |
| TC-EI-007 | negative | HIGH | UC-EI-001 | REQ-EI-008, REQ-EI-NF001 | ST-EI-003 | cross-truth-negative-fixture | planned | n/a | CR-046 CP2 pending |
| TC-EI-008 | precheck | HIGH | UC-EI-001 | REQ-EI-009 | ST-EI-004 | state-finalization-check | planned | n/a | CR-046 CP2 pending |
| TC-EI-009 | permission | HIGH | UC-EI-001 | REQ-EI-010, REQ-EI-C003 | ST-EI-004 | authorization-fixture | planned | n/a | CP2 不授权 runtime/credentials/publish |
| TC-EI-010 | positive | HIGH | UC-EI-004 | REQ-EI-011, REQ-EI-012, REQ-EI-013 | ST-EI-005 | telemetry-contract-test | planned | n/a | 缺 telemetry 的合法结果是 unavailable |
| TC-EI-011 | boundary | HIGH | UC-EI-003 | REQ-EI-014, REQ-EI-015, REQ-EI-C001, REQ-EI-C002 | ST-EI-006 | replay-compatibility-fixture | planned | n/a | 历史 checker provenance 可 unavailable，不得合成 |
| TC-EI-012 | positive | HIGH | UC-EI-003 | REQ-EI-014, REQ-EI-015, REQ-EI-NF002 | ST-EI-006 | replay-reproducibility-test | planned | n/a | CR-046 CP2 pending |
| TC-EI-013 | positive | HIGH | UC-EI-005 | REQ-EI-016, REQ-EI-017, REQ-EI-018, REQ-EI-NF002 | ST-EI-007 | append-only-migration-replay | planned | n/a | pilot 需后续独立授权；业务源码 immutable |
| TC-EI-014 | negative | HIGH | UC-EI-001, UC-EI-002 | REQ-EI-019, REQ-EI-NF001 | ST-EI-002, ST-EI-004 | compaction-restore-semantic-fixture | planned | n/a | CR-046 CP2 pending；CP7 必须覆盖 fallback conflation 负例 |
| TC-EI-015 | positive | HIGH | UC-EI-003, UC-EI-005 | REQ-EI-020, REQ-EI-016, REQ-EI-NF002 | ST-EI-007 | post-close-correction-contract-test | planned | n/a | 通用 lifecycle 先于 CR-163 pilot；非法历史改写必须拒绝 |
| TC-EI-016 | positive | HIGH | UC-EI-001, UC-EI-004 | REQ-EI-021, REQ-EI-013, REQ-EI-014 | ST-EI-004, ST-EI-006 | audit-report-golden-fixture | planned | n/a | 报告必须由工具生成并与已知计数 oracle 对账 |
| TC-EI-017 | negative | HIGH | UC-EI-003 | REQ-EI-022, REQ-EI-014, REQ-EI-015, REQ-EI-NF002 | ST-EI-006 | null-provenance-dogfood-fixture | planned | n/a | R1 原始结果保留；strict profile 不得报 fully replayable |
| TC-EI-018 | boundary | HIGH | UC-EI-002 | REQ-EI-023, REQ-EI-005, REQ-EI-C001 | ST-EI-002 | dispatch-evidence-level-fixture | planned | n/a | 当前调度仅 session-observed/repository-unverifiable，直到有 platform receipt |

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
| REQ-EI-001 | TC-EI-001, TC-EI-002 |
| REQ-EI-002 | TC-EI-001, TC-EI-002 |
| REQ-EI-003 | TC-EI-003, TC-EI-005 |
| REQ-EI-004 | TC-EI-005 |
| REQ-EI-005 | TC-EI-003, TC-EI-004 |
| REQ-EI-006 | TC-EI-006 |
| REQ-EI-007 | TC-EI-006 |
| REQ-EI-008 | TC-EI-007 |
| REQ-EI-009 | TC-EI-008 |
| REQ-EI-010 | TC-EI-009 |
| REQ-EI-011 | TC-EI-010 |
| REQ-EI-012 | TC-EI-010 |
| REQ-EI-013 | TC-EI-010 |
| REQ-EI-014 | TC-EI-011, TC-EI-012 |
| REQ-EI-015 | TC-EI-011, TC-EI-012 |
| REQ-EI-016 | TC-EI-013 |
| REQ-EI-017 | TC-EI-013 |
| REQ-EI-018 | TC-EI-013 |
| REQ-EI-019 | TC-EI-014 |
| REQ-EI-020 | TC-EI-015 |
| REQ-EI-021 | TC-EI-016 |
| REQ-EI-022 | TC-EI-017 |
| REQ-EI-023 | TC-EI-018 |
| REQ-EI-C001 | TC-EI-004, TC-EI-011 |
| REQ-EI-C002 | TC-EI-011 |
| REQ-EI-C003 | TC-EI-009 |
| REQ-EI-NF001 | TC-EI-002, TC-EI-007 |
| REQ-EI-NF002 | TC-EI-012, TC-EI-013 |

## 当前缺口

| 缺口 | 状态 | 处理 |
|---|---|---|
| 自动化测试均未实现 | EXPECTED | CR-037 active 但 CP2 pending，尚未通过 CP2/CP3/CP5，未授权实现 |
| CP7 执行证据不存在 | EXPECTED | 当前只是 baseline，不是 verification report |
| CR-046 自动化均未实现 | EXPECTED | CP2/CP3/CP5 尚未通过；本矩阵只承诺后续验证覆盖 |
| platform receipt 或 token telemetry 可能不可用 | ACCEPTED-DEGRADATION | 必须显式 unavailable；不得合成或标记为 verified/measured |
| 当前 Codex meta-pm dispatch 无 repository-verifiable platform receipt | OBSERVED-LIMITATION | CP2 披露为 `session-observed/repository-unverifiable`；后续 receipt 只能追加升级证据 |
| CR-046 R1 CP1/CP2 checker provenance 为空 | REQUIRED-DOGFOOD | 保留原始 result/hash，作为 TC-EI-017 strict negative fixture；不得静默重写 |
