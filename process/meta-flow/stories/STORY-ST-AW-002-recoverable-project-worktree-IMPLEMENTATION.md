---
status: implemented
version: "1.1"
story_id: "ST-AW-002"
story_slug: "recoverable-project-worktree"
feature_id: "FEAT-AW-02"
implementation_type: "mixed"
source_story: "process/stories/STORY-ST-AW-002-recoverable-project-worktree.md"
source_design_evidence: "process/stories/STORY-ST-AW-002-recoverable-project-worktree-LLD.md"
created_by: "meta-dev-debugger"
created_at: "2026-07-18T12:41:35+00:00"
updated_at: "2026-07-19T00:00:00+00:00"
---

# Implementation: ST-AW-002 可恢复项目 Worktree

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 在 fixture-only 边界内落成长期 project worktree 的 typed observation/health、capacity proof、durable intent、exact-OID create-only bootstrap、create/check/list/safe-remove 与可恢复 switch/resume 原语。 |
| 行为变化 | 新增三个 ST-AW-002 primary 模块；最小扩展 argv-only Git adapter；所有不确定状态在 Git mutation 前 fail closed，命令退出码不单独形成成功。 |
| 范围边界 | 不修改 `meta_flow/cli.py`、ST-AW-001 primary、真实 artifact repo/ref/worktree/link/migration；不实现 artifact main 自动同步；不使用 reset/clean/stash/force/branch-delete/rm 恢复。 |
| CP6 证据 | 本文、`process/returns/ST-AW-002.CP6.return.json` 与 `process/evidence/ST-AW-002.CP6.index.json`；本 lane 不批准 CP6。 |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| Story | `process/stories/STORY-ST-AW-002-recoverable-project-worktree.md` | 范围、量化 AC、文件 owner、W01–W11。 |
| Story LLD | `process/stories/STORY-ST-AW-002-recoverable-project-worktree-LLD.md` | rich observation port、capacity 算法、durable chain、状态机与禁止动作；读取前已登记 `deep_review`。 |
| CP5 | `process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md` | 批次 `approved`；O-AW-01/02 不可 waiver。 |
| 上游 Return | `process/returns/ST-AW-001.CP7-R2.return.json` | `verified_with_risk`，公共 route/context contract 未变。 |
| Feature | `process/docs/features/cr051-worktree/{DESIGN,TEST-PLAN,TASKS}.md` | CAP-01..11、DUR-01..14、WT-01..14、TC-AW 和 TASK 映射。 |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| process route health | PASS | 当前 `STATE.current.json` 指向本 Story CP6 packet，`CURRENT.json.health=ok`；不创建或修复 process route。 |
| 上游 Feature 设计存在 | PASS | DESIGN/TEST-PLAN/TASKS 已完整读取。 |
| Story 范围明确 | PASS | `status=dev-ready`，11 个 TASK 与 7 条量化 AC 完整。 |
| 待确认问题已关闭 | PASS | CP5 checkpoint `status=approved`；LLD `open_items=0`。 |
| 上游依赖满足 | PASS | ST-AW-001 CP7 R2=`verified_with_risk`，`contract_changes.public_api_changed=false`。 |
| 当前 Wave / dev_gate 满足 | PASS | Story `wave=W2` 且 `design_evidence_confirmed/lld_confirmed/dependencies_satisfied/file_conflict_free=true`。 |
| 文件所有权无冲突 | PASS | 工作区仅有 ST-AW-001 的 `routing.py`/`project_artifact_routing.py` 及其测试改动；本 Story 不修改这些文件。`git_sync.py` 当前无未提交冲突，进入 ST-AW-002 单写窗口。 |
| 真实运行授权 | PASS（fixture-only） | 只允许 `tmp_path`、临时 worktree 与 local bare remote；真实 artifact 变更未授权。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 必须 | 验证方式 |
|---|---|---|---|---|
| `meta_flow/workspace/worktree_capacity.py` | code / schema | deterministic upper bound、512 MiB bounded profile、双 FS 与 calibration revoke | yes | CAP-01..11 |
| `meta_flow/workspace/worktree_journal.py` | code / schema | store-local exclusive temp、fsync/replace/dir-fsync/readback、checksum chain、seal、scan/lock | yes | DUR-01..14 |
| `meta_flow/workspace/project_worktree.py` | code / schema | observation/health、branch role、bootstrap/create/switch/resume/safe-remove | yes | PORT-W-01..08、WT-01..14 |
| `meta_flow/workspace/git_sync.py` | shared code | exact remote observation、create-only ref、worktree/common-dir/HEAD/Git-op probes | yes | shared regression + dangerous argv scan |
| `tests/test_cr051_worktree_faults.py` | guardrail-test | CAP/DUR deterministic fault matrix 与 counters | yes | pytest fixture IDs |
| `tests/test_cr051_project_worktree.py` | guardrail-test | WT/PORT/TC-AW、两项目和 local bare remote fixture | yes | pytest fixture IDs |
| 既有 Git 回归测试 | guardrail-test | 证明 CR-050 与 workspace Git 行为不回归 | yes | targeted pytest |
| IMPLEMENTATION / Return / Evidence | docs-handoff | CP6 可审计交接，不包含 CP6 批准 | yes | return-check/evidence-index/checksum |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| `WorktreeObservation -> WorktreeHealth(observation)` 唯一 port | LLD §5/§6 | `project_worktree.py` | frozen dataclasses + pure evaluator；unknown/stale/mismatch 非 HEALTHY | PORT-W-01..08 |
| bounded profile 必须 0 false-safe/underestimate | O-AW-01 | `worktree_capacity.py` | 可复算 upper/3:2/floor、校准匹配、反例 revoke | CAP-01..11 |
| 任一 durable fault 必须发生在 Git mutation 前 | O-AW-02 | `worktree_journal.py`、coordinator | intent+seal 完整链验证后才返回 mutation capability | DUR-01..14 |
| integration 仅 exact-OID create-only | ADR-AW-005 | `git_sync.py`、`project_worktree.py` | exact query；existing 零 mutation；reject 后 fresh same/different 分类 | WT-01..04、TC-AW-004 |
| switch 退出码不形成终态 | CP3-DC-01 | `project_worktree.py` | 每次 attempt 后 fresh observation；third/dirty/op-active preserve | WT-05..10、TC-AW-015 |
| safe remove 独立授权且只允许 non-force | LLD §6.10 | `project_worktree.py` | exact identity/idle/clean/no-active/recovery 检查 | WT-11..14、TC-AW-011 |
| sibling/project 隔离 | REQ-AW-NF003 | journal namespace、identity 与 tests | project-scoped lock/store/refs；cross-project mismatch fail closed | TC-AW-005/010/014 |
| 禁止危险恢复与真实目标 | Story / authz | 所有 runner/fixtures | 固定 argv、危险 token 扫描、tmp-only mutation gate | SEC-W-01..04 |

## 6. 单元测试 / Fixture 计划

| 测试对象 | 测试类型 | 输入 / Fixture | 期望 | 覆盖风险 | 状态 |
|---|---|---|---|---|---|
| capacity profiler | unit/property | bounded entries、512 边界、profile mismatch、EACCES/unknown、双 FS、measured write oracle | CAP-01..11；PASS 集合 false-safe/underestimate=0；反例立即 REVOKED | false-safe checkout | passed（11/11） |
| journal | deterministic fault fixture | write/flush/file-fsync/replace/dir-fsync/readback/kill/cross-device/corrupt/gap | DUR-01..14；seal 前 Git calls=0；resume 稳定 | torn/non-durable intent | passed（14/14） |
| public port | unit/contract | rich snapshot、typed unknown、stale/digest/project/route/journal mismatch | PORT-W-01..08 | 下游 schema 漂移 | passed（8/8 contract） |
| bootstrap | local bare remote | absent/existing/race-same/race-different | create-only 恰好一次或零 mutation | remote overwrite | passed（WT-01..04） |
| lifecycle | fake runner + temp worktree | exit0 mismatch、timeout target/original/third、durable rollback、remove negative | WT-05..14 | destructive recovery | passed（WT-05..14） |
| isolation | two-project temp fixture | idle/active refs、dirty sibling、same CR id、project lock/path audit | cross-project touched=0 | sibling contamination | passed（TC-AW-005/010/014） |
| regression | existing suites | workspace git + CR-050 branch lifecycle | 全部通过 | shared adapter regression | passed |

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 输出文件 | 局部验证 | 状态 |
|---|---|---|---|---|---|
| IMPL-AW2-S1 | W01/W02/PORT | schema、observation、health evaluator | `project_worktree.py` + tests | PORT-W-01..08 | done |
| IMPL-AW2-S2 | W03/W04/O-AW-01 | capacity + fault tests | `worktree_capacity.py` | CAP-01..11 | done |
| IMPL-AW2-S3 | W05/W06/O-AW-02 | journal + fault tests | `worktree_journal.py` | DUR-01..14 | done |
| IMPL-AW2-S4 | W07/ADR-AW-005 | shared exact/create-only primitives | `git_sync.py` + regression | WT-01..04 | done |
| IMPL-AW2-S5 | W08/W09/CP3-DC-01 | lifecycle mutation/recovery/safe-remove | `project_worktree.py` | WT-05..14 | done |
| IMPL-AW2-S6 | W10/W11/TC-AW | two-project/local-bare fixtures、整体回归与证据 | tests + evidence | full verification plan | done |

## 8. 变更说明

| 文件 | 动作 | 实际行为 |
|---|---|---|
| `meta_flow/workspace/worktree_capacity.py` | create | 冻结 capacity schema、3/2 deterministic upper bound、calibrated 512 MiB floor、双 FS、measured outcome revoke。 |
| `meta_flow/workspace/worktree_journal.py` | create | project handle lock、append-only checksum chain、intent seal、same-dir temp/file-fsync/replace/dir-fsync/readback 和 fail-closed scan。 |
| `meta_flow/workspace/project_worktree.py` | create | registration/observe/health/check/list/plan/bootstrap/create/switch/resume/safe-remove；authorization/capacity 缺失时 pre-mutation BLOCKED。 |
| `meta_flow/workspace/git_sync.py` | modify | exact remote ref typed observation、ordinary create-only push 与 read-only worktree/common-dir/HEAD/status probes。 |
| `tests/test_cr051_project_worktree.py` | create | PORT、WT、TC-AW、local bare remote、两项目与危险 argv fixture。 |
| `tests/test_cr051_worktree_faults.py` | create | CAP/DUR、measured oracle、deterministic filesystem faults、project lock fixture。 |
| `tests/test_workspace_git_sync.py` | modify | exact ref 与只读 probes 共享回归。 |
| `tests/test_git_branch_lifecycle.py` | unchanged / executed | CR-050 lifecycle 回归；无 Story diff。 |

## 9. 平台差异处理

| 平台 | 检查项 | 预期 | 结果 |
|---|---|---|---|
| POSIX | handle lock、file/dir fsync、same-directory replace | 能证明则启用；异常 fail closed | PASS（Linux temp fixture） |
| Windows | locking/dir-fsync 能力不足时 | 不声明 durable，manual-only/BLOCKED | N/A（未授权真实平台 pilot；代码无能力证明时 fail closed） |
| Git | argv-only；版本差异不得引入危险 fallback | probe unknown 即阻断 | PASS（local repo/worktree/bare remote） |
| Claude/Codex/install | 本 Story 不改 Prompt/Agent/安装器 | N/A | N/A |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| 新测试首次收集 | EXPECTED FAIL | 目标模块/共享原语尚不存在：2 collection errors，形成测试先行红灯基线。 |
| 指定完整 pytest suite | PASS | `83 passed in 4.60s`；CAP-01..11、DUR-01..14、WT-01..14、PORT 与适用 TC-AW 加共享回归。 |
| Ruff（4 模块 + 4 测试文件） | PASS | `All checks passed!`。 |
| `python -m py_compile`（4 模块） | PASS | exit 0。 |
| `git diff --check` | PASS | exit 0。 |
| dangerous argv / boundary audit | PASS | 实现只构造 exact query、ordinary push、read probes、`worktree add`、`switch`、non-force `worktree remove`；reset-hard/clean/stash/force/branch-delete/rm 恢复构造数为 0。 |
| 真实目标审计 | PASS | 所有执行 Git mutation 仅发生于 pytest `tmp_path` 与 local bare remote；真实 artifact ref/worktree/link/migration/commit/push/main-sync 为 0。 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| 真实 shared remote/worktree pilot | 本 Story 明确未授权任何真实 artifact mutation | CP7/CP8 后独立 `runtime_authorization` 决策；不能以风险接受替代 fixture 证明。 |
| Windows 真实 dir-fsync/locking pilot | 当前执行环境为 Linux 且不授权外部 runtime | 不支持能力时实现必须 fail closed；后续平台验证。 |

## 12. 风险与回滚

| Risk ID | 风险 | 影响 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|---|
| R-AW2-01 | capacity false-safe | checkout 中间态 | 默认 disabled、calibrated bounded profile、revoke | 任一 false-safe/underestimate 禁用 auto switch。 |
| R-AW2-02 | journal 未 durable 却授权 Git | 盲重放/错误恢复 | intent+seal 双阶段、chain/readback、fault injection | DUR 任一失败保持 Git mutation=0/manual-only。 |
| R-AW2-03 | integration race 覆盖外部 ref | 数据/历史破坏 | ordinary create-only + fresh exact reobserve | different/unknown race BLOCKED。 |
| R-AW2-04 | timeout 后 destructive recovery | 用户改动丢失 | fresh observe + preserve third/dirty/op-active | 禁止 reset/clean/stash/force/delete。 |

## 13. 设计缺口反馈

CP8 终验发现实现相对 LLD 的保守收窄未被记录。已创建并合并 `process/design-deltas/ST-AW-002.delta.json`，同步 Feature DESIGN 1.2 与 LLD 1.2.0；当前 `design_delta_required=true` 且 `design_delta_status=merged`。

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 独立重跑 CAP-01..11、DUR-01..14、PORT-W-01..08、WT-01..14 与适用 TC-AW；不得把本 lane 的断言计数当人工批准。
- 核验 Git mutation counter、危险 argv=0、两项目 touched-path/ref/index.lock=0 和真实 artifact 变更=0。

### Review 关注点

- capacity profile 是否只对完整可枚举的 bounded transform 集合 PASS。
- durability seal 是否真的晚于 file/dir fsync 和 readback，且 resume 不消费 gap/corrupt/unsealed chain。
- 命令退出码是否在任何路径上绕过 fresh observation。

### Doc 关注点

- CLI 由 ST-AW-004 接线；本 Story 只交付 core 调用契约，不应在用户文档中声称真实 worktree 已迁移或 auto switch 已默认启用。

## 15. CP8 终验回修实施（R2）

| 回修项 | 实现结果 | 验证 |
|---|---|---|
| 只读/manual-only resume | `resume_worktree_operation` 只 scan + fresh observe，永远不执行 Git、不创建 rollback attempt | resume mutation=0；目标/原态 NO_CHANGE，第三态 RECOVERY_REQUIRED |
| typed CapacityProof | 新增与 project/repo/operation/attempt/before/target/profile/calibration/FS/expiry 绑定的 proof 与 digest | 错绑、过期、revoked、profile/counter 漂移均 pre-Git BLOCKED |
| owner/calibration persistence | durable store 新增 `owner.json` 与 `calibrations/<profile-digest>.json` | 跨项目复用拒绝；重载后 revocation 持续有效 |
| explicit phase | `CAPACITY_PROOF→INTENT→INTENT_SEAL→OBSERVATION_REQUIRED→FINAL_OBSERVATION` | 非法 transition、identity tamper、chain mismatch 拒绝 |
| switch terminal idempotency | 同一 operation lock 下 proof/intent/fresh-before 重验；一次 switch 后 fresh observe | final attempt 重放新增 Git mutation=0 |

R2 定向 worktree 回修 74 项通过；CR-051 跨模块集合 307 项与 58 个 subtests 通过；全仓 697 项与 70 个 subtests 通过。Ruff scoped format/check、Python 3.11 compile、delivery guardrail 与 `git diff --check` 全部通过。真实 worktree/ref/remote/link/migration mutation=0。
