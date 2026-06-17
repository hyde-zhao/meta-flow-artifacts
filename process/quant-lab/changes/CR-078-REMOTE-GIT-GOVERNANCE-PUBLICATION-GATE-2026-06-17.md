---
cr_id: "CR-078"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "涉及 Git index、commit、push、远端治理、默认分支、history rewrite、force push、tag 和外部 process project 治理，必须保持 standard。"
rollback_to: "CR078 follow-up candidate not started; CR082 closed-current-delivery / READY_WITH_RISK remains unchanged"
approval_result: "approved"
created_at: "2026-06-17T10:08:12+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-17T10:30:39+08:00"
execution_result: "local-commit-complete"
commit_hash: "d4d2881"
executed_at: "2026-06-17T10:37:21+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-082"
source_checkpoint: "process/checkpoints/CP8-CR082-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR082-05"
follow_up_type: "remote-git-governance-publication-gate"
risk_class: "high"
owner: "host-orchestrator"
revisit_condition: "CR081/CR082 关闭后需要处理 staged process/** removals、pointer files、CR081/CR082 docs 的提交策略或远端发布策略。"
acceptance_criteria: "CR078 formal CR、CP2/CP3/CP5 context capsule、自动预检、Decision Brief、人工门禁稿和 launch message 完整；用户已确认提交范围、commit/push 授权、remote/default branch/tag/history/force 边界和外部 process project 治理方案。仅允许一次 allowlist git add 和一次本地 git commit；不允许 git push 或 remote write。"
close_condition: "用户完成 CR078 CP2/CP3/CP5 人工确认后，已按确认范围完成一次本地 allowlist commit；后续真实 push 或外部 process Git/NAS 治理仍需后续独立门禁执行。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-078 Remote Git Governance / Publication Gate

## 变更描述

CR082 已关闭为 `closed-current-delivery / READY_WITH_RISK`，并明确把 staged `process/**` removals、ledger pointer files、CR081/CR082 quality / release / release-context docs 的提交与推送策略交给 CR078。当前 Git index 中已有 1572 个 staged `process/**` removals，这是 CR081/CR082 把过程台账移出当前业务项目 Git 后留下的事实；当前尚未执行 `git commit`、`git push`、remote URL 修改、默认分支治理、tag、force push 或 history rewrite。

CR078 的目标不是立即发布，而是先形成 publication gate：明确哪些变更属于本轮发布范围、是否允许在当前分支提交、是否允许推送、是否允许任何远端治理动作，以及外部 `/home/hyde/workspace/process` 应如何治理。用户批准前，不执行任何改变 Git index、commit history 或远端状态的命令。

当前分支：`work/chapter3-factor-gap-remediation-20260608`。CR078 默认不切换分支、不修改默认分支、不 tag、不 force push、不重写历史。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-082` |
| 来源检查点 | `process/checkpoints/CP8-CR082-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR082-05` |
| follow-up 类型 | `remote-git-governance-publication-gate` |
| 风险等级 | high |
| owner | host-orchestrator |
| 重访条件 | CR081/CR082 关闭后需要处理 staged process removals、pointer files 和发布策略。 |
| 验收标准 | 三门 Decision Brief 完整且 human-gate 校验通过；批准前不执行 Git 写动作。 |
| 关闭条件 | 用户确认并执行或明确保持 no-op；如涉及 push 或外部 process Git/NAS 另按批准边界处理。 |

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| CR046 | related-no-overlap | CR046 仍 user-paused；CR078 不恢复 QMT/MiniQMT、runtime 或交易验证。 |
| CR081 | dependency-closed | CR081 已关闭；其 staged removals 和 pointer files 是 CR078 输入。 |
| CR082 | dependency-closed | CR082 已关闭；`process -> ../process/quant-lab` 和 handoff 是 CR078 输入。 |
| CR077 | no-overlap | CR077 old root retirement 仍为 candidate；CR078 不删除、重命名或移动旧根。 |
| staged process removals | in-scope-for-decision | 1572 个 staged removals 只允许在 CP5 批准后处理。 |
| dirty worktree | in-scope-for-classification | `.gitignore`、README、用户手册、`pyproject.toml`、`uv.lock`、CR081/CR082 docs 和大量历史未跟踪文档必须分组决策。 |
| data/reports/credentials/NAS/runtime | out-of-scope | 本 CR 不读取或修改数据、报告、凭据、NAS 内容、provider/lake/catalog 或 runtime。 |

冲突预检结论：允许启动正式 CR078 并发起 CP2/CP3/CP5 publication gate；批准前不得执行 `git add`、`git commit`、`git push`、remote write、branch/default branch governance、tag、force push 或 history rewrite。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-078-REMOTE-GIT-GOVERNANCE-PUBLICATION-GATE-2026-06-17.md` | 新增 | N/A | 本文件 frontmatter / 正文 | pending |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused、CR081/CR082 closed 和 CR078 candidate 来源 | N/A | pending |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR078 candidate 来源并转 active formal CR | N/A | pending |
| `process/context/CP2-CR078-GIT-PUBLICATION-CONTEXT.yaml` | 新增 | N/A | N/A | pending |
| `process/context/CP3-CR078-GIT-PUBLICATION-CONTEXT.yaml` | 新增 | N/A | N/A | pending |
| `process/context/CP5-CR078-GIT-PUBLICATION-CONTEXT.yaml` | 新增 | N/A | N/A | pending |
| `process/checks/CP2-CR078-GIT-PUBLICATION-BASELINE.md` | 新增 | N/A | N/A | pending |
| `process/checkpoints/CP2-CR078-GIT-PUBLICATION-BASELINE.md` | 新增 | N/A | N/A | pending |
| `process/checks/CP3-CR078-GIT-PUBLICATION-HLD-CONSISTENCY.md` | 新增 | N/A | N/A | pending |
| `process/checkpoints/CP3-CR078-GIT-PUBLICATION-HLD-REVIEW.md` | 新增 | N/A | N/A | pending |
| `process/checks/CP5-CR078-GIT-PUBLICATION-READINESS.md` | 新增 | N/A | N/A | pending |
| `process/checkpoints/CP5-CR078-GIT-PUBLICATION-READINESS.md` | 新增 | N/A | N/A | pending |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR082 CP8 `DQ-CP8-CR082-04` | CR078 CP5 staged removals 决策 | 保留 CR082 CP8 原决策 | CR082 保持 staged state；CR078 决定是否提交。 |
| CR082 handoff | CR078 scope / context | 保留 handoff 原文 | handoff 是 CR078 启动最小上下文。 |
| CR073 / CR074 `CR078-candidate` | CR078 formal CR | 保留 candidate 来源 | CR078 承接 remote Git governance after cutover。 |
| CR070 Git governance no-op gate | CR078 governance 参考 | 保留 CR070 closed-current-delivery | CR078 不继承 CR070 的 no-op 结论；本轮新增 process externalization publication scope。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增或重定义发布治理需求 | CR078 scope / staged removals / pointer files | true | 新增 CR078 publication gate，先决策再执行。 |
| 场景层 | 是否改变验证覆盖范围 | Git index audit、allowlist、human-gate、no-secret/no-data boundary | true | 增加 staged diff、dirty worktree 分类和不授权项验证。 |
| 计划层 | 是否改变 Phase / Gate | CP2 / CP3 / CP5 | true | 生成三门门禁；批准前不执行 Git 写动作。 |
| 安全层 | 是否引入高风险动作或权限要求 | commit、push、remote、branch/default branch、tag/history/force | true | 所有真实 Git 写动作必须被 DQ 显式授权；默认不授权 push 和远端治理。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | process ledger pointer、CR081/CR082 docs、release context | true | 本轮仅生成治理文档；后续按批准范围执行 allowlist commit 或 no-op。 |

## 回退决策

- 影响范围：局部，限 CR tracking、publication gate 文档和后续 Git 执行授权。
- 回退到阶段：`CR078 follow-up candidate not started`。
- 需要重新确认的对象：是否继续启动 CR078、是否保持 staged removals、是否恢复候选状态、是否取消提交或转入外部 process project governance CR。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 主题涉及 Git index、commit/push 和远端发布治理。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 涉及 process 外部化后的 Git 跟踪边界和远端写入边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 涉及当前项目 Git 与外部 process project 的归属边界。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要 CP3/CP5 固化 publication topology、allowlist 和回退。 |
| 是否保持 fast-lane | false | 保持 standard。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- batch_id：`CR078-GIT-PUBLICATION-GATE`
- 批次范围来源：CR078 publication governance 影响分析
- 批次内 Story：N/A，当前为 Git publication governance gate，不新增业务 Story。
- 批次人工确认稿：`process/checkpoints/CP5-CR078-GIT-PUBLICATION-READINESS.md`
- 开发启动条件：
  - [ ] CP2 publication scope 人工确认 approved
  - [ ] CP3 governance architecture 人工确认 approved
  - [ ] CP5 execution readiness / authorization 人工确认 approved
  - [ ] 如需 `git add`、`git commit` 或 `git push`，必须由 CP5 DQ 精确授权

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 启动 CR078 formal CR | CR082 handoff / CR-INDEX candidate | 本 CR、context、checks、checkpoints | 文件生成完成 | 运行静态校验 |
| 2 | `host-orchestrator` | 发起 CP2/CP3/CP5 人工确认 | 本 CR、Git status 摘要、handoff | 人工门禁稿和 launch messages | human-gate 校验通过 | 等待用户回复 |
| 3 | `host-orchestrator` | 用户批准后执行精确 allowlist 或 no-op | 用户 `approve` / `修改` / `reject` | Git action plan / no-op 结果 | 只能执行获批命令 | 回写 CP6/CP7/CP8 或修订门禁 |
| 4 | `host-orchestrator` | 若用户要求 push 或远端治理 | 用户明确授权 | push / remote governance 子门禁 | 必须二次确认或在 CP5 DQ 中明确 | 不允许 force/history/tag/default branch 默认执行 |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：N/A
- 自动通过条件：N/A
- 授权原文：N/A
- 授权时间：N/A
- 回填要求：无自动终验；CR078 的 commit / push / remote write 必须由用户在人工门禁中明确确认。

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：`process/changes/CR-INDEX.yaml`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .`
- 状态取值：CR078 已从 candidate 转为 `closed-current-delivery`，本地 commit `d4d2881` 已完成；push 未授权。

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR078-candidate | Remote Git governance after cutover | closed-current-delivery | CR | 5 | `process/changes/CR-078-REMOTE-GIT-GOVERNANCE-PUBLICATION-GATE-2026-06-17.md` | CR046 paused no-overlap; CR082 closed dependency | closed | 本地 commit `d4d2881` 已完成，push 未授权 | 如需 push、remote/default/tag/force/history 或 external process Git/NAS，另起独立门禁 |

## 处理结论

- 审批结论：approved / local commit complete
- [ ] 自动批准（低风险）
- [x] 人工确认通过（中风险）
- [x] 人工审批通过（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Handoff | `process/context/CR082-CLOSURE-TO-CR078-HANDOFF-2026-06-17.md` | CR082 关闭后的最小恢复上下文。 |
| Checkpoint | `process/checkpoints/CP8-CR082-DELIVERY-READINESS.md` | CR082 CP8 approved，明确 staged removals 交给 CR078。 |
| Git index | staged `process/**` removals = 1572 | 只作为输入；批准前不提交。 |
| Worktree | `.gitignore` / `LEDGER.md` / `ledger.yaml` / `scripts/link-engineering-ledger.sh` / CR081-CR082 docs / other dirty files | 必须由 CR078 DQ 分类。 |
