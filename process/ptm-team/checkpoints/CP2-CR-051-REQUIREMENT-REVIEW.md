# CP2 - 需求/场景/范围基线确认（CR-051）

> 人工检查点 | 结果: **APPROVED**
> 确认日期: 2026-09-01 | 确认方式: 用户会话指令确认
> 确认依据: 用户消息二原文 "先拉取远端代码，并创建本地分支 zhaohaibo，再落成正式的 CR，并开始实施这个 CR" —— 该指令包含对需求基线的推进授权；需求基线内容（复盘 8 条改进 + 候选打印硬性要求）由用户消息一提供并已在会话中达成共识。

## Entry Criteria

- [x] CP1 自检 PASS（`process/checks/CP1-CR-051-REQUIREMENT-CHECK.md`）
- [x] CR-051 已 bootstrap，`STATE.current.json.active_change=CR-051`
- [x] 需求基线冻结（IMP-051-01~08 + 候选打印硬性要求）

## 需求基线确认清单

| # | 确认项 | 结果 | 说明 |
|---|--------|------|------|
| 1 | 候选打印硬性要求（P0） | ✅ 通过 | 人工门禁发起时打印候选测试因子 + 候选原子操作 id 全文，逐项附 decision |
| 2 | IMP-051-01~08 改进清单 | ✅ 通过 | 8 条改进与复盘四维度对应，无遗漏项 |
| 3 | Story 拆解（STORY-051-01~07） | ✅ 通过 | 与改进清单一一映射，无孤儿 Story |
| 4 | 冲突预检（CR-049 并行） | ✅ 通过 | 影响面不重叠，可并行 |
| 5 | 授权边界与不授权项 | ✅ 通过 | approve 不授权 ptm-tae 开发、不授权 runtime/生产写入 |

## 人工审查结果

**审批者摘要**：本 CR 服务 ptm-tde agent/skills 复盘整改，核心交付物为门禁候选清单打印协议（P0）与引用链/计数/台账标准化。approve 后 Host Orchestrator 将按 STORY-051-01（P0）→ 051-02/03/04（P1）→ 051-05/06/07（P2）顺序实施，不涉及任何运行时、凭据或生产写入。

**决策分层**：

| 层级 | 决策项 | 处理 |
|------|--------|------|
| 必须用户决策 | 候选打印协议内容与展示粒度 | 本门禁确认 |
| 高风险策略确认 | 无 | — |
| agent 默认处理 | ref-integrity 校验实现细节、计数断言形式 | 实施时自决 |
| 仅审计记录 | GATE-5-Delivery-Exit 出处核实 | 遗留 OPEN，不阻塞 |

**Decision Collection Coverage**：已扫描来源（复盘原文、gate-spec.md、agents/ptm-tde.md、8 个 skill、CLAUDE.md 7 条规则对照），纳入待决策项 0 项（改进清单已全部与用户确认，无未决范围问题）；遗留 OPEN 1 项（GATE-5 文件出处，审计记录类，不阻塞）。

## Exit Criteria

- [x] 需求/场景/范围基线确认通过
- [x] 可进入 story-execution 实施（本 CR 无设计阶段，改进直接落 agent/skill 契约）

## Deliverables

- 本检查点文件（`process/checkpoints/CP2-CR-051-REQUIREMENT-REVIEW.md`）
- CP1 自检（`process/checks/CP1-CR-051-REQUIREMENT-CHECK.md`）
- CR-051.md（需求基线 + 五维度影响分析 + Story 拆解 + 冲突预检）
