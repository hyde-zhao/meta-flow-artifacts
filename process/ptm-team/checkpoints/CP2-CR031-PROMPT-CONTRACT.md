---
checkpoint_id: "CP2-CR031-PROMPT-CONTRACT"
checkpoint_name: "提示词契约范围确认"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-17T08:20:00Z"
reviewed_by: "user"
reviewed_at: "2026-07-17"
approval_source: "Current conversation: ptm-team only defines the Agent/Skill rules; installed ptm-tse owns later local data governance."
auto_check_result: "process/checks/CP2-CR031-PROMPT-CONTRACT.result.json"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "agents/ptm-tse.md"
    - "skills/itr-ticket-ingestion/SKILL.md"
    - "skills/reverse-analysis/SKILL.md"
    - "skills/improvement-tracker/SKILL.md"
---

# CP2 人工审查 - CR-031 提示词契约范围

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR031-PROMPT-CONTRACT.result.json` | PASS | 0 | 范围、运行授权和四方契约边界一致。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 为已安装 ptm-tse 的后续本地运行数据治理提供一致的 Agent/Skill 提示词契约。 |
| 推荐动作 | 采用用户明确指定的提示词交付范围。 |
| approve 后会发生什么 | 仅静态验证四份源提示词并准备 CP8 交付确认。 |
| approve 不授权什么 | 不授权读取、修改或修复 `/home/hyde/projects/ptm-tse/data/`，也不授权安装、DAO、schema、外部请求或发布。 |
| 不确认会阻塞什么 | 阻塞提示词交付；已安装项目保持不变。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 0 | 用户已在当前对话明确范围。 |
| 高风险策略确认 | 0 | 真实运行授权被明确排除。 |
| agent 默认处理 | 1 | 默认 runtime root 为安装项目根，data root 为 `<runtime-root>/data/`。 |
| 仅审计记录 | 1 | 当前 CP2 只固化用户范围决定。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| 用户已明确本轮范围 | PASS | 当前对话 | 不再重复询问已决范围。 |
| CR-031 已收紧为 docs-lite | PASS | `process/changes/CR-031.md` | 不进入 HLD、LLD 或代码实现。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 只修改 Agent/Skill 源提示词 | PASS | CR-031 文档处理决策 | 已确认。 |
| 2 | 已安装 ptm-tse 是本地数据动作 Owner | PASS | 当前对话、Agent 契约 | 已确认。 |
| 3 | 本轮不授权真实数据操作 | PASS | CR-031 不授权范围 | 已确认。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 范围与授权边界无阻断项 | PASS | 本文件、CP2 result | 进入 CP8 静态交付预检。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR 范围基线 | `process/changes/CR-031.md` | PASS | 已收紧。 |
| 四份目标提示词 | `agents/ptm-tse.md`、`skills/*/SKILL.md` | PASS | 进入静态验证。 |

## 人工审查结果

已批准。批准来源见 frontmatter；不包含任何安装项目的数据操作授权。
