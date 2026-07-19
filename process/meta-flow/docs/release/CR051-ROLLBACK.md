---
status: procedural-plan-only
version: "0.5.0-proposed"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# CR-051 Rollback

## 1. 回滚摘要

| 项目 | 内容 |
|---|---|
| 当前是否需要回滚 | 否；未执行真实部署、迁移、链接或仓库 mutation |
| 回滚目标 | 最近经确认的源码 default branch 与对应项目 integration 基线 |
| 回滚范围 | 未来授权后的 source leg、artifact leg、worktree 驻留、路由元数据与软链接 |
| 数据恢复 | 当前 N/A；未来迁移需保留 manifest、前后 OID 与链接目标证据 |
| 不可回滚项 | 当前无；未来外部发布/不可逆写入必须单独评估 |
| 决策人 | 人工 owner；本文件不授权执行 |

## 2. 回滚触发条件

| Trigger ID | 条件 | 证据 | 决策人 |
|---|---|---|---|
| RB-AW-001 | worktree switch intent 无法恢复或驻留分支与预期不一致 | durable intent + worktree status 摘要 | project owner |
| RB-AW-002 | source/artifact CR ID、ref 或完成目标不一致 | 双 leg finish evidence | CR owner |
| RB-AW-003 | 聚合状态与两 leg 最差状态不一致 | immutable aggregate record + selector readback | workflow owner |
| RB-AW-004 | migration preflight 非 READY、manifest 漂移或 denied path 被触及 | manifest digest + preflight result | migration owner |
| RB-AW-005 | 软链接/目录映射与批准计划不一致 | 前后 link/map inventory | project owner |

## 3. 当前执行

当前没有真实 mutation，所以回滚执行为 N/A。不得为了“验证回滚”而创建、切换、合并或删除真实分支/worktree，也不得移动文件或重挂软链接。

## 4. 未来授权后的程序性回滚步骤

| Step | 操作意图 | 前置条件 | 验证 | 风险 |
|---:|---|---|---|---|
| 1 | 停止新的 CR 与同步操作，冻结相关项目 write window | 精确项目与仓库范围已确认 | 无并发 writer | 误冻结 sibling 项目 |
| 2 | 保存 source/artifact leg evidence、manifest、前后 OID 和 routing/link inventory | 证据存储可写且不覆盖旧记录 | digest/readback 一致 | 证据不完整时不得继续 |
| 3 | 将 artifact worktree 恢复到经确认的项目 integration 驻留状态 | worktree clean 或已隔离未提交内容 | branch/ref 与预期一致 | 强制切换会丢失内容，禁止隐式执行 |
| 4 | 恢复该项目批准的 artifact 路由和链接目标 | 已有明确旧目标与恢复授权 | 路由健康检查通过 | 绝对路径/跨设备差异 |
| 5 | 撤销尚未发布的该项目 CR 合流，或创建显式修复变更 | 人工选择非破坏性策略 | owned paths 与基线一致 | 不得覆盖 sibling 项目历史 |
| 6 | 重新执行双 leg 与聚合只读验证 | source/artifact evidence 完整 | 2/2 状态和 selector 一致 | 外部 remote 仍需授权 |

以上仅描述意图，不提供或执行具体 Git/文件系统命令。实际回滚必须在独立 runtime authorization 下由人工确认精确对象。

## 5. 回滚验证

| 验证项 | 方法 | 当前结果 |
|---|---|---|
| source default lifecycle | 只读 ref/OID 与 finish evidence 核对 | NOT-EXECUTED |
| artifact project integration lifecycle | 只读 ref/OID、worktree 与 routing 核对 | NOT-EXECUTED |
| 双 leg 聚合 | fixture/readback/CAS 回归 | PASS_WITH_RISK |
| migration/link recovery | 前后 manifest 与 link inventory | N/A-NOT-AUTHORIZED |
