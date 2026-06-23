---
checkpoint_id: "CP5-CR121"
checkpoint_name: "Directory Routing Remediation Authorization Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-23T09:20:34+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-23T09:30:34+08:00"
auto_check_result: "process/checks/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-PRECHECK.md"
target:
  phase: "story-planning"
  story_id: ""
  artifacts:
    - "process/changes/CR-121-MERGE-INDUCED-DIRECTORY-ROUTING-REGRESSION-HARDENING-2026-06-23.md"
    - "process/context/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-CONTEXT.yaml"
---

# CP5 CR121 Directory Routing Remediation Authorization 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-PRECHECK.md` | PASS | 0 | 可发起 CP5 remediation authorization。 |

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或用户要求时读取完整来源。 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_decision_ids` | scanned | 4 | 4 | 本轮 CP5 四项决策纳入待确认。 |
| 用户显式请求 | 当前对话 | scanned | 5 | 4 | 用户指定只做授权门禁准备、裁剪子集、区分 audit-only / allowed remediation / forbidden；合并为 4 项 DQ。 |
| CP2 checkpoint | `process/checkpoints/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-REVIEW.md` | scanned | 4 | 4 | CP2 已批准的范围和不授权边界作为 CP5 输入。 |
| directory routing audit | `process/checks/CR120-DIRECTORY-ROUTING-MERGE-AUDIT-2026-06-23.md` | scanned | 6 | 4 | symlink、tracked ignored、merge brought-back、公开 release 入口纳入裁剪矩阵。 |
| CR121 formal CR | `process/changes/CR-121-MERGE-INDUCED-DIRECTORY-ROUTING-REGRESSION-HARDENING-2026-06-23.md` | scanned | 4 | 4 | 五维影响、文档处理和禁止项已消费。 |
| runtime / NAS / credentials | N/A | n/a | 0 | 0 | 本 CP5 不需要外部运行时；明确列为不授权。 |

### Remediation Scope Matrix

| 路径 / 对象 | CP5 分类 | 推荐处理 | 允许的后续动作 | 禁止动作 |
|---|---|---|---|---|
| `docs/design` | allowed-remediation | 纳入 CP6 index-only untrack 子集 | CP5 approve 后，CP6 可执行 index-only untrack，保留本地 symlink 和 artifact 目标内容 | 删除 symlink、删除目标目录、移动/覆盖内容 |
| `docs/features` | allowed-remediation | 纳入 CP6 index-only untrack 子集 | 同上 | 同上 |
| `docs/quality` | allowed-remediation | 纳入 CP6 index-only untrack 子集 | 同上 | 同上 |
| `process` | allowed-remediation | 纳入 CP6 index-only untrack 子集 | CP5 approve 后，CP6 可执行 index-only untrack，保留外置 process symlink | 删除 process symlink、重建本地 process、删除 artifact process |
| `checkpoints` | allowed-remediation | 纳入 CP6 index-only untrack 子集 | CP5 approve 后，CP6 可执行 index-only untrack，保留本地 symlink | 删除 symlink 或 artifact checkpoint 内容 |
| `.claude/**` | allowed-remediation | 纳入 CP6 index-only untrack 子集 | CP5 approve 后，CP6 可执行 index-only untrack，不删除工作区文件 | 删除 `.claude` 文件、改 Claude 配置语义 |
| `AGENTS.md` | audit-only | 本 CR 不整改 | 仅记录 tracked/ignored 冲突；若需改跟踪策略另起 CR | untrack、删除、移动、改写规则 |
| `docs/qmt/**` | audit-only | 本 CR 不整改 | 仅记录 merge 回流风险；后续按公开文档 vs 内部归档另起内容治理 CR | 删除、移动、批量归档、内容改写 |
| `docs/goldminer/**` | audit-only | 本 CR 不整改 | 同上 | 同上 |
| `docs/release/*.md` | audit-only | 本 CR 保留公开 release 入口，不处理 CR-specific release 历史 | 仅审计公开 / 内部边界 | 删除或迁移 release 入口 |
| `.gitignore` | audit-only | 本 CR 不修改 | 保留现有 ignore 规则；若规则需要修订另起 CP5 | 修改 `.gitignore` |
| guardrail / checker scripts | audit-only | 本 CR 不改代码 | 后续如需 checker hardening 另起实现 CR | 修改源码、tests、checker |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR121-01 | implementation | 是否授权后续 CP6 执行最小 index-only remediation 子集？ | 授权仅对 `docs/design`、`docs/features`、`docs/quality`、`process`、`checkpoints`、`.claude/**` 执行 index-only untrack；不删除工作区文件和 symlink 目标。 | A. 只处理 artifact symlink：`docs/design`、`docs/features`、`docs/quality`、`process`、`checkpoints`；B. design-only，不授权任何 untrack；C. 拆出 `.claude/**` 为独立 CR。 | 推荐方案覆盖 tracked ignored 的主要回归面；A 更保守但保留 `.claude` 冲突；B 无 remediation；C 更细但增加门禁轮次。 | CP6 会产生 Git index 变更，后续是否提交/发布必须另起 remote/publication gate。 | 若 CP6 前发现路径不在 Git index 或存在 symlink 异常，停止并回 CP5 修订子集。 |
| DQ-CP5-CR121-02 | scope | 哪些对象仅审计、不在 CR121 中整改？ | 将 `AGENTS.md`、`docs/qmt/**`、`docs/goldminer/**`、`docs/release/*.md`、`.gitignore`、guardrail/checker scripts 标记为 audit-only。 | A. 把 `AGENTS.md` 也纳入 untrack；B. 把 `docs/qmt/**` / `docs/goldminer/**` 纳入本 CR 内容治理；C. 同时授权 `.gitignore` 和 checker hardening。 | 推荐方案避免混合内容治理和代码实现；A 可能破坏当前规则入口；B/C 明显扩大范围。 | audit-only 会保留部分 tracked/ignored 或 merge 回流风险，需要后续 CR 决策。 | 若用户要处理 audit-only 对象，必须新建或重开更明确的内容 / 代码 / policy CR。 |
| DQ-CP5-CR121-03 | security | CP5 approve 后仍禁止哪些动作？ | 即使 CP5 approve，也继续禁止删除/移动/覆盖、`.gitignore` 修改、源码/tests/checker 修改、git add/commit/push/remote write、checkout/reset/clean、runtime/NAS/凭据/交易/provider/lake/catalog/publish。 | A. 同时授权 `.gitignore` 修改；B. 同时授权 commit；C. 同时授权 checker hardening。 | 推荐方案权限最小；备选会把 index cleanup、发布治理和代码治理混在一个门禁里。 | 防止 CP5 remediation 被误解为发布或代码实现授权。 | 如需任一禁止动作，另起对应 CP5 / high-risk gate。 |
| DQ-CP5-CR121-04 | risk_acceptance | 是否接受 CP6 index-only cleanup 后出现本地 Git index dirty 状态，且不自动提交 / 推送？ | 接受：CP6 只做本地 index-only remediation 和验证；提交、推送、远端治理、default branch / tag / history 均不授权。 | A. CP6 后立即另起 publication gate；B. 本轮不执行 cleanup；C. 将 cleanup 和 commit 合并到一个高风险门禁。 | 推荐方案风险可控且保留审计；A 增加后续门禁但清晰；B 保守但不修复；C 范围过大。 | 本地 index 变更需要后续人类决定是否纳入仓库发布。 | 若用户不能接受 dirty index，选择 B 或要求先设计 publication gate。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP5 四项推荐方案，授权后续 CP6 执行最小 index-only remediation 子集。 |
| 备选方案 | `修改: <具体修改点>` 指明要调整的决策；或 `reject` 停止 CR121 remediation。 |
| 影响维度 | Git index hygiene、artifact symlink 路由、merge regression hardening、仓库公开文档面、权限边界 |
| 风险与回退 | 当前为中风险 repository governance；CP5 approve 后的 CP6 仍只允许 index-only untrack，不删除文件、不提交、不推送。 |
| 用户需决策事项 | 是否批准 `DQ-CP5-CR121-01..04`。 |
| 自动终验授权 | `auto_final_authorization: false`；CP5 approve 不等于 CP8 终验授权、commit/push 授权或 runtime 授权。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR121 CP2 approved | 通过 | `process/checkpoints/CP2-CR121-DIRECTORY-ROUTING-REGRESSION-REVIEW.md` | 已具备进入 CP5 的范围基础。 |
| CP5 context ready | 通过 | `process/context/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-CONTEXT.yaml` | read_profile=minimal。 |
| 自动预检 PASS | 通过 | `process/checks/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-PRECHECK.md` | 0 blockers。 |
| 整改子集已裁剪 | 通过 | 本文件 Remediation Scope Matrix | 用户已接受推荐子集。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | allowed-remediation 子集清楚 | 通过 | `DQ-CP5-CR121-01` | 仅 index-only untrack。 |
| 2 | audit-only 对象清楚 | 通过 | `DQ-CP5-CR121-02` | 不处理内容治理 / `.gitignore` / checker。 |
| 3 | 禁止项清楚 | 通过 | `DQ-CP5-CR121-03` | 删除/移动/覆盖、代码、远端、runtime 全部禁止。 |
| 4 | dirty index 风险已暴露 | 通过 | `DQ-CP5-CR121-04` | CP6 后不自动 commit/push。 |
| 5 | Decision Collection Coverage 完整 | 通过 | 本文件 Decision Brief | 6 个来源已扫描。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户确认 CP5 | 通过 | 本文件人工审查结果 | 用户回复 `approve`。 |
| 可进入 CP6 remediation | 通过 | DQ-CP5-CR121-01..04 | 仅限推荐方案：后续 CP6 最小 index-only remediation；不授权删除、移动、`.gitignore`、代码、commit/push 或 runtime。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 context capsule | `process/context/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-CONTEXT.yaml` | 通过 | ready / minimal。 |
| CP5 precheck | `process/checks/CP5-CR121-DIRECTORY-ROUTING-REMEDIATION-AUTHORIZATION-PRECHECK.md` | 通过 | PASS, 0 blockers。 |
| CR121 formal CR | `process/changes/CR-121-MERGE-INDUCED-DIRECTORY-ROUTING-REGRESSION-HARDENING-2026-06-23.md` | 通过 | CP5 authorization approved; CP6 cleanup 尚未执行。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-23T09:30:34+08:00
- 修改意见：无；用户回复 `approve`，接受 `DQ-CP5-CR121-01..04` 推荐方案。
- 风险接受项：接受后续 CP6 index-only remediation 可能产生本地 Git index dirty；不自动提交、不推送、不执行远端治理。CP5 approve 不授权删除 / 移动 / 覆盖、`.gitignore` 修改、源码 / tests / checker 修改、commit / push / remote write、checkout / reset / clean、runtime / NAS / 凭据 / 交易 / provider / lake / catalog / publish。
