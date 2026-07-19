---
status: baseline
version: "2.0"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
active_change_ref: "CR-051"
ready_for_design: false
approval_context: "CR-051 CP2 changes_requested R3 当前候选基线：一个逻辑 CR 使用异构 source/artifact 双 leg；source leg 从 fresh 源码默认分支创建并回到该默认分支，artifact leg 从 fresh 项目 integration 创建并只回到同一 integration，绝不在 per-CR 生命周期 refresh/merge/update artifact shared main；单一聚合门按 BLOCKED > FAIL > IN_PROGRESS > PASS，且仅全部必需 leg PASS 才完成；integration 缺失时从 fresh origin/main exact OID create-only 初始化；shared main↔integration 同步完全在 CR 外人工执行。R3 产品基线仍待 CP2 总体 approve，ready_for_design=false；不授权 HLD、Story、LLD、源码实现、真实 artifact 文件/软链接/worktree/branch/ref mutation、remote publication、credentials、runtime、production write、publish 或 trading。"
source_use_cases:
  - UC-PG-001
  - UC-PG-002
  - UC-PG-003
  - UC-PG-004
  - UC-PG-005
  - UC-PG-006
  - UC-PG-007
  - UC-EI-001
  - UC-EI-002
  - UC-EI-003
  - UC-EI-004
  - UC-EI-005
  - UC-WT-001
  - UC-WT-002
  - UC-WT-003
  - UC-WT-004
  - UC-WT-005
  - UC-WT-006
  - UC-WT-007
  - UC-GB-001
  - UC-GB-002
  - UC-GB-003
  - UC-GB-004
  - UC-AW-001
  - UC-AW-002
  - UC-AW-003
  - UC-AW-004
  - UC-AW-005
source_plan: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-STATE-ENFORCEMENT-IMPLEMENTATION-PLAN-2026-07-02.md"
---

# Meta Flow 项目治理与状态强制需求

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.5 | 2026-07-14 | host-orchestrator-inline / meta-se | CP3 R2 度量精确化：同步修订 REQ-WT-006 与 RA-WT-004，把固定“21”重述为 `B0_pre` 历史快照，并以 CP7 动态 `B0_cp7`、可解释 delta、active/default-required blocker 作为验收契约；不改变 UC、Story、范围或 CP2 批准结论 | 原文档增量更新；无需重开 CP2 |
| 1.6 | 2026-07-15 | host-orchestrator inline fallback | 为 CR-050 增量加入原生 Git 双仓 CR branch open/publish/finish 契约、fail-closed merge 证明、幂等与 partial failure；保留全部既有 REQ/RA，等待 CP2 确认五项产品边界 | 原文档增量更新 |
| 1.6.1 | 2026-07-16 | host-orchestrator inline fallback | 记录 CR-050 CP2-DQ-01..05 推荐方案获批并将 `ready_for_design=true`；不修改 REQ/RA/范围，继续保留实现和真实远端 mutation 不授权边界 | CP2 状态同步 |
| 1.7 | 2026-07-16 | host-orchestrator inline fallback | 用户批准将独立两仓 fast-forward-only merge 纳入 CR-050；新增 REQ-GB-011..014、REQ-GB-C004、REQ-GB-NF003、RA-GB-005..006，并把里程碑调整为 publish→merge→cleanup；`ready_for_design=false`，等待 CP2 R2。 | 原文档增量更新；保留 v1.6.1 与全部既有 REQ/RA |
| 1.8 | 2026-07-17 | meta-pm | 为 CR-051 增量新增 REQ-AW-001..017、约束/NFR/风险和里程碑；将 CR-050 “artifact 整仓工作树”限定为 legacy/dedicated-artifact 适用前提，shared-artifact 模式改为当前 project worktree；保留全部既有 REQ/RA。 | 原文档增量更新；不改写 CR-050 历史正文 |
| 1.9 | 2026-07-17 | meta-pm | CR-051 CP2 R2：按 DQ-01..03 用户决策修订 REQ-AW-003..012/016、NFR、风险与里程碑，冻结长期项目 integration 分支、短期 CR 分支、shared main、显式 merge-main refresh 与 existing-control+sibling-worktree 边界；REQ/RA/里程碑 ID 均不变。 | 原文档增量更新；R1 保留为 superseded 自动预检证据 |
| 2.0 | 2026-07-18 | meta-pm | CR-051 CP2 R3：保持 27 个 REQ-AW ID 不变，修订为异构 source/artifact 双 leg、单一聚合门、integration create-only 初始化与 CR 外人工 main/integration 同步；CP2-DQ-02 保留历史并由 DQ-04 supersede。 | 原文档增量更新；R1/R2 历史追溯保留，R3 为当前候选基线 |
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活、CR-036 暂停未完成和 CP2 pending 不授权实现的状态语义 | 小范围状态语义同步 |
| 1.2 | 2026-07-11 | meta-pm | 为 CR-046 增量增加 evidence-integrity、replayability、telemetry 和 CR-163 append-only pilot 需求；保留全部 REQ-PG ID | 原文档增量更新 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：新增 REQ-EI-019..023，覆盖 compaction 语义保持、通用 post-close correction、机器 audit report、null-provenance dogfooding 与 dispatch 证据限制；保留全部既有 REQ ID | 原文档增量更新 |
| 1.4 | 2026-07-13 | meta-pm | 为 CR-047 增量增加 workflow truth、canonical CR tracking、artifact/docs 路由、Doctor、clean-clone guardrail、Ruff、非交互安装和 CR-046 状态收敛需求；记录修复前实测基线并保留全部既有 REQ ID | 原文档增量更新 |
| 1.0 | 2026-07-02 | meta-pm | 基于已批准实施计划提取产品需求基线 | 初始化长期可追踪需求基线 |

## 功能需求

> CR-051 适用性规则：`REQ-GB-*` 的安全输入、显式授权、逐仓结果与 fail-closed 契约继续有效，但 `REQ-GB-011..014` 的 paired-default merge 仅适用于 source + dedicated/legacy artifact 两仓各自 default 分支的前提。shared-artifact 模式显式覆盖该前提：source leg 的目标仍是源码默认分支，artifact leg 的目标是当前项目 integration；任何要求 artifact default/main merge 的旧契约在该模式下不适用。control checkout 或整个 shared working tree 均不得成为当前项目写入面。

| ID | 需求描述 | 优先级 | 验收条件 | 来源场景 |
|---|---|---|---|---|
| REQ-PG-001 | 系统必须为 `STATE.current.json` 建立 allowlist schema，仅允许声明字段写入。 | P0 | Given 待写入 patch 包含未声明顶层 key When 执行 current state 写入 Then audit 模式输出 WARN，enforce 模式拒绝写入并返回可追踪错误。 | UC-PG-001 |
| REQ-PG-002 | 系统必须为 `STATE.current.json` 合法字段执行字段级预算校验。 | P0 | Given `next_action`、`source_refs` 或 `open_risks` 超过预算 When 执行写入 Then 写入失败，错误指出字段和预算原因。 | UC-PG-001 |
| REQ-PG-003 | 系统必须提供受控 current state 更新入口，支持 deep-merge patch、actor 和 reason。 | P0 | Given 调用 `update_current_state(project_root, patch, actor, reason)` When patch 合法 Then 完整 state 写入前后均通过 schema 和预算校验。 | UC-PG-001 |
| REQ-PG-004 | 内部 current state 更新路径不得绕过受控 writer。 | P0 | Given CR lifecycle 或 state-router 需要更新 current state When 执行更新 Then 不存在直接 `write_text()` 写 `STATE.current.json` 的路径。 | UC-PG-001 |
| REQ-PG-005 | Agent / Skill 写契约必须禁止直接编辑 `STATE.current.json`。 | P0 | Given 交付规则和 state-router 文档更新完成 When 审查 agent / skill contract Then 明确声明合法更新入口和重型状态落盘位置。 | UC-PG-001 |
| REQ-PG-006 | `STATE.current.json` allowlist 必须包含 `project_state_ref`，用于引用项目级状态。 | P0 | Given P1 项目状态启用 When 写入 `project_state_ref` Then current state schema 校验通过且字段预算生效。 | UC-PG-002 |
| REQ-PG-007 | 系统必须创建 `process/project/` scaffold。 | P1 | Given 初始化或迁移 workspace When 执行 bootstrap Then `process/project/` 被创建且测试覆盖该行为。 | UC-PG-002 |
| REQ-PG-008 | 系统必须提供 refs-only 的 `PROJECT.current.json`，并限制大小。 | P1 | Given 项目状态包含 roadmap / milestone / health / deferred refs When 执行 project state check Then 文件不超过 16KB 且不包含 roadmap 全文或 deferred 详情。 | UC-PG-002 |
| REQ-PG-009 | 系统必须通过 `PROJECT-SCALE.yaml` 表达 `lite / standard / full` 三档项目规模和 gate profile bias。 | P1 | Given 项目设为 `full` When 读取 project scale Then 可看到默认 gate profile bias 和 reason，且未修改 `GATE-PROFILES.json`。 | UC-PG-003 |
| REQ-PG-010 | `feature_refs` 必须引用标准 `FEATURE-REGISTRY.yaml` 中已注册 feature ID。 | P1 | Given 新 CR 或 migration 声明 `feature_refs` When 运行 feature check Then 未注册 ID 被报告为 blocked finding 或 follow-up。 | UC-PG-004 |
| REQ-PG-011 | `capability_refs` 必须引用标准 `CAPABILITY-STATUS.yaml` 中已注册 capability ID。 | P1 | Given 新 CR 或 migration 声明 `capability_refs` When 运行 capability check Then 未注册 ID 被报告为 blocked finding 或 follow-up，不被自动创建。 | UC-PG-004 |
| REQ-PG-012 | 系统必须将 `impact_surface` 限定为治理面枚举，并将路径写入 `affected_paths`。 | P1 | Given 输入包含 `process/checks` 或 `trading/strategy_runner` When 运行 migration Then 路径进入 `affected_paths`，可映射治理面进入 `impact_surface`。 | UC-PG-005 |
| REQ-PG-013 | 系统必须为历史 impact surface 漂移生成 migration report，避免普通 `cr check` 对历史数据刷屏。 | P1 | Given 历史 CR 含路径类 impact surface When 运行常规 `cr check` Then 不刷屏；When 运行 migration report Then 输出可追踪 finding。 | UC-PG-005 |
| REQ-PG-014 | Roadmap refresh 必须使用独立 result schema 和 checker，不复用 CP result checker。 | P1 | Given 生成 `ROADMAP-REFRESH-<id>.result.json` When 运行 roadmap-refresh checker Then 校验 refresh schema、decision、source、machine_updates、follow-up 字段。 | UC-PG-006 |
| REQ-PG-015 | Roadmap refresh decision 必须使用固定枚举。 | P1 | Given refresh 完成 When 写入 result Then decision 只能为 `NO_CHANGE`、`UPDATED`、`UPDATED_WITH_DOC_IMPACTS`、`BLOCKED` 或 `FAILED`。 | UC-PG-006 |
| REQ-PG-016 | Roadmap refresh cascade 自动写入范围必须限制在过程归档库。 | P1 | Given refresh 发现 quant-lab 发布库文档陈旧 When 执行 cascade Then 发布库文件不被自动修改，陈旧项进入 `must_check`、`stale_items` 或 `follow_up_candidates`。 | UC-PG-006 |
| REQ-PG-017 | Roadmap refresh 事件必须写入 `GATE-LEDGER` 或等价 CR 相关 ledger。 | P1 | Given refresh result 写入成功 When 追加 ledger Then ledger event 包含 refresh gate、status 和 result ref。 | UC-PG-006 |
| REQ-PG-018 | FU-RF、SP-RF、RA-RF 候选必须被跟踪模板和检查器接受。 | P1 | Given roadmap refresh 生成 follow-up When 运行 cr-tracking Then `FU-RF001-001`、`SP-RF001-001`、`RA-RF001-001` 被识别且不写入 `RELEASE-CONTEXT`。 | UC-PG-006 |
| REQ-PG-019 | 系统必须提供 project stale check，识别跨对象语义陈旧。 | P1 | Given project state 进入 runtime 或 paper/live 阶段但 HLD / TEST-STRATEGY 仍停留 backtest-only When 运行 stale check Then 输出 stale finding 和 follow-up 候选。 | UC-PG-006 |
| REQ-PG-020 | quant-lab 迁移必须清理污染的 `STATE.current.json` 并通过 state check。 | P2 | Given quant-lab current state 含 forbidden / unknown / oversized 字段 When 执行迁移 Then 清理后通过 state check，并保留迁移报告。 | UC-PG-007 |
| REQ-PG-021 | quant-lab capability 来源必须归一到标准 registry。 | P2 | Given quant-lab capability 线索分散在 CR、代码和研究注册文档中 When 执行迁移 Then 真实 capability 被核查并进入标准 registry 或 blocked follow-up。 | UC-PG-007 |
| REQ-PG-022 | quant-lab 迁移必须输出 roadmap stale report 和 FU-RF 候选，且不得自动改发布库。 | P2 | Given quant-lab 发布库存在陈旧 HLD / TEST / release 文档 When 执行迁移 Then 只输出 stale report 和 FU-RF 候选，不修改发布库文件。 | UC-PG-007 |
| REQ-EI-001 | 系统必须校验 checkpoint、human gate、state 与 ledger 的时序不变量。 | P0 | Given gate/result/state 事件违反前置或因果顺序 When 运行 chronology checker Then 非法 fixture 100% 被拒绝，并定位冲突事件与规则。 | UC-EI-001 |
| REQ-EI-002 | 条件式批准必须保留条件、满足证据和生效事件，未满足条件时不得视为最终批准。 | P0 | Given human approval 含未满足条件 When state transition 尝试进入后续阶段 Then transition 被拒绝；When 条件证据满足并追加生效事件 Then 可重放为有效批准。 | UC-EI-001 |
| REQ-EI-003 | 每个 execution/dispatch attempt 必须具有唯一 ID、生命周期状态和 terminal closure。 | P0 | Given spawn/resume/send_input attempt 已开始 When 工作结束、失败或被替代 Then attempt 必须进入 terminal status，未关闭 attempt 被 checker 阻断。 | UC-EI-002 |
| REQ-EI-004 | retry 与 supersession 必须显式引用被替代 attempt，并保持完整链路。 | P0 | Given attempt 失败后重试 When 写入 retry event Then 新 attempt 引用旧 attempt，旧 attempt terminal，且 final chain 无循环和悬空引用。 | UC-EI-002 |
| REQ-EI-005 | Dispatch evidence 必须表达 platform-attested、session-observed、repository-verifiable 或 unavailable 的证明层级。 | P0 | Given 平台提供可验证 receipt When 检查 dispatch Then receipt 与 attempt correlation 可验证；Given 平台不提供 receipt Then attestation status 为 unavailable 且不被提升为 verified。 | UC-EI-002 |
| REQ-EI-006 | CP result 必须记录 check attempt、适用输入 artifact hash 和 supersession 关系。 | P0 | Given 适用 CP result 被生成 When 运行 result checker Then check attempt 与 input hash 覆盖率为 100%，重复检查通过 supersedes 形成无环链。 | UC-EI-001 |
| REQ-EI-007 | 最终 CP result 必须引用最终 terminal dispatch/check attempt，且 terminal outcome 与 CP decision 兼容。 | P0 | Given 多次 attempt 后产生 final result When 运行 correlation checker Then final ref 只指向最终 terminal attempt；不兼容 outcome/decision 被拒绝。 | UC-EI-001, UC-EI-002 |
| REQ-EI-008 | 系统必须校验 result、checkpoint、gate、state 与 ledger 的跨真相源一致性。 | P0 | Given 任一对象在 decision、CR、checkpoint、attempt 或 ref 上冲突 When 运行 consistency check Then 输出冲突对象和字段，且不得以 summary 覆盖机器真相。 | UC-EI-001 |
| REQ-EI-009 | delivered state 必须清空 active refs，并引用存在且属于当前 CR 的 workflow health 生成物。 | P0 | Given workflow 尝试 finalized/delivered When 运行 state check Then active change/story/delegation/gate refs 为 0，health ref 目标存在且 CR attribution 匹配。 | UC-EI-001 |
| REQ-EI-010 | deny-default read expansion 必须记录授权、原因、CR attribution 和 scoped context slicing。 | P0 | Given agent 展开 deny-default 文档 When 运行 read-expansion check Then 授权引用、允许原因、CR ID 与 scope 均可验证；未授权事件数为 0。 | UC-EI-001 |
| REQ-EI-011 | token usage 必须为每条适用记录声明 `measurement_status`。 | P0 | Given agent/checker/context run 被计量 When 生成 usage record Then status 只能表达 measured、estimated/proxy 或 unavailable，适用记录覆盖率为 100%。 | UC-EI-004 |
| REQ-EI-012 | 只有平台报告的 token usage 可标记为 measured；估算与不可用必须独立表达。 | P0 | Given 仅有文本长度估算或无 telemetry When 生成成本报告 Then measured 字段为空，estimate 仅进入 proxy 字段或 status 为 unavailable。 | UC-EI-004 |
| REQ-EI-013 | 成本记录必须可按 CR、阶段、agent/attempt 和 checker run 归属与聚合。 | P1 | Given 多阶段 workflow usage records When 生成 cost report Then 可按 CR/phase/agent/attempt 聚合，且无法归属项显式列出。 | UC-EI-004 |
| REQ-EI-014 | replay report 必须记录 checker version、commit、schema hash、policy hash 和输入 artifact hash。 | P0 | Given evidence 被重放 When 生成 report Then checker provenance 与所有适用 hash 非空；确实不可取得的字段明确标记 unavailable。 | UC-EI-003 |
| REQ-EI-015 | replay report 必须同时保留 as-executed 与 current-replay 结论及差异分类。 | P0 | Given 当前 checker 与历史 checker 结论不同 When 重放 Then 两个结论均保留，差异被分类且不改写历史结论。 | UC-EI-003 |
| REQ-EI-016 | quant-lab CR-163 迁移必须 append-only，仅追加 manifest、correction、supersession 和 replay evidence。 | P1 | Given 迁移前后历史事件集合 When 校验 immutable prefix/hash Then 原事件逐字节不变，新事件仅追加且回链原对象。 | UC-EI-005 |
| REQ-EI-017 | quant-lab CR-163 current-checker replay fixture 的目标必须为 23/23 PASS。 | P1 | Given migration fixture 完成 When 使用已识别 current checker 重放 Then 23 个目标证据全部 PASS，否则逐项输出 blocker。 | UC-EI-005 |
| REQ-EI-018 | CR-046 不得修改 quant-lab lineage 业务实现。 | P0 | Given CR-163 pilot diff When 按受保护业务路径检查 Then lineage contract、recorder、producer、consumer、admission 源码 diff 为 0。 | UC-EI-005 |
| REQ-EI-019 | Ledger compaction/restore 必须保持 event、dispatch、attempt、run 的类型与关系语义，并保留 terminal attempt selection、correction chain 和 workflow-health refs。 | P0 | Given 含重试、修正和 health ref 的 ledger fixture When compact 后 restore/replay Then 标识类型不混淆、关系图和最终 terminal selection 与 compact 前语义等价；任一丢失或 fallback conflation 100% 被拒绝。 | UC-EI-001, UC-EI-002 |
| REQ-EI-020 | 系统必须提供可复用的 CP8 后 append-only correction lifecycle，而不是仅为 CR-163 定制迁移事件。 | P0 | Given closed workflow 需要修正允许字段 When 追加 correction Then 使用 versioned schema，声明允许字段/范围、author、reason、evidence、supersedes，并形成可独立检查的无环 audit trail；非法原位改写 100% 被拒绝。 | UC-EI-003, UC-EI-005 |
| REQ-EI-021 | 系统必须从 ledgers/results 机器生成 provenance-bearing audit report，并分离 event-row、attempt、thread、terminal outcome 与 token measurement metrics。 | P0 | Given 含 retry、多 event rows 和混合 measured/proxy/unavailable usage 的已知 fixture When 生成 audit report Then 各维计数与 fixture 100% 一致，checker provenance 与输入 hashes 非空，且不得从 dispatch 行数推断 attempt/thread 或 token 占比。 | UC-EI-001, UC-EI-004 |
| REQ-EI-022 | CR-046 CP1/CP2 原始 null-provenance results 必须保留为 first-class dogfooding fixtures，并按新 strict profile 失败或报告 legacy/unavailable。 | P0 | Given 原始 R1 CP1/CP2 result 未含 checker_provenance When strict replay/check 执行 Then 原文件 hash 不变，结果不得标 fully replayable；迁移/current result 只能通过 append-only correction 或新结果记录 measured provenance。 | UC-EI-003 |
| REQ-EI-023 | Dispatch 报告必须披露 session-observed 与 repository-verifiable 的独立状态，以及平台 receipt 不可用时的证明上限。 | P0 | Given agent/thread/tool 仅在当前 Codex session 可观察且仓库无平台 receipt When 生成 dispatch evidence/audit report Then 状态为 `session-observed/repository-unverifiable`，receipt 为 unavailable，且不得声称 repository-verified 或 platform-attested。 | UC-EI-002 |
| REQ-WT-001 | State/checker 必须优先消费 `process/state/STATE.current.json`，并拒绝 active change 指向 closed/cancelled/superseded 或不存在的 CR。 | P0 | Given state 指向已关闭 CR-037 When 运行 state/cr consistency check Then 检查失败并定位 `active_change` 与正式 CR 生命周期冲突；Given CR-047 active 且存在于 JSON index Then 检查通过。 | UC-WT-001 |
| REQ-WT-002 | `process/current/CURRENT.json` 必须与 State v2 的 active change、story、context、checkpoint 和 status 一致。 | P0 | Given CURRENT 任一 active ref 与 `STATE.current.json` 不同 When 运行 workspace/state check Then 退出非零并列出冲突字段；一致时冲突数为 0。 | UC-WT-001 |
| REQ-WT-003 | Canonical CR tracking 必须只使用 `CR-INDEX.json`，迁移 canonical 目录中的 legacy YAML，并把 CR-033 记录为 candidate 而非 active。 | P0 | Given `CR-INDEX.yaml` 与 JSON 并存或 CR-033 缺失 When 运行 `meta-flow check cr-tracking` Then 修复前失败；整改后退出码 0、legacy canonical 文件数为 0、CR-033 candidate 条目为 1。 | UC-WT-001 |
| REQ-WT-004 | clean clone 必须能通过一次显式 `workspace link` 建立 `process -> <artifact-root>/process/meta-flow`，且路由元数据使用锚点 + 相对路径。 | P0 | Given 两仓 clean clone When 从源码根运行 link/check Then `process_link_health=ok`，actual target 正确，metadata 不含设备绝对路径。 | UC-WT-002 |
| REQ-WT-005 | Meta Flow 内部产品/设计/质量过程文档必须以 artifact `process/meta-flow/docs` 为 canonical，根 `docs/` 只保留 tracked 公开入口。 | P0 | Given 同一产品文档被更新 When 检查路由 Then canonical 可写副本数量为 1，`process/docs/product/*` 可通过 process link 访问，根 `docs/product` 不被新建。 | UC-WT-002 |
| REQ-WT-006 | Artifact/Token Doctor 必须按 policy 预算检查活动对象，并通过 compact summary、evidence index、hash/annotation 或 archive 隔离收敛历史超限，不得截断机器结果。 | P0 | Given CR-047 产物生成前历史锚 `B0_pre.observed=21`，且 CP7 开始时采集动态 `B0_cp7`（observed/classified/unclassified/blocking_active/warning） When 整改并重跑 Then `blocking_active=0`、`unclassified=0`、每个对象均有 lifecycle/read class 与 remediation ref、从 `B0_cp7` 到终态的新增/删除/重分类 delta 全部可解释；active/default-required 超预算始终为 blocker，历史机器结果原始 hash 保持或存在 append-only correction ref。 | UC-WT-003 |
| REQ-WT-007 | Quality model 必须把 `process/state/READ-EXPANSION-LEDGER.ndjson` 列为合法 derived source，并对 6 个历史缺口使用 append-only legacy/unavailable correction。 | P0 | Given 修复前 quality/workflow doctor 报 source 缺失和 6 个 read-expansion 错误 When 整改后重跑 Then source-path error=0、未解释历史错误=0，无法恢复的授权不得标 PASS。 | UC-WT-003 |
| REQ-WT-008 | `[CP2-DQ-02 approved]` Doctor 绿色定义为 blocking error=0；warning 必须计数和披露，但空/legacy ledger 等 warning 不自动等价于 blocker。 | P0 | Given Doctor 只剩声明为 warning 的 legacy/empty-ledger 项 When 执行组合门 Then 命令退出码为 0 且报告 warning count；Given 任一 blocking item Then 退出码非零。 | UC-WT-003 |
| REQ-WT-009 | `[CP2-DQ-01 approved]` 根规则 canonical source 为 tracked `delivery/rules/AGENTS.md`；ignored/generated 根 wrapper 由安装器生成，guardrail 验证 tracked source 与 installer dry-run。 | P0 | Given `git archive HEAD` clean tree When 运行 guardrail Then 不因 ignored 根 `AGENTS.md` 缺失产生错误；tracked source 与 generated wrapper 漂移可检测。 | UC-WT-004 |
| REQ-WT-010 | Delivery guardrail 必须基于 clean/tracked 输入可独立执行，并区分 tracked cache 与 ignored 本机 cache。 | P0 | Given clean archive 与运行过 pytest 的本机 tree When 分别执行 guardrail Then clean archive 退出码 0；tracked cache 仍阻断；ignored cache 按批准策略 warning/清理而不形成永久红灯。 | UC-WT-004, UC-WT-006 |
| REQ-WT-011 | Ruff 必须达到 error=0，并成为 CI 或发布前组合质量门的一部分。 | P0 | Given 修复前 `ruff check .` 报 90 项、其中 84 项可自动修复 When 完成机械修复与人工 B/F 审查 Then `uv run --python 3.11 ruff check .` 退出码 0。 | UC-WT-005 |
| REQ-WT-012 | Ruff 修复后必须完整回归现有测试基线。 | P0 | Given lint 修复完成 When 运行 pytest with no cache provider Then 至少 377 tests 与 70 subtests 全部通过，失败数为 0。 | UC-WT-005 |
| REQ-WT-013 | README 必须为 Codex、Claude、Qoder 提供显式 `--project-dir` 的非交互 project-scope full dry-run 示例。 | P1 | Given 非 TTY 环境按 README 执行 3 条命令 When 安装器 dry-run Then 3/3 退出码 0，且错误信息不再要求补 `--project-dir`。 | UC-WT-006 |
| REQ-WT-014 | 发布 preflight 必须定义 Python cache 处理策略，避免测试后 ignored cache 持续阻断，同时保留 tracked cache 阻断。 | P1 | Given tree 含 ignored `__pycache__`/`.pyc` When 执行 preflight/guardrail Then ignored cache 被清理或报告 warning；Given cache 被 Git 跟踪或进入打包输入 Then 结论为 blocking。 | UC-WT-006 |
| REQ-WT-015 | CR-046 产品矩阵与设计状态必须收敛到 `closed / READY_WITH_RISK` 和 7/7 Story `PASS_WITH_RISK`，但不得改写 recovered/post-hoc 时序。 | P0 | Given CR-046 formal CR、7 个 CP7 result 和产品矩阵 When 运行交叉检查 Then 7/7 Story 为 implemented + PASS_WITH_RISK，产品矩阵不再写 CP2 pending/0 implemented，原始历史 hash 不被倒填。 | UC-WT-007 |
| REQ-WT-016 | CR-045/046 的 platform receipt、独立 QA、token telemetry 与真实 pilot 风险必须保持 unavailable/OPEN/follow-up，直到存在新证据和独立授权。 | P0 | Given fixture 能拒绝伪造 receipt 但平台未签发 receipt When 生成 release 结论 Then 最高为 `READY_WITH_RISK`，不得写 platform-attested 或 independent-runtime-verified。 | UC-WT-007 |
| REQ-WT-017 | Story/Run ledger 对未来执行必须强制追加真实事件；历史补录只能使用 `recovered/legacy-unverified`，不得伪造运行时间或 receipt。 | P1 | Given修复前 RUN ledger 为空且存在历史执行 When 收敛治理 Then 新验证命令均有 run event；历史事件显式 recovery status，伪造 original timestamp/receipt 的 fixture 100% 被拒绝。 | UC-WT-003, UC-WT-007 |
| REQ-GB-001 | 系统必须从 remote symbolic HEAD 识别默认主分支，并允许显式 override；无法唯一识别时必须阻断。 | P0 | Given `origin/HEAD -> origin/main` When 开启 CR Then default branch=`main`；Given remote HEAD 缺失且未提供 override Then 退出非零，不猜测 `main/master`。 | UC-GB-001 |
| REQ-GB-002 | CR branch open 前必须对 project/artifact 两仓执行 route、Git repo、clean tree、非 detached HEAD、remote、同名 ref 和 branch-name precheck。 | P0 | Given 任一仓 dirty、detached、remote 缺失、同名 branch 存在或 `check-ref-format` 失败 When 执行 open Then 两仓新 local/remote ref 数为 0，并定位失败仓和规则。 | UC-GB-001 |
| REQ-GB-003 | 两仓默认主分支只能通过 `fetch --prune` 和 `pull --ff-only` 刷新；divergence 不得通过 reset、rebase 或 force 自动消解。 | P0 | Given local default branch behind remote When open Then fast-forward 到 remote tip；Given history diverged Then open BLOCKED，原 refs 不变。 | UC-GB-001 |
| REQ-GB-004 | 系统必须从两仓刷新后的 exact remote default tip 创建同名、可预测且合法的 CR branch，并使用 `push -u` 建立 upstream。 | P0 | Given `CR-050` 与 slug When open 成功 Then 2/2 local/remote branch 名一致、base OID 等于各自 remote default tip、upstream 建立率 100%。 | UC-GB-001 |
| REQ-GB-005 | 分支发布必须只推送已经显式提交的 ref；不得隐式 stage、commit、amend 或自动选择文件。 | P0 | Given 任一仓 dirty/uncommitted When publish Then 退出非零且 remote ref 不变；Given 两仓 clean 且有新 commit Then push 后 remote CR ref 等于 local HEAD。 | UC-GB-002 |
| REQ-GB-006 | open、publish、merge、finish 必须提供 dry-run，输出逐仓、有序、可机器解析的计划且零 local/remote ref 副作用。 | P0 | Given 任一合法 fixture When dry-run Then 计划覆盖率 100%，local branch/HEAD/index/worktree 与 bare remote refs 变化数均为 0。 | UC-GB-001, UC-GB-002, UC-GB-003, UC-GB-004 |
| REQ-GB-007 | finish 必须重新 fetch 并验证 exact branch identity、tip 未漂移且 CR tip 是 remote default branch 的祖先；证明不足时不得删除。 | P0 | Given ancestry-preserving merge When finish Then ancestry check 通过；Given squash/rebase/non-ancestor 或 tip drift Then remote/local branch 保留且结果 BLOCKED。 | UC-GB-003 |
| REQ-GB-008 | finish 不得自动或隐式 merge；删除范围只能是经 CR ID/branch contract 精确识别的非 protected branch，且 remote 删除成功后才允许 `branch -d` 删除 local ref。 | P0 | Given target 是 main/master/perennial/protected 或 branch 与 CR 不匹配 When finish Then 100% 拒绝；Given合法目标 Then只删除目标 branch，不触及 tag/其他 ref。 | UC-GB-003 |
| REQ-GB-009 | 远端托管平台已自动删除 CR branch 时，finish 必须使用仍存在的 local 或已记录 exact tip 完成 ancestry 证明；缺 tip 时不得把“branch 不存在”当作已安全清理。 | P0 | Given remote branch absent、local branch tip 仍存在且已是 remote main 祖先 When finish Then可继续 local cleanup；Given local/recorded tip 均缺失 Then BLOCKED 并要求外部 receipt。 | UC-GB-003 |
| REQ-GB-010 | 双仓操作必须逐仓记录 terminal status、before/after OID、执行/跳过步骤和恢复入口；不得宣称跨仓原子事务。 | P0 | Given第二仓 push/delete 注入失败 When命令结束 Then overall=`PARTIAL/BLOCKED`，第一仓事实被保留并披露，destructive 后续步骤停止，且每仓均有恢复建议。 | UC-GB-001, UC-GB-002, UC-GB-003 |
| REQ-GB-011 | 系统必须提供独立显式 `merge` 操作；`publish` 和 `finish` 均不得隐式触发 merge。 | P0 | Given 两仓已 publish When 只执行 publish 或 finish Then default branch OID 不因隐式 merge 改变；Given 操作者显式调用 merge 且提供授权 Then 才进入 merge preflight。 | UC-GB-004 |
| REQ-GB-012 | merge 必须先对 project/artifact 两仓完成全部只读 preflight，并按 artifact→project 的确定顺序执行 default-branch update。 | P0 | Given 任一仓 remote CR tip/default tip/authorization/policy preflight 失败 When 执行 merge Then 两仓 default branch mutation 数为 0；Given 全部通过 Then artifact 先验证更新，project 后更新，顺序在 result 中 100% 可复核。 | UC-GB-004 |
| REQ-GB-013 | merge 只允许把 fresh remote default fast-forward 到 exact published CR tip；不得创建 merge commit、rebase、force 或自动解决冲突。 | P0 | Given CR tip 包含 fresh default tip When merge Then remote default 精确等于 CR tip；Given default 已推进且 CR tip 不包含它、需要 merge commit 或发生冲突 Then merge BLOCKED，禁止命令执行数为 0。 | UC-GB-004 |
| REQ-GB-014 | 双仓 merge 的部分成功必须保留两仓 CR branch、输出逐仓结果并阻断 finish；系统不得自动回滚已经成功的 default-branch fast-forward。 | P0 | Given artifact merge PASS 而 project 被并发推进、branch protection 或权限拒绝 When operation 结束 Then overall=PARTIAL、2/2 CR branch 仍存在、finish 不可执行、结果给出重新观察后的恢复入口，且已更新 default branch 不被自动 reset/force。 | UC-GB-003, UC-GB-004 |
| REQ-AW-001 | 系统必须以 project identity 为第一路由键，把目标 artifact 语义表达为 `<project_name>/docs` 与 `<project_name>/process`。 | P0 | Given project=`meta-flow` 且 layout version 为 project-first When 解析 artifact routing Then 恰好返回当前 project worktree 内 `meta-flow/docs` 与 `meta-flow/process`，不得返回 sibling namespace。 | UC-AW-001 |
| REQ-AW-002 | 系统必须为 project-first 与 legacy `docs/<project>` / `process/<project>` 提供显式版本化 dual-read；写目标不得因路径存在性静默切换。 | P0 | Given legacy/new 路径同时存在或 metadata 缺 layout version When 解析写目标 Then 结果 BLOCKED 并列出冲突；Given 显式版本唯一 When 解析 Then 重复结果一致。 | UC-AW-001 |
| REQ-AW-003 | artifact 路由元数据必须记录 existing control checkout、configurable sibling worktree parent、project worktree、project namespace、layout/sparse/owned-path policy、integration/CR/shared-main branch role，以及 integration 初始化来源与 expected OID 的锚点 + 相对路径语义。 | P0 | Given 同一仓库移到另一设备 When 重新解析 metadata Then 不依赖原设备绝对路径，project identity、目标语义、branch role与可验证 OID 语义不变。 | UC-AW-001, UC-AW-002 |
| REQ-AW-004 | 系统必须管理长期常驻的每项目 worktree；worktree 空闲时必须驻留 `projects/<project-name>/integration`，CR 活跃时使用 `projects/<project-name>/cr/<cr-id>-<slug>` 短期分支；仅当远端 integration ref 不存在时，才允许从 fresh `origin/main` exact OID create-only 初始化，已存在时禁止 recreate/reset/orphan。 | P0 | Given integration 缺失且已观察 fresh `origin/main` exact OID When 初始化 Then 恰好创建目标 integration 且基准 OID 一致；Given integration 已存在 When create/check Then保持原 ref，不 recreate/reset/orphan；Given CR finish/abort Then worktree 回到同一 integration 且不得滞留已结束 CR branch。 | UC-AW-002 |
| REQ-AW-005 | worktree create 前必须验证 common Git dir、control repo identity、目标路径、branch 占用、project name/ref 合法性和禁止嵌套边界。 | P0 | Given 目标在 control checkout 内、branch 已被其他 worktree 占用、identity 不匹配或目标非空未知目录 When create Then mutation 数为 0，结果定位失败规则和目标。 | UC-AW-002 |
| REQ-AW-006 | 系统必须提供 per-project worktree 的 create/check/list/remove plan 与结构化 health，并报告 `idle-integration` / `active-cr` 等 branch role；remove 必须满足精确身份、clean、branch/ref 和恢复证明。 | P0 | Given 两个已登记项目 When list/check Then 各自 path/layout/branch/branch-role/health 可区分；Given remove 目标 dirty、未知、仍在 active CR 或 ref 仍需恢复 Then 目录/ref 删除数为 0。 | UC-AW-002 |
| REQ-AW-007 | 每项目长期分支必须精确匹配 `projects/<project-name>/integration`，每 CR 短期分支必须匹配 `projects/<project-name>/cr/<cr-id>-<slug>`；二者均须通过 Git ref-format、project identity 与 collision 检查，`main` 不得被登记为项目 owned/idle branch。 | P0 | Given meta-flow 与 quant-lab 同时存在 `CR-051` When 生成 artifact branch Then得到不同的 `projects/meta-flow/cr/CR-051-<slug>` 与 `projects/quant-lab/cr/CR-051-<slug>`，各自可回链 integration/project/CR；非法 project/CR/slug 100%拒绝。 | UC-AW-002, UC-AW-003 |
| REQ-AW-008 | shared-artifact 模式下，一个逻辑 CR 必须使用异构 source/artifact 双 leg：source leg 从 fresh 源码默认分支创建并完成到同一默认分支；artifact leg 从 fresh 项目 integration 创建并只完成到同一 integration；不得使用 artifact control checkout 或 shared main 作为 per-CR 写入目标。 | P0 | Given route 指向 shared artifact control repo When open/publish/finish/abort preflight Then source/artifact base 与 target 分别解析为 source default 和 project integration，artifact main mutation plan 为 0，control checkout mutation 数为 0。 | UC-AW-003 |
| REQ-AW-009 | sibling project dirty、checkout branch 或 index 状态不得阻断当前项目；当前 project worktree dirty 或 identity/path mismatch 必须阻断有风险 mutation。 | P0 | Given sibling dirty/current clean When preflight Then 当前项目可继续且 sibling touched paths 为 0；Given current dirty When preflight Then后续 ref/worktree mutation 数为 0。 | UC-AW-003 |
| REQ-AW-010 | 每个 project worktree 必须声明 owned path；Git 周期的 changed/staged/committed paths 越过 ownership 时必须 fail closed。 | P0 | Given当前 branch 含 sibling project path change When publish/merge precheck Then结果 BLOCKED并列出越界路径，不推送或更新 default。 | UC-AW-003 |
| REQ-AW-011 | artifact leg 必须从已观察的 fresh project integration exact OID 建立短期 CR branch，并仅在当前 integration 仍匹配 expected OID 时完成回该 integration；shared main/integration divergence 本身不得阻断单个 CR。 | P0 | Given shared main 已推进但 project integration 未漂移 When artifact leg open/finish Then 不因 main divergence 阻断；Given current integration 不等于 expected OID When finish Then BLOCKED、integration/main mutation 数为0，并要求重新观察与安全恢复。 | UC-AW-003, UC-AW-004 |
| REQ-AW-012 | 一个逻辑 CR 的必需 legs 必须独立输出结果并由单一协调者聚合；优先级必须为 `BLOCKED > FAIL > IN_PROGRESS > PASS`，仅全部必需 leg PASS 才整体完成；`PARTIAL` 只能表达 progress/effect。 | P0 | Given 任一 leg BLOCKED/FAIL/IN_PROGRESS When 聚合 Then overall 分别按最高优先级取值且 CR 不自动关闭；Given 全部必需 leg PASS Then overall=PASS；任一失败不得自动回滚已成功 leg。 | UC-AW-004 |
| REQ-AW-013 | worktree、异构双 leg、integration bootstrap 与 aggregate gate 必须提供 dry-run，输出确定、有序、可机器解析的计划且不改变文件、link、worktree、branch、index 或 remote ref。 | P0 | Given合法与负例 fixture When dry-run Then计划字段覆盖率100%，所有本地/远端 mutation计数为0。 | UC-AW-001..005 |
| REQ-AW-014 | 系统必须提供逐项目 migration preflight/manifest，记录 legacy→project-first mapping、hash/数量、link plan、worktree/ref readiness、验证和回滚入口。 | P1 | Given未迁移项目 When生成 preflight Then manifest完整且可复跑；未授权状态下文件搬迁与link/ref变更数为0。 | UC-AW-005 |
| REQ-AW-015 | migration preflight 不得自动执行文件搬迁、软链接创建/替换、worktree/ref mutation、commit 或 remote publication。 | P0 | Given用户只调用preflight或CR-051测试 When结束 Then真实 artifact tree、软链接和Git refs快照无变化。 | UC-AW-005 |
| REQ-AW-016 | leg result 与 aggregate gate 必须共享 CR ID/attempt correlation，并逐 leg 记录 repo role、base/target、expected/current OID、terminal status、executed/skipped steps 和 resume/abort route；ledger 单写、receipt/OID 防自引用的精确 schema 留 CP3。 | P0 | Given任一步骤失败或partial When检查结果 Then source/current-artifact 两 leg 均有独立终态且 aggregate 可复算，不会把 sibling/control/shared main 冒充 artifact target。 | UC-AW-001..004 |
| REQ-AW-017 | 验证必须使用临时仓库、本地 bare remote 与至少两个项目 namespace/worktree，覆盖 integration create-only、异构双 leg、聚合优先级、expected-OID 漂移和 CR 外人工同步边界；真实 remote 只作为后续独立 pilot。 | P0 | Given自动验证完成 When检查证据 Then至少2个项目可并行走完 fixture lifecycle，聚合组合覆盖完整、跨项目/shared-main mutation=0；真实remote未验证被显式披露。 | UC-AW-002..005 |

### CR-051 CP2 R3 已解决产品决策

| 决策 ID | 用户选择 | 产品契约 | 状态 | 设计阶段仍需细化 |
|---|---|---|---|---|
| CP2-DQ-01 | 每项目长期 integration + 每 CR 短期 branch + shared main | idle=`projects/<project-name>/integration`；active=`projects/<project-name>/cr/<cr-id>-<slug>`；`main`=共享集成基线 | resolved-by-user | attach/switch/finish/abort 的命令、expected OID 与清理状态机 |
| CP2-DQ-02 | R2 曾接受显式 merge-main 推荐 | R2 历史语义不再是当前基线 | superseded-by-user / CP2-DQ-04 | 不得在 CP3 恢复为 per-CR artifact main refresh |
| CP2-DQ-03 | 接受 existing-control+sibling-worktree 推荐 | 保留现有 control checkout；worktree 位于 configurable sibling root；执行 namespace/sparse/owned-path gate | resolved-by-user | metadata/path schema、sparse 行为、owned-path gate 落点与健康修复策略 |
| CP2-DQ-04 | 一个逻辑 CR 使用异构 source/artifact legs | source 从/回源码默认分支；artifact 从/回项目 integration；artifact CR 不接触 shared main | resolved-by-user | 精确命令、expected OID、状态机与安全恢复 |
| CP2-DQ-05 | 单一 aggregate completion gate | `BLOCKED > FAIL > IN_PROGRESS > PASS`；仅全 PASS 完成；`PARTIAL` 仅为 progress/effect；不自动回滚或关闭 | resolved-by-user | leg result correlation、receipt/OID 防自引用与聚合 schema |
| CP2-DQ-06 | integration create-only 初始化与 CR 外人工同步 | integration 缺失时从 fresh `origin/main` exact OID 创建；存在时不 recreate/reset/orphan；main↔integration 同步仅在 CR 外人工执行 | resolved-by-user | 初始化 CAS、人工同步 precheck 与“无活跃 artifact CR”检查 |

> 六项 DQ 均已分类，当前 pending decision items=0；DQ-02 仅为 superseded 历史，其余为 resolved-by-user。CP2 R3 总体基线仍须由用户 `approve`，且该批准不授权任何真实 Git/worktree/link/remote mutation。

## 约束需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|---|---|---|---|---|
| REQ-PG-C001 | 不得新增第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系。 | P0 | Given 设计或实现 diff When 审查新增对象 Then 不出现与现有体系平行的新治理体系。 | 明确排除 |
| REQ-PG-C002 | 不得声明或实现跨仓原子事务。 | P1 | Given roadmap refresh 涉及过程归档库和 quant-lab 发布库 When 审查实现 Then 仅过程归档库自动写入，发布库仅生成 follow-up。 | UC-PG-006 |
| REQ-PG-C003 | 不得直接修改 `process/policies/GATE-PROFILES.json` 表达 project scale。 | P1 | Given project scale 实现完成 When 审查文件变更 Then gate profile bias 位于 `PROJECT-SCALE.yaml`。 | UC-PG-003 |
| REQ-PG-C004 | migration 不得凭空创造 capability 或 feature ID。 | P1 | Given registry 缺失目标 ID When 运行 migration Then 输出 blocked finding 或 FU-RF，不创建未经确认 ID。 | UC-PG-004 |
| REQ-PG-C005 | Ledger compaction 不得复用 `meta-flow state compact` 名称。 | P0 | Given 新增 ledger compaction 命令 When 检查 CLI Then 命令为 `ledger compact` 或 `event compact` 等独立名称。 | 实施计划 P0 后段 |
| REQ-EI-C001 | 不得伪造或推断不存在的 platform receipt、签名、token telemetry 或 checker identity。 | P0 | Given 来源字段缺失 When 生成 evidence Then 使用 unavailable/unknown 的合法表达，不生成看似 verified/measured 的值。 | UC-EI-002, UC-EI-003, UC-EI-004 |
| REQ-EI-C002 | canonical checker 必须以 canonical JSON CR index 为输入，不得被 legacy YAML 状态干扰。 | P1 | Given JSON 与 legacy YAML 并存或冲突 When 运行 canonical checker Then 只消费 canonical JSON，并对 legacy 干扰给出迁移提示。 | UC-EI-003 |
| REQ-EI-C003 | CP2 approval 不授权 credentials、runtime、production write、publish、交易、repository publication 或 quant-lab business-code changes。 | P0 | Given 用户批准 CP2 When 检查授权边界 Then 上述动作仍为未授权，必须独立获得授权。 | UC-EI-005 |
| REQ-WT-C001 | 不得读取、删除、迁移或修改 `/home/hyde/workspace/meta-flow.process-prelink-backup-20260713T100930`。 | P0 | Given CR-047 任一 Story 执行 When 检查 touched paths Then 该 backup 路径及其子路径变更数为 0。 | 用户显式排除 |
| REQ-WT-C002 | 不得通过原位改写历史 CP/result/ledger、伪造平台 receipt 或补写虚假原始运行时间来“修绿”治理检查。 | P0 | Given 历史缺口无法恢复 When 生成 correction/summary Then 使用 append-only legacy/unavailable 语义，原始 hash 保留，伪造 fixture 100% 被拒绝。 | UC-WT-003, UC-WT-007 |
| REQ-WT-C003 | CP2 approval 不授权 credentials、runtime、SaaS、production write、publish、trading、repository commit/push 或 CR-033 runtime follow-up。 | P0 | Given 用户批准 CR-047 CP2 When 审查授权范围 Then上述动作仍为 not-authorized，需独立请求。 | CR-047 不授权范围 |
| REQ-GB-C001 | 不得要求安装 `gb`、Git Town、GitPython 或 forge CLI 作为 CR branch lifecycle 的必需运行依赖。 | P0 | Given clean supported environment 只有 Git 2.43+ 与 Meta Flow When运行 branch lifecycle fixture Then无需额外 executable/package；可选 adapter 不改变默认契约。 | SGA-GB-01 |
| REQ-GB-C002 | 不得执行 force-push、force-delete、reset --hard、自动 rebase、自动冲突解决、隐式 merge、merge commit 或未经证明的 branch deletion。 | P0 | Given negative fixture 诱导任一禁止命令 When执行 plan/operation Then禁止命令执行数为 0，并输出 blocking reason。 | UC-GB-001, UC-GB-003, UC-GB-004 |
| REQ-GB-C003 | CP2 approval 只确认产品/场景/范围，不授权源码实现、commit、真实远端 branch/default-branch mutation、forge API 或凭据。 | P0 | Given用户批准 CR-050 CP2 R2 When检查授权边界 Then上述动作仍需后续门禁或独立显式授权。 | CR-050 不授权范围 |
| REQ-GB-C004 | 真实 default-branch write 必须有本次操作的独立显式授权并服从远端 branch protection；保护策略拒绝不得被绕过或降级为成功。 | P0 | Given 无授权、授权对象/OID不匹配或远端拒绝 direct update When merge Then对应仓结果为 BLOCKED/PARTIAL，default OID 不被工具以其他路径改写，错误保留远端拒绝语义。 | UC-GB-004 |
| REQ-AW-C001 | CR-051 不得搬迁真实 `meta-flow-artifacts` 文件、修改任何现有软链接，或在真实仓创建/删除 worktree、branch、commit、tag、stash 或 remote ref。 | P0 | Given CR-051 touched-path/ref audit When验证 Then上述真实对象变化数为0；仅临时fixture允许Git mutation。 | 用户明确范围 / UC-AW-005 |
| REQ-AW-C002 | 不得在 artifact control checkout 内嵌套 project worktree，也不得把未知/非空目录当作可安全覆盖目标。 | P0 | Given目标位于control worktree内部或含未知文件 When create/remove plan Then结果BLOCKED且文件删除/覆盖数为0。 | UC-AW-002 |
| REQ-AW-C003 | 不得读取、stage、commit、merge、reset、clean、remove或删除 sibling project owned path/ref/worktree。 | P0 | Given sibling含dirty/untracked/active branch When当前项目执行任何生命周期动作 Then sibling文件/ref/index变化数为0。 | UC-AW-003 |
| REQ-AW-C004 | 单个 artifact CR 不得 refresh、merge 或直接更新 shared main，也不得自动执行 main↔integration 同步、stash、rebase、force/force-with-lease、解决冲突、选择提交文件或跨 leg 回滚。 | P0 | Given负例诱导上述行为 When执行plan/operation Then禁止命令执行数为0，并返回显式 CR 外人工同步或安全恢复入口。 | UC-AW-003, UC-AW-004 |
| REQ-AW-C005 | CP2 approval 只确认产品/场景/范围，不授权HLD后续门禁跳过、源码实现、真实迁移、软链接变更、repository publication、default write、delete、credentials或runtime。 | P0 | Given用户批准CR-051 CP2 When检查授权边界 Then上述动作仍需CP3/CP5及对应操作级授权。 | CR-051不授权范围 |

## 非功能需求

| ID | 需求描述 | 优先级 | 验收条件 | 来源 |
|---|---|---|---|---|
| REQ-PG-NF001 | 所有新增机器状态对象必须 refs-only、可校验、可预算。 | P0 | Given state / project state 写入 When 运行 checker Then 长正文、历史详情和大型列表被拒绝或转移到 ledger / result / summary。 | UC-PG-001, UC-PG-002 |
| REQ-PG-NF002 | 所有检查失败必须输出可定位的字段、路径或 ID。 | P1 | Given schema、registry、surface 或 refresh 检查失败 When 查看结果 Then 能定位失败字段、输入来源和建议处理方式。 | UC-PG-004, UC-PG-005, UC-PG-006 |
| REQ-PG-NF003 | P0 enforcement 必须支持 audit -> enforce 灰度。 | P0 | Given unknown field 出现 When 处于 audit 阶段 Then WARN 不阻断；When 处于 enforce 阶段 Then ERROR 阻断。 | UC-PG-001 |
| REQ-PG-NF004 | quant-lab 迁移必须可重复验证。 | P2 | Given 迁移完成 When 重新运行 state / capability / feature / capability-claims / stale checks Then 结果与迁移报告一致。 | UC-PG-007 |
| REQ-EI-NF001 | evidence-integrity checker 失败必须包含对象 ID、失败字段、规则 ID、证据 ref 和安全路由。 | P0 | Given 任一 chronology/provenance/replay/telemetry 检查失败 When 输出 finding Then 审计者可定位原对象、原因与回退/修复入口。 | UC-EI-001..005 |
| REQ-EI-NF002 | 新 evidence contract 必须具备 versioned schema 和 legacy read-only compatibility fixture。 | P0 | Given 旧证据进入新 checker When 执行兼容检查 Then 可读对象被重放，不兼容对象显式 BLOCKED/WARN，不静默改写。 | UC-EI-003, UC-EI-005 |
| REQ-WT-NF001 | 同一 Git HEAD 与 artifact commit 在支持环境中必须产生确定的 workspace/state/checker 结果。 | P0 | Given 两台设备的源码与 artifact commit 相同 When 分别 link/check Then routing target 语义、state active refs、CR index decision 和 quality gate decision 差异为 0。 | UC-WT-001, UC-WT-002 |
| REQ-WT-NF002 | 所有失败必须输出可操作路径、字段、计数、严重度和推荐路由。 | P1 | Given cr-tracking/doctor/guardrail/ruff/install 任一失败 When 查看输出 Then 至少包含失败对象、severity、证据路径或命令以及修复/回退入口。 | UC-WT-001..006 |
| REQ-WT-NF003 | 全部 Python 检查、测试和脚本示例必须使用 `uv run`，且不得要求裸 pip 或系统 Python。 | P0 | Given README、CI 与 release preflight 被扫描 When 检查 Python 命令 Then裸 `python`/`pip install` 违规数为 0。 | UC-WT-005, UC-WT-006 |
| REQ-GB-NF001 | 对相同 repo refs/config/input，branch lifecycle plan 与 decision 必须确定；失败必须包含 repo、step、command-safe summary、OID 和恢复路由。 | P0 | Given相同 fixture 重复运行 When比较 JSON result Then计划顺序、decision、OID 和 error code 一致；失败定位字段覆盖率 100%。 | UC-GB-001..004 |
| REQ-GB-NF002 | branch、remote、CR ID 和 slug 必须作为参数列表传给 Git，不得经 shell 插值；所有 branch 名必须通过 `git check-ref-format --branch`。 | P0 | Given含空格、选项前缀、换行或 shell 元字符的输入 When运行 precheck Then 100% 拒绝且无额外命令执行。 | UC-GB-001..004 |
| REQ-GB-NF003 | merge plan/result 对相同 fresh refs、policy 与 authorization 必须确定，并完整记录两仓执行顺序、before/after OID、published tip、default tip、decision 和 resume route。 | P0 | Given 相同 bare-remote fixture 重复 dry-run或重放结果 When 对比 Then plan顺序/decision/OID字段一致率100%；缺任一仓 terminal result或恢复入口时检查失败。 | UC-GB-004 |
| REQ-AW-NF001 | 对相同metadata、repo refs、project identity、leg results与filesystem fixture，路由/worktree/heterogeneous-leg/aggregate plan和decision必须确定。 | P0 | Given同一fixture重复执行 When比较结果 Thenresolved path、步骤顺序、leg/aggregate decision、OID和error code一致率100%。 | UC-AW-001..004 |
| REQ-AW-NF002 | project-first路由元数据必须跨设备可移植，不得持久化设备相关绝对路径作为canonical contract。 | P0 | Givenworkspace整体搬迁 When重新check Then锚点+相对路径可解析，设备绝对前缀违规数为0。 | UC-AW-001, UC-AW-002 |
| REQ-AW-NF003 | 多项目并发隔离必须避免共享checkout、index与项目branch；并行fixture中的每个 worktree 必须只在自身 integration/CR branch 间切换，共享 `main` 不得成为项目 working branch。 | P0 | Given两个项目并行执行本地fixture lifecycle When完成 Thenindex.lock争用数、wrong-branch写入数、shared-main checkout占用数和cross-project touched paths均为0。 | UC-AW-002, UC-AW-003 |
| REQ-AW-NF004 | Git参数必须以argv列表传递并执行project/path/ref输入校验；错误必须包含project、repo role、step、safe command summary、OID和恢复路由。 | P0 | Given含路径穿越、选项前缀、换行或shell元字符的输入 Whenprecheck Then100%拒绝且无额外命令执行。 | UC-AW-001..004 |
| REQ-AW-NF005 | create/check/list/remove/bootstrap/leg-preflight/aggregate必须可重入；重复执行不得静默改变已确认路由、重建既有 integration 或删除恢复证据。 | P1 | Given相同成功/失败操作重复执行 When比较 Then返回PASS/NO_CHANGE/BLOCKED等稳定状态，非预期mutation为0。 | UC-AW-002, UC-AW-004, UC-AW-005 |

## 风险与假设

| ID | 类型 | 内容 | 关联需求 | 缓解措施 |
|---|---|---|---|---|
| RA-PG-001 | RISK | allowlist 可能误伤存量但仍合理的 current state 字段。 | REQ-PG-001 | 使用 audit -> enforce 灰度，先 WARN 收集真实字段。 |
| RA-PG-002 | RISK | 历史 impact surface 漂移数量大，直接阻断会干扰当前开发。 | REQ-PG-013 | 历史静默，提供独立 migration report。 |
| RA-PG-003 | RISK | capability ID 归一时真实来源分散，容易误建自由字符串。 | REQ-PG-011, REQ-PG-021 | 只允许 registry 引用；缺失项输出 blocked finding / FU-RF。 |
| RA-PG-004 | RISK | Roadmap refresh 跨仓边界不清会误改发布库。 | REQ-PG-016, REQ-PG-C002 | 明确自动写入只限过程归档库。 |
| RA-PG-005 | ASSUMPTION | 已批准实施计划是本轮产品基线事实来源。 | 全部 | 本文档引用 source_plan；若计划变更，必须通过后续 CR 增量更新。 |
| RA-WT-001 | RISK | 历史 artifact 超预算对象若被原位压缩会破坏证据 hash 与审计时序。 | REQ-WT-006, REQ-WT-C002 | 优先 compact summary + evidence index + archive/annotation；所有修正 append-only。 |
| RA-WT-002 | RISK | 根规则 source 选择错误会让 clean clone 或本机 wrapper 二选一失败。 | REQ-WT-009, REQ-WT-010 | CP2 显式选择 canonical source，clean archive 与 installed workspace 双路径验证。 |
| RA-WT-003 | RISK | 把所有 Doctor warning 当 blocker 会导致 legacy/empty ledger 永久阻断；反之忽略 error 会虚假全绿。 | REQ-WT-008 | severity 枚举、exit-code contract 与 warning count 分离验证。 |
| RA-WT-004 | ASSUMPTION | 修复前实测快照包括 cr-tracking FAIL、Doctor `B0_pre.observed=21`、quality/read-expansion/run-ledger findings、Ruff 90、guardrail cache 阻断；2026-07-14 CP3 评审已观测 Doctor=22，`+1` 为 `CR-047.summary.json`，证明 observed count 会随合规过程产物变化。 | REQ-WT-003, REQ-WT-006..012, REQ-WT-014 | 保留 `B0_pre` 作为历史回归锚；CP7 开始采集 `B0_cp7` 作为分类锚并解释全部 delta。新增 active/default-required 超预算不得因计数漂移而降级为 warning。 |
| RA-GB-001 | RISK | project/artifact 两仓没有跨仓原子 ref transaction，第一仓成功、第二仓失败可能产生 partial state。 | REQ-GB-010 | 预检全部仓后再执行；逐仓 terminal result；失败后停止 destructive step，并提供幂等 resume/cleanup 路由。 |
| RA-GB-002 | RISK | squash/rebase merge 不保留 branch tip ancestry，仅靠 Git DAG 无法证明代码已进入 main。 | REQ-GB-007, REQ-GB-009 | MVP fail closed；未来 forge receipt adapter 需独立 CR 和最小权限授权。 |
| RA-GB-003 | RISK | 隐式 stage/commit 可能把无关文件、个人配置或 secret 纳入远端分支。 | REQ-GB-005 | MVP 仅推送已提交 refs；提交文件选择保持显式并继续使用现有 secret/guardrail 检查。 |
| RA-GB-004 | ASSUMPTION | 受支持仓库提供 Git 2.43+、命名 remote 和可解析 symbolic default branch，或操作者显式提供 override。 | REQ-GB-001, REQ-GB-C001 | `git --version` 与 remote HEAD precheck；不满足时 BLOCKED，不安装同名 `gb` 替代。 |
| RA-GB-005 | RISK | 直接 fast-forward 远端默认分支可能被 branch protection、review requirement 或 merge queue 拒绝。 | REQ-GB-011..013, REQ-GB-C004 | 将远端拒绝视为合法 BLOCKED/PARTIAL；不绕过策略，不自动切 forge API；需要受保护仓 merge 时转未来 forge adapter。 |
| RA-GB-006 | RISK | artifact default 已更新但 project default 更新失败会造成跨仓 main 短暂不一致。 | REQ-GB-012, REQ-GB-014, REQ-GB-NF003 | preflight-all、artifact→project 确定顺序、逐仓 post-check、保留两仓 CR branch、阻断 finish；恢复前重新观察 refs，不自动回滚已成功事实。 |
| RA-AW-001 | RISK | control repo、project worktree与namespace identity若解析错误，会让当前项目命令修改其他项目。 | REQ-AW-001..010, REQ-AW-C003 | project identity、common Git dir、worktree path、owned path四重校验；歧义fail closed。 |
| RA-AW-002 | RISK | 长期常驻worktree可能残留stale integration/CR branch、prunable metadata、已结束CR占用或断链。 | REQ-AW-004..006, REQ-AW-NF005 | check/list校验branch role；repair只输出显式建议；finish/abort后的integration回归、remove与CR branch清理均要求clean/ref/恢复证明。 |
| RA-AW-003 | RISK | shared main 与 project integration 可能长期分歧；若把该分歧误作单 CR blocker会重新引入跨项目耦合，若忽略 integration expected-OID 漂移又会覆盖并发结果。 | REQ-AW-011..012 | main/integration divergence 仅由 CR 外人工维护；单 CR 只以 integration expected OID 判定 artifact finish；漂移 fail closed，不自动 merge/rebase/force/回滚。 |
| RA-AW-004 | RISK | legacy与project-first布局共存可能形成两个可写真相源。 | REQ-AW-002..003, REQ-AW-NF001..002 | layout version和write target显式化；多解BLOCKED；迁移manifest记录切换点与回滚。 |
| RA-AW-005 | RISK | 能力实现误调用真实artifact仓会提前触发用户计划外迁移或ref变更。 | REQ-AW-013..015, REQ-AW-C001 | 默认dry-run/fixture；真实路径denylist与touched-ref审计；任何真实mutation需独立授权。 |
| RA-AW-006 | ASSUMPTION | 用户将逐项目执行真实artifact迁移和软链接挂接，本CR只交付能力与交接包。 | REQ-AW-014..015 | 在MVP、Backlog、迁移手册和CP8不授权项中持续保留边界。 |

## 里程碑建议

| 里程碑 | 包含需求 | 交付物 | 前置里程碑 |
|---|---|---|---|
| M0：Second-system Guardrail | REQ-PG-C001 | 设计 / 实现审查阻断第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系 | 无 |
| M1：Current State Enforcement | REQ-PG-001..006, REQ-PG-NF001, REQ-PG-NF003 | allowlist、field budgets、update API、写契约、audit/enforce gate | 无 |
| M1.5：Ledger Hygiene Boundary | REQ-PG-C005 | ledger compact / event compact 与 state compact 的命令边界和帮助文档职责区分 | M1 |
| M2：Project Governance Objects | REQ-PG-007..009, REQ-PG-NF001 | `process/project/`、`PROJECT.current.json`、`PROJECT-SCALE.yaml`、ROADMAP / MILESTONES 基线 | M1 |
| M3：Reference and Impact Normalization | REQ-PG-010..013, REQ-PG-C004, REQ-PG-NF002 | capability / feature registry 校验、impact surface migration report | M2 |
| M4：Roadmap Refresh and Follow-up | REQ-PG-014..019, REQ-PG-C002 | ROADMAP-REFRESH result/checker、GATE-LEDGER event、FU-RF 支持、stale check | M3 |
| M5：quant-lab Migration | REQ-PG-020..022, REQ-PG-NF004 | quant-lab state 清理、capability 归一、impact surface 迁移、stale report | M4 |
| M-WT1：Canonical Truth and Routing | REQ-WT-001..005, REQ-WT-015, REQ-WT-NF001 | State/CURRENT/CR index 一致性、JSON-only tracking、portable process/docs route、CR-046 status convergence | CR-047 CP2/CP3 |
| M-WT2：Deterministic Quality Gates | REQ-WT-006..012, REQ-WT-017, REQ-WT-NF002..003 | Doctor、history correction、clean-clone guardrail、Ruff、pytest、Run ledger contract | M-WT1 |
| M-WT3：Operator Entry and Release Risk | REQ-WT-013..016, REQ-WT-C001..003 | 非交互安装、cache preflight、不授权边界、READY_WITH_RISK 收敛 | M-WT2 |
| M-GB1：Safe CR Branch Open | REQ-GB-001..004, REQ-GB-006, REQ-GB-C001..002, REQ-GB-NF001..002 | 双仓 precheck、remote refresh、同名 branch、upstream 与 dry-run | CR-050 CP2/CP3/CP5 |
| M-GB2：Committed Ref Publication | REQ-GB-005, REQ-GB-006, REQ-GB-010 | 只推送已提交 refs、逐仓核验与 partial result | M-GB1 |
| M-GB3：Explicit Paired Fast-forward Merge | REQ-GB-011..014, REQ-GB-C002..004, REQ-GB-NF003 | 两仓全量预检、artifact→project fast-forward-only default update、逐仓结果与 partial/resume | M-GB2；default-branch write 独立授权 |
| M-GB4：Proof-gated Cleanup | REQ-GB-007..010, REQ-GB-014, REQ-GB-C002 | merge 后重新证明 ancestry/tip/protected-ref，并幂等清理 local/remote CR branch | M-GB3；2/2 merge terminal PASS |
| M-AW1：Project-first Routing Compatibility | REQ-AW-001..003, REQ-AW-013, REQ-AW-NF001..002 | layout schema、project identity、legacy dual-read、portable metadata与dry-run解析 | CR-051 CP2/CP3/CP5 |
| M-AW2：Per-project Worktree Management | REQ-AW-004..007, REQ-AW-013, REQ-AW-C002, REQ-AW-NF003..005 | create/check/list/remove计划、registry/health、长期integration与短期CR分支命名、existing-control+sibling-root拓扑及多项目隔离fixture | M-AW1；CP2-R2-DQ-01/03 resolved；待CP3/CP5 |
| M-AW3：Heterogeneous Legs and Aggregate Gate | REQ-AW-008..013, REQ-AW-016..017, REQ-AW-C003..005 | source-default leg、artifact-integration leg、expected-OID门、单一聚合、结构化结果和并发fixture | M-AW2；CP2-DQ-04..06 resolved；待CP3/CP5 |
| M-AW4：Migration Handoff without Mutation | REQ-AW-014..015, REQ-AW-C001, REQ-AW-NF005 | per-project migration preflight/manifest、验证/回滚checklist与用户手册；真实迁移0 | M-AW3 |

## 明确排除项

- 不做代码实现细节、模块拆分、CLI 参数设计或数据模型字段类型细化；这些属于设计 / 实现阶段。
- 不在本轮产品文档中修改已批准设计文档。
- 不把 quant-lab 迁移和 capability 新能力实现混为同一个交付范围。
- 不把发布库 stale finding 自动升级为发布库写入授权。
- 不把 CR-051 能力开发扩成真实 `meta-flow-artifacts` 目录迁移、软链接挂接、control repo 转 bare、真实远端试运行或批量项目切换。
- 不在产品阶段冻结具体Python模块、CLI参数名、metadata字段类型或Git命令序列；这些由CP3/CP5在上述安全契约内设计。
