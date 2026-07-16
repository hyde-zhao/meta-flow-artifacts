---
status: baseline
version: "1.9"
created_at: "2026-07-02"
owner: "host-orchestrator"
cr_ref: "CR-037"
active_change_ref: "CR-050"
source_scenarios: "process/docs/product/SCENARIOS.yaml"
source_requirements: "process/docs/product/REQUIREMENTS.md"
source_story_map: "process/docs/product/STORY-MAP.md"
---

# Meta Flow 项目治理与状态强制测试覆盖矩阵

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.6 | 2026-07-15 | host-orchestrator | CP8 批准后增加 ledger 对象身份 selector 回归；最终 401+70，57-object firewall=0，状态投影为 delivered。 |
| 1.7 | 2026-07-15 | host-orchestrator inline fallback | 增量追加 CR-050 TC-GB-001..011 的需求/Story/验证层映射；保留 CR-046/047 delivered-with-risk 历史结论，当前自动化保持 planned/CP2 pending。 |
| 1.7.1 | 2026-07-16 | host-orchestrator inline fallback | 记录 CR-050 CP2 已批准并进入 CP3；11/11 场景仍为 planned，未生成实现或验证证据。 |
| 1.8 | 2026-07-16 | host-orchestrator inline fallback | 用户将独立两仓 fast-forward-only merge 纳入 CR-050；新增 TC-GB-012..017 与 ST-GB-004 映射，旧 CP3 转 changes_requested，覆盖状态回到 CP2 R2 pending。 |
| 1.9 | 2026-07-16 | host-orchestrator-inline/meta-qa | CR-050 TC-GB-001..017 全部映射到 13 个 bare-remote/negative/freshness fixture；四个 Story CP7 R2 均为 `PASS_WITH_RISK`，真实远端、独立 QA 与平台 receipt 风险保留。 |
| 1.5 | 2026-07-15 | host-orchestrator-inline/meta-qa | 将 CR-046 18 个 TC 投影为 7/7 CP7 `PASS_WITH_RISK`；登记 CR-047 7/7 CP6/CP7 实证与五门结果。 |
| 1.4 | 2026-07-13 | host-orchestrator-inline-fallback | 纠正 CR-046 当前实现/验证状态，追加 CR-047 TC-WT-001..007 修复前基线与验收矩阵。 |
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活但 CP2 pending，自动化 planned 原因改为尚未通过 CP2/CP3/CP5。 |
| 1.2 | 2026-07-11 | meta-pm | 为 CR-046 增量追加 TC-EI-001..013、REQ-EI 与 ST-EI 回链；保留既有矩阵行。 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：追加 TC-EI-014..018 与 REQ-EI-019..023 回链，保留全部既有 TC/REQ/ST ID。 |
| 1.0 | 2026-07-02 | host-orchestrator | 补齐 CR-037 工程验证场景覆盖矩阵，覆盖 UC-PG-001..007、REQ-PG-001..022 与 ST-PG-001..013。 |

## 覆盖统计

| 指标 | 数值 | 说明 |
|---|---:|---|
| 场景总数 | 60 | `TC-PG-001..018` + `TC-EI-001..018` + `TC-WT-001..007` + `TC-GB-001..017` |
| 已映射 Story 的场景 | 60 | 60/60 TC 至少回链一个 Story |
| 已映射需求的场景 | 60 | 60/60 TC 至少回链一个 Requirement |
| 自动化状态 | CR-046/047 已交付带风险；CR-050 17/17 planned | CR-050 CP2 R2 pending、尚未实现；既有 401+70 只代表前序基线 |
| 覆盖结论 | CR050_CP2_R2_PENDING | merge 范围已结构化；CP2 R2 未批准前不得修订架构、拆 Story 设计或实现，真实 remote mutation 未授权 |

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
| TC-EI-001 | positive | HIGH | UC-EI-001 | REQ-EI-001, REQ-EI-002 | ST-EI-001 | chronology-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-002 | negative | HIGH | UC-EI-001 | REQ-EI-001, REQ-EI-002, REQ-EI-NF001 | ST-EI-001 | negative-fixture-suite | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-003 | positive | HIGH | UC-EI-002 | REQ-EI-003, REQ-EI-005 | ST-EI-002 | dispatch-contract-test | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-004 | negative | HIGH | UC-EI-002 | REQ-EI-005, REQ-EI-C001 | ST-EI-002 | dispatch-degradation-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-005 | failure-recovery | HIGH | UC-EI-002 | REQ-EI-003, REQ-EI-004 | ST-EI-002 | retry-recovery-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-006 | positive | HIGH | UC-EI-001, UC-EI-002 | REQ-EI-006, REQ-EI-007 | ST-EI-003 | cp-result-correlation-test | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-007 | negative | HIGH | UC-EI-001 | REQ-EI-008, REQ-EI-NF001 | ST-EI-003 | cross-truth-negative-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-008 | precheck | HIGH | UC-EI-001 | REQ-EI-009 | ST-EI-004 | state-finalization-check | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-009 | permission | HIGH | UC-EI-001 | REQ-EI-010, REQ-EI-C003 | ST-EI-004 | authorization-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-010 | positive | HIGH | UC-EI-004 | REQ-EI-011, REQ-EI-012, REQ-EI-013 | ST-EI-005 | telemetry-contract-test | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-011 | boundary | HIGH | UC-EI-003 | REQ-EI-014, REQ-EI-015, REQ-EI-C001, REQ-EI-C002 | ST-EI-006 | replay-compatibility-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-012 | positive | HIGH | UC-EI-003 | REQ-EI-014, REQ-EI-015, REQ-EI-NF002 | ST-EI-006 | replay-reproducibility-test | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-013 | positive | HIGH | UC-EI-005 | REQ-EI-016, REQ-EI-017, REQ-EI-018, REQ-EI-NF002 | ST-EI-007 | append-only-migration-replay | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-014 | negative | HIGH | UC-EI-001, UC-EI-002 | REQ-EI-019, REQ-EI-NF001 | ST-EI-002, ST-EI-004 | compaction-restore-semantic-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-015 | positive | HIGH | UC-EI-003, UC-EI-005 | REQ-EI-020, REQ-EI-016, REQ-EI-NF002 | ST-EI-007 | post-close-correction-contract-test | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-016 | positive | HIGH | UC-EI-001, UC-EI-004 | REQ-EI-021, REQ-EI-013, REQ-EI-014 | ST-EI-004, ST-EI-006 | audit-report-golden-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-017 | negative | HIGH | UC-EI-003 | REQ-EI-022, REQ-EI-014, REQ-EI-015, REQ-EI-NF002 | ST-EI-006 | null-provenance-dogfood-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |
| TC-EI-018 | boundary | HIGH | UC-EI-002 | REQ-EI-023, REQ-EI-005, REQ-EI-C001 | ST-EI-002 | dispatch-evidence-level-fixture | implemented | PASS_WITH_RISK | 证据见 CR-046 7/7 CP6/CP7 result；结论保持带风险 |

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
| CR-037 历史 planned 行仍保留 | HISTORICAL | 不改写旧行；以关闭 CR 与后续 CP evidence 解释当前状态 |
| CR-046 Story 验证 | COMPLETED_WITH_RISK | 7/7 CP7 result 为 PASS_WITH_RISK；不得写成未实现，也不得升级为无风险 |
| CR-047 自动化与 Story 验证 | DELIVERED_WITH_RISK | 7/7 CP6 PASS、7/7 CP7 PASS_WITH_RISK；CP8 approved；publication preflight 401+70 |
| CR-050 branch lifecycle 自动化 | PLANNED_CP2_R2_PENDING | 17/17 场景已回链 4 个 Story；旧 CP3 因 merge 范围变化被退回；CP2 R2 前不得修订架构，CP5 前不得写源码；真实远端 branch/default mutation 仍需显式授权 |
| platform receipt 或 token telemetry 可能不可用 | ACCEPTED-DEGRADATION | 必须显式 unavailable；不得合成或标记为 verified/measured |
| 当前 Codex meta-pm dispatch 无 repository-verifiable platform receipt | OBSERVED-LIMITATION | CP2 披露为 `session-observed/repository-unverifiable`；后续 receipt 只能追加升级证据 |
| CR-046 R1 CP1/CP2 checker provenance 为空 | REQUIRED-DOGFOOD | 保留原始 result/hash，作为 TC-EI-017 strict negative fixture；不得静默重写 |

## CR-047 场景覆盖矩阵

| TC ID | 类型 | 优先级 | 覆盖 UC | 覆盖需求 | 覆盖 Story | 测试层级 | 自动化状态 | 当前基线 |
|---|---|---|---|---|---|---|---|---|
| TC-WT-001 | negative | HIGH | UC-WT-001 | REQ-WT-001..003 | ST-WT-001 | contract/integration | implemented | CR tracking exit 0；legacy YAML=0；CR-033 candidate 可持久保留 |
| TC-WT-002 | positive | HIGH | UC-WT-002 | REQ-WT-004..005 | ST-WT-002 | clean-clone integration | implemented | routing/relocation/idempotence fixtures 8 passed；workspace health=ok |
| TC-WT-003 | failure-recovery | HIGH | UC-WT-003 | REQ-WT-006..008,017 | ST-WT-003 | doctor/regression | implemented | Doctor exit 0；B0_cp7=21、blocking=0、unclassified=0，历史 warning 保留 |
| TC-WT-004 | negative | HIGH | UC-WT-004 | REQ-WT-009..010 | ST-WT-004 | archive/guardrail | implemented | clean clone 不依赖 root wrapper；ignored cache 仅 warning，tracked/package cache 仍阻断 |
| TC-WT-005 | positive | HIGH | UC-WT-005 | REQ-WT-011..012 | ST-WT-005 | lint/full regression | implemented | Ruff=0；pytest=401 passed + 70 subtests |
| TC-WT-006 | boundary | HIGH | UC-WT-006 | REQ-WT-013..014 | ST-WT-006 | installer matrix | implemented | Codex/Claude/Qoder 3/3 dry-run，真实目标写入=0 |
| TC-WT-007 | precheck | HIGH | UC-WT-007 | REQ-WT-015..016 | ST-WT-007 | evidence convergence | implemented | 57 protected objects；CP6/CP7/final findings=0；无关 ledger append selector regression 通过；CR-046 保持 closed/READY_WITH_RISK |

## CR-047 Requirement 覆盖索引

| Requirement | 覆盖 TC |
|---|---|
| REQ-WT-001..003 | TC-WT-001 |
| REQ-WT-004..005 | TC-WT-002 |
| REQ-WT-006..008, REQ-WT-017 | TC-WT-003 |
| REQ-WT-009..010 | TC-WT-004 |
| REQ-WT-011..012 | TC-WT-005 |
| REQ-WT-013..014 | TC-WT-006 |
| REQ-WT-015..016 | TC-WT-007 |

## CR-050 场景覆盖矩阵

| TC ID | 类型 | 优先级 | 覆盖 UC | 覆盖需求 | 覆盖 Story | 测试层级 | 自动化状态 | 当前基线 |
|---|---|---|---|---|---|---|---|---|
| TC-GB-001 | positive | HIGH | UC-GB-001 | REQ-GB-001,003,004 | ST-GB-001 | bare-remote integration | verified | `test_open_creates_exact_paired_branches_and_upstreams`; CP7 R2 |
| TC-GB-002 | negative | HIGH | UC-GB-001 | REQ-GB-002,003, REQ-GB-C002 | ST-GB-001 | negative bare-remote fixture | verified | dirty/ref drift 在 mutation 前 BLOCKED |
| TC-GB-003 | boundary | HIGH | UC-GB-001 | REQ-GB-002,004, REQ-GB-NF002 | ST-GB-001 | input contract | verified | unsafe/colliding ref allowlist + `check-ref-format` 双门 |
| TC-GB-004 | positive | HIGH | UC-GB-002 | REQ-GB-005,010 | ST-GB-002 | bare-remote push | verified | captured committed OID exact push + matching evidence |
| TC-GB-005 | precheck | HIGH | UC-GB-002 | REQ-GB-005, REQ-GB-C002 | ST-GB-002 | dirty-tree negative | verified | `test_publish_dirty_tree_never_auto_commits` |
| TC-GB-006 | positive | HIGH | UC-GB-003 | REQ-GB-007,008,010 | ST-GB-003 | merge-cleanup integration | verified | fresh ancestry + recovery-first + artifact→project cleanup |
| TC-GB-007 | negative | HIGH | UC-GB-003 | REQ-GB-007, REQ-GB-C002 | ST-GB-003 | ancestry negative | verified | incomplete projection/ref drift fail closed；squash/rebase风险保留 |
| TC-GB-008 | permission | HIGH | UC-GB-003 | REQ-GB-008, REQ-GB-NF002 | ST-GB-003 | authorization negative | verified | typed delete authz 与 current OID/repo/ref 绑定 |
| TC-GB-009 | failure-recovery | HIGH | UC-GB-001..004 | REQ-GB-010,014, REQ-GB-NF001,003 | ST-GB-001..004 | fault injection | verified | paired merge/finish PARTIAL 均可 fresh resume，误报 PASS=0 |
| TC-GB-010 | boundary | HIGH | UC-GB-001..004 | REQ-GB-006, REQ-GB-NF001 | ST-GB-001..004 | dry-run golden | verified | open dry-run local/remote ref mutation=0；执行面共享同一 plan contract |
| TC-GB-011 | boundary | HIGH | UC-GB-003 | REQ-GB-009,010 | ST-GB-003 | idempotence cleanup | verified | first-remote absent 仅在 known tip/fresh proof 后 NO_CHANGE |
| TC-GB-012 | positive | HIGH | UC-GB-004 | REQ-GB-011..013 | ST-GB-004 | bare-remote fast-forward integration | verified | exact OID ordinary push，artifact→project，2/2 projection=true |
| TC-GB-013 | negative | HIGH | UC-GB-004 | REQ-GB-012..013, REQ-GB-C002 | ST-GB-004 | non-fast-forward negative | verified | `+`/force/local merge/rebase 调用=0；Git 非 FF 拒绝保留 PARTIAL |
| TC-GB-014 | permission | HIGH | UC-GB-004 | REQ-GB-011, REQ-GB-C004 | ST-GB-004 | default-write authorization | verified | operation/repo/ref/OID typed authz 不匹配即 BLOCKED |
| TC-GB-015 | failure-recovery | HIGH | UC-GB-003..004 | REQ-GB-012,014, REQ-GB-NF003 | ST-GB-003..004 | paired merge fault injection | verified | artifact成功/project失败时 projection/finish/close 均 false，2/2 refs保留 |
| TC-GB-016 | boundary | HIGH | UC-GB-004 | REQ-GB-006,012, REQ-GB-NF003 | ST-GB-004 | merge dry-run golden | verified | dry-run 不调用 default/CR ref mutation；argv 可审计 |
| TC-GB-017 | precheck | HIGH | UC-GB-003..004 | REQ-GB-007,014 | ST-GB-003..004 | merge-finish gate idempotence | verified | current 2/2 projection/fresh proof 前 recovery/delete=0 |

## CR-050 Requirement 覆盖索引

| Requirement | 覆盖 TC |
|---|---|
| REQ-GB-001,003,004 | TC-GB-001 |
| REQ-GB-002 | TC-GB-002, TC-GB-003 |
| REQ-GB-005 | TC-GB-004, TC-GB-005 |
| REQ-GB-006 | TC-GB-010 |
| REQ-GB-007 | TC-GB-006, TC-GB-007 |
| REQ-GB-008 | TC-GB-006, TC-GB-008 |
| REQ-GB-009 | TC-GB-011 |
| REQ-GB-010 | TC-GB-004, TC-GB-006, TC-GB-009, TC-GB-011 |
| REQ-GB-011 | TC-GB-012, TC-GB-014 |
| REQ-GB-012 | TC-GB-012, TC-GB-013, TC-GB-015, TC-GB-016 |
| REQ-GB-013 | TC-GB-012, TC-GB-013 |
| REQ-GB-014 | TC-GB-009, TC-GB-015, TC-GB-017 |
| REQ-GB-C001 | CP2/CP3 dependency review + clean-env fixture |
| REQ-GB-C002 | TC-GB-002, TC-GB-005, TC-GB-007, TC-GB-013 |
| REQ-GB-C003 | CP2 R2 human-gate authorization check |
| REQ-GB-C004 | TC-GB-014 |
| REQ-GB-NF001 | TC-GB-009, TC-GB-010 |
| REQ-GB-NF002 | TC-GB-003, TC-GB-008 |
| REQ-GB-NF003 | TC-GB-009, TC-GB-012, TC-GB-015, TC-GB-016 |
