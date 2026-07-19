---
feature_id: "FEAT-AW-05"
change_id: "CR-051"
story_ids: ["ST-AW-005"]
status: "ready-for-story-design"
version: "1.0"
---

# FEAT-AW-05 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 将 read-only manifest、scope/hash、handoff、O-AW-03 evaluator 与 deny-mutation validation 拆为稳定 TASK-ID。 |

## 1. Story 与证据级别

全部任务归属 `ST-AW-005`。当前 `lld_policy=technical-note`，正式证据写入 Story 卡 `## 技术说明` 并引用本三件套。以下任务不得被解释为真实迁移授权；任何 mutation 或自动同步需求会触发 full LLD 重访。

## 2. 任务 DAG

```text
TASK-AW-005-01
  ├─> TASK-AW-005-02 ─┐
  └─> TASK-AW-005-03 ─┼─> TASK-AW-005-04
```

| TASK-ID | 状态 | depends_on | 动作与产物 | 完成准则 |
|---|---|---|---|---|
| TASK-AW-005-01 | planned | ST-AW-001/002 contracts；ST-AW-003/004 verified | 在 Story technical note 固化 read-only API、MigrationManifest schema、scope/symlink/readiness/error enum、policy重访条件 | 字段、调用方向、read boundary、deny mutation与follow-up边界100%明确 |
| TASK-AW-005-02 | planned | TASK-AW-005-01 | 实现 allowlisted lstat/enumeration/hash/count 与 mapping/validation/rollback handoff 候选切片 | incomplete/unreadable fail closed；portable relative mapping；mutation=0 |
| TASK-AW-005-03 | planned | TASK-AW-005-01 | 实现 O-AW-03 read-only threshold evaluator 和 deduplicated follow-up candidate | 三阈值边界准确；insufficient-data不猜测；helper调用/启用=0 |
| TASK-AW-005-04 | planned | TASK-AW-005-02、TASK-AW-005-03 | 建立 schema、permission/symlink/scope、mutation spy、snapshot、threshold/idempotency tests | TEST-PLAN P0/P1 100%通过；目标前后快照一致；越界读取=0 |

## 3. 文件所有权

| 分类 | 路径 | owner / 规则 |
|---|---|---|
| primary | `meta_flow/workspace/artifact_migration.py` | ST-AW-005 独占写；只读 preflight |
| primary | `tests/test_artifact_migration.py` | ST-AW-005 独占写；临时 fixture |
| read-only | `meta_flow/workspace/routing.py` | FEAT-AW-01 owner；不修改 |
| read-only | FEAT-AW-02 worktree health module | 不调用 create/switch/recover/remove |
| external merge owner | `meta_flow/cli.py` | ST-AW-004；ST-AW-005 如需接线必须串行提交给 owner |
| forbidden | 目标项目文件/link/worktree/ref、artifact main、remote、scheduler/helper enable、`process/quant-lab/**`、sibling content | CR-051 mutation=0 / out-of-scope |

## 4. 门禁与完成定义

- Dev Entry：ST-AW-005 technical note 经 CP5 批准；上游 route/worktree/aggregate contracts稳定；只读 scope 与临时 fixture 确认。
- Dev Exit：manifest schema、mapping/hash/count、readiness、validation/rollback、threshold evaluator 有实现证据；mutation spy 和 before/after snapshot均为0变化。
- Verification：meta-qa 独立复核 denied paths、symlink no-follow、permission failure、portable paths、三阈值和 candidate-only语义。

## 5. 升级/回退条件

出现以下任一项立即停止并把 policy 升为 full LLD：写入 route/manifest runtime truth source；move/copy/delete/link/worktree/ref/commit/push；新增跨 Story mutation contract；创建/启用/调用 sync helper。若只是平台无法可靠读取 permission/symlink 信息，则保持技术说明、输出 MANUAL_REVIEW/BLOCKED，不得用 mutation 探测替代只读预检。

## 6. Gotchas

- “生成迁移脚本但不执行”仍会扩大交付授权和回滚面；本 Story 只生成 manifest/checklist，不生成可执行 mutation script。
- O-AW-03 是治理重访条件，不是 CR-051 的性能验收或自动化开关。
- 验证后迁移步骤属于未来项目级 CR，不能回填为本 Story 已完成的 implementation evidence。
