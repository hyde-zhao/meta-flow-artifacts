---
checkpoint_id: "CP2-CR070-GIT-GOVERNANCE-BASELINE"
checkpoint_name: "CR070 Git Governance Requirements Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-15T23:48:32+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T00:27:16+08:00"
auto_check_result: "process/checks/CP2-CR070-GIT-GOVERNANCE-BASELINE.md"
context_capsule: "process/context/CP2-CR070-GIT-GOVERNANCE-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "documentation"
  story_id: ""
  artifacts:
    - "process/changes/CR-070-GIT-REMOTE-GOVERNANCE-DEFAULT-BRANCH-CUTOVER-GATE-2026-06-15.md"
    - "process/context/CP2-CR070-GIT-GOVERNANCE-CONTEXT.yaml"
---

# CP2 CR070 Git Governance Requirements Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR070-GIT-GOVERNANCE-BASELINE.md` | PASS_WITH_RISK | 0 | CR070 formal CR、CP2 context 和范围基线已生成；风险来自后续远端治理主题，本轮不执行。 |

## Decision Brief

推荐决策：批准 CR070 作为远端 Git governance / default branch cutover gate 的正式 CR 范围基线，并保持当前本轮执行边界为“只生成本地门禁产物与校验”。CP2 approved 仅表示接受 CR070 的范围、来源和不授权边界，不表示授权任何远端 Git 写动作、branch/default branch 变更或 NAS/data lake/runtime 操作。

备选方案：保持 CR070 为 candidate 不启动；缩小为仅记录 residual risk；扩大为立即执行远端治理。推荐方案优先，因为它能先建立可审计门禁，又不会把文档生成误读为远端写入授权。

影响维度：CR tracking、STATE、CR069 follow-up、远端 Git governance 风险、默认分支 residual risk、安全授权边界。

风险与回退：若用户 reject，回退到 `CR069 closed-current-delivery / CR070 follow-up candidate not started`；若用户要求扩大范围，必须重新生成 CP2/CP3/CP5 Decision Brief。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR070-GIT-GOVERNANCE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前未发现独立队列项；CR070 DQ 从 CR / CP8 / capsule 聚合。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮未调度子 agent；host-orchestrator 生成门禁稿。 |
| 自动预检结果 | `process/checks/CP2-CR070-GIT-GOVERNANCE-BASELINE.md` | scanned | 2 | 2 | 范围与不授权风险纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | CR070 本轮为 follow-up gate，用户请求已明确，无新增 SGQ。 |
| 下游正式产物 | CR070 formal CR / CR069 CP8 | scanned | 5 | 5 | 来源、范围、安全、不授权和 residual risk 纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 2 | 2 | 用户明确要求启动 CR070 formal CR，并禁止远端 Git 写与 NAS 操作。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR070-01 | scope | 是否正式启动 CR070 作为 Git Remote Governance / Default Branch Cutover Gate？ | 启动为 active formal CR，并只生成本地门禁产物。 | 保持 CR070 candidate；取消本候选。 | 推荐方案可审计且不执行；保持候选会延后治理；取消会留下 residual risk。 | 影响 CR tracking 和后续门禁路径。 | reject 时回退为 candidate not started。 |
| DQ-CP2-CR070-02 | follow_up_tracking | CR070 与 CR069 CP8 follow-up 的关系如何记录？ | 记录为 CR069 CP8 第 3 项决策的正式后续 CR。 | 记录为独立用户请求；并入 CR069。 | 推荐方案追溯最清晰；并入 CR069 会重开已关闭交付。 | 影响审计链。 | 用户要求改来源时更新 frontmatter 与索引。 |
| DQ-CP2-CR070-03 | security | CP2 是否继续禁止所有远端 Git 写动作？ | 确认禁止 `remote add/set-url/push/tag`、branch/default branch、history rewrite、force push、remote deletion。 | 允许只读远端检查；允许立即执行部分远端治理。 | 推荐方案符合用户限制；允许远端动作会扩大当前范围。 | 保持外部状态不变。 | 需要远端动作时重新发起授权。 |
| DQ-CP2-CR070-04 | runtime_authorization | CP2 是否继续禁止 NAS/data lake/runtime/credential/CR046 recovery？ | 确认禁止，相关事项均需独立 CR 和授权。 | 合并 NAS 或 runtime；恢复 CR046。 | 推荐方案风险分层；备选越权且扩大影响面。 | 避免外部系统副作用。 | 用户明确启动对应 CR 时切换。 |
| DQ-CP2-CR070-05 | risk_acceptance | 是否接受 default branch / branch naming residual risk 暂不改变？ | 接受；本轮只冻结门禁与不授权边界。 | 立即处理 default branch；完全取消治理。 | 推荐方案可控；立即处理需要远端授权；取消会保留无门禁风险。 | 远端仓库状态不变。 | 用户后续批准 CP3/CP5 和运行授权后再处理。 |

### CP2 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 用户真实意图 | 启动 CR070 formal CR，但本轮只生成门禁产物并运行本地校验。 |
| 场景覆盖 | 覆盖 follow-up 启动、范围冻结、安全不授权、residual risk 保留。 |
| 认知盲区补充 | `approve` 只接受 DQ 推荐方案，不授权远端 Git 或 NAS/runtime 行为。 |
| Scenario Gray Areas 处理结果 | N/A，本轮用户已给出明确边界；无新增场景澄清。 |
| Deferred Ideas | 真实远端执行、default branch 修改、branch rename、NAS/data lake/runtime 均 deferred。 |
| 用户选择影响 | approve 后 CR070 CP2 范围通过，仍不可执行远端动作。 |
| 回退方式 | 回退为 CR070 candidate not started，保留 CR069 closed。 |
| discussion log / checkpoint | N/A，理由见 Decision Collection Coverage。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权 `git remote add`、`git remote set-url`、`git push`、`git tag`。
- 不授权 branch rename、default branch governance、history rewrite、force push 或 remote deletion。
- 不授权 NAS/data lake/provider/catalog/runtime 操作。
- 不授权读取 `.env`、token、账号、密码、cookie、session、private key、SSH agent 细节或交易事实。
- 不授权 physical move、bulk import rewrite 或 CR046 recovery。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR070 formal CR 已创建 | 待审查 | `process/changes/CR-070-GIT-REMOTE-GOVERNANCE-DEFAULT-BRANCH-CUTOVER-GATE-2026-06-15.md` |  |
| CP2 context capsule 已生成 | 待审查 | `process/context/CP2-CR070-GIT-GOVERNANCE-CONTEXT.yaml` |  |
| 自动预检可读 | 待审查 | `process/checks/CP2-CR070-GIT-GOVERNANCE-BASELINE.md` |  |
| 不授权边界已列出 | 待审查 | 本文件 Decision Brief |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 范围是否清楚且有限 | 待审查 | DQ-CP2-CR070-01 |  |
| 2 | 来源追溯是否正确 | 待审查 | DQ-CP2-CR070-02 |  |
| 3 | 远端 Git 写动作是否继续禁止 | 待审查 | DQ-CP2-CR070-03 |  |
| 4 | NAS/runtime/credential 是否继续禁止 | 待审查 | DQ-CP2-CR070-04 |  |
| 5 | residual risk 是否可接受 | 待审查 | DQ-CP2-CR070-05 |  |
| 6 | 自动终验授权是否关闭 | 待审查 | frontmatter `auto_final_authorization: false` |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 本文件人工审查结果 |  |
| approve 不授权外部动作 | 待审查 | 不授权项 |  |
| 若修改范围则重新生成门禁 | 待审查 | 回退 / 切换条件 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR070 formal CR | `process/changes/CR-070-GIT-REMOTE-GOVERNANCE-DEFAULT-BRANCH-CUTOVER-GATE-2026-06-15.md` | 待审查 |  |
| CP2 context capsule | `process/context/CP2-CR070-GIT-GOVERNANCE-CONTEXT.yaml` | 待审查 |  |
| CP2 auto precheck | `process/checks/CP2-CR070-GIT-GOVERNANCE-BASELINE.md` | 待审查 |  |
| CP2 launch message | `process/checks/CP2-CR070-HUMAN-GATE-LAUNCH-MESSAGE.md` | 待审查 |  |

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-16T00:27:16+08:00
- 修改意见：N/A
- 风险接受项：用户回复“同意，继续推进”，按本检查点发起消息解析为接受 DQ-CP2-CR070-01..05 推荐方案；该批准只确认 CR070 CP2 门禁，不授权 `git remote add/set-url/push/tag`、branch rename/default branch governance、history rewrite、force push、remote deletion、NAS/data lake/provider/catalog/runtime/credential、physical move、bulk import rewrite 或 CR046 recovery。
