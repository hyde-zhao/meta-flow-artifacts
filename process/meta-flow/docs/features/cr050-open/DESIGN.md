---
status: draft
version: "1.0"
feature_id: "FEAT-GB-01"
feature_name: "Paired Branch Open"
source_blueprint: "process/docs/design/CR050-GIT-BRANCH-BLUEPRINT.md"
source_hld: "process/docs/design/CR050-GIT-BRANCH-HLD.md"
source_adr: "process/docs/design/CR050-GIT-BRANCH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR050-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-GB-001"]
lld_policy_summary: "1 full-lld"
---

# Feature Design: Paired Branch Open

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline/meta-se | 冻结双仓无写preflight、default refresh、local branch、bootstrap复用和tracking push契约。 |

## 摘要

| 项目 | 内容 |
|---|---|
| Feature目标 | 从project/artifact各自fresh remote default exact OID开启同名CR branch并建立upstream。 |
| 推荐方案 | typed open intent + route discovery + preflight-all + serial paired executor。 |
| 关键取舍 | local准备后失败保留PARTIAL，不自动清理；旧bootstrap不新增隐式remote write。 |
| 下游Story | ST-GB-001 |
| LLD策略 | full-lld：跨仓、remote write、共享writer。 |

## 背景与问题

手工open容易从陈旧main、错误仓、dirty/detached状态创建分支；而CR bootstrap会写artifact，使clean gate和CR-first产生顺序循环。本Feature用intent预授权边界和显式coordinator断环。

## 上游依据与目标/非目标

- 依据：UC-GB-001、REQ-GB-001..004/006、TC-GB-001..003/009..010、ADR-R2-001。
- Goal：全仓无写preflight失败时new refs=0；成功时2/2 base/upstream exact。
- Non-Goal：自动stage/commit、merge、finish、force、跨仓事务。

## Feature边界与现有代码

| 对象 | 负责 | 不负责 | 位置/变化 |
|---|---|---|---|
| route/status | 发现唯一project/artifact root并观察clean/branch/upstream | 写CR阶段 | `meta_flow/workspace/git_sync.py`扩展typed probe |
| open coordinator | intent→local prepare→bootstrap→tracking push | 改变旧bootstrap默认语义 | `meta_flow/workflow/git_branch_lifecycle.py`新建；`cr_lifecycle.py`最小接线 |
| CLI | 显式open/dry-run/output/authz参数 | 隐式调用publish | `meta_flow/cli.py`修改 |

## 推荐方案、数据与接口

`BranchLifecycleIntent(operation=open)`、`RepositoryTarget`、`RefSnapshot`、`BranchOperationPlan/Attempt`由共享模块定义。接口`plan_open(intent, repos)->Plan`不写；`execute_open(plan)->Attempt`只执行allowlisted argv；coordinator在local branches准备后调用既有bootstrap writer，再逐仓push tracking ref并post-check。

## 关键流程与失败

1. 校验CR ID/slug、route、remote/default override和publication authz。
2. 2/2 read-only preflight：git repo、唯一root、clean、attached、branch collision/default symref。
3. actual才fetch/prune和本地default ff-only；从exact remote default创建local CR branch。
4. 调用现有bootstrap writer；逐仓普通`push -u` exact branch并验证OID/upstream。
5. 任一全局preflight失败BLOCKED且mutation=0；写后失败为PARTIAL并保留逐仓事实/resume route。

## 权限、安全、测试与实现顺序

- argv-only、拒绝非法ref/NUL/换行/选项前缀；无`shell=True`。
- publication authz与后续default-write/delete authz互不复用。
- unit覆盖typed validation；bare integration覆盖fresh/dirty/detached/collision/partial/dry-run；command-spy确保commit/force=0。
- 顺序：共享types/probe→planner→coordinator/CLI→bare fixtures。

## Story拆分与下游消费

ST-GB-001消费本DESIGN/TEST-PLAN/TASKS并生成full LLD。story-manager在CP4登记shared file owner；meta-dev在CP5冻结函数/错误码；meta-qa在CP7执行bare fixture但不得冒充真实平台receipt。

## 风险与回退

bootstrap复用若引入隐式remote写，回退为Host显式多步但保持同一intent/result链；不得为复用放松clean gate。partial保留local/remote事实，不自动删分支。
