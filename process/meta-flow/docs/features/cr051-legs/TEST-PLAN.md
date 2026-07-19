---
feature_id: "FEAT-AW-03"
change_id: "CR-051"
story_ids: ["ST-AW-003"]
validation_mode: "mixed"
status: "ready-for-story-design"
version: "1.2"
---

# FEAT-AW-03 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 建立异构 target、correlation、authz、drift、PARTIAL 与 no-artifact-main-mutation 测试矩阵。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP5 R2：增加唯一 WorktreeHealth.observation 消费契约、payload/receipt/published-handle 无自引用与 evidence-only retry fixtures。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验回修：落实 TP-AW03-004/006/008/014，并增加 containment proof 与 exact expected-OID CAS cleanup 断言。 |

## 1. 范围与方法

采用 ISTQB 正向/负向/边界/状态转换方法，并以风险为基础优先验证 target 错配、陈旧 OID、权限错绑、单腿成功误投影和不可恢复副作用。native Git bare-remote fixture 是主验证层；command spy 用于证明禁止 argv 和 mutation=0；真实远端写不属于 CP6/CP7 默认授权。

## 2. Entry / Exit Criteria

Entry：Feature DESIGN 与 ST-AW-003 full LLD 已通过 CP5；route/worktree contract fixture 可用；所有 Git 操作可指向隔离的临时 bare remote。

Exit：下表 P0/P1 自动化用例 100% 执行，P0 通过率 100%，artifact main/control/sibling mutation=0，陈旧/错 attempt/unpublished handle 接受数=0，平行 WorktreeHealth schema数=0，dry-run mutation=0，legacy source/dedicated mode回归通过。任一安全不变量失败则 `NEEDS_REWORK`。

## 3. 测试矩阵

| Test ID | 层级 | 场景 / 输入 | 预期 | 覆盖 |
|---|---|---|---|---|
| TP-AW03-001 | unit | source-default mode plan | base/target=source remote default exact OID | REQ-AW-008；TC-AW-008 |
| TP-AW03-002 | unit | shared-artifact artifact plan | base/target=project integration exact OID | REQ-AW-009；TC-AW-009 |
| TP-AW03-003 | policy | artifact plan 指向 main/default | BLOCKED，adapter 调用=0 | RULE-AW-05；TC-AW-014 |
| TP-AW03-004 | contract | CR-050 applicability matrix | argv/ref/OID/authz复用；paired-default target被 override | R-AW-05 |
| TP-AW03-005 | integration | 两条 leg 均完成且 writer成功 | 分别输出匹配、fresh、可重读验证的 PASS PublishedLegResultHandle；不写overall | REQ-AW-011/013 |
| TP-AW03-006 | negative | source PASS、artifact FAIL | 两个真实结果保留；无跨 leg rollback/close | REQ-AW-012；TC-AW-008/009 |
| TP-AW03-007 | negative | health非HEALTHY、observation null/digest mismatch，或nested observation显示dirty/staged/untracked/Git-op | artifact leg 100% BLOCKED；Git mutation=0；消费侧不读取平铺字段 | RULE-AW-02；F02 |
| TP-AW03-008 | isolation | sibling checkout dirty | 当前项目可继续；sibling read/write=0 | TC-AW-010/014 |
| TP-AW03-009 | concurrency | precheck 后 target OID 漂移 | step 不执行或结果 FAIL/BLOCKED；不得 PASS | REQ-AW-016 |
| TP-AW03-010 | authz | token 的 action/repo/target/attempt 任一错配 | BLOCKED(authz)，mutation=0 | NF-AW-003/004 |
| TP-AW03-011 | dry-run | 完整 open/publish/finish 计划 | exact argv/evidence 可审计，local/remote mutation=0 | NF-AW-004 |
| TP-AW03-012 | correlation | stale attempt、错CR/project/leg、raw payload/ref、receipt/ref/digest/key错配 | published validator重读payload后100%拒绝 | OBJ-AW-06；F03 |
| TP-AW03-013 | recovery | execute 成功但 post-observe 不确定 | RECOVERY_REQUIRED/BLOCKED；保留现场、不自动 reset | REQ-AW-016 |
| TP-AW03-014 | cleanup | artifact finish proof | 只针对 integration/CR branch fresh proof；main 不读取为 target | CR-050 override |
| TP-AW03-015 | evidence | payload事实PASS但writer append失败 | 只返回unpublished/evidence-pending，无PASS handle；重试byte-identical payload且Git调用=0 | NF-AW-005；F03 |
| TP-AW03-016 | regression | legacy source/dedicated route | 原 paired-default contract 不被 shared mode 改写 | compatibility |
| TP-AW03-017 | schema | serialize `LegResultPayload` 后扫描字段 | result_ref、append/write receipt、writer_id、written_at、receipt_digest字段=0；prewrite digest可复算 | F03 |
| TP-AW03-018 | single-write | 同key同/不同payload digest并发append | 同digest幂等返回同receipt；不同digest冲突；payload写入次数=1 | F03 |
| TP-AW03-019 | receipt | 构造/篡改external receipt与published handle | receipt digest绑定derived key/ref/payload digest/writer/time；任何篡改拒绝 | F03 |
| TP-AW03-020 | port contract | 传入Lane B frozen WorktreeHealth rich fixture | 仅通过health.observation读取identity/common-dir/HEAD/OID/dirty/Git-op/role；HEALTHY+non-null+digest match放行 | F02 |

## 4. Fixture 与观测

- 两个隔离 bare remotes、source/artifact working repositories、project integration worktree、额外 dirty sibling checkout。
- command spy 记录 argv、cwd、环境、调用次数；明确断言不存在 `reset --hard`、`clean`、`stash`、force、rebase、artifact main update/delete。
- ref drift fixture 在 precheck 与 execute 之间推进 OID；authz fixture 逐字段错绑。
- result fixture 覆盖raw/unpublished、缺失、重复、陈旧、payload/receipt/key/ref digest错误和evidence append failure；WorktreeHealth fixture直接复用Lane B frozen shape。

## 5. 追踪与证据

| 设计不变量 | 证据 |
|---|---|
| source-default / artifact-integration exact target | plan snapshot + fresh ref observation |
| artifact main/control/sibling mutation=0 | command-spy denylist + before/after OID |
| PARTIAL 非 terminal success | typed result assertion |
| 无跨 leg 自动回滚 | step receipt 与 command count |
| 每 leg payload 单写且 published handle可重读验证 | prewrite digest + external receipt + reader/validator |
| WorktreeHealth唯一端口 | nested observation字段访问spy + digest match/mismatch matrix |

## 6. 不执行项与剩余风险

CP4 只生成计划，不执行测试；CP6 默认只允许临时 bare remote，不允许真实 remote/default mutation。真实托管平台 branch protection、网络中断和凭据授权需要显式 runtime authorization 后另行验证，并作为剩余风险记录，不降低本地 contract test 的硬门。

## 7. Gotchas

- 只断言命令退出码不足以证明成功，必须断言 fresh observed OID。
- sibling dirty fixture 必须与 current worktree dirty fixture分开，否则会掩盖 isolation 缺陷。
- artifact integration 名称相同不代表 repo identity 相同；测试要同时错配 ref 与 common Git dir。

## 8. CP8 终验回修执行证据

| Test ID | 自动化证据 | 关键断言 | 状态 |
|---|---|---|---|
| TP-AW03-004 | `test_cr050_paired_projection_compatibility` | CR-050 paired/default 投影保持既有契约 | PASS |
| TP-AW03-006 | `test_source_pass_is_preserved_when_artifact_aborts_without_cross_leg_rollback` | source PASS 与 artifact abort 独立保留；跨 leg rollback=0 | PASS |
| TP-AW03-008 | `test_dirty_sibling_is_not_read_or_touched_by_current_project_leg` | sibling dirty 不误阻断；sibling touched path/argv=0 | PASS |
| TP-AW03-014 | `test_artifact_finish_requires_containment_and_uses_exact_cas_cleanup` | integration containment、完整 ref/OID lease、main target/read=0 | PASS |

定向 ST-AW-003/legacy/ST-AW-004 集合 127 项通过；包含 worktree/state/CR lifecycle 的跨模块集合 307 项与 58 个 subtests 通过。真实 remote mutation=0。
