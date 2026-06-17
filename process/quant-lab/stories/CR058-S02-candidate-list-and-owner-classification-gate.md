---
story_id: "CR058-S02"
status: "verified-with-risk-static-only"
change_id: "CR-058"
title: "Candidate List and Owner Classification Gate"
lld_policy:
  required_level: "full-lld"
feature_design_refs:
  - "FEAT-CR058-02"
dev_gate: "Candidate list schema approved; no repo-local move authorized."
---

# CR058-S02 Candidate List and Owner Classification Gate

## 技术说明

本 Story 定义后续 repo-local relayout candidate list 的必填字段，不生成真实 candidate 文件，不执行移动。

候选项字段：

| 字段 | 必填 | 说明 |
|---|---:|---|
| `path_pattern` | yes | repo-relative exact path 或路径族。 |
| `classification` | yes | `mechanical-candidate` / `manual-review` / `preserve-audit` / `blocked_sensitive` / `keep`。 |
| `owner` | yes | 负责确认的人或角色。 |
| `action_class` | yes | `future_docs_rewrite` / `repo_relayout_candidate` / `keep` / `manual_review` / `block`。 |
| `risk` | yes | `low` / `medium` / `high` / `blocked`。 |
| `verification_rule` | yes | 后续 CP7 或人工复核入口。 |
| `rollback_ref_required` | yes | `true` for any rewrite / move candidate。 |

验收条件：缺任一字段时 candidate 不得进入执行清单。
