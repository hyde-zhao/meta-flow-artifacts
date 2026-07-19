---
cr_id: CR-051
stage: CP8-preparation
status: no-open-blocking-fix
generated_at: "2026-07-18T16:55:34Z"
---

# CR-051 修复与后续输入

## 1. 本轮已完成修复

| Finding | 修复摘要 | 验证 | 状态 |
|---|---|---|---|
| REV-AW003-001 | 默认时钟 freshness 使用一致时间基准，合法路径不再自拒绝，真实 stale/future 仍在执行前阻断 | 6 default-clock；44 directed；185 combined + 9 subtests | CLOSED |
| REV-AW005-001 | 对每个 source/target object 在 probe、读取、hash、mapping、descend 前执行 deny guard；相对 symlink 指向 denied target 时不暴露 target | 3 independent probes；26 directed；280 combined + 9 subtests | CLOSED |

## 2. 当前无需回修的事项

- ST-AW-001/002 bounded Host fallback 是验证独立性上限，不是已发现实现缺陷。
- capability registry 缺失是仓库 canonical 路由/注册表缺口，meta-doc 与本 meta-qa 均无权越界创建。
- 真实 remote、Windows、worktree、同步、迁移与发布均未授权，不能通过本轮测试或文档写作“补做”。

## 3. Follow-up tracking candidates

| Candidate | 类型 | 推荐 owner | 关闭条件 | 重访条件 |
|---|---|---|---|---|
| FU-CR051-001 | capability registry | Host Orchestrator / canonical docs owner | registry 恢复且两份用户文档 capability-claims checker PASS | 下一次公开发布或 capability 声明变化前 |
| FU-CR051-002 | module boundary manifest | architecture owner | canonical manifest 存在并通过边界检查 | 新模块继续扩展或跨包依赖变化时 |
| FU-CR051-003 | Windows native pilots | platform QA | Git/durability/lock/no-follow 原生 fixture 全通过 | 首次 Windows 接入前 |
| FU-CR051-004 | selector stale-lock recovery | meta-se/meta-dev | 有界恢复协议、崩溃 fixture 与 fail-closed 回归通过 | lock 残留在真实使用中出现或自动恢复被提出时 |
| FU-CR051-005 | directory enumeration TOCTOU | future migration CR | full-LLD 与 dirfd/openat 或等价链路验证通过 | 迁移输入可能受不可信并发修改时 |
| FU-CR051-006 | per-project real migration pilot | project owner | 独立授权、只读 preflight、人工审阅、可回滚 cutover 与双仓库证据完成 | 用户选择首个迁移项目时 |

这些条目只是 CP8 台账候选，不表示已经创建或启动新的 CR。

## 4. CP8 推荐

接受 `READY_WITH_RISK`；保持所有真实运行与仓库写操作不授权。若用户要求 `READY` 无风险，必须先补全独立 QA、capability registry、真实环境与平台 pilots，并重新进入质量复核。

## 5. CP8 R2 已完成项（2026-07-19）

| Candidate / Finding | 当前状态 | 证据 |
|---|---|---|
| FU-CR051-007 / ST-AW-002 design delta、独立性与 calibration | `CLOSED-IN-CURRENT-CR`（独立性部分改为显式风险，不伪称已补独立 QA） | `ST-AW-002.delta.json`、CP6/CP7-R2、R2 quality report |
| FU-CR051-008 / ST-AW-003 四个高优测试 | `CLOSED-IN-CURRENT-CR` | TP-AW03-004/006/008/014、CP6/CP7-R3 |
| FU-CR051-009 / ST-AW-004 PARTIAL/target policy/tests | `CLOSED-IN-CURRENT-CR` | `ST-AW-004.delta.json`、CP6/CP7-R2 |

FU-CR051-001..006 仍为 candidate-only；007..009 不再需要创建后续 CR。
