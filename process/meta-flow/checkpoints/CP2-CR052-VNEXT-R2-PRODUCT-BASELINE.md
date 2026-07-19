---
checkpoint_id: "CP2-CR052-VNEXT-R2-PRODUCT-BASELINE"
checkpoint_name: "CR-052 Meta Flow vNext 产品基线"
type: "auto_then_manual"
status: "pending"
owner: "host-orchestrator"
created_at: "2026-07-19T09:18:54Z"
reviewed_by: ""
reviewed_at: ""
auto_check_result: "process/checks/CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2.result.json"
context_ref: "process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml"
decision_brief_profile: "compact"
target:
  phase: "requirement-clarification"
  next_phase_if_approved: "solution-design"
  artifacts:
    - "process/docs/product/USE-CASES.md"
    - "process/docs/product/REQUIREMENTS.md"
    - "process/docs/product/SCENARIOS.yaml"
    - "process/docs/product/TEST-MATRIX.md"
    - "process/docs/product/STORY-MAP.md"
    - "process/docs/product/MVP-SCOPE.md"
    - "process/docs/product/RELEASE-SLICES.md"
    - "process/docs/product/BACKLOG.md"
---

# CP2 — CR-052 Meta Flow vNext 产品基线人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR052-USE-CASE-COMPLETENESS-VNEXT-R2.result.json` | PASS | 0 | 6 个 UC、20 个工程场景、4 个灰区和 1 条用户可见 SGQ 完整 |
| `process/checks/CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2.result.json` | PASS | 0 | 32 条 P0/P1 需求全部有场景与验收追溯，未覆盖数为 0 |
| `process/checks/CP2-CR052-VNEXT-R2-DISCUSSION-CHECKPOINT.json` | PASS | 0 | vNext 候选方向已确认，5 个详细决策仍待本人工门统一批准 |
| 主编排器独立校验 | PASS_WITH_BASELINE_DEBT | 0 | 两份 result-check、YAML/JSON/NDJSON、diff-check 通过；全局 checkpoint ledger 的 CR-051 重复 event_id 是既有历史错误，本轮新增重复数为 0 |

> 自动 `PASS` 仅证明候选产品基线具备审查条件。CP2 未人工批准前，禁止进入 HLD、正式 Story、LLD 或实现；本次 `approve` 也不授权迁移、push 或生产操作。

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 以每项目独立发布库/过程库消除跨项目文档联动，并用四层长期治理、G0/G1/G2 和 Work-scoped 读写检查/token 控制，把 Meta Flow 改造成简单、可靠、低治理成本的 vNext。 |
| 推荐动作 | `approve`：接受 DQ-VNEXT-01..05 推荐方案，进入 CP3/HLD；推荐值以试点数据可调整，不在 CP2 固化不可变实现细节。 |
| approve 后会发生什么 | 调度 meta-se 形成 BLUEPRINT/HLD/ADR，明确仓库绑定、Project/Roadmap/Phase/Work、publisher/CAS、风险路由、scope/token 计量、兼容和迁移架构；之后仍需 CP3、CP5 才能实施。 |
| approve 不授权什么 | 不授权源码立即实现，不授权创建/迁移真实过程仓，不授权远端 commit/push/publication，不授权批量迁移其他项目、历史重写、自动 merge、force/reset/rebase/orphan、凭据、runtime、production 或 trading。 |
| 不确认会阻塞什么 | CR-052 保持 requirement-clarification；不得生成正式 HLD/Story/LLD，不得修改源码或执行迁移。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 capsule；3 个必读真相源，产品增量最多从明确的 allowed/read_if_needed 扩展 |
| 全文档读取扩展 | 5 个 read_if_needed 产品对象；为八份正式基线和跨文档追溯所必需，逐项写入 capsule 与 `READ-EXPANSION-LEDGER.ndjson` |
| 预算偏差 | 事实源总数 12，超过 `max_source_files=8` 四个；未读取完整 CR、STATE、archive、Story、旧 checkpoint 或完整会话，偏差不阻断本次产品基线，但作为 vNext token 设计输入 |

### 产品基线摘要

| 对象 | 数量 | 说明 |
|---|---:|---|
| Use Case | 6 | `UC-VNEXT-001..006` |
| Requirement | 32 | 功能 21 + 约束 6 + NFR 5，均为 P0/P1 |
| Engineering Scenario | 20 | positive 7、negative 4、boundary 4、permission 3、failure-recovery 2 |
| 候选 Story | 6 | `ST-VNEXT-001..006`，只是 CP2 产品候选，不冒充 CP4 Story |
| Release Slice | 4 | A 双库隔离、B 四层/Work/CAS、C G profile/scope/token、D 快照试点 |
| 成功指标 | 12 | 覆盖 route=1、跨项目变化=0、预算、CAS、只读历史和 2×2 试点 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| current state / CR reframe | `STATE.current.json`、vNext summary、`REQUEST.md` | scanned | 5 | 5 | 仓库边界、并发、迁移、预算、试点均需用户确认 |
| meta-pm return | `CR052-CP2-VNEXT-REFRAME-META-PM-RETURN-SUMMARY.md` | scanned | 5 | 5 | DQ-VNEXT-01..05 完整，有推荐、备选和切换条件 |
| 自动预检 | CP1/CP2 VNEXT-R2 result | scanned | 0 | 0 | blockers=0、waivers=0 |
| discussion | VNEXT-R2 log/checkpoint | scanned | 4 | 5 | 4 个灰区映射 5 个 DQ，SGQ 只确认候选方向 |
| 八份产品基线 | `process/docs/product/*` vNext 增量 | scanned | 5 | 5 | In/Out/Deferred、验收、风险与追溯一致 |
| 旧 CP2 | `CP2-CR052-MIGRATION-READINESS-BASELINE.md` | historical | 10 | 0 | 已 changes_requested，只作 v1.1 审计，不进入当前决策 |
| 用户当前指令 | “打 tag，然后按照建议开始实施整改” | scanned | 1 | 0 | 授权启动正式流程和创建本地 tag；不预批准后来形成的 5 个详细 DQ |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | DQ-VNEXT-01/02/03/05 |
| 高风险策略确认 | 1 | DQ-VNEXT-04 的 token/读写/检查硬上限决定流程负担和升级频率 |
| agent 默认处理 | 7 | schema 字段名、CLI 命令、文件名、receipt 编码、检查器内部实现、模板渲染、错误文案由 CP3/LLD 选择 |
| 仅审计记录 | 6 | 本地 tag、基线 OID、路由健康、两个 CP result PASS、历史 checkpoint changes_requested、全局旧 ledger 重复项 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-VNEXT-01 | architecture | 发布库复用现有源码/交付仓，还是另建第三仓或共享发布仓？ | 复用现有仓；每项目只新增独立过程仓，总计恰好 2 个逻辑仓。 | A. 新建专用发布仓；B. 多项目共享发布仓。 | 推荐仓库最少且发布代码/文档同 commit；第三仓同步复杂，共享仓违背隔离目标。 | 决定初始化、权限、迁移和发布证据绑定。 | 只有现有仓无法满足强制合规/权限隔离并重开 CP2/CP3 时切换。 |
| DQ-VNEXT-02 | architecture | process main-only 如何避免多 Agent/设备竞争同一 ref？ | 单写 publisher 串行提交 main，fresh expected-OID CAS；漂移拒绝，不自动 merge。 | A. per-Work branch；B. 多写者自动 merge。 | 推荐状态少、失败清晰；分支恢复治理成本，自动 merge 风险最高。 | 决定并发吞吐、幂等、恢复和 receipt。 | 连续两个试点周期违反吞吐 SLO 且 CAS 为主因时，新 CR 评估 per-Work branch。 |
| DQ-VNEXT-03 | scope | 是否拆分旧 Git 历史或把旧 CP/CR/Story 无损转换？ | 只迁当前快照；旧仓永久只读审计，默认兼容读取在观察期后关闭；不拆史、不转换、双写=0。 | A. 按项目 rewrite 历史；B. 自动转换旧对象。 | 推荐风险最低且回滚直接；拆史改 OID，转换易制造伪语义。 | 决定迁移工具、审计、兼容期和回滚。 | 仅强制审计无法由只读旧仓满足并经独立 CR 批准时做针对性导出。 |
| DQ-VNEXT-04 | implementation | G0/G1 的 read/write/check/token 硬上限取值？ | G0=8/8/3/32k；G1=20/24/8/96k；高风险或超限前进入 G2。 | A. 更紧的 5/5/3/20k 与 10/16/6/40k；B. 只告警不设硬上限。 | 推荐有可验收硬边界并允许试点校准；更紧可能 underfit，只告警不能止住失控。 | 决定上下文、检查颗粒度、升级频率和 token 成本。 | 2×2 试点按 P50/P95 重开 DQ；调整必须版本化，禁止静默放宽。 |
| DQ-VNEXT-05 | scope | 首版试点需要几个项目、几个 Work 周期和多长观察期？ | 2 项目×各至少2周期；meta-flow+1个切换前用户批准项目；各覆盖 G0/G1；观察30天。 | A. 仅 meta-flow；B. 一次迁移全部项目。 | 推荐是验证跨项目隔离的最小样本；单项目证据不足，批量迁移爆炸半径过大。 | 决定验收时间、真实仓授权、回滚和旧仓退役判断。 | 任一跨项目变化/旧仓写入/CAS覆盖/不可回滚即停止并回旧只读基线。 |

#### DQ-VNEXT-01 — 每项目发布库边界

| 字段 | 内容 |
|---|---|
| 决策类型 | `architecture` |
| 待确认问题 | “每项目两个独立归档库”中的发布库是否复用项目现有源码/交付仓？如果再建专用发布归档仓，一个项目会实际变成源码仓+发布仓+过程仓三个仓，增加同步和失败恢复成本。 |
| 推荐方案 | 复用现有项目源码/交付仓作为发布库；每项目只新增一个独立过程仓，逻辑仓库总数恰好为 2。 |
| 备选 A | 新建专用发布归档仓，形成三个仓；仅适用于现有源码仓因合规/权限无法承载发布资产。 |
| 备选 B | 多项目共享发布仓；仓库少，但会重新引入跨项目边界和治理成本。 |
| 优劣分析 | 推荐方案仓库最少、发布代码和文档同 commit 可追溯；代价是需要在现有仓明确公开发布文档边界。三仓隔离更强但跨仓一致性更复杂；共享发布仓与本次隔离目标冲突。 |
| 影响 / 风险 | 决定仓库数量、初始化、发现、权限、迁移和发布证据绑定。 |
| 回退 / 切换条件 | 若 CP3 证明现有仓无法满足强制合规/权限隔离，可重开 CP2/CP3 切换到专用发布仓；不得静默增加第三仓。 |

#### DQ-VNEXT-02 — 过程仓并发模型

| 字段 | 内容 |
|---|---|
| 决策类型 | `architecture` |
| 待确认问题 | 过程仓 main-only 时，不同 Agent/设备即使写不同目录也会竞争同一 Git ref；是否采用单写 publisher 和 expected-OID CAS？ |
| 推荐方案 | 所有 Work 可并行准备内容，但只有一个 process publisher 串行提交 `main`；每次提交绑定 fresh expected OID，漂移时拒绝并重新规划，不自动 merge。 |
| 备选 A | 每 Work 建分支后人工 merge；兼容 Git 习惯，但恢复分支治理和 working-tree 切换成本。 |
| 备选 B | 多写者自动 merge；吞吐高，但冲突、重放和审计复杂，不符合首版简单可靠目标。 |
| 优劣分析 | 推荐方案状态少、失败语义清晰、可用 CAS 验证；代价是发布点串行。分支方案隔离提交但流程重；自动 merge 复杂且风险最高。 |
| 影响 / 风险 | 决定并发吞吐、幂等、恢复、receipt 和跨设备行为。 |
| 回退 / 切换条件 | 连续两个完整试点周期违反已批准吞吐 SLO，且证据证明单写 CAS 是主要瓶颈时，才启动新 CR 评估 per-Work branch；不直接跳到自动 merge。 |

#### DQ-VNEXT-03 — 历史与兼容迁移

| 字段 | 内容 |
|---|---|
| 决策类型 | `scope` |
| 待确认问题 | 是否拆分旧共享过程仓的完整 Git 历史，或把旧 CP/CR/Story 无损转换成四层模型？两者都会显著增加迁移成本和语义失真风险。 |
| 推荐方案 | 只迁当前快照；旧共享过程仓永久保持只读审计源，但默认兼容读取只保留试点观察期，之后仅按显式审计请求访问；不拆史、不做旧对象无损转换、双写数为 0。 |
| 备选 A | 按项目 filter/rewrite 历史；保留项目提交链，但成本高且重写 OID。 |
| 备选 B | 自动转换旧对象；查询统一，但旧 CP/CR 与新 Work 语义不等价，容易制造伪追溯。 |
| 优劣分析 | 推荐方案迁移最简单、回滚最直接、历史原貌不变；代价是审计时要回旧仓。拆史或转换提升表面统一性，但风险和维护成本显著上升。 |
| 影响 / 风险 | 决定迁移工具、存储保留、审计查询、兼容期和回滚。 |
| 回退 / 切换条件 | 只有强制审计/恢复需求无法由只读旧仓满足，并经独立 CR 批准后，才做针对性导出；禁止在本 CR 扩为全量历史重写。 |

#### DQ-VNEXT-04 — G0/G1 资源硬上限

| 字段 | 内容 |
|---|---|
| 决策类型 | `implementation` |
| 待确认问题 | 首版需要可执行的读/写/检查/token 上限，既要防止 token 失控，又不能因过紧频繁误升级。以下数字是 Work 全周期硬上限，不是鼓励消耗到上限。 |
| 推荐方案 | G0：≤8 个读取文件、≤8 个写入路径、≤3 个检查组、≤32,000 总 token；G1：≤20 个读取文件、≤24 个写入路径、≤8 个检查组、≤96,000 总 token；高风险或任一上限无法满足时必须在超限前进入 G2 并给出新预算。 |
| 备选 A | 更紧：G0=5/5/3/20k，G1=10/16/6/40k；成本更低，但首轮 underfit 和频繁 G2 的概率更高。 |
| 备选 B | 不设硬上限，只告警；实现简单，但不能解决用户提出的 token 失控。 |
| 优劣分析 | 推荐方案给出可验收硬边界，并允许试点校准；备选 A 更省但可能增加路由开销；备选 B 无法形成可靠控制。 |
| 影响 / 风险 | 决定默认上下文、检查颗粒度、升级频率和试点成功率；必须明确 token 计量不可用时的 telemetry-unavailable 状态。 |
| 回退 / 切换条件 | 2×2 试点若出现系统性 underfit/overfit，以实际 P50/P95 使用量重开该 DQ；任何调整必须版本化，禁止静默放宽。 |

#### DQ-VNEXT-05 — 试点范围与观察期

| 字段 | 内容 |
|---|---|
| 决策类型 | `scope` |
| 待确认问题 | 单项目试点无法证明跨项目隔离；一次批量迁移又会扩大爆炸半径。首版试点应覆盖多少项目和周期？ |
| 推荐方案 | 2 个项目 × 每项目至少 2 个完整 Work 周期；第一项目为 meta-flow，第二项目在切换前由用户明确批准；每项目至少覆盖 1 个 G0 和 1 个 G1。旧仓只读观察期 30 天，试点全部通过后才允许另行批准批量迁移。 |
| 备选 A | 仅 meta-flow 单项目试点；更快，但无法验证跨项目变化数=0。 |
| 备选 B | 一次迁移所有项目；样本多，但不可控且与 ptm-team/ptm-atomic 当前排除边界冲突。 |
| 优劣分析 | 推荐方案以最小双项目样本验证核心隔离目标，仍保持可回滚；单项目证据不足，批量迁移风险过高。 |
| 影响 / 风险 | 决定验收时间、真实仓授权、回滚演练和旧仓退役判断。 |
| 回退 / 切换条件 | 任一项目出现跨项目变化、旧仓写入、CAS 覆盖或不可回滚，立即停止试点并回到旧只读基线；第二项目必须在 CP5/迁移授权前明确。 |

### 推荐决策总览

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 DQ-VNEXT-01..05 全部推荐方案并进入 CP3/HLD。 |
| 备选方案 | 可用 `修改: DQ-VNEXT-xx ...` 对单项采用备选或调整数值；也可 `reject` 保留整改前 tag 和 legacy 状态。 |
| 影响维度 | 用户价值、实现复杂度、token 成本、可靠性、迁移回滚、平台兼容和长期维护。 |
| 风险与回退 | CP3/CP5 仍是硬门；任何真实迁移和远端 publication 另需逐项目、逐操作授权。 |
| 用户需决策事项 | DQ-VNEXT-01、DQ-VNEXT-02、DQ-VNEXT-03、DQ-VNEXT-04、DQ-VNEXT-05。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| process 路由健康 | PASS | `meta-flow workspace check` | `process_link_health=ok`；artifact dirty 属当前 CR 审计写入，不代表迁移完成 |
| CP1 vNext 场景完整 | PASS | CP1 VNEXT-R2 result | blockers=0 |
| CP2 vNext 自动预检 | PASS | CP2 VNEXT-R2 result | blockers=0、waivers=0 |
| 场景讨论证据 | PASS | VNEXT-R2 discussion log/checkpoint | 4 个 SGA、1 个 SGQ |
| 旧 CP2 已失效 | PASS | 旧 checkpoint `status=changes_requested` | 未复用旧 Migration Readiness PASS |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 用户真实目标是否准确 | 待审查 | `REQUEST.md`、UC-VNEXT | 双库隔离、长期治理、流程/token 减负 |
| 2 | P0/P1 场景与需求是否完整 | 待审查 | CP1/CP2 result、TEST-MATRIX | 未覆盖数=0 |
| 3 | In/Out/Deferred 是否合理 | 待审查 | MVP-SCOPE、BACKLOG | 不拆史、不无损转换、不自动 merge、不批量迁移 |
| 4 | DQ-VNEXT-01..05 是否接受 | 待审查 | 本 Decision Brief | 可逐项修改 |
| 5 | approve 不授权项是否清楚 | 待审查 | 本 Decision Brief | 真实迁移、push、runtime 等仍禁止 |
| 6 | token 扩读是否透明 | 待审查 | capsule、READ-EXPANSION ledger | 5 次扩读均有原因 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 产品基线人工确认 | 待审查 | 本文件人工审查结果 | 需 `approved` 才可进入 CP3 |
| 5 个 DQ 全部有结论 | 待审查 | DQ-VNEXT-01..05 | 修改项需先回写产品基线 |
| 不授权边界保持 | 待审查 | CR/state/policy refs | approve 不授予真实操作 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| vNext 产品基线 | `process/docs/product/*` 八份对象 | 待审查 | 增量更新，旧 ID 保留并状态化 |
| CP1 自动结果 | `process/checks/CP1-CR052-USE-CASE-COMPLETENESS-VNEXT-R2.result.json` | PASS | schema check OK |
| CP2 自动结果 | `process/checks/CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2.result.json` | PASS | schema check OK |
| CP2 context | `process/context/CP2-CR052-VNEXT-REFRAME-CONTEXT.yaml` | PASS | minimal + 5 次受控扩读 |
| meta-pm return | `process/handoffs/CR052-CP2-VNEXT-REFRAME-META-PM-RETURN-SUMMARY.md` | PASS | 单写边界符合 |

## 人工审查结果

- 结论：`pending`
- 审查人：
- 审查时间：
- 接受的决策：
- 修改意见：
- 风险接受项：
- 不授权项：真实迁移、远端 push/publication、历史重写、自动 merge、批量项目迁移、force/reset/rebase/orphan、credentials/runtime/production/publish/trading。
