---
cr_id: "CR-050"
checkpoint: "CP2"
status: "r2-ready-for-human-gate"
mode: "user-freeform-plus-inline-analysis"
owner: "host-orchestrator"
created_at: "2026-07-15"
---

# CR-050 CP2 Scenario Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline fallback | 形成 open→publish→external-merge→finish 产品讨论与五项 CP2 决策。 |
| 1.1 | 2026-07-16 | host-orchestrator inline fallback | 用户指出现有功能缺少 merge，并批准独立两仓 fast-forward-only merge 推荐边界；重开 SGA-GB-03，新增 SGA-GB-06，回退 CP2 R2。 |

## 用户原始场景确认

用户原话：

> 帮我实现git分支管理功能，开启cr时从远端主分支拉取最新代码，创建cr分支，然后提交推送到远程分支，然后将远程分支合并到后将分支删除掉。你可以分析一下直接使用git命令还是安装gb命令

复述结论：核心用户旅程已经确认；其中“合并到后”解释为外部合并完成后再清理，而不是默认授权 Meta Flow 自动 merge。该解释与工具选型、双仓边界、squash/rebase 证明和 commit 策略一起进入 CP2，不作为静默假设。

## R2 用户范围修订

在 CP3 等待期间，用户进一步确认“当前 Git 生命周期缺少合并分支操作”，询问实现后能否支持 publish 后合并，并对推荐方案回复 `approve`。Host Orchestrator 将该回复解释为批准把以下产品边界提交 CP2 R2，而不是批准已经过时的 CP3 v1.0：

1. 新增独立显式 `merge` 操作；`publish` 和 `finish` 不隐式 merge。
2. 两仓先完成全部 preflight，再按 artifact→project 更新 remote default。
3. 只允许 fast-forward-only；禁止 merge commit、rebase、force 和自动冲突解决。
4. 真实 default-branch write 需要操作级独立授权，并服从 branch protection。
5. 任一仓失败时整体 `PARTIAL/BLOCKED`，保留两仓 CR branch，禁止 `finish`；不自动回滚已成功的 default update。

该修订改变了已批准 CP2-DQ-03，因此按 CR 产品基线规则回到 CP2 R2；不新建 CR-051。

## Scenario Gray Areas

| ID | 问题 | 推荐 | 备选 | 影响 | 状态 |
|---|---|---|---|---|---|
| SGA-GB-01 | 原生 Git、`gb` 还是 Git Town | 复用原生 Git subprocess service | 安装 Git Town；或保持手工 Git | 依赖、跨平台、同步/force 策略、测试 | decision-item |
| SGA-GB-02 | 只管理源码仓还是 project/artifact 成对管理 | 两仓同名 branch | 只管源码仓；或 artifact 始终 main | 跨设备恢复、partial failure、发布一致性 | decision-item |
| SGA-GB-03 | 是否提供 merge，还是验证外部 merge 后 cleanup | 独立显式 fast-forward-only merge；不由publish/finish隐式触发 | 保持verify-only；forge API merge | branch protection、review、权限、回滚 | reopened / CP2-R2-DQ-01 |
| SGA-GB-04 | squash/rebase 如何证明已合并 | non-ancestor fail closed | patch-id 猜测；forge receipt adapter | 误删风险、平台依赖 | decision-item |
| SGA-GB-05 | publish 是否自动 stage/commit | 只推送 committed refs | `git add -A` + 自动 commit；显式 path allowlist commit | secret/无关文件、易用性、审计 | decision-item |
| SGA-GB-06 | 两仓 merge 顺序、授权与 partial | preflight-all；artifact→project；default-write独立授权；partial保留分支 | project→artifact；只管源码仓；forge adapter | 跨仓真相、保护策略、恢复与删除安全 | decision-item / CP2-R2-DQ-02..04 |

## 证据化工具评估

| 候选 | 观察 | 结论 |
|---|---|---|
| 原生 Git | 项目已有 `meta_flow.workspace.git_sync`，无 Python runtime dependency；Git 2.43.0 可用 | 主选；扩展现有 service 与 fixture |
| `gb` | 本机无 `gb`；名称可指 shell alias `git branch` 或无关 Go build tool | 不具备唯一、可移植产品契约 |
| Git Town | 官方提供 `hack/sync/propose/ship`，`ship` 可 merge/delete，但官方也建议通常通过 forge UI/merge queue；部分 sync 策略会 safe force-push | 未来 stacked/offline adapter 候选，不作为 MVP 必需依赖 |

## 用户可见场景确认证据

| Question ID | 用户回答 | 复述确认 | 状态 |
|---|---|---|---|
| SGQ-GB-001 | 用户自由表达了 open → commit/push → merged → delete 的完整旅程 | 核心旅程 confirmed；五项 destructive/architecture 细节等待 CP2 | confirmed |
| SGQ-GB-002 | 用户询问是否应实现 publish 后 merge，并回复 `approve` | 显式 ff-only merge进入产品R2；执行授权仍独立，旧CP3不再可批 | confirmed-for-CP2-R2 |

## Deferred Ideas

- DEF-GB-001：forge receipt adapter for squash/rebase merge。
- DEF-GB-002：Git Town/stacked branch adapter。
- DEF-GB-003：自动 stage/commit planner。

## 不授权范围

- CP2 前的源码实现、commit、真实远端 push/delete。
- 隐式 merge、merge commit、forge API/credential、force-push/force-delete、reset/rebase/history rewrite、自动冲突解决或 branch-protection 绕过。
- CP2 R2 approval 不授权真实 default-branch write；需在实现门后按具体仓/ref/OID单独授权。
- `process/quant-lab/**` 与 prelink backup。
