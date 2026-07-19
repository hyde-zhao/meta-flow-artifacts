---
story_id: "ST-AW-005"
story_slug: "read-only-migration-handoff"
change_id: "CR-051"
stage: "CP6-R2"
status: "implemented-r2"
version: "1.1"
updated_at: "2026-07-19"
implementation_owner: "meta-dev/dev-yang"
design_evidence_ref: "process/stories/STORY-ST-AW-005-read-only-migration-handoff.md#技术说明"
---

# ST-AW-005 只读迁移预检与人工交接实现

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev/dev-yang | 实现 11 分区 immutable/portable manifest、allowlisted no-follow 读取、fail-closed readiness、validation/rollback handoff 与 O-AW-03 candidate-only 评估器；补齐 TDD、相邻回归和风险交接。 |
| 1.1 | 2026-07-19 | meta-dev-debugger/dev-yang | 关闭 `REV-AW005-001`：deny policy 前移到每个 source/target portable descendant 的 filesystem probe/mapping 前门，并阻断相对 symlink 归一化后的 denied target；新增三类 R2 红灯与回归证据，schema/no-follow/O-AW-03 不变。 |

## 实现前置检查

| 检查项 | 结果 | 证据 |
|---|---|---|
| CP5 设计证据已确认 | PASS | Story `lld_gate.status=approved`、`confirmed=true`，approval ref 为 `process/checkpoints/CP5-CR051-ALL-STORIES-LLD-BATCH.md` |
| W4 runtime 依赖满足 | PASS_WITH_RISK | `process/returns/ST-AW-003.CP7-R2.return.json` 与 `process/returns/ST-AW-004.CP7.return.json` 均为 `verified_with_risk` |
| 文件所有权 | PASS | 仅写 packet 允许的 migration 模块、定向测试与本 Story CP6 证据；未修改 CLI、routing、worktree 或其他共享源码 |
| 运行授权边界 | PASS | 未执行真实文件迁移、软链接、Git/worktree/ref/remote、helper、scheduler、commit、push 或 publish |
| 阻塞项 | 0 | 无需新增需求、架构或文件所有权决策 |

## 实现对象清单

| 对象 | 路径 | 实现内容 | 验证入口 |
|---|---|---|---|
| code / schema | `meta_flow/workspace/artifact_migration.py` | frozen value objects、固定 11 分区 `MigrationManifest`、canonical serialization/digest | `test_manifest_has_all_11_immutable_portable_sections`、portability fixture |
| code / read-only preflight | 同上 | 显式单项目 route/worktree identity precheck、portable scope、allowlisted lstat/scandir/readlink/hash/count、target state 与 readiness | identity/scope/permission/symlink/missing/conflict tests |
| code / handoff | 同上 | proposed/manual link plan、worktree/ref readiness、validation 与 rollback checklist；全部 `executed=false` | handoff/schema fixture |
| code / governance | 同上 | O-AW-03 T1/T2/T3 纯函数、insufficient-data 与单 candidate 去重 | threshold boundary/idempotency tests |
| guardrail tests | `tests/test_artifact_migration.py` | 26 个 schema、negative、isolation、portability、snapshot、mutation-spy、per-descendant deny 与 metrics tests | 定向 pytest |
| docs handoff | 本文件 | 实现映射、验证、剩余风险与 QA 入口 | CP6/CP7 消费 |

## 设计契约映射

| 设计契约 | 实现位置 / 动作 | 验证 |
|---|---|---|
| Manifest 11 类分区覆盖率 100% | `REQUIRED_MANIFEST_SECTIONS` + frozen `MigrationManifest` 与 11 个 section value objects | 精确 key 顺序、字段、冻结赋值失败与 serialize/digest test |
| portable mapping 无设备绝对路径 | runtime `Path` 仅存在于 `ExplicitMigrationProject/MigrationRoot` 输入；manifest 只持久化 `PortablePathRef(anchor, relative_path)` | 两个不同临时根、相同相对布局得到相同 mapping/summary/readiness/content digest；payload 不含临时根 |
| 只读 allowlisted scope | route read target必须与 source/target anchor、relative path、runtime root 同时一致；root-level deny 先阻断；每个 descendant 在 source/target lstat/readlink/open/hash/descend/mapping 前再次匹配 deny，任一侧命中即不读取、不映射、不下钻，并留下不含路径内容的 reason | identity mismatch、宽 root 中 `process/quant-lab/**` source deny、target-portable deny；denied lstat/open/hash/mapping/descend=0 |
| symlink no-follow | symlink 用 lstat/readlink；绝对、越界、broken 与 nonportable link 只记录分类，不读取目标内容 | sibling sentinel access=0、secret/path 不进入 manifest、非 READY |
| relative symlink denied target | 读取允许 symlink 自身的 link text 后，同时按 source 与 mapped target portable prefix 归一化；任一归一化目标命中 deny 时，在 target lstat/open 前返回 `relative-denied`，丢弃 link target text并标记 `denied-symlink-target` | target-side-only deny fixture：probe=0、link_target=None、payload 无 target text/content、非 READY |
| regular-file lstat→open 竞态 fail closed | `os.open(O_RDONLY|O_NOFOLLOW[|O_BINARY])` 后 `os.fstat` 确认 regular file，再 `os.fdopen` 读取；缺可靠 `O_NOFOLLOW` 时不读取 | symlink-swap fixture 中 swapped file 成功内容打开数=0、hash=None、MANUAL_REVIEW；无 O_NOFOLLOW fixture 亦非 READY |
| unreadable/incomplete 不得 READY | lstat/enumerate/hash/readlink error 进入 portable `ReadError`，readiness 为 BLOCKED 或 MANUAL_REVIEW | permission、broken link、unsupported no-follow fixture |
| missing target 不得 READY | target root missing 在 source enumeration 前 BLOCKED；单个 mapping target missing 为 MANUAL_REVIEW，reason=`missing-target/manual-migration-required` | root scan=0 与 per-item missing test |
| mutation=0 | 模块无 filesystem/Git/command mutator import；manifest evidence 固定为 mutation/command=0 | Path/os mutator spy、before/after inode/type/content/link snapshot、危险 API/import scan |
| O-AW-03 candidate-only | T1 ≥3 次/周且连续 4 周；T2 median >600s；T3 avoidable rate >5%；任一/多项只构造一个不可执行 candidate | 3 周/4 周、600/600+epsilon、5%/>5%、combined 与 insufficient-data tests |

## 单元测试与 Fixture 计划执行结果

| TEST-PLAN 关注点 | Fixture / 断言 | 结果 |
|---|---|---|
| TP-AW05-001/002 schema、mapping/hash/count | ordinary file、Unicode file、empty dir、relative symlink 与相同 target tree | PASS |
| TP-AW05-003/004 permission、broken/out-of-scope symlink | injected EACCES、broken relative link、absolute sibling sentinel、lstat→open symlink swap | PASS；误报 READY=0，越界内容读取=0 |
| TP-AW05-005/006 isolation、identity | root-level quant-lab deny、宽 allow root 内 denied source subtree、source allowed/target descendant denied、relative symlink→target-side denied、route/project mismatch与 read spy | PASS；denied descendant lstat/readlink-target/open/hash/mapping/descend=0，误报 READY=0 |
| TP-AW05-007/008 readiness、handoff | target root missing、mapping target missing、manual step/authorization/executed=false | PASS |
| TP-AW05-009 deny mutation | common Path/os mutator spy + before/after snapshot + static forbidden call scan | PASS；mutation/command=0 |
| TP-AW05-010 portability | machine-a/machine-b 不同 runtime roots、相同 portable layout | PASS；canonical digest一致 |
| TP-AW05-011..017 metrics | T1/T2/T3 两侧边界、incomplete data、multi-hit | PASS；candidate最多1个，helper/scheduler/remote write字段均为0 |
| TP-AW05-018 idempotency | 相同 immutable input 重建 manifest | PASS；canonical payload/digest稳定 |

## 最小实现切片

1. `TASK-AW-005-01`：冻结 immutable schema、portable ref 与 deny-mutation边界。
2. `TASK-AW-005-02`：实现 route/scope root fail-fast、allowlisted enumeration、hash/count、mapping/readiness 和人工 validation/rollback handoff。
3. `TASK-AW-005-03`：实现 O-AW-03 pure evaluator、strict threshold boundary、insufficient-data 与 deduplicated candidate。
4. `TASK-AW-005-04`：先红后绿实现 23 个 fixture；收到 host 深审后以最小切片关闭 regular-file symlink-swap 风险并重跑全部回归。
5. `CP6-R2 / FIX-AW005-001`：先以三类 probe 复现 3 failed，再只增加 per-object deny guard 和双 portable-prefix symlink target guard；不改变 manifest schema、glob DSL、no-follow port、O-AW-03 或任何 writer。

## 平台差异处理

- POSIX/Linux：普通文件内容读取强制 `O_NOFOLLOW`，并以 `fstat` 再确认 regular file。
- 不提供可靠 `O_NOFOLLOW` 的平台：不使用 pre/post lstat 推测，不读取文件内容，显式进入 unreadable/MANUAL_REVIEW；fixture 已覆盖。
- Windows native permission/symlink 行为未在当前 Linux 环境实机验证；当前 contract 坚持 fail closed，不因平台差异放宽 scope。
- 本 Story 不执行真实 migration、Git、worktree/ref 或 remote pilot，因此这些平台运行差异不属于 CP6 已验证事实。

## 验证结果

| 验证 | 命令 / 结果 |
|---|---|
| TDD 红灯 | 定向 pytest 首次收集因目标模块不存在而 1 error，证明测试先于实现落盘 |
| CP6-R2 红灯 | 三类 denied descendant probe 首次运行 → **3 failed, 23 deselected**；均错误返回 READY，精确复现 `REV-AW005-001` |
| CP6-R2 最小修复探针 | 同三类 probe → **3 passed, 23 deselected** |
| 定向 | `pytest -q tests/test_artifact_migration.py` → **26 passed** |
| routing/worktree 相邻 | migration + `test_cr051_project_artifact_routing.py` + `test_cr051_project_worktree.py` → **95 passed** |
| CR-051 组合回归 | migration、routing/worktree、leg/aggregate、branch/git-sync、state/cr-lifecycle → **280 passed, 9 subtests passed** |
| Ruff | `ruff check` → All checks passed |
| Format | `ruff format --check` → 2 files already formatted |
| Compile | Python 3.11 `py_compile`，cache 隔离到 `/tmp` → PASS |
| Whitespace | scoped `git diff --check` 与 untracked no-index check无 whitespace diagnostics；no-index exit 1仅表示文件有内容差异 |
| Dangerous API/import | mutation/Git/worktree/remote/helper/scheduler executor/import regex 命中=0；唯一 OS 文件打开为 read-only no-follow port |
| Story Return / Evidence | `meta-flow story return-check` 与 `story evidence-check` 均为 OK；return expected-path warning 仅来自外置 `process` 软链接的逻辑路径差异 |

## 未覆盖项与剩余风险

1. `R-AW005-DIR-ENUM-TOCTOU`（non-blocking / CP8 follow-up）：当前目录枚举以静态、非对抗临时 fixture 为验证模型。regular-file `lstat→open` swap 已由 no-follow 关闭，但目录路径若被并发不可信进程 rename/replace，现实现未提供 dirfd/openat 全链路 no-follow 证明。CR-051 不执行真实迁移、无 adversarial runtime authorization；若未来允许对抗式并发目标树，必须升级 full-lld，采用 dirfd/openat 或等价能力并重新经过 scope/security/runtime gate。
2. `CR051-RISK-REAL-MIGRATION-UNVERIFIED`：真实项目文件迁移、软链接挂接、worktree/ref/remote 变更均未授权、未执行；本实现只能生成 manifest/handoff，不能宣称迁移完成。
3. `CR051-RISK-WINDOWS-NATIVE-NOFOLLOW`：Windows native no-follow/symlink/permission 行为未实机验证；没有可靠能力时当前实现会 fail closed。
4. `CR051-RISK-REMOTE-UNVERIFIED`：真实托管 remote、credentials、branch protection 与 publication仍不在授权范围。

## 设计缺口反馈

- 未修改 Feature DESIGN、ADR、HLD 或长期 contract；无需 Design Delta。
- host 深审发现的 regular-file TOCTOU 风险已在原技术说明边界内最小修复并由 fixture关闭。
- CP7 `REV-AW005-001` 已在原 per-object read-scope contract 内最小修复；未改变 approved glob 语义、public manifest schema 或 readiness enum，无需 Design Delta。
- 目录对抗式并发风险不在当前 non-runtime Story 验证模型内，已显式状态化并设置 full-lld重访条件，未静默宣称完成。

## 后续交接

meta-qa 应优先独立复核：

1. 11 分区 schema 与 canonical digest 是否确实不含 runtime absolute path；
2. denied scope、route/runtime root binding 与 sibling sentinel 是否在 root-level 及每个 descendant 的 lstat/readlink/open/hash/descend/mapping 前阻断；重点复核 source deny、target-portable deny 和双 prefix relative symlink target 三类 R2 probe；
3. missing target、EACCES、broken/out-of-scope symlink、无 O_NOFOLLOW 与 symlink-swap 是否始终非 READY；
4. mutation spy + before/after snapshot +静态 import/call审计是否共同证明当前 fixture mutation=0；
5. O-AW-03 严格边界、多阈值单 candidate、`executable/helper/scheduler/remote_write` 全为 false/0；
6. 将 `R-AW005-DIR-ENUM-TOCTOU`、真实迁移未授权和 Windows native no-follow列入 CP8 风险，不把 manifest 误读为执行授权。
