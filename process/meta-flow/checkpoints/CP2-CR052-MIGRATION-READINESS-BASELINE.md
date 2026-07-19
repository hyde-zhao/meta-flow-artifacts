---
checkpoint_id: "CP2-CR052-MIGRATION-READINESS-BASELINE"
checkpoint_name: "CR-052 Migration Readiness Hardening Product Baseline"
type: "auto_then_manual"
status: "changes_requested"
owner: "host-orchestrator"
created_at: "2026-07-19T06:07:27Z"
reviewed_by: "user"
reviewed_at: "2026-07-19T08:44:54Z"
auto_check_result: "process/checks/CP2-CR052-REQUIREMENTS-BASELINE.result.json"
context_ref: "process/context/CP2-CR052-REQUIREMENT-CONTEXT.yaml"
machine_context_ref: "process/context/CP2-CR052.context.json"
decision_brief_profile: "full-high-risk"
target:
  phase: "requirement-clarification"
  next_phase_if_approved: "solution-design"
  artifacts: ["process/docs/product/USE-CASES.md", "process/docs/product/REQUIREMENTS.md", "process/docs/product/SCENARIOS.yaml", "process/docs/product/TEST-MATRIX.md", "process/docs/product/STORY-MAP.md", "process/docs/product/MVP-SCOPE.md", "process/docs/product/RELEASE-SLICES.md", "process/docs/product/BACKLOG.md"]
---

# CP2 — CR-052 Migration Readiness Hardening 产品基线人工审查

> 状态说明：用户在人工审批前提供了新的核心诉求，本检查点于 2026-07-19 标记为 `changes_requested`。以下 Decision Brief 只保留为旧 v1.1 审计证据，不得作为 vNext CP2 批准或实现输入。新检查点将在 meta-pm 完成 vNext 产品基线后生成。

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 豁免项 | 说明 |
|---|---|---:|---:|---|
| `process/checks/CP0-CR-052-BOOTSTRAP.result.json` | PASS | 0 | 0 | legacy process 路由健康；source 分支从 fresh `origin/main` exact OID 本地创建；未创建 artifact integration/worktree/ref |
| `process/checks/CP1-CR052-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | 0 | CR-052 新增 7 UC、31 REQ、22 TC，六类场景和失败恢复闭环完整 |
| `process/checks/CP2-CR052-REQUIREMENTS-BASELINE.result.json` | PASS | 0 | 0 | 7 候选 Story、6 发布切片、10 项量化指标和 DQ52-1..10 已形成正式决策输入 |
| `process/checks/CP2-DISCUSSION-CHECKPOINT.json` | PASS | 0 | 0 | 4 个 Scenario Gray Areas 已分类；SGQ-MR-001 记录真实用户输入，未伪造额外回答 |

> 自动预检 `PASS` 只表示产品基线具备人工审查条件，不表示 CP2 已批准，也不授权源码实现、真实 artifact 布局 mutation、真实 Git ref/worktree/link 操作、commit 或 push。

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-052 只把 CR-051 的库级契约补成可安全执行、可失败恢复、可持久审计并能在隔离临时拓扑完成真实 mutation E2E 的迁移准备能力；真实 meta-flow artifact 布局迁移严格留给 CR-053。 |
| 推荐动作 | `approve`：接受 DQ52-1..10 的推荐方案，批准本产品基线进入 CP3/HLD。 |
| approve 后会发生什么 | Host Orchestrator 将调度 meta-se，形成 BLUEPRINT、HLD、ADR 和 CP3 Decision Brief；CP3 必须冻结 schema v2、三态 health、两阶段 evidence、transitional CP0、seed-prune、typed authorization、失败恢复和 audit-tail OID 契约。 |
| approve 不授权什么 | 不授权 Story/LLD 跳门、不授权源码实现、不授权真实 meta-flow-artifacts 的 integration/worktree/ref/link/layout mutation，不授权 commit/push/shared-main 写入，不授权 force/tag/reset/rebase/orphan，也不授权凭据、runtime、production、publish/live/trading 操作。 |
| 不确认会阻塞什么 | CR-052 保持在 requirement-clarification；不得进入 CP3、不得生成正式 Story/LLD、不得修改源码或执行任何真实仓迁移及发布动作；CR-053 也不得启动。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR052-REQUIREMENT-CONTEXT.yaml` |
| 机器 context pack | `process/context/CP2-CR052.context.json` |
| capsule 状态 | `ready-for-human-gate` |
| read_profile | `compact` |
| 默认读取策略 | capsule first；默认只消费 `must_read` 和 `allowed_reads`，完整 CR、STATE、archive、旧 Story 与 sibling project 均 deny-default |
| 全文档读取 | Host 在 CP0 做过审计扩展；meta-pm 因 CR summary 不含完整 DQ 语义，以 `deep_review` 读取完整 `CR-052.md`，由 Host 补记 READ-EXPANSION ledger |
| 最小事实 | CR-052 增量为 UC=7、REQ=31、TC=22、候选 Story=7、Slice=6、成功指标=10、SGA=4、SGQ=1；全量产品基线为 UC=35、REQ=161、TC=97、候选 Story=43 |
| 路由注意事项 | `process` 为外置软链接；机器 context check 必须显式传入源码项目根，避免对 resolved artifact 路径误推断项目根 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| current state | `process/state/STATE.current.json`、`process/current/CURRENT.json` | scanned | 0 | 0 | 轻量状态；本 checkpoint 开门时由 Host 回写 gate/context |
| 用户显式输入 | 当前对话、`process/REQUEST.md`、SGQ-MR-001 | scanned | 1 | 0 | “按照你的计划实施”确认启动和场景基线，不替代正式 CP2 决策，不授权真实仓 mutation/publication |
| CR 影响分析 | `process/changes/CR-052.md`、CR summary | scanned | 10 | 10 | DQ52-1..10 均为 `decision-item`，无未分类 blocker |
| meta-pm 回交 | `process/handoffs/CR052-CP1-CP2-META-PM-RETURN-SUMMARY.md` | scanned | 10 | 10 | 产品语义与类型齐全，正式决策由 Host 一次性收集 |
| discussion | discussion log、discussion checkpoint | scanned | 4 | 8 | 四个灰区映射 DQ52-1/2/3/4/5/8/9；DQ52-6/7/10 来自运行授权、治理与 follow-up |
| 八份产品基线 | `process/docs/product/**` 当前 CR-052 增量 | scanned | 10 | 10 | In/Out/Deferred、风险、验收与追踪矩阵一致 |
| 自动预检 | CP0、CP1、CP2 result | scanned | 0 | 0 | blockers=0、waivers=0，result schema 校验通过 |
| 主编排器独立复核 | result-check、context check、YAML/JSON、引用、diff 边界 | scanned | 0 | 0 | 无新增人工问题；正式 gate 尚未批准 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | DQ52-1 范围拆分、DQ52-6 运行授权边界、DQ52-8 临时真实 mutation 风险接受、DQ52-10 follow-up 闭环 |
| 高风险策略确认 | 5 | DQ52-2/3/4/5/9 涉及 schema、自举、证据真相源、受控删除与逐动作安全授权，CP2 确认方向、CP3 冻结可执行契约 |
| agent 默认处理 | 1 | DQ52-7 的 scoped/fingerprint 具体输出、命令参数和历史 fingerprint 编码由设计/实现选择，但不得扩大历史整改范围 |
| 仅审计记录 | 9 | fresh OID、legacy route、branch-open dirty 复现、19 个 CR lifecycle 历史错误、1 个 HANDOFF-LEDGER 历史 `context_ref` 错误、无 artifact ref/worktree/link、无 commit/push、meta-pm dispatch、产品计数 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ52-1 | scope | 是否继续拆分 CR-052 能力建设与 CR-053 真实迁移？ | 保留拆分；CR-052 只实现能力并在隔离临时三并列 Git 拓扑做真实 mutation E2E，CR-053 才迁移真实布局 | 合并到 CR-052；或把 CR-052 降为 dry-run/mock-only | 拆分使授权、回滚和审计边界最清晰，但多一个 CR；合并周期短却把未验证能力直接用于真实仓；mock-only 风险低但不能声明 migration-ready | 合并会重现 bootstrap paradox 和范围爆炸；mock-only 会留下关键真实性缺口 | CP3 若证明 CR-053 第一写无法消费 CR-052 coordinator，再回 CP2 评估把最小 transitional operation 前移；不得静默扩大范围 |
| DQ52-2 | architecture | 外部 sibling 拓扑怎样表达且不放宽路径穿越？ | schema v2 新增 `workspace_parent=project_root.parent` anchor，legacy/v1/v2 dual-read、v2-only write | schema v2 使用显式 `workspace_root`；或继续 v1 并新增字段 | `workspace_parent` 对当前同父目录拓扑最小且可移植；显式 root 支持异构拓扑但配置和校验面更大；留在 v1 会造成读写语义分裂 | 锚点歧义会造成 path escape、跨项目写或旧 reader 误判 | 当 project/control/sibling 不共享唯一父目录，切换显式 `workspace_root`；任何解析不唯一或越界都 fail closed |
| DQ52-3 | architecture | CR-053 如何跨越首次 bootstrap 自举循环？ | 采用显式 transitional CP0：pre-CR immutable intent/receipt，CP0 审计导入，third-state/resume，由 CR-052 交付 coordinator | 追求 native-first；或 CR 外单独 Change Package | transitional 协议诚实表达首次写入并可恢复；native-first 表面整洁但无法解释先有 route 还是先有 CR；独立 Change Package 增加治理对象 | 错误自举可能制造无账本 mutation、重复 ref 或不可恢复中间态 | coordinator 无法以幂等状态机覆盖全部中间态时停止 CR-053，回 CP3/CP2；不得用手工隐式步骤填洞 |
| DQ52-4 | architecture | leg/aggregate 结果怎样跨进程持久化且避免过程仓永久 dirty？ | 两阶段 evidence：out-of-band content-addressed execution store 加 project-local immutable audit-tail receipt | 纯 tracked project-local store；或仅内存 store | 两阶段兼顾跨进程恢复和稳定 target OID，复杂度较高；纯 tracked 更简单但产生 aggregate 后尾提交与 OID 自引用；内存方案不能恢复 | 真相源不清会导致 aggregate 无法重放、证据漂移或功能完成后无法 clean | CP3 必须冻结 durability、信任边界、parent/target OID 和 reader restart 契约；无法证明时退回纯 tracked 并显式接受 evidence-tail commit |
| DQ52-5 | implementation | integration 从 shared-main 继承 sibling seed 后如何收紧 owned path？ | 先执行 manifest-bound seed prune，只允许 exact inherited sibling deletion；receipt 验证后再激活 steady `docs`/`process` ownership | 保留 seed 并在 schema 中加入 exclusions；或把 shared-main 清理前置 | prune 后 steady 模型最简单且不改 shared main，但受控删除风险高；exclusions 增加长期复杂度；前置清理会影响其他项目 | 错删 sibling 或过早激活 ownership 会造成跨项目破坏或 route conflict | pre/post manifest、OID/hash/mode 和 allowlisted deletion 任一不匹配即停止并保留现场；不得自动 reset/clean/force |
| DQ52-6 | runtime_authorization | CR-052 当前允许哪些真实仓写入和 publication？ | 允许 legacy `process/meta-flow/**` 本地审计写入；commit 与 push 均延后到 CP5/CP8，按 exact OID 和具体 ref 单独授权 | 全程 local-only 不 commit/push；或现在授权普通 commit/push | 推荐方案保留过程证据并把不可逆 publication 推迟到证据成熟时；local-only 最安全但跨设备恢复弱；立即 push 审计过早且无法撤回 | 过程仓持续 dirty 会阻断现有 branch-open；过早 publication 会把未批准设计写入远端 | CP5/CP8 若无独立授权则保持 local-only；任何 fresh OID 漂移都使授权失效；本次 approve 不授权 commit/push |
| DQ52-7 | scope | 全局历史治理错误是否并入 CR-052？ | 实现 scoped/delta governance 验收，冻结 19 个 CR lifecycle 错误和 1 个 HANDOFF-LEDGER `context_ref` 错误，要求 CR-052 新增 fingerprint 为 0；历史修复另开 follow-up | 本 CR 修复全部历史错误；或忽略全局基线 | scoped/fingerprint 保持整改聚焦且可证明不倒退；全修复扩大范围并污染验收；忽略基线会掩盖新增错误 | 全局 FAIL 若直接作为门会使本 CR 永远不可关闭；完全忽略又可能引入回归 | 只有历史错误实质阻断 scoped contract 时才通过正式 scope 变更扩大；否则保持 follow-up |
| DQ52-8 | risk_acceptance | 是否接受在隔离临时仓执行真实 destructive-like mutation E2E？ | 接受：临时三并列 Git 仓和 local bare remote 内强制执行 ref/worktree/prune/link/resume/partial-failure E2E；真实 meta-flow/artifacts 仅 dry-run/read-only | 临时环境也只 dry-run/mock；或允许真实仓 pilot | 临时真实 E2E 能证明文件系统和 Git 语义且不触碰真实数据；mock-only 无法证明关键风险；真实仓 pilot 在能力未就绪前风险过高 | 测试仍会创建、删除临时 ref/worktree/link；若隔离边界错误可能误触真实路径 | runner 必须验证临时根、plan digest、expected OID 和 local remote；任何真实路径命中立即 fail closed；无法执行则 CP8 不得标 READY |
| DQ52-9 | security | migration runner 的 mutation 授权粒度如何设计？ | ref、worktree、seed-prune、link、push 分动作 typed authorization；单次有效，绑定 project/repo/root/ref/OID/plan digest | 一个总授权覆盖整次迁移；或仅交互确认 | 分动作授权最小化重放面并支持中断恢复，证据和操作较多；总授权简单但权限过宽；交互确认难以自动审计和 resume | 授权重放、参数替换或跨项目复用会造成越权 mutation | 任一绑定字段变化、attempt 已消费或 fresh observation 不符即拒绝；禁止隐式续期或降级成布尔开关 |
| DQ52-10 | follow_up_tracking | CR-052 不处理的合规与迁移工作怎样闭环？ | 建立有 owner、触发条件和授权边界的 follow-up：CR-053 真实迁移、shared-main README-only、ptm-team/ptm-atomic/quant-lab 分项目迁移、历史 CR 错误治理 | 把 shared README 纳入 CR-052；或仅写无 owner backlog | 显式 follow-up 保持本 CR 目标单一且可追踪；立即改 shared main 扩大 publication 边界；无 owner backlog 容易永久遗忘 | 遗留项未闭环会形成长期规范漂移；越早改 shared main 会影响未迁移项目 | CP8 必须输出 owner、restart condition、状态和下一 CR 候选；缺失则 CR-052 不得关闭 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ52-1..10 全部推荐方案并进入 CP3 |
| 备选方案 | `修改: <具体修改点>` 返回 requirement-clarification 修订相应 DQ；或 `reject` 停止 CR-052 |
| 影响维度 | 范围、schema 兼容、安全边界、Git/文件系统副作用、失败恢复、证据可信度、治理可关闭性和 CR-053 可执行性 |
| 优劣分析 | 推荐组合把真实迁移和 publication 后置，同时要求临时真实 E2E，能以较低生产风险获得较强迁移就绪证据；代价是需要 schema v2、两阶段 evidence、transitional coordinator 和逐动作授权等完整设计 |
| 风险与回退 | CP3 发现契约不可实现则回 CP2；CP5 发现 LLD 不可实施则回 CP3；临时 E2E 未通过则 CP8 NOT_READY；任何真实仓路径或 OID 不符都 fail closed |
| 用户需决策事项 | DQ52-1、DQ52-2、DQ52-3、DQ52-4、DQ52-5、DQ52-6、DQ52-7、DQ52-8、DQ52-9、DQ52-10 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR-052 已 bootstrap 且 route plan 为 architecture-major | PASS | CR-052、CP0 route plan/result | 通过 |
| source 分支基于 fresh `origin/main` exact OID，本地且无 publication | PASS | `cr/cr-052-migration-readiness-hardening`、`435434bf...` | 通过 |
| meta-pm 真实子 Agent 调度和回交完成 | PASS | handoff、return summary、dispatch/handoff ledger | 通过 |
| 八份产品基线完成增量更新并保留旧 ID/修订历史 | PASS | `process/docs/product/**` | 通过 |
| Scenario Gray Areas 和至少一条真实 SGQ 完整 | PASS | discussion log/checkpoint、SGQ-MR-001 | 通过 |
| CP1 自动检查通过 | PASS | CP1 result；blockers=0、waivers=0 | 通过 |
| CP2 自动预检通过 | PASS | CP2 result；blockers=0、waivers=0 | 通过 |
| 真实布局、真实 artifact ref/worktree/link、commit/push mutation 为 0 | PASS | git/OID/route preflight 与授权边界 | 通过 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | CR-052/CR-053 范围与授权边界清晰 | 待人工确认 | DQ52-1/6/8/10、MVP-SCOPE |  |
| 2 | schema v2 external anchor 方向不放宽 traversal | 待人工确认 | DQ52-2、REQ-MR、TC-MR |  |
| 3 | transitional bootstrap 可审计、可恢复且不伪称 native-first | 待人工确认 | DQ52-3、failure-recovery 场景 |  |
| 4 | durable evidence 真相源和 audit-tail 风险有 CP3 冻结条件 | 待人工确认 | DQ52-4、REQ-MR-NF |  |
| 5 | seed-prune 与 steady ownership 顺序和失败路径清晰 | 待人工确认 | DQ52-5、permission/failure 场景 |  |
| 6 | scoped CR check 不掩盖新增错误且不扩大历史整改范围 | 待人工确认 | DQ52-7、frozen fingerprint |  |
| 7 | 临时真实 E2E 与真实仓禁止边界清晰 | 待人工确认 | DQ52-8、MVP-SCOPE |  |
| 8 | typed authorization 最小权限、单次和绑定字段清晰 | 待人工确认 | DQ52-9、安全约束 |  |
| 9 | follow-up 有 owner、触发条件和关闭门 | 待人工确认 | DQ52-10、BACKLOG |  |
| 10 | `approve` 只进入 CP3，不授权实现、commit、push 或真实迁移 | 待人工确认 | 审批者摘要、不授权项 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| P0 产品基线自动 blocker 为 0 | PASS | CP2 result |  |
| DQ52-1..10 全部得到人工结论 | PENDING | 本 checkpoint 人工审查结果 |  |
| 用户明确允许进入 solution-design | PENDING | 本人工门结论 |  |
| `ready_for_design=true` 只在批准后写入 | PENDING | CR/state/checkpoint |  |
| 真实仓 layout/ref/worktree/link/commit/push mutation 为 0 | PASS | 当前执行边界 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CR-052 产品基线 | `process/docs/product/**` 八份文件 | 自动预检 PASS |  |
| CP1 result | `process/checks/CP1-CR052-USE-CASE-COMPLETENESS.result.json` | PASS |  |
| CP2 result | `process/checks/CP2-CR052-REQUIREMENTS-BASELINE.result.json` | PASS |  |
| Discussion / SGQ | discussion log、discussion checkpoint | PASS |  |
| Context Capsule | YAML capsule、machine context pack | ready-for-human-gate |  |
| Meta-PM Return | `process/handoffs/CR052-CP1-CP2-META-PM-RETURN-SUMMARY.md` | returned |  |

## 人工审查结果

- 结论：`pending`
- 审查人：待用户填写
- 审查时间：待用户填写
- 决策结果：DQ52-1..10 待确认
- 修改意见：无；等待用户回复
- 授权边界：即使 `approve`，也只允许进入 CP3；不授权源码实现、真实布局或 Git ref/worktree/link mutation、commit、push、shared-main 写入及任何被禁止操作。

## 可接受回复

- `approve`
- `修改: <具体修改点>`
- `reject`
