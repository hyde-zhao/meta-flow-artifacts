---
title: "CR115 Human Gate Launch Message Checkable Draft Contract"
cr_id: "CR-115"
status: "closed-ready"
created_at: "2026-06-22T17:05:02+08:00"
owner: "host-orchestrator"
scope: "no-code-no-runtime"
cp2_approved_at: "2026-06-22T17:15:31+08:00"
cp8_approved_at: "2026-06-22T17:23:57+08:00"
---

# CR115 Human Gate Launch Message Checkable Draft Contract

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 启动 CR115 CP2 复盘范围、draft contract 和检查项。 |
| v0.2 | 2026-06-22 | host-orchestrator | 回填 CP2 approve，冻结 no-code/no-runtime 范围，并将 draft contract 标记为 CP8 待终验。 |
| v1.0 | 2026-06-22 | host-orchestrator | 回填 CP8 approve，关闭 CR115 READY，并将 checker implementation 分流为后续候选。 |

## 目标

CR115 只做 human gate launch message 的可检查草案契约。目标不是实现 checker，也不是修改源码，而是明确发起 CP2 / CP3 / CP5 / CP8 人工门禁时，对话消息必须能从 checkpoint 文件中验证。

## Draft Contract

| 字段 | 必须出现 | 来源 | 检查方式 |
|---|---|---|---|
| checklist 路径 | 是 | `process/checkpoints/CP*.md` | 消息包含可读路径。 |
| 自动预检结论 | 是 | `## 自动预检摘要` | 消息中列出 PASS / WAIVED / BLOCKED。 |
| Context Capsule Summary | 是 | `### Context Capsule Summary` | 消息包含 capsule 路径、状态、read_profile。 |
| Decision Collection Coverage 摘要 | 是 | `### Decision Collection Coverage` | 消息包含扫描来源数量、候选问题数、纳入 DQ 数。 |
| 待决策项数量 | 是 | `### 待人工决策清单` | 消息明确本轮 DQ 数。 |
| DQ 表 | 有 DQ 时必须出现 | `### 待人工决策清单` | 至少包含 ID、类型、问题、推荐、备选、影响 / 风险。 |
| 不授权项 | 是 | checkpoint DQ / non-authorized table | 明确 approve 不授权的高风险范围。 |
| exact 回复 | 是 | checkpoint launch protocol | 只展示 `approve`、`修改: <具体修改点>`、`reject`。 |

## Draft Launch Message 形态

```text
请审查：process/checkpoints/<CP-ID>-<SLUG>.md
自动预检结论：PASS
上下文胶囊：process/context/<CP-ID>-<SLUG>-CONTEXT.yaml（read_profile=minimal）
本轮待人工决策项：N
决策收集覆盖：已扫描 S 个来源，发现候选问题 C 个，纳入待决策 D 个；N/A / 缺失来源 M 个，原因见 checklist 的 Decision Collection Coverage。

如果你回复 approve，表示你接受以下 N 项推荐方案，不表示授权以下 M 项禁止操作。

待人工决策清单：
| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| <DQ-ID> | <type> | <question> | <recommendation> | <alternatives> | <pros-cons> | <impact-risk> |

不授权项：
- <item>

回复：
approve
修改: <具体修改点>
reject
```

## 初始检查项排序

| 排名 | 检查项 | 风险 | 建议处理 |
|---:|---|---|---|
| 1 | DQ 表与 checkpoint 决策表一致 | 高 | 本 CR 内定义草案和人工检查；实现 checker 另起 CR。 |
| 2 | approve 不授权项必须显式列出 | 高 | 本 CR 内保留为必填项。 |
| 3 | exact 回复必须只保留三项 | 中 | 本 CR 内保留为必填项。 |
| 4 | Decision Collection Coverage 摘要必须有数字 | 中 | 本 CR 内定义口径。 |
| 5 | launch message checker 自动化 | 中 | 不在本 CR 内实现；转后续 implementation candidate。 |

## CP2 审批后收敛结论

| 项目 | 结论 |
|---|---|
| 范围 | approved：CR115 只做 human gate launch message checkable draft contract。 |
| 检查项排序 | approved：优先检查 DQ 表一致性、不授权项、exact 回复、coverage 数字和 checker 可验证性。 |
| 不授权边界 | approved：不授权源码、tests、checker implementation、runtime、NAS、凭据、交易写、CR105、CR089 恢复或 publish。 |
| CP8 交付对象 | 本文件、CR115 变更单、CP2 审查稿、CP8 Context Capsule、Release Context 和 CP8 检查点。 |
| 后续实现 | 若需要自动化 checker，必须另起独立 implementation CR。 |

## 不授权边界

| 范围 | CR115 结论 |
|---|---|
| 源码修改 | 不授权 |
| tests 修改 | 不授权 |
| checker implementation change | 不授权 |
| CR105 / order-write / simulation / live | 不启动 |
| CR089 恢复 | 不启动 |
| QMT / MiniQMT / XtQuant / gateway runtime | 不启动 |
| NAS / 凭据 / 账户 / raw log | 不访问 |
| provider fetch / lake write / catalog publish | 不执行 |

## CP2 待确认

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR115-01 | scope | 是否接受本 CR 只做 no-code/no-runtime human gate launch message draft contract | 接受当前范围 | A. 暂停；B. 改 pending queue audit；C. 改 implementation CR | 推荐方案风险低；C 需要新授权。 |
| DQ-CP2-CR115-02 | follow_up_tracking | 是否接受当前检查项排序准则 | 接受五项排序准则 | A. 只输出样例；B. 先实现 checker；C. 只保留原则 | 推荐方案利于后续人工作业门禁治理。 |
| DQ-CP2-CR115-03 | runtime_authorization | approve 是否继续不代表 runtime / 源码 / NAS / trading 授权 | 继续禁止隐式授权 | A. 另起源码 CR；B. 另起 checker CR；C. 另起 runtime gate | 推荐方案避免误授权；备选需要独立门禁。 |

## CP8 待确认

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP8-CR115-01 | follow_up_tracking | 是否接受 CR115 draft contract 和检查项排序作为本轮交付结果？ | 接受当前 contract，关闭 CR115 为 READY | A. 调整字段；B. 降级为样例消息；C. 转 checker implementation CR | 推荐方案风险低；C 需要源码 / tests 授权。 |
| DQ-CP8-CR115-02 | runtime_authorization | 是否确认 CP8 approve 不授权源码、tests、checker implementation、runtime、NAS、凭据、交易写或 publish？ | 是，CP8 approve 只关闭 no-code/no-runtime draft contract | A. 另起源码实现 CR；B. 另起 checker implementation CR；C. 另起 runtime/NAS/trading gate | 推荐方案避免误授权；备选需要独立门禁。 |

## CP8 审批后关闭结论

| 项目 | 结论 |
|---|---|
| CR115 交付状态 | closed-current-delivery / READY |
| 用户确认 | 2026-06-22T17:23:57+08:00 用户回复 `approve`。 |
| 接受决策 | `DQ-CP8-CR115-01..02` 推荐方案。 |
| 关闭范围 | human gate launch message checkable draft contract、字段映射、检查项排序和 no-code/no-runtime 不授权边界。 |
| 后续候选 | `FU-CR115-001` Human Gate Launch Message Checker Implementation Candidate，仅登记，不自动启动。 |
| 不授权边界 | 源码、tests、checker implementation、runtime、NAS、凭据、交易写、CR105、CR089 恢复、provider fetch、lake write、catalog publish 均未授权。 |
