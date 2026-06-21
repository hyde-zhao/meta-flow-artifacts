---
checkpoint_id: "CP2-CR109-DOCS-FUNCTIONAL-ORG-REVIEW"
checkpoint_name: "CR109 Docs Functional Organization Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-21T22:26:34+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-21T22:43:00+08:00"
auto_check_result: "process/checks/CP2-CR109-DOCS-FUNCTIONAL-ORG-SCOPE.md"
context_capsule: "process/context/CP2-CR109-DOCS-FUNCTIONAL-ORG-CONTEXT.yaml"
---

# CP2 CR109 Docs Functional Organization 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR109-DOCS-FUNCTIONAL-ORG-SCOPE.md` | PASS_WITH_RISK | 0 | CR108 未完成目录功能化，CR109 范围成立。 |

## Decision Brief

推荐决策：批准 CR109 按功能能力重组 `docs/design`、`docs/features`、`docs/quality`。顶层 current-facing 入口按功能域保留；CR-scoped 历史材料迁入功能域下 `evidence/by-cr/` 或 `docs/quality/by-feature/<feature>/by-cr/`，并生成 migration map。

备选方案：只新增功能索引但不移动文件；或只移动 `docs/features`，`docs/design` / `docs/quality` 后续再做。

影响维度：文档路径、历史引用、后续 Agent 默认读取入口、质量证据索引、CR 审计追溯。

风险与回退：物理移动可能导致旧路径引用失效；必须先产出 migration map，保留历史正文，不改业务代码。若引用修复范围过大，先移动顶层 CR 目录到功能域 evidence，CR-scoped design/quality 文件分批处理。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR109-DOCS-FUNCTIONAL-ORG-CONTEXT.yaml` |
| capsule 状态 | ready-for-scope-review |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；需要迁移时读取 docs/design、docs/features、docs/quality |
| 全文档读取扩展 | 0 次 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| User correction | 当前对话 | scanned | 1 | 1 | docs 应按功能组织。 |
| CR109 design | `docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md` | scanned | 3 | 3 | 物理移动、design CR 文件、quality CR 文件均需决策。 |
| Existing dirs | `docs/design`; `docs/features`; `docs/quality` | scanned | 3 | 1 | 当前 CR-number organization 确认。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR109-01 | scope | 是否确认 CR108 未完成 docs 功能化，CR109 承接？ | 确认，由 CR109 承接。 | 重开 CR108；或只记录 follow-up。 | 新 CR 保持历史清晰；重开 CR108 不符合现有工具流；只记录无法修复目录。 | 改变后续 CR 顺序，CR tracking hygiene 顺延。 | 若用户要求保持原计划，则 CR109 可转 tracking，docs 功能化另建 CR110。 |
| DQ-CP2-CR109-02 | implementation | 是否允许物理移动 CR-scoped docs 到功能域 evidence/by-cr？ | 允许，先生成 migration map，再移动。 | 只新增功能索引不移动；只移动 docs/features。 | 推荐方案真正解决目录按 CR 组织；只索引风险低但目录仍旧；只移动 features 范围小。 | 影响旧路径引用，需要迁移映射。 | 若引用修复过大，分批移动。 |
| DQ-CP2-CR109-03 | implementation | 是否允许 `docs/quality` 按 by-feature/by-cr 重组？ | 允许，但保留历史报告正文。 | 暂缓 quality，只建立 README。 | 推荐方案解决质量证据平铺；暂缓风险低但不完整。 | 大量文件移动，需索引。 | 若迁移量过大，先处理 CR091-CR104 近期文件。 |
| DQ-CP2-CR109-04 | risk_acceptance | 是否接受移动文档路径带来的旧引用风险？ | 接受，要求 migration map 和后续静态引用扫描。 | 不接受，只做索引。 | 接受后可达成功能目录目标；不接受则只能部分修复。 | 旧 process/CR 引用可能仍指旧路径。 | 出现大量 broken refs 时停止并转分批迁移。 |

### 用户需决策事项

| 决策 ID | 用户需决策事项 | 当前建议 |
|---|---|---|
| DQ-CP2-CR109-01 | 是否由 CR109 承接 docs 功能化缺口 | approve |
| DQ-CP2-CR109-02 | 是否允许物理移动 CR-scoped docs 到功能域 evidence/by-cr | approve |
| DQ-CP2-CR109-03 | 是否允许 docs/quality 按 by-feature/by-cr 重组 | approve |
| DQ-CP2-CR109-04 | 是否接受旧引用风险并要求 migration map | approve |

### 不授权项

- 不授权读取凭据、`.env`、token、API key、cookie、私钥、账号。
- 不授权 runtime、SaaS、trace upload、production write、真实交易或真实发布。
- 不授权启动 CR-033。
- 不授权关闭或处理 CR102-CR104。
- 不授权业务代码整改。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR109 已创建 | approved | `process/changes/CR-109.md` | 用户已批准继续。 |
| Functional organization design 已创建 | approved | `docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md` | 用户已批准按 migration map 迁移。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否接受 CR109 承接 CR108 未完成项 | approved | DQ-CP2-CR109-01 | 用户回复 `approve`。 |
| 2 | 是否允许物理移动 CR-scoped docs | approved | DQ-CP2-CR109-02 | 用户授权按 CR109 migration map 做功能化目录迁移。 |
| 3 | 是否允许重组 docs/quality | approved | DQ-CP2-CR109-03 | 用户授权继续迁移。 |
| 4 | 是否接受旧引用风险和 migration map 策略 | approved | DQ-CP2-CR109-04 | 用户保留不授权边界。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话：`approve,授权我继续按 CR109 的 migration map 做功能化目录迁移...` | 已批准。 |
| approve 后进入迁移执行 | approved | 不授权项 | 只迁移 docs / process 证据，不改业务代码。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| Functional organization design | `docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md` | approved | 目标结构和迁移规则已批准。 |
| CP2 scope check | `process/checks/CP2-CR109-DOCS-FUNCTIONAL-ORG-SCOPE.md` | approved | 自动预检已接受。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-21T22:43:00+08:00
- 修改意见：按 CR109 migration map 做功能化目录迁移；仍不授权业务代码、凭据、runtime、交易或 CR102-CR104 关闭。
- 风险接受项：接受文档路径迁移带来的旧引用风险，要求 migration map 和静态引用扫描。
