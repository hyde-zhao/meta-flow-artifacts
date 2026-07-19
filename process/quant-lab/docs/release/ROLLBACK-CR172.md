---
status: ready-for-human-gate
version: "1.0"
release_artifact_profile: full
release_decision: READY_WITH_RISK
rollback_scope: repository-local-only
created_at: "2026-07-19T01:47:00+08:00"
---

# CR-172 PATH-I Rollback

## 1. 回滚摘要

| 项目 | 内容 |
|---|---|
| 回滚目标 | CR-172 交付前的最后一个人工批准 repository snapshot |
| 回滚范围 | PATH-I 四个 repository contracts、相关 tests/fixture，以及 CR172 专属质量/发布文档 |
| 是否涉及数据恢复 | no |
| 是否存在不可回滚项 | no（因为真实操作=0） |
| deployment | not-authorized / not-executed |
| 决策人 | human + Host Orchestrator |

本方案只描述 repository-local 合同/代码回退，不涉及数据湖、NAS、执行机、runtime、signal、trading 或生产状态恢复。

## 2. 回滚触发条件

| Trigger ID | 条件 | 监控 / 证据 | 决策人 |
|---|---|---|---|
| RB-T01 | 15/27/11 semantic mismatch 非零 | S05 integrated QAC | human/Host |
| RB-T02 | 六动作 permission union 或真实授权非零 | authorization regression | human/Host |
| RB-T03 | artifact seal/verifier/selection 链回归 | S02-S04 combined suite | human/Host |
| RB-T04 | 九项 false claim 任一被错误提升 | release/claim review | human/Host |
| RB-T05 | 两仓 delivery snapshot 不可重现 | workspace git-status / paired delivery evidence | human/Host |

## 3. 回滚步骤

| Step | 操作 | 前置条件 | 验证 | 风险 |
|---:|---|---|---|---|
| 1 | 停止合并、交付和任何 activation 讨论 | 发现触发条件 | deploy/publish/runtime counters 仍为 0 | 无运行态影响 |
| 2 | 确认批准前 snapshot/commit 与 CR-owned 文件清单 | source/artifact 两仓均可识别 | snapshot refs 可解析 | 不得覆盖用户无关改动 |
| 3 | 由有权限的人对 CR-owned repository 变更执行受控 revert | 明确本地写权限；不含 remote push | diff 只覆盖 CR-owned surface | 共享文件冲突时停止并人工处理 |
| 4 | 重跑最小 Story 回归及 combined 154 suite | source rollback 完成 | tests、semantic trace、zero-op 均回到目标基线 | fixture 不能证明 runtime |
| 5 | artifact repo 同步回滚 CP8/过程证据或保留 rejected audit 记录 | Host 决定审计保留策略 | source/artifact refs 一致 | 不删除历史 finding/decision |

上述步骤是发布后可执行方案说明，不构成本轮执行授权；本轮没有执行 revert、commit 或 push。

## 4. 回滚验证

| 验证项 | 方法 | 预期 |
|---|---|---|
| repository contracts | S01-S05 relevant tests | target snapshot tests PASS |
| trace | 15/27/11 semantic probe | mismatch=0 |
| zero operation | authorization/claim oracle | 0/6 authorized/executed；external ops=0 |
| state/config/data | N/A | 本 CR 无迁移、无真实写入 |
| paired delivery | workspace git-status + approved refs | source/artifact 对齐 |

## 5. 不可回滚项

| 对象 | 是否存在 | 原因 | 处理 |
|---|---|---|---|
| production data/runtime state | no | 未授权且未执行 | N/A |
| NAS/execution cache | no | 未授权且未执行 | N/A |
| signal/trade/deploy state | no | 未授权且未执行 | N/A |
| Git remote history | no new action | remote write=0 | 若未来 push，另行制定 remote rollback |
