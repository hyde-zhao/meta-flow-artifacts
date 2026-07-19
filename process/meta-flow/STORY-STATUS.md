---
last_updated: "2026-07-18T10:59:12Z"
active_change_ref: "CR-051"
---

## Story 状态汇总

| Story ID | 标题 | Wave | 状态 | 负责人 | 阻塞 |
|----------|------|------|------|--------|------|
| UCD-01 | use-case-discovery 核心增强 | W1 | verified | meta-qa | 否 |
| UCD-02 | use-case-discovery 模板与引用增强 | W1 | verified | meta-qa | 否 |
| LLD-01 | lld-designer 六阶段管道与输入输出契约 | W1 | verified | meta-qa | 否 |
| UCD-03 | UCD 上下游契约同步 | W2 | verified | meta-qa | 否 |
| RG-01 | meta-po 评审协调核心 | W2 | verified | meta-qa | 否 |
| LLD-02 | STORY-LLD 模板增强（14 章节固定） | W2 | verified | meta-qa | 否 |
| RG-02 | review 产物协议与静态前置检查 | W3 | verified | meta-qa | 否 |
| LLD-03 | LLD 生产/消费契约对齐 | W3 | verified | meta-qa | 否 |
| RG-03 | review mode 第一组扩展 | W4 | verified | meta-qa | 否 |
| RG-04 | review mode 第二组扩展 | W4 | verified | meta-qa | 否 |
| RG-05 | 分派表与灰度落地 | W5 | verified | meta-qa | 否 |
| LLD-04 | LLD 方法试点 Story | W6 | verified | meta-qa | 否 |

## Wave 进度

| Wave | 总数 | done | verified | in-dev | blocked |
|------|------|------|----------|--------|---------|
| W1 | 3 | 0 | 3 | 0 | 0 |
| W2 | 3 | 0 | 3 | 0 | 0 |
| W3 | 2 | 0 | 2 | 0 | 0 |
| W4 | 2 | 0 | 2 | 0 | 0 |
| W5 | 1 | 0 | 1 | 0 | 0 |
| W6 | 1 | 0 | 1 | 0 | 0 |

## 阻塞项清单

- 当前无阻塞项

## CR-037 Story 状态汇总

| Story ID | 标题 | Wave | 状态 | 负责人 | 阻塞 |
|----------|------|------|------|--------|------|
| CR037-S00 | second-system guardrail | CR037-W0 | verified-with-risk | meta-dev | 否 |
| CR037-S01 | current-state schema and budgets | CR037-W1 | verified-with-risk | meta-dev | 否 |
| CR037-S02 | controlled update API and writer refactor | CR037-W1 | verified-with-risk | meta-dev | 否 |
| CR037-S03 | agent contract and guardrail sync | CR037-W1 | verified-with-risk | meta-dev | 否 |
| CR037-S04 | ledger compaction policy and CLI | CR037-W1A | verified-with-risk | meta-dev | 否 |
| CR037-S05 | project scaffold and PROJECT.current | CR037-W2 | verified-with-risk | meta-dev | 否 |
| CR037-S06 | PROJECT-SCALE and roadmap objects | CR037-W2 | verified-with-risk | meta-dev | 否 |
| CR037-S07 | feature/capability registry and resolver | CR037-W3 | verified-with-risk | meta-dev | 否 |
| CR037-S08 | impact surface field split and migration report | CR037-W3 | dev-ready | meta-dev | 否 |
| CR037-S09 | roadmap refresh result schema and checker | CR037-W4 | dev-ready | meta-dev | 否 |
| CR037-S10 | process-only cascade and Gate Ledger event | CR037-W4 | dev-ready | meta-dev | 否 |
| CR037-S11 | FU-RF tracking support | CR037-W4 | dev-ready | meta-dev | 否 |
| CR037-S12 | project stale-check | CR037-W4 | dev-ready | meta-dev | 否 |
| CR037-S13 | quant-lab migration dry-run and reports | CR037-W5 | dev-ready | meta-dev | 否 |

## CR-037 Wave 进度

| Wave | 总数 | lld-ready | lld-review | dev-ready | implementing | ready-for-verification | verified-with-risk | blocked |
|------|------|-----------|------------|-----------|--------------|------------------------|--------------------|---------|
| CR037-W0 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| CR037-W1 | 3 | 0 | 0 | 0 | 0 | 0 | 3 | 0 |
| CR037-W1A | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| CR037-W2 | 2 | 0 | 0 | 0 | 0 | 0 | 2 | 0 |
| CR037-W3 | 2 | 0 | 0 | 1 | 1 | 0 | 0 | 0 |
| CR037-W4 | 4 | 0 | 0 | 4 | 0 | 0 | 0 | 0 |
| CR037-W5 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |

## CR-037 实施期风险跟踪

| Risk ID | Story | 状态 | 实施期处理要求 |
|---------|-------|------|--------------------|
| R-CR037-S13-LONG-CHAIN | CR037-S13 | 实施期跟踪中，非 CP5 阻断 | S13 作为 P2 quant-lab migration 验证切片依赖 8 个 P0/P1 Story。真实 migration 必须等上游 Story verified / dev_gate 满足；若上游延期，只允许保持 blocked，或缩窄为只读 inventory/report。 |

## CR-051 Story 状态汇总

| Story ID | 标题 | Wave | 状态 | 设计证据 | lld_policy | Dev Gate | 负责人 | 阻塞 |
|---|---|---|---|---|---|---|---|---|
| ST-AW-001 | project-first routing | W1 | verified-with-risk | PASS / CP5 approved | full-lld | CP7-R2 PASS_WITH_RISK | complete | F01/F02 CLOSED；module manifest/跨设备 pilot 留 CP8 |
| ST-AW-002 | recoverable project worktree | W2 | verified-with-risk | PASS / CP5 approved | full-lld | CP7 PASS_WITH_RISK | complete | CAP/DUR 强制义务已通过；真实/Windows pilot 留 CP8 |
| ST-AW-003 | heterogeneous Git legs | W3 | dev-ready | PASS / CP5 approved | full-lld | open | meta-dev | git_sync 默认只读；不足时 design delta + 单写窗口 |
| ST-AW-004 | aggregate evidence gate | W3 | dev-ready | PASS / CP5 approved | full-lld | open | meta-dev | 与 ST-AW-003 primary 隔离，CLI/shared 由本 Story 单写 |
| ST-AW-005 | read-only migration handoff | W4 | lld-approved | PASS / CP5 approved | technical-note | blocked-by-runtime | meta-dev | ST-AW-003/004 必须 verified；mutation 增量会升级 full-lld |

ST-AW-001 已在 CP7-R2 以 `PASS_WITH_RISK` 收敛，F01/F02 CLOSED；W2 的 ST-AW-002 现已解锁，但只允许临时 fixture/local bare remote，不授权真实 worktree/ref/remote mutation。

## CR-051 并行队列

| 队列 | Story | 依据 |
|---|---|---|
| lld_ready | — | 5/5 设计证据已生成 |
| lld_running | — | 三个 meta-dev lane 已交还 |
| lld_review | — | CP5 已批准，设计批次关闭 |
| lld_batch_review | — | `CP5-CR051-DQ-01` 已接受 |
| dev_ready | ST-AW-003, ST-AW-004 | ST-AW-002 verified；W3 contract/runtime gates 满足 |
| dev_running | — | 未进入实现 |
| verify_ready | — | ST-AW-001 已完成重验 |
| verified_with_risk | ST-AW-001, ST-AW-002 | 剩余真实/平台 pilot 风险进入 CP8 |
| needs_rework | — | 无 |
| needs_design_clarification | — | 无 |
| blocked_by_dependency | ST-AW-005 | 等待 ST-AW-003/004 verified |

## CR-051 Story 检查点结果

| Story ID | CP5 设计证据可实现性 | CP5 人工确认 | CP6 编码完成 | CP7 验证完成 | 说明 |
|---|---|---|---|---|---|
| ST-AW-001 | `process/checks/CP5-CR051-ST-AW-001-LLD-IMPLEMENTABILITY-R2.result.json` | `process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md` | `process/checks/CP6-CR051-ST-AW-001-project-first-routing-CODING-DONE.result.json` | `process/checks/CP7-CR051-ST-AW-001-project-first-routing-VERIFICATION-DONE.result.json` | NEEDS_REWORK；F01/F02 |
| ST-AW-002 | `process/checks/CP5-CR051-ST-AW-002-LLD-IMPLEMENTABILITY.result.json` | 同上 | `process/checks/CP6-ST-AW-002-recoverable-project-worktree-CODING-DONE.md` | `process/checks/CP7-ST-AW-002-recoverable-project-worktree-VERIFICATION-DONE.md` | PASS；O-AW-01/02 强制证据 |
| ST-AW-003 | `process/checks/CP5-CR051-ST-AW-003-LLD-IMPLEMENTABILITY.result.json` | 同上 | `process/checks/CP6-ST-AW-003-heterogeneous-git-legs-CODING-DONE.md` | `process/checks/CP7-ST-AW-003-heterogeneous-git-legs-VERIFICATION-DONE.md` | PASS；CP5 approved，等待 W3/runtime gate |
| ST-AW-004 | `process/checks/CP5-CR051-ST-AW-004-LLD-IMPLEMENTABILITY.result.json` | 同上 | `process/checks/CP6-ST-AW-004-aggregate-evidence-gate-CODING-DONE.md` | `process/checks/CP7-ST-AW-004-aggregate-evidence-gate-VERIFICATION-DONE.md` | PASS；CP5 approved，等待 W3 调度 |
| ST-AW-005 | `process/checks/CP5-CR051-ST-AW-005-TECHNICAL-NOTE-IMPLEMENTABILITY.result.json` | 同上 | `process/checks/CP6-ST-AW-005-read-only-migration-handoff-CODING-DONE.md` | `process/checks/CP7-ST-AW-005-read-only-migration-handoff-VERIFICATION-DONE.md` | PASS；CP5 approved，等待 W4/runtime gate |

## CR-051 Wave 进度

| Wave | 总数 | lld-ready | lld-review | dev-ready | in-dev | verified | verified-with-risk | needs-rework | needs-design-clarification | blocked |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| W1 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |
| W2 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |
| W3 | 2 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | 0 | 0 |
| W4 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |

## CR-051 阻塞与开放项

| ID | Story | 分类 | 状态 | 路由 |
|---|---|---|---|---|
| O-AW-01 | ST-AW-002 | CP5 evidence obligation | non-blocking-open | 证明失败则 auto_switch=disabled |
| O-AW-02 | ST-AW-002 | CP5 evidence obligation | non-blocking-open | persistence fault 前 BLOCKED |
| O-AW-03 | ST-AW-005 | follow-up tracking | non-blocking-open | 阈值命中只建 CR candidate |

当前 blocking clarification 数为 0；CP5 已批准。O-AW-01/02 不是 waiver，必须在 ST-AW-002 的 CP6/CP7 执行 CAP/DUR 证明；`real-remote-unverified` 保留到 CP8 风险接受。
