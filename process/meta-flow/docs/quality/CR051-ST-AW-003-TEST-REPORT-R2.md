---
status: final
version: "2.0"
scope: "CR-051 / ST-AW-003 CP7-R2"
created_at: "2026-07-18T15:15:41Z"
validation_mode: "mixed"
verification_result: "PASS_WITH_RISK"
created_by: "meta-qa-critical (qa-cao)"
---

# Test Report R2: CR-051 / ST-AW-003

## 验证范围

| 项 | 内容 |
|---|---|
| Story | `ST-AW-003` |
| 目标 | 独立关闭 `REV-AW003-001`，证明 default-clock freshness 修复不放宽 stale/future 门且无异构 leg 回归 |
| 非范围 | 源码/测试修改；真实 remote/worktree/ref/link/publication；凭据、网络、Windows native Git |
| 实现证据 | CP6-R2 return/evidence/result |
| validation_mode | mixed：fixture runtime + static + manual |

## 验证对象与追踪

| 对象 / Scenario | 契约 | Test / Check | 结果 | 风险 |
|---|---|---|---|---|
| source default clock | observer后采样 | 6-case selector | PASS | N/A |
| artifact default clock + health | observation/health各自fresh | 6-case selector | PASS | N/A |
| ordinary resume | new attempt + fresh replan | 6-case selector | PASS | N/A |
| post-observe | runner后fresh OID proof | 6-case selector | PASS | N/A |
| stale/future | runner=0 fail closed | 2 negative cases | PASS | N/A |
| target/authz/health/publication/recovery | frozen contract | 44 directed / 62 adjacent | PASS | N/A |
| aggregate/state/CR lifecycle | W3 compatibility | 185 tests + 9 subtests | PASS | N/A |
| hosted/Windows | platform pilot | not authorized | N/A | MEDIUM/LOW open |

## 设计契约验证

| 契约 | 结果 | 证据 |
|---|---|---|
| 合法 now=None 四路径 4/4 | PASS | tests lines 607-741；6-case output |
| genuine stale/future 2/2 BLOCKED、runner=0 | PASS | tests lines 744-779；6-case output |
| 显式 now 及原 target/authz/health/publication/recovery 无变化 | PASS | 44 directed / source review |
| 真实 publication=0 | PASS | packet boundary / tmp fixture only |

## 分层验证结果

| 层 | 命令 / 方法 | 结果 |
|---|---|---|
| Finding closure | `pytest ... -k default_clock` | 6 passed, 38 deselected |
| Directed | `pytest tests/test_artifact_leg_lifecycle.py` | 44 passed |
| Adjacent | + branch lifecycle + workspace git sync | 62 passed |
| W3 combined | Host exact six-file command | 185 passed, 9 subtests passed |
| Lint/format | Ruff | PASS |
| Compile | Python 3.11 py_compile | PASS |
| Diff/whitespace | scoped diff + no-index check | PASS |
| Security | dangerous API/import/ref audit | PASS，critical=0 |

## 测试环境

| 字段 | 值 |
|---|---|
| Runtime | Linux / Python 3.11 / uv / pytest |
| Git | pytest tmp bare remote；in-memory runner/writer spy |
| Credentials/network | not provided / not read / not used |
| Primary status | 两文件为 created/untracked assets；R2 logical change由CP6-R2 return记录 |

## Fixture 结果

| ID | 场景 | 预期 | 实际 | 结果 |
|---|---|---|---|---|
| R2-FX-01 | source complete now=None | PASS, runner=1 | PASS, mutation=1 | PASS |
| R2-FX-02 | artifact complete now=None | PASS, runner=1 | PASS, mutation=1 | PASS |
| R2-FX-03 | ordinary resume now=None | attempt=2, PASS | attempt=2, PASS | PASS |
| R2-FX-04 | post-observe after runner | PASS after fresh proof | PASS | PASS |
| R2-FX-05 | stale | BLOCKED, runner=0 | BLOCKED, mutation=0 | PASS |
| R2-FX-06 | future skew | BLOCKED, runner=0 | BLOCKED, mutation=0 | PASS |

## 覆盖结果

| Acceptance | 覆盖状态 | 证据 | 缺口 |
|---|---|---|---|
| 4合法 now=None | covered | 4 positive tests | none |
| 2 genuine stale/future | covered | 2 negative tests | none |
| 44/62/185+9 | covered | exact outputs | none |
| static/security | covered | Ruff/format/compile/diff/rg | none |
| finding closure/publication=0 | covered | review + command scope | hosted behavior intentionally unverified |

## 失败与缺口

无测试失败、无新实现缺口。`REV-AW003-001` 已关闭。

## 剩余风险

| Risk ID | 等级 | 状态 | 下一动作 |
|---|---|---|---|
| CR051-RISK-REMOTE-UNVERIFIED | MEDIUM | open-not-authorized | CP8 风险分流；独立授权前禁止真实 publish |
| CR051-RISK-WINDOWS-NATIVE-GIT-PILOT | LOW | open | 后续等价环境 pilot candidate |

## 结论

`PASS_WITH_RISK`

## 阶段决策

| 结论 | 路由 | 说明 |
|---|---|---|
| PASS_WITH_RISK | host-orchestrator | 可组装 CP7 result；保留平台风险与不授权边界 |
