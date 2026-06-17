# CP2-CR061 Requirements Baseline 人工检查点

## 自动预检摘要

自动预检结论：PASS。CR061 可作为 CR060 后续正式 package/import/layout convergence CR 启动；当前只冻结范围、候选对象和不授权边界，不执行真实代码改动。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR061-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在字段不足或人工审计时读取完整文档 |
| 全文档读取扩展 | 已读取 CR060 release context、handoff、CR-INDEX 和 repo scan 摘要 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前无既有 pending DQ。 |
| 自动预检结果 | `process/checks/CP2-CR061-REQUIREMENTS-BASELINE.md` | scanned | 4 | 5 | 范围、冲突、不授权转 DQ。 |
| CR060 release context | `process/release/RELEASE-CONTEXT-CR060.yaml` | scanned | 5 | 3 | package/import deferred 转 CR061。 |
| CR061 candidate docs | `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md` | scanned | 16 | 5 | 合并为范围决策。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | 用户明确启动 CR061。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR061-01 | scope | 是否正式启动 CR061，承接 CR060 deferred package/import/layout boundary？ | 启动 CR061，并登记为迁移并行推进对象；CR046 继续暂停。 | 备选 A：保持为 follow-up candidate；备选 B：先启动 CR062 Git remote。 | 推荐方案按既定迁移顺序处理代码结构；备选 A 停滞；备选 B 会在包结构未稳定前发布远端。 | CR061 会影响包/导入/测试设计，但当前不执行真实改动。 | 用户要求先远端发布时切 CR062；要求暂停时保持 candidate。 |
| DQ-CP2-CR061-02 | scope | CR061 当前范围是否限定为 gate-only，不直接改代码？ | 当前只创建 CR、candidate list、layout plan、rollback/preserve-audit 和 CP2/CP3/CP5。 | 备选 A：立即执行 pyproject/import 改动；备选 B：只写 CR 不写门禁。 | 推荐方案可审计且不越权；备选 A 风险高；备选 B 不能进入实现准备。 | 延后一轮实现，但降低误改风险。 | 用户明确授权实现后进入下一步。 |
| DQ-CP2-CR061-03 | risk_acceptance | 是否接受当前 dirty worktree 只用于 planning，不作为真实实现基线？ | 接受；真实实现前必须冻结 include/exclude 或刷新 rollback_ref。 | 备选 A：现在停止，先 commit/clean；备选 B：忽略 dirty state 直接实现。 | 推荐方案不阻断门禁；备选 A 更干净但会转入 Git 操作；备选 B 不可接受。 | 真实实现前仍有阻断项。 | 实现前若无法冻结 dirty manifest，则 BLOCKED。 |
| DQ-CP2-CR061-04 | security | 是否继续禁止凭据、runtime、NAS/lake/provider 和 Git remote？ | 继续禁止，分别保留给 CR062-CR065 或 CR046 recovery。 | 备选 A：合并 Git remote；备选 B：合并 NAS/lake/runtime。 | 推荐方案边界清晰；备选会把风险面扩大。 | 后续 CR 数量增加，但可回滚。 | 用户要求合并时必须重开影响分析。 |
| DQ-CP2-CR061-05 | implementation | 是否接受 staged compatibility-first 作为后续设计方向？ | 接受：先保留 legacy roots，再考虑 `quant_lab` bridge，bulk move 二次授权。 | 备选 A：一次性 bulk move 到 `src/quant_lab`；备选 B：永久保留旧 roots。 | 推荐方案兼顾迁移和稳定；bulk move 干净但风险高；永久旧 roots 无法完成身份收敛。 | 新旧 import 会共存一段时间。 | full dry-run 和用户二次授权后可切 bulk move。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP2-CR061-01..05 推荐方案。 |
| 备选方案 | 保持 candidate、先 CR062、一次性 bulk move、永久保留旧 roots。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全 / 权限、交付影响。 |
| 优劣分析 | 见上表。 |
| 风险与回退 | 真实实现前必须有 rollback_ref、dirty manifest 和 CP5 / 二次授权。 |
| 用户需决策事项 | DQ-CP2-CR061-01..05。 |

不授权项：CP2 通过不授权代码改动、file move/rename/delete、bulk import rewrite、Git remote/push/tag/history rewrite、NAS copy/move/delete、data lake root replacement、provider fetch、lake write、catalog publish、凭据读取、QMT/MiniQMT runtime 或恢复 CR046。

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR060 已关闭 | PASS |
| 用户明确启动 CR061 | PASS |
| CR046 不恢复 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 影响分析完成 | PASS |
| 决策项完整 | PASS |
| 不授权项列明 | PASS |
| gate-only 边界明确 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户确认后可进入 CP3 / CP5 审查 | PASS |

## Deliverables

- `process/checks/CP2-CR061-REQUIREMENTS-BASELINE.md`
- `process/checkpoints/CP2-CR061-REQUIREMENTS-BASELINE.md`
- `process/context/CP2-CR061-REQUIREMENT-CONTEXT.yaml`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T07:52:05+08:00
- 修改意见：用户回复“同意”，按 `approve` 解析，接受 DQ-CP2-CR061-01..05 推荐方案。
- 风险接受项：接受 CR061 以 gate-only 方式启动；CR046 继续暂停；真实实现前必须冻结 rollback_ref 和 dirty worktree include/exclude。
