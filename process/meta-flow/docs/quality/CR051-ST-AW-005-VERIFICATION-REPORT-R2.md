---
status: "PASS_WITH_RISK"
version: "2.0"
scope: "CR-051 / ST-AW-005 / CP7-R2"
created_at: "2026-07-18T16:36:53Z"
created_by: "meta-qa-critical"
---

# Verification: ST-AW-005 CP7-R2

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | `PASS_WITH_RISK` |
| validation_mode | `mixed`（临时目录 fixture runtime + 静态审查） |
| 关闭 finding | `REV-AW005-001` |
| 路由 | host-orchestrator 组装 CP7-R2 result；四项剩余风险进入 CP8 |

逐对象 denied-descendant 修复已通过三类独立夹具和精确回归。合法宽 root 不再成为读取、哈希或映射 denied descendant 的旁路；相对软链接指向 denied descendant 时输出会脱敏并 fail closed。本次复验未发现新的实现缺陷。

## 2. 验证范围

本轮只验证 `REV-AW005-001` 的 bounded R2，以及修复对既有 manifest、no-follow、zero-mutation、O-AW-03 契约的回归影响。未执行真实 migration、软链接挂接、Git/worktree/ref/remote mutation，也未验证未来的对抗式目录替换方案。

## 3. 验证对象清单

| 对象 | 验证方式 | 结果 |
|---|---|---|
| `artifact_migration.py` per-object deny guard | 全文静态审查 + 独立夹具 | PASS |
| 宽 source root 内 denied descendant | 独立临时目录、probe/hash/mapping/leak 计数 | PASS |
| source allowed / mapped target denied | 独立临时目录、source/target probe 计数 | PASS |
| relative symlink → denied target | 独立临时目录、target probe 与 payload 脱敏 | PASS |
| 26 条定向测试 | pytest | PASS |
| routing/worktree 与跨 Story 相邻回归 | 95 / 280+9 精确套件 | PASS |
| 风险 API、格式、语法、whitespace | Ruff / format / py_compile / diff / `rg` | PASS |

## 4. 验证追踪矩阵

| 契约 / 风险 | 实现点 | 验证证据 | 状态 |
|---|---|---|---|
| `REV-AW005-001` / denied descendant probe=0 | `_deny_descendant()` 在 lstat/open/hash/mapping/descend 前执行 | 三类独立 probe + 3 closure tests | CLOSED |
| mapped target deny 不能通过 source 读取旁路 | source/target portable path 同时匹配 deny | mapped-target 独立 probe | PASS |
| relative symlink target 脱敏 | source 与 mapped-target prefix 归一化检查 | `link_target=null`、`relative-denied` | PASS |
| sentinel/content hash/mapping 泄漏=0 | denied object 不进入 mapping | payload 与 SHA-256 负向断言 | PASS |
| manifest/no-follow/zero-mutation/O-AW-03 不回归 | public schema 与现有路径不变 | 26 directed + 280/9 combined | PASS |

## 5. 设计契约验证清单

| 契约 | 阻断性 | 结果 |
|---|---|---|
| 每个 filesystem probe、mapping、descend 前应用 deny policy | BLOCKING | PASS |
| denied descendant 命中后必须非 `READY` 并留下 reason | BLOCKING | PASS；三类均为 `MANUAL_REVIEW` |
| denied policy 可保留于 scope 审计字段，但不得泄漏 sentinel/hash/descendant mapping | BLOCKING | PASS |
| 不改变 manifest schema、glob DSL、readiness enum 或 O-AW-03 | BLOCKING | PASS |
| mutation/Git/worktree/ref/remote/helper/scheduler 调用=0 | BLOCKING | PASS |

## 6. 分层验证计划与结果

| 层级 | 必跑 / 条件 | 结果 |
|---|---|---|
| 独立 negative fixture | 必跑 | 3/3 PASS |
| 定向测试 | 必跑 | 26/26 PASS |
| 相邻契约回归 | 必跑 | 95/95 PASS |
| exact combined | 必跑 | 280/280 + 9 subtests PASS |
| 静态质量与危险 API | 必跑 | 全部 PASS |
| workflow eval | N/A | bounded Python core-code 修复不生成 workflow/prompt；packet 未提供 eval refs，以原生精确 suite 作为等价证据 |
| 真实 migration / remote / Windows native | 未授权或条件触发 | 未执行，转剩余风险 |

## 7. 自动化与独立夹具结果

- closure selector：`3 passed, 23 deselected`。
- directed：`26 passed`。
- routing/worktree adjacent：`95 passed`。
- exact combined：`280 passed, 9 subtests passed`。
- Ruff：`All checks passed`；format：2 files already formatted；py_compile、diff check 均无错误；危险 API/import `rg` 无命中。
- 独立宽 root：decision=`MANUAL_REVIEW`，denied probe/mapping/leak=`0/0/0`。
- 独立 mapped target deny：decision=`MANUAL_REVIEW`，denied probe/mapping/leak=`0/0/0`。
- 独立 relative symlink：decision=`MANUAL_REVIEW`，`link_target=null`、class=`relative-denied`、denied target probe/leak=`0/0`。

## 8. 8 维度验收

| 维度 | 结果 | 说明 |
|---|---|---|
| 完整性 | PASS | R2 代码、测试、CP6-R2 证据齐备 |
| 平台适配 | PASS_WITH_RISK | Linux fixture 通过；Windows native no-follow 保留低风险 |
| 验收标准覆盖 | PASS | packet 6 项 acceptance 全有证据 |
| 安全合规 | PASS | deny/read-scope、no-follow、危险 API 审计通过 |
| 命名规范 | PASS | 既有模块与测试命名不变 |
| Frontmatter | N/A | 本 Story 交付对象不是 Agent/Skill frontmatter |
| 可安装性 | N/A | 未修改安装器或安装结构 |
| 文档覆盖 | PASS_WITH_RISK | R2 质量证据完整；最终用户文档由 documentation/CP8 汇总 |

## 9. 人工 / 语义质量审查

deny pattern 出现在 `scope.denied_paths` 属于可审计 policy，不等于泄漏；禁止的是 denied descendant 的 object probe、hash、mapping、link target 文本和 sentinel 内容。实现与测试已清楚地区分这两类信息。`MANUAL_REVIEW` 也没有被误写为运行授权或迁移完成。

## 10. 问题清单

当前 BLOCKER/HIGH finding 为 0。`REV-AW005-001` 状态为 `CLOSED-R2`。

## 11. 剩余风险与 CP8 输入

| 风险 | 级别 | 路由 | 推荐处理 |
|---|---|---|---|
| `R-AW005-DIR-ENUM-TOCTOU` | MEDIUM | follow-up candidate / risk acceptance | 当前静态只读范围接受；未来不可信并发目录访问时升级 full-lld，采用 dirfd/openat 全链路设计 |
| `CR051-RISK-REAL-MIGRATION-UNVERIFIED` | MEDIUM | not_authorized / runtime_authorization | 保持不授权；逐项目 migration CR 单独批准和验证 |
| `CR051-RISK-WINDOWS-NATIVE-NOFOLLOW` | LOW | risk acceptance / follow-up | 接受当前无可靠能力即 fail-closed；Windows 实际使用前做 native pilot |
| `CR051-RISK-REMOTE-UNVERIFIED` | MEDIUM | not_authorized / runtime_authorization | 保持不授权；托管 remote、凭据、branch protection、publication 需独立授权 |

`PASS_WITH_RISK` 只允许继续组装 CP7-R2/CP8 证据，不授权任何真实迁移、Git/worktree/ref/remote/link mutation、凭据访问、push 或 publish。
