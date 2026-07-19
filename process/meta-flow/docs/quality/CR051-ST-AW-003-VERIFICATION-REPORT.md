---
status: final
version: "1.0"
story_id: "ST-AW-003"
story_slug: "heterogeneous-git-legs"
feature_id: "FEAT-AW-03"
change_id: "CR-051"
validation_mode: "mixed"
verification_result: "NEEDS_REWORK"
source_story: "process/stories/STORY-ST-AW-003-heterogeneous-git-legs.md"
source_implementation: "process/returns/ST-AW-003.CP6.return.json"
context_ref: "process/context/stories/STORY-ST-AW-003.CP7.verify-packet.json"
created_by: "meta-qa-critical (qa-cao)"
created_at: "2026-07-18T14:34:53Z"
updated_at: "2026-07-18T14:34:53Z"
---

# Verification: CR-051 / ST-AW-003 异构 Git legs

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | `NEEDS_REWORK` |
| validation_mode | `mixed`：隔离临时 bare-remote fixture + 静态检查 + 独立语义审查；真实 remote / worktree / ref / link mutation 未授权且未执行 |
| 是否可进入下一阶段 | no |
| 需要路由 | `meta-dev` |
| CP7 证据 | `process/evidence/ST-AW-003.CP7.index.json`；本 lane 不写 CP7 result，不批准 gate |

定向与相邻回归全部通过，但独立默认时钟复现发现 1 个阻断性可靠性缺陷：`execute_leg()` 在调用 fresh observer 前固定 `executed_at`，随后把该较早时间作为 freshness 上界；正常 observer 以 `datetime.now(UTC)` 生成的更新时间戳会满足 `observed_at > executed_at`，被 `_validate_freshness()` 判为 `stale_observation`。结果是未注入固定 `now` 的真实执行在 runner 前 fail-closed，无法完成 source/artifact leg。现有通过用例均注入固定时间，未覆盖默认时钟路径。

## 2. 验证范围

| 项 | 内容 |
|---|---|
| Feature / Story | `FEAT-AW-03` / `ST-AW-003` |
| 验证范围 | source-default / artifact-integration exact target、typed authz、fresh health/OID、immutable payload / external receipt / published handle、evidence-only retry、ordinary resume、coordination-only abort、no-cross-leg-rollback、相邻 CR-050 / git_sync 回归 |
| 非范围 | aggregate projection（ST-AW-004）；真实 shared remote、forge、凭据、branch protection、Windows native Git；真实 remote/worktree/ref/link mutation |
| 上游设计 | `process/docs/features/cr051-legs/TEST-PLAN.md` 与 Story 卡片中的已批准 full-LLD 引用；本轮未扩张读取 LLD/IMPLEMENTATION 全文 |
| 实现摘要 | `process/returns/ST-AW-003.CP6.return.json`、`process/evidence/ST-AW-003.CP6.index.json`、CP6 result |
| 已接受风险 | 无本轮新接受项；`CR051-RISK-REMOTE-UNVERIFIED` 与 `CR051-RISK-WINDOWS-NATIVE-GIT-PILOT` 仍开放 |
| 阻塞条件 | P0/P1 契约失败，或默认运行路径无法完成 fresh observation；本轮触发后一条件 |

读取边界严格限定为 verify packet 的 `allowed_reads`。未读取 `process/STATE.md`、其他 Story、其他 LLD/IMPLEMENTATION、archive 或 discussion；未产生 read expansion。

## 3. 验证对象清单

| 对象 | 类型 | 来源 / 变更原因 | 验证方式 | 是否阻塞 | 证据 |
|---|---|---|---|---|---|
| `meta_flow/workflow/artifact_leg_lifecycle.py` | code / schema / lifecycle | ST-AW-003 primary SUT | 全文审查、Ruff、format、py_compile、危险 argv/import 审计、定向与回归、默认时钟复现 | yes | CMD-01..CMD-10；REV-AW003-001 |
| `tests/test_artifact_leg_lifecycle.py` | guardrail-test / fixture | ST-AW-003 primary test | 全文审查、38-test 定向执行、覆盖映射、固定时钟偏差检查 | yes | CMD-01；`tests/test_artifact_leg_lifecycle.py:580`、`:608`、`:860` |
| `tests/test_git_branch_lifecycle.py` | regression | CR-050 相邻契约 | 只按 packet 命令执行，不扩张全文读取 | yes | CMD-02 |
| `tests/test_workspace_git_sync.py` | regression | git_sync 相邻契约 | 只按 packet 命令执行，不扩张全文读取 | yes | CMD-02 |
| CP6 return / evidence / result | state-process | 验证入口与实现证据 | schema/引用/边界人工核对 | yes | packet allowed reads；CP6 decision=`PASS` |
| CP7 四份质量报告、return、evidence | state-process / docs | 本轮独立 QA 输出 | 模板结构、return/evidence checker | yes | 本报告与 `process/returns/ST-AW-003.CP7.return.json` |

## 4. 验证追踪矩阵

| Scenario / Test | Requirement / Acceptance | Story | Design Contract | Implementation | Test / Check | Status | Risk |
|---|---|---|---|---|---|---|---|
| TP-AW03-001..004 | source/artifact target 2/2；artifact main/default/control/sibling mutation=0 | ST-AW-003 | mode-specific exact target、project integration only | target resolver + safe step | 38 directed；command-spy assertions；静态 ref 审计 | PASS | N/A |
| TP-AW03-007..010/020 | authz/OID/health/drift 任一失败 runner mutation=0 | ST-AW-003 | typed authz + fresh nested WorktreeHealth + exact OID | preparation/execution revalidation | 38 directed；typed mismatch、dirty/digest/drift fixtures | PASS（注入时钟） | 默认时钟 fresh proof 缺陷另见 REV-AW003-001 |
| TP-AW03-012/015/017..019 | immutable payload、external receipt、reread handle tamper 拒绝率 100% | ST-AW-003 | payload 无 append-time 自引用；receipt/handle 外置绑定 | payload seal/publish/readback validator | tamper、writer failure、threaded single-write fixtures | PASS | N/A |
| TP-AW03-006/013 | 单腿事实保留；无跨 leg rollback | ST-AW-003 | leg 独立、失败只保留本 leg facts | partial/recovery payload | post-proof failure + argv denylist | PASS | N/A |
| TP-AW03-015 | evidence-only retry Git=0 | ST-AW-003 | retry 仅调用 evidence writer | `retry_unpublished_payload` / `resume_leg` 分支 | writer failure + evidence retry fixture | PASS | N/A |
| TP-AW03-013 | ordinary resume 使用新 attempt | ST-AW-003 | fresh replan + new correlation | `resume_leg` | ordinary-resume fixture | PASS（注入时钟） | 默认时钟路径会在 execute fresh proof 阻断 |
| abort / recovery | coordination-only abort；Git=0 | ST-AW-003 | abort 不接受 runner | `abort_leg` | abort fixture +静态调用边界 | PASS | N/A |
| 默认时钟 fresh observation | fresh health/OID 必须接受正常当前时间观察 | ST-AW-003 | fresh proof 是执行硬门，但更新鲜快照不得被误判 stale | `execute_leg` + `_validate_freshness` | CMD-10 默认 `now=None` 复现 | FAIL | `REV-AW003-001`（HIGH/BLOCKING） |
| TP-AW03-016 | CR-050 / git_sync legacy 行为不回归 | ST-AW-003 | target override 不改 legacy paired behavior | adjacent modules unchanged | 56-test adjacent regression | PASS | N/A |
| publication boundary | 真实仓库 publication=0 | ST-AW-003 | fixture-only mutation | pytest tmp fixture / spies only | 命令范围审计 | PASS | hosted behavior 未验证 |

## 5. 设计契约验证清单

| 契约 | 来源 | 验证方式 | 是否阻塞 | 结果 | 证据 |
|---|---|---|---|---|---|
| source base/target=`source default`；artifact base/target=`projects/<project>/integration` | Story / TEST-PLAN TP-AW03-001..003 | policy tests + source review | yes | PASS | CMD-01；target resolver |
| artifact main/default/control/sibling mutation=0 | Story acceptance / RULE-AW-05 | command spy + static safe-step/ref audit | yes | PASS | CMD-01、CMD-08/09 |
| typed authz 绑定 action/repo/target/OID/correlation/attempt | TP-AW03-010 | mismatch matrix | yes | PASS | CMD-01 |
| health/OID/drift 不满足时 mutation=0 | TP-AW03-007/009/020 | negative fixtures + runner call count | yes | PASS | CMD-01 |
| fresh observation 正常路径可执行 | Story objective / fresh proof hard gate | 默认时钟动态 observer 复现 | yes | FAIL | CMD-10；`artifact_leg_lifecycle.py:819,844-850,1744-1746` |
| immutable payload 不含 receipt/ref/writer/time 自引用 | TP-AW03-015/017 | schema serialization / digest tests | yes | PASS | CMD-01 |
| external receipt + reread published handle 防错配 | TP-AW03-012/019 | tamper matrix | yes | PASS | CMD-01 |
| evidence-only retry 不重复 Git | TP-AW03-015 | API shape + fixture | yes | PASS | CMD-01 |
| ordinary resume 新 attempt；abort coordination-only | Story acceptance | state transition fixtures | yes | PASS（固定时钟） | CMD-01 |
| 禁止跨 leg rollback | Story / TP-AW03-006 | destructive argv scan + failure fixture | yes | PASS | CMD-01、CMD-08/09 |

## 6. 分层验证计划

| 验证层 | 方法 | 目标 | 触发条件 | 必跑 | 结果 | 未覆盖风险 |
|---|---|---|---|---|---|---|
| 静态检查 | Ruff / format / py_compile / diff check | 语法、格式、基础缺陷 | code | yes | PASS | 无 |
| 单元测试 | 38 directed | 核心 policy/schema/lifecycle | P0 code | yes | PASS | 默认时钟未被既有 suite 覆盖 |
| 契约测试 | target/authz/health/publication/recovery matrix | 上下游契约 | shared contracts | yes | PASS（固定时钟） | REV-AW003-001 |
| 集成测试 | 临时 bare remote source complete | Git argv 与 fresh OID proof | fixture-only | yes | PASS（固定时钟） | 真实 hosted remote 未授权 |
| 回归测试 | 56 directed+adjacent | CR-050 / git_sync compatibility | shared behavior | yes | PASS | Windows 未执行 |
| 安全扫描 | dangerous argv / forbidden import / scoped path | 禁止 shell/destructive/cross-leg behavior | runtime-high-risk | yes | PASS | 无 critical 命中 |
| 人工审查 | findings-first | happy-path 偏差、时钟边界、授权范围 | high-risk full-LLD | yes | FAIL | REV-AW003-001 |
| Workflow eval | N/A | generated workflow 质量 | packet 明确 `workflow_eval_required=false` 且 suite refs 为空 | no | N/A | 本 Story 是代码 lifecycle，不是 aggregate/generated workflow；若 validation target 改变需重访 |
| 平台 dry-run | N/A | installer/platform rendering | 不涉及安装器 | no | N/A | 无 |

## 7. 自动化验证结果

| Command ID | 命令 / 检查 | 结果 | 证据 / 摘要 | 说明 |
|---|---|---|---|---|
| CMD-01 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_artifact_leg_lifecycle.py` | PASS | `38 passed in 0.29s` | 定向 suite |
| CMD-02 | 同环境执行 `pytest -q tests/test_artifact_leg_lifecycle.py tests/test_git_branch_lifecycle.py tests/test_workspace_git_sync.py` | PASS | `56 passed in 4.18s` | 定向 + 相邻回归 |
| CMD-03 | `uv run --python 3.11 ruff check ...` | PASS | `All checks passed!` | 限定两个 primary 文件 |
| CMD-04 | `uv run --python 3.11 ruff format --check ...` | PASS | `2 files already formatted` | 格式检查 |
| CMD-05 | `PYTHONPYCACHEPREFIX=/tmp/cr051-staw003-cp7-pycompile uv run --python 3.11 python -m py_compile ...` | PASS | exit 0 | cache 隔离到 `/tmp` |
| CMD-06 | `git diff --check -- <two primary paths>` | PASS | 无 diagnostics | 两文件当前为 untracked；另用 no-index `--check` 验证其全文 whitespace，exit 1 仅表示与 `/dev/null` 有内容差异且无 diagnostics |
| CMD-07 | scoped `git status --short` | PASS | 两个 primary 路径均为 `??` | 与 CP6 `created` 一致；未读取/评审无关 diff |
| CMD-08 | 生产模块 dangerous API/argv `rg` 审计 | PASS | 唯一匹配为 `_validate_safe_step` 的 destructive token denylist | 无 subprocess/os.system/shell=True |
| CMD-09 | import 与 artifact-main/control/cross-leg 关键词审计 | PASS | 生产模块仅 stdlib + `git_sync` / `project_worktree`；无 unexpected import | 测试中的 subprocess 仅临时 fixture Git |
| CMD-10 | 默认 `now=None`、observer=`datetime.now(UTC)`、in-memory writer/runner-spy 复现 | FAIL | `{'status':'BLOCKED','blocker':'stale_observation','mutation_count':0,'runner_calls':0}` | 不执行 Git；证明真实默认时钟 happy path 被误阻断 |

## 8. Fixture 验证

| Fixture ID | 输入 / 场景 | 期望 | 实际 | 结果 | 证据 |
|---|---|---|---|---|---|
| FX-001 | source-default / artifact-integration target matrix | 精确 target，artifact main/control/sibling=0 | 与预期一致 | PASS | CMD-01 |
| FX-002 | action/repo/target/OID/mode/attempt authz 错配 | runner=0 | runner=0 | PASS | CMD-01 |
| FX-003 | health missing/digest mismatch/dirty、pre-execute drift | runner=0 | runner=0 | PASS | CMD-01 |
| FX-004 | payload/receipt/handle tamper | 100% 拒绝 | 既有 tamper matrix 全部拒绝 | PASS | CMD-01 |
| FX-005 | writer failure + evidence-only retry | 第一次无 handle；retry Git=0 | 与预期一致 | PASS | CMD-01 |
| FX-006 | ordinary resume / coordination abort | 新 attempt / Git=0 abort | 与预期一致（固定时钟） | PASS | CMD-01 |
| FX-007 | normal live-clock observer | fresh snapshot 可继续到 runner | 误报 `stale_observation`，runner=0 | FAIL | CMD-10 |

## 9. 平台与运行边界验证

| 边界 | 检查项 | 预期 | 结果 | 证据 |
|---|---|---|---|---|
| Linux / Python 3.11 | directed / regression / lint / compile | pass | PASS | CMD-01..05 |
| Git fixture | 仅 pytest tmp bare remote | 真实仓库 mutation=0 | PASS | packet 命令范围；测试 fixture |
| Windows native Git | argv/path pilot | 本轮不授权/不执行 | N/A | `CR051-RISK-WINDOWS-NATIVE-GIT-PILOT` |
| hosted shared remote | network/credential/branch protection | 本轮不授权/不执行 | N/A | `CR051-RISK-REMOTE-UNVERIFIED` |
| install / Agent / Prompt | 不属于本 Story | N/A | N/A | 无对应对象 |

## 10. 8 维度与人工 / 语义质量审查

| 维度 / 检查项 | 结果 | 是否阻塞 | 说明 |
|---|---|---|---|
| 完整性 | PASS | yes | 两个 primary 实现/测试对象及六个 CP7 输出对象均有验证路径 |
| 平台适配 | RISK | no | Linux fixture 通过；Windows / hosted remote 未授权 |
| 验收标准覆盖 | FAIL | yes | target/authz/publication/recovery 回链完整，但默认时钟下 fresh observation 正常路径失败 |
| 安全合规 | PASS | yes | 无危险执行入口；destructive Git 词汇只出现在 denylist；真实 publication=0 |
| 命名规范 | PASS | no | 模块、测试与报告命名符合 Story packet |
| Frontmatter / schema 完整性 | PASS | no | 报告 frontmatter 与 return/evidence schema 均可机械校验 |
| 可运行性 | FAIL | yes | 默认 `execute_leg(now=None)` 的 fresh observer 路径被误阻断 |
| 文档覆盖 | N/A | no | 文档阶段/aggregate 接线不在本 Story；四份 QA 报告已记录行为与缺陷 |
| 需求一致性 | FAIL | yes | fail-closed 安全方向正确，但把合法 fresh 观察误判为 stale，无法交付 required leg |
| 场景覆盖 | RISK | yes | 既有 suite 覆盖固定时钟矩阵，遗漏默认时钟时间顺序 |
| Prompt / Agent 边界 | N/A | no | 非 Prompt/Agent 产物 |
| 错误信息可行动 | RISK | no | `stale_observation` 本身稳定，但无法区分真实陈旧与“观察发生在基准时间之后” |
| 是否只覆盖 happy path | FAIL | yes | 大量负向场景已覆盖，但最基本的默认 live-clock happy path 未覆盖 |

## 11. 问题清单

| ID | 等级 | 问题 | 影响 | 建议处理 | Owner | 状态 |
|---|---|---|---|---|---|---|
| REV-AW003-001 | HIGH / BLOCKING | `execute_leg()` 在 observer 之前捕获 freshness 上界，正常 observer 的较新时间戳被判为 future/stale；既有测试通过注入固定 `now` 遮蔽缺陷 | source 与 artifact 非 dry-run 默认路径无法到达 runner；ordinary resume 同样受影响；功能不可用但仍保持 mutation=0 | 调整执行期时钟契约：在每次 observer/health observer 返回后取得校验基准，或显式注入单调 clock 并定义可接受未来偏差；增加 `now=None` 的 source、artifact、post-observe、ordinary-resume 回归 | meta-dev | OPEN |

## 12. 剩余风险

| Risk ID | 风险 | 等级 | 是否接受 | 接受人 / 条件 | 后续处理 |
|---|---|---|---|---|---|
| REV-AW003-001 | 默认时钟执行被误阻断 | HIGH | no | 修复并通过复验后关闭 | 当前 CP7 路由 `meta-dev` |
| CR051-RISK-REMOTE-UNVERIFIED | 仅临时 bare remote，未验证 hosted policy/credential/network | MEDIUM | no（本轮不授权） | 仅在独立 runtime_authorization 后可执行 | 继续作为 CP8 风险候选；不得因此执行真实 publish |
| CR051-RISK-WINDOWS-NATIVE-GIT-PILOT | 未执行 Windows native Git argv/path pilot | LOW | no | Windows 等价环境可用时重访 | 后续跟踪候选 |
| CURRENT-STALE-REF-AUDIT | `CURRENT.json` 仍指向 CP6 packet / 上一 handoff，而 `STATE.current.json` 已指向本 CP7 | INFO | n/a | host-orchestrator 状态收敛时检查 | 不作为 SUT rework；机器状态真相以 STATE.current 为准 |

## 13. 质量评审与修复输入

| 产物 | 路径 | 结论 |
|---|---|---|
| TEST-REPORT | `process/docs/quality/CR051-ST-AW-003-TEST-REPORT.md` | `NEEDS_REWORK` |
| REVIEW | `process/docs/quality/CR051-ST-AW-003-REVIEW.md` | `request-changes` |
| FIXES | `process/docs/quality/CR051-ST-AW-003-FIXES.md` | 1 个 pending blocking fix |

## 14. 阶段决策

| 结论 | 路由 | 条件 / 说明 |
|---|---|---|
| `NEEDS_REWORK` | `meta-dev` | 修复 REV-AW003-001，并复跑默认时钟 source/artifact/resume/post-observe 定向用例、现有 38-test suite、56-test 相邻回归、Ruff/format/py_compile/diff/安全边界检查；在此之前不得把 Story 标记为 verified |

## 15. CP8 输入

| 输入项 | 内容 |
|---|---|
| 风险接受候选 | 当前不进入 CP8；修复后仍需携带 `CR051-RISK-REMOTE-UNVERIFIED`、`CR051-RISK-WINDOWS-NATIVE-GIT-PILOT` |
| 后续 CR 候选 | Windows native Git pilot 可作为 follow-up candidate；不得预建正式 CR |
| 不授权项 | credentials、network、hosted remote、真实 worktree/ref/link mutation、repository publication、aggregate projection、live/data write 均未授权 |
| 发布准备关注点 | CP7 未通过，不得生成 release-ready 或请求 gate |
