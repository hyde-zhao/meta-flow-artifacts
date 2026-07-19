---
cr_id: CR-051
stage: CP8-preparation
decision: PASS_WITH_RISK
generated_at: "2026-07-18T16:55:34Z"
---

# CR-051 测试报告

## 1. 测试结论

5/5 Story 最新 CP7 均通过允许推进的质量门，结果全部为 `PASS_WITH_RISK`。自动化与独立 probe 覆盖了 project-first routing、worktree 恢复、双 leg 生命周期、聚合持久化以及只读迁移 preflight；本轮没有执行真实外部仓库、真实迁移或真实发布验证。

## 2. 聚合结果

| 维度 | 结果 | 量化摘要 |
|---|---|---|
| Story CP7 完成 | PASS_WITH_RISK | 5/5 |
| 开放 BLOCKER | PASS | 0 |
| 开放 HIGH | PASS | 0 |
| 已关闭 HIGH | PASS | 2/2 |
| 真实 remote/worktree/migration/publish | N/A-NOT-AUTHORIZED | 0 次 |
| Windows native pilots | N/A-RISK | 0 次 |
| capability claims 自动检查 | BLOCKED-RISK | 2 份用户文档均因 registry 缺失无法运行 |

## 3. Story 级测试证据

| Story | Directed / Focused | Adjacent / Combined | 结果 |
|---|---|---|---|
| ST-AW-001 | 38 targeted R2 | finding-specific + static checks | PASS_WITH_RISK |
| ST-AW-002 | 83 directed | CAP/DUR/WT/PORT 全组 | PASS_WITH_RISK |
| ST-AW-003 | 6 default-clock；44 directed | 62 adjacent；185 + 9 subtests | PASS_WITH_RISK |
| ST-AW-004 | 16/16 状态组合、14 invalid/unpublished、15 non-PASS | 179 + 9 subtests | PASS_WITH_RISK |
| ST-AW-005 | 3 independent closure probes；26 directed | 95 adjacent；280 + 9 subtests | PASS_WITH_RISK |

CP8 准备阶段由 Host Orchestrator 执行全仓只读回归：`PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q`，结果为 `680 passed, 70 subtests passed in 6.83s`。

静态聚合检查中，`ruff check meta_flow tests`、delivery guardrail 与 `git diff --check` 通过。`ruff format --check meta_flow tests` 未全绿（98 个文件会被重排、15 个已格式化），属于仓库既有全局格式基线债务；没有执行批量格式化，且各 Story 定向 format checks 已通过。

计数为不同阶段/范围的运行摘要，Story 集合与全仓回归存在重叠，不应相加解释为唯一测试用例总数。

## 4. 质量维度

| 质量维度 | 结果 | 证据/限制 |
|---|---|---|
| 功能适合性 | PASS | 五个 Story 的批准契约均有实现与 CP7 证据 |
| 可靠性 | PASS_WITH_RISK | fail-closed、durable intent、CAS、stale/future、deny descendant 已测；真实环境未测 |
| 安全性 | PASS_WITH_RISK | dangerous API/argv/forbidden import 审计通过；未涉及凭据或真实外部写入 |
| 可维护性 | PASS_WITH_RISK | ruff/format/compile 通过；module boundary manifest 缺失 |
| 可移植性 | PASS_WITH_RISK | Linux fixture 通过；Windows native 与 cross-device pilot 未执行 |
| 兼容性 | PASS_WITH_RISK | 源码 default lifecycle 保留；artifact 新拓扑需要未来逐项目迁移 |
| 易用性 | PASS_WITH_RISK | 用户文档已更新；capability registry 缺失阻止自动 claims 校验 |
| 性能效率 | N/A | 本 CR 无已批准性能 SLO；容量估算验证关注 fail-closed 可靠性而非吞吐基准 |

## 5. 缺陷与修复结果

| ID | 首轮严重度 | 修复验证 | 当前状态 |
|---|---|---|---|
| REV-AW003-001 | HIGH | default-clock 6 个重点路径 + 相邻/组合回归 | CLOSED |
| REV-AW005-001 | HIGH | 3 个独立 closure probes + 26/95/280 回归 | CLOSED |

## 6. 未执行项

以下项目因授权边界而未执行，不应解读为测试通过：真实 hosted remote/branch protection/credentials；真实项目 worktree/ref/branch 切换；main↔integration 人工同步；真实迁移、复制、移动、删除或软链接重挂；Windows native Git/durability/lock/no-follow；源码或 artifact commit、push、publish/release。

## 7. 证据入口

- CR 级验证摘要：`process/docs/quality/CR051-VERIFICATION-REPORT.md`
- Story Return Packet：`process/returns/ST-AW-001.CP7-R2.return.json` 至 `process/returns/ST-AW-005.CP7-R2.return.json`（按各 Story 最新命名）
- 最新 CP7 machine result：`process/checks/CP7-CR051-*.result.json`

## 8. CP8 R2 增量测试结果（2026-07-19）

| 范围 | 最新结果 |
|---|---:|
| ST-AW-002 终验回修 | 74 passed |
| ST-AW-003/legacy/ST-AW-004 | 127 passed |
| CR-051 跨模块 | 307 passed、58 subtests passed |
| 全仓 | 697 passed、70 subtests passed |

新增覆盖关闭 ST-AW-002 proof/owner/calibration/phase/idempotency、ST-AW-003 TP-AW03-004/006/008/014 和 ST-AW-004 target-policy/PARTIAL/dependency/DAG 缺口。真实 runtime mutation=0；Host inline-fallback 不等于独立 QA。
