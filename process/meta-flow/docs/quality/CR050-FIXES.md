---
cr_id: "CR-050"
status: "complete"
updated_at: "2026-07-16"
---

# CR-050 Fixes

| 修复 | 影响文件 | 回归证据 | 结果 |
|---|---|---|---|
| unsafe ref allowlist + Git grammar 双门 | `git_branch_lifecycle.py`、branch tests | canonical/unsafe ref fixture | PASS |
| open fetch 后 exact default OID reproof | lifecycle + branch tests | concurrent default advance fixture | PASS，new refs=0 |
| finish recovery/delete 前 CR identity/tip reproof | lifecycle + branch tests | post-plan CR tip drift fixture | PASS，recovery/delete=0 |
| finish 删除顺序与 resume | ST-GB-003 LLD + lifecycle tests | artifact→project partial resume | PASS |
| commit gap 提示与 CLI 命名 | 四份 LLD | CP5 R2 precheck | PASS |
| CP6/CP7 R2 supersession | 8 个 R2 result + summaries + ledger events | strict result-check + event check | PASS |

修复没有扩大权限：真实远端 ref、commit/push、default write/delete、凭据、forge API、runtime 和 production write 均未执行。
