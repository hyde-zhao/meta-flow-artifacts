---
checkpoint_id: "CP2-CR049-BRANCH-RETIREMENT-SCOPE"
checkpoint_name: "CR-049 Branch Retirement Scope"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-15T15:01:32Z"
reviewed_by: "user"
reviewed_at: "2026-07-15"
auto_check_result: "process/checks/CP2-CR049-BRANCH-RETIREMENT-SCOPE.result.json"
context_ref: "process/context/CP0-CR049.context.json"
---

# CP2 CR-049 Branch Retirement Scope

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR049-BRANCH-RETIREMENT-SCOPE.result.json` | PASS | 0 | 删除集合已固定为两个已合并辅助分支，tag-before-delete 为强制顺序 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 在保留可验证 Git 历史的前提下退役 CR-048 已合并辅助分支 |
| 推荐动作 | 为两个精确提交创建 annotated archive tag，远端核验后仅删除两个辅助分支 |
| approve 后会发生什么 | 推送两个 archive tag；核验 peeled commit；删除两个远端辅助分支及存在的同名本地分支；记录证据 |
| approve 不授权什么 | 移动/删除 main、force-push、覆盖/删除 tag、删除其他分支、quant-lab/prelink/history rewrite |
| 不确认会阻塞什么 | 两个辅助分支继续保留在远端，不执行任何删除 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP0-CR049.context.json` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 只读取 CR-049、route plan、Git refs 与本检查点 |
| 全文档读取扩展 | 0 |
| 缺失 / waived 理由 | CP2 复用 CP0 最小 capsule；本 CR 不改变产品基线 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` | scanned | 0 | 0 | 启动前无 active/blocked CR |
| 自动预检结果 | `process/checks/CP0-CR-049-BOOTSTRAP.result.json` | scanned | 0 | 0 | PASS |
| Git 远端 refs | `origin/main` 与两个 CR-048 辅助分支 | scanned | 1 | 1 | 删除具有外部状态影响，纳入 CP2-DQ-01 |
| 用户显式指令 | 当前会话 | scanned | 1 | 1 | 用户明确要求“归档打 tag，该删除的删除” |
| 产品/架构/Story 产物 | N/A | n/a | 0 | 0 | 本 CR 不改变产品、设计或代码 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 1 | CP2-DQ-01，已由当前用户指令批准 |
| 高风险策略确认 | 0 | 无 force-push/history rewrite/tag overwrite |
| agent 默认处理 | 1 | 标签命名、peeled ref 和祖先关系机械校验 |
| 仅审计记录 | 1 | CR-048 merge DAG 与精确提交 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-DQ-01 | implementation | 是否先以 annotated tag 归档两个已合并分支，再删除对应本地/远端 branch refs | 接受 CR-049 的两个固定 tag 和两个固定 branch ref；tag 远端核验通过后才删除 | 暂缓，保留两个远端分支不做任何变更 | 推荐方案减少长期分支噪声且由 tag/merge DAG 保留历史；备选零变更但保留冗余 ref | 误删风险通过精确 ref、tag-before-delete、main hash/祖先校验控制 | 任一 ref 漂移、tag 冲突或远端核验失败即停止；删除后可从 archive tag 恢复同名分支 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| process 路由健康 | PASS | `meta-flow workspace check` | symlink 指向 artifact repo 的 `process/meta-flow` |
| 当前无 active/blocked CR | PASS | `meta-flow check cr-tracking` 启动前结果 | 仅 CR-033 candidate，不占执行锁 |
| 精确 refs 已核验 | PASS | `git ls-remote --heads origin` | main/integration=`ad3f6f42...`，fix=`d266e056...` |
| archive tag 未占用 | PASS | `git ls-remote --tags origin refs/tags/archive/*` | 空 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 两个 tag 均为 annotated tag 且目标固定 | PASS | `process/changes/CR-049.md` | 禁止覆盖既有 tag |
| 2 | 删除只发生在两个已列明分支 | PASS | `process/changes/CR-049.md` | main 与其他 ref 均不授权 |
| 3 | tag 必须先推送并远端核验 | PASS | CR-049 执行顺序 | 校验失败即阻断 |
| 4 | 用户授权含义未扩大 | PASS | 当前会话指令 | 不含 force-push、tag 删除或 quant-lab |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP2-DQ-01 有明确用户结论 | PASS | 当前用户指令 | approved |
| CP6 可执行边界唯一 | PASS | 两个 tag + 两个 branch ref | 任何漂移均停止 |
| 回退路径可操作 | PASS | 从 archive tag 恢复 branch | 不修改 tag 或历史 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR-049 | `process/changes/CR-049.md` | PASS | 精确范围与禁区 |
| Route plan | `process/checks/CP0-CR-049.route-plan.json` | PASS | process-lite |
| CP2 result | `process/checks/CP2-CR049-BRANCH-RETIREMENT-SCOPE.result.json` | PASS | 机器结果 |

## 人工审查结果

- 结论：`approved`
- 审查人：用户
- 审查时间：2026-07-15
- 决策来源：当前会话用户明确要求“你合并完了了，不需要的分支该归档大tag的打tag，该删除的删除”。
- 记录语义：本文件记录已发生的用户授权；`created_at` 是审计文件生成时间，不伪造原始消息时间戳。
- 接受项：CP2-DQ-01 推荐方案。
- 不授权项：main 移动/删除、force-push/history rewrite、tag 覆盖/删除、其他分支删除、quant-lab/prelink、CR-048 原件改写。
