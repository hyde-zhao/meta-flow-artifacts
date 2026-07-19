---
checkpoint_id: "CP3-CR051-ARTIFACT-WORKTREE-HLD-REVIEW"
checkpoint_name: "CR-051 Project-first Artifact Worktree Architecture Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-18T04:27:02Z"
reviewed_by: "user"
reviewed_at: "2026-07-18T05:46:40Z"
approval_source: "user-current-instruction-after-review-revision"
auto_check_result: "process/checks/CP3-CR051-HLD-CONSISTENCY-R2.result.json"
context_ref: "process/context/CP3-CR051.context.json"
human_context_ref: "process/context/CP3-CR051-DESIGN-CONTEXT.yaml"
discussion_checkpoint_ref: "process/checks/CP3-CR051-DISCUSSION-CHECKPOINT.json"
decision_brief_profile: "full"
---

# CP3 — CR-051 Project-first Artifact Worktree 架构评审

## 自动预检摘要

| 预检 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| CP3 HLD consistency | PASS | 0 | 17/17 项通过，0 FAIL，0 WAIVED。 |
| Architecture Gray Areas | PASS | 0 | 4 项完成分类：3 个 `decision-item`、1 个 `resolved-by-user`。 |
| CP3-DC-01 消费 | PASS | 0 | HLD、ADR、Domain Map、Dependency Map、失败路径、NFR 和 fixture 策略均明确消费。 |
| 需求 / 场景追溯 | PASS | 0 | 27/27 REQ-AW 与 15/15 TC-AW 有架构和验证落点。 |
| 当前 machine result 预算 | PASS | 0 | 7,753 / 8,192 bytes；重复解释保留在人类摘要。repo-wide doctor 仍把两个已 superseded 的 CP2 R1/R2 result 按“active CR 文件名”误判为 default-required，属于既有分类器债务，不是本门默认输入。 |
| 权限 / 范围 | PASS | 0 | 未实现源码、未拆正式 Story、未执行真实 Git/worktree/link/ref/remote/main-sync mutation。 |
| 评审修订 R2 | PASS | 0 | O-AW-01 空间估算、O-AW-02 durable store、O-AW-03 manual-sync 运维成本均已转为可验证契约；R2 仍为 17/17 PASS。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 冻结 CR-051 的 project-first artifact Git/worktree 架构，使每个项目拥有长期 integration worktree、每个 CR 使用短期项目分支，并让非原子 branch switch、异构双 leg、聚合完成和人工 main/integration 同步都可判定、可审计、可恢复。 |
| 推荐动作 | `approve`：接受 `CP3-CR051-DQ-01..03` 的推荐方案。HLD 已完成 17 项自动一致性检查且无 blocker；推荐方案在不扩大真实运行权限的前提下关闭了最关键的文件安全、证据真实性和远端竞态风险。 |
| approve 后会发生什么 | 架构与七项 ADR 被视为已确认；流程进入 `story-planning`，由 `meta-se` 生成 FEATURE-DESIGN-MATRIX、必要的 Feature DESIGN/TEST-PLAN/TASKS、正式 Story/Development Plan，执行 CP4 自动预检，并在全量设计证据准备后停在 CP5 人工门。 |
| approve 不授权什么 | 不授权源码或测试实现、真实文件迁移、软链接变更、真实 Git worktree/branch/ref/remote mutation、真实 shared main↔integration 同步、commit/push/publish、force/history rewrite、凭据、runtime、SaaS、production write、trading 或 sibling project 修改。 |
| 不确认会阻塞什么 | CR-051 保持在 `solution-design/CP3`；不得进入正式 Feature/Story planning、CP4、LLD 或实现。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR051-DESIGN-CONTEXT.yaml`；machine context=`process/context/CP3-CR051.context.json`。 |
| capsule 状态 | `ready-for-human-gate`。 |
| read_profile | `compact`。 |
| 默认读取策略 | 先读 capsule、HLD、ADR 和 CP3 result；Blueprint/Domain/Dependency/Discussion Checkpoint 为允许读取，产品基线全文只在追溯审计时按需展开。 |
| 全文档读取扩展 | 1 次：`RE-20260718T042656Z0000-620c98a6`，reason=`deep_review`，覆盖 CP3 Architecture Gray Areas discussion；机器 result 已引用并通过校验。 |
| 缺失 / waived 理由 | 无缺失、无 waiver。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue / prior gate | `STATE.current.json`、CP2 R3 checkpoint、`GATE-LEDGER.ndjson` | scanned | 1 | 0 | CP2 R3 已 approved；`CP3-DC-01` 作为强制架构约束承接，不重开 CP2。 |
| 委托 Agent 交还摘要 | `process/handoffs/CR051-CP3-META-SE-RETURN-SUMMARY.md` | scanned | 6 | 3 | DQ-01..03 已批准；O-AW-01..03 均为 non-blocking 且有 CP5/follow-up 路由。 |
| 自动预检结果 | `process/checks/CP3-CR051-HLD-CONSISTENCY-R2.result.json` | scanned | 3 | 3 | 17 PASS、0 blocker、0 waiver；三项 architecture DQ 已按用户指令批准。 |
| discussion log / checkpoint | `CP3-CR051-HLD-DISCUSSION-LOG.md`、`CP3-CR051-DISCUSSION-CHECKPOINT.json` | scanned | 4 | 3 | AGA-01..03 为 `decision-item`；AGA-04 已由 CP2 用户决策解决。 |
| 下游正式产物 | Blueprint、Domain Map、Dependency Map、HLD、Architecture Decision | scanned | 7 ADR | 3 | 七项 ADR 中 ADR-001/004/005/006 受三项 DQ 约束；其余承接 CP2 但随整体 HLD 一并确认。 |
| 用户显式输入 | 当前对话、CP2 R3 approval event | scanned | 1 | 1 | 用户要求解决 worktree switch 非原子性并推进到下一人工门；该问题被 DQ-01 完整承接，未冒充 CP3 已批准。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 3 | `CP3-CR051-DQ-01..03` 已由用户接受推荐方案，状态为 approved。 |
| 高风险策略确认 | 2 | DQ-01 涉及用户文件安全与恢复；DQ-03 涉及远端 ref 竞态和 shared main 边界，均保持 fail closed。 |
| agent 默认处理 | 3 | O-AW-01 estimator 与 O-AW-02 durable store 进入 FEAT-AW-02/CP5 fixture；O-AW-03 仅按量化阈值建立 follow-up CR candidate。 |
| 仅审计记录 | 8 | CP2 已批准产品边界、17 项自动检查、三次受限中断与一次恢复调度、read expansion、无真实 mutation、CR-050 适用性覆盖、repo-wide Artifact Doctor 对 superseded CP2 result 的既有分类器债务。 |

### 已批准人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP3-CR051-DQ-01 | architecture | Git worktree branch switch 非原子时，如何避免进程异常后误报 `active-cr` / `idle-integration` 或破坏用户文件？ | 在目标 worktree 外写 durable intent；依次执行 identity/lock/clean/no-Git-op/ref+OID/permission/space precheck、一次 switch、fresh post-observation；仅在 clean、无 Git op、original integration OID 未漂移且权限/空间复检通过时条件式回 integration，否则保留 worktree/CR branch 并 `RECOVERY_REQUIRED/BLOCKED`。 | 禁用自动 switch，只输出人工操作与 observation-based 检查/恢复步骤；拒绝“仅看退出码”及自动 reset/clean/stash/force/delete。 | 推荐方案保留自动化旅程且可恢复、可审计，代价是 journal/lock/fault fixture 较复杂；备选实现简单、权限最小，但日常 begin/finish 退化为人工操作。 | 高：文件完整性、恢复幂等、磁盘/权限故障、进程中断；错误实现可能覆盖用户数据或制造假 PASS。 | journal 无法可靠持久化、CP5 无法证明恢复幂等，或平台无法提供可靠 fresh observation 时，切到“禁用自动 switch”并回退 Feature Design；不得降级为破坏性恢复。 |
| CP3-CR051-DQ-02 | architecture | worktree operation、source/artifact leg、aggregate 与 publication receipt 如何避免多写竞争、陈旧 attempt 聚合和 OID 自引用？ | 使用 shared envelope + typed append-only evidence DAG：lifecycle writer 单写 journal，各 leg executor 单写 leg result，aggregate coordinator 单写 aggregate；以 CR ID/logical attempt/required-leg set/stable evidence ref 关联，artifact commit OID 由更晚的 binding event 关联。 | A：每个 Feature 独立 schema；B：后续接入外部不可变账本；拒绝单一可变 lifecycle state。 | 推荐方案统一 correlation、便于离线复算且不引入新服务，代价是 schema/owner 契约必须冻结；A 局部简单但易漂移；B 审计强但增加依赖、迁移和运行授权。 | 高：证据真实性、aggregate 完成门、并发单写、审计可追溯；错误聚合会把 partial/blocked CR 宣称完成。 | Feature 真正独立发布并拆 HLD 时可切 A；外部账本能提供同等单写、稳定 ID、离线 fixture 后可另 CR 切 B；CP5 无法证明无环/单写时回退 CP3。 |
| CP3-CR051-DQ-03 | architecture | integration 首建和 CR 外人工 main↔integration 同步如何处理远端竞态、保护策略和权限边界？ | 使用 project-scoped lock + fresh remote observation + exact ordinary ref update + fresh postcheck。integration 仅缺失时从 fresh `origin/main` exact OID create-only；同步显式选择方向、默认无 active artifact CR、绑定 target/expected OID 的独立授权，冲突保留两端并 `BLOCKED`。 | 后续 forge API / PR / merge-queue adapter；拒绝 reset/rebase/force 对齐及在单 CR finish 中隐式同步 main。 | 推荐方案无新平台依赖、竞态可判定且不扩大单 CR 权限，代价是人工同步有运维成本且远端保护策略可能拒绝；forge 备选适合强保护分支但需凭据、平台契约和独立高风险 CR。 | 高：远端 ref 覆盖、shared main 安全、多项目并发、权限与运维；不确定时必须零强制写入。 | 远端强制 PR/merge queue 或拒绝 ordinary exact update 时阻断 native path，另开 forge-adapter CR；不得绕过保护策略。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 DQ-01 的可恢复非原子 switch、DQ-02 的 append-only 单写证据 DAG、DQ-03 的 exact-OID create-only/manual-sync 边界。 |
| 备选方案 | DQ-01 可禁用自动 switch；DQ-02 可在独立发布或外部账本条件成熟后切换；DQ-03 可在远端强制审批时另立 forge adapter CR。 |
| 影响维度 | 用户文件安全、日常 worktree 体验、Git 并发与远端保护、证据真实性、双 leg 完成判定、实现复杂度、可测试性、维护成本和后续逐项目迁移。 |
| 优劣分析 | 推荐组合最大化复用既有 typed native-Git 契约并保持 fail closed；代价是状态机、schema 和 fault fixture 较密。备选均保留清晰切换条件，不把破坏性命令当作可用回退。 |
| 风险与回退 | 三项均不接受“状态未知仍继续”；无法证明时回退到人工/只读路径或重开 CP3/后续 CR，不扩大运行权限。 |
| 用户需决策事项 | `CP3-CR051-DQ-01`、`CP3-CR051-DQ-02`、`CP3-CR051-DQ-03`。 |

### CP3 设计评审摘要

| 维度 | 结论 |
|---|---|
| 候选架构适用条件 | 比较了三种真实方案：扩展现有 typed lifecycle（推荐）、独立 artifact-worktree 服务、外部 forge/脚本编排。推荐方案最适合当前单仓代码和本地可验证边界；独立服务仅在 artifact 有独立团队/SLA 时切换；forge 方案仅在远端强制 PR/queue 时另立 CR。 |
| 优化项 | 复用 CR-050 的 argv-only、exact OID、typed authz、fresh observation、append-only attempt、dry-run；新增 project-first routing、可恢复 worktree、异构 leg 与纯聚合 gate。 |
| 牺牲项 | 引入 durable journal、project lock、typed evidence DAG 和 fault-injection fixture；shared main 同步仍需人工独立动作。 |
| 影响面 | Project Context、Worktree Controller、Leg Executors、Aggregate Gate、Evidence Writer、Migration/Handoff 六个边界；五个 Feature 均需后续 implementation-design。 |
| Use Case → Architecture Traceability | UC-AW-002→bootstrap/worktree；UC-AW-003→失败事实保留与 aggregate BLOCKED；UC-AW-004→matching required-leg 全 PASS 后才整体 PASS。 |
| 关键场景模拟 | integration 首建、source PASS + artifact switch 失败、两腿 matching PASS、active CR 下 manual-sync、空间估算失败、重复 resume 均有预期终态。 |
| 未决风险 | DQ-01..03 已批准；O-AW-01/02 是 CP5 non-blocking 验证义务，无法证明时禁用 auto switch；O-AW-03 仅保留人工同步成本观察。 |
| discussion 证据 | `process/discussions/CP3-CR051-HLD-DISCUSSION-LOG.md`；结构化恢复点=`process/checks/CP3-CR051-DISCUSSION-CHECKPOINT.json`。 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP2 R3 已批准 | PASS | `process/checkpoints/CP2-CR051-PROJECT-FIRST-WORKTREE-BASELINE-R3.md`、对应 gate approval event。 |
| CP3-DC-01 已形成强制设计约束 | PASS | CP2 R3 checkpoint、`process/checks/CP3-CR051-ARCHITECTURE-INPUTS.json`。 |
| meta-se 阶段委托已真实执行并交还 | PASS | handoff、dispatch ledger、return summary；正式设计/result 已落盘。 |
| CP3 context capsule 可用 | PASS | `process/context/CP3-CR051-DESIGN-CONTEXT.yaml`、`process/context/CP3-CR051.context.json`。 |

## Checklist

| # | 检查项 | 结果 | 证据 |
|---:|---|---|---|
| 1 | 27/27 REQ-AW 与 15/15 TC-AW 有架构落点 | PASS | HLD §8、CP3 result CP3-01/13/14。 |
| 2 | 至少两个真实候选架构及适用性矩阵 | PASS | HLD §3/§3.1，实际为 A/B/C 三方案。 |
| 3 | Blueprint/Domain/Dependency 内部一致 | PASS | CP3 result CP3-02/04/16。 |
| 4 | CP3-DC-01 覆盖 precheck、durable intent、fresh observation、恢复和禁止项 | PASS | HLD §5.1、ADR-AW-001、Domain Map §3。 |
| 5 | source-default / artifact-integration 异构 leg 不回退到 artifact main | PASS | ADR-AW-003、HLD §5.3。 |
| 6 | aggregate 仅全必需 leg PASS，PARTIAL 非终态 | PASS | ADR-AW-004、HLD §5.3/§7。 |
| 7 | integration create-only 与 manual-sync 竞态 fail closed | PASS | ADR-AW-005/006。 |
| 8 | schema/ledger 单写与 OID 自引用规避明确 | PASS | Domain Map §8、ADR-AW-004。 |
| 9 | NFR 可量化、失败路径可验证 | PASS | HLD §9、CP3 result CP3-07/08/09。 |
| 10 | HLD 拆分判定与下游 Feature Design 触发明确 | PASS | HLD §9.1/§10、CP3 result CP3-17。 |
| 11 | Architecture Gray Areas 完成分类 | PASS | 3 decision-item、1 resolved-by-user、0 blocker。 |
| 12 | 未执行 Story、实现或真实 Git mutation | PASS | HLD/ADR 边界、handoff return、只读状态检查。 |

## Exit Criteria

| 条目 | 当前结果 |
|---|---|
| CP3 自动结果 `PASS` 且 blocker/waiver 为 0 | PASS |
| 三项架构决策获用户确认 | PASS；用户当前指令批准 DQ-01..03 推荐方案。 |
| 七项 ADR 已确认 | PASS；ADR-AW-001..007 已转 approved/confirmed。 |
| 允许进入 story-planning / CP4 | PASS；仅允许设计规划，不授权实现或真实 mutation。 |
| 实现与真实运行授权 | NOT AUTHORIZED；不由 CP3 approve 改变 |

## Deliverables

| 交付物 | 路径 | 状态 |
|---|---|---|
| Blueprint | `process/docs/design/CR051-ARTIFACT-WORKTREE-BLUEPRINT.md` | PASS |
| Domain Map | `process/docs/design/CR051-ARTIFACT-WORKTREE-DOMAIN-MAP.md` | PASS |
| Dependency Map | `process/docs/design/CR051-ARTIFACT-WORKTREE-DEPENDENCY-MAP.md` | PASS |
| HLD | `process/docs/design/CR051-ARTIFACT-WORKTREE-HLD.md` | confirmed |
| Architecture Decisions | `process/docs/design/CR051-ARTIFACT-WORKTREE-ARCHITECTURE-DECISION.md` | approved |
| Machine result / summary | `process/checks/CP3-CR051-HLD-CONSISTENCY-R2.result.json` / `.summary.md` | PASS |
| CP3 context | `process/context/CP3-CR051.context.json` / `CP3-CR051-DESIGN-CONTEXT.yaml` | PASS |

## 人工审查结果

- 结论：`approved`
- 审查人：用户
- 审查时间：2026-07-18T05:46:40Z
- 修改意见：批准前必须按评审意见固化 O-AW-01 空间估算可靠性与 bounded 512MiB fallback、O-AW-02 durable intent store 故障契约、O-AW-03 manual-sync 运维成本重访阈值；上述修订已在五份设计产物和 CP3 R2 自动结果中完成。
- 风险接受项：无默认风险接受；三项为 architecture decision，所有失败路径继续 fail closed。
- 授权说明：`approve` 只批准架构并允许进入设计规划，不授权任何真实实现、迁移、Git/remote/main-sync 或发布操作。
