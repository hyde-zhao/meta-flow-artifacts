---
cr_id: "CR-050"
checkpoint: "CP7"
decision: "PASS_WITH_RISK"
validation_mode: "local-deterministic"
verified_at: "2026-07-16"
verified_by: "host-orchestrator-inline/meta-qa"
independent_qa_attestation: "unavailable"
---

# CR-050 Verification Report

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-16 | host-orchestrator-inline/meta-qa | 完成 ST-GB-001 branch-open 的本地 deterministic fixture 验证；其余 Story 待按 DAG 验证。 |
| 0.2 | 2026-07-16 | host-orchestrator-inline/meta-qa | 完成 ST-GB-002 committed-ref publish 验证；确认 dirty-tree 零写和无隐式 commit。 |
| 0.3 | 2026-07-16 | host-orchestrator-inline/meta-qa | 完成 ST-GB-004 paired fast-forward/投影 firewall 验证；PARTIAL 不推进 paired truth。 |
| 0.4 | 2026-07-16 | host-orchestrator-inline/meta-qa | 完成 ST-GB-003 fresh proof/recovery/cleanup 验证；四个 Story 均进入 PASS_WITH_RISK。 |
| 0.5 | 2026-07-16 | host-orchestrator-inline/meta-qa | 纳入安全回修 R2：unsafe ref allowlist、open/finish fresh reproof；全量 447 tests + 70 subtests、Ruff、Doctor、guardrail 通过。 |

## Story 追踪

| Story | 验证对象 | 自动化证据 | 结论 |
|---|---|---|---|
| ST-GB-001 | typed open、exact base/upstream、unsafe/dirty/default/ref fail-closed、dry-run | 5 open/name/freshness fixtures + 36 legacy regression + Ruff/diff | PASS_WITH_RISK |
| ST-GB-002 | captured OID publish、matching evidence、dirty 零写 | 2 publish fixtures + 36 legacy regression + CLI/Ruff/diff | PASS_WITH_RISK |
| ST-GB-004 | typed authz、artifact-first exact FF、2/2 projection firewall、PARTIAL resume | 2 merge/projection fixtures + 36 legacy regression + CLI/Ruff/diff | PASS_WITH_RISK |
| ST-GB-003 | current projection、fresh identity/tip/ancestry、recovery-first、artifact→project cleanup/resume | 4 finish/freshness fixtures + 36 legacy regression + CLI/Ruff/diff | PASS_WITH_RISK |

## ST-GB-001 分层结果

- Unit/integration：临时 bare remote 上 4 个 open/name 场景通过；2/2 exact OID/upstream、dirty preflight 和 dry-run 零 mutation 均有断言。
- Regression：`tests/test_workspace_git_sync.py` 与 `tests/test_cr_lifecycle.py` 共 36 个测试通过；旧 bootstrap/workspace push 默认行为未变。
- Static：touched Python path Ruff 0；`git diff --check` 0。
- 权限：测试 remote 全部位于 pytest 临时目录；当前 project/artifact 的真实 branch create/push 数为 0。

## 最终质量门

- 全量 pytest：`447 passed, 70 subtests passed`。
- CR-050 lifecycle fixture：13/13 passed，TC-GB-001..017 映射 17/17。
- Ruff：0 error；delivery guardrail：OK；Doctor：OK（21 个历史超预算对象全部已分类为 warning，active/default-required blocker=0，unclassified=0）。
- 真实 project/artifact remote mutation：0；commit/push/default write/branch delete：0。

## 当前风险

验证由用户授权的 inline fallback 完成，不是独立 meta-qa；真实 branch protection、凭据、平台 receipt 与跨设备试运行未授权。四个 Story 的结论上限均为 `PASS_WITH_RISK`，不能声称 platform-attested、independent-QA-verified 或 production-ready。Squash/rebase 在没有 trusted receipt 时按设计 fail closed。
