---
status: baseline
version: "3.0"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
active_change_ref: "CR-052"
current_baseline: "CR-052-VNEXT-R2-candidate"
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
  - UC-MR-001
  - UC-MR-002
  - UC-MR-003
  - UC-MR-004
  - UC-MR-005
  - UC-MR-006
  - UC-MR-007
  - UC-VNEXT-001
  - UC-VNEXT-002
  - UC-VNEXT-003
  - UC-VNEXT-004
  - UC-VNEXT-005
  - UC-VNEXT-006
source_requirements: "process/docs/product/REQUIREMENTS.md"
---

# Meta Flow 项目治理与状态强制 Story Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 3.0 | 2026-07-19 | meta-pm | CR-052 vNext R2：新增 ACT-VNEXT-01..05、ST-VNEXT-001..006 与 SL-VNEXT-A..D；保留全部历史 Activity/Story/Slice ID，将 PG/EI/WT 标记 reframed、GB/AW/MR 标记 superseded，并明确 ST-VNEXT 只是 CP2 产品候选。 | 原文档增量 reframe；CP2 人工门待批准 |
| 1.4 | 2026-07-13 | host-orchestrator-inline-fallback | 增量追加 CR-047 ACT-WT-01..05、ST-WT-001..007 与 SL-WT-01..03；保留既有 Story/Slice ID。 | 原文档增量更新 |
| 1.5 | 2026-07-15 | host-orchestrator inline fallback | 增量追加 CR-050 ACT-GB-01..03、ST-GB-001..003 与 SL-GB-01..03；保留全部既有 Activity/Story/Slice ID。 | 原文档增量更新 |
| 1.6 | 2026-07-16 | host-orchestrator inline fallback | 用户批准将 publish 后的显式 fast-forward-only merge 纳入 CR-050；新增 ACT-GB-04、ST-GB-004 与 SL-GB-04，并将执行顺序明确为 ST-GB-001→002→004→003；不重编号既有 ID，等待 CP2 R2。 | 原文档增量更新 |
| 1.7 | 2026-07-17 | meta-pm | 增量追加 CR-051 ACT-AW-01..04、ST-AW-001..005 与 SL-AW-01..04；保留全部既有 Activity/Story/Slice ID，并明确真实迁移是后续项目级工作。 | 原文档增量更新 |
| 1.8 | 2026-07-17 | meta-pm | CR-051 CP2 R2：不新增或重编号 Activity/Story/Slice，修订 ST-AW-002..004 与 SL-AW-02..03 的 branch lifecycle，明确项目长期 integration、每 CR 短期 branch、shared main、显式 merge-main 和 finish/abort 回归语义。 | 原文档增量更新；R2总体门仍待approve |
| 1.9 | 2026-07-18 | meta-pm | CR-051 CP2 R3：保持 ACT-AW/ST-AW/SL-AW ID 与数量不变，将当前旅程修订为integration create-only bootstrap、异构source/artifact双leg、单一聚合门和CR外人工main/integration同步。 | 原文档增量更新；R2历史保留，R3总体门仍待approve |
| 2.0 | 2026-07-19 | meta-pm | 为 CR-052 增量新增 ACT-MR-01..06、ST-MR-001..007 与 SL-MR-01..06；保留全部 CR-051 Activity/Story/Slice ID，并明确 ST-MR 只是 CP2 产品候选，不是 CP4 正式 Story。 | 原文档增量更新；CR-052 CP2 人工门待批准 |
| 1.0 | 2026-07-02 | meta-pm | 基于产品场景和需求建立用户故事地图 | 初始化长期可追踪产品规划基线 |
| 1.2 | 2026-07-11 | meta-pm | 增量追加 CR-046 的 5 个活动、7 个 outcome Story 与 release slice；保留 ST-PG-001..013 | 原文档增量更新 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：不新增或重编号 Story，扩展 ST-EI-002/004/006/007 的 requirement refs 与验收语义 | 原文档增量更新 |

## 用户活动与任务

### vNext 历史规划状态

| 历史范围 | vNext 状态 | 规划处理 |
|---|---|---|
| ACT/ST/SL-PG、EI、WT | reframed | 保留历史 outcome 与验证事实；后续能力只作为 vNext Work scoped 依赖，不按原大链路整体重做 |
| ACT/ST/SL-GB、AW、MR | superseded | 保留 ID 与审计；共享 artifact worktree / migration-readiness 不再进入 CR-052 当前 release slices |
| ACT/ST/SL-VNEXT | current-candidate | 当前 CP2 产品规划输入；CP2 approved 前不得转为正式 Story 卡片或 LLD |

| Activity ID | 用户活动 | 目标用户 | 用户任务 | 来源场景 |
|---|---|---|---|---|
| ACT-PG-01 | 保护轻量运行态入口 | P-01, P-02 | 在写入 current state 前识别非法字段、超预算字段和绕过 writer 的路径 | UC-PG-001 |
| ACT-PG-02 | 建立项目级治理状态 | P-01, P-04 | 用 refs-only 项目状态表达项目定位、规模、milestone 和 roadmap | UC-PG-002, UC-PG-003 |
| ACT-PG-03 | 归一治理引用语义 | P-01, P-03 | 将 capability / feature / impact surface 从自由文本归一到可检查对象 | UC-PG-004, UC-PG-005 |
| ACT-PG-04 | 刷新 roadmap 且不越权写发布库 | P-01, P-04 | 生成 roadmap refresh result、ledger event、stale finding 和 follow-up 候选 | UC-PG-006 |
| ACT-PG-05 | 迁移真实长期项目 | P-03, P-04 | 使用新机制迁移 quant-lab 并输出可复现验证证据 | UC-PG-007 |
| ACT-EI-01 | 证明门禁时序 | P-05, P-01 | 重建 checkpoint/gate/result/state/ledger 的有效因果链并拒绝非法顺序 | UC-EI-001 |
| ACT-EI-02 | 证明真实平台调度 | P-05, P-06 | 追踪 attempt、receipt、retry、supersession 与 terminal closure | UC-EI-002 |
| ACT-EI-03 | 重放历史证据 | P-05, P-03 | 用已识别 checker 输出 as-executed/current-replay 双口径 | UC-EI-003 |
| ACT-EI-04 | 度量 workflow 成本 | P-06, P-04 | 区分 measured、estimated 和 unavailable token usage | UC-EI-004 |
| ACT-EI-05 | 验收 append-only pilot | P-03, P-05 | 在不改业务实现的前提下迁移并重放 CR-163 证据 | UC-EI-005 |
| ACT-WT-01 | 统一 workflow truth 与路由 | P-01, P-05 | 让不同设备从同一 State/CR/artifact 真相恢复 | UC-WT-001, UC-WT-002 |
| ACT-WT-02 | 收敛 Doctor 与历史治理 | P-01, P-04 | 消除阻断错误并保留可审计 warning/历史语义 | UC-WT-003 |
| ACT-WT-03 | 建立 clean-clone 质量门 | P-02, P-04 | 让规则、guardrail、Ruff 与 cache 策略确定执行 | UC-WT-004, UC-WT-005 |
| ACT-WT-04 | 提供非交互安装入口 | P-03 | 在 CI/Agent 中直接完成三平台 dry-run | UC-WT-006 |
| ACT-WT-05 | 收敛 CR-046 当前事实 | P-05, P-04 | 让产品矩阵与正式证据一致并保留风险 | UC-WT-007 |
| ACT-GB-01 | 开启可恢复的 CR 分支 | P-01, P-03 | 从刷新后的远端默认分支为 project/artifact 建立同名 CR branch 与 upstream | UC-GB-001 |
| ACT-GB-02 | 发布已提交的 CR 变更 | P-01, P-02 | 只把显式提交的两仓 refs 推送到匹配远端 branch，并核验 OID | UC-GB-002 |
| ACT-GB-03 | 在合并证明后清理分支 | P-01, P-04 | 验证 exact tip/ancestry/protected-ref 后删除目标 remote/local branch | UC-GB-003 |
| ACT-GB-04 | 显式快进两仓默认分支 | P-01, P-04 | 在独立授权下先预检两仓，再按 artifact→project 把 exact published tip fast-forward 到 remote default | UC-GB-004 |
| ACT-AW-01 | 定位当前项目 artifact | P-01, P-03 | 按 project identity 和 layout version 唯一解析当前项目 docs/process，同时兼容未迁移布局 | UC-AW-001 |
| ACT-AW-02 | 管理项目独立 worktree | P-01, P-03 | 创建、检查、列举和安全移除长期项目 worktree；空闲驻留项目 integration，CR 期间切换到项目命名的短期 branch | UC-AW-002 |
| ACT-AW-03 | 执行并聚合异构 Git legs | P-01, P-02, P-04 | source leg 从/回源码默认分支，artifact leg 从/回项目integration；逐leg产出结果并由单一聚合门判定逻辑CR完成，shared main同步留在CR外 | UC-AW-003, UC-AW-004 |
| ACT-AW-04 | 准备逐项目迁移交接 | P-03, P-04 | 生成只读 migration manifest、验证和回滚清单，由用户后续逐项目执行 | UC-AW-005 |
| ACT-MR-01 | 建立可移植 route truth | P-01, P-03, P-07 | 用 schema v2 external anchor 和 health mode table 区分 legacy、migrated 与 conflict | UC-MR-001 |
| ACT-MR-02 | 规范 seed 与 ownership | P-03, P-04, P-07 | 冻结 manifest，只 prune inherited seed，并在 receipt 后激活 steady ownership | UC-MR-004 |
| ACT-MR-03 | 持久化执行证据 | P-05, P-06, P-07 | 跨进程写读 leg/aggregate evidence，并把 audit tail 与执行完成事实分离 | UC-MR-002 |
| ACT-MR-04 | 协调 transitional bootstrap | P-01, P-04, P-07 | 以 intent/receipt/third-state/resume 完成首次迁移前置，并由 CP0 显式 import | UC-MR-003 |
| ACT-MR-05 | 受权执行 migration plan | P-04, P-07 | 用通用 engine 和逐动作 typed authorization 计划/执行/refuse 高风险 mutation | UC-MR-005 |
| ACT-MR-06 | 证明 readiness 并交接真实迁移 | P-04, P-05, P-07 | 在临时三并列拓扑真实演练、故障恢复、scoped check，并生成 CR-053 handoff | UC-MR-006, UC-MR-007 |
| ACT-VNEXT-01 | 隔离每个项目的发布与过程真相 | P-08, P-01 | 复用现有发布仓，并为每项目绑定唯一独立过程仓，证明跨项目变化为0 | UC-VNEXT-001 |
| ACT-VNEXT-02 | 维护长期治理骨架 | P-08, P-04 | 用 Project/Roadmap/Phase 表达长期方向，用 Work 承载一次执行并克制回写投影 | UC-VNEXT-002, UC-VNEXT-003 |
| ACT-VNEXT-03 | 为 Work 选择风险等级 | P-08, P-04 | 按范围和风险把 Work 唯一路由到 G0/G1/G2，并在风险/预算扩大时升级 | UC-VNEXT-004 |
| ACT-VNEXT-04 | 约束 Work 的读取、写入、检查和 token | P-08, P-02, P-05 | 只执行声明 scope，达到上限前停止并记录扩展或升级 | UC-VNEXT-005 |
| ACT-VNEXT-05 | 快照迁移并完成隔离试点 | P-03, P-04, P-08 | 只迁当前态、旧仓只读，以2项目×2周期证明隔离、预算和回滚 | UC-VNEXT-006 |

## Story 列表

| Story ID | 用户故事 | 优先级 | 来源需求 | 关键验收 |
|---|---|---|---|---|
| ST-PG-001 | 作为维护者，我要用 allowlist 和字段预算约束 `STATE.current.json`，以防止轻量入口继续膨胀。 | P0 | REQ-PG-001, REQ-PG-002, REQ-PG-NF003 | unknown field 在 audit WARN、enforce ERROR；超预算字段被定位并拒绝。 |
| ST-PG-002 | 作为维护者，我要所有 current state 写入走受控更新入口，以便写入前后都可校验。 | P0 | REQ-PG-003, REQ-PG-004 | 内部绕过 writer 的路径改为受控 API；直接写入路径被测试覆盖。 |
| ST-PG-003 | 作为 Agent / Skill 作者，我要清楚知道 current state 写契约，以避免把重型状态写入轻量状态。 | P0 | REQ-PG-005, REQ-PG-NF001 | 规则、state-router、README 同步写明禁止直编和替代落盘位置。 |
| ST-PG-004 | 作为维护者，我要用 `PROJECT.current.json` 表达 refs-only 项目状态，以承载长期项目治理。 | P1 | REQ-PG-006, REQ-PG-007, REQ-PG-008 | bootstrap 创建 `process/project/`；project state ref 可从 current state 访问；大小预算通过。 |
| ST-PG-005 | 作为审批者，我要看到项目规模和 gate profile bias 的原因，以便理解后续门禁默认倾向。 | P1 | REQ-PG-009, REQ-PG-C003 | `PROJECT-SCALE.yaml` 记录三档规模、bias 和 reason；不修改 gate profile 源文件。 |
| ST-PG-006 | 作为迁移执行者，我要 capability / feature refs 只引用标准 registry，以避免能力名称漂移。 | P1 | REQ-PG-010, REQ-PG-011, REQ-PG-C004 | 未注册 ID 进入 blocked finding 或 FU-RF；不自动创造 ID。 |
| ST-PG-007 | 作为维护者，我要把 impact surface、路径、feature、capability 分开，以恢复冲突检测语义。 | P1 | REQ-PG-012, REQ-PG-013 | 新 CR 使用治理面枚举；历史漂移有 migration report；普通 cr check 不刷屏。 |
| ST-PG-008 | 作为维护者，我要 roadmap refresh 生成独立 result 和 checker，以区别于 CP gate。 | P1 | REQ-PG-014, REQ-PG-015 | result decision 使用固定枚举；checker 校验 refresh 专属 schema。 |
| ST-PG-009 | 作为审批者，我要 refresh cascade 只自动更新过程归档库，以避免误改发布库。 | P1 | REQ-PG-016, REQ-PG-017, REQ-PG-C002 | 发布库陈旧项只进入 must_check / stale_items / follow_up_candidates；ledger 记录 refresh event。 |
| ST-PG-010 | 作为维护者，我要 RF 前缀 follow-up 和 stale check，以持续追踪 roadmap 影响。 | P1 | REQ-PG-018, REQ-PG-019 | FU-RF / SP-RF / RA-RF 被 tracking 接受；stale check 输出跨对象陈旧 finding。 |
| ST-PG-011 | 作为迁移执行者，我要用新治理机制迁移 quant-lab，以验证机制能服务真实长期项目。 | P2 | REQ-PG-020, REQ-PG-021, REQ-PG-022, REQ-PG-NF004 | quant-lab state / capability / feature / stale 检查可复跑；发布库未被自动改写。 |
| ST-PG-012 | 作为维护者，我要 ledger compaction 与 state compact 在命令和职责上明确分离，以避免长期审计数据治理和 current state 渲染语义混淆。 | P0.5 | REQ-PG-C005 | ledger compaction 使用 `ledger compact` 或 `event compact`；不复用 `state compact`；帮助文档说明二者职责差异。 |
| ST-PG-013 | 作为维护者，我要所有 CR-037 子能力复用现有 state、context、result、ledger 和 registry 体系，以避免为整改再造第二套治理机制。 | P0 | REQ-PG-C001 | 设计和实现审查能识别 hot/warm/cold、roadmap_impact、PROJECT-LEDGER、自由 capability 命名空间等平行机制，并阻断。 |
| ST-EI-001 | 作为工作流审计者，我要机器校验 gate 时序和条件式批准，以便拒绝提前推进或回填式历史证明。 | P0 | REQ-EI-001, REQ-EI-002, REQ-EI-NF001 | chronology negative fixtures 100% 被拒绝；合法条件式批准可重放。 |
| ST-EI-002 | 作为证据生产者，我要每个 dispatch attempt 都有可验证 provenance、retry/supersession 和 terminal closure，以便证明平台调用真实发生。 | P0 | REQ-EI-003..005, REQ-EI-019, REQ-EI-023, REQ-EI-C001 | receipt 可用时验证；仅 session 可观察而仓库无 receipt 时披露 `session-observed/repository-unverifiable`；所有 attempts 有终态；compaction/restore 不混淆 event/dispatch/attempt/run 或改变 terminal selection。 |
| ST-EI-003 | 作为工作流审计者，我要 CP result 与最终 attempt、输入 hash 和其他真相源一致，以便不再只依赖 schema PASS。 | P0 | REQ-EI-006..008 | input/check attempt 覆盖 100%；final correlation 合法；跨真相源冲突被阻断。 |
| ST-EI-004 | 作为维护者，我要 delivered finalization、workflow health 与 read expansion 授权可检查，以便关闭残留 active refs 和隐式全文读取。 | P0 | REQ-EI-009, REQ-EI-010, REQ-EI-019, REQ-EI-021, REQ-EI-C003 | delivered active refs 为 0；health 目标存在且归属当前 CR；compaction 保留 health refs；未授权 expansion 为 0；机器 audit report 分离 event/attempt/thread/outcome/token 指标并携带 provenance/input hashes。 |
| ST-EI-005 | 作为成本分析者，我要诚实区分 measured、estimated 和 unavailable token usage，以便用可信数据优化上下文和调度。 | P0 | REQ-EI-011..013, REQ-EI-C001 | measurement status 100% 覆盖；估算不冒充实测；成本可归属聚合。 |
| ST-EI-006 | 作为审计者，我要记录 checker identity 并输出双口径 replay，以便解释 checker 演进而不改写历史。 | P0 | REQ-EI-014, REQ-EI-015, REQ-EI-021, REQ-EI-022, REQ-EI-C002, REQ-EI-NF002 | checker/version/hash 可追踪；as-executed/current-replay 均保留；机器 audit report 的计数维度与 provenance 可复核；CR-046 R1 null-provenance results 保留且 strict profile 不得报 fully replayable。 |
| ST-EI-007 | 作为迁移执行者，我要以 append-only 方式迁移 quant-lab CR-163 证据，以便用 23/23 replay 验收机制且业务源码不变。 | P1 | REQ-EI-016..020, REQ-EI-NF002 | pilot 消费通用 versioned post-close correction lifecycle；correction 限定允许字段/范围并带 author/reason/evidence/supersedes/audit trail；原历史不变；23/23 PASS；lineage 业务源码 diff 为 0。 |
| ST-WT-001 | 作为维护者，我要 State、CURRENT 与 JSON CR index 共享一个可校验真相，以免 closed CR 或 legacy index 继续驱动流程。 | P0 | REQ-WT-001..003 | stale/closed/missing active refs 被拒绝；CR-033 为 candidate；cr-tracking 退出 0。 |
| ST-WT-002 | 作为迁移执行者，我要 clean clone 通过一次 portable link 得到唯一过程/文档真相源。 | P0 | REQ-WT-004..005 | symlink health=ok；metadata 无设备绝对路径；canonical writable docs 副本为 1。 |
| ST-WT-003 | 作为审批者，我要 Doctor 区分 blocker、warning 与历史 unavailable，以便在不改写历史的前提下判断发布。 | P0 | REQ-WT-006..008, REQ-WT-017 | blocking errors=0；warning 有计数/理由；历史修正 append-only。 |
| ST-WT-004 | 作为维护者，我要 clean-clone guardrail 使用 tracked canonical rules，并合理处理本机 cache。 | P0 | REQ-WT-009..010 | archive tree pass；tracked cache block；ignored cache 按批准策略 warning/preflight。 |
| ST-WT-005 | 作为开发者，我要 Ruff 与完整 pytest 成为同一发布质量门。 | P0 | REQ-WT-011..012 | Ruff 0；至少 377 tests + 70 subtests pass。 |
| ST-WT-006 | 作为自动化调用方，我要 README 提供三平台非交互 dry-run 和 cache preflight。 | P1 | REQ-WT-013..014 | Codex/Claude/Qoder 3/3 dry-run pass。 |
| ST-WT-007 | 作为审计者，我要 CR-046 产品状态与 7/7 Story evidence 收敛，同时保留 READY_WITH_RISK。 | P0 | REQ-WT-015..016 | 无 CP2-pending/0-implemented 陈旧声明；风险不被虚假关闭。 |
| ST-GB-001 | 作为 Host Orchestrator，我要从两仓刷新后的远端默认分支开启同名 CR branch，以便源码和过程证据在另一设备可一起恢复。 | P0 | REQ-GB-001..004, REQ-GB-006, REQ-GB-C001..002, REQ-GB-NF001..002 | 2/2 branch base/upstream 正确；dirty/detached/divergence/collision 100% 阻断；dry-run 零 ref 变更。 |
| ST-GB-002 | 作为实现维护者，我要只发布已经显式提交的 CR refs，以便避免隐式 stage/commit 夹带无关文件。 | P0 | REQ-GB-005..006, REQ-GB-010, REQ-GB-C002, REQ-GB-NF001 | clean committed refs 推送后 2/2 remote=local HEAD；dirty 时远端变化为 0；partial 有逐仓恢复入口。 |
| ST-GB-003 | 作为审批者，我要只在 exact tip 已被远端主分支包含时清理 CR branch，以便不误删未合并、漂移或受保护 refs。 | P0 | REQ-GB-007..010, REQ-GB-C002, REQ-GB-NF001..002 | non-ancestor/squash/ref-drift/protected 100% 阻断；合法目标精确删除；remote 已自动删时仍要求 known tip。 |
| ST-GB-004 | 作为审批者，我要在 publish 后显式将两仓 CR tip 以 fast-forward-only 方式合入各自远端默认分支，以完成受治理旅程而不绕过远端策略。 | P0 | REQ-GB-006, REQ-GB-011..014, REQ-GB-C002..004, REQ-GB-NF001..003 | preflight-all 后按 artifact→project 执行；2/2 default=CR tip 才 PASS；merge commit/rebase/force/conflict-resolution 为 0；partial 保留两仓 branch 并阻断 finish。 |
| ST-AW-001 | 作为维护者，我要按 project identity 和显式 layout version 唯一解析 artifact docs/process，以便新旧布局共存时不会写错项目。 | P0 | REQ-AW-001..003, REQ-AW-013, REQ-AW-NF001..002 | project-first解析唯一；legacy dual-read可控；歧义写目标100%阻断；metadata跨设备可移植。 |
| ST-AW-002 | 作为迁移执行者，我要管理长期、独立、项目命名的 artifact worktree，并安全初始化缺失的integration，以便后续软链接稳定且不同项目不争用checkout/index/branch。 | P0 | REQ-AW-004..007, REQ-AW-013, REQ-AW-C002..003, REQ-AW-NF003..005 | integration缺失时从fresh `origin/main` exact OID create-only；存在时不recreate/reset/orphan；idle/active分支角色精确；危险remove为0。 |
| ST-AW-003 | 作为Host Orchestrator，我要让一个逻辑CR使用异构source/artifact legs，以便source回源码default、artifact只回项目integration且不把跨项目main同步塞进单CR。 | P0 | REQ-AW-008..011, REQ-AW-013, REQ-AW-016, REQ-AW-C003..005, REQ-AW-NF003..004 | 两leg base/target精确；sibling dirty可继续；artifact main/control/sibling mutation为0；integration expected-OID漂移阻断artifact finish。 |
| ST-AW-004 | 作为审批者，我要由单一协调者聚合两个leg的独立结果，以便仅在全PASS时完成逻辑CR，并在部分失败时保留真实成功事实和恢复入口。 | P0 | REQ-AW-011..013, REQ-AW-016..017, REQ-AW-C004, REQ-AW-NF001, REQ-AW-NF005 | 聚合优先级=`BLOCKED > FAIL > IN_PROGRESS > PASS`；PARTIAL仅progress/effect；失败不自动关闭/回滚；main↔integration同步仅在CR外人工执行。 |
| ST-AW-005 | 作为迁移执行者，我要获得逐项目migration preflight和交接清单，以便后续自行搬迁文件和挂接软链接且本轮不发生真实迁移。 | P1 | REQ-AW-014..017, REQ-AW-C001, REQ-AW-NF005 | manifest含mapping/hash/link/readiness/验证/回滚；CR-051真实文件/link/worktree/ref变化为0。 |
| ST-MR-001 | 作为维护者，我要用 schema v2 external anchor 与事实驱动 health mode 唯一识别 legacy/project-first/conflict，以便兼容可读不会冒充已迁移。 | P0 | REQ-MR-001..005, REQ-MR-C003, REQ-MR-NF001,004 | v2 route可移植；legacy/v1/v2 dual-read、v2-only write；traversal/多解100%阻断；三种mode与exit-code确定。 |
| ST-MR-002 | 作为迁移执行者，我要先按冻结manifest规范 inherited seed，再激活 steady ownership，以便不会越界删除 sibling 内容或制造双写真相源。 | P0 | REQ-MR-006..008, REQ-MR-C002, REQ-MR-NF003 | exact prune集合=manifest；unexpected=0；shared-main/sibling ref/OID/hash不变；receipt前route activation=0。 |
| ST-MR-003 | 作为审计者，我要 leg/aggregate evidence 跨进程持久化并安全追加audit tail，以便进程重启后仍能重算且target OID不自引用。 | P0 | REQ-MR-009..011, REQ-MR-NF002,004 | opaque handle跨进程readback=100%；错归属/篡改拒绝；aggregate可复算；tail parent/target OID可验证。 |
| ST-MR-004 | 作为迁移准备协调者，我要以 intent、receipt、third-state 和幂等resume执行 transitional bootstrap，以便CR-053不会伪造native-first CP0。 | P0 | REQ-MR-012..015, REQ-MR-NF003,004 | intent先于mutation；逐步receipt完整；每故障点可resume且重复mutation=0；CP0显式import并保留transitional attribution。 |
| ST-MR-005 | 作为审批者，我要通用migration engine默认只plan，并以逐动作单次typed authorization控制execute，以便错OID、错digest、过期或重放不能触碰真实对象。 | P0 | REQ-MR-016..017, REQ-MR-C001,003..005, REQ-MR-NF001,004 | dry-run副作用0；错误授权拒绝率100%；真实仓mutation/publication默认禁用；CLI不复制状态机。 |
| ST-MR-006 | 作为迁移准备协调者，我要在临时三并列拓扑执行真实mutation和故障恢复，以便migration-ready结论不是只由mock/dry-run推断。 | P0 | REQ-MR-018..020, REQ-MR-C001..004, REQ-MR-NF003..005 | 临时E2E完整；half-push/staleOID/reader restart可恢复；fixture外mutation=0；scoped blocker=0且全局新增fingerprint=0。 |
| ST-MR-007 | 作为审批者，我要一份只引用已交付能力与证据的CR-053 readiness handoff，以便真实迁移缺少任何关键前置时自动NOT_READY。 | P0 | REQ-MR-015,020..021, REQ-MR-C001..005, REQ-MR-NF005 | critical refs覆盖率100%；缺任一能力/E2E证据即NOT_READY；handoff不新增核心逻辑、不授权真实mutation或publication。 |
| ST-VNEXT-001 | 作为项目负责人，我要每个项目恰好拥有一个现有发布库和一个独立过程库，以便切换项目时 sibling 的文档、ref、index 和状态变化为 0。 | P0 | REQ-VNEXT-001..003, REQ-VNEXT-NF001..002 | 两项目路由各为1+1、第三仓0、跨项目变化0、ownership错误0。 |
| ST-VNEXT-002 | 作为项目负责人，我要用 Project/Roadmap/Phase/Work 四层维护长期目标和单次交付，以便项目多年演进而不把每次执行全文写回长期对象。 | P0 | REQ-VNEXT-004..006, REQ-VNEXT-NF004 | 4/4层可解析、唯一父引用；每Work最多1个Phase+1个Roadmap投影，全文复制0。 |
| ST-VNEXT-003 | 作为 Host Orchestrator，我要以受控 Work 生命周期和过程 main expected-OID CAS 发布结果，以便并发写入不静默覆盖或自动 merge。 | P0 | REQ-VNEXT-007..008, REQ-VNEXT-016..017, REQ-VNEXT-C003, REQ-VNEXT-NF003 | 非法转移100%拒绝；并发恰好1成功1stale；唯一receipt；重复请求新commit0。 |
| ST-VNEXT-004 | 作为审批者，我要每个 Work 唯一路由到 G0/G1/G2，以便简单任务走轻流程，高风险任务仍有必要门禁。 | P0 | REQ-VNEXT-008..011, DQ-VNEXT-04 | G0/G1四项上限精确；高风险和G1超限100%转G2；静默降级0。 |
| ST-VNEXT-005 | 作为执行者和审计者，我要读写检查与 token 都受 Work scope 约束，以便上下文和验证成本随风险增长而不是默认全量。 | P0 | REQ-VNEXT-012..015, REQ-VNEXT-NF003..005 | scope外读写/检查0；G0≤8/8/3/32k、G1≤20/24/8/96k；telemetry诚实。 |
| ST-VNEXT-006 | 作为迁移执行者，我要只迁当前快照并保留旧共享仓只读，再完成2项目×2周期试点，以便低风险切换且不重写历史。 | P1 | REQ-VNEXT-018..021, REQ-VNEXT-C001..006, REQ-VNEXT-NF002,004..005 | 当前快照100%；旧仓新增写入0；历史拆分/无损转换0；至少4周期且每项目G0/G1各≥1。 |

## 推荐发布切片

| Slice ID | 切片名称 | 包含 Story | 用户价值 | 前置依赖 |
|---|---|---|---|---|
| SL-PG-00 | Second-system Guardrail | ST-PG-013 | 先冻结“不造第二套机制”的跨切面边界，后续所有整改切片默认复用既有 state/context/result/ledger/registry 体系 | 无 |
| SL-PG-01 | P0 State Enforcement | ST-PG-001, ST-PG-002, ST-PG-003 | 先关闭 current state 污染入口，避免后续治理对象继续被错误状态拖累 | SL-PG-00 |
| SL-PG-01A | P0.5 Ledger Hygiene Boundary | ST-PG-012 | 先冻结 ledger compact 与 state compact 的命令边界，避免后续实现混淆审计数据压缩与 current state 渲染 | SL-PG-01 |
| SL-PG-02 | P1 Project State Foundation | ST-PG-004, ST-PG-005 | 建立长期项目治理的最小机器状态和规模偏好 | SL-PG-01 |
| SL-PG-03 | P1 Reference Normalization | ST-PG-006, ST-PG-007 | 恢复 capability、feature、impact surface 的可检查语义 | SL-PG-02 |
| SL-PG-04 | P1 Roadmap Refresh | ST-PG-008, ST-PG-009, ST-PG-010 | 让 roadmap 变更可自动刷新过程状态，同时控制跨仓风险 | SL-PG-03 |
| SL-PG-05 | P2 quant-lab Migration | ST-PG-011 | 用真实长期项目验证治理机制，并输出迁移证据 | SL-PG-04 |
| SL-EI-01 | Evidence Integrity Core | ST-EI-001, ST-EI-002, ST-EI-003, ST-EI-004 | 先让 chronology、dispatch、CP correlation、state/read 边界成为可执行治理 contract | CR-046 CP2/CP3/CP5 |
| SL-EI-02 | Replay and Cost Observability | ST-EI-005, ST-EI-006 | 在可信核心上补齐 token telemetry 与 checker 双口径重放 | SL-EI-01 |
| SL-EI-03 | quant-lab CR-163 Acceptance Pilot | ST-EI-007 | 用 append-only 真实样本证明当前 checker 23/23 可重放 | SL-EI-02；独立 pilot 授权 |
| SL-WT-01 | Truth and Routing Closure | ST-WT-001, ST-WT-002, ST-WT-007 | 先统一机器真相、路由和历史当前状态 | CP2/CP3 |
| SL-WT-02 | Deterministic Quality Gate | ST-WT-003, ST-WT-004, ST-WT-005 | 让 Doctor、guardrail、lint 和回归形成确定门 | SL-WT-01 |
| SL-WT-03 | Operator Usability | ST-WT-006 | 提供 CI/Agent 非交互安装与 preflight | SL-WT-02 |
| SL-GB-01 | Safe Paired Branch Open | ST-GB-001 | 源码与过程证据从各自最新主分支进入同名、可恢复 CR branch | CR-050 CP2/CP3/CP5 |
| SL-GB-02 | Committed Ref Publication | ST-GB-002 | 只发布显式提交事实，并逐仓披露成功/失败 | SL-GB-01 |
| SL-GB-04 | Explicit Paired Fast-forward Merge | ST-GB-004 | 以独立授权和 fast-forward-only contract 完成两仓 default update，部分成功仍可恢复 | SL-GB-02；default-branch write 单独授权 |
| SL-GB-03 | Proof-gated Branch Cleanup | ST-GB-003 | merge 后重新观察 ancestry/tip 再安全删除，不把 merge result 当作删除授权 | SL-GB-04；2/2 merge PASS；delete 单独授权 |
| SL-AW-01 | Project-first Routing Compatibility | ST-AW-001 | 让每个项目只解析到自己的docs/process，并为后续逐项目迁移保留legacy兼容 | CR-051 CP2/CP3/CP5 |
| SL-AW-02 | Persistent Worktree and Integration Bootstrap | ST-AW-002 | 为每项目提供稳定working tree、长期integration、短期CR branch identity和create-only初始化 | SL-AW-01；CP2-DQ-01/03/06 resolved；待CP3/CP5 |
| SL-AW-03 | Heterogeneous Legs and Aggregate Gate | ST-AW-003, ST-AW-004 | 让多个项目并行执行source-default与artifact-integration双leg，并以单一聚合门完成逻辑CR，不触碰artifact main | SL-AW-02；CP2-DQ-04/05 resolved；待CP3/CP5 |
| SL-AW-04 | Migration Handoff without Mutation | ST-AW-005 | 给用户后续逐项目迁移提供manifest/checklist，同时保持本CR真实迁移为0 | SL-AW-03 |
| SL-MR-01 | External Route and Health Truth | ST-MR-001 | 先让系统正确区分“兼容可读”“已迁移”和“冲突”，为所有mutation建立fail-closed入口 | CR-052 CP2/CP3/CP5 |
| SL-MR-02 | Seed Normalization and Ownership Stage | ST-MR-002 | 把一次性 inherited seed prune 与长期 steady ownership 分离，避免越界删除和双写 | SL-MR-01；DQ52-5 |
| SL-MR-03 | Durable Evidence and Audit Tail | ST-MR-003 | 让leg/aggregate事实可跨进程恢复，同时不让证据尾写入污染完成语义 | SL-MR-01；DQ52-4 |
| SL-MR-04 | Transitional Bootstrap Coordinator | ST-MR-004 | 以显式transitional CP0解决route尚未原生化的bootstrap悖论，并支持故障续跑 | SL-MR-02, SL-MR-03；DQ52-3 |
| SL-MR-05 | Guarded Migration Runner | ST-MR-005 | 通过确定plan和逐动作单次授权控制ref/worktree/prune/link/push风险 | SL-MR-04；DQ52-9 |
| SL-MR-06 | Real Temporary E2E and CR-053 Handoff | ST-MR-006, ST-MR-007 | 用真实临时mutation、故障恢复和scoped check证明readiness，再把真实布局迁移交给CR-053 | SL-MR-05；DQ52-1/7/8/10；CP7/CP8 |
| SL-VNEXT-A | Per-project Dual-repo Isolation | ST-VNEXT-001, ST-VNEXT-003 | 先消除跨项目过程仓/working-tree联动，并以单写CAS提供可恢复写入基础 | CR-052 vNext CP2/CP3/CP5 |
| SL-VNEXT-B | Long-lived Governance and Work | ST-VNEXT-002 | 在隔离过程库上建立四层长期真相和克制投影，让日常工作可持续积累 | SL-VNEXT-A |
| SL-VNEXT-C | Risk-scoped Execution | ST-VNEXT-004, ST-VNEXT-005 | 让G0/G1真正减读、减写、减检查、减token，同时把高风险准确升级G2 | SL-VNEXT-B；DQ-VNEXT-04 |
| SL-VNEXT-D | Snapshot Migration and 2x2 Pilot | ST-VNEXT-006 | 用只迁当前态、旧仓只读和4个完整周期证明可切换、可回滚、跨项目变化0 | SL-VNEXT-C；DQ-VNEXT-01..05；独立迁移/远端授权 |

## 规划边界

- Story 只表达用户 outcome，不规定具体代码模块拆分。
- 每个 Story 必须回链至少一个 UC 和 REQ。
- P2 迁移必须等待 P0 / P1 机制可用后再执行。
- 任何发布库写入都不由 roadmap refresh 自动授权。
- CR-046 CP2 只确认产品/场景/范围基线，不授权实现、runtime、credentials、publish、commit/push 或 quant-lab business-code 修改。
- ST-EI-007 是 append-only process-evidence pilot，不得承担 quant-lab lineage 业务功能重做。
- CR-050 的 Story 只定义用户 outcome；具体 CLI、result schema、Git command plan 与恢复算法由 CP3/CP5 设计。
- `ST-GB-002` 不授权隐式 stage/commit；`ST-GB-004` 只允许独立、显式、fast-forward-only merge，不授权 merge commit/force/策略绕过；`ST-GB-003` 不授权 force-delete 或按 patch 相似度猜测已合并。
- `ST-AW-*` 是CR-051产品候选Story，不是CP4正式Story卡片；R3 pending decision items=0，但CP2 R3总体门未approve前不得据此生成HLD/LLD或实现。
- `ST-AW-005` 只交付migration preflight/manifest/手册；真实文件迁移、软链接挂接、真实worktree/ref操作由用户后续逐项目启动和授权。
- artifact branch必须带project identity，因为不同项目的CR ID可重复；产品层已冻结idle=`projects/<project-name>/integration`、active=`projects/<project-name>/cr/<cr-id>-<slug>`；artifact leg只回integration，shared main同步在CR外。CP3只细化attach/switch/finish/abort、expected OID、leg correlation、聚合schema、bootstrap CAS和branch cleanup。
- `ST-MR-*` 是 CR-052 产品候选 Story，不是 CP4 Story 卡片；DQ52-1..10 与 CP2 总体门未批准前不得形成正式 Feature/Story/LLD 或修改源码。
- CR-052 的“真实 mutation”只指隔离临时目录/local bare remote fixture；真实 meta-flow artifact 布局、link、worktree/ref、publication和 sibling项目内容始终在当前范围外。
- `ST-MR-007` 交付 readiness/handoff，不执行 CR-053；若缺任一 critical capability 或 E2E evidence，结论必须 NOT_READY。
- `ST-VNEXT-*` 是 CR-052 vNext R2 产品候选 Story，不是 CP4 Story 卡片；DQ-VNEXT-01..05 与 CP2 总体门未批准前不得进入 HLD、正式 Story、LLD 或源码实现。
- `SL-VNEXT-A..D` supersede `SL-MR-01..06` 作为 CR-052 当前候选交付顺序；旧切片只保留历史审计，不得并行实施。
- `SL-VNEXT-D` 只定义逐项目快照试点 outcome；CP2 不授权创建仓库、迁移文件、commit/push 或 production cutover。
