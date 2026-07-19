---
artifact: "CR051-CP5-INDEPENDENT-REVIEW-FINDINGS"
reviewer: "meta-qa-critical"
lane: "lane-quality"
input_type: "review_findings"
round: 1
status: completed
governance_mode: review-gated
---

# Review Findings

## 1. 审查范围

- 目标对象：CR-051 的 4 份 full LLD、1 份 technical-note、5 份单 Story CP5 result、Development Plan 和 CP4 result。
- 审查目标：独立验证 schema 可实现性、跨 Story port/evidence DAG 一致性、O-AW-01/02/03 义务和授权边界。
- 审查依据：`quality-review`、`review-artifact-protocol`、CP5 capsule、CP3 HLD/ADR 和 5 个 Feature pack。
- read-expansion：`RE-20260718T071723Z0000-60e96340`、`RE-20260718T071723Z0000-eb5d1bf6`、`RE-20260718T071723Z0000-02e97ad6`、`RE-20260718T071723Z0000-c99442d7`、`RE-20260718T071723Z0000-d55c2fef`、`RE-20260718T071723Z0000-e2c28ff7`。

## 2. Findings

### Advisor Table（CP3 方案形成输入适用）

N/A：本轮是 CP5 `review_findings`，不是 CP3 方案形成输入。

<!-- findings-table -->

| ID | Severity | Rule Ref | Evidence | Impact | Suggestion | Anchor |
|----|----------|----------|----------|--------|------------|--------|
| CP5-QA-R1-F01 | 严重 | Design Review 1/3/5 | ST-AW-001 将 `PathRef.anchor` 枚举限定为 `project_root/artifact_control_root/sibling_root`，但 `docs_relative/process_relative` 又要求 `anchor=project_worktree`。 | schema 按自身类型约束不可构造；routing contract 无法稳定实现。 | 把 `project_worktree` 加入 anchor 枚举，并显式定义 anchor DAG、允许父子关系与拒绝环；同步测试/DoD/result。 | `STORY-ST-AW-001-project-first-routing-LLD.md#51-值对象` |
| CP5-QA-R1-F02 | 严重 | Design Review 1/3/5 | ST-AW-002 的 `WorktreeHealth` 只含 decision/state/journal/ref/reasons，rich identity/common-dir/HEAD/OID/dirty/Git-op/role/digest 位于独立 `WorktreeObservation`；ST-AW-003 却把这些 rich 字段声明为 `WorktreeHealth` 强制输出。 | ST-AW-002→003 port 无法按冻结 schema 对接；实现者必须猜测字段或复制状态。 | 冻结一个公共 port：`WorktreeHealth` 明确封装 immutable `WorktreeObservation`（或等价 rich snapshot）及 health decision；ST-AW-003 只从该封装读取，禁止发明平行字段。同步两份 LLD、测试矩阵与 results。 | `STORY-ST-AW-002-recoverable-project-worktree-LLD.md#51-数据对象`；`STORY-ST-AW-003-heterogeneous-git-legs-LLD.md#port-contracts` |
| CP5-QA-R1-F03 | 严重 | Design Review 1/3/5 | ST-AW-003 把 `append_receipt` 放入 immutable `LegResult` 且作为 PASS 必需条件，同时又要求先 append candidate 才取得 receipt、随后才允许 PASS。 | 形成 payload/receipt 自引用或需要二次改写 immutable result；single-write 与 digest 可复算性无法同时成立。 | 改为无自引用 DAG：`LegResultDraft`（无 receipt、可确定 digest）→ append-only writer → external `LegResultWriteReceipt` → `PublishedLegResult`/validated handle；aggregate 只消费 matching published handles。同步 ST-AW-003、ST-AW-004 及相关 results。 | `STORY-ST-AW-003-heterogeneous-git-legs-LLD.md#53-attempt-与结果`；`#核心处理流程` |

## 3. 汇总结论

- blocking_count: 3
- required_count: 3
- optional_count: 0
- recommended_next_action: `revise-and-resubmit`
- decision_impact: `不得打开 CP5 人工门；定点修订 ST-AW-001/002/003/004 并重新生成受影响 CP5 results 后，进行第二轮独立复核。`
- trade_off_note: `修订增加少量 schema/对象层次，但消除不可实现、自引用和跨 Story 猜字段；不改变用户批准的 project-first/integration/main 人工边界。`

## 4. 待确认项

- None：三项均属于已批准架构内的 implementation-design 一致性修复，不新增用户业务或架构决策。
