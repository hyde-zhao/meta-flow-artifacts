---
status: accepted
version: "1.1"
cr_ref: "CR-047"
hld_ref: "process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md"
decision_gate: "CP3"
confirmed_by: "user"
confirmed_at: "2026-07-14T15:29:31Z"
---

# CR-047 Workflow Truth Architecture Decisions

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-14 | host-orchestrator-inline / meta-se | CP3 R2：ADR-WT-003 改为双基线与 delta 契约；ADR-WT-005 增加 protected-original 子 CR 触发；新增 ADR-WT-006 叠加式验证结论上限。 |
| 1.0 | 2026-07-13 | host-orchestrator-inline / meta-se | 形成 workflow truth、内部文档路由、历史质量治理、clean-clone preflight 与 HLD 拆分决策候选。 |

## 决策状态说明

以下 ADR 已由用户在 CP3 R2 批准并转为 `accepted`。该批准只允许进入 Story planning、Feature design 和 CP4/CP5 准备，不授权代码实现、提交、推送、runtime、SaaS 或生产写入。

## ADR-WT-001：保留 source-owned truth graph，拒绝新增总真相文件

| 字段 | 内容 |
|---|---|
| 状态 | accepted / CP3-DQ-01 |
| 背景 | State v2、formal CR/index/ledger 与 CURRENT 分别表达执行态、生命周期和发现投影。当前问题是跨对象关系未被完整校验，而不是缺少第四个可写真相源。 |
| 决策 | 保留各 owner 单写；增加只读 consistency relation。执行顺序固定为 `workspace health → State/CR controlled write → CURRENT refresh → state/CR/current consistency check`。 |
| 调用契约 | Host/CLI 在状态写入、CR status-sync、workspace check 或 release preflight 后触发；输入为 `STATE.current.json`、`CR-INDEX.json`、formal CR、CR ledger、`CURRENT.json`；输出字段级 finding 与非零/零判定。 |
| 失败行为 | active CR 不存在、生命周期已结束、CURRENT scalar/ref 不一致或 legacy canonical index 存在时 fail-closed；不从 Markdown 或 CURRENT 反向猜测修复。 |
| 后续衔接 | Story planning 将 State reader、CR tracker、CURRENT builder 分配为显式 owner，并加入 CR-037 stale-state 回归 fixture。 |
| 推荐理由 | 修复关系验证而不增加平行状态；可保留现有 writer/schema/ledger 契约。 |
| 备选 A | 新建单一总状态文件：读取简单，但迁移成本高且形成第四套 truth。 |
| 备选 B | 仅修 `STATE.md` parser：改动小，但继续把人类摘要当机器入口，不能覆盖 CURRENT/CR lifecycle 冲突。 |
| 风险 | 若 finder 与 writer 各自复制 lifecycle 规则，仍可能漂移。 |
| 缓解 | lifecycle enum 与 resolver 复用现有 canonical helpers；fixture 覆盖 closed/cancelled/superseded/missing/candidate。 |
| 回退 / 切换 | 只有现有 State/CR schema 无法表达一个已批准的新领域对象时，才由新 CR 评估集中模型。 |

## ADR-WT-002：内部文档只以 artifact `process/docs/**` 为 canonical

| 字段 | 内容 |
|---|---|
| 状态 | accepted / CP3-DQ-02 |
| 背景 | meta-flow 自身的内部产品、设计、质量文档属于 artifact repo；源码根 `docs/` 只跟踪公开入口。本机兼容目录或软链接会重新制造 clean clone 不可复现。 |
| 决策 | `process/docs/product|design|features|quality|release/**` 是唯一内部可写副本；源码根 `docs/` 只保留 tracked public docs。不默认建立 `docs/product|design|quality` 兼容 symlink。 |
| 调用契约 | workspace link 建立 `process` 路由；context/checkpoint/CR refs 使用 project-relative `process/docs/**`；routing checker 检查 canonical writable copy count=1。 |
| 失败行为 | process 断链、metadata 冲突、project_name 错误或发现第二个内部可写副本时 BLOCKED；不得静默复制文档。 |
| 后续衔接 | Story planning 需要盘点 legacy refs，并在同一 Story 内同步模板、测试和规则说明。 |
| 推荐理由 | 两仓职责清晰，跨设备只需一次 link，避免 ignored 本机视图成为隐式依赖。 |
| 备选 A | 创建 ignored 根 symlink：旧路径兼容，但 clean clone 仍缺该视图。 |
| 备选 B | 把内部文档迁回源码仓：clone 简单，但违背已批准的 artifact routing，且扩大源码仓噪声。 |
| 风险 | 外部脚本可能硬编码 `docs/product/**`。 |
| 缓解 | 先以 `rg`/fixture 发现硬编码；对确有外部契约的工具使用显式 generated read-only view，并另立决定。 |
| 回退 / 切换 | 只有正式平台契约证明必须使用根路径时，才引入单向 generated view；它不得成为 canonical writer。 |

## ADR-WT-003：Doctor 使用严重度与生命周期感知的历史治理

| 字段 | 内容 |
|---|---|
| 状态 | accepted / CP3-DQ-03；继承 CP2-DQ-02/03 |
| 背景 | 现有 Doctor 将活动必读对象与 closed 历史对象按同一活动预算扫描，并存在 Quality Model 路径错误与历史 read-expansion provenance 缺口。直接扩大全局阈值或截断历史会破坏预算意义或证据完整性。 |
| 决策 | 只有 `ERROR/FAIL/BLOCKED` 使 Doctor 非零；WARN 必须分类、计数和披露。`B0_pre.observed=21` 仅是 CR-047 产物生成前的历史回归锚；CP7 开始采集动态 `B0_cp7`（observed/classified/unclassified/blocking_active/warning/cold），终态解释其全部新增/删除/重分类 delta。活动/default-required read 严格受限；closed/cold history 保留原件，并以 archive ref 或 compact summary/index/hash/append-only correction 进入默认读面。 |
| 调用契约 | Doctor 读取 retention/quality policies、formal CR lifecycle、context read policy、CP results 与 ledgers；输出 finding severity、object/path、bytes/limit、reason、remediation route。 |
| 失败行为 | `blocking_active>0`、`unclassified>0`、当前 required-read 超限、delta 无法解释、derived source 缺失、证据伪造或无法分类的必需对象均为 blocker；CR-047 自身新产物不享受自动降级。legacy/unavailable 可为 warning，但不得标 PASS。 |
| 历史修复 | Quality Model 路径指向 `process/state/READ-EXPANSION-LEDGER.ndjson`；历史 CP/read/run 缺口只追加 correction/recovery event，保留原 hash，不补写猜测的 actor/timestamp/receipt。 |
| 后续衔接 | Story planning 将 policy/schema/checker、历史迁移与 fixture 分离；CP7 固化 `B0_cp7`，验证 delta、原 hash、warning 计数和 blocker exit code。 |
| 推荐理由 | 同时满足发布可用性、上下文预算和证据不可伪造。 |
| 备选 A | 提高全局阈值：最快，但活动对象会继续膨胀。 |
| 备选 B | 原位截断/重写：可快速绿，但违反 evidence integrity，明确禁止。 |
| 风险 | 错误 cold 分类可能隐藏当前执行所需证据。 |
| 缓解 | `must_read/default-read` 优先于 lifecycle；任何 active context 引用的对象不得仅存在于 archive。 |
| 回退 / 切换 | 若 classification 无法稳定推导，则先把对象保持 blocking，不允许为了绿灯降级。 |

## ADR-WT-004：扩展现有 guardrail/preflight，不新建 release orchestrator

| 字段 | 内容 |
|---|---|
| 状态 | accepted / CP3-DQ-04；继承 CP2-DQ-01/04 |
| 背景 | clean archive 缺 ignored 根 `AGENTS.md`，现有 guardrail 却要求它；所有本机 cache 又被统一阻断；Ruff 尚未进入组合门。 |
| 决策 | tracked `delivery/rules/AGENTS.md` 为 canonical，根 wrapper 由 installer 生成。扩展现有 checker/installer/guardrail：tracked 或 package-input cache BLOCKING，纯 ignored local cache WARN/可清理；组合 preflight 串行执行 truth、Doctor、guardrail、Ruff、pytest、三平台 installer dry-run，并写真实 Run event。 |
| 调用契约 | release/CI 或人工 preflight 触发；输入为 Git tracked set、package manifest、installer plan、Python source/tests；输出分步骤 verdict、finding severity、command evidence/ref。 |
| 失败行为 | canonical rule 缺失、generated wrapper drift、tracked/package cache、Ruff/pytest/dry-run 失败均非零；ignored cache 仅 warning，但若进入 staging/package 立即升级 blocker。 |
| 后续衔接 | README 给 Codex/Claude/Qoder 提供显式 `--project-dir .`；CP7 在 clean archive 与测试后本机 tree 各执行一次。 |
| 推荐理由 | 复用现有 CLI 和 CI，避免再增加调度层；每个底层检查仍可独立运行。 |
| 备选 A | 新建 release orchestrator：边界清晰，但当前单仓规模下引入额外状态、配置和测试。 |
| 备选 B | 保持手工命令：改动最小，但无法保证发布前真的执行同一集合。 |
| 风险 | 单个 guardrail 脚本职责继续扩大。 |
| 缓解 | 保持 checker 可组合、输出 typed finding；组合层只编排，不复制业务判断。 |
| 回退 / 切换 | 当多语言/多团队需要独立 release owner 或组合层出现三次以上职责冲突时，另立 CR 拆 orchestrator。 |

## ADR-WT-005：CR-047 保持一个 HLD，Story 在 CP3 后拆分

| 字段 | 内容 |
|---|---|
| 状态 | accepted / CP3 architecture review |
| 背景 | 当前范围有 7 个 Story，触发“评估拆分 HLD”的数量信号；但它们共享同一 workflow-truth 目标、四项架构决策、一个 release gate 和强顺序依赖。 |
| 决策 | CP3 使用一个 CR-scoped HLD；CP3 批准后按 FEAT-WT-01..05 拆 Feature design 和 7 个 Story 设计证据。 |
| 推荐理由 | 防止 State/Doctor/guardrail 各自作出冲突的 canonical/历史/严重度决定，同时允许下游按文件 owner 并行。 |
| 备选 | 分成 truth/routing、quality、delivery 三份 HLD；局部更短，但会重复 ADR 和跨 HLD gate。 |
| 风险 | 单 HLD 过大或局部变更互相等待。 |
| 缓解 | HLD 只保留共享 contract；实现细节下沉 Feature/LLD；Development Plan 明确 3 Waves 和文件 owner。 |
| 切换条件 | 若某 Feature 获得独立 release/approval/rollback/owner、共享 ADR 不再成立，或 ST-WT-007 必须修改任何 CR-046 protected original，则立即停止当前 Story，在独立子 CR/HLD 中声明原 hash、replacement/supersession、审批人、风险与对 closed 状态的影响。 |

## ADR-WT-006：验证结论上限按证据层级和继承风险叠加计算

| 字段 | 内容 |
|---|---|
| 状态 | accepted / CP3-DQ-06 |
| 背景 | 用户要求不使用子 Agent，当前只存在已批准的 inline fallback；CR-046 另有 receipt、独立 QA、token telemetry、real pilot、working-tree-only 五项 OPEN 风险。 |
| 决策 | 有效 inline fallback 且其他验证满足时，CR-047 Story CP7 最高 `PASS_WITH_RISK`；既无独立 QA 也无有效 fallback 证据时为 `BLOCKED`。任一 CR-046 继承风险仍 OPEN 时，CP8 最高 `READY_WITH_RISK`。各风险上限独立、叠加计算。 |
| 禁止声明 | 不得声称 `independent-QA-verified`、`platform-attested` 或 `runtime-profile-verified`；fixture 拒绝伪造不等于平台签发 receipt。 |
| 备选 | 若要求 `PASS/READY`，须在 CP7 前解除不使用子 Agent 约束并取得可验证独立 QA/平台证据，同时逐项关闭其他继承风险；否则停止而非抬高结论。 |
| 风险 | 把一个风险关闭误读为所有上限解除，或把用户授权 inline 执行写成独立验证。 |
| 回退 / 切换 | 新证据到达时只重新计算对应风险维度；未关闭的其他维度继续限制 CP7/CP8。 |

## 决策一致性矩阵

| ADR | Requirement | Use Case | Blueprint Feature | 验证场景 |
|---|---|---|---|---|
| ADR-WT-001 | REQ-WT-001..003, NF001..002 | UC-WT-001 | FEAT-WT-01 | TC-WT-001 |
| ADR-WT-002 | REQ-WT-004..005, C001 | UC-WT-002 | FEAT-WT-02 | TC-WT-002 |
| ADR-WT-003 | REQ-WT-006..008,017, C002 | UC-WT-003,007 | FEAT-WT-03 | TC-WT-003,007 |
| ADR-WT-004 | REQ-WT-009..014, NF003 | UC-WT-004..006 | FEAT-WT-04,05 | TC-WT-004..006 |
| ADR-WT-005 | REQ-WT-001..017 | UC-WT-001..007 | FEAT-WT-01..05 | CP4 DAG/ownership precheck |
| ADR-WT-006 | REQ-WT-015..016, C002 | UC-WT-007 | FEAT-WT-05 | TC-WT-007 verdict-ceiling fixture |

## Gotchas

- “Doctor 退出 0”不等于没有 warning；CP8 必须披露 warning count 和 release ceiling。
- generated root wrapper 可以被校验，但不能被 clean-clone checker 当作入库前置文件。
- archive/cold 是读取分类，不是删除许可；任何历史路径变更都要先验证 ledger/ref/hash 可追溯。
- `B0_pre` 不是固定通过数；`B0_cp7` 增长也不是自动合法，active/default-required 超预算仍然阻断。
- CR-046 与 CR-047 对象同目录混居；protected-object 身份不得退化为路径前缀匹配。
- CP3 接受 ADR 不等于允许实现；全量 Story 设计证据仍必须经过 CP5。
