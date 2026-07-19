---
status: draft-for-cp4
version: "1.1"
feature_id: "FEAT-AW-02"
related_story: "ST-AW-002"
open_obligations: ["O-AW-01", "O-AW-02"]
---

# FEAT-AW-02 Feature Tasks

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 初版 W01–W11 任务 DAG。 |
| 1.1 | 2026-07-19 | Host Orchestrator（inline-fallback） | 回写实施状态并增加 W12–W14 终验回修任务。 |

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| TASK-AW-W01 | 1 | 冻结 registration/health/operation/bootstrap/capacity/journal schema、phase 与 error enum | DESIGN、Domain Map、ADR-AW-001/005 | `meta_flow/workspace/project_worktree.py`、`worktree_capacity.py`、`worktree_journal.py` | primary | schema/state unit | completed |
| TASK-AW-W02 | 2 | 扩展 argv-only worktree/common-dir/symbolic-HEAD/Git-op/fresh-ref probes | W01、existing git adapter | `meta_flow/workspace/git_sync.py` | shared；需单写窗口 | `tests/test_workspace_git_sync.py` | completed |
| TASK-AW-W03 | 3 | 实现 O-AW-01 deterministic capacity profiler、512MiB bounded eligibility 与 measured oracle | W01 | `meta_flow/workspace/worktree_capacity.py`、`tests/test_cr051_worktree_faults.py` | primary | CAP-01..11；0 false-safe/underestimate | completed |
| TASK-AW-W04 | 4 | 实现 O-AW-02 store-local temp/fsync/replace/dir-fsync/checksum/readback 与 append-only chain | W01 | `meta_flow/workspace/worktree_journal.py`、fault tests | primary | DUR-01..14；early mutation=0 | completed |
| TASK-AW-W05 | 5 | 实现 project lock、registration、fresh health、branch role、check/list/stale diagnosis | W01/W02 + FEAT-AW-01 PASS API | `meta_flow/workspace/project_worktree.py` | primary | WT-11/12/14、TC-AW-007/010/015 | completed |
| TASK-AW-W06 | 6 | 实现 integration fresh observe 与 exact-OID create-only bootstrap/race classification | W02/W05 | 同上 | primary | WT-01..04、TC-AW-004 | completed |
| TASK-AW-W07 | 7 | 实现 create 与 independently authorized safe-remove，禁止 force/rm/branch-delete | W04..06 | 同上 | primary | TC-AW-004/011；dangerous argv=0 | completed |
| TASK-AW-W08 | 8 | 实现 CP3-DC-01 switch/post-observe 与只读/manual-only resume | W02..05 | 同上 | primary | WT-05..10；恢复幂等；mutation resume=0 | completed |
| TASK-AW-W09 | 9 | 建立两项目 existing-control+sibling-worktree/bare-remote 集成 fixture 与 dry-run golden | W06..08 | `tests/test_cr051_project_worktree.py` | primary | TC-AW-005/012/014；crossover=0 | completed |
| TASK-AW-W10 | 10 | 统一 CLI facade 接线（若 core Development Plan 指定本 Story 为 owner） | W05..09 | `meta_flow/cli.py` | shared；条件式单写 | CLI dry-run/help contract | completed |
| TASK-AW-W11 | 11 | 执行 Ruff、定向/回归、dangerous-command、真实路径/ref deny 与 design-contract 审计 | W01..10 | evidence only | none | TEST-PLAN 命令与 CP5 proof | completed |
| TASK-AW-W12 | 12 | 增加 attempt-bound typed CapacityProof 与同锁 mutation 前重验 | W03/W08 | `worktree_capacity.py`、`project_worktree.py` | primary | proof 错绑/过期/revocation mutation=0 | completed |
| TASK-AW-W13 | 13 | 持久化 owner/calibration/revocation，并校验 record identity/phase transition | W04 | `worktree_journal.py` | primary | DUR-15..18 | completed |
| TASK-AW-W14 | 14 | 将 resume 保守偏离回写为 design delta 并验证 terminal 幂等 | W08/W12/W13 | DESIGN/LLD/tests/evidence | primary | delta merged；重复 execute mutation=0 | completed |

## DAG 与并行边界

```text
W01 -> W02
W01 -> W03
W01 -> W04
W02 + FEAT-AW-01 API -> W05
W05 + W02 -> W06
W04 + W05 + W06 -> W07
W02 + W03 + W04 + W05 -> W08
W06 + W07 + W08 -> W09 -> W10? -> W11
```

- W03 与 W04 可并行，文件不重叠；二者都通过后才允许 W08 进入任何 mutation path。
- `git_sync.py`、`cli.py` 是 shared 文件，必须由 core Development Plan 安排单写窗口；本 pack 不授权并行修改。
- FEAT-AW-02 只读 import FEAT-AW-01 的 resolver contract；不得修改其主文件。
- `git_branch_lifecycle.py` 默认不修改；若 Story LLD 证明需要抽公共纯类型，必须先提交 design delta，防止 paired-default 语义泄漏。

## CP5 强制关闭条件

| Obligation | Required evidence | 失败路由 |
|---|---|---|
| O-AW-01 | CAP-01..11；适用 bounded profile false-safe=0、underestimate=0；无权限/无法枚举 fail closed | `auto_switch=disabled`，不得以 512MiB 静默放行 |
| O-AW-02 | DUR-01..14；ENOSPC/EACCES/fsync/replace/corruption/kill/cross-device 均符合；persistence fault 前 Git mutation=0；resume 幂等 | Git mutation 前 BLOCKED，切 manual-only |
| CP3-DC-01 | WT-05..10；命令成功不等于 terminal；第三态保留现场 | CP5 FAIL/回设计，不得 destructive fallback |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| N/A | N/A | 无新架构决策；O-AW-01/02 是 CP5 可执行证明义务，不是本阶段 blocker | CP5 reviewer | 按失败路由 fail closed，不能风险接受后继续 auto switch |
