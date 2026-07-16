---
cr_id: "CR-050"
checkpoint: "CP7"
status: "PASS_WITH_RISK"
tested_at: "2026-07-16"
validation_mode: "local-deterministic"
---

# CR-050 Test Report

## 结论

CR-050 的四个 Story 均完成 CP6，并以 `PASS_WITH_RISK` 完成 CP7。17/17 个 TC-GB 场景已映射到 13 个临时 bare-remote fixture；真实 project/artifact 远端没有发生 branch create、push、default write 或 delete。

## 质量门

| 门 | 命令 / 范围 | 结果 |
|---|---|---|
| Full regression | 全仓 pytest | `447 passed, 70 subtests passed` |
| CR-050 lifecycle | `tests/test_git_branch_lifecycle.py` | 13/13 passed；TC-GB-001..017 为 17/17 |
| Legacy Git regression | workspace sync + CR lifecycle | 36/36 passed |
| Static quality | `ruff check .` | 0 errors |
| Diff hygiene | `git diff --check` | 0 errors |
| Delivery guardrail | repository guardrail | OK；仅 ignored local cache warning |
| Doctor | workspace/token/context/artifact/quality/workflow | OK；21 个历史超预算对象均已分类，blocker=0、unclassified=0 |
| State / route | workspace check + State v2 enforce + CR tracking | OK；active CR=CR-050，route health=ok |
| Event ledgers | checkpoint + gate | OK；R2 event IDs 唯一 |
| Installer matrix | Codex / Claude / Qoder project dry-run | 3/3 passed；真实目标写入=0 |

## Story 结果

| Story | 主要证明 | 结论 | 当前结果 |
|---|---|---|---|
| ST-GB-001 | 安全 ref 语法、fresh remote default exact OID、双仓 open、dry-run/dirty/drift fail-closed | PASS_WITH_RISK | `CP7-CR050-ST-GB-001-VERIFICATION-DONE-R2.result.json` |
| ST-GB-002 | 仅发布调用前已有 commit、captured OID、matching evidence、零 implicit commit | PASS_WITH_RISK | `CP7-CR050-ST-GB-002-VERIFICATION-DONE-R2.result.json` |
| ST-GB-004 | typed authz、artifact→project ordinary FF push、2/2 projection firewall、PARTIAL resume | PASS_WITH_RISK | `CP7-CR050-ST-GB-004-VERIFICATION-DONE-R2.result.json` |
| ST-GB-003 | fresh identity/tip/ancestry、recovery-first、artifact→project delete、PARTIAL resume | PASS_WITH_RISK | `CP7-CR050-ST-GB-003-VERIFICATION-DONE-R2.result.json` |

## 未验证边界

- 没有独立 meta-qa dispatch/receipt；验证由用户授权的 host inline fallback 完成。
- 没有真实托管远端、branch protection、merge queue 或凭据环境试运行。
- squash/rebase merge 没有 trusted forge receipt 时按设计 fail closed。
- `meta-flow cr check` 仍报告 CR-001..033 的历史 index schema 债务；canonical `meta-flow check cr-tracking` 已通过，CR-050 未修改这些历史对象。
