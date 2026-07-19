---
story_id: "ST-AW-004"
title: "以单写 evidence gate 聚合全部 required legs"
story_slug: "aggregate-evidence-gate"
lld_version: "1.2"
tier: "L"
status: "ready-for-review"
confirmed: false
created_by: "meta-dev-critical"
created_at: "2026-07-18T06:50:00Z"
confirmed_by: ""
confirmed_at: ""
shared_fragments: []
feature_design_refs:
  - "process/docs/features/cr051-aggregate/DESIGN.md"
  - "process/docs/features/cr051-aggregate/TEST-PLAN.md"
  - "process/docs/features/cr051-aggregate/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["single-writer", "correlation", "evidence-dag", "workflow-projection"]
  rationale: "AggregateResult 是 overall 单写边界并控制 CR/state/current 投影，必须冻结 correlation、evidence、并发冲突和 failure route。"
open_items: 0
---

# LLD: ST-AW-004 — 以单写 evidence gate 聚合全部 required legs

> 本 LLD 是 ST-AW-004 的 CP5 正式设计证据，当前 `confirmed=false`。它只定义可实施设计，不授权源码、测试、Git/worktree/ref/remote、commit、push、publish 或 main-sync 变更。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-dev-critical | 初版：冻结 2/2 aggregate、single-writer 与 controlled projection 边界。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP5 R2：关闭 CP5-QA-R1-F03；aggregate 只消费 reread-validated published handles，并冻结无自引用 payload/receipt DAG。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验 design delta：增加 consumer-side canonical target policy 校验、PARTIAL staged projection、幂等 retry 与 exact DAG/dependency 测试。 |

## 0. 上游设计依据（工程依据）

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| CP5 Capsule | `process/context/CP5-CR051-LLD-CONTEXT.yaml` | LLD 批次、禁止实现边界、ST-AW-003/004 contract gate、无 clarification blocker |
| Development Plan | `process/DEVELOPMENT-PLAN.yaml` | W3 条件并行、依赖类型、文件 owner、ST-AW-004 为 CR-051 CLI merge owner |
| CP4 Result | `process/checks/CP4-CR051-STORY-DAG-PARALLEL-SAFETY.result.json` | 5 节点/5 边 DAG、ownership、parallel safety 与 Feature evidence PASS |
| HLD | `process/docs/design/CR051-ARTIFACT-WORKTREE-HLD.md` | 异构 required legs、固定优先级、单写 evidence、受控投影 |
| ADR | `process/docs/design/CR051-ARTIFACT-WORKTREE-ARCHITECTURE-DECISION.md`；ADR-AW-004/005/007 | 独立 leg + aggregate gate、现有 evidence/writer 复用、禁止自引用与隐式 mutation |
| Feature Matrix | `process/docs/design/CR051-FEATURE-DESIGN-MATRIX.md` | FEAT-AW-04 required；ST-AW-004 full-lld；matching 2/2 PASS hard gate |
| Feature Pack | `process/docs/features/cr051-aggregate/{DESIGN,TEST-PLAN,TASKS}.md` | Aggregate schema、4×4 决策表、single-writer、projection、TASK-AW-004-01..05 |
| Story Card | `process/stories/STORY-ST-AW-004-aggregate-evidence-gate.md` | 范围、AC、文件所有权、依赖和 dev/verification gate |
| 上游 Contract | ST-AW-003 `LegResultPayload → LegResultWriteReceipt → PublishedLegResultHandle` contract / `cr051-legs` DESIGN | immutable payload、external receipt、published handle重读、operation/attempt/CR/project/leg correlation |
| CP5 R1 findings | `process/docs/quality/CR051-CP5-INDEPENDENT-REVIEW-FINDINGS.md` / F03 | payload不可依赖自己的append receipt/ref；aggregate只消费通过ref/readback/receipt/digest/key验证的published handles |

## 1. Goal

创建 `meta_flow/workflow/artifact_aggregate.py` 与定向测试，并通过 ST-AW-004 单写的 CLI/controlled-writer 接线，把显式 source/artifact `PublishedLegResultHandle` 重读验证、聚合、持久化和投影拆成可重试的四阶段：只有 matching required source+artifact 两个 published handles 均通过 ref/receipt/digest/key/correlation 校验且 payload 2/2 terminal PASS，才允许逻辑 CR 完成投影；raw/unpublished、non-PASS、PARTIAL、缺失、陈旧、重复或不可信输入均保持非完成且不触发 Git/worktree/manual-sync 操作。

## 2. Requirements / 需求（Functional / Non-Functional）

### 2.1 Functional

- F-01：接受显式 `AggregateRequest` 和恰好两个 `PublishedLegResultHandle`，不接受 raw/unpublished payload，也不按目录 mtime 或“最新文件”猜 current attempt。
- F-02：从每个 handle 的 `result_ref` 重读 immutable payload，对 external receipt、payload/ref/receipt digest、derived single-write key、schema/version、operation/attempt/CR/project、leg kind/mode、required set、唯一性和 target policy 做完整验证。
- F-03：验证失败生成 `BLOCKED(invalid-result-set)` 设计结果；不得把不可信输入降级为业务 FAIL 或忽略。
- F-04：对可信集合按 `BLOCKED > FAIL > IN_PROGRESS > PASS` 计算；四状态 4×4 共 16 组合必须完全确定。
- F-05：`PARTIAL` 只写入 progress/effect；overall 枚举仅 `BLOCKED|FAIL|IN_PROGRESS|PASS`。
- F-06：以两个 validated published payload digests 生成 immutable `AggregateResult`；payload 在写入前确定 digest，不包含自己的 result_ref/write receipt/receipt digest/writer id/written_at；相同输入幂等，冲突输入不 last-write-wins。
- F-07：writer 返回 external `AggregateWriteReceipt` 且 readback 成功后，才允许 projection guard 处理；只有 2/2 PASS 且 aggregate 仍为 current 才调用既有 controlled writer。
- F-08：projection 失败只记录 projection receipt 并可重试，不改写 aggregate，不重跑 leg，不自动 close/rollback/sync。
- F-09：提供显式 CLI/status 入口，输出 input/result/evidence refs 与 next route；dry-run 不持久化 aggregate、不投影。
- F-10：保留 CR-050 legacy paired-default workflow 的既有行为；CR-051 aggregate 只消费 mode-matching、reread-validated published handle contract。

### 2.2 Non-Functional

- NF-01（正确性）：16 个状态组合决策一致率 100%；只有 PASS/PASS 得到 overall PASS。
- NF-02（安全）：aggregate 模块静态 import 与运行调用 Git adapter、worktree executor、manual sync 的数量均为 0。
- NF-03（一致性）：相同 validated handles 的 `aggregate_id` 与 payload digest 一致率 100%；conflicting current selector 更新 100% BLOCKED；payload/receipt/handle任一篡改接受数为0。
- NF-04（可审计）：DAG 精确为 `LegResultPayload → LegResultWriteReceipt → validated PublishedLegResultHandle → immutable AggregateResult → AggregateWriteReceipt → controlled projection`；receipt/refs均为payload外置证据，不得自引用。
- NF-05（性能）：required legs 固定为 2；验证与聚合为 O(N)，N=2；不扫描仓库、不遍历历史 ledger。
- NF-06（兼容）：既有 CR/state/current writer 仍是唯一 truth owner；不新增第二套状态或 ledger。
- NF-07（授权）：CP5 只确认设计；实现和任何运行写入仍由后续 gate/显式授权控制。

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `artifact_aggregate.models`（同文件内类型区） | `AggregateRequest`、validation/status/error/result/external receipt typed models | 不复制 payload/handle owner；只 import ST-AW-003 的稳定协议类型 |
| `artifact_aggregate.validate_leg_set` | 接收恰好2个published handles，从显式refs重读payload并校验 receipt/digest/key/correlation/required-set/target | 纯验证；拒绝raw/unpublished；不执行 Git/worktree |
| `artifact_aggregate.compute_aggregate` | 实现 4×4 precedence、progress/effect 和 next route | 纯函数、无 I/O、输入顺序不影响 digest |
| `artifact_aggregate.persist_aggregate` | append immutable `AggregateResult`、返回external receipt、readback、current-selector CAS 与冲突分类 | receipt不回写payload；复用既有 evidence adapter，不新增 ledger |
| `artifact_aggregate.project_if_eligible` | revalidate current aggregate 后调用 controlled CR/state/current writer | 只接受 persisted/readback PASS receipt；non-PASS 永不调用 writer |
| `artifact_aggregate.coordinate` | 组织 validate→compute→persist→project，生成 typed command result | 每阶段 receipt 可独立重试；不隐藏失败 |
| `meta_flow/cli.py` | CR-051 显式 aggregate/status 参数解析与输出 | ST-AW-004 是 CR-051 唯一 merge owner；CLI 不自行计算状态 |
| `cr_lifecycle.py` / `state/current.py` | 复用既有 controlled projection API | 不改变 legacy 默认行为，不复制 writer |
| `tests/test_artifact_aggregate.py` | 决策表、raw/unpublished拒绝、published handle重读篡改、CAS并发、无自引用、projection、dependency与回归fixture | 使用 fake payload/receipt/handle/evidence/writer ports，无真实 Git/remote |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `meta_flow/workflow/artifact_aggregate.py` | 定义 models、ports、validator、pure aggregate、immutable persistence、single-writer CAS、projection guard 与 coordinator |
| 修改 | `meta_flow/cli.py` | 增加 CR-051 显式 aggregate/status 入口；把 typed request 交给 coordinator；输出 refs/decision/next route |
| 修改 | `meta_flow/workflow/cr_lifecycle.py` | 通过既有 controlled update 边界接收 persisted aggregate PASS ref；non-PASS 明确 hold |
| 修改 | `meta_flow/state/current.py` | 通过现有受控写 API 投影 aggregate ref/decision；保留 owner/version/CAS 语义 |
| 创建 | `tests/test_artifact_aggregate.py` | 建立 16 组合、validation、idempotency、并发、projection、CLI 与 legacy regression 测试 |

明确不修改：`meta_flow/workflow/artifact_leg_lifecycle.py`（ST-AW-003 owner）、`meta_flow/workspace/git_sync.py`、worktree lifecycle、artifact main/manual-sync 模块、`process/quant-lab/**`。删除文件数=0。

## 5. 数据模型与持久化设计

### 5.1 Typed models

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `AggregateStatus` | enum | `BLOCKED|FAIL|IN_PROGRESS|PASS` | 不含 PARTIAL |
| `AggregateRequest.operation_id` | non-empty string | 与 current CR operation 完全匹配 | 不从路径推断 |
| `logical_attempt` | positive integer | 显式选择；旧 attempt 仍可审计但不可投影 current | 不按 mtime 选取 |
| `cr_id` / `project_id` | validated ID | 与 route/current state 匹配 | 错配即 validation BLOCKED |
| `required_legs` | tuple | canonical 固定 `("source","artifact")` | 缺失、重复、unknown 均非法 |
| `published_leg_handles` | mapping leg→`PublishedLegResultHandle` | exactly 2；source/artifact各1；handle含payload/result_ref、payload_digest、external receipt | 不接受raw/unpublished payload或自由文件遍历 |
| `ValidatedPublishedLegResult` | immutable view | reader从result_ref重读payload后，校验ref/payload/receipt/digest/key并保存kind/mode/status/terminal/target proof | 不含 executor handle；不信任调用方内嵌payload |
| `ValidatedLegSet` | tuple + correlation | 两项均为validated published结果且correlation 100%一致 | 输入按leg kind canonical排序 |
| `AggregateResult.aggregate_id` | sha256 string | `sha256(canonical request identity + policy_version + input_digest)` | 相同输入确定 |
| `input_digest` | sha256 string | canonical source/artifact payload digest + validated handle/receipt identity | 输入顺序不影响；raw handle字段不直接授权 |
| `overall` | `AggregateStatus` | 按固定优先级 | validation error 先生成 BLOCKED |
| `terminal` | boolean | 结论是否稳定；不等于成功 | BLOCKED/FAIL 可 terminal |
| `progress` / `effect` | typed summary | 可含 `PARTIAL` | 只描述事实，不驱动 PASS |
| `projection_decision` | enum | `HOLD|ELIGIBLE` | 仅 2/2 terminal PASS 为 ELIGIBLE |
| `AggregateWriteReceipt` | external immutable receipt | aggregate_ref、payload_digest、writer_id、written_at、receipt_digest、readback/CAS result | 不序列化回AggregateResult；readback=false禁止投影 |
| `ProjectionReceipt` | immutable receipt | aggregate ref、writer refs、status/error | 失败可重试，不改 aggregate |

### 5.2 Canonicalization 与 Evidence DAG

```text
LegResultPayload(source)   → LegResultWriteReceipt(source)   ─┐
                                                               ├→ validated PublishedLegResultHandle set
LegResultPayload(artifact) → LegResultWriteReceipt(artifact) ──┘
                                                               ↓
                                                     immutable AggregateResult
                                                               ↓
                                                     AggregateWriteReceipt
                                                               ↓
                                                     controlled projection
```

- validator必须从每个`result_ref`重读payload，重算payload digest、derived single-write key和receipt digest；handle/receipt/ref任一错配均在aggregate计算前BLOCKED。
- canonical JSON 使用 UTF-8、稳定 key 顺序、无浮点时间参与 identity；payload时间若是已冻结leg事实可审计，但aggregate writer/time只存在external receipt，不参与payload identity。
- `AggregateResult`在writer调用前完成canonical digest；payload禁止包含自身aggregate_ref、write/append receipt、receipt_digest、writer_id、written_at或projection receipt。
- `aggregate_id` 包含 validated input digest，因此同一 logical attempt 随显式published handles演进时产生新的 immutable aggregate节点，不覆盖旧节点。
- operation/attempt 的 current aggregate selector 通过 expected-previous-ref CAS 单写更新；CAS 冲突返回 BLOCKED，禁止 last-write-wins。
- 相同 `aggregate_id + payload digest` 的重复 append 返回 `idempotent-existing`；相同 ID 不同 payload 为 corruption/conflict BLOCKED。
- aggregate append、checksum/readback、current-selector CAS完成后才形成external可投影receipt；receipt绝不回写aggregate payload。
- 不把未来Git commit OID或任何publication receipt写入承载该aggregate的payload；如需publication binding，由后置事件追加。

### 5.3 持久化边界

无新增数据库、无新增 ledger。复用现有 result/evidence reader/writer adapter 与 controlled current writer；新增的只是 CR-051 typed payload/receipt/handle consumer schema 和调用约束。raw/unpublished输入、leg handle重读失败、aggregate持久化/readback/CAS失败均保持已有leg事实，projection=0；聚合模块从不回写leg payload/receipt/handle。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 / 对应测试 |
|---|---|---|---|---|
| `validate_published_leg_results(request, handles, reader)` | `AggregateRequest`、恰好2个`PublishedLegResultHandle`、`LegResultReader` | `ValidationOutcome[ValidatedLegSet]` | coordinator/status | 从result_ref重读payload并校验receipt/ref/digest/key；TP-AW04-004..008/015/017..019 |
| `compute_aggregate(validated)` | `ValidatedLegSet` | `AggregateResultDraft` | coordinator/unit | TP-AW04-001..003；纯函数 |
| `blocked_from_validation(request, errors)` | request + typed errors | BLOCKED draft | coordinator | TP-AW04-005..008 |
| `persist_aggregate(result, store, expected_current_ref)` | prewrite-digested immutable result、store port、CAS expectation | external `AggregateWriteReceipt` | coordinator | payload不含自己的ref/receipt；TP-AW04-009..011/019/020 |
| `project_if_eligible(receipt, projector)` | persisted/readback receipt | `ProjectionReceipt` | coordinator | TP-AW04-012..014 |
| `coordinate_aggregate(request, ports, dry_run=False)` | typed request + explicit ports | `AggregateCommandResult` | CLI/workflow | TP-AW04-009/013/016 |
| CLI `meta-flow cr aggregate` | project、CR、attempt、source-handle、artifact-handle、`--dry-run` | structured decision/refs/next route；exit category | operator/Host | TP-AW04-016/017；不接受raw/unpublished，不扫描“latest” |
| existing controlled writer adapter | persisted PASS ref + expected current version | CR/state/current write receipts | projection guard | TP-AW04-012/013；non-PASS不调用 |

所有 port 以 protocol/adapter 注入。`artifact_aggregate.py` 的 import graph 禁止出现 `git_sync`、Git subprocess、project worktree executor 或 manual-sync coordinator。

## 7. 核心处理流程

```mermaid
sequenceDiagram
    participant C as CLI/Workflow
    participant A as Aggregate Coordinator
    participant V as Result Validator
    participant S as Evidence Store
    participant P as Projection Guard
    participant W as Controlled Writers

    C->>A: AggregateRequest + source/artifact published handles
    A->>V: validate handle/receipt/ref/schema/correlation
    V->>S: reread immutable payloads from result_ref
    alt validation invalid
        V-->>A: typed errors
        A->>A: build BLOCKED draft
    else validated
        V-->>A: ValidatedPublishedLegResult set
        A->>A: compute precedence/progress/effect
    end
    alt dry-run
        A-->>C: decision preview; writes=0
    else persist
        A->>S: append immutable aggregate payload + readback + current CAS
        alt persist/readback/CAS failed
            S-->>A: BLOCKED receipt
            A-->>C: HOLD + retry/resolve route
        else persisted current aggregate
            S-->>A: AggregateWriteReceipt
            A->>P: project_if_eligible(receipt)
            alt overall PASS and still current
                P->>W: controlled projection(expected version)
                W-->>P: writer receipts
            else non-PASS or stale
                P-->>A: HOLD; writer calls=0
            end
            A-->>C: AggregateCommandResult + evidence refs
        end
    end
```

失败恢复遵循事实保留：raw/unpublished或reread验证失败不会进入compute；aggregate writer失败不暴露可投影receipt。任何失败都不回滚leg、不删除旧aggregate、不改artifact main。调用方根据next route显式选择refresh-handles、retry-persist、retry-projection、resume-leg、abort-attempt或human-review。

## 8. 技术细节与设计取舍

### 8.1 固定决策表

| source \ artifact | BLOCKED | FAIL | IN_PROGRESS | PASS |
|---|---|---|---|---|
| BLOCKED | BLOCKED | BLOCKED | BLOCKED | BLOCKED |
| FAIL | BLOCKED | FAIL | FAIL | FAIL |
| IN_PROGRESS | BLOCKED | FAIL | IN_PROGRESS | IN_PROGRESS |
| PASS | BLOCKED | FAIL | IN_PROGRESS | PASS |

规则顺序：raw/unpublished或handle/receipt/ref/digest/key/correlation validation error→BLOCKED；两个reread-validated published payload中任一BLOCKED→BLOCKED；否则任一FAIL→FAIL；否则任一missing/non-terminal/IN_PROGRESS→IN_PROGRESS；否则且仅当2/2 terminal PASS→PASS。缺失handle可以在request预检阶段表现为IN_PROGRESS，若request宣称complete required handles却缺失则validation BLOCKED；两者都不可PASS。

### 8.2 terminal、progress 与 effect

- `terminal=false`：至少一个 required leg 尚可产生新 result，overall 通常 IN_PROGRESS/BLOCKED。
- `terminal=true`：当前 attempt 的匹配事实已稳定；BLOCKED/FAIL/PASS 均可 terminal。
- `progress=PARTIAL` 或 `effect=PARTIAL`：至少一条 leg 已产生不可忽略效果但 overall 非 PASS；projection 始终 HOLD。
- aggregate 不生成跨 leg rollback plan；next route 只引用 leg owner 的显式 resume/abort 能力。

### 8.3 Projection 二阶段门

1. 持久化阶段证明 aggregate payload/digest/readback/current-selector CAS。
2. 投影阶段重新检查external aggregate receipt对应current selector、overall PASS、2/2 validated published input identities仍匹配、expected writer version。
3. controlled writer 分别返回 CR/state/current receipts；任一失败记录 PARTIAL projection effect，保持可重试，不把 aggregate 改为 FAIL，也不重跑 Git。
4. 只有全部 required projection receipts 成功，命令结果才报告 projection complete；aggregate PASS 与 projection complete 是不同字段。

### 8.4 兼容性与依赖选择

- 复用 ST-AW-003 `LegResultPayload` / external receipt / `PublishedLegResultHandle` protocol，但通过reader port解耦文件实现；CP5冻结schema/version后才允许W3并行开发。
- 复用既有 CR/state/current controlled writer，新增 mode/payload mapping，不改变 CR-050 的 paired-default 读取与投影。
- CLI 由 ST-AW-004 单写接线；ST-AW-001/002/003/005 不并行修改 `meta_flow/cli.py`。
- 图示选择：使用时序图，因为验证、持久化、CAS、投影和失败重试跨 5 个模块且存在补偿边界。

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 输入安全 | 只接受typed published handles；result refs经adapter allowlist；reread payload并校验receipt/digest/key；不接受raw/unpublished、路径遍历或自由glob | wrong handle/receipt/ref/path/correlation negative fixtures |
| 权限 | aggregate 本身不需要 Git/runtime auth；projection 只使用既有 controlled writer 权限边界 | spy 证明 Git/worktree/manual-sync=0；writer auth regression |
| 完整性 | leg payload/receipt/handle全链路重读、aggregate prewrite digest、external receipt/readback、current-selector CAS | tamper/no-self-ref/readback/CAS conflict tests |
| 最小权限 | aggregate import graph 不含 Git/worktree；non-PASS不调用 writer | static import check + spies |
| 隐私/凭据 | schema 不含 token/credential；authorization只引用脱敏 typed ref | serialization snapshot |
| 时间复杂度 | required legs 固定 2，validator/aggregate O(2) | property/performance sanity test |
| I/O | 每次命令从两个显式result_ref各重读1个leg payload、追加1个aggregate、条件投影；不扫描历史 | reader/store call-count assertions |
| 并发 | deterministic ID + immutable append + expected-current CAS | barrier/concurrent-writer fixtures |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| 16 状态组合 | 两个 matching reread-validated published legs | 枚举 4×4 | 与 §8.1 100%一致；仅 payload PASS/PASS 得 PASS | TP-AW04-001 property/unit |
| 2/2 PASS | 两个validated published handles的payload均terminal PASS | compute+persist+project | ELIGIBLE，controlled writer 可调用一次 | TP-AW04-002/012 |
| PARTIAL | source PASS、artifact FAIL | compute | overall FAIL、effect PARTIAL、projection HOLD | TP-AW04-003 |
| 缺失/非 terminal | handle缺失或published payload为IN_PROGRESS | validate/compute | IN_PROGRESS或BLOCKED，永不PASS | TP-AW04-004 |
| stale attempt | result attempt≠request | validate | BLOCKED invalid-correlation，writer=0 | TP-AW04-005 |
| identity错配 | wrong CR/project/operation/leg | validate | 100%拒绝，writer=0 | TP-AW04-006 |
| required-set异常 | duplicate source、unknown、缺artifact | validate | BLOCKED(required-set) | TP-AW04-007 |
| corruption | payload checksum/readback/schema错误 | validate | BLOCKED(untrusted-result) | TP-AW04-008 |
| aggregate append failure | fake store失败 | coordinate | HOLD，可重试，不投影 | TP-AW04-009 |
| same-input 并发 | 两 writer相同payload | barrier persist | 单个immutable节点/idempotent receipt | TP-AW04-010 |
| conflicting current CAS | 两 writer不同input sets | barrier persist | 一个成功；另一个BLOCKED/refresh，非last-write-wins | TP-AW04-011 |
| projection成功 | persisted current PASS | project | CR/state/current receipts齐全 | TP-AW04-012 |
| projection部分失败 | 某controlled writer timeout | project/retry | aggregate不改、不重跑leg、重试幂等 | TP-AW04-013 |
| dependency boundary | 任意aggregate/status | spy imports/calls | Git/worktree/manual-sync调用=0 | TP-AW04-014 |
| legacy compatibility | CR-050 result fixture | CR-051 validator | 不误消费；旧路径行为不变 | TP-AW04-015 |
| CLI dry-run | 显式2 published handles、`--dry-run` | CLI | 输出决策/refs；store/writer=0 | TP-AW04-016 |
| raw/unpublished 输入 | raw PASS payload、writer失败outcome或缺receipt handle | validate | 100% BLOCKED；compute/store/project调用=0 | TP-AW04-017 |
| published handle tamper | result_ref、payload_digest、writer_id、written_at或receipt_digest逐项篡改 | reread validate | 100%拒绝；受信payload数=0；projection=0 | TP-AW04-018 |
| aggregate无自引用 | matching valid handles | compute writer前payload digest，再persist | payload forbidden self-ref/receipt/writer/time字段=0；digest在writer前稳定 | TP-AW04-019 |
| exact evidence DAG | 2/2 valid PASS handles | validate→compute→persist→project | 顺序恰为payload→external leg receipt→validated handle→immutable aggregate→external aggregate receipt→projection；回写次数=0 | TP-AW04-020 |

测试只使用 typed fixtures、临时 evidence store 与 fake writers；本 Story 实现/验证默认不需要真实 Git/remote。

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-AW-004-01 | 创建 | `meta_flow/workflow/artifact_aggregate.py` | 定义 published-handle consumer protocols/models、canonicalization、validation errors、status/precedence、single-writer keys 与 external receipts | TP-AW04-001..008/017..020 |
| TASK-AW-004-02 | 创建 | `meta_flow/workflow/artifact_aggregate.py` | 实现从result_ref重读payload、receipt/digest/key validation、blocked-from-validation与纯`compute_aggregate` | TP-AW04-001..008/014/015/017/018 |
| TASK-AW-004-03 | 创建 | `meta_flow/workflow/artifact_aggregate.py` | 实现immutable aggregate prewrite digest、append/readback、external receipt、deterministic ID、idempotent duplicate、current-selector CAS与conflict route | TP-AW04-009..011/019/020 |
| TASK-AW-004-04 | 修改 | `artifact_aggregate.py`、`meta_flow/cli.py`、`meta_flow/workflow/cr_lifecycle.py`、`meta_flow/state/current.py` | 实现 projection二阶段门、next route、显式 CLI/status 与 legacy-preserving controlled writer mapping | TP-AW04-012..016 |
| TASK-AW-004-05 | 创建 | `tests/test_artifact_aggregate.py` | 建立16组合、raw/unpublished、handle/receipt tamper、no-self-ref/exact-DAG、concurrency、projection、dependency、CLI与legacy fixtures | TP-AW04-001..020 |

实施严格串行遵循 01→(02,03)→04→05；02/03 可在同一 Story 内分切片，但合并前由 Story owner统一检查 schema。CP5 批次未批准前所有 TASK 仅为计划。

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-AW004-01 | ST-AW-003/004 是否可并行开发 | 推荐：published-result contract在CP5冻结且primary文件分离后并行；备选：全串行 | 推荐已由Plan/CP4固定；CLI只由ST-AW-004写 | 接口、文件owner、测试、Wave | Development Plan、CP4 Result、Feature DESIGN | payload/receipt/handle在CP5后仍需破坏性schema改动 |
| LCQ-AW004-02 | PARTIAL 是否成为 overall 第五状态 | 推荐：仅 progress/effect；备选：overall PARTIAL | 推荐已由 HLD/Matrix 固定；备选禁止 | schema、projection、兼容、测试 | Matrix Gotchas、Feature DESIGN | 不重访；若业务要求则回CP3 |
| LCQ-AW004-03 | 投影失败是否回滚/重跑 leg | 推荐：保留 aggregate与leg事实，只重试 controlled projection；备选：自动回滚/重跑 | 推荐；自动补偿禁止 | evidence DAG、幂等、安全 | ADR/Feature DESIGN | existing writer 无法幂等重试时回设计澄清 |

当前需 Host broker 的未决 clarification：0。

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| R-AW004-01：published-result schema 漂移 | W3并行实现无法编译或误聚合 | CP5冻结payload/receipt/handle protocol/version；consumer contract tests；不改ST-AW-003 primary |
| R-AW004-02：current selector 并发冲突 | stale aggregate 被投影 | expected-current CAS；冲突 BLOCKED 并 refresh explicit refs |
| R-AW004-03：projection 部分成功 | CR/state/current 暂时不一致 | 独立 projection receipts、既有 controlled writer、幂等 retry、aggregate事实不改 |
| R-AW004-04：自引用 evidence | digest不可满足或反复改写 | leg/aggregate payload均先定digest；result_ref/receipt/writer/time外置且不回写；后置publication binding event |
| R-AW004-06：伪published handle绕过writer | raw/unpublished PASS误推进overall | result_ref重读 + receipt/ref/digest/key全校验；raw/unpublished拒绝；exact-DAG test |
| R-AW004-05：legacy regression | CR-050现有流程被CR-051 mode覆盖 | 显式mode dispatch、legacy fixtures、默认行为不改 |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | N/A | 无 OPEN / Spike；上述取舍均由批准基线收敛 | 无 | 无 |

## 13. 回滚与发布策略

- 发布方式：在 CP5批准、ST-AW-003 contract frozen、依赖和文件门满足后，按 TASK切片合并；默认以显式 CR-051 mode/CLI入口启用，不改变 CR-050 legacy默认路径。
- 发布前检查：16组合、invalid set、CAS并发、projection、dependency boundary和legacy regression全部通过；aggregate模块Git/worktree/manual-sync import/call=0。
- 回滚触发条件：non-PASS发生projection、current-selector可被stale写覆盖、相同输入非确定、aggregate需要Git/worktree调用、legacy回归失败或evidence自引用。
- 回滚动作：停止CR-051 aggregate入口与projection接线，回退本Story代码切片；保留append-only leg/aggregate/projection evidence，不删除或改写历史；不执行Git/ref/worktree自动回滚。
- 数据迁移：无数据库迁移。新schema是append-only evidence payload；旧消费者未声明支持时忽略CR-051类型，不做就地重写。
- 恢复：修复设计/实现后使用显式published handles重新validate/aggregate；leg writer失败由ST-AW-003 evidence-only retry，不重跑Git；projection失败只重试projection，不重跑leg。

## 14. Definition of Done（DoD）

- [x] 14 个章节全部填写完成，且无占位符。
- [x] 文件影响范围、接口、数据、错误、测试与 TASK-ID 可直接指导编码。
- [x] 16组合、仅两个validated published payload 2/2 PASS、PARTIAL非overall、single-writer、无自引用exact evidence DAG和projection边界已冻结。
- [x] 实现灰区与取舍记录已覆盖，Host clarification queue item=0。
- [x] OPEN / Spike 已显式清点为无。
- [x] frontmatter `tier=L`、`status=ready-for-review`、`confirmed=false`。
- [x] 文件所有权明确：ST-AW-004 primary/CLI merge owner；ST-AW-003 primary只读。
- [x] 回滚与发布不删除历史 evidence、不执行真实 Git/worktree/ref 自动恢复。
- [x] raw/unpublished payload接受数=0；validator从result_ref重读并逐项校验leg receipt/ref/payload digest/receipt digest/single-write key。
- [x] `AggregateResult`禁止包含自身aggregate_ref、write/append receipt、receipt digest、writer id、written_at；external `AggregateWriteReceipt`不回写payload。
- [x] evidence DAG精确为`LegResultPayload → LegResultWriteReceipt → validated PublishedLegResultHandle → immutable AggregateResult → AggregateWriteReceipt → controlled projection`。
- [ ] Host 收齐 5 个 Story 设计证据并完成 CP5 全量人工确认。
- [ ] CP5批准后再计算 dev_gate；当前不得进入实现。

## 15. CP8 终验 design delta

### 15.1 Consumer-side policy validation

`AggregateRequest` 仅接受 source=`source-default`、artifact=`shared-artifact-project-first`。validator 在重读 payload 并通过 receipt/digest/correlation 后，仍须调用共享纯 policy 独立验证 base/target/active ref；任一漂移返回 `TARGET_POLICY_MISMATCH`，不进入 aggregate/projection。

### 15.2 PARTIAL staged projection

controlled projection 按 state fact → CR ledger/current selector 分阶段写入并返回逐阶段 receipt。状态语义固定为：

- state 写入前失败：`FAILED`；
- state 已成功、后续 ledger/current 失败：`PARTIAL`，保留 state receipt，路由 `retry-controlled-projection`；
- 全部完成：`COMPLETE`。

retry 对已存在且 digest 相同的 state fact 幂等，只补未完成阶段；不得重写 aggregate、重跑 Git leg或把 PARTIAL 当 overall PASS。CLI mode 参数使用 canonical 枚举，不再暴露可漂移默认值。

### 15.3 验证闭环

自动化覆盖 consumer target-policy 拒绝、Git/worktree/leg/manual-sync dependency spy、exact evidence DAG 顺序、state-success/ledger-failure PARTIAL 以及 retry 后 COMPLETE。本 delta 由 `process/design-deltas/ST-AW-004.delta.json` 跟踪并回写 Feature DESIGN 1.2。

## 人工确认区

> 本 lane 不创建或修改人工 checkpoint。Host Orchestrator 收齐 4 份 full LLD、1 份 technical-note、5 个 Story CP5 自动结果及 CP4 摘要后，统一生成 CP5 Decision Brief 并请求用户确认。

| # | CP5 自动检查项 | 当前状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 Story AC | PASS | §2、§10、§14 |
| 2 | 与 HLD/ADR/Matrix一致 | PASS | §0、§8、§12 |
| 3 | 文件影响与 owner明确 | PASS | §3、§4、§11 |
| 4 | 接口/数据/错误契约完整 | PASS | §5、§6、§7 |
| 5 | 测试与 dev_gate可计算 | PASS | §10、§11、§14 |
| 6 | clarification queue收敛 | PASS | §12.1；未决=0 |
| 7 | 人工批次确认 | PENDING | 由 Host 的 CP5 batch checkpoint 统一回填 |

人工确认结果：`pending`；确认人、确认时间、修改意见与风险接受项均由 Host 在批次 checkpoint 回填，不在本文件提前声明。
