---
change_id: CR-023-process-routing-and-agent-projection-boundary
title: process 外置路由与 meta-flow Agent 本地投影边界澄清
status: implemented
implementation_status: implemented
workflow_mode: fast-lane
created: 2026-06-23
approved: 2026-06-23
impact_level: medium
rollback_to: process local-directory backup process.before-artifact-routing-20260623
source: user
owner: host-orchestrator
---

# CR-023 - process 外置路由与 meta-flow Agent 本地投影边界澄清

## 变更描述

用户确认实施轻量整改：

- 将当前项目 `process/` 运行态过程文档复制归档到 `/home/hyde/projects/meta-flow-artifacts/process/ptm-team`。
- 将 `/home/hyde/projects/ptm-team/process` 替换为相对软链接 `../meta-flow-artifacts/process/ptm-team`。
- 在 `AGENTS.md` / `CLAUDE.md` 标注 `.claude/agents/*` 与 `.codex/agents/*` 是本地平台投影，不是当前仓库的 meta-flow Agent canonical source。

## CR 冲突预检

| 项 | 结论 |
|---|---|
| 当前 active CR | `CR-018` |
| 是否重叠 | 与 CR-018 的流程合规治理方向相关，但本 CR 只处理过程文档路由和说明性边界，不修改 ptm-tde 业务流程语义 |
| 并行推进依据 | 用户直接确认实施整改；本 CR 不修改 Agent/Skill 运行逻辑和交付安装器 |
| 冲突处理 | 保留 CR-018 状态不变；本 CR 作为轻量治理切片落地 |

## 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 | 批准状态 |
|---|---|---|---|
| `process/` | 外置路由 | 原目录保留为 `process.before-artifact-routing-20260623`，artifact 侧保留完整复制 | approved |
| `process/.meta-flow-process.yaml` | 新增 | 不适用 | approved |
| `AGENTS.md` | 原文档更新 | 保留既有角色职责，仅修正 source/projection 边界 | approved |
| `CLAUDE.md` | 原文档更新 | 保留既有执行规则，仅补充 source/projection 边界 | approved |
| `.gitignore` | 原文档更新 | 保留 `process/*` 忽略规则，新增本地迁移备份忽略 | approved |

## 五维度影响分析

| 维度 | 结论 | 说明 |
|---|---|---|
| 需求层 | 无业务需求变更 | 不改变 ptm-tde 功能或输出契约 |
| 场景层 | 无测试场景变更 | 不改变 KYM/MFQ/PPDCS 流程语义 |
| 计划层 | 轻量治理变更 | 只改变运行态过程文档存储位置和规则说明 |
| 安全层 | 低风险 | 不引入网络、凭据、真实运行或发布授权 |
| 交付层 | 中等影响 | `process` 路径保持不变，但真实存储转为 artifact repo |

## 实施结果

| 对象 | 结果 |
|---|---|
| `/home/hyde/projects/meta-flow-artifacts/process/ptm-team` | 已创建并复制当前 `process/` 完整内容 |
| `/home/hyde/projects/ptm-team/process` | 已替换为相对软链接 `../meta-flow-artifacts/process/ptm-team` |
| `process/.meta-flow-process.yaml` | 已写入路由元数据 |
| `process.before-artifact-routing-20260623` | 保留本地备份目录 |
| `AGENTS.md` / `CLAUDE.md` | 已标注 `.claude/.codex` 是本地平台投影，canonical source 暂未纳入当前仓库 |

## 验收标准

- [x] `process` 是软链接。
- [x] `process/STATE.md` 可通过软链接读取。
- [x] artifact 侧文件数与迁移前一致。
- [x] `diff -qr` 复制校验无差异。
- [x] `.meta-flow-process.yaml` 可读取，且 `project_name=ptm-team`。
- [x] `AGENTS.md` / `CLAUDE.md` 不再把 `agents/meta-*.md` 表述为当前仓库真实 canonical source。

## 回滚方式

如需回滚到本地目录模式：

1. 删除软链接 `process`。
2. 将 `process.before-artifact-routing-20260623` 改名回 `process`。
3. 将 `process/.meta-flow-process.yaml` 或 `STATE.md.artifact_routing` 标记为 `routing_mode=local-directory`。
