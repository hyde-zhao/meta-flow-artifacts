# CP2-CR066 Repository Hygiene Baseline 人工检查点

## 自动预检摘要

自动预检结论：PASS。CR066 可作为 CR062 blocker 的后续正式 CR 启动，用于冻结 repository hygiene / clean publication strategy 范围；本门禁不授权任何 Git 写动作、branch 操作、history rewrite、NAS/data lake/provider/catalog/runtime/凭据、physical move、bulk import rewrite 或 CR046 recovery。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR066-REPOSITORY-HYGIENE-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在字段不足或人工审计时读取完整文档 |
| 全文档读取扩展 | 已读取 CR062 preflight、STATE、CR-INDEX、CR066 scope |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 1 | 1 | DQ-CR062-BLOCKER-01 已由用户同意，转 CR066。 |
| CR062 preflight | `process/checks/CR062-POST-APPROVAL-PREFLIGHT.md` | scanned | 5 | 3 | blocker findings 转 CP2 DQ。 |
| CR066 scope | `docs/release/REPOSITORY-HYGIENE-SCOPE-CR066.md` | scanned | 6 | 4 | 范围和禁止项转 DQ。 |
| CR-INDEX | `process/changes/CR-INDEX.yaml` | scanned | 3 | 1 | 编号冲突和后续 CR 顺序转 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | 用户“同意”接受 follow-up。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR066-01 | scope | 是否正式启动 CR066，承接 CR062 blocker 的 clean publication strategy？ | 启动 CR066；CR062 保持 blocked，CR046 继续暂停。 | 备选 A：保持 CR062 blocked 不启动；备选 B：直接接受当前分支发布风险；备选 C：恢复 CR046。 | 推荐方案可控推进；不启动会停滞；直接发布违反 exclude policy；恢复 CR046 越界。 | 当前只创建策略门禁，不执行 Git 写动作。 | 用户选择不启动时，CR062 保持 blocked。 |
| DQ-CP2-CR066-02 | scope | CR066 范围是否限定为 repository hygiene / clean publication strategy？ | 限定为策略和门禁，不执行 clean branch / commit / push。 | 备选 A：合并执行；备选 B：合并 history rewrite；备选 C：只记录风险不设计。 | 推荐风险最小；合并执行需要更强授权；history rewrite 高风险；只记录不解决 blocker。 | 后续仍需执行 CR。 | 用户要求合并执行时重开 CP5 runtime_authorization。 |
| DQ-CP2-CR066-03 | security | `.env.example` 是否需要单独例外策略？ | 需要；仅在 placeholder-only / no-secret scan PASS 后作为条件 include。 | 备选 A：始终排除；备选 B：始终允许。 | 推荐兼顾实用和安全；始终排除降低易用性；始终允许风险高。 | 需要后续扫描。 | 若 scan 命中真实值则排除。 |
| DQ-CP2-CR066-04 | implementation | tracked `reports/**` / `runs/**` 如何处理？ | 默认不进入 clean public baseline；若要发布需单独风险接受。 | 备选 A：接受当前分支 as-is；备选 B：全部纳入私有仓库；备选 C：history rewrite 移除。 | 推荐符合 CR062 policy；as-is 快但风险高；私有仓库改变目标；rewrite 高风险。 | public publication 延迟。 | 用户接受风险时需新的 DQ；rewrite 需独立 CR。 |
| DQ-CP2-CR066-05 | follow_up_tracking | CR066 后续执行如何安排？ | CR066 只定策略；后续 CR067 Clean Publication Execution Gate 单独授权。 | 备选 A：回到 CR062 继续；备选 B：直接启动 NAS/lake/runtime；备选 C：取消 publication。 | 推荐保持边界；回 CR062 仍有 blocker；NAS/lake/runtime 不相关；取消会放弃 GitHub baseline。 | 增加一个执行 CR。 | 用户要求直接执行时必须重新做 CP5 授权。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP2-CR066-01..05 推荐方案。 |
| 备选方案 | 保持 blocked、接受当前分支风险、合并执行、history rewrite、取消 publication。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全 / 权限、交付影响。 |
| 优劣分析 | 见上表。 |
| 风险与回退 | 当前没有 Git 写动作；若策略不通过则 CR062 保持 blocked。 |
| 用户需决策事项 | DQ-CP2-CR066-01..05。 |

不授权项：CP2 通过不授权 `git add`、`git commit`、`git remote add`、`git push`、`git tag`、branch creation / rename、history rewrite、force push、remote deletion、NAS/data lake/provider/catalog/runtime、凭据读取、QMT/MiniQMT runtime、CR046 recovery、physical move 或 bulk import rewrite。

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR062 blocker 已记录 | PASS |
| 用户同意 follow-up | PASS |
| CR066 编号不冲突 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 影响分析完成 | PASS |
| 决策项完整 | PASS |
| 不授权项列明 | PASS |
| 策略-only 边界明确 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户确认后可进入 CR066 CP3 / CP5 审查 | PASS |

## Deliverables

- `process/checks/CP2-CR066-REPOSITORY-HYGIENE-BASELINE.md`
- `process/checkpoints/CP2-CR066-REPOSITORY-HYGIENE-BASELINE.md`
- `process/context/CP2-CR066-REPOSITORY-HYGIENE-CONTEXT.yaml`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T10:46:59+08:00
- 修改意见：N/A
- 风险接受项：接受 DQ-CP2-CR066-01..05 推荐方案；CR066 启动并限定为 repository hygiene / clean publication strategy，CR062 保持 blocked，CR046 继续暂停。CP2 approved 不授权任何 Git 写动作、NAS/data lake/provider/catalog/runtime/凭据操作、physical move、bulk import rewrite 或 CR046 recovery。
