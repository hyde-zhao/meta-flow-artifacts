---
checkpoint_id: "CP2-CR121"
checkpoint_name: "Merge-Induced Directory Routing Regression Scope Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-23T09:08:54+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-23T09:16:46+08:00"
auto_check_result: "process/checks/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-PRECHECK.md"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/changes/CR-121-MERGE-INDUCED-DIRECTORY-ROUTING-REGRESSION-HARDENING-2026-06-23.md"
    - "process/context/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-CONTEXT.yaml"
---

# CP2 CR121 Directory Routing Regression 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-PRECHECK.md` | PASS | 0 | 可发起 CP2 scope confirmation。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或用户要求时读取完整来源。 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_decision_ids` | scanned | 4 | 4 | 本轮 CP2 四项决策纳入待确认。 |
| 用户显式请求 | 当前对话 | scanned | 4 | 4 | 用户指定启动 FU、CP2 only、不改代码、不 `git rm --cached`、不删除、不 runtime。 |
| source follow-up tracking | `process/changes/CR-120-FOLLOW-UP-TRACKING-2026-06-23.md#FU-CR120-001` | scanned | 3 | 3 | 目录路由、tracked ignored、merge 回流问题纳入 `DQ-CP2-CR121-01..03`。 |
| directory routing audit | `process/checks/CR120-DIRECTORY-ROUTING-MERGE-AUDIT-2026-06-23.md` | scanned | 3 | 3 | symlink 现状、ignored-but-tracked、merge brought-back 路径纳入初始检查对象。 |
| CR tracking index | `process/changes/CR-INDEX.yaml` | scanned | 1 | 1 | formal CR 转换、active / cp2_pending 状态纳入 CP2。 |
| runtime / NAS / credentials | N/A | n/a | 0 | 0 | 本 CR CP2 不需要外部运行时；明确列为不授权。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR121-01 | scope | 是否接受将 `FU-CR120-001` 升格为 CR121，并以 merge-induced directory routing regression hardening 为独立范围？ | 接受 CR121 作为独立 standard CR，CP2 只确认范围和候选清单。 | A. 仅保留 follow-up candidate；B. 拆成多个 CR；C. reject 并取消候选。 | 推荐方案保留追溯并避免目录治理问题被 CR120 closure 吞掉；保留候选最保守但不推进；拆分会增加门禁成本。 | 影响 `docs/` 分层路由、artifact symlink、ignored-but-tracked 路径和后续 merge guardrail。 | reject 时关闭 CR121 并把 `FU-CR120-001` 标记 cancelled；暂缓时回到 candidate。 |
| DQ-CP2-CR121-02 | implementation | 是否接受分阶段路线：CP2 仅做 read-only audit / scope freeze，所有 `.gitignore`、Git index、checker、guardrail 或 `git rm --cached` 等整改必须等 CP5？ | 接受 staged route，CP2 不改代码、不改 `.gitignore`、不清理 index。 | A. design-only 不进入 CP5；B. 直接进入 CP5；C. 拆成 audit CR 和 cleanup CR。 | 推荐方案权限边界清楚；design-only 无 enforcement；直接 CP5 更快但容易混淆授权；拆分更重。 | CP2 暂不产生源码、`.gitignore`、tests 或 index diff。 | 若用户要求立即整改，必须先生成 CP5 implementation / cleanup gate，不得直接执行。 |
| DQ-CP2-CR121-03 | scope | 是否冻结初始检查对象全集？ | 接受对象全集：`docs/design`、`docs/features`、`docs/quality`、`process`、`checkpoints`、`.claude/**`、`AGENTS.md`、`docs/qmt/**`、`docs/goldminer/**`、`docs/release/*.md`、`.gitignore`、guardrail/checker scripts；CP5 再裁剪整改子集。 | A. 只处理 `docs/design` / `docs/features`；B. 排除 `.claude/**` / `AGENTS.md`；C. 只处理 ignored-but-tracked 路径。 | 推荐方案不漏掉 merge 回流路径；窄范围风险更低但可能遗漏同类回归；只处理 tracked ignored 会忽略旧文档回流。 | 范围较宽，但 CP2 不执行整改。 | CP5 前如发现对象全集过宽，可在 CP5 裁剪 owner / remediation subset。 |
| DQ-CP2-CR121-04 | runtime_authorization | 是否继续禁止 runtime/NAS/credentials/trading/provider/lake/catalog/publish，并禁止 `git rm --cached`、删除、移动、reset/checkout、远端写？ | 继续禁止；只允许本地静态检查和 process 文件更新。 | A. 暂停所有验证命令；B. 另起 git index cleanup CP5；C. 另起 runtime gate。 | 推荐方案足够支撑 CP2；暂停所有验证会降低可检查性；git cleanup 和 runtime 都应独立门禁。 | 防止把目录路由 hardening 误解为 Git index cleanup 或外部系统授权。 | 任一真实清理、外部访问或运行需求必须另起 CP5 / high-risk gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP2 四项推荐方案，允许进入 CP5 scope / remediation authorization 准备。 |
| 备选方案 | `修改: <具体修改点>` 指明要调整的决策；或 `reject` 停止 CR121。 |
| 影响维度 | docs/artifact routing、Git index hygiene、merge guardrail、human gate / CR tracking、权限边界 |
| 风险与回退 | 当前风险低到中；CP2 只写 process artifacts。若不接受，退回 `FU-CR120-001` candidate 或取消。 |
| 用户需决策事项 | 是否批准 CR121 scope、staged route、初始检查对象全集和不授权边界。 |
| 自动终验授权 | `auto_final_authorization: false`；CP2 approve 不等于 CP5 整改授权或 CP8 终验授权。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确启动候选 | 通过 | 当前对话：启动 `FU-CR120-001` | 用户已 approve。 |
| CP2 context ready | 通过 | `process/context/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-CONTEXT.yaml` | 用户已 approve。 |
| 自动预检 PASS | 通过 | `process/checks/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-PRECHECK.md` | 用户已 approve。 |
| 不授权范围明确 | 通过 | 本文件 Decision Brief | 用户已 approve。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | CR121 范围清楚 | 通过 | CR121 变更单 | directory routing / merge regression hardening。 |
| 2 | 分阶段授权清楚 | 通过 | DQ-CP2-CR121-02 | CP2 不改代码、不改 `.gitignore`、不清理 index。 |
| 3 | 初始检查对象清楚 | 通过 | DQ-CP2-CR121-03 | CP5 再裁剪实际整改子集。 |
| 4 | 不授权范围清楚 | 通过 | DQ-CP2-CR121-04 | runtime、NAS、凭据、交易写、publish、Git cleanup、删除/移动禁止。 |
| 5 | Decision Collection Coverage 完整 | 通过 | 本文件 Decision Brief | 6 个来源已扫描。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认 CP2 | 通过 | 本文件人工审查结果 | 用户回复 `approve`。 |
| 可进入 CP5 准备 | 通过 | DQ-CP2-CR121-01..04 | 仍不授权具体整改。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR121 formal CR | `process/changes/CR-121-MERGE-INDUCED-DIRECTORY-ROUTING-REGRESSION-HARDENING-2026-06-23.md` | 通过 |  |
| CP2 context capsule | `process/context/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-CONTEXT.yaml` | 通过 |  |
| CP2 precheck | `process/checks/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-PRECHECK.md` | 通过 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-23T09:16:46+08:00
- 修改意见：用户回复 `approve`，接受 `DQ-CP2-CR121-01`、`DQ-CP2-CR121-02`、`DQ-CP2-CR121-03`、`DQ-CP2-CR121-04` 推荐方案；允许进入 CP5 remediation authorization 准备。
- 风险接受项：接受 CR121 当前初始检查对象全集和分阶段路线；CP2 approve 仍不授权代码、`.gitignore`、checker/tests、`git rm --cached`、Git index cleanup、删除/移动/覆盖、checkout/reset、远端写、runtime、NAS、凭据、交易写、provider/lake/catalog publish。
