---
discussion_id: "CP3-CR172-PATH-I-HLD-20260717"
change_id: "CR-172"
phase: "solution-design"
checkpoint: "CP3"
status: "ready-for-host-decision-brief"
owner: "meta-se-critical"
created_at: "2026-07-17T17:10:54+08:00"
context_ref: "process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml"
checkpoint_ref: "process/checks/CP3-CR172-PATH-I-DISCUSSION-CHECKPOINT.json"
---

# CR-172 PATH-I CP3 架构讨论日志

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-17 | meta-se-critical | 前置识别 4 个 Architecture Gray Areas，完成 table-first advisor discussion、候选架构比较、HLD 拆分判定和 CP3 Decision Brief 输入。 |
| 1.1 | 2026-07-18 | meta-se-critical | CP3 review correction R1：关闭 return-definition ADR 归属、native producer CP5 单一插桩义务、六授权执行资格 DAG 三项评审问题；不新增或改变原 4 个 CP3 DQ。 |

## 1. 讨论边界与输入

| 项 | 结论 |
|---|---|
| 已批准输入 | DQ-CR172-009～015 已在 CP2 `7/7` 批准；本轮不重开产品范围。 |
| 当前阶段 | PATH-I CP3 design-only；只冻结蓝图、HLD、ADR 和未来 fixture 可验证合同。 |
| 默认读取入口 | `process/context/CP3-CR172-PATH-I-DESIGN-CONTEXT.yaml`，`read_profile=compact`。 |
| 扩读理由 | `deep_review`：为完成 REQ-CR172-009～015 `7/7` 架构追踪、现有蓝图增量合并和字段冲突核对，读取 capsule `allowed_reads` 中的 CP2、CR172 产品段落和三份设计基线。 |
| 不授权项 | 目录创建、runner 修改、真实 lake/NAS 读写、multi-trial、trial-return/R、sync/pull/materialize、SignalBatch 生成/传输、交易、迁移、发布/部署、Git remote write 均为 `0`。 |
| Advisor 形成方式 | 未新增 reviewer 子 agent；由已真实调度的 `meta-se-critical` 以 `lane-product`、`lane-architecture`、`lane-quality`、`lane-docs-check` 四个分析视角形成方案输入，不伪造独立角色结论。 |

## 2. Architecture Gray Areas

| 灰区 ID | 关键问题 | 为什么影响 HLD | 影响面 | canonical refs | 状态 |
|---|---|---|---|---|---|
| AGA-CR172-I-01 | trial-return 由 native multi-trial instrumentation 生成、由外部 import，还是保持 absent？manifest 与 seal 的先后顺序如何冻结？ | 决定 source owner、artifact schema、失败语义、lineage 边界和未来 Story 分组；顺序错误会让未 seal 对象进入副本链。 | 范围 / 模块 / 数据 / 验证 / 回滚 | REQ-009、REQ-014、SC-I01、SC-N02、SC-N04 | selected：native-first；import/absent 保留切换条件 |
| AGA-CR172-I-02 | research-local canonical、NAS replica 和 execution immutable cache 是三段复制链，还是共享盘/单机保守模式？ | 决定跨机 identity、故障域、freshness 判定、materialization 原子边界和 direct-NAS 禁止规则。 | 架构 / 存储 / 安全 / 可靠性 / 运维 | REQ-010、REQ-012、SC-I02、SC-N03、SC-B02/B03、SC-F02 | selected：三段隔离链 |
| AGA-CR172-I-03 | 六类真实动作在何处独立授权、撤销与 fail-closed？ | 决定 blast radius、partial approval 是否会权限并集、操作中撤销后的回滚边界和审计证据。 | 安全 / 权限 / 运行 / 回滚 / 验证 | REQ-011、REQ-012、SC-A02、SC-Q02 | selected：六个独立 authorization envelope + 六个判定点 |
| AGA-CR172-I-04 | empirical R 何时为 `declared_exact`、`empirical`、`typed_unavailable` 或 `BLOCKED`；`FU-CR173-001` 是否阻断降级设计？ | 决定 C1 claim ceiling、错误修复是否越权和 PATH-C/A 后续门禁；绝对阻断会破坏 DQ-003，过宽放行会产生 positive empirical overclaim。 | 方法 / 数据 / 声明 / 风险 / 后续 CR | REQ-014、SC-Q02、SC-C02、FU-CR173-001 | selected：分类显式；v2 只硬阻断 positive claim |

## 3. Table-first Advisor Discussion

### 3.1 AGA-CR172-I-01：source、manifest 与 seal

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Native instrumentation-first：未来 multi-trial runner 输出 per-trial series；payload 验证 → manifest → seal → local canonical | lineage 最完整；family/run/trial 同源；可对 seal/hash/URI 做 fixture 契约验证 | 后续需要独立 runtime 与 generation 授权；实现量高于 import | source owner / schema / seal / lineage / fixture / rollback | 推荐 | 假设未来 native producer 可插桩且 owner 明确；若 CP4/CP5 证明 producer 不可接入，切 B |
| B. Import-first quarantine：外部预计算 series 先进入 research-local quarantine，通过 import/provenance contract 后再使用同一 manifest/seal 链 | 可在 native runner 不可用时保留真实上游入口；下游 identity/replica 合同可复用 | provenance 较弱；需独立 import/validation CR；不能把外部路径当 identity | integration / provenance / security / validation | 条件备选 | 仅当 native producer 不可行且外部源具备完整 provenance、hash、window、trial identity 时启用 |
| C. Source absent + typed_unavailable | 零运行风险；不伪装现有能力 | 不产生 empirical R 或 positive C1 | scope / claim ceiling / delivery | 安全回退 | 任一 owner、schema、source provenance 或授权缺失时立即回退；仍允许 PATH-I 合同设计 |

Advisor 视角摘要：`lane-product` 需要保留未来真实 overfit 评估价值，因此选 A；`lane-architecture` 要求 source 与 CR-163 metadata/ref、`layered_returns.csv`、CR-173 R 输入解耦；`lane-quality` 要求未 seal/wrong-kind/alignment failure `100%` fail-closed；`lane-docs-check` 要求明确 A 是未来合同而非当前实现。

### 3.2 AGA-CR172-I-02：canonical、replica 与 materialization

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. research-local active canonical → NAS verified replica → execution local immutable cache | 单一 canonical；runtime 与共享盘隔离；logical URI + hash 跨挂载稳定；可原子回滚 pointer | 需要两段验证和 staging；运维合同较多 | storage / identity / reliability / security / operations | 推荐 | 仅在 manifest/seal/hash/freshness `100%` 可验证且六动作继续分权时使用 |
| B. research-local canonical only，NAS/执行分发 blocked | 最小故障域；NAS 不可用时仍保留研究证据 | 不支持执行机离线消费；后续 activation 受阻 | delivery / operations / user value | 条件备选 | NAS、materializer 或授权模型不满足时切换；不得用 direct-NAS 代替 |
| C. NAS runtime canonical 或 execution direct-NAS read | 路径表面简单、少一次复制 | 违反 CP2；扩大共享盘故障域；stale/partial/hash mismatch 易被消费 | security / consistency / runtime / rollback | 禁止 | 无 PATH-I 内切换条件；只能经独立安全/合规 CR 重开上游范围 |

Advisor 视角摘要：四个视角一致选择 A；关键牺牲是 staging/verification 合同复杂度，换取 runtime 隔离和可追溯回滚。B 是 NAS 或执行校验能力缺失时的保守降级；C 不是候选架构，只用于记录禁止边界。

### 3.3 AGA-CR172-I-03：六类逐动作授权

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 六个独立 authorization envelope，在 dereference/launch/write/compute/sync/pull 前分别判定 | blast radius 最小；逐项撤销和归因；partial approval 不产生权限并集 | 门禁和证据较多；实现需统一 schema | security / runtime / audit / rollback / tests | 推荐 | 授权存储能表达 action_kind、scope revision/hash、allow/deny paths、有效期/撤销、evidence；否则先切 C |
| B. runtime+generation、sync+pull 两组合并 | 决策数量较少 | 撤销和归因变粗；可能隐含 workspace write 或跨机读写 | security / maintenance / incident response | 不推荐 | 只有未来独立安全评审证明不降低 deny-default 时才可考虑 |
| C. 六类全部 deny，仅 static/fixture | 风险最低；与当前 CP3/CP5/CP7 守卫直接一致 | 无法执行真实链路 | delivery / activation | 当前阶段强制回退 | 当前真实授权即为 C：`0/6`；未来只有逐动作门禁批准后才按 A 局部打开 |

Advisor 视角摘要：架构长期采用 A，当前运行状态采用 C；两者不矛盾。授权在操作前判定，操作中撤销时不得推进任何 pointer，partial staging 保持 non-distributable/non-runtime 并等待单独受权清理。

### 3.4 AGA-CR172-I-04：empirical disposition 与 positive claim

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 显式四态：declared_exact / empirical / typed_unavailable / BLOCKED；FU-CR173-001 只硬阻断 positive empirical result | 保留 DQ-003 降级；防止 declared-exact 冒充 empirical；错误与缺失可区分 | claim guard 和 provenance 字段较多 | methodology / data / validation / release wording | 推荐 | positive available count 或 `c1_computable=true` 只有 v2+hash、uncertainty、有效域/偏差界限、独立验证和 computation 授权全部满足时成立 |
| B. 所有 pre-v2 empirical 路径统一 BLOCKED | 最保守；规则简单 | 会把 typed_unavailable 设计路径错误封死；违背已批准 DQ-003 | scope / sequencing / user value | 不推荐 | 仅在用户撤销 DQ-003 时重开，不属于 PATH-I 默认 |
| C. 只允许 declared_exact fixture；empirical 永久 deferred | 便于本地 fixture；无 sampling-error overclaim | 无法支撑真实 effective count；长期价值不足 | methodology / delivery | 条件备选 | 若 FU-CR173-001 不立项且业务接受永久无 positive empirical claim，可切换 |

Advisor 视角摘要：选择 A。`typed_unavailable` 用于缺 source、缺授权、缺 v2 等可解释前置不足；`BLOCKED` 用于 hash/identity/alignment 冲突、篡改或未授权 repair 等完整性/违规事件。两者均保持 `c1_computable=false`。

## 4. SignalBatch scope containment（非独立 Gray Area）

SignalBatch 产品边界已在 CP2 冻结，不作为本轮重新选择的架构灰区。PATH-I 只记录以下精确 `8/8` mandatory contract slots，额外 mandatory slot=`0`：

1. `schema_version`
2. `batch_id`
3. `strategy_id`
4. `strategy_package_hash`
5. `content_sha256`
6. `signature/key_id`
7. `valid_from/valid_until`
8. `sequence_no`

其中第 6、7 项保留为 CP2 批准的复合 slot；物理序列化时是嵌套还是拆列，连同 mailbox path、状态机、ack、idempotency/replay、exchange、consumer 和 intraday，全部 deferred。当前 signal Story=`0`、详细模块=`0`、实现=`0`、真实传输=`0`。

## 5. 候选架构与推荐

| 候选 | 端到端形态 | 满足 REQ-009～015 | 主要代价 | 结论 |
|---|---|---:|---|---|
| ARCH-A Native Sealed Artifact Pipeline | future native producer → research-local validate/manifest/seal/canonical → authorized NAS replica → authorized execution staging/verify/atomic cache；六动作逐项 gate；R 四态 claim guard | `7/7` | 合同与验证点最多 | 推荐 |
| ARCH-B Import-first Sealed Artifact Pipeline | external series → research-local quarantine/import provenance → 与 A 相同 seal/replica/materialization；无合格 import 时 typed_unavailable | `7/7`（需 future import CR） | provenance 与供应方风险更高 | 条件备选 |
| ARCH-C Contract-only / distribution blocked | source absent 或 research-local only；不分发、不计算 empirical R，保持 typed_unavailable | `6/7` 完整、REQ-010/012 用户价值部分降级 | 无执行机消费和 positive empirical 输出 | 安全回退，不作为目标态 |

推荐 `ARCH-A`。切换到 B 的条件是 native producer 在 CP4/CP5 证明不可插桩但外部 source provenance 可完整验证；切换到 C 的条件是 owner、授权、source、NAS、materializer 或 v2 任一前置缺失。任何情况下都不得切换到 NAS runtime canonical、direct-NAS runtime read、GitHub 数据面扩大或 declared-exact→empirical 重标。

## 6. HLD 拆分判定

| 判定信号 | 观察 | 结论 |
|---|---|---|
| 核心产物 > 1 | 核心产物是同一 sealed trial-return artifact chain；manifest、seal、replica、materialization、authorization 和 R disposition 是该链的强耦合合同。 | 不触发 |
| 职责跨层 | 不修改 host-orchestrator 全局机制；只定义 PATH-I 领域合同。 | 不触发 |
| Story 数 > 5 | CP3 禁止提前创建 Story；当前 Story=`0`。 | 不可作为拆分依据 |
| ADR 分簇 | identity/replica/auth/R ADR 共享同一 artifact identity、failure boundary 与 claim ceiling，拆分会产生双向引用。 | 反信号 |
| 可独立交付 | SignalBatch 仅 8 字段 boundary，无 exchange 模块、状态机或独立成功标准；不足以形成独立 HLD。 | 反信号 |
| 评审者差异 | 数据、权限、方法评审均围绕同一 PATH-I gate，不存在独立发布节奏。 | 反信号 |

结论：保留单份 `HLD-TRIAL-RETURN-DEPLOYMENT-CONTRACTS.md`。若未来启动 `DF-CR172-SIGNAL-BATCH-EXCHANGE`、`DF-CR172-INTRADAY-REALTIME-SIGNAL`、`FU-CR173-001` 或外部 R import，则必须各自新建 CR/HLD，不回填扩张本 HLD。

## 7. CP3 Decision Brief 输入

| Decision ID | 决策类型 | 待确认问题 | 推荐方案 | 可执行备选 | 优劣与影响 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|
| CP3-DQ-CR172-I-01 | architecture | 是否确认 native sealed artifact pipeline 作为目标态，并保留 import/absent 两级回退？ | `ARCH-A` | `ARCH-B` import-first；`ARCH-C` typed_unavailable | A lineage 最完整但后续实现复杂；B provenance 风险较高；C 最安全但无 positive empirical 价值 | native 不可插桩切 B；provenance/owner 缺失切 C |
| CP3-DQ-CR172-I-02 | architecture | 是否确认 payload 验证→manifest→seal→local canonical→replica verify→materialize verify 的严格顺序和 pointer-only rollback？ | 确认严格顺序；副本/缓存不重新 seal，校验失败不推进 pointer | research-local only、distribution blocked | 推荐可证明身份稳定和原子回滚；备选降低跨机价值 | 任一 manifest/seal/hash/freshness 校验能力缺失时回退 blocked；不得 direct-NAS |
| CP3-DQ-CR172-I-03 | security | 是否确认六动作六判定点、partial approval 无权限并集、mid-operation revoke 不推进 pointer？ | 六个独立 envelope；当前全部 deny `0/6` | 全不授权，仅 fixture；不采用粗粒度合并 | 推荐审计/撤销最精细，代价是门禁多；全 deny 是当前安全态 | 授权系统不能安全表达时维持全 deny，回到设计澄清 |
| CP3-DQ-CR172-I-04 | risk_acceptance | 是否确认 empirical 四态与 v2 条件式前置，且 SignalBatch 只保留 8-slot boundary？ | 四态 claim guard；typed_unavailable 可设计；positive claim 需 FU-CR173-001；signal detailed scope=0 | declared-exact-only / permanent typed_unavailable；signal local-only | 推荐保留演进同时防 overclaim；备选更保守但价值降低 | 未完成 v2 或授权缺失即 typed_unavailable；完整性冲突为 BLOCKED；详细 signal 另起 CR |

## 8. 讨论结论与未决项状态

| 项 | 状态 | 结论 |
|---|---|---|
| AGA-01～04 | RESOLVED-FOR-DRAFT | 推荐方案已形成，待 host 汇入 CP3 人工 Decision Brief。 |
| HLD 拆分 | RESOLVED-FOR-DRAFT | 单份 HLD；future signal/v2/import 各自独立 CR/HLD。 |
| BLOCKING 缺失信息 | NONE | 不阻断 HLD 成文。 |
| CP3 人工决策 | OPEN | 上述 `4` 项由 host-orchestrator 统一发起，meta-se 不直接询问用户。 |
| 真实操作 | NOT-AUTHORIZED / NOT-EXECUTED | 六类动作 `0/6`；其他 lake/NAS/runtime/signal/trading/deploy/Git remote 操作均 `0`。 |

## 9. CP3 review correction R1

| 整改 ID | 评审问题 | 设计决定 | 量化守卫 | 状态 |
|---|---|---|---|---|
| CORR-CR172-CP3-R1-01 | `simple_return` 是 MVP 必填，但 return-definition ADR 归属和 net/gross/nav 演进未明确 | 新增 `ADR-CR172-I-010`；CP3 architecture owner 冻结业务语义、schema identity 和演进规则；v1 canonical payload 恰好为 `timestamp + simple_return` 两列；CP5 只选实现库，字段变化需 versioned ADR + schema bump，高风险语义变化拆后续 CR | canonical columns=`2/2`；silent net/gross/nav addition=`0` | RESOLVED |
| CORR-CR172-CP3-R1-02 | native producer 插桩位置仍是 OPTIONAL | 新增 `DO-CR172-CP5-001`；CP5 必须证明唯一 integration point、source/file owner inventory、调用合同、授权顺序、失败回退、测试入口和 merge order；无法证明则 `NEEDS_DESIGN_CLARIFICATION` 或 Spike/ARCH-B | CP5 obligation=`6/6`；未证明即进入实现=`0` | RESOLVED |
| CORR-CR172-CP3-R1-03 | 六类授权独立但执行依赖顺序未冻结 | 保持六 record 独立审批/撤销，同时增加 execution eligibility DAG：`data_lake_read -> runtime -> generation -> empirical_R`，以及 `generation -> NAS sync -> execution materialize`；每步还需本动作独立授权、直接前置和同 scope/sealed provenance | runtime own-auth/no-data-read 时 eligible=`false`；runner/workspace/pointer=`0/0/0`；permission union=`0` | RESOLVED |

本轮整改属于已批准产品边界内的架构合同补全，不形成第 5 个 CP3 DQ，不改变 `CP3-DQ-CR172-I-01～04` 的推荐值、备选或风险接受语义。真实动作授权/执行保持 `0/6`，PATH-I claim ceiling 不变。
