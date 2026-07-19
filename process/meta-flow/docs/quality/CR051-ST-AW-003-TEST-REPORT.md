---
status: final
version: "1.0"
scope: "CR-051 / ST-AW-003"
created_at: "2026-07-18T14:34:53Z"
validation_mode: "mixed"
verification_result: "NEEDS_REWORK"
created_by: "meta-qa-critical (qa-cao)"
---

# Test Report: CR-051 / ST-AW-003

## 验证范围

| 项 | 内容 |
|---|---|
| Feature / Story | `FEAT-AW-03` / `ST-AW-003` |
| 验证范围 | exact target、typed authz、fresh health/OID、immutable publication、evidence retry、resume/abort、no-cross-leg-rollback、相邻回归、默认时钟正常路径 |
| 非范围 | ST-AW-004 aggregate、真实 hosted remote、凭据、Windows native Git、真实 repository publication/worktree/ref/link mutation |
| 上游设计 | `process/docs/features/cr051-legs/TEST-PLAN.md` |
| 实现证据 | CP6 return/evidence/result；SUT 与定向测试全文 |
| validation_mode | `mixed`（fixture runtime + static + manual） |

## 验证对象清单

| 对象 | 类型 | 验证方式 | 是否阻塞 | 证据 |
|---|---|---|---|---|
| `artifact_leg_lifecycle.py` | code / schema / lifecycle | unit/contract/integration/static/manual | yes | 38/56 tests、Ruff、compile、REV-AW003-001 |
| `test_artifact_leg_lifecycle.py` | fixture / test | coverage review + execution | yes | 38 passed |
| CR-050 / git_sync adjacent tests | regression | packet-specified execution | yes | 56 passed |
| CP6 return/evidence/result | state-process | entry/boundary audit | yes | CP6 PASS |

## 验证追踪矩阵

| Scenario | Requirement | Story | Design Contract | Implementation | Test / Check | Status | Risk |
|---|---|---|---|---|---|---|---|
| TP-AW03-001..004 | REQ-AW-008/009；RULE-AW-05 | ST-AW-003 | exact mode target | route/plan policy | directed suite + static argv | PASS | N/A |
| TP-AW03-007..010/020 | REQ-AW-016；NF-AW-003/004 | ST-AW-003 | typed authz + fresh health/OID | preparation/execution guards | negative matrix | PASS | 默认时钟正常路径 FAIL |
| TP-AW03-012/015/017..019 | REQ-AW-011/013；NF-AW-005 | ST-AW-003 | immutable external publication | payload/receipt/handle | tamper/single-write/retry | PASS | N/A |
| TP-AW03-006/013 | recovery invariants | ST-AW-003 | preserve facts, no rollback | partial/abort/resume | state tests + denylist | PASS | resume 默认时钟受 REV-AW003-001 影响 |
| live-clock happy path | required leg executable | ST-AW-003 | fresh observation accepted | execute freshness ordering | in-memory default clock reproduction | FAIL | HIGH/BLOCKING |
| TP-AW03-016 | compatibility | ST-AW-003 | legacy behavior unchanged | adjacent modules | 56-test regression | PASS | N/A |

## 设计契约验证

| 契约 | 来源 | 验证方式 | 是否阻塞 | 结果 | 证据 |
|---|---|---|---|---|---|
| source/artifact target matrix 100% | Story acceptance | test + review | yes | PASS | 38 tests |
| artifact main/control/sibling mutation=0 | Story acceptance | spy + static scan | yes | PASS | 38 tests / dangerous scan |
| authz/OID/health/drift fail closed | Story acceptance | negative test | yes | PASS | 38 tests |
| immutable payload/receipt/handle tamper rejection | Story acceptance | tamper matrix | yes | PASS | 38 tests |
| evidence-only retry Git=0 | Story acceptance | writer failure fixture | yes | PASS | 38 tests |
| ordinary resume new attempt；abort/cross rollback Git=0 | Story acceptance | transition fixtures | yes | PASS（固定时钟） | 38 tests |
| normal fresh observer must not be rejected solely because it ran after function entry | fresh-proof runtime contract | default clock reproduction | yes | FAIL | REV-AW003-001 |

## 分层验证计划

| 验证层 | 方法 | 目标 | 必跑 | 结果 | 未覆盖风险 |
|---|---|---|---|---|---|
| 静态检查 | Ruff/format/compile/diff | code hygiene | yes | PASS | 无 |
| 单元/契约 | 38 directed | policy/schema/recovery | yes | PASS | 固定 clock 偏差 |
| 集成 | tmp bare remote | native Git target/OID | yes | PASS | hosted remote 未授权 |
| 回归 | 56 combined | adjacent behavior | yes | PASS | Windows 未执行 |
| 安全 | argv/import/scope audit | no destructive/cross-leg action | yes | PASS | 无 critical finding |
| 人工/语义 | default clock/happy path | runtime usability | yes | FAIL | REV-AW003-001 |
| Workflow eval | N/A | generated workflow | no | N/A | packet 明确不要求 |

## 测试环境

| 字段 | 值 |
|---|---|
| Runtime | Linux；Python 3.11 via `uv`；pytest；Git temp fixtures |
| Commit / Diff | 两个 primary 路径为 untracked created files，与 CP6 return 一致；审查限定到 packet paths |
| Validation Env | 等价 fixture 验证；无凭据、网络、生产/真实 repository mutation |

## 测试命令

| Command ID | 命令 | 结果 | 证据 |
|---|---|---|---|
| T-CMD-01 | directed pytest | PASS | 38 passed / 0 failed |
| T-CMD-02 | directed + adjacent regression pytest | PASS | 56 passed / 0 failed |
| T-CMD-03 | Ruff check + format check | PASS | All checks passed；2 files formatted |
| T-CMD-04 | py_compile | PASS | exit 0 |
| T-CMD-05 | scoped diff/no-index whitespace | PASS | 无 `--check` diagnostics |
| T-CMD-06 | dangerous argv / forbidden import audit | PASS | destructive token 仅 denylist；无 subprocess/shell in SUT |
| T-CMD-07 | default `now=None` dynamic observer reproduction | FAIL | BLOCKED/stale_observation；mutation_count=0；runner_calls=0 |

## Fixture 验证

| Fixture ID | 输入 / 场景 | 期望 | 结果 | 证据 |
|---|---|---|---|---|
| FX-AW03-TARGET | source/artifact exact target | 2/2 correct | PASS | directed suite |
| FX-AW03-AUTHZ | typed mismatch matrix | mutation=0 | PASS | directed suite |
| FX-AW03-HEALTH | health/OID/drift matrix | mutation=0 | PASS | directed suite |
| FX-AW03-PUBLISH | payload/receipt/handle tamper | 100% reject | PASS | directed suite |
| FX-AW03-RECOVERY | retry/resume/abort/cross-leg | contract preserved | PASS（fixed clock） | directed suite |
| FX-AW03-CLOCK | observer timestamp generated after execute entry | accept as fresh | FAIL | default-clock reproduction |

## 平台适配验证

| 平台 | 检查项 | 预期 | 结果 | 证据 |
|---|---|---|---|---|
| Linux/Python 3.11 | native fixture / static checks | pass | PASS | T-CMD-01..06 |
| Windows | native Git argv/path | not authorized | N/A | open low risk |
| hosted remote | branch protection/network/credential | not authorized | N/A | open medium risk |
| installer/Prompt platforms | 不适用 | N/A | N/A | Story scope |

## 覆盖结果

| Scenario ID | Story ID | 测试类型 | 覆盖状态 | 证据 | 缺口 / 原因 |
|---|---|---|---|---|---|
| TP-AW03-001..020 applicable | ST-AW-003 | unit/contract/integration | covered | 38 tests | 默认时钟维度未包含在既有 fixture |
| TC-AW-005..010/012/014 applicable | ST-AW-003 | regression/manual | covered | 56 tests + review | live-clock happy path gap |
| DEFAULT-CLOCK-FRESHNESS | ST-AW-003 | runtime boundary | gap/fail | T-CMD-07 | observer 在基准时间之后创建快照被误判 stale |

## 失败与缺口

| Finding ID | 严重度 | 问题 | 影响 | 下一动作 | 责任方 |
|---|---|---|---|---|---|
| TST-AW003-001 / REV-AW003-001 | BLOCKING | 默认时钟 freshness 上界早于 observer 时间戳 | non-dry-run source/artifact/resume 无法进入 runner | 修复时钟采样顺序/契约并新增无显式 `now` 回归 | meta-dev |

## 剩余风险

| Risk ID | 风险 | 等级 | 是否接受 | 接受人 / 条件 | 后续处理 |
|---|---|---|---|---|---|
| REV-AW003-001 | 默认时钟执行失败 | HIGH | no | 修复 + 复验 | 阻断 CP7 |
| CR051-RISK-REMOTE-UNVERIFIED | hosted remote 未验证 | MEDIUM | no | 独立 runtime authorization | CP8 风险候选 |
| CR051-RISK-WINDOWS-NATIVE-GIT-PILOT | Windows 未验证 | LOW | no | 等价环境 | follow-up candidate |

## 结论

`NEEDS_REWORK`

## 阶段决策

| 结论 | 路由 | 条件 / 说明 |
|---|---|---|
| `NEEDS_REWORK` | `meta-dev` | 修复 REV-AW003-001 后复跑最小回归与全套 packet 验证；本轮不得标记 verified |
