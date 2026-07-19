---
artifact: "CP5-CR172-LLD-REVIEW-R1-SUMMARY"
round: 1
status: complete
decision: revise
blocking_count: 2
required_count: 2
optional_count: 1
---

# Review Summary

## 1. 输入清单

- findings_files:
  - `process/checks/CP5-CR172-LLD-REVIEW-R1-FINDINGS.md`
- reviewed_objects:
  - `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md`
  - `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md`
  - `process/stories/STORY-CR172-S03-nas-replica-verification-LLD.md`
  - `process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md`
  - `process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-LLD.md`
  - `engine/mature_multifactor_research.py` 与 `engine/experiment_family_lineage_store.py` scoped source facts

## 2. 严重度汇总

| Severity | Count | Owner |
|----------|-------|-------|
| blocking | 2 | `meta-se-critical + lineage owner + research methodology owner` |
| required | 2 | `meta-dev / meta-se-critical` |
| optional | 1 | `meta-dev` |

## 3. 决策

- decision: `revise`
- rationale: `五份 LLD 结构均合格，三项用户整改的大方向也已进入设计，但两个核心承诺当前不可实施：现有 append-only lineage 无法在 finish 中途失败后消除已写入的 SUCCEEDED 事件；现有 net_forward_return 是带可配置 horizon 的 forward-label proxy，却被映射为 next_rebalance_date 上的 trial portfolio simple return。另有 fixture/real decision 类型边界和 S02→S03 seal digest/public type 尚未闭合。`
- next_checkpoint: `仍停留在 CP5 准备；完成 R2 设计修订并通过独立重审后再发起 CP5 人工门禁。`

## 4. Decision Brief 输入

| 字段 | 内容 |
|---|---|
| 推荐决策 | `修改`：暂不 approve CP5，先修订 F-001～F-004 并进行 R2 correlation review。 |
| 备选方案 | `备选 A（推荐）`：增加 lineage 原子/补偿前置设计，并实现真实持有期 trial-return producer；保留五 Story 主链。`备选 B（治理收窄）`：本 CR 只交付 schema、hash、fixture ports 与 zero-op guards，移除 native runner hook/真实 trial-return 声明，把 producer+lineage 事务拆为后续高风险 CR。 |
| 影响维度 | `用户价值`：避免把 proxy 当真实 trial return；`实现复杂度`：A 增加 lineage/return 计算工作，B 延后 activation；`可验证性`：两者都比当前不可满足 oracle 可计算；`安全权限`：需闭合 fixture/real decision boundary；`维护成本`：S02→S03 seal digest 单一真相会减少双实现。 |
| 优劣分析 | A 能保留 PATH-I 对 activation 的直接价值，但需要跨 lineage owner 的显式授权和更高测试成本；B 风险最低、可较快交付合同层，但不产生 native trial-return，也不能声称为 PATH-C/A 的完整前置。 |
| 风险与回退 | 若 lineage owner 不批准原子/补偿能力，切换 B；若不能证明现有 source 是定义正确的 period return，则禁止沿用 `trial_portfolio_return_series@v1` 名称，回到 proxy/deferred。任何路线均保持六真实动作 authorized/executed=`0/6`,`0/6`。 |
| 用户需决策事项 | R2 前需确认：lineage 原子性采用前置 CR 还是收窄 claim；canonical source 采用真实持有期回报还是 proxy/deferred。 |

## 5. 后续动作

1. 由 `meta-se-critical` 处理 F-001/F-002 的 HLD/Feature/ADR 影响与 owner 路由，不在 LLD 中用实现细节掩盖。
2. 由 S01/S02/S03 LLD owner 处理 F-003/F-004，并同步 S04/S05 public contract inventory 与测试。
3. 机械修复 F-005 后，重跑五份 `lld-check`、跨合同 correlation、source fact review 与 review artifact validator。
4. blocking/required 均为 0 后，才能生成 `decision=proceed` 的 CP5 Decision Brief 输入。

