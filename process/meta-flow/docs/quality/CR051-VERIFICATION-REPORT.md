---
cr_id: CR-051
stage: CP8-preparation
validation_mode: mixed
decision: PASS_WITH_RISK
generated_at: "2026-07-18T16:55:34Z"
---

# CR-051 聚合验证报告

## 1. 结论

CR-051 的 5 个 Story 均已取得最新 CP7 `PASS_WITH_RISK`，开放 BLOCKER 为 0，开放 HIGH finding 为 0。实现与离线 fixture 证据足以进入 CP8 人工终验，但不证明真实托管 remote、branch protection、Windows native、真实 worktree/ref、main↔integration 同步、迁移、软链接、提交、推送或发布可安全执行。CR 级发布准备结论最高为 `READY_WITH_RISK`。

Host Orchestrator 在 CP8 准备阶段额外执行了全仓只读回归：`680 passed, 70 subtests passed in 6.83s`。该结果补强相邻回归信心，但不改变真实 runtime 未授权边界，也不应与各 Story 运行计数相加解释为唯一测试总数。

Host 同时确认 `ruff check meta_flow tests`、delivery guardrail 与 `git diff --check` 通过。全仓 `ruff format --check meta_flow tests` 报告 98 个文件会被重排、15 个已格式化，这是既有全局格式基线债务；本轮没有越界执行批量格式化，也不把它归因于 CR-051 定向文件格式失败。

## 2. 验证对象与最新结果

| Story | 能力 | 最新 CP7 | 代表性验证 | 证据入口 |
|---|---|---|---|---|
| ST-AW-001 | project-first artifact routing 与 owned-path 隔离 | PASS_WITH_RISK | 38/38 targeted R2 | `process/returns/ST-AW-001.CP7-R2.return.json` |
| ST-AW-002 | 可恢复 worktree switch、容量估算、durable intent | PASS_WITH_RISK | 83 directed | `process/returns/ST-AW-002.CP7.return.json` |
| ST-AW-003 | source/artifact 异构 Git leg 生命周期 | PASS_WITH_RISK | 6 default-clock；44 directed；62 adjacent；185 combined + 9 subtests | `process/returns/ST-AW-003.CP7-R2.return.json` |
| ST-AW-004 | 双 leg 最差状态聚合与 fail-closed 持久化/CAS | PASS_WITH_RISK | 179 combined + 9 subtests | `process/returns/ST-AW-004.CP7.return.json` |
| ST-AW-005 | 只读迁移 preflight manifest 与 handoff | PASS_WITH_RISK | 3 independent closure probes；26 directed；95 adjacent；280 combined + 9 subtests | `process/returns/ST-AW-005.CP7-R2.return.json` |
| CR 全仓回归 | 当前工作树只读回归 | PASS | 680 passed + 70 subtests | Host Orchestrator CP8 aggregate verification |

## 3. 生命周期契约验证

| 契约 | 验证结论 | 边界 |
|---|---|---|
| 源码仓库 leg | CR 从最新 default branch（`main` 或 `master`）派生，完成目标为 default branch | 本轮未 commit、push、merge 或 publish |
| artifact 仓库 leg | 保留 shared `main`；每项目长期 `projects/<project>/integration`；每 CR 使用项目命名空间短期分支 | CR 只回合到项目 integration，不直接接触 shared `main` |
| main↔integration 同步 | 明确为 CR 外人工操作 | 本轮未执行任何方向的同步 |
| worktree 驻留 | 空闲驻留项目 integration，活动 CR 时切到对应短期分支 | 仅临时 fixture 验证，没有真实项目 worktree 切换 |
| 双 leg 完成 | source/artifact 记录同一 CR ID，整体状态取最差状态，2/2 PASS 才投影完成 | 任一 leg 非 PASS 均 fail closed |
| 迁移 preflight | 只读 manifest、deny descendant、no-follow、zero-mutation | 没有迁移 CLI，也没有执行真实迁移或软链接变更 |

## 4. 已关闭 HIGH finding

| Finding | 原问题 | 关闭证据 | 状态 |
|---|---|---|---|
| REV-AW003-001 | 默认时钟 freshness happy path 可能自拒绝 | 4 条合法 `now=None` 路径通过，2 条真实 stale/future 在 runner 前阻断 | CLOSED-R2 |
| REV-AW005-001 | 宽允许根下被 deny 的后代仍可能被读取/映射并误报 READY | 3 个独立 deny-descendant probe 证明 probe/mapping/leak 均为 0，结果非 READY | CLOSED-R2 |

## 5. QA provenance 与独立性上限

- ST-AW-003 R2、ST-AW-004、ST-AW-005 R2 由实际独立 meta-qa lane 验证。
- ST-AW-001 R2 与 ST-AW-002 在 QA timeout 后采用 bounded Host fallback；证据可复跑且未发现开放 HIGH，但不能宣称“五个 Story 全部由独立 QA 完成”。
- ST-AW-005 首轮问题由实际 QA 发现；QA agent timeout 后 Host 仅按已报告 finding 有界落盘首轮报告文件。随后同一 QA lane 独立重建 3 个 R2 probe 并关闭 finding。该 provenance 不等于 Host 伪造独立验证。

## 6. 文档验证

| 检查 | 结果 | 说明 |
|---|---|---|
| 文档 diff whitespace | PASS | `delivery/README.md`、`delivery/doc/USER-MANUAL.md` 无 whitespace error |
| delivery guardrail | PASS_WITH_WARNINGS | guardrail OK；仅忽略本地 `__pycache__` |
| capability claims checker | BLOCKED-RISK | canonical `docs/design/CAPABILITY-STATUS.yaml` 缺失，无法自动验证两份用户文档的 capability claims |
| 用户文档语义核对 | PASS_WITH_RISK | 已明确区分源码与 artifact 生命周期、现有 CLI、offline fixture 与 not-authorized 边界 |
| 全仓 ruff format baseline | NON-BLOCKING-LIMITATION | 98 个既有文件会被重排；CR-051 定向 Story format checks 已通过 |

capability registry 缺失未造成当前用户文档声称真实 runtime-ready；文档已显式标注能力边界，因此作为 CP8 风险披露而非开放实现 BLOCKER。恢复 canonical registry 后必须重跑 capability-claims checker。

## 7. 剩余风险

| Risk ID | 严重度 | 状态 | 处置 |
|---|---|---|---|
| CR051-RISK-PARTIAL-INDEPENDENT-QA | MEDIUM | open-risk-acceptance | CP8 明确接受独立性上限；需要时另行独立复验 ST-AW-001/002 |
| DOC-CR051-CAPABILITY-REGISTRY-MISSING | MEDIUM | open-follow-up | 恢复 canonical registry 并重跑文档声明检查 |
| CR051-RISK-REMOTE-UNVERIFIED | MEDIUM | open-not-authorized | 真实 hosted remote、凭据、branch protection 与 publication 另行授权 |
| CR051-RISK-WINDOWS-NATIVE-PILOTS | LOW | open-follow-up | Windows native Git/durability/lock/no-follow 另行 pilot |
| R-AW004-LOCK-CRASH | LOW | open-fail-closed | crash 残留 lock directory 时人工诊断或受控恢复 |
| R-AW005-DIR-ENUM-TOCTOU | MEDIUM | open-follow-up | 不可信并发目录场景需 full-LLD + dirfd/openat 设计 |
| CR051-RISK-REAL-MIGRATION-UNVERIFIED | MEDIUM | open-not-authorized | 每项目迁移与切换必须另行授权并执行 preflight |
| CR051-RISK-MODULE-BOUNDARIES-MANIFEST | MEDIUM | open-follow-up | 当前边界证据来自代码/测试；后续补 canonical manifest |

## 8. CP8 输入

- 推荐：以 `READY_WITH_RISK` 进入 CP8 人工终验。
- 阻断项：0。
- 风险接受：接受 bounded Host fallback 的独立性上限及 offline/fixture-only 交付边界。
- 不授权：任何真实 Git/worktree/ref/remote、main↔integration 同步、迁移、链接、commit、push、publish 或凭据/网络操作。

## 9. CP8 R2 终验回修增量（2026-07-19）

ST-AW-002/003/004 已按终验意见回修并生成新的 CP6/CP7 证据：ST-AW-002 CP7-R2、ST-AW-003 CP7-R3、ST-AW-004 CP7-R2 均为 `PASS_WITH_RISK`。三份 design delta 已合并；新增 typed proof/owner/calibration/phase、四个 leg 高优测试、containment/exact CAS cleanup、consumer target policy、PARTIAL/retry 与 dependency/DAG 证明。

最新全仓回归为 `697 passed, 70 subtests passed in 6.62s`；Ruff lint、11 个核心文件 scoped format、Python 3.11 compile、delivery guardrail、`git diff --check`、3/3 delta 与 3/3 LLD 结构检查均通过。全仓 format 基线现为 92 个历史文件待重排、22 个已格式化。

本轮用户明确禁止拉起子 Agent，因此新增 R2/R3 验证由 Host inline-fallback 执行，不声称新的独立 QA 隔离。结论仍为 `PASS_WITH_RISK`。
