# CP4 自动预检 — Story DAG / 并行安全（CR-038）

> 本文件为 CP4 自动预检结果（machine checkpoint），非人工门禁。摘要、风险与开放项由 host-orchestrator 汇入 CP5 Decision Brief。

- 检查时间：2026-08-15
- 检查对象：`process/DEVELOPMENT-PLAN-CR-038.yaml`（status=active，13 Story / 4 Wave）
- 真相源：`process/DEVELOPMENT-PLAN-CR-038.yaml` + `docs/design/FEATURE-DESIGN-MATRIX-CR-038.md`

## Entry Criteria

| 条件 | 结果 |
|---|---|
| `docs/design/HLD-CR-038.md` confirmed=true（CP3 approve） | PASS |
| FEATURE-DESIGN-MATRIX 已生成（active） | PASS |
| 13 张 Story 卡片已生成（status=lld-ready） | PASS |
| 3 个 required Feature 已产出 DESIGN/TEST-PLAN/TASKS | PASS |

## Checklist

| 检查项 | 结果 | 证据 |
|---|---|---|
| DAG 无环（depends_on 无循环） | PASS | 脚本 DFS 校验，13 节点 0 环 |
| Wave 依赖方向（依赖落在更早 Wave） | PASS | 所有 depends_on 指向更小 wave 序号 |
| 文件所有权无未串行化冲突 | PASS | pool_merge.py(S02→S13)、physical_pool.py(S03→S13)、exporter.py(S06→S13) 均通过 S13 硬依赖串行 |
| 并行 Story 输出文件不冲突 | PASS | 无同一 Wave 内两 Story 写同一文件 |
| feature_design_refs 覆盖全部 Story | PASS | 3 required Feature 全部引用；waived E4/E5 显式空 ref + N/A 理由 |
| lld_policy 覆盖全部 Story 且合规 | PASS | full-lld×5 / technical-note×5 / waived×3；无 batch-lld |
| 高风险标记禁用 batch-lld | PASS | security/credential（S03 cipher 密文）用 full-lld，无 batch-lld |
| 并行策略声明 | PASS | max_parallel_lld=3 / dev=3 / qa=2 |

## lld_policy 分布

| required_level | 数量 | Story |
|---|---|---|
| full-lld | 5 | 01 / 03 / 05 / 06 / 13 |
| technical-note | 5 | 02 / 04 / 07 / 10 / 11 |
| waived | 3 | 08 / 09 / 12 |
| batch-lld | 0 | —（无低风险同质共享实现面 Story 群） |

## 共享文件串行化明细

| 文件 | 写入方 | 串行方式 |
|---|---|---|
| skills/topo-planning/src/pool_merge.py | S02(W1) → S13(W3) | S13 depends_on S02（hard） |
| skills/topo-config/src/physical_pool.py | S03(W1) → S13(W3) | S13 depends_on S03（hard） |
| skills/topo-planning/src/exporter.py | S06(W2) → S13(W3) | S13 depends_on S06（hard） |

## 风险与开放项（汇入 CP5）

| 项 | 说明 |
|---|---|
| Q-038-001 | ptm-atomic PPPoE op 是否已暴露（RA-038-001），S05 full-lld 三选一闭环，不阻塞 DAG |
| Q-038-002 | H3C 真机 PPPoE 命令与参考命令族差异，dry-run diff + 真机独立授权核对 |
| 无新增用户决策项 | 实现层决策 FD-CR038-01~04 已确定性落地于 Feature DESIGN |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| DAG 无环 + 无文件冲突 + lld_policy 合规 | PASS |
| 不阻塞全量设计证据写作（CP5 前） | 是 |

## Deliverables

- `process/DEVELOPMENT-PLAN-CR-038.yaml`（active）
- `docs/design/FEATURE-DESIGN-MATRIX-CR-038.md`（active）
- `process/stories/STORY-038-01.md` ~ `STORY-038-13.md`（lld-ready）
- `docs/features/cr038-sw-mapping/{DESIGN,TEST-PLAN,TASKS}.md`
- `docs/features/cr038-pppoe-config/{DESIGN,TEST-PLAN,TASKS}.md`
- `docs/features/cr038-loopback-envfile/{DESIGN,TEST-PLAN,TASKS}.md`

## 结论

**CP4 自动预检 PASS**。13 Story / 4 Wave 依赖无环、文件所有权串行化、lld_policy 合规。交还 host-orchestrator 计算覆盖全部 13 个目标 Story 的 LLD 证据队列（full-lld×5 分轮 + technical-note×5 卡片内补齐 + waived×3 理由核验）。
