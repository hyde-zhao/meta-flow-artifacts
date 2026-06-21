---
checkpoint_id: "CP2-CR109-DOCS-FUNCTIONAL-ORG-SCOPE"
checkpoint_name: "CR109 Docs Functional Organization Scope"
type: "auto_precheck"
status: "PASS_WITH_RISK"
owner: "host-orchestrator"
created_at: "2026-06-21T22:26:34+08:00"
checked_at: "2026-06-21T22:26:34+08:00"
target:
  phase: "requirement-clarification"
  artifacts:
    - "process/changes/CR-109.md"
    - "docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md"
manual_checkpoint: "process/checkpoints/CP2-CR109-DOCS-FUNCTIONAL-ORG-REVIEW.md"
---

# CP2 CR109 Docs Functional Organization Scope

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 用户指出 CR108 未完成目录功能化 | PASS | 当前对话 | 结论成立。 |
| CR109 已创建 | PASS | `process/changes/CR-109.md` | 正式修复 CR。 |
| functional organization design 已创建 | PASS | `docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md` | 定义目标形态和迁移规则。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | CR108 问题是否确实未完成 | PASS | `find -L docs/features -mindepth 1 -maxdepth 1 -type d` | 顶层仍混有 CR 目录。 |
| 2 | 是否需要新 CR 而非继续 CR108 | PASS | CR108 已关闭；本轮范围是目录结构变更 | 用 CR109 承接，CR tracking hygiene 顺延。 |
| 3 | 是否保持历史审计 | PASS | `docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md` | 移动文件但不改写历史正文。 |
| 4 | 不授权边界 | PASS | CR109 frontmatter / context | 不读凭据、不跑 runtime、不改业务代码。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 可进入 docs functional migration design / execution | PASS_WITH_RISK | 本文件 | 需要先确认物理移动策略。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Functional organization design | `docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md` | PASS | 初版。 |
| CP2 context | `process/context/CP2-CR109-DOCS-FUNCTIONAL-ORG-CONTEXT.yaml` | PASS | compact。 |

## 结论

- 结论：`PASS_WITH_RISK`
- 阻断项：无
- 风险：物理移动 docs 会影响旧引用；必须生成 migration map。
- 下一步：发起 CP2 人工确认，批准后执行功能化迁移。
