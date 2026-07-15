---
discussion_id: "CP3-CR047-HLD-DISCUSSION"
change_id: "CR-047"
phase: "solution-design"
status: "r2-ready-for-host-broker"
owner: "host-orchestrator-inline / meta-se"
created_at: "2026-07-13T04:09:26Z"
question_broker: "host-orchestrator"
---

# CR-047 CP3 HLD Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-14 | host-orchestrator-inline / meta-se | 收录独立评审与用户 changes_requested：动态 Doctor 双基线、对象身份制 CR-046 firewall、叠加式 verdict ceiling；形成 CP3 R2 DQ-01..06。 |
| 1.0 | 2026-07-13 | host-orchestrator-inline / meta-se | 记录 CP2 已批准策略、Architecture Gray Areas、advisor inline synthesis 与 CP3 待决策项。 |

## 交互与证据边界

用户已批准 CP2-DQ-01..04，并明确要求“不使用子agent，继续推进”。本阶段使用 user-approved inline fallback，不声称子 Agent、custom profile/model 或独立 runtime 已执行。R1 独立评审发现 fixed count、CR-046 原件防火墙和证据上限仍不够可执行；用户于 2026-07-14 授权 CP3 R2 精确化，方案 A 与 CP2 范围不变。

## 已继承的 CP2 决策

| 决策 | 已批准结论 | CP3 架构落实 |
|---|---|---|
| CP2-DQ-01 | tracked `delivery/rules/AGENTS.md` canonical；根 wrapper generated | ADR-WT-004 |
| CP2-DQ-02 | Doctor 仅 blocker 非零；WARN 计数披露 | ADR-WT-003 |
| CP2-DQ-03 | 历史用 archive/cold 或 summary/index/hash/correction，不改写 | ADR-WT-003 |
| CP2-DQ-04 | tracked/package cache block；pure ignored cache warn/preflight | ADR-WT-004 |

## Architecture Gray Areas

| ID | Advisor 输入 | 推荐 | 可执行备选 | 当前状态 |
|---|---|---|---|---|
| AGA-WT-01 | 多 source owner 与跨源关系 | source-owned truth graph + read-only consistency check | 新总状态；仅修 Markdown parser | recommended-pending-CP3 / CP3-DQ-01 |
| AGA-WT-02 | artifact internal docs 与根路径兼容 | `process/docs/**` only；根只保留 public docs | ignored symlink；迁回源码仓 | recommended-pending-CP3 / CP3-DQ-02 |
| AGA-WT-03 | 固定 21 已漂移到 22；活动预算、closed history 与 integrity | `B0_pre` 历史锚 + 动态 `B0_cp7`/delta；active strict | 固定 observed count；全局阈值；原位截断（禁止） | revised-pending-CP3 / CP3-DQ-03 |
| AGA-WT-04 | guardrail/Ruff/pytest/installer 的组合边界 | 扩展现有 checker/preflight，组合层只编排 | 新 release orchestrator；手工命令 | recommended-pending-CP3 / CP3-DQ-04 |
| AGA-WT-05 | CR-046/047 同目录，原则性“不改写”无法机器证明 | 对象身份 manifest；CP6 pre-implementation/CP6/CP7 hash 双验 | 人工检查；prefix-only allowlist（禁止） | added-pending-CP3 / CP3-DQ-05 |
| AGA-WT-06 | inline fallback 与五项 CR-046 OPEN 风险限制证明层级 | CP7/CP8 上限独立叠加 | CP7 前取得真实独立证据；否则阻断 READY 诉求 | added-pending-CP3 / CP3-DQ-06 |

## Advisor table-first inline synthesis

| Lane | 关键判断 | 进入 HLD 的约束 |
|---|---|---|
| product | 一个跨设备 truth/quality release objective | 一个 CP3/HLD，7 Story 后拆 |
| architecture | owner 单写、projection 单向、关系只读校验 | 禁止 State/CURRENT 双向同步 |
| quality | warning 不能掩盖 blocker，历史不能为绿灯被改写 | unknown current/package/security fail-closed |
| documentation | artifact internal docs + project-relative refs | 不新增本机兼容副本；README 命令非交互完整 |

## 场景推演摘要

| 场景 | 设计判定 | 说明 |
|---|---|---|
| stale closed CR | PASS | closed/missing active ref 非零退出，保留稳定状态 |
| clean clone link | PASS | relative route、canonical copy=1；断链 BLOCKED |
| legacy history budget | PASS | original immutable；cold summary/index/hash/correction |
| root rule + ignored cache | PASS | tracked rule/installer plan；ignored warn，package cache block |
| full quality gate | PASS | truth→Doctor→guardrail→Ruff→pytest→3 dry-run |
| unavailable receipt | PASS | release ceiling 保持 READY_WITH_RISK |
| Doctor count drift | PASS | `B0_pre=21`、review=22；CP7 动态采集并解释 delta |
| protected original mutation | PASS | object identity + CP6/CP7 hash；变更即阻断并拆子 CR |

以上是架构设计模拟，不是 CP7 运行证据。

## 待 Host Orchestrator 收集的 CP3 决策

1. `CP3-DQ-01`：接受 source-owned truth graph，还是引入新总状态/只修 legacy parser。
2. `CP3-DQ-02`：接受 artifact `process/docs/**` 唯一 canonical，还是创建兼容视图/迁回源码仓。
3. `CP3-DQ-03`：接受 `B0_pre/B0_cp7` 动态 Doctor 与 append-only history，还是固定计数/放宽阈值。
4. `CP3-DQ-04`：接受扩展现有 checker/preflight，还是新建 release orchestrator/维持手工组合。
5. `CP3-DQ-05`：接受对象身份制 protected-original firewall、CP6/CP7 双验和越界子 CR 路由，还是仅人工审查。
6. `CP3-DQ-06`：接受 inline/独立证据与继承风险独立叠加的 CP7/CP8 结论上限，还是在 CP7 前要求真实独立证据。

## 当前结论

- Blueprint/Domain Map/Dependency Map/HLD/ADR：v1.1 R2 草案完成；HLD 预算不超过 20 KiB。
- 自动架构预检：待 Host 写入并校验 CP3 R2 result。
- 产品/设计 blocker：0；人工决策：6。
- formal CP3 approval：未记录。
