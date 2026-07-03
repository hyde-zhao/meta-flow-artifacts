---
version: "1.1"
confirmed: true
confirmed_by: "meta-po-delegated"
confirmed_at: "2026-04-23T11:47:36+08:00"
last_updated: "2026-05-15T00:00:00+08:00"
---

# Story Backlog

## 修订记录

| 版本 | 日期 | 修订人 | 关键变更 |
|---|---|---|---|
| 1.0 | 2026-04-23 | meta-se | 基于 3 份 HLD 生成 12 Story / 6 Wave 基线 |
| 1.1 | 2026-05-15 | meta-po | CR-004：追加 Codex agent 生命周期、安装组件、确认协议与交付路由综合修订，不重排既有 Story 编号 |

## Story 列表

| Story ID | 标题 | 来源 HLD | 类型 | 优先级 | Wave | depends_on | 说明 |
|---|---|---|---|---|---|---|---|
| UCD-01 | use-case-discovery 核心增强 | `HLD.md` | feature | P0 | W1 | — | 实现产物分类、头脑风暴、恢复与治理字段输出 |
| UCD-02 | use-case-discovery 模板与引用增强 | `HLD.md` | feature | P0 | W1 | — | 补齐 8 维框架、USE-CASES 模板、治理附录 |
| LLD-01 | lld-designer 六阶段管道与输入输出契约 | `HLD-lld-writing-method.md` | feature | P0 | W1 | — | 将 LLD 方法固化到 Skill 主体 |
| UCD-03 | UCD 上下游契约同步 | `HLD.md` | feature | P1 | W2 | UCD-01,UCD-02 | 同步 meta-pm、requirement-extraction、skills/README |
| RG-01 | meta-po 评审协调核心 | `HLD-review-gate.md` | feature | P0 | W2 | UCD-01,UCD-02 | 实现 gate 调度、聚合、升级仲裁 |
| LLD-02 | STORY-LLD 模板增强（14 章节固定） | `HLD-lld-writing-method.md` | feature | P0 | W2 | LLD-01 | 固化 14 章节、图示规则、OPEN/Spike 与 tier 语义 |
| RG-02 | review 产物协议与静态前置检查 | `HLD-review-gate.md` | infra | P1 | W3 | RG-01 | review findings/summary 模板与轻量检查脚本 |
| LLD-03 | LLD 生产/消费契约对齐 | `HLD-lld-writing-method.md` | feature | P0 | W3 | LLD-01,LLD-02 | 同步 meta-dev、meta-qa、rules 对 LLD 的消费语义 |
| RG-03 | review mode 第一组扩展 | `HLD-review-gate.md` | feature | P1 | W4 | RG-01,RG-02 | 为 meta-pm、meta-se、meta-qa 增加 review mode |
| RG-04 | review mode 第二组扩展 | `HLD-review-gate.md` | feature | P1 | W4 | RG-01,RG-02 | 为 meta-dev、meta-doc 增加 review mode |
| RG-05 | 分派表与灰度落地 | `HLD-review-gate.md` | feature | P1 | W5 | UCD-03,RG-03,RG-04 | 同步 README/rules，完成 HLD/LLD 先试点的 rollout |
| LLD-04 | LLD 方法试点 Story | `HLD-lld-writing-method.md` | design-only | P1 | W6 | LLD-03,RG-05 | 选取 1 个试点 Story 验证新 LLD 方法与 gate 的联动 |

## Wave 分组

> 先前的 4-Wave 方案是方向性草案；按依赖映射收敛后，正式计划细化为 **6 个 Wave**。

| Wave | parallel | Stories | 目标 |
|---|---|---|---|
| W1 | true | UCD-01, UCD-02, LLD-01 | 优先落盘 UCD 核心与 LLD 方法主干 |
| W2 | true | UCD-03, RG-01, LLD-02 | 接上游契约、评审协调核心、模板固定 |
| W3 | true | RG-02, LLD-03 | 补 review 协议和 LLD 消费契约 |
| W4 | true | RG-03, RG-04 | 分两组扩展 review mode |
| W5 | false | RG-05 | 汇总分派表、规则同步与灰度落地 |
| W6 | false | LLD-04 | 用实际 Story 试点验证方法与门禁 |

## 阻塞项

| ID | 状态 | 阻塞项 | 影响 Story | 当前处理 |
|---|---|---|---|---|
| B1 | closed | RG-02 的 review 共享资产归属 | RG-02, RG-05 | 已收敛为 `delivery/skills/review-artifact-protocol/`，模板与校验脚本随 Skill 同树安装 |
| B2 | open | LLD Ready Check 中 `ARCHITECTURE-DECISION.md` 的“条件必需”边界仍需在实现时细化 | LLD-01, LLD-03 | 作为实现期细化决策，不阻断 Story 规划 |
| B3 | open | Phase 6 Checkpoint Handoff 采用文件状态变更还是显式调用 | LLD-03, RG-01 | 进入实现期后在 Agent 协议中定稿 |

## CR-004 追加 Patch Story

| Story ID | 标题 | 类型 | 优先级 | Wave | depends_on | 说明 |
|---|---|---|---|---|---|---|
| CR004-01 | Codex agent 生命周期与最小上下文 handoff | governance | P0 | CR004-W1 | — | 单 meta-po、active_agents registry、同任务复用、checkpoint/handoff 后 close、默认不 fork 全量上下文 |
| CR004-02 | Story Package 合并确认流程 | workflow | P0 | CR004-W1 | CR004-01 | meta-se 产出 Story Package 草案，meta-dev 产出当前 Wave LLD 包，meta-po 发起一次确认后实现 |
| CR004-03 | 安装组件 CLI 与 uv tool 入口 | install | P0 | CR004-W2 | — | `scope-pack install`、`--component rules|agent|full`、user 默认 rules、project 默认 agent、legacy `--content` 兼容 |
| CR004-04 | use-case 头脑风暴与交付出口路由 | product | P1 | CR004-W2 | — | 一次一问、2-3 方案比选、分段确认；production 先读目标 README/docs，无约定则询问 |
| CR004-05 | 平台确认协议与文档/guardrail 回归 | qa-doc | P1 | CR004-W3 | CR004-01,CR004-02,CR004-03,CR004-04 | Claude/Codex 结构化优先，Codex exact 文本兜底；README/USER-MANUAL/rules/guardrail 同步 |

## CR-037 Project Governance Remediation Story

> 本节由 `CP3-CR037-DQ-07` 确认命名策略后追加。正式 Story / evidence 使用 `CR037-Sxx`，原 `CR-A..CR-H` 仅作为计划 alias。CP5 已由用户批准，所有 CR-037 Story 已进入 `dev-ready`；后续仍必须逐 Story 通过 CP6 / CP7，且不授权 runtime、production write、publish、live、凭据读取或 quant-lab 发布库写入。

| Story ID | 标题 | 来源 Story | Feature 设计引用 | lld_policy | 优先级 | Wave | depends_on | Alias | 状态 |
|---|---|---|---|---|---|---|---|---|---|
| CR037-S00 | second-system guardrail | ST-PG-013 | HLD / ADR / Feature Matrix | technical-note | P0 | CR037-W0 | — | SL-PG-00 | verified-with-risk |
| CR037-S01 | current-state schema and budgets | ST-PG-001 | current-state-enforcement | full-lld | P0 | CR037-W1 | CR037-S00 | CR-A-S01 | verified-with-risk |
| CR037-S02 | controlled update API and writer refactor | ST-PG-002 | current-state-enforcement | full-lld | P0 | CR037-W1 | CR037-S01 | CR-A-S02 | verified-with-risk |
| CR037-S03 | agent contract and guardrail sync | ST-PG-003 | current-state-enforcement | technical-note | P0 | CR037-W1 | CR037-S01,CR037-S02 | CR-A-S03 | verified-with-risk |
| CR037-S04 | ledger compaction policy and CLI | ST-PG-012 | ledger-compaction | full-lld | P0.5 | CR037-W1A | CR037-S01 | CR-B-S01 | verified-with-risk |
| CR037-S05 | project scaffold and PROJECT.current | ST-PG-004 | project-state-governance | full-lld | P1 | CR037-W2 | CR037-S01,CR037-S02 | CR-C-S01 | verified-with-risk |
| CR037-S06 | PROJECT-SCALE and roadmap objects | ST-PG-005 | project-state-governance | full-lld | P1 | CR037-W2 | CR037-S05 | CR-C-S02 | verified-with-risk |
| CR037-S07 | feature/capability registry and resolver | ST-PG-006 | capability-feature-registry | full-lld | P1 | CR037-W3 | CR037-S05 | CR-D-S01 | verified-with-risk |
| CR037-S08 | impact surface field split and migration report | ST-PG-007 | impact-surface-normalization | full-lld | P1 | CR037-W3 | CR037-S07 | CR-E-S01 | dev-ready |
| CR037-S09 | roadmap refresh result schema and checker | ST-PG-008 | roadmap-refresh-governance | full-lld | P1 | CR037-W4 | CR037-S06,CR037-S07 | CR-F-S01 | dev-ready |
| CR037-S10 | process-only cascade and Gate Ledger event | ST-PG-009 | roadmap-refresh-governance | full-lld | P1 | CR037-W4 | CR037-S09 | CR-F-S02 | dev-ready |
| CR037-S11 | FU-RF tracking support | ST-PG-010 | roadmap-follow-up-tracking | technical-note | P1 | CR037-W4 | CR037-S09 | CR-G-S01 | dev-ready |
| CR037-S12 | project stale-check | ST-PG-010 | project-stale-check | full-lld | P1 | CR037-W4 | CR037-S09,CR037-S11 | CR-G-S02 | dev-ready |
| CR037-S13 | quant-lab migration dry-run and reports | ST-PG-011 | quant-lab-migration-readiness | full-lld | P2 | CR037-W5 | CR037-S01,CR037-S05,CR037-S07,CR037-S08,CR037-S09,CR037-S10,CR037-S11,CR037-S12 | CR-H-S01 | dev-ready |
