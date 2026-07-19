---
status: final
version: "1.0"
scope: "CR-051 / ST-AW-003"
artifact: "meta_flow/workflow/artifact_leg_lifecycle.py"
reviewer: "meta-qa-critical (qa-cao)"
lane: "quality-critical"
input_type: "review_findings"
round: 1
decision: "request-changes"
created_at: "2026-07-18T14:34:53Z"
---

# Review: CR-051 / ST-AW-003

## Findings

| ID | 严重度 | 位置 | 问题 | 影响 | 建议 |
|---|---|---|---|---|---|
| REV-AW003-001 | HIGH / BLOCKING | `meta_flow/workflow/artifact_leg_lifecycle.py:819,844-850,859-864,1744-1746`；覆盖缺口见 `tests/test_artifact_leg_lifecycle.py:608-614,860-867` | `execute_leg()` 在调用 `observer` / `health_observer` 前捕获 `executed_at`，随后 `_validate_freshness()` 将任何 `observed_at > executed_at` 判为 stale。正常 observer 在被调用时才取当前时间，因此默认 `now=None` 路径把合法且更新鲜的快照误判为 future/stale。现有正向测试都显式注入固定 `now`，使基准时间等于或晚于 fixture observation，遮蔽了真实调用顺序。 | source、artifact 和 ordinary resume 的非 dry-run 默认路径会在 runner 前返回 `BLOCKED(stale_observation)`，required leg 无法执行。安全上仍 fail-closed 且 mutation=0，但功能适合性/可靠性 P0 不满足。 | 在 observation 返回后取得相应校验基准，或注入单调/墙钟 clock 并定义有限 future-skew；pre/post/health 校验应使用不早于快照生成的基准。新增不传 `now` 的 source、artifact、ordinary-resume、post-observe 回归，并保留旧 stale/future-clock-skew 负向测试。 |

复现（纯内存 observer、runner spy、无 Git mutation）：默认 `execute_leg(..., now=None)` 输出 `status=BLOCKED`、`blocker=stale_observation`、`mutation_count=0`、`runner_calls=0`。这不是真实 remote 行为差异，而是确定性的本地时间顺序问题。

## 测试缺口

| Gap ID | 来源 | 缺口 | 风险 | 建议 |
|---|---|---|---|---|
| GAP-AW003-001 | ST-AW-003 fresh proof / TP-AW03-005/009/020 | 所有 execute 正向测试都注入固定 `now`；没有 `now=None` + observer 自取当前时间的 happy-path fixture | HIGH：全绿 suite 无法证明默认 API 可运行 | 增加默认时钟四路回归；至少断言 runner=1、post-proof PASS，并单独保留真正未来 skew 仍 fail-closed |
| GAP-AW003-002 | platform risk | Windows native Git 与 hosted branch protection 未验证 | LOW/MEDIUM：平台/hosted 行为未知 | 保持不授权；仅在后续独立 runtime authorization / 等价环境执行，不影响本次缺陷定级 |

## 人工 / 语义质量审查

| 检查项 | 结果 | 是否阻塞 | 说明 |
|---|---|---|---|
| 需求一致性 | FAIL | yes | fresh proof 应拒绝陈旧快照，但不应拒绝 observer 调用期间产生的更新快照 |
| 场景覆盖 | RISK | yes | target/authz/publication/recovery 负向覆盖良好；默认 API happy path 缺失 |
| Prompt / Agent 边界 | N/A | no | 非 Prompt/Agent 产物 |
| 文档可用性 | RISK | no | 模块顶部 docstring 仍称“不会执行任何 Git 命令”，与当前完整 executor 不一致；不单独阻断，但修复时应同步更新 |
| 错误信息可行动 | RISK | no | 稳定错误码存在，但当前 `stale_observation` 混淆陈旧与观察发生在基准之后 |
| 是否只覆盖 happy path | FAIL | yes | 负向路径丰富，但默认 live-clock happy path 反而未覆盖 |

## 设计契约与实现证据审查

| 检查项 | 结果 | 风险 | 建议 |
|---|---|---|---|
| 验证对象清单完整 | PASS | 无 | packet 两个 primary + 相邻回归 + CP6 evidence 均已消费 |
| 验证追踪矩阵完整 | PASS | 已暴露 clock gap | 复验时将新增 default-clock case 回链到 fresh proof |
| 设计契约验证完整 | FAIL | fresh proof 正常路径未满足 | 修复并复验，不修改上游验收目标 |
| 实现执行证据可验证 | PASS | CP6 自证未发现 clock gap | CP6 return/evidence/result 完整；独立 CP7 finding 优先于自证结论 |
| exact target / no main / no sibling | PASS | 无 | 既有定向与静态边界全绿 |
| typed authz / OID / health fail closed | PASS | 无越权 mutation | 保留现有 negative matrix |
| immutable payload / external receipt / reread handle | PASS | hosted store 未验证 | 本地 contract/tamper matrix 全绿 |
| evidence retry / resume / abort / no cross rollback | PASS（固定时钟） | resume 默认时钟受 finding 影响 | 修复后补默认时钟 resume 回归 |

## 安全扫描结果

| 匹配 | 位置 | 级别 | 判断 |
|---|---|---|---|
| `--force` / `reset` / `clean` / `stash` / `rebase` | `_validate_safe_step` denylist | INFO | 防护代码，不是执行命令 |
| `run_git` | `_default_git_runner` | EXPECTED-HIGH-RISK-PORT | argv list、exact push、no shell；由 typed policy/observer 前置门约束 |
| subprocess | 仅定向测试临时 Git fixture | FIXTURE-ONLY | 不存在于 SUT import/call；不触及真实仓库 |

总体危险命令风险：`LOW`；critical 命中 0。当前阻断来自可靠性，不是危险 mutation。

## 合并建议

| 结论 | 条件 |
|---|---|
| `request-changes` | 关闭 REV-AW003-001；新增默认时钟回归；38 directed、56 adjacent、Ruff/format/compile/diff/安全审计全部复验通过后，重新执行独立 CP7 |
