# PTM Team Backlog

> 版本：v1.0 · 更新：2026-06-08

---

## Active（当前活跃）

| ID | 标题 | Agent | 优先级 | 状态 |
|---|---|---|---|---|
| — | 原子能力框架完善 | ptm-tae | P0 | 🔄 进行中 |
| — | 防火墙配置原子能力补齐 | ptm-tae | P0 | 🔄 进行中 |
| — | Topo 管理（大网+交换机隔离） | ptm-tae | P0 | 🔄 进行中 |

## Candidate（候选，待启动）

| ID | 标题 | Agent | 优先级 | 来源 |
|---|---|---|---|---|
| C-001 | ptm-te Agent 开发 | ptm-te | P0 | Slice 2 |
| C-002 | 用例解析与执行 Skill | ptm-te | P0 | Slice 2 |
| C-003 | 禅道任务集成（领取+回写） | ptm-te | P1 | Slice 2 |
| C-004 | 自动化翻译（执行记录→脚本） | ptm-tae | P1 | Slice 2 |
| C-005 | 回归触发 + 失败自修复 | ptm-tae | P1 | Slice 2 |
| C-006 | tde→te→tae 最小闭环端到端 | 跨 Agent | P0 | Slice 2 |
| C-007 | ptm-tm Agent 开发 | ptm-tm | P1 | Slice 3 |
| C-008 | ptm-tse Agent 开发 | ptm-tse | P1 | Slice 3 |
| C-009 | ptm-qa Agent 开发 | ptm-qa | P2 | Slice 3 |

## Deferred（延后）

| ID | 标题 | 来源 | 跟踪 |
|---|---|---|---|
| D-001 | 断点恢复机制 | CR-011-T-01 | `process/changes/CR-011-FOLLOW-UP-TRACKING-2026-06-02.md` |
| D-002 | 关键词调优 | CR-011-T-02 | 同上 |
| D-003 | Codex 平台整改 | CR-015-T-01 | `process/changes/CR-015-FOLLOW-UP-TRACKING-2026-06-04.md` |

## Spike Candidate（调研候选）

| ID | 标题 | 说明 |
|---|---|---|
| S-001 | atomic-ops aliases 补充（47 ops） | 剩余无歧义 op 按需补充 |
| S-002 | 应用层流量模拟方案调研 | HTTP/FTP/DNS 等 L7 协议模拟 |
| S-003 | WPS/文档在线协作集成 | docx/xlsx ↔ Markdown 双向转换 |

---

*Backlog 随项目进展动态更新。正式启动的候选 Story 通过 CR 流程管理。*
