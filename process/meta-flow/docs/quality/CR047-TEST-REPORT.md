---
cr_id: "CR-047"
checkpoint: "CP7"
status: "PASS_WITH_RISK"
tested_at: "2026-07-15"
---

# CR-047 Test Report

| 门 | 命令/范围 | 结果 |
|---|---|---|
| Full regression | pytest 全仓 | 400 passed + 70 subtests，exit 0 |
| Static quality | `ruff check .` | 0 errors，exit 0 |
| Delivery guardrail | repository guardrail | exit 0；15 个 ignored local cache directory warnings |
| Doctor | workspace/token/context/artifact/quality/workflow | exit 0；observed=21，blocking=0，unclassified=0 |
| CR tracking | State v2 + JSON index + CURRENT | exit 0；CR-047 active，CR-033 candidate |
| Installer matrix | codex/claude/qoder dry-run | 3/3 exit 0；真实写入=0 |
| Evidence firewall | CR-046 object identities | 57/57 match；findings=0 |
| Diff hygiene | `git diff --check` | exit 0 |

测试未访问 credentials、SaaS、外部 runtime、production write/publish/trading，也未 commit/push。guardrail/Doctor 的 warning 被保留为风险证据，而非转写为无风险 PASS。
