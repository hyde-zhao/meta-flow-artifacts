# CP3-CR062 HLD Review 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。CR062 架构推荐 gate-first fail-closed publication：CP2 / CP3 / CP5 approved 后，先执行 post-approval preflight 和 scan，冻结 commit manifest，再按 exact target remote 执行普通 commit / remote add / push。tag、branch rename、history rewrite、force push 和 remote deletion 均不授权。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR062-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；架构冲突时读 scope / manifest / scan / rollback |
| 全文档读取扩展 | 已读取 CR062 scope、publication manifest、scan plan、rollback ref |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前无未批准 CR062 DQ。 |
| 自动预检结果 | `process/checks/CP3-CR062-HLD-CONSISTENCY.md` | scanned | 6 | 5 | 架构风险转 DQ。 |
| scope | `docs/release/GIT-PUBLICATION-SCOPE-CR062.md` | scanned | 8 | 3 | include/exclude 合并。 |
| manifest | `docs/release/REPOSITORY-PUBLICATION-MANIFEST-CR062.yaml` | scanned | 12 | 4 | post-approval sequence 和 forbidden actions 合并。 |
| rollback | `docs/release/ROLLBACK-REF-CR062.md` | scanned | 6 | 2 | fail-closed 和 no rewrite 转 DQ。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR062-01 | architecture | GitHub publication 架构是否采用 gate-first fail-closed sequence？ | 采用 CP2/CP3/CP5 approve -> preflight/scan -> manifest -> commit -> remote -> push。 | 备选 A：门禁前先 remote precheck；备选 B：门禁后跳过 scan 直接 push。 | 推荐方案审计完整；门禁前 precheck 会触发外部面；跳过 scan 风险高。 | 发布会多一步扫描。 | scan 失败时 BLOCKED，不执行 push。 |
| DQ-CP3-CR062-02 | security | scan gate 是否 fail-closed？ | 是；secret/data/large/path scan 任一 BLOCKING/HIGH 即停止。 | 备选 A：只警告不阻断；备选 B：只做 secret scan。 | 推荐安全；只警告风险高；只 secret scan 覆盖不足。 | 可能阻断发布直到清理。 | 用户单项豁免需新风险接受 DQ。 |
| DQ-CP3-CR062-03 | implementation | branch policy 是否保持当前分支、不 rename？ | 保持当前分支，不做 branch rename；目标分支冲突时停止。 | 备选 A：rename to main；备选 B：新建发布分支。 | 推荐最少侵入；rename 破坏本地历史预期；新分支增加治理。 | 若 GitHub 默认分支不匹配，需要后续决策。 | 分支冲突时不 push，回到人工确认。 |
| DQ-CP3-CR062-04 | implementation | process/docs evidence 是否纳入 Git publication？ | 选择性纳入 CR058-CR062 迁移审计必需 evidence；禁止全量历史过程文件。 | 备选 A：code-only；备选 B：全量 process/docs。 | 推荐可追踪且控制体积；code-only 审计弱；全量风险高。 | 需要 manifest 审查。 | 若 evidence 含敏感或过大文件，移出 manifest。 |
| DQ-CP3-CR062-05 | security | remote handling 是否 fail-safe？ | 若 remote absent 可 post-approval add exact target；若 existing remote differs，停止并问用户。 | 备选 A：直接 set-url；备选 B：添加 secondary remote。 | 推荐避免误改远端；set-url 风险高；secondary remote 复杂。 | 可能需要用户再决策。 | remote 冲突时 BLOCKED。 |

不授权项：CP3 通过不授权 Git remote add / set-url、commit、push、tag、branch rename、history rewrite、force push、NAS/data lake/provider/catalog、凭据读取、QMT/MiniQMT runtime、CR046 recovery、physical move 或 bulk import rewrite。

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR062 baseline 已创建 | PASS |
| scope / manifest / scan / rollback 存在 | PASS |

## Checklist

| 检查项 | 结果 |
|---|---|
| 架构方案有备选和切换条件 | PASS |
| fail-closed scan 策略明确 | PASS |
| branch / remote conflict policy 明确 | PASS_WITH_RISK |
| process evidence 策略明确 | PASS_WITH_RISK |
| 外部操作边界明确 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户确认后可进入 CP5 repository publication readiness gate | PASS |

## Deliverables

- `process/checks/CP3-CR062-HLD-CONSISTENCY.md`
- `process/checkpoints/CP3-CR062-HLD-REVIEW.md`
- `process/context/CP3-CR062-DESIGN-CONTEXT.yaml`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T09:59:22+08:00
- 修改意见：用户回复“同意”，按 `approve` 解析，接受 DQ-CP3-CR062-01..05 的推荐方案。
- 风险接受项：接受 gate-first / fail-closed publication 架构、扫描硬门、当前分支不 rename、远端冲突 fail-safe 和选择性过程证据纳入策略；CP3 本身不授权 tag、branch rename、history rewrite、force push、NAS/data lake/provider/catalog/runtime/凭据、physical move、bulk import rewrite 或 CR046 recovery。
