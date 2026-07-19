---
feature_id: "FEAT-AW-05"
feature_name: "Migration Preflight & Handoff"
change_id: "CR-051"
story_ids: ["ST-AW-005"]
status: "ready-for-story-design"
applicability: "required"
lld_policy: "technical-note"
version: "1.0"
updated_at: "2026-07-18"
---

# FEAT-AW-05 只读迁移预检与交接设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 固化 read-only MigrationManifest、deny-mutation、验证/回滚 handoff，以及 O-AW-03 指标阈值只生成 follow-up candidate。 |

## 1. 目标与适用性

FEAT-AW-05 为一个显式目标项目读取 legacy 与 project-first route/worktree 事实，生成可审计的 `MigrationManifest` 和人工执行 handoff。CR-051 在本 Feature 内真实文件移动、复制、删除、软链接变更、worktree/ref 变更、commit/publish 均为 0。

该 Feature 的 implementation design 判定为 `required`，因为 manifest schema、read scope、hash/count、readiness 与 deny-mutation 属于跨模块交接契约。`ST-AW-005` 当前保持 `technical-note`：实现只读，无新持久化 schema writer、无 mutation、无跨 Story runtime 写契约；若后续加入任何文件/link/worktree/ref mutation 或自动同步，必须升级为 full LLD 并重新经过设计确认。

量化成功标准：

- 对显式目标项目输出 source→target mapping、hash/count、link plan、readiness、validation、rollback 与 authorization status 字段覆盖率 100%。
- preflight/handoff 执行期间文件/link/worktree/ref/remote mutation 数=0，commit/publish 数=0。
- 不确定或不可读对象 100% 进入 BLOCKED/manual action，不猜测、不降级为 ready。
- sibling project、`process/quant-lab/**` 与未显式目标内容读取数=0。
- O-AW-03 三个阈值判定可复算；达阈值时只输出 follow-up CR candidate，sync helper enable/invoke 数=0。

## 2. 边界

### 2.1 In Scope

- 读取 FEAT-AW-01 的 validated route decision 与 FEAT-AW-02 的 read-only worktree health。
- 枚举显式目标项目的 legacy source 与 proposed project-first target，生成 canonical mapping。
- 计算可读取普通文件的 count/size/content hash；记录 link 类型与目标文本，不跟随越界链接。
- 输出 proposed link/worktree/ref steps、人工授权点、prechecks、post-validation 和 rollback checklist。
- 只读消费 manual-sync metrics summary，按 O-AW-03 生成 `none` 或 `follow-up-candidate` 建议。

### 2.2 Out of Scope 与相邻边界

| 对象 | 禁止行为 | 后续正确路径 |
|---|---|---|
| 文件系统 | move/copy/delete/write/chmod/chown/symlink/replace | 项目级 migration CR + 明确 runtime authorization |
| Git/worktree | add/remove/switch/reset/clean/stash/ref update/commit/push | 独立执行计划、锁、dry-run 与人工门 |
| Routing | 写 config/route metadata 或改变唯一 write target | FEAT-AW-01 controlled writer |
| Worktree | bootstrap/create/repair/recover | FEAT-AW-02 lifecycle |
| Manual Sync | 自动执行 main↔integration、注册定时任务、启用 helper | CR 外命令；达到阈值后另建条件式 CR |
| 跨项目 | 扫描 sibling owned paths 或 quant-lab 内容 | 逐项目显式 target，默认拒绝扩展 |

## 3. `MigrationManifest` 契约

| 分区 | 必需字段 / 约束 |
|---|---|
| identity | schema/version、manifest ID、project ID、route mode、source/target repo identity、observed_at |
| scope | explicit roots、allowed read paths、denied paths、symlink policy、enumeration completeness |
| mapping | 每项 source relative path、target relative path、type、size、hash/link text、conflict/readability |
| summary | file/link/dir counts、total bytes、hash algorithm、missing/unreadable/conflicting counts |
| link plan | proposed link path/target、preconditions、manual authorization、expected post-state；仅计划 |
| worktree/ref plan | proposed integration/worktree role 与 expected OIDs；仅引用 fresh health，不执行 |
| readiness | `READY | BLOCKED | MANUAL_REVIEW`、逐项 reason、authorization status |
| validation | 执行后应运行的 hash/count/link/route/worktree/ledger checks；本 CR 不宣称已执行 |
| rollback | 备份/恢复建议、触发条件、owner、验证；不自动执行 |
| ops follow-up | metrics window/completeness、三阈值结果、candidate decision/reason；不含 helper activation |
| evidence | input refs、content digest、read errors、generated_at；不得自引用未来 commit OID |

Manifest 只能使用 anchor + relative path 表达可迁移位置，不写设备相关绝对路径为持久化 contract。运行时诊断可显示 resolved path，但必须与 portable field 分离。

## 4. Read-only preflight 流程

```text
explicit project selector
  -> validate ProjectContext + RouteDecision
  -> read WorktreeHealth (no lifecycle call)
  -> resolve allowlisted source/target roots
  -> lstat/enumerate without following out-of-scope links
  -> hash/count readable target-project objects
  -> classify conflict/unreadable/incomplete
  -> attach proposed manual link/worktree/ref steps
  -> evaluate O-AW-03 metrics summary
  -> emit manifest to caller/stdout/evidence handoff
  -> zero mutation
```

### 4.1 前置校验与失败行为

| Precheck | 不满足 | 输出 | mutation |
|---|---|---|---|
| project selector 唯一且 identity 匹配 | BLOCKED | identity conflict | 0 |
| route metadata 可验证 | BLOCKED | route errors | 0 |
| source/target 均在 allowed scope | BLOCKED | scope violation | 0 |
| 普通文件可读且枚举完整 | MANUAL_REVIEW/BLOCKED | unreadable/incomplete 清单 | 0 |
| symlink target 可按文本归档且不需越界跟随 | MANUAL_REVIEW | link plan | 0 |
| worktree health fresh | BLOCKED | required re-observation | 0 |
| metrics window完整 | 不触发阈值结论 | `insufficient-data` | 0 |

## 5. O-AW-03 运维指标与 follow-up

本 Feature 只消费由 CR 外 Manual Sync operation owner 记录的频率、开始/结束时间、duration、blocker category、是否可避免与 project identity，不成为新的 metrics writer。

| Threshold ID | 固定条件 | 满足后的唯一动作 |
|---|---|---|
| O-AW-03-T1 | 单项目每周 manual sync ≥3 次，且连续 4 周 | 在 manifest 标记 `follow-up-candidate` |
| O-AW-03-T2 | 观察窗口内 manual sync duration 中位数 >10 分钟 | 同上 |
| O-AW-03-T3 | 可避免的调度阻塞率 >5% | 同上 |

任一阈值满足只允许建立“条件式同步助手”独立 CR 候选，候选需重新确认 scope、security、runtime authorization、rollback 和调度策略。CR-051 不创建实现 Story、不启用命令、不注册 scheduler、不执行 remote write。指标不足、窗口不完整或 blocker 分类不可信时输出 `insufficient-data`，不得猜测满足。

## 6. 调用契约

| 调用方 → 被调用方 | 时机 | 输入 | 输出 | 失败/降级 | 同步范围 |
|---|---|---|---|---|---|
| Migration Preflight → Context Resolver | 开始 | explicit project ID | validated context/route | BLOCKED，0 read expansion/mutation | FEAT-AW-01 schema |
| Migration Preflight → Worktree Health Reader | context 后 | project/worktree identity | fresh read-only health | BLOCKED；不得调用 repair/switch | FEAT-AW-02 contract |
| Enumerator → Filesystem Reader | scope 验证后 | allowlisted roots | lstat/count/hash/errors | unreadable/incomplete 显式记录 | 不跟随越界 symlink |
| Follow-up Evaluator → Metrics Summary Reader | manifest 汇总时 | project/window summary | threshold booleans/reasons | insufficient-data | CR 外 metrics owner |
| Caller → Manifest Renderer | 预检完成 | typed manifest | JSON/Markdown handoff | 渲染失败不执行任何 migration | ST-AW-005 technical note |

## 7. 模块与文件影响候选

| 类型 | 候选路径 | 责任 / owner | 约束 |
|---|---|---|---|
| 新模块 | `meta_flow/workspace/artifact_migration.py` | ST-AW-005：read-only preflight、manifest、threshold evaluator | primary |
| 测试候选 | `tests/test_artifact_migration.py` | ST-AW-005 | primary；临时 fixture only |
| 只读 contract | `meta_flow/workspace/routing.py` | route/context | 不修改 |
| 只读 contract | FEAT-AW-02 worktree health module | health | 不调用 mutation API |
| CLI | `meta_flow/cli.py` | 由 ST-AW-004 统一接线；若增加 migration 子命令须串行交给该 owner | ST-AW-005 不并行写 |

## 8. Story 技术说明要求

`ST-AW-005` Story 卡的 `## 技术说明` 是正式设计证据，必须引用本三件套并明确：read-only API、manifest schema、allowlisted reads、symlink policy、hash/count、readiness 分类、validation/rollback handoff、O-AW-03 evaluator 与 deny-mutation tests。任何 mutation/schema writer/cross-Story runtime contract 增量都会触发 `technical-note -> full-lld` 重访。

## 9. 风险与 Gotchas

| 风险 | 缓解 | 验证 |
|---|---|---|
| “plan”被当执行授权 | manifest 明示 proposed/manual/authorization status | mutation spy=0 |
| hash 时越界读取 sibling | explicit scope + lstat + no-follow | sentinel fixture |
| 不完整枚举误报 READY | completeness 字段 + fail closed | EACCES/broken link fixture |
| 指标达阈值偷渡自动 sync | output enum 仅 candidate；无 executor dependency | helper call=0 |

Gotchas：生成 manifest 文件本身若由调用环境落盘，也只是 process evidence 写入，不得写目标项目迁移路径；rollback checklist 是未来执行方的步骤，不是本 Feature 的恢复实现；绝对路径可以出现在临时诊断中，但不能成为 portable mapping 的真相字段。
