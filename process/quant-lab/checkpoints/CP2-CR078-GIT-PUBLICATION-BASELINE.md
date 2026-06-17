---
checkpoint_id: "CP2-CR078-GIT-PUBLICATION-BASELINE"
checkpoint_name: "CR078 Git Publication Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T10:08:12+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T10:30:39+08:00"
auto_check_result: "process/checks/CP2-CR078-GIT-PUBLICATION-BASELINE.md"
context_capsule: "process/context/CP2-CR078-GIT-PUBLICATION-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "documentation"
  story_id: ""
  artifacts:
    - "process/changes/CR-078-REMOTE-GIT-GOVERNANCE-PUBLICATION-GATE-2026-06-17.md"
    - "process/context/CP2-CR078-GIT-PUBLICATION-CONTEXT.yaml"
---

# CP2 CR078 Git Publication Scope Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR078-GIT-PUBLICATION-BASELINE.md` | PASS | 0 | CR078 可作为 publication scope gate 发起；批准前不执行 Git 写动作。 |

## Decision Brief

推荐决策：批准 CR078 正式启动，并把本轮范围限定为 staged `process/**` removals、process ledger pointer files、CR081/CR082 quality / release docs、commit / push 授权边界、远端治理禁区和外部 process project 治理选项的决策。CP2 approve 不授权执行 `git add`、`git commit` 或 `git push`；执行授权只在 CP5 中决策。

备选方案：保持 CR078 为 candidate；合并 CR077 old root retirement；直接提交 / 推送。推荐方案优先，因为它先冻结 scope，避免把旧根退役、数据访问、运行时或远端写入混入同一门禁。

影响维度：Git index、dirty worktree、process ledger ownership、clone recovery、远端发布、安全权限和回退。

风险与回退：若用户 reject，CR078 恢复为 candidate，staged removals 保持不提交；若用户要求扩大范围，需要重做影响分析和 Decision Brief。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR078-GIT-PUBLICATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 无 CR078 既有 pending queue。 |
| CR082 handoff | `process/context/CR082-CLOSURE-TO-CR078-HANDOFF-2026-06-17.md` | scanned | 7 | 7 | 用户列出的 7 类问题全部纳入。 |
| 自动预检结果 | `process/checks/CP2-CR078-GIT-PUBLICATION-BASELINE.md` | scanned | 5 | 5 | staged removals、branch、scope、不授权项纳入。 |
| 下游正式产物 | CR078 formal CR / CR-INDEX | scanned | 5 | 5 | active 转换、rollback、dirty worktree 分类纳入。 |
| 用户显式选择题 | 当前对话 | scanned | 7 | 7 | 用户要求先决策再执行 Git 动作。 |
| discussion log / checkpoint | `process/discussions/*` | n/a | 0 | 0 | Git publication gate 不需要场景讨论日志。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR078-01 | scope | 是否正式启动 CR078 remote Git governance / publication gate？ | 启动 CR078，先决策 publication scope，不执行 Git 写动作。 | 保持 candidate；取消 CR078。 | 推荐方案承接 CR082 handoff；保持 candidate 会让 staged removals 悬空。 | 影响 Git index 与发布治理。 | reject 时恢复 candidate。 |
| DQ-CP2-CR078-02 | scope | staged `process/**` removals 是否进入 CR078 决策范围？ | 纳入决策范围，CP5 再决定是否提交。 | 立即恢复 removals；立即提交。 | 推荐方案保留事实且不越权；立即动作风险高。 | 1572 个 staged deletion 仍保持。 | CP5 可改为 restore 或 commit。 |
| DQ-CP2-CR078-03 | implementation | `.gitignore`、`LEDGER.md`、`ledger.yaml`、bootstrap script 是否进入同一 publication scope？ | 纳入同一 scope，保证 process 外部化可恢复。 | 单独提交；暂不纳入。 | 推荐方案保持语义闭环；拆开会增加 clone 恢复断层。 | 影响新环境恢复。 | CP5 可拆分提交。 |
| DQ-CP2-CR078-04 | implementation | CR081/CR082 quality / release / release-context docs 如何处理？ | `docs/quality` 和 `docs/release` 的 CR081/CR082 文档纳入候选；`process/release` release-context 保留在外部 process ledger。 | 全部纳入 quant-lab Git；全部不纳入。 | 推荐方案遵守 process 外部化；force-add process/release 会破坏边界。 | 影响审计证据可见性。 | 用户要求时改为 force-add 或完全排除。 |
| DQ-CP2-CR078-05 | runtime_authorization | 是否在 CP2 授权 commit / push？ | 不授权；CP2 只确认范围。 | CP2 同时授权 commit；CP2 同时授权 push。 | 推荐方案分层清楚；备选会跳过 CP5 执行清单。 | 避免误提交或误推送。 | CP5 再决定。 |
| DQ-CP2-CR078-06 | security | remote URL、默认分支、tag、force push、history rewrite 是否进入默认执行范围？ | 默认不授权，只作为禁区记录。 | 纳入默认分支治理；允许 tag / force / rewrite。 | 推荐方案保护远端历史；备选不可逆风险高。 | 影响远端仓库安全。 | 需新 DQ 和精确命令。 |
| DQ-CP2-CR078-07 | follow_up_tracking | 外部 `/home/hyde/workspace/process` 治理如何处理？ | 当前仅作为本地 external process ledger；是否 Git/NAS/hybrid 另起门禁。 | 本轮初始化外部 Git；本轮推 NAS；Git+NAS 混合。 | 推荐方案避免把外部项目治理混入 quant-lab commit。 | 外部 process 仍未远端化。 | 用户明确选择时启动独立 CR。 |

### CP2 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 用户真实意图 | 先决策 CR078 Git publication governance，再执行任何 Git 动作。 |
| 场景覆盖 | staged removals、pointer files、docs、commit/push、remote 禁区、外部 process governance。 |
| Deferred Ideas | old root retirement、data/reports、NAS compare、provider rebuild、CR046 recovery。 |
| 回退方式 | 恢复 CR078 candidate；不触碰 staged removals。 |

### 不授权项

如果你回复 approve，表示接受上述 7 项推荐方案；不表示授权以下操作：

- 不授权 `git add`、`git commit`、`git push`。
- 不授权 remote URL 修改、默认分支变更、branch rename、tag、force push 或 history rewrite。
- 不授权读取或修改 `data/`、`reports/`、`.env`、token、账号、cookie、session、private key。
- 不授权 NAS 内容访问、NAS compare、provider fetch、lake write、catalog publish。
- 不授权 QMT/MiniQMT runtime、simulation/live、CR046 recovery。
- 不授权 old root retirement、backup 删除或 destructive cleanup。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR082 handoff 已读取 | 通过 | `process/context/CR082-CLOSURE-TO-CR078-HANDOFF-2026-06-17.md` |  |
| CR078 formal CR 已生成 | 通过 | `process/changes/CR-078-REMOTE-GIT-GOVERNANCE-PUBLICATION-GATE-2026-06-17.md` |  |
| Git 写动作未执行 | 通过 | 当前会话 |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | CR078 是否正式启动 | 通过 | DQ-CP2-CR078-01 |  |
| 2 | staged removals 是否纳入范围 | 通过 | DQ-CP2-CR078-02 |  |
| 3 | pointer files 是否同范围 | 通过 | DQ-CP2-CR078-03 |  |
| 4 | CR081/CR082 docs 分类是否接受 | 通过 | DQ-CP2-CR078-04 |  |
| 5 | CP2 是否不授权 commit/push | 通过 | DQ-CP2-CR078-05 |  |
| 6 | remote/default/tag/force/history 禁区是否接受 | 通过 | DQ-CP2-CR078-06 |  |
| 7 | 外部 process governance 是否分离 | 通过 | DQ-CP2-CR078-07 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 通过 | 本文件人工审查结果 |  |
| CP2 approve 不授权 Git 写动作 | 通过 | 不授权项 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 context | `process/context/CP2-CR078-GIT-PUBLICATION-CONTEXT.yaml` | 通过 |  |
| CP2 auto check | `process/checks/CP2-CR078-GIT-PUBLICATION-BASELINE.md` | 通过 |  |
| CP2 launch message | `process/checks/CP2-CR078-HUMAN-GATE-LAUNCH-MESSAGE.md` | 通过 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：用户
- 审查时间：2026-06-17T10:30:39+08:00
- 修改意见：用户回复“同意”，按 approve 处理，接受本文件 Decision Brief 中全部推荐方案。
- 风险接受项：接受 CR078 CP2 scope baseline 的不授权边界；CP2 本身不授权 Git 写动作。
