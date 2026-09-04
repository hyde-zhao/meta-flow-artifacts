---
doc_type: follow_up_tracking
cr_id: CR-038
created_at: "2026-08-15"
owner: host-orchestrator
status: open
parent_cr: CR-037
release_decision: READY_WITH_RISK
---

# CR-038 Follow-Up Tracking 台账

> CP8 后续候选台账。所有候选默认 `candidate`（不占执行锁），只有用户决定推进某项时才创建正式 CR。
> 状态取值：`candidate` / `active` / `blocked` / `spike_candidate` / `converted-to-spike` / `closed` / `cancelled` / `superseded`。

## 候选清单（8 项）

| 候选 ID | 类型 | 状态 | 来源风险 | 摘要 | 触发条件 |
|---|---|---|:---:|---|---|
| CR-038-FU-RF009 | scope 回写 | candidate | CR038-R-RF009 | R-F-009 `device_groups.sw` vs 实现 `switch` 块命名/字段集语义差 | 用户决定统一命名口径，走 CR 回写 REQUIREMENTS-CR-038.md 原文或 S08 卡片措辞 |
| CR-038-FU-OPEN0602 | 后续 Story/CR | candidate | CR038-R-OPEN0602 | 环回完整 ip_plan 端到端受 IP_PLAN_SW_VLAN_MISSING 限制 | 需 vlan_assign/IPAM 补 SW VLAN 分配时转正式 CR |
| CR-038-FU-S04 | 真机确认 | candidate | CR038-R-S04 | virtual-template IP 复用 ip_pool_gateway + bind_interface 未指定物理接口 | 真机下发授权后，确认物理接口/网关取值 |
| CR-038-FU-PPPOEOP | 外部依赖 | superseded | CR038-R-PPPOEOP | ptm-atomic PPPoE op 缺失，S05 当前 atomic-skip | 已关闭：ptm-atomic 已实现 PPPoE op（用户确认 2026-08-17），S05 atomic-skip 可由后续 CR 切回原子化 |
| CR-038-FU-S01 | 设计重访 | candidate | CR038-R-S01 | explicit_sw_reserved 互斥可能过度保守（ADR-CR038-01 备选） | 发现实际拓扑需放宽时评估切备选 |
| CR-038-FU-DRYRUN | 设计回修 | candidate | CR038-R-DRYRUN | 领域事实更正（用户 2026-08-17）：仅子类型为 PPPoE 的 H3C 交换机支持 PPPoE；Ruijie/Huawei 及其他 H3C 不支持 → commands.py 三厂商 PPPoE 命令族过度设计 | 走 CR 回修 R-F-003：收窄为 H3C PPPoE 子类型，删 ruijie/huawei pppoe 块 + switch_configurator 支持列表 |
| CR-038-FU-S13 | schema 复核 | candidate | CR038-R-S13 | devices.yaml `instances` schema 固化待复核（LCQ-13-01） | 集成期 / 下一 CR 复核 schema 与 trex_instances 一致 |
| CR-038-FU-RETURN-SCHEMA | 工具归一化 | cancelled | meta-dev 观察 | meta-flow v0.4.0 return-check status 枚举收紧，现有 20 个 CP6 return packet 用 `ready-for-verification` | 已关闭：meta-flow 后续升级，暂不处理（用户确认 2026-08-17） |
| CR-038-FU-DOC-CASES-PATH | 文档措辞 | candidate | meta-doc P2 | `执行指导.md` §5.2 表 `[1]` 行仍写 `cases/upload/*.md`（CR-033 前旧入口），与 README §2 已改的 `cases/<协议族>/.../` 跨文件不一致 | 统一 ptm-te 全链路 cases 路径措辞时处理 |

## 不授权项（runtime_authorization，非台账 candidate）

> 以下为独立运行授权决策项（CP8-DQ-038-RUNTIME），不进入 follow-up candidate，需未来单独授权 + 真实设备/凭据。

| # | 事项 | 关联 Story | 决策类型 |
|---|---|---|---|
| 1 | H3C telnet 真机下发 | S03/S04 | runtime_authorization |
| 2 | trex 真机发流（loss=0 最终判定） | S10/S13 | runtime_authorization |
| 3 | `--execute` 真机执行 | S07（DQ-038-02） | runtime_authorization |
| 4 | DUT PPPoE Client 真机拨号 | S05 | runtime_authorization |

## 关闭条件

- 所有 candidate 由用户决定推进（转正式 CR）或关闭（closed/cancelled/superseded）。
- 本台账不阻塞 CR-038 关闭；CR-038 关闭后本台账持续有效，供后续 CR 启动冲突预检消费。
