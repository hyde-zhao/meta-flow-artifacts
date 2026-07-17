---
checkpoint_id: "CP8-CR031-PROMPT-CONTRACT"
checkpoint_name: "提示词契约交付就绪"
type: "auto_then_manual"
status: "pending"
owner: "host-orchestrator"
created_at: "2026-07-17T08:25:00Z"
reviewed_by: ""
reviewed_at: ""
auto_check_result: "process/checks/CP8-CR031-PROMPT-CONTRACT.result.json"
target:
  phase: "documentation"
  story_id: ""
  artifacts:
    - "agents/ptm-tse.md"
    - "skills/itr-ticket-ingestion/SKILL.md"
    - "skills/reverse-analysis/SKILL.md"
    - "skills/improvement-tracker/SKILL.md"
---

# CP8 人工审查 - CR-031 提示词契约交付就绪

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP8-CR031-PROMPT-CONTRACT.result.json` | PASS | 0 | 运行根、预检、权限、输出位置和显式本地授权边界已在四份提示词中一致定义。 |
| 源码根工件清理 | PASS | 0 | 未跟踪 `data/`、构建/虚拟环境/缓存、旧 checkpoint/迁移副本和空 `.input/` 已删除；已跟踪源码、`process` 软链接及安装项目均未触碰。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 交付 ptm-tse Agent 和三个 Skill 的运行数据治理提示词，并清除 ptm-team 源码根不应保留的未跟踪本地工件。未来安装项目可用自己的 `<runtime-root>/data/` 受控处理运行数据。 |
| 推荐动作 | `approve`：接受四份源提示词的静态交付和本地工件清理结果。 |
| approve 后会发生什么 | CR-031 的提示词与源码根清理交付关闭；安装后的 ptm-tse 项目可应用新提示词，并在其自身获得明确本地授权后执行实际数据治理。 |
| approve 不授权什么 | 不授权读取、修改或修复 `/home/hyde/projects/ptm-tse/data/`；不授权 ITR 请求、凭据、外部写入、生产操作、发布或自动 reviewer 动作。 |
| 不确认会阻塞什么 | 阻塞 CR-031 的提示词交付关闭；已安装项目不受影响。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP8-CR031-PROMPT-CONTRACT.context.json` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 只读取 CR 摘要、CP2 结果和四份目标提示词；不读取已安装项目 data。 |
| 全文档读取扩展 | 1 次，历史 CR-031 范围核对，引用 `RE-20260717T073117Z0000-e7403009`。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.current.json` / gate ledger | scanned | 0 | 0 | 无未决人工问题。 |
| 用户直接范围决定 | 当前对话 / CP2-CR031-PROMPT-CONTRACT | scanned | 1 | 0 | 已在 CP2 记录为已决。 |
| 自动预检 | `process/checks/CP8-CR031-PROMPT-CONTRACT.result.json` | scanned | 0 | 0 | 无 FAIL 或 WAIVED。 |
| 下游安装项目 | `/home/hyde/projects/ptm-tse/data/` | n/a | 0 | 0 | 用户明确排除；本轮不读写。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 0 | 无新增选择题。 |
| 高风险策略确认 | 0 | 所有真实运行行为均明确不在授权范围。 |
| agent 默认处理 | 1 | `runtime_root` 默认安装项目根，`data_root` 为 `<runtime-root>/data/`。 |
| 仅审计记录 | 2 | 静态检查通过；运行时验证和安装验证均 N/A。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 范围已确认 | PASS | `process/checkpoints/CP2-CR031-PROMPT-CONTRACT.md` | 用户范围已记录。 |
| 四份目标提示词已更新 | PASS | CP8 result | 交付对象完整。 |
| 未访问安装项目运行数据 | PASS | CR-031 不授权范围 | 与用户范围一致。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 运行根不回退到 ptm-team、全局目录或 CWD | PASS | 四份提示词治理契约 | 一致。 |
| 2 | 敏感数据和 support 文件分类明确 | PASS | Agent、ingestion Skill | 一致。 |
| 3 | 0700/0600、预检阻断和显式本地授权明确 | PASS | 四份提示词治理契约 | 一致。 |
| 4 | 分析与改进 Skill 不执行本地修复 | PASS | reverse-analysis、improvement-tracker | 一致。 |
| 5 | 不触碰已安装 data | PASS | CR-031、CP8 result | 一致。 |
| 6 | 不需要的源码根本地目录已移除且保留真实 `process` | PASS | CP8 result | 已核验。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 自动预检无未豁免失败 | PASS | CP8 result | 可发起人工终验。 |
| 用户终验确认 | pending | 本文件 | `approve` 后关闭提示词交付。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| ptm-tse Agent 契约 | `agents/ptm-tse.md` | PASS | 运行根和跨 Skill 门控。 |
| 摄取 Skill 契约 | `skills/itr-ticket-ingestion/SKILL.md` | PASS | 合规预检和新数据写入边界。 |
| 分析 Skill 契约 | `skills/reverse-analysis/SKILL.md` | PASS | 只读分析与受限报告输出。 |
| 改进 Skill 契约 | `skills/improvement-tracker/SKILL.md` | PASS | 受限输出与非修复边界。 |
| 源码根本地工件清理 | `data/`、构建/缓存、旧过程副本、空 `.input/` | PASS | 全部未跟踪；已删除并保留交付源、真实 `process` 与安装项目。 |
| Release context | `process/release/RELEASE-CONTEXT-CR031.yaml` | PASS | minimal / READY。 |

## 人工审查结果

待用户确认。回复 `approve` 接受提示词交付并关闭 CR-031；回复 `修改: <具体修改点>` 保持门禁 pending 并按指定内容调整。
