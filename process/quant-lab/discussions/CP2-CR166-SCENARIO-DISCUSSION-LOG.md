# CP2 CR166 Scenario Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-07-13 | host-orchestrator-inline | 将用户两轮审核与本轮推进授权固化为 CR166 Scenario Gray Areas 证据；不替代正式 CP2 approval。 |

## 讨论范围

- 目标：界定 C2 Walk-forward/OOS producer foundation 的产品范围、fixture/static 边界、P0/P1 分层、future extension 和 Stage claim ceiling。
- 事实基线：现有 `WalkForwardValidationPlan` 与 gate consumer 存在，但无通用 fold-level producer、时间/泄漏/lineage producer validation。
- 当前权限：仅允许本地产品/过程产物；不允许源码实现、真实数据或 runtime/external 操作。

## Scenario Confirmation Interactions

| SGQ | 用户审核意见 | 归一化理解 | 状态 |
|---|---|---|---|
| SGQ-CR166-001 | C2 应明确为 Stage 3 前置桥接，不是 Stage 3 本身。 | Stage 2 已完成且保持 complete；CR166 只交付 fixture/static foundation；真实数据与 Stage 3 独立授权。 | confirmed |
| SGQ-CR166-002 | C2 envelope 必须预留 C3/C4 扩展点。 | 使用稳定 header + versioned typed component registry；当前 C3/C4 calculators=0，unknown component no-PASS。 | confirmed |
| SGQ-CR166-003 | daily/ML 必须，event 可能过度设计。 | daily multifactor 与 ML purged-embargo compatibility 为 P0；event 为 P1 applicability，语义未冻结时明确 N/A。 | confirmed |
| SGQ-CR166-004 | fail-closed 可分层，避免 runtime resolver/hash 边界阻塞。 | 未授权 ref 的深度 resolver 集成保持 P1，但零解引用 guard 与 canonical determinism 是 foundation 的 P0 验收。 | confirmed |
| SGQ-CR166-005 | repository suite 需区分 CR166 回归与 CR165 基线。 | CR166 新增代码路径引入失败=0；触及 CR165 已重基线路径时 CP7 逐项说明触发与归因。 | confirmed |
| SGQ-CR166-006 | CP8 必须声明 Stage 2/3 状态。 | Stage2 complete=true；Stage3 started=false；real OOS evidence available=false。 | confirmed |

## 认知盲区与处理

| 盲区 | 状态 | 处理 |
|---|---|---|
| event-time 与 calendar-time fold 语义未冻结 | OPEN-CP3 | CP3 applicability decision；可 N/A，不建空壳。 |
| purge/embargo 最低线需与 label horizon/policy 对齐 | OPEN-CP3 | HLD/ADR 冻结 policy mapping 与边界 decision table。 |
| C3/C4 共享输入合同是否同一 CR | DEFERRED | 未来 CP3 决定共享 input-contract wave；C3/C4 计算仍独立验证。 |

## 结论

SGQ 已满足 use-case-discovery 的用户可见交互要求；四项 CP2 整体决策仍须在 `process/checkpoints/CP2-CR166-WALK-FORWARD-OOS-EVIDENCE-SCOPE.md` 正式批准。
