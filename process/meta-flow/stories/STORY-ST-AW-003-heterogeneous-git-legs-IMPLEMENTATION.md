---
status: ready-for-verification
version: "1.2"
story_id: "ST-AW-003"
story_slug: "heterogeneous-git-legs"
feature_id: "FEAT-AW-03"
implementation_type: "mixed"
source_story: "process/stories/STORY-ST-AW-003-heterogeneous-git-legs.md"
source_design_evidence: "process/stories/STORY-ST-AW-003-heterogeneous-git-legs-LLD.md"
created_by: "meta-dev"
created_at: "2026-07-18T14:00:18+00:00"
updated_at: "2026-07-19T00:00:00+00:00"
---

# Implementation: ST-AW-003 — 异构 Git legs

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev | 落成 source-default / artifact-integration schema、policy、typed authz、fresh health、单 leg executor、immutable payload、external receipt、published handle、resume/abort 与定向 fixture。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP6-R2 仅关闭 `REV-AW003-001`：补齐四条 `now=None` 正向和 stale/future-skew 负向测试，在观察端口返回后重新采样默认 validation clock，并同步模块说明；详见 §15。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验回修：抽取 canonical policy，增加 containment + exact-OID CAS cleanup，补齐 TP-AW03-004/006/008/014，并合并 design delta。 |

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 在独立模块中实现 source-default 与 artifact-integration 两条 leg 的规划、单腿执行、证据发布和非破坏性恢复契约。 |
| 行为变化 | 调用方可生成 mode-specific exact-ref plan；artifact 只消费 nested `WorktreeHealth.observation`；非 dry-run 必须通过 typed authz 与 fresh proof；结果按 immutable payload → external receipt → reread-validated handle 发布。 |
| 范围边界 | 不修改 `git_sync.py`、`git_branch_lifecycle.py`、CLI 或 aggregate；不计算 overall；不接触 artifact main/control/sibling；不执行真实远端或生产仓 mutation。 |
| CP6 证据 | 初始实现见 `process/returns/ST-AW-003.CP6.return.json` 与 `process/evidence/ST-AW-003.CP6.index.json`；R2 回修见 `process/returns/ST-AW-003.CP6-R2.return.json` 与 `process/evidence/ST-AW-003.CP6-R2.index.json`。CP6 gate 结论仍由 Host Orchestrator 生成，本实现不批准 CP6。 |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| Story | `process/stories/STORY-ST-AW-003-heterogeneous-git-legs.md` | 范围、量化 AC、primary/shared/forbidden 文件边界。 |
| Story 设计证据 | `process/stories/STORY-ST-AW-003-heterogeneous-git-legs-LLD.md` v1.1 | schema、target matrix、authz、fresh proof、publication DAG、resume/abort、TASK-ID。 |
| Feature DESIGN | `process/docs/features/cr051-legs/DESIGN.md` v1.1 | `WorktreeHealth.observation` 唯一端口、artifact integration hard gate、external receipt。 |
| Feature TEST-PLAN | `process/docs/features/cr051-legs/TEST-PLAN.md` v1.1 | TP-AW03-001..020、临时 bare remote、command spy、tamper/single-write fixture。 |
| Feature TASKS | `process/docs/features/cr051-legs/TASKS.md` | TASK-AW-003-01..05、开发门、文件 owner、回退条件。 |
| 上游 verified return | `process/returns/ST-AW-002.CP7.return.json` | `WorktreeHealth.observation` contract verified-with-risk；真实 shared remote 不授权。 |

完整 LLD 在实现前已按 `reason=deep_review` 记录到 `process/state/READ-EXPANSION-LEDGER.ndjson`，然后才开始业务文件修改。

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| process route health | PASS | `process/current/CURRENT.json.health=ok`，routing ref 可用；未重建或修复 process。 |
| 上游 Feature 设计存在 | PASS | legs DESIGN / TEST-PLAN / TASKS 已读取。 |
| Story 范围明确 | PASS | Story 为 `dev-ready`，`lld_gate.approved=true`。 |
| 全量 CP5 / 设计证据 | PASS | `design_evidence_confirmed=true`；approval ref 指向 CP5 batch。 |
| 运行依赖 | PASS_WITH_RISK | ST-AW-002 CP7 为 `verified_with_risk`，允许消费 frozen health port；真实 remote 风险保留。 |
| 待确认问题 | PASS | LLD `open_items=0`，无 blocking clarification。 |
| 验证方式 | PASS | Python 3.11、pytest、ruff、py_compile、临时 bare remote 与 command spy。 |
| 文件所有权 | PASS | 只写本 Story 两个 primary 与批准的 IMPLEMENTATION / return / evidence 路径。 |
| shared helper 适用性 | PASS | `git_sync.run_git`、`GitCommandResult` 足以作为 argv-only port；未发现必须修改 shared 文件的 design delta。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 必须 | 验证方式 |
|---|---|---|---|---|
| `meta_flow/workflow/artifact_leg_lifecycle.py` | code / template-schema | schema、policy、executor、publication、validator、resume/abort | yes | unit / contract / command spy / bare remote / ruff / py_compile |
| `tests/test_artifact_leg_lifecycle.py` | guardrail-test | target、authz、health、drift、single-write、tamper、recovery 与 no-cross rollback | yes | pytest |
| 本文件 | docs-handoff | 实现契约、切片、验证、剩余风险与 QA 入口 | yes | structure review |
| `process/returns/ST-AW-003.CP6.return.json` | evidence schema | touched files、boundary、commands、risk、next route | yes | `meta-flow story return-check` |
| `process/evidence/ST-AW-003.CP6.index.json` | evidence index | CP6 机器消费入口 | yes | `meta-flow story evidence-index` + index check |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| source base/target=source default；artifact base/target=project integration | LLD F-01..03 | canonical ref helpers、`_resolve_target` | policy 重算，request 仅作断言；artifact main/default/control fail closed | target matrix tests |
| typed authz 绑定 action/repo/ref/OID/correlation/attempt/mode/expiry | LLD F-04 / NF-04 | `LegAuthorization`、`_validate_authorization` | 非 dry-run mutation 前逐字段精确比对 | authz mismatch matrix |
| expected OID 与 fresh observation | LLD F-05 / NF-05 | `LegObservation`、plan/execute precheck/post-proof | digest、age、identity、refs/OIDs、dirty/Git-op 校验 | drift / post-proof tests |
| artifact 只消费 nested observation | LLD F-11 | `_validate_worktree_health` | 重建 Lane B digest；校验 identity、route、dirty、role、journal；未定义平铺 health schema | health matrix / artifact executor tests |
| dry-run mutation=0 | LLD F-12 | `execute_leg` dry-run branch | 输出 exact plan 与 IN_PROGRESS payload，不调用 runner/writer | dry-run spy |
| payload 无 append-time/self-ref 字段 | LLD F-06..08 | `LegResultPayload`、`payload_to_dict`、`seal_leg_result_payload` | digest 排除自身字段；拒绝 receipt/result_ref/writer/time 嵌入 | schema/roundtrip/forbidden field tests |
| external receipt 与 reread handle | LLD §5.3 / §6 | `LegResultWriteReceipt`、`PublishedLegResultHandle`、validator | receipt digest 外置；按 result_ref 重读并校验 key/ref/digest/correlation/mode | tamper matrix |
| same-key single-write | LLD F-06 | writer Protocol、fixture store | 同 key+同 digest 幂等；不同 digest `result_conflict` | sequential + threaded tests |
| writer失败不产生 handle；evidence retry不重跑Git | LLD NF-06 | unpublished outcome、retry API | 保存 byte-identical payload，retry API 不接收 runner | writer failure / retry test |
| resume fresh replan；abort coordination-only | LLD F-09 / §8.4 | `resume_leg`、`abort_leg` | 普通 resume 新 attempt + fresh ports；abort 只追加 result | resume/abort tests |
| no cross-leg rollback | LLD F-09 | 单 target plan/executor + denylist | 一次只接受一个 `LegPlan`；无 reset/clean/stash/rebase/force/aggregate 调用 | partial/no-cross command-spy tests |

## 6. 单元测试与 Fixture 计划

| 测试对象 | 类型 | 输入 / Fixture | 期望 | 覆盖风险 | 状态 |
|---|---|---|---|---|---|
| target policy | unit | source/artifact routes、main assertion、all artifact operations | required target 2/2；main/default 0 | target drift | passed |
| WorktreeHealth port | contract | healthy/null/digest mismatch/dirty/role/journal/identity | 仅 HEALTHY+non-null+digest match 放行 | duplicated/flat health | passed |
| authz / freshness | negative / boundary | per-field mismatch、missing auth、OID drift、unsafe refs | runner 调用 0 | stale/unauthorized mutation | passed |
| executor | command spy | dry-run、source complete、artifact complete、post-proof mismatch | exact single-leg argv；PASS only after proof；partial preserved | wrong mutation / false PASS | passed |
| publication | schema / contract | payload roundtrip、receipt tamper、key/ref/digest/correlation/mode tamper | invalid acceptance 0 | evidence corruption | passed |
| single write | concurrency | same/different payload digest、8 parallel same payload | append exactly once；conflict fail closed | duplicate result | passed |
| recovery | state transition | writer failure、evidence retry、ordinary resume、abort | retry Git=0；resume new attempt；abort Git=0 | duplicate effect / destructive recovery | passed |
| native Git | temporary bare remote | isolated local repo + bare origin | source complete updates only exact source target | adapter/runtime mismatch | passed |
| legacy regression | existing suite | CR-050 lifecycle + workspace git sync tests | no regression | shared helper drift | passed |

所有 Git/remote 测试均限定在 pytest 临时目录；没有读取凭据、访问真实 remote 或修改当前仓库 refs/worktrees。

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 局部验证 | 状态 |
|---|---|---|---|---|
| IMPL-AW003-S1 | schema / mode / target / authz / health | lifecycle schema/policy + 核心红灯测试 | collection red → 16 passed | done |
| IMPL-AW003-S2 | operation plan / exact argv / dry-run | plan steps + executor skeleton | command-spy tests | done |
| IMPL-AW003-S3 | immutable payload / receipt / handle | publication schema、digest、store/validator | tamper + concurrency tests | done |
| IMPL-AW003-S4 | fresh execute / partial / no-cross rollback | executor post-proof/effect | source/artifact/partial tests | done |
| IMPL-AW003-S5 | evidence retry / resume / abort | recovery API | evidence-only / new attempt / coordination-only tests | done |
| IMPL-AW003-S6 | temporary Git + legacy regression | bare remote fixture + existing CR-050 suites | 56 passed | done |

## 8. 变更说明

### 8.1 代码变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `meta_flow/workflow/artifact_leg_lifecycle.py` | create | 新增独立异构 leg domain/policy/executor/publication/recovery 模块；shared Git 模块只读调用 `run_git` / typed result。 |

### 8.2 Prompt / Skill 变更

N/A：本 Story 不修改 Prompt、Agent 或 Skill。

### 8.3 模板 / Schema 变更

| Schema | 说明 |
|---|---|
| `LegCorrelation` / `LegRequest` / `LegRouteProof` / `LegTarget` / `LegObservation` / `LegAuthorization` / `LegPlan` | 规划和权限/freshness 输入。 |
| `LegResultPayload` | immutable Git fact；不含 overall、result_ref、receipt、writer/time。 |
| `LegResultWriteReceipt` | external append receipt；digest 不含自身。 |
| `PublishedLegResultHandle` | result_ref + payload digest + external receipt + correlation/mode。 |
| `UnpublishedLegResultOutcome` | writer failure 后的 evidence-pending 恢复入口。 |

### 8.4 Guardrail / 测试变更

| 文件 / 命令 | 动作 | 说明 |
|---|---|---|
| `tests/test_artifact_leg_lifecycle.py` | create / R2 update | 初始 38 个定向场景；R2 新增 4 条默认时钟正向与 stale/future-skew 2 条负向场景，合计 44 个场景。 |
| CR-050 / git_sync suites | run only | 18 个既有回归场景；与本 Story 44 个场景合计 62；shared 源文件未由本 Story 修改。 |

### 8.5 文档变更

| 文件 | 动作 | 说明 |
|---|---|---|
| 本文件 | create | 完整实现证据与 QA/Review 交接。 |
| CP6 return / evidence index | create | 机器可消费交接；不替代 Host CP6 result。 |

## 9. 平台差异处理

| 平台 | 检查项 | 预期 | 结果 |
|---|---|---|---|
| Native Git | argv-only，无 shell | 所有 mutation 仅 exact `git push <remote> <oid>:<full-ref>` 或 fresh-proved delete refspec | PASS |
| Linux / macOS | Python 3.11 + pathlib | path 只作为 injected repo root，不序列化凭据 | PASS |
| Windows | subprocess argv / path | 代码不使用 shell；本轮未执行 Windows native Git pilot | PASS_WITH_RISK |
| Claude / Codex / install | Agent/Skill/platform installer | 不适用 | N/A |
| Forge API / SaaS | 真实凭据与 branch protection | 未授权，不执行 | N/A / remaining risk |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| 首轮红灯：`pytest -q tests/test_artifact_leg_lifecycle.py` | EXPECTED FAIL | collection error：目标模块尚不存在；证明核心测试先落盘。 |
| 第一切片局部：同上 | PASS | `16 passed`。 |
| 第二切片定向：同上 | PASS | `38 passed`，含临时 bare remote。 |
| `pytest -q tests/test_artifact_leg_lifecycle.py tests/test_git_branch_lifecycle.py tests/test_workspace_git_sync.py` | PASS | 最终复跑 `56 passed in 4.24s`。 |
| `ruff check meta_flow/workflow/artifact_leg_lifecycle.py tests/test_artifact_leg_lifecycle.py` | PASS | all checks passed。 |
| `ruff format` / format check | PASS | 两文件 canonical formatted。 |
| `python -m py_compile ...` | PASS | Python 3.11 编译通过，pycache 定向到 `/tmp`。 |
| forbidden import / ownership audit | PASS | lifecycle 不 import CLI/aggregate/CR-050 executor；只从 `git_sync` 读取 typed runner/result，从 `project_worktree` 读取 frozen health port。 |
| dangerous argv audit | PASS | 可执行 plan 只构造 `git push` exact refspec；禁止 reset/clean/stash/rebase/force/cross-leg token。 |
| `git diff --check` / whitespace | PASS | 目标文件无格式错误；untracked primary 由 ruff format/check 覆盖。 |
| `meta-flow story return-check` | PASS | Return Packet Check OK；外置 `process` symlink 的 resolved-path 比较仅产生非阻断 warning。 |
| `meta-flow story evidence-check` | PASS | Evidence Index Check OK；生成器输出的设备绝对 `return_ref` 已按路由规则归一化为 `process/returns/ST-AW-003.CP6.return.json`。 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| 真实 shared remote / forge branch protection / 网络中断 | `NO_REPOSITORY_PUBLICATION` 与 `FIXTURE_ONLY_GIT_REMOTE`；本 Story无 runtime authorization | 保留 `CR051-RISK-REMOTE-UNVERIFIED`，另行授权 pilot。 |
| Windows native Git / filesystem durability | 当前验证环境为 Linux | CP7/CP8 决策是否增加 Windows pilot；不豁免 contract tests。 |
| CLI 与 aggregate 端到端 | 分别由 ST-AW-004 merge owner 与 aggregate primary owner负责 | ST-AW-004 只消费 schema + reader/validator，不 import executor。 |

## 12. 风险与回滚

| Risk ID | 风险 | 影响 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|---|
| CR051-RISK-REMOTE-UNVERIFIED | 本地 bare remote 不覆盖托管平台 policy | 真实 publish/complete 可能被远端拒绝 | 保持 fail closed、普通 non-force push、fresh post-proof；不宣称真实 remote 已验证 | 远端拒绝时保留 FAIL/PARTIAL，不 reset/force；需独立 runtime authorization。 |
| CR051-RISK-LEGACY-LAYOUT-COMPAT | route proof 错配 | 错 target | policy 重算、owned target/digest/fingerprint 精确校验 | 任一 artifact main/default plan 立即禁用 shared-artifact mode。 |
| CR051-RISK-EVIDENCE-WRITER | Git effect 后 writer失败 | aggregate 无可消费结果 | unpublished outcome + byte-identical evidence-only retry | 不重跑 Git；writer恢复后仅 append。 |

模块未接 CLI，回滚可移除 ST-AW-004 后续接线并禁用 shared-artifact mode；不会自动重写或删除已发生的 remote fact。

## 13. 设计缺口反馈

| Gap ID | 发现阶段 | 问题 | 应反馈到 | 是否阻塞 | 推荐处理 |
|---|---|---|---|---|---|
| N/A | implementation | 现有 `git_sync.run_git` / typed result 足够支撑本 Story，无需修改 shared policy 或新增长期设计。 | N/A | no | 不生成 design delta。 |

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 对 source/artifact 各 operation 复核 exact base/target/active ref；artifact main/default/control/sibling mutation 必须为 0。
- 独立验证 authz 逐字段错绑、pre-execute drift、post-proof mismatch、dry-run 与 unsafe ref 输入的 runner 调用数为 0。
- 对 raw/unpublished、stale attempt、wrong mode/correlation、result_ref/payload/receipt/key digest tamper 做拒绝矩阵。
- 验证 evidence-only retry 不调用 Git；普通 resume 使用新 attempt 与 fresh plan；abort 保留 effect 且 Git=0。

### Review 关注点

- `LegResultPayload` 不得新增 result_ref、receipt、writer/time、overall/projection 字段。
- Aggregate 只 import schema/reader/validator，不 import executor；CLI ownership 仍归 ST-AW-004。
- `git_sync.py`、`git_branch_lifecycle.py` 当前工作树可能含其他 Story 并行改动，本 Story 的 touched files / boundary 必须以 return packet 为准。

### Doc 关注点

- 用户文档只说明 fixture-validated contract；不得表述为真实 remote 已启用或已授权。
- 发布说明应保留 `real-remote-unverified` 与 Windows pilot 风险。

## 15. CP6-R2：`REV-AW003-001` 回修

### 15.1 回修输入与边界

| 项目 | 结论 |
|---|---|
| Finding | `REV-AW003-001`：`execute_leg(now=None)` 在 observer 之前固定 `executed_at`，端口正常生成的更新鲜快照会被误判为 future skew。 |
| Gap | `GAP-AW003-001`：初始测试全部显式注入 `now`，未覆盖 production 默认时钟路径。 |
| 允许变更 | lifecycle 主模块、其定向测试、本 IMPLEMENTATION、CP6-R2 return/evidence。 |
| 明确不变 | frozen target/authz/health digest/publication/recovery/schema 契约；shared、CLI、aggregate 与其他 Story 文件。 |
| 运行授权 | 仅 command spy 与 pytest 临时 Git fixture；真实仓库、凭据、remote/worktree/ref/link/publication 均为 0。 |
| 设计判断 | freshness 语义未改变，只修复采样顺序；无需 LLD/HLD/ADR 回写，design delta=0。 |

### 15.2 TDD 红灯与最小修复

先只新增以下默认时钟场景，不修改实现：

1. source `complete`：pre/post observer 均在调用期间生成快照；
2. artifact `complete`：fresh leg observer + 已绑定 fresh health；
3. ordinary `resume`：新 attempt 经 fresh replan 后执行；
4. post-observe：precheck 已通过，runner 后生成 post-proof；
5. genuinely stale observation：必须 `BLOCKED` 且 runner=0；
6. genuine future-clock-skew：必须 `BLOCKED` 且 runner=0。

红灯命令 `pytest -q tests/test_artifact_leg_lifecycle.py -k default_clock` 得到 `4 failed, 2 passed`：四条合法正向分别在 pre/post freshness 被旧入口时钟误拒，两个真正 stale/future-skew 负向仍通过。

最小实现只调整 `execute_leg` 的 validation clock：

- 保留 `executed_at = explicit now or initial clock`，不改变 authorization 与 receipt schema；
- `now is None` 时，在 `observer`、`health_observer`、post-`observer` 各自返回后重新采样 `datetime.now(UTC)`，仅将该值用于对应 freshness 校验；
- 显式 `now` 时仍复用固定 `executed_at`，保持既有确定性测试和调用契约；
- `_validate_freshness` 本身未放宽，`observed_at > validation_now` 的真正 future skew 继续 fail closed；
- 同步模块与 `build_leg_plan` docstring，删除“无执行步骤/不会执行 Git”的过期描述。

修复后同一命令得到 `6 passed, 38 deselected`。四条正向均到达 runner/post-proof 并完成；两个负向均保持 `BLOCKED`、`mutation_count=0`、runner 调用 0。

### 15.3 R2 验证结果

| 检查 | 结果 | 证据摘要 |
|---|---|---|
| 默认时钟红灯 | EXPECTED FAIL | `4 failed, 2 passed`；失败仅为四条合法 `now=None` 正向。 |
| 默认时钟绿灯 | PASS | `6 passed, 38 deselected`。 |
| 全量定向 | PASS | `44 passed in 0.28s`。 |
| 相邻回归 | PASS | lifecycle + branch lifecycle + workspace git sync：`62 passed in 4.32s`。 |
| Ruff | PASS | `ruff check` all checks passed；`ruff format --check` 两文件已格式化。 |
| Python 3.11 compile | PASS | `py_compile` 通过，pycache 定向到 `/tmp/cr051-staw003-cp6-r2-pycompile`。 |
| forbidden import / API | PASS | AST 审计确认 lifecycle 不 import CLI/aggregate/branch lifecycle/subprocess，且不存在 `shell=True`。 |
| dangerous argv / no-cross-leg | PASS | 44 个定向场景继续覆盖 exact push、unsafe ref、dry-run、partial/abort 与 reset/clean/stash/rebase/force denylist。 |
| diff / whitespace | PASS | scoped `git diff --check` 与两文件 trailing-whitespace 审计通过。 |
| CP6-R2 return check | PASS | `Story Return Packet Check: OK`；外置 process symlink 的 resolved-path 比较仅产生非阻断 warning。 |
| CP6-R2 evidence check | PASS | `Evidence Index Check: OK`；`return_ref` 使用 project-root 相对路径。 |
| production mutation | PASS | 真实 repository publication=0；仅 existing pytest 临时 bare remote fixture。 |

### 15.4 回修结论与交接

- `REV-AW003-001`：实现侧 `RESOLVED`，等待独立 CP7 复核；没有顺带关闭其他 finding。
- frozen contract、public schema、CLI/aggregate/shared owner 边界均未变化。
- 既有风险 `CR051-RISK-REMOTE-UNVERIFIED`、Windows native pilot 风险保持不变；未新增风险接受项。
- CP6-R2 只生成 Story return/evidence 并交还 Host Orchestrator；不写 CP6 result、不批准任何 gate。

## 16. CP8 终验回修实施（R3）

| 回修项 | 实现结果 | 安全边界 |
|---|---|---|
| canonical target policy | 新增 `artifact_policy.py`；leg producer 与 aggregate consumer 共用纯规则 | artifact main/default/错误 mode/ref 仍 hard deny |
| finish containment | fresh `target_oid == active_oid` 才允许 cleanup | containment 不可证明时 mutation=0 |
| scoped CAS delete | 仅允许 `--force-with-lease=<active-ref>:<active-oid>` + empty-source deletion | ordinary force、错 ref/OID/step、wildcard/main 均拒绝 |
| TP-AW03-004 | CR-050 paired/default regression | legacy contract 保持 |
| TP-AW03-006 | source PASS + artifact abort | 两 leg 事实独立；cross-leg rollback=0 |
| TP-AW03-008 | dirty sibling | sibling path 不读取/不触碰，不误阻断 current |
| TP-AW03-014 | artifact finish cleanup proof | integration 包含 tip；main target/read=0；exact lease |

正式 design delta `process/design-deltas/ST-AW-003.delta.json` 已合并到 Feature DESIGN 1.2 与 LLD 1.2。ST-AW-003/legacy/ST-AW-004 定向集合 127 项通过，跨模块集合 307 项与 58 个 subtests 通过，全仓 697 项与 70 个 subtests 通过。真实 remote/ref/worktree 操作仍未授权且未执行。
