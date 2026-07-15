---
checkpoint_id: "CP3-CR047-HLD-REVIEW"
checkpoint_name: "CR-047 Workflow Truth Architecture Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-13T04:45:04Z"
review_round: 2
reviewed_by: "user"
reviewed_at: "2026-07-14T15:29:31Z"
auto_check_result: "process/checks/CP3-CR047-HLD-CONSISTENCY-R2.result.json"
supersedes_auto_check_result: "process/checks/CP3-CR047-HLD-CONSISTENCY.result.json"
context_ref: "process/context/CP3-CR047-DESIGN-CONTEXT.yaml"
decision_brief_profile: "compact"
target:
  phase: "solution-design"
  artifacts:
    - "process/docs/design/CR047-WORKFLOW-TRUTH-BLUEPRINT.md"
    - "process/docs/design/CR047-WORKFLOW-TRUTH-DOMAIN-MAP.md"
    - "process/docs/design/CR047-WORKFLOW-TRUTH-DEPENDENCY-MAP.md"
    - "process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md"
    - "process/docs/design/CR047-WORKFLOW-TRUTH-ARCHITECTURE-DECISION.md"
---

# CP3 CR-047 Workflow Truth Architecture Review

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR047-HLD-CONSISTENCY-R2.result.json` | PASS | 0 | check_attempt=2；8 项通过；显式 supersedes R1；6 项待人工决策 |
| `process/checks/CP3-CR047-DISCUSSION-CHECKPOINT.json` | r2-ready-for-host-broker | 0 | AGA-WT-01..06 已形成推荐、备选、影响和切换条件 |
| `process/context/CP3-CR047-DESIGN-CONTEXT.yaml` | ready-for-human-gate | 0 | compact capsule；11 个 read-expansion refs 已登记 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 冻结 CR-047 的 source-owned workflow truth、artifact internal docs 路由、severity/lifecycle-aware Doctor、clean-clone preflight 与 7-Story 下游边界。 |
| 推荐动作 | `approve`：接受 CP3-DQ-01..06 与 ADR-WT-001..006，进入 Story planning、Feature design、DAG/文件 owner 和 CP4/CP5 准备。 |
| approve 后会发生什么 | Host 继续使用用户批准的 inline fallback，拆分 5 个 Feature/7 个 Story、建立 3 Waves、Feature Design Matrix、Development Plan、CP4 自动预检和 CP5 全量设计证据 Decision Brief；CP5 前不实现代码。 |
| approve 不授权什么 | 代码实现、历史原件改写、runtime、credentials、SaaS、production write、publish、trading、repository commit/push、prelink backup 或 quant-lab 修改、真实平台 receipt 声明。 |
| 不确认会阻塞什么 | 阻塞 CP4、Story 拆分、Feature design、LLD 和后续实现；当前源码/Doctor/Ruff/guardrail 缺口保持未整改。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR047-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | capsule-first；只有 traceability/conflict/deep review 才展开正式产品或设计全文 |
| 全文档读取扩展 | 11 条：R1 的 5 条 + R2 深度评审的 Requirements/五份设计 6 条；均在 READ-EXPANSION-LEDGER |
| 缺失 / waived 理由 | blocker=0；waiver=0；platform runtime/receipt 不在本门证明范围 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.current.json`、gate ledger | scanned | 2 | 0 | R1 open 与 changes_requested 已留事件；当前仍为 CP3 |
| 委托交还摘要 | `process/handoffs/CR047-CP2-CP3-META-SE.md` | scanned | 4 | 4 | 用户批准 inline fallback；架构灰区全部进入 DQ |
| 自动预检结果 | CP3 R2 result | scanned | 0 | 0 | PASS，blocker=0，waiver=0，supersedes R1 |
| discussion log/checkpoint | CP3 R2 discussion artifacts | scanned | 6 | 6 | AGA-WT-01..06 → CP3-DQ-01..06 |
| 下游正式产物 | Requirements v1.5 + Blueprint/Domain/Dependency/HLD/ADR v1.1 | scanned | 7 | 6 | 一项是 HLD split/manifest 支撑对象的派生边界 |
| independent review | 用户提供的 R1/R2 独立评审 | scanned | 6 | 3 | DQ-03/05/06；其余为同 DQ 的检查点/identity/修订记录精化 |
| 用户显式输入 | CP2 approval、禁用子 Agent、CP3 R2 修改授权 | scanned | 5 | 3 | 已批准事实保留；本轮修改进入 DQ-03/05/06 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 6 | CP3-DQ-01..06 |
| 高风险策略确认 | 5 | DQ-01 truth、DQ-03 history、DQ-04 gate、DQ-05 firewall、DQ-06 ceiling |
| agent 默认处理 | 5 | module/file 细分、fixture 命名、Ruff safe-fix 分批、README 排版、Run event 命令封装 |
| 仅审计记录 | 8 | 八项设计模拟、inline fallback、unavailable receipt、排除路径与 R1 supersession |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP3-DQ-01 | architecture | workflow truth 是保留多个 source owner，还是新建单一总状态？ | State/CR catalog 各自单写；CURRENT 仅投影；新增只读 relation check，顺序为 route→write/status-sync→refresh→check。 | A：新总状态文件；B：只修 `STATE.md` parser。 | 推荐不增加第四套 truth，能覆盖 closed/missing active 和 CURRENT drift；代价是跨模块契约测试更多。A 迁移大且双写风险；B 不能解决 lifecycle/projection 冲突。 | 错误 owner 边界会再次产生 split-brain。 | 只有现有 schema 无法表达已批准的新对象时另立集中模型 CR；失败时保持上一稳定状态。 |
| CP3-DQ-02 | architecture | meta-flow 内部产品/设计/质量文档的 canonical 路径如何确定？ | artifact `process/docs/**` 是唯一可写 canonical；源码根 `docs/` 仅 public docs；不默认建兼容 symlink。 | A：ignored 根 symlink；B：内部文档迁回源码仓。 | 推荐跨设备可复现且与已完成 process link 一致；代价是要修 legacy ref。A 会恢复“本机有、clone 无”；B 混淆源码/过程仓职责。 | 硬编码根路径的外部脚本可能失效。 | 只有正式外部契约要求根路径时，另立单向 generated read-only view；不得成为 writer。 |
| CP3-DQ-03 | risk_acceptance | Doctor 如何在 observed count 已从 21 漂移到 22 时保持可判定且不改写历史？ | `B0_pre=21` 仅留历史锚；CP7 采集 `B0_cp7` 的 observed/classified/unclassified/blocking_active/warning/cold；终态 `blocking_active=0`、`unclassified=0` 且全部 delta 可解释。 | A：固定 observed count；B：提高全局阈值；C：原位截断（禁止）。 | 推荐允许合规产物改变计数，但 active/default-required 仍阻断；固定值会被过程产物反复击穿，全局放宽会掩盖膨胀。 | 错误分类或无解释 delta 会隐藏必读证据。 | `must_read/default-read` 优先；CR-047 自身超预算不自动合法；任一 blocker/unclassified/delta 缺口阻断。 |
| CP3-DQ-04 | architecture | clean-clone guardrail、cache、Ruff、pytest 和 installer 是否需要新 release orchestrator？ | 扩展现有 checker/installer/guardrail，组合层只串行编排；tracked/package cache block，ignored-only warn；Ruff+pytest+3 dry-run 写真实 Run event。 | A：新 release orchestrator；B：继续手工执行。 | 推荐复用现有 CLI/CI、底层检查仍独立；代价是 guardrail 需 typed classifier。A 当前过度设计；B 无法保证同一门。 | 大脚本职责膨胀或 warning 误降级。 | 组合层不得复制业务判断；多语言/多团队或三次职责冲突后再拆 orchestrator。 |
| CP3-DQ-05 | architecture | ST-WT-007 如何机器证明没有改写 CR-046 protected originals？ | 对象身份=`source_cr+object_type+canonical_ref+provenance_ref`；CP6 pre-implementation 生成 manifest，CP6 完成首验、CP7 再验；禁止 prefix-only allowlist；变更即阻断并拆子 CR。 | A：只做人工 diff；B：目录前缀 allowlist（禁止）。 | 推荐能处理 CR-046/047 同目录混居并保留 hash；人工检查不可稳定重放，prefix 会误伤/漏判。 | 漏列或身份歧义会破坏 evidence integrity。 | 身份/原 hash 不明或 hash 变化即 `BLOCKED/NEEDS_DESIGN_CLARIFICATION`；子 CR 声明 replacement/supersession 与审批。 |
| CP3-DQ-06 | risk_acceptance | 是否接受当前证明层级形成的叠加式 CP7/CP8 上限？ | 有效 inline fallback 时 CP7 最高 `PASS_WITH_RISK`；无独立 QA 且无有效 fallback 则 BLOCKED；五项 CR-046 继承风险任一 OPEN 时 CP8 最高 `READY_WITH_RISK`。 | A：CP7 前解除禁用子 Agent 并取得可验证独立 QA/平台证据；B：若要求无风险 READY，则在证据齐备前阻断。 | 推荐诚实反映 evidence ceiling；代价是本 CR 不能宣称无风险 READY。新证据只解除对应维度。 | 把 fixture/inline 写成独立或平台 attestation 会形成虚假结论。 | 风险独立叠加；关闭独立 QA 不自动关闭 receipt/telemetry/pilot/working-tree 风险。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | 批准 CP3-DQ-01..06 的推荐方案，并接受 ADR-WT-005 单 HLD/越界子 CR 与 ADR-WT-006 叠加式结论上限 |
| 备选方案 | 可逐项使用 `修改: CP3-DQ-xx <具体修改点>`；也可 reject 回到 solution-design |
| 影响维度 | source ownership、跨设备 portability、evidence integrity、发布退出语义、维护成本、operator usability |
| 优劣分析 | 推荐组合不新建总状态/发布编排器，也不靠本机 symlink 或历史改写修绿；代价是 CP4/CP5 跨模块设计面较大 |
| 风险与回退 | fail-closed、projection 可重建、append-only correction、unknown current/package/security 保持 blocker |
| 用户需决策事项 | `CP3-DQ-01`、`CP3-DQ-02`、`CP3-DQ-03`、`CP3-DQ-04`、`CP3-DQ-05`、`CP3-DQ-06` |

### 架构摘要

- 推荐方案：A — in-place contract convergence。
- 数据方向：workspace health → State/CR source write → CURRENT projection → truth check → Doctor → delivery preflight → status/docs。
- 模块：9 个；Feature：5 个；Story：7 个；Wave：3 个。
- Use Case 追踪：UC-WT-001..007 全部映射到 REQ、module、ADR 和 TC。
- 设计模拟：SIM-WT-01..08 共 8/8 PASS；它们不是 CP7 执行证据。
- HLD 拆分：单 HLD 保持；manifest 只是验证支撑对象。独立 release/owner/rollback 或 protected original mutation 触发子 CR/HLD。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 approved | PASS | CP2 checkpoint + gate ledger | 四项产品策略已批准 |
| Requirements v1.5 与五份架构 v1.1 存在 | PASS | product/design refs | revision/gotchas/traceability 齐备；HLD=20353 bytes |
| Context ready | PASS | CP3 context | compact；read-expansion refs=11 |
| 自动预检通过 | PASS | CP3 R2 result | attempt=2；blocker=0；waiver=0；supersedes R1 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | source-owned truth graph 与 projection 单向边界 | approved | DQ-01、ADR-WT-001 | 接受推荐方案 |
| 2 | artifact internal docs canonical 路由 | approved | DQ-02、ADR-WT-002 | 接受推荐方案 |
| 3 | Doctor B0_pre/B0_cp7、delta、active blocker 与历史不可变 | approved | DQ-03、ADR-WT-003 | 接受推荐方案与风险边界 |
| 4 | tracked rule/cache 与组合 preflight | approved | DQ-04、ADR-WT-004 | 接受推荐方案 |
| 5 | 对象身份制 CR-046 firewall、CP6/CP7 双验、子 CR 路由 | approved | DQ-05、ADR-WT-005、HLD §8.5 | 接受推荐方案；protected original 变更必须阻断并拆子 CR |
| 6 | 叠加式 CP7/CP8 evidence ceiling | approved | DQ-06、ADR-WT-006、HLD §15 | 接受 PASS_WITH_RISK / READY_WITH_RISK 上限 |
| 7 | 单 HLD、7 Story、3 Wave 与切换条件 | approved | ADR-WT-005、HLD §13-14 | 接受单 HLD 与越界拆分条件 |
| 8 | 失败路径、NFR、风险、回退可执行 | approved | HLD §8-12、§15 | 接受 fail-closed 路由 |
| 9 | 不授权边界完整 | approved | CR、context、HLD §2 | 不扩大授权范围 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 六项核心架构/风险决策关闭 | approved | CP3-DQ-01..06 | 用户批准全部推荐方案 |
| 五份设计可作为 CP4 输入 | approved | CP3 result | 进入 story-planning |
| Story/Feature/DAG 前置边界明确 | approved | HLD §13-14 | 允许准备 CP4/CP5，不授权实现 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| Blueprint | `process/docs/design/CR047-WORKFLOW-TRUTH-BLUEPRINT.md` | approved | CP4 输入 |
| Domain Map | `process/docs/design/CR047-WORKFLOW-TRUTH-DOMAIN-MAP.md` | approved | CP4 输入 |
| Dependency Map | `process/docs/design/CR047-WORKFLOW-TRUTH-DEPENDENCY-MAP.md` | approved | CP4 输入 |
| HLD | `process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md` | approved | Option A 与 v1.1 修订冻结 |
| ADR | `process/docs/design/CR047-WORKFLOW-TRUTH-ARCHITECTURE-DECISION.md` | approved | ADR-WT-001..006 accepted |
| CP3 R2 result/context | process refs | PASS / ready |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-14T15:29:31Z
- 修改意见：无；批准 CP3 R2 全部推荐方案并要求继续推进到下一人工门禁。
- 风险接受项：CP3-DQ-03、CP3-DQ-06；接受动态 Doctor 双基线与叠加式 CP7/CP8 结论上限，不扩大 runtime/Git/历史原件等授权。

## R1 审查历史（不可覆盖）

- 结论：`changes_requested`
- 审查来源：用户提供的两轮独立评审并于 2026-07-14 明确授权 R2
- Gate event：`GATE-CR047-CP3-CHANGES-REQUESTED-R2-20260714T151055Z`
- 修改意见：动态 Doctor 双基线；对象身份制 CR-046 firewall；叠加式 CP7/CP8 上限；同步 Requirements/HLD/ADR/Blueprint/Domain/Dependency/Decision Brief。
- 被替代自动结果：`process/checks/CP3-CR047-HLD-CONSISTENCY.result.json`（仍保留）

## 可接受回复

- `approve`
- `修改: CP3-DQ-xx <具体修改点>`
- `reject`
