---
checkpoint_id: "CP2-CR074-ROOT-CUTOVER-BASELINE"
checkpoint_name: "CR074 Root Cutover Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-16T22:14:44+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T22:41:34+08:00"
auto_check_result: "process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md"
context_capsule: "process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md"
---

# CP2 CR074 Root Cutover Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | PASS | 0 | CR074 范围明确：logical root cutover，批准前不执行。 |

## Decision Brief

推荐决策：批准 CR074 CP2 范围基线，把本轮限定为“逻辑 root cutover / daily workflow switch”门禁。CP2 approve 只确认范围，不直接切换当前会话、不删除旧根、不恢复 env/data/reports。

备选方案：保持 CR074 candidate；继续停留旧根；扩大为物理目录切换。推荐方案优先，因为 CR073 已使 target 达到 readiness，下一步应先审查 root authority，而不是直接执行破坏性动作。

影响维度：后续会话默认工作根、STATE / CR-INDEX root authority 语义、旧根保留策略；不影响 runtime、凭据、远端 Git 或 CR046。

风险与回退：若 reject，则不启动 root cutover；若 approve 后需要回退，旧根仍保留，可通过后续 CR 切回 authoritative marker。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Scenario Gray Areas N/A：CR074 来源为 CR073 follow-up candidate，用户已回复“继续”。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 创建 CR074 前无当前 gate DQ；本文件创建后回写。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |
| 自动预检结果 | `process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | scanned | 5 | 5 | root cutover 范围、旧根、禁止项、风险纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 用户直接要求继续；无新增 SGQ。 |
| 下游正式产物 | CR074 formal CR / CR073 verification | scanned | 5 | 5 | CR074 scope 和 CR073 readiness 纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | “继续”解释为启动 CR074 candidate；实际切换仍需审批。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR074-01 | scope | 是否正式启动 CR074 root cutover / daily workflow switch？ | 启动为正式 active gate。 | 保持 candidate；取消 CR074。 | 推荐方案承接 CR073 readiness；candidate 会继续停留旧根；取消会保留 target 不可用作日常入口。 | 更新 STATE / CR-INDEX，不触发切换。 | reject 时不切 root。 |
| DQ-CP2-CR074-02 | scope | 本轮 cutover 是否限定为 logical root authority，而非物理目录操作？ | 仅做 logical root authority。 | 物理移动/重命名；系统级 symlink。 | 推荐方案可回退；备选有破坏性和平台副作用。 | 后续会话入口语义变化。 | 需要系统级动作时另起 ops CR。 |
| DQ-CP2-CR074-03 | security | 是否确认旧根保留且 forbidden paths 仍不触碰？ | 保留旧根，不复制/读取 reports/data/.venv/node_modules/.env。 | 允许迁移部分 forbidden paths；允许删除旧根。 | 推荐方案保留回退路径；备选扩大敏感/破坏面。 | target 仍可能非 runtime-ready。 | CR075/CR076/CR077 分别处理。 |
| DQ-CP2-CR074-04 | runtime_authorization | CP2 approve 是否授权 env rebuild、远端 Git、data lake、QMT/MiniQMT 或 CR046 recovery？ | 不授权。 | 合并某类外部动作；恢复 CR046。 | 推荐方案聚焦 root authority；备选引入外部副作用。 | 防止越权。 | 用户另起 runtime_authorization 时切换。 |
| DQ-CP2-CR074-05 | risk_acceptance | 是否接受 target 作为 authoritative root 但仍非 runtime-ready？ | 接受；env/data/reports 后续 CR。 | 等 CR075/CR076 后再 cutover。 | 推荐方案更快完成工作根切换；备选更保守但延后。 | 后续运行仍需单独验证。 | 若需要运行保障，先推进 CR075/CR076。 |

### 用户需决策事项 summary

| 决策 ID | 用户需决策事项 |
|---|---|
| DQ-CP2-CR074-01 | 是否启动 CR074 为正式 active gate。 |
| DQ-CP2-CR074-02 | 是否确认本轮只做 logical root authority。 |
| DQ-CP2-CR074-03 | 是否确认旧根保留且 forbidden paths 不触碰。 |
| DQ-CP2-CR074-04 | 是否确认 CP2 approve 不授权 env/runtime/remote/data lake/CR046。 |
| DQ-CP2-CR074-05 | 是否接受 target authoritative 但仍非 runtime-ready。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR074 formal CR 存在 | approved | `process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md` | 用户回复“同意”，接受 CP2 推荐方案。 |
| CP2 自动预检 PASS | approved | `process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | 自动预检 PASS。 |
| CR073 readiness PASS | approved | `process/checks/CR073-TARGET-CUTOVER-READINESS-VERIFICATION-2026-06-16.md` | 执行前已重跑 target preflight。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否启动 CR074 | approved | DQ-CP2-CR074-01 | 接受推荐方案。 |
| 2 | 是否限定 logical cutover | approved | DQ-CP2-CR074-02 | 接受推荐方案。 |
| 3 | 旧根与 forbidden paths 边界 | approved | DQ-CP2-CR074-03 | 接受推荐方案。 |
| 4 | 外部动作是否继续不授权 | approved | DQ-CP2-CR074-04 | 接受推荐方案。 |
| 5 | runtime-ready 风险是否接受 | approved | DQ-CP2-CR074-05 | 接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 用户回复“同意” | 按 `approve` 处理。 |
| CP2 approve 前不执行 root cutover | approved | 无 CP2 前 cutover evidence | CP2 前未执行。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck | `process/checks/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | approved |  |
| CP2 context | `process/context/CP2-CR074-ROOT-CUTOVER-CONTEXT.yaml` | approved |  |
| CP2 manual checkpoint | `process/checkpoints/CP2-CR074-ROOT-CUTOVER-BASELINE.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-16T22:41:34+08:00
- 修改意见：无；用户回复“同意”，接受 DQ-CP2-CR074-01..05 推荐方案。
- 风险接受项：接受 target 作为后续 authoritative root 但仍非 runtime-ready；env/data/reports/runtime 继续由后续 CR 单独授权。
