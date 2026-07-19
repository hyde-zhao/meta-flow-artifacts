---
status: confirmed
version: "3.2.1"
confirmed_by: "user"
confirmed_at: "2026-07-19T18:58:05+08:00"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
source_plan: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-STATE-ENFORCEMENT-IMPLEMENTATION-PLAN-2026-07-02.md"
baseline_note: "CR-052 vNext 用户旅程 R3.2.1 已由用户确认：以 P-08 项目负责人/Work 发起者和按需 P-04 Reviewer 的真实旅程为主体，形成 UC-VNEXT-UJ-001..016；覆盖项目双仓、弹性长期治理、问询式需求、Work/CR 与 G0/G1/G2、scope/token、执行/评审/验证/推送/恢复/查询、项目复盘、进化建议审议和有界自进化。旧 UC-VNEXT-001..006 保留历史并标记 superseded。用户授权开始本地实现并允许精简冗余流程；真实迁移、远端 publication、runtime/production 和破坏性 Git 操作仍需独立授权。"
engagement_mode: meta-self-dev
scenario_subject_type: implementation-carrier
scenario_subject_id: "meta-flow"
target_artifact_type: workflow
governance_mode: review-gated
review_policy: strict
delivery_routing:
  mode: meta-flow-delivery
  output_root: "process/docs/product"
  source: meta-self-dev
active_change_ref: "CR-052"
total_use_cases: 57
current_baseline: "CR-052-VNEXT-UJ-R3.2.1-confirmed"
---

# Meta Flow 项目治理与状态强制用户场景

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 3.2.1 | 2026-07-19 | host-orchestrator | 用户确认 `UC-VNEXT-UJ-001..016`，授权按精简 Meta Flow 流程开始本地实现；保留公共契约、真实迁移和远端写入的必要人工授权门。 | 场景基线从 draft 提升为 confirmed；不删除历史 UC |
| 3.2 | 2026-07-19 | meta-pm / host-orchestrator | 新增 `UC-VNEXT-UJ-014..016`：项目/阶段完成后以证据复盘流程规范、流程适配、质量、效率和 token；单独审议改进建议并批准有界进化包；在正常 Work/CR 生命周期中实现、试点、回测和回退 Meta Flow 自进化。 | 仅增量更新 USE-CASES；保持 draft；复盘报告不自动授权修改 Meta Flow |
| 3.1 | 2026-07-19 | meta-pm / host-orchestrator | CR-052 vNext 用户旅程返工：按真实用户旅程新增 UC-VNEXT-UJ-001..013；补入既有项目接入、Work 暂停/恢复/交接、Work 关闭最小回写、检查/CAS/push 中断恢复及按需读取项目真相五个遗漏；旧 UC-VNEXT-001..006 显式 superseded 并建立映射。 | 仅增量更新 USE-CASES；保持 draft，等待用户逐项确认后再启动其他产品基线 |
| 3.0 | 2026-07-19 | meta-pm | CR-052 vNext R2：新增 UC-VNEXT-001..006、SGA-VNEXT-01..04、SGQ-VNEXT-001、SM-VNEXT-01..12 与历史基线状态映射；把共享 artifact worktree migration-readiness 重构为每项目双库、四层治理、G0/G1/G2、Work scope/token 和快照迁移。保留全部既有 UC/SGA/SGQ/Deferred ID 与修订历史。 | 原文档增量 reframe；旧 GB/AW/MR 主线标记 superseded，CP2 人工门待批准 |
| 1.1 | 2026-07-02 | host-orchestrator | 同步 CR-037 已激活、CR-036 暂停未完成和 CP2 pending 不授权实现的状态语义 | 小范围状态语义同步 |
| 1.2 | 2026-07-11 | meta-pm | 为 CR-046 增量加入可信时序、平台调度证明、checker 重放、token telemetry 与 append-only 历史迁移场景；保留 UC-PG-001..007 | 原文档增量更新 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：显式加入 compaction 语义保持、通用 post-close correction、机器生成 audit report、null-provenance dogfooding 与 session-observed/repository-unverifiable dispatch 披露；保留全部 UC ID | 原文档增量更新 |
| 1.4 | 2026-07-13 | meta-pm | 为 CR-047 增量加入跨设备 workflow truth、canonical CR tracking、artifact/docs 路由、Doctor、clean-clone guardrail、Ruff、非交互安装与 CR-046 状态收敛场景；保留全部既有 UC ID | 原文档增量更新 |
| 1.5 | 2026-07-15 | host-orchestrator inline fallback | 为 CR-050 增量加入双仓 CR 分支开启、已提交 ref 发布、合并证明与安全清理场景；保留全部既有 UC ID，并把 Git/`gb`、双仓、merge 与 commit 边界提交 CP2 | 原文档增量更新 |
| 1.5.1 | 2026-07-16 | host-orchestrator inline fallback | 记录 SGA-GB-01..05 已由用户在 CP2 批准推荐方案；不改变 UC、指标、Deferred 或不授权范围 | CP2 状态同步 |
| 1.6 | 2026-07-16 | host-orchestrator inline fallback | 用户澄清 Meta Flow 需要在 publish 后提供独立 merge 能力；新增 UC-GB-004、SGA-GB-06 和量化指标，重开 SGA-GB-03，并将隐式/merge-commit/force 行为继续排除；等待 CP2 R2。 | 原文档增量更新；保留 1.5.1 基线与全部既有 UC ID |
| 1.7 | 2026-07-17 | meta-pm | 为 CR-051 增量加入 project-first artifact、每项目独立 worktree、项目作用域 Git 生命周期、共享 main 刷新与迁移前交接场景；保留全部既有 UC/SGA/SGQ/Deferred ID。 | 原文档增量更新；CR-050 历史事实不改写 |
| 1.8 | 2026-07-17 | meta-pm | CR-051 CP2 changes_requested R2：按用户决策把 idle detached 候选修订为长期 `projects/<project-name>/integration`，明确每 CR 使用 `projects/<project-name>/cr/<cr-id>-<slug>`、`main` 保持共享集成基线，并确认显式 merge-main refresh 与 existing-control+sibling-worktree 拓扑；既有 UC/REQ/TC/Story/Slice ID 不变。 | 原文档增量更新；保留 R1 与既有历史正文 |
| 1.9 | 2026-07-18 | meta-pm | CR-051 CP2 changes_requested R3：不新增 UC/SGA/SGQ/Deferred ID；以异构 source/artifact 双 leg、单一聚合门、integration create-only 初始化和 CR 外人工 main/integration 同步替代 R2 的 per-CR artifact main refresh 当前语义；R2 决策保留为历史并标注 superseded。 | 原文档增量更新；保留 R1/R2 历史追溯，R3 为当前候选基线 |
| 2.0 | 2026-07-19 | meta-pm | 为 CR-052 增量新增 UC-MR-001..007、SGA-MR-01..04、SGQ-MR-001、SM-MR-01..10 与 Deferred；冻结“能力补齐在 CR-052、真实迁移在 CR-053”的产品边界，保留全部 CR-051 UC/SGA/SGQ/Deferred ID 和修订历史。 | 原文档增量更新；CR-052 CP2 人工门待批准 |
| 1.0 | 2026-07-02 | meta-pm | 基于已批准实施计划建立产品侧场景基线 | 初始化长期可追踪产品基线 |

## 用户画像

| 画像 ID | 角色名称 | 典型背景 | 核心诉求 | 技术水平 |
|---|---|---|---|---|
| P-01 | Host Orchestrator 维护者 | 维护 meta-flow 状态机、检查点、CR 和交付规则 | 防止轻量状态膨胀，确保推进流程可审计、可恢复 | 高级 |
| P-02 | 功能 Agent / Skill 作者 | 编写或维护 meta-flow agent、skill、规则和 CLI | 清楚知道哪些状态可写、如何写、写错时如何被拦截 | 中高级 |
| P-03 | 项目迁移执行者 | 将 quant-lab 等长期项目迁移到新的治理机制 | 在不污染发布库的前提下完成状态清理、路线图刷新和能力引用归一 | 高级 |
| P-04 | 审批者 / Reviewer | 审批高风险流程、迁移计划和治理变更 | 快速看到自动更新范围、人工决策项、风险和回退条件 | 中高级 |
| P-05 | 工作流审计者 | 复核 checkpoint、gate、dispatch、state 与 ledger 的一致性 | 以机器证据重建实际发生顺序，并区分平台证明、仓库证明与不可用证据 | 高级 |
| P-06 | 证据生产者 / 成本分析者 | 维护自动检查、dispatch producer、context/read 记录与成本报告 | 让每次 attempt、checker 身份、输入摘要和 token 用量具有一致且不可伪造的语义 | 高级 |
| P-07 | 迁移准备协调者 | 负责在不触碰真实布局的前提下验证迁移计划、bootstrap、证据和恢复能力 | 获得可重放、可恢复、可审计的 migration-ready 证明，再把真实迁移交给独立 CR | 高级 |
| P-08 | 项目负责人 / Work 发起者 | 同时推进多个长期项目，需要低成本启动、执行、审查和关闭日常 Work | 每个项目过程真相物理隔离；按风险获得足够而不过量的流程、上下文和检查 | 中高级 |

> **当前 active 场景角色约束**：UC-VNEXT-UJ-001..016 的用户故事主体只使用真实用户 `P-08`，需要审批或评审时加入 `P-04`。Host Orchestrator、Agent、checker、publisher 等只作为系统协作者出现在处理逻辑中，不作为用户故事主体；其他画像仅保留历史场景追溯。

## 成功指标

| 指标 ID | 指标名称 | 度量方式 | 目标值 |
|---|---|---|---|
| SM-01 | Current state 瘦身合规 | `STATE.current.json` schema / size / unknown field 检查 | audit 阶段可报告，enforce 阶段可阻断非法写入 |
| SM-02 | 项目级治理对象可追踪 | `PROJECT.current.json`、`PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml` 引用完整性检查 | 关键 refs 完整，`PROJECT.current.json` 不超过 16KB |
| SM-03 | 影响面语义归一 | 新 CR 的 `impact_surface` 只包含治理面枚举，路径进入 `affected_paths` | 新 CR enforce 阶段 unknown surface 为 0 |
| SM-04 | Roadmap refresh 边界清晰 | refresh result 中自动写入项、must_check、stale_items、follow_up_candidates 可区分 | 不自动修改 quant-lab 发布库 |
| SM-05 | quant-lab 迁移可验证 | 迁移后 state check、capability check、feature check、capability-claims check 结果 | P2 迁移报告可复现、失败项可追踪 |
| SM-06 | 时序负例阻断 | chronology negative fixtures 的 checker 结果 | 非法 gate / dispatch / result / state 顺序 100% 被拒绝 |
| SM-07 | Evidence attempt 完整性 | execution/check attempt 的 terminal status、supersession 与 final correlation 覆盖率 | 适用 attempt 覆盖率 100%，final result 只引用最终 terminal attempt |
| SM-08 | Telemetry 诚实性 | usage `measurement_status` 与 measured/proxy 字段检查 | 适用记录覆盖率 100%；不可用写 `unavailable`，估算值不得标为 measured |
| SM-09 | 历史重放可复现 | checker identity、schema/policy hash、as-executed/current-replay 报告 | quant-lab CR-163 current replay 23/23 PASS；lineage 业务源码 diff 为 0 |
| SM-10 | Compaction 语义保持 | compact/restore 前后关系图和 terminal selection 对比 | event/dispatch/attempt/run 标识不混淆；attempt、correction、workflow-health refs 语义差异为 0 |
| SM-11 | Post-close correction 可审计 | versioned correction schema、允许字段、supersedes closure 与 audit trail 检查 | 非法历史改写 100% 拒绝；合法 correction 100% 可独立重放 |
| SM-12 | Audit report 统计可信 | 机器生成报告与已知 fixture 逐维对账 | event rows、attempts、threads、terminal outcomes 与 measured/proxy/unavailable token 指标 100% 正确，且携带 checker provenance/input hashes |
| SM-WT-01 | 跨设备 workflow truth 一致 | 两台 clean clone 分别 link 同一版本 artifact 后比对 state/current/CR index | `active_change`、phase、pending gate、canonical CR 状态差异为 0 |
| SM-WT-02 | Canonical CR tracking 可执行 | `meta-flow check cr-tracking --project-root .` | 退出码 0；canonical 目录无 legacy YAML 干扰；CR-033 仅以 candidate 身份可追踪 |
| SM-WT-03 | Workspace / docs 路由确定 | clean clone + `workspace link` + `workspace check` | `process_link_health=ok`；内部产品文档只有 1 个 canonical artifact 路径；根 `docs/` 继续只承载公开入口 |
| SM-WT-04 | Doctor 阻断归零 | `meta-flow doctor all --project-root .` | 退出码 0；阻断错误为 0；warning 单独披露且不冒充 blocker |
| SM-WT-05 | 静态质量门闭合 | `ruff check .` 与既有全量 pytest | Ruff error=0；不少于 377 个测试及 70 个 subtests 全部通过 |
| SM-WT-06 | Clean-clone guardrail 自洽 | `git archive HEAD` 等价 clean tree 运行 guardrail | 不依赖 ignored 根规则或本机 cache；guardrail 退出码 0 |
| SM-WT-07 | 非交互安装入口可复现 | Codex / Claude / Qoder project-scope full dry-run | 3/3 命令显式提供 `--project-dir` 且退出码 0 |
| SM-WT-08 | CR-046 状态收敛诚实 | 对产品矩阵、CR、CP7 与 release 状态做交叉检查 | 7/7 Story 显示 implemented + `PASS_WITH_RISK`；平台 receipt / 独立 QA 不被写成已具备 |
| SM-GB-01 | CR 分支起点一致 | 对 project/artifact 两仓记录 remote default tip、local branch tip 与 upstream | 2/2 仓 branch tip 等于刷新后的 remote default tip，upstream 建立率 100% |
| SM-GB-02 | 发布只包含已提交事实 | 比对 local HEAD、remote CR ref、dirty/index 状态 | 2/2 远端 CR ref 等于本地 HEAD；dirty 或未提交变更场景 100% 阻断 |
| SM-GB-03 | 合并后删除证明充分 | `merge-base --is-ancestor`、exact ref/tip 与删除结果 | 未证明祖先关系的远端删除数为 0；已证明且无漂移的目标分支清理率 100% |
| SM-GB-04 | 双仓部分失败可恢复 | 注入任一仓 fetch/push/delete 失败并检查结构化结果 | 每仓均有 terminal status、已执行命令和恢复入口；partial success 不被报告为 PASS |
| SM-GB-05 | Dry-run 零副作用 | dry-run 前后 local/remote refs、HEAD、worktree hash 对比 | local/remote ref 变化数为 0，计划步骤覆盖率 100% |
| SM-GB-06 | 显式双仓快进合并 | 对两仓 fresh remote default、published CR tip、merge result 与 branch retention 做交叉检查 | 2/2 default branch 仅以 fast-forward 更新；merge commit/rebase/force/自动冲突解决执行数为 0；部分成功时被删 CR branch 数为 0 |
| SM-AW-01 | 项目路由唯一性 | 对 project identity、layout version、worktree registry 与解析结果做交叉检查 | 每个已登记项目恰好解析到 1 个 `<project>/docs` 和 1 个 `<project>/process`；歧义解析 100% 阻断 |
| SM-AW-02 | 项目 worktree 隔离 | 在两个项目 worktree 中并行执行 integration→CR→integration 的本地 fixture Git 周期 | 两个 idle worktree 均驻留各自 `projects/<project>/integration`；活动 CR 分支均匹配 `projects/<project>/cr/<cr-id>-<slug>`；sibling dirty 错误阻断、跨项目 touched path/branch/ref 和 index lock 争用均为 0 |
| SM-AW-03 | 当前项目保护 | 对当前项目 worktree dirty、identity/path mismatch 与 branch collision 负例执行 preflight | 危险 mutation 前阻断率 100%；误操作其他项目 worktree/ref 数为 0 |
| SM-AW-04 | 异构双 leg 聚合确定 | 注入 source/artifact leg 的 PASS、FAIL、BLOCKED、IN_PROGRESS 组合及 artifact integration expected-OID 漂移 | 聚合优先级严格为 `BLOCKED > FAIL > IN_PROGRESS > PASS`；仅全部必需 leg PASS 才整体完成；`PARTIAL` 只表达进度/影响，失败不自动回滚或关闭已成功 leg |
| SM-AW-05 | Legacy 兼容确定性 | 对未迁移 `process/<project>` / `docs/<project>` 与新 `<project>/process|docs` fixture 重放 | legacy/new layout 均可按显式版本解析；同一配置重复解析结果一致率 100%；不静默切换写目标 |
| SM-AW-06 | 能力开发零真实迁移 | 对 CR-051 touched paths、真实 artifact tree、软链接和 worktree/ref 快照做前后比较 | 现有 artifact 文件搬迁数、软链接变更数、真实 worktree/branch/ref mutation 数均为 0 |
| SM-MR-01 | External route 可移植且封闭 | 在不同 sibling 根和恶意 traversal fixture 中解析 schema v2 route | 合法 v2 external anchor 解析成功率 100%；绝对路径、越界 `..` 和未声明 anchor 接受数为 0 |
| SM-MR-02 | Health 模式判定诚实 | 对 legacy、project-first 和冲突布局矩阵重复执行 workspace health | 三种模式判定一致率 100%；`route-conflict` 退出码恒为非零；legacy 不被误报为 migrated |
| SM-MR-03 | Durable evidence 可恢复 | 在 writer 进程退出后由独立 reader 重读 leg/aggregate evidence 和 opaque handle | 适用证据跨进程 readback 成功率 100%；缺失、篡改或错归属 evidence 被拒绝率 100% |
| SM-MR-04 | Transitional bootstrap 可续跑 | 在 intent、mutation、receipt import 各边界注入失败并重复 resume | 每个故障点均产生唯一 third-state 与幂等 resume；重复 mutation 和误报 PASS 数为 0 |
| SM-MR-05 | Seed prune 与 steady ownership 分离 | 对 manifest-bound seed 和额外 sibling 内容执行 normalize/activate fixture | 只删除 manifest 精确列出的 inherited seed；unexpected deletion/add/modify 数为 0；无 prune receipt 时 steady route 激活数为 0 |
| SM-MR-06 | Typed authorization 防重放 | 对 action/ref/OID/plan digest/expiry/single-use 组合运行负例 | 缺失、过期、错对象、错 OID、错 digest 和重复授权拒绝率 100% |
| SM-MR-07 | 临时真实拓扑 E2E 完整 | 在 source、artifact control、project worktree 三并列目录和 local bare remote 执行真实 mutation、故障注入与恢复 | 规定的 E2E 路径全部形成 terminal evidence；partial 被误报 PASS 数为 0；fixture 外 mutation 数为 0 |
| SM-MR-08 | Evidence tail 不污染完成语义 | 对 aggregate 完成后的 audit-tail 写入检查 parent/target OID、dirty state 和 readback | receipt 的 parent/target OID 100% 可复核；证据尾写入不会把未完成 leg 投影为完成 |
| SM-MR-09 | Scoped CR 检查不被历史噪声淹没 | 比较 CR-052 scoped result 与冻结的全局 error fingerprint | CR-052 scoped blocker=0；全局新增 fingerprint=0；既有历史错误不被静默改写 |
| SM-MR-10 | CR-053 只消费已交付能力 | 校验 readiness handoff、能力版本、验证证据和真实 mutation denylist | CR-053 前置引用覆盖率 100%；在 CR-052 内真实布局/link/worktree/ref/publication mutation 数为 0 |
| SM-VNEXT-01 | 每项目双库唯一 | 对项目注册、发布库路由和过程库路由做唯一性检查 | 每个试点项目恰好 1 个发布库路由 + 1 个独立过程库路由；新增第三仓数为 0 |
| SM-VNEXT-02 | 跨项目隔离 | 在两个试点项目间交替激活 Work，并比较 sibling 项目的 branch/ref/path/state/hash | 切换项目导致 sibling 项目变化数为 0；共享 checkout、working tree、index 和过程分支数均为 0 |
| SM-VNEXT-03 | 路由唯一 | 对 project identity、release repo 和 process repo 解析结果重复检查 | 每个项目 release/process route 唯一数均为 1；0 解或多解在写入前 100% 阻断 |
| SM-VNEXT-04 | 弹性治理层级合法 | 校验项目选择的 `Project→Roadmap→Phase→Work`、`Project→Phase→Work` 或 `Project→Work` profile 及父子引用 | 用户选择的层级 100% 可解析；孤儿 Work、多父归属和未获确认的空壳层级接受数为 0 |
| SM-VNEXT-05 | Work 投影克制 | 比较 Work 关闭前后适用的 Phase/Roadmap 长期对象变更 | 每个 Work 最多更新 1 个所属 Phase 投影和 1 个 Roadmap 进度投影；所选 profile 不含对应层时写入数为 0；复制 Work 全文次数为 0 |
| SM-VNEXT-06 | G0 资源上限 | 统计 G0 Work 的默认读取文件、允许写入文件、检查组和总 token | 每个 G0 Work ≤8 个读取文件、≤8 个写入文件、≤3 组检查、≤32,000 token；静默超限数为 0 |
| SM-VNEXT-07 | G1 资源上限 | 统计 G1 Work 的默认读取文件、允许写入文件、检查组和总 token | 每个 G1 Work ≤20 个读取文件、≤24 个写入文件、≤8 组检查、≤96,000 token；静默超限数为 0 |
| SM-VNEXT-08 | Work scope 执行 | 比较声明的 read/write/check scope 与实际访问/变更/检查 | scope 外读取、scope 外写入和未声明检查执行数均为 0；扩读/扩写均有理由和升级记录 |
| SM-VNEXT-09 | 过程 main CAS | 对单写 publisher 注入 expected-OID 漂移与并发更新 | stale expected OID 拒绝率 100%；静默覆盖和自动 merge 数为 0；每次成功写入恰有 1 个 publisher receipt |
| SM-VNEXT-10 | 快照迁移只迁当前态 | 比较新过程库初始快照、旧共享过程仓与历史转换输出 | 当前快照对象覆盖率 100%；历史拆分数、旧 CP/CR/Story 无损转换数和旧共享仓新增写入数均为 0 |
| SM-VNEXT-11 | 试点覆盖 | 统计试点项目与完成的 Work 周期 | 恰好 2 个试点项目，每项目至少 2 个完整 Work 周期，总计至少 4 个周期；每项目至少覆盖 1 个 G0 和 1 个 G1 Work |
| SM-VNEXT-12 | 未授权操作为零 | 审计远端 publication、批量迁移、force/history rewrite、runtime/production 操作 | 当前 CR 内上述执行数均为 0；CP2 approve 不改变该不授权结论 |

## 明确排除

- 不新增第二套上下文治理、影响分析、capability 命名、CP result 或 ledger 体系。
- 不声明或实现跨仓原子事务。
- 不把 roadmap refresh 自动写入 quant-lab 发布库代码、测试或正式文档。
- 不直接修改 `process/policies/GATE-PROFILES.json` 来表达 project scale。
- 不让 `capability_refs` 或 `feature_refs` 成为自由字符串命名空间。
- 不把 `PROJECT.current.json` 设计成新巨型状态文件。
- 不伪造历史 platform receipt、签名、token telemetry 或 checker identity。
- 不修改 quant-lab lineage contract、recorder、producer、consumer 或 admission 业务实现。
- 不让 CP2 approval 隐式授权 credentials、runtime、production write、publish、交易、commit 或 push。
- 不把 shell alias `gb`、同名 Go 构建工具或 Git Town 设为 Meta Flow 必需依赖。
- 不把 merge 隐式塞进 `publish` 或 `finish`；不创建 merge commit，不自动审批 PR、调用 forge API、猜测 squash/rebase 已合并或执行 force-push/force-delete。
- 不隐式 `git add -A`、不替用户选择提交范围；branch publication 只发布已提交 refs。
- 不在 CR-051 中搬迁任何现有 artifact 文件、修改现有软链接，或在真实 `meta-flow-artifacts` 上创建/删除 worktree、branch、commit、tag 或 remote ref。
- 不让 sibling project 的 dirty 状态、branch 或 index 成为当前项目的默认阻断条件；也不允许当前项目命令读取、暂存、提交或清理 sibling project 路径。
- 不把 project-first 解释为“每项目独立 artifact 仓库”；共享 Git object database 与 remote 仍保留，每项目通过 namespace、worktree 和 branch identity 隔离。
- 不在 CR-052 迁移真实 `meta-flow` artifact 文件或软链接，也不创建真实 `projects/meta-flow/integration`、artifact CR ref 或 project worktree。
- 不读取、发现、迁移或修改 `ptm-team`、`ptm-atomic`、`quant-lab` 内容；sibling 隔离只用合成 fixture 证明。
- 不全局放宽 `..`、不接受任意绝对路径、不用单个总授权覆盖 ref/worktree/prune/link/push，也不允许过期或重复使用授权。
- 不在 CR-052 执行 repository publication、shared-main 内容写入、main↔integration 同步、force/tag/reset/rebase/orphan、自动 merge/冲突解决或跨 leg 自动回滚。
- vNext 首版不拆分旧共享过程仓 Git 历史，不把每项目历史重写到新过程库。
- vNext 首版不做旧 CP/CR/Story 的无损语义转换；旧对象仅通过只读索引和来源引用保留。
- 不实现多个 publisher 对同一过程库 `main` 的分布式自动 merge；并发写入只允许 expected-OID CAS 失败后重新观察。
- 不批量迁移其他项目；每个项目必须单独确认快照、路由、回滚和 cutover。
- 不让 CP2 产品批准隐式授权 repository publication、凭据、runtime、production、publish、live 或 trading。

## Scenario Gray Areas

本轮输入来自已批准实施计划，灰区已在设计评审中收敛；产品基线保留以下场景取舍，供后续 CP2 / CR 审批追踪。

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 用户选择 | 状态 |
|---|---|---|---|---|---|
| SGA-01 | `STATE.current.json` 使用黑名单还是 allowlist | 决定能否阻止下一个自造字段继续污染轻量状态 | 范围 / 验证 / 后续门控 | allowlist schema + 字段预算 | resolved |
| SGA-02 | Roadmap refresh 是否跨仓自动更新 | 决定 quant-lab 发布库是否被自动改写以及回滚边界 | 范围 / 运行风险 / 交付出口 | 只自动更新过程归档库，发布库只输出 follow-up | resolved |
| SGA-03 | capability / feature 引用是否允许自由字符串 | 决定迁移后冲突检测和能力追踪是否可信 | 验证 / 维护成本 / 数据治理 | 必须引用标准 registry | resolved |
| SGA-04 | 项目规模是否引入更多档位 | 影响 gate profile、状态复杂度和审批负担 | 范围 / 复杂度 / 后续门控 | 使用 `lite / standard / full` 三档 | resolved |
| SGA-05 | 平台没有可验证 receipt 时是否推断为已证明 | 决定 dispatch provenance 是否可信 | 安全 / 审计 / 降级 | 明确记录 `unavailable`，保留 session-observed 与 repository-verifiable 分层，不合成 receipt | resolved |
| SGA-06 | token telemetry 缺失时能否用估算替代实测 | 决定成本报告是否误导审批与优化 | 度量 / 审计 / 用户信任 | `measured`、`estimated`、`unavailable` 分离；估算仅作 proxy | resolved |
| SGA-07 | quant-lab CR-163 历史证据如何修正 | 决定迁移能否保留事实链和可追溯性 | 迁移 / 回滚 / 验证 | append-only correction / supersession / migration events；不改写原事件 | resolved |
| SGA-08 | checker 重放只看当前结论还是保留执行时结论 | 决定历史证据能否解释 checker 演进 | 兼容性 / 审计 / 发布 | 同时记录 checker version/commit/schema-policy hash 与 as-executed/current-replay 双口径 | resolved |
| SGA-WT-01 | 根 `AGENTS.md` 的 canonical source 是 tracked 根文件，还是 tracked `delivery/rules/AGENTS.md` + generated wrapper | 决定 clean clone 能否独立通过 guardrail，以及个人配置是否会混入共享规则 | 交付出口 / 维护成本 / clean-clone 验证 | tracked `delivery/rules/AGENTS.md` + generated root wrapper | resolved-by-user / CP2-DQ-01 |
| SGA-WT-02 | Doctor 的绿色是否要求零 warning，还是只要求零 blocking error 并完整披露 warning | 决定历史兼容提示、空 ledger 等 warning 是否阻断发布 | 验证 / 门控 / 发布结论 | blocking error=0；warning 计数与披露 | resolved-by-user / CP2-DQ-02 |
| SGA-WT-03 | 历史超预算证据采用 compact summary + archive/hash，还是原位压缩 | 决定历史不可变性、读取预算和恢复成本 | 审计 / 维护 / 回退 | active/default-read 严格预算；closed 历史用 summary/index/hash/correction/archive，不改写 | resolved-by-user / CP2-DQ-03 |
| SGA-WT-04 | 平台 receipt、独立 QA 与真实 pilot 缺失是否纳入本轮强行“补绿” | 决定是否会把不可用平台事实伪装为仓库证明 | 安全 / 权限 / 发布 | 固定为不伪造；保留 `READY_WITH_RISK` / follow-up，真实运行需独立授权 | resolved |
| SGA-GB-01 | 使用原生 Git、`gb` 还是 Git Town | 决定安装依赖、跨平台可移植性、同步/force-push 策略和可测试性 | 复杂度 / 验证 / 交付 | 复用现有原生 Git subprocess service；外部工具只保留 adapter 候选 | resolved-by-user / CP2-DQ-01 |
| SGA-GB-02 | CR 分支只管理源码仓还是 project + artifact 成对管理 | 决定过程证据能否与源码分支在另一设备恢复到同一 CR | 范围 / 失败恢复 / 交付 | 两仓同名 CR 分支；不宣称跨仓原子事务 | resolved-by-user / CP2-DQ-02 |
| SGA-GB-03 | Meta Flow 是否提供 merge，还是只验证外部 merge 后清理 | 决定完整旅程是否仍需操作者手工完成关键步骤，以及 default branch 写入如何单独授权 | 安全 / 门控 / 回滚 | 提供独立显式 `merge`；不得由 `publish`/`finish` 隐式触发，只允许 fast-forward-only | reopened-by-user / CP2-R2-DQ-01 |
| SGA-GB-04 | squash/rebase merge 后是否用 patch 相似度猜测已合并 | 决定远端删除是否有充分证明 | 验证 / 数据保留 / 风险 | fail closed；未来由 forge receipt adapter 处理 | resolved-by-user / CP2-DQ-04 |
| SGA-GB-05 | `cr-publish` 是否隐式 stage/commit 工作树 | 决定提交边界是否可能夹带无关文件或秘密 | 安全 / 可审计性 / 易用性 | 只推送已提交 refs；commit 继续由显式 Git/Host 操作完成 | resolved-by-user / CP2-DQ-05 |
| SGA-GB-06 | 两仓 merge 的顺序、部分成功和 default-branch 写入授权如何处理 | Git 不提供跨仓事务；第一仓成功后第二仓可能被 branch protection 或并发推进拒绝 | 失败恢复 / 权限 / 删除安全 | 先预检两仓，再按 artifact→project 合并；每仓单独授权/结果；任一失败保留两仓 CR branch，禁止 finish | decision-item / CP2-R2-DQ-02..04 |
| SGA-AW-01 | artifact 继续按类型优先 `docs/<project>` / `process/<project>`，还是迁移到项目优先 `<project>/docs` / `<project>/process` | 决定所有路由、ownership、迁移和文档发现契约 | 范围 / 兼容 / 交付出口 | 用户选择 project-first；当前只开发能力，真实迁移后续逐项目执行 | resolved-by-user / SGQ-AW-001 |
| SGA-AW-02 | 每项目 worktree 长期常驻、每 CR 临时创建，还是二者混合 | 决定软链接稳定性、清理成本、并发模型和 branch 占用 | 复杂度 / 维护 / 失败恢复 / 后续门控 | 长期保留每项目 worktree；空闲驻留 `projects/<project-name>/integration`，CR 期间使用 `projects/<project-name>/cr/<cr-id>-<slug>`；`main` 只作为共享集成基线 | resolved-by-user / CP2-R2-DQ-01 |
| SGA-AW-03 | 一个逻辑 CR 的 source/artifact legs 是否采用相同 base/target，以及如何判定整体完成 | 决定 artifact CR 是否错误接触 shared main、部分成功是否被误报完成，以及失败时是否发生跨仓回滚 | 安全 / 可恢复性 / 验证 / 后续门控 | source leg 从/回源码默认分支；artifact leg 从/回项目 integration；单一协调者按 `BLOCKED > FAIL > IN_PROGRESS > PASS` 聚合，仅全 PASS 完成，`PARTIAL` 不是终态 | resolved-by-user / CP2-DQ-04..05；supersedes CP2-DQ-02 |
| SGA-AW-04 | project integration 如何首次建立，以及 shared main 与 integration 如何同步 | 决定首次接入是否覆盖既有项目历史、main divergence 是否错误阻断 CR、同步是否越过当前 CR 授权 | 兼容 / 运维 / 回退 / 授权 | integration 缺失时从 fresh `origin/main` exact OID create-only 初始化；已存在时禁止 recreate/reset/orphan；双向同步仅由 CR 外人工维护，默认要求该项目无活跃 artifact CR；existing control+sibling-root 拓扑继续有效 | resolved-by-user / CP2-DQ-03/06 |
| SGA-MR-01 | 能力补齐与真实布局迁移是否拆成 CR-052 / CR-053 | 决定临时拓扑可验证性与真实仓 mutation 的授权、回滚、审计边界是否混在一起 | 范围 / 验证 / 授权 / 后续门控 | 推荐拆分；用户“按照你的计划实施”确认启动该计划，但不替代 CP2 正式批准 | decision-item / DQ52-1 / SGQ-MR-001 |
| SGA-MR-02 | external route 采用隐式 `workspace_parent` 还是显式 `workspace_root` | 决定跨设备可移植性、拓扑适用范围和错误配置的 fail-closed 方式 | 架构 / 兼容 / 验证 / 维护成本 | 推荐 schema v2 + `workspace_parent=project_root.parent`；不满足同父目录拓扑时切换显式 workspace_root | decision-item / DQ52-2 |
| SGA-MR-03 | durable evidence 采用两阶段 out-of-band store + project-local audit tail，还是纯 tracked store | 决定 aggregate 完成后证据是否制造 dirty tail、跨进程/跨机器恢复边界和审计成本 | 架构 / 可靠性 / 审计 / 后续门控 | 推荐两阶段 evidence；跨机器 durability 与 audit-tail OID 契约由 CP3 冻结 | decision-item / DQ52-4 |
| SGA-MR-04 | transitional bootstrap、seed prune 与 typed authorization 如何共同约束真实 mutation | 决定首次迁移能否在 route 尚未原生化时安全启动，以及删除/链接/ref 操作能否被重放或越权 | 安全 / 失败恢复 / 实现复杂度 / 授权 | 推荐显式 transitional CP0、manifest-bound prune、steady ownership 后置激活和逐动作单次授权 | decision-item / DQ52-3/5/8/9 |
| SGA-VNEXT-01 | 发布库是复用项目现有源码/交付仓，还是为 vNext 再建独立发布仓 | 决定仓库数量、迁移成本和用户对“每项目双库”的理解是否稳定 | 范围 / 复杂度 / 交付出口 / 后续门控 | 推荐复用现有源码/交付仓，使每项目总计恰好 2 个逻辑仓库 | decision-item / DQ-VNEXT-01 |
| SGA-VNEXT-02 | 过程库采用 main-only 单写 publisher + expected-OID CAS，还是延续每 Work/CR 分支和多写者 merge | 决定长期治理能否真正消除分支/working-tree 联动与并发覆盖 | 架构 / 可靠性 / 维护成本 / 验证 | 推荐 main-only 单写 publisher；CAS 漂移 fail closed，不做自动 merge | decision-item / DQ-VNEXT-02 |
| SGA-VNEXT-03 | G0/G1 的 read/write/check/token 默认上限如何取值，超限何时升级 G2 | 决定轻量 Work 是否真的减负，以及复杂 Work 是否会因预算过小失真 | 范围 / 成本 / 验证 / 后续门控 | 推荐 G0=8读/8写/3检查/32k token；G1=20读/24写/8检查/96k token；超限或高风险进入 G2 | decision-item / DQ-VNEXT-04 |
| SGA-VNEXT-04 | 首版迁移历史还是只迁当前快照，旧共享仓保留多久只读 | 决定切换风险、历史语义转换成本和回滚窗口 | 迁移 / 兼容 / 回退 / 维护成本 | 推荐只迁当前快照；旧仓永久只读索引，至少跨越 2×2 试点和 30 天观察期后再评估退役 | decision-item / DQ-VNEXT-03, DQ-VNEXT-05 |

## CR-050 用户可见场景确认证据

| Question ID | 问题 | 选项 / 候选理解 | 推荐方案 | 用户回答 | 复述确认 | 影响面 | 来源 | 状态 |
|---|---|---|---|---|---|---|---|---|
| SGQ-GB-001 | CR 生命周期是否应覆盖“刷新主分支 → 创建并推送 CR 分支 → 提交后发布 → 远端合并后删除分支”的完整旅程？ | 用户自由表达；安全细节进入 CP2 | 覆盖完整旅程，并对 destructive 步骤 fail closed | “开启cr时从远端主分支拉取最新代码，创建cr分支，然后提交推送到远程分支，然后将远程分支合并到后将分支删除掉” | 核心旅程已由用户明确；工具/双仓/merge 证明/commit 边界由 CP2 决策，不将模糊处静默解释为 force 或自动 merge 授权 | scope / validation / security / gate | 用户请求 / CR-050 | confirmed |
| SGQ-GB-002 | 当前 Git 生命周期缺少 merge 操作，是否需要让 Meta Flow 在 publish 后显式合并？ | 独立 `merge`、保持外部 merge、或 forge adapter | 增加显式两仓 fast-forward-only merge，默认不隐式触发 | 用户询问“该需要实现后可以支持推送后合并分支吗”，并回复 `approve` 接受推荐边界 | `publish` 仍只推送 CR ref；`merge` 需要独立调用和 default-branch-write 授权；只允许快进，两仓部分成功保留分支并阻断 `finish` | scope / authorization / recovery / gate | 用户对话 / CR-050 CP3 changes requested | confirmed-for-CP2-R2 |

## CR-051 用户可见场景确认证据

**Discussion Log**：`process/discussions/CP2-CR051-SCENARIO-DISCUSSION-LOG.md`

**Checkpoint**：`process/checks/CP2-CR051-DISCUSSION-CHECKPOINT.json`

| Question ID | 问题 | 选项 / 候选理解 | 推荐方案 | 用户回答 | 复述确认 | 影响面 | 来源 | 状态 |
|---|---|---|---|---|---|---|---|---|
| SGQ-AW-001 | 多项目共享 artifact 时，是否改为 project-first 并让不同项目使用独立 worktree？ | A. `<project>/docs|process` + 每项目 worktree；B. 保持 `docs|process/<project>` + 单 worktree；C. 每项目独立仓库 | A：共享仓库保留、项目 namespace 与 working tree 隔离 | “方案就按项目优先的方案来做……meta-flow-artfacts 的目录采用不同的 worktree” | 目标语义冻结为 `<project_name>/docs` 与 `<project_name>/process`；不同项目不再共享同一 checkout/index/branch，但仍共享 artifact Git 仓库和 remote | scope / compatibility / validation / delivery | 用户对话 / CR-051 | confirmed |
| SGQ-AW-002 | 本轮是立即迁移所有 artifact 和软链接，还是先交付能力再逐项目迁移？ | A. 先开发能力、后逐项目迁移；B. 能力与全量迁移一次完成；C. 仅写迁移文档 | A：先完成 meta-flow 能力，真实迁移独立执行 | “后续我逐个项目完成 meta-flow-artfacts 文件迁移和软连接挂接” | CR-051 只交付路由、worktree、Git 周期、兼容读取、preflight 与迁移手册；真实文件搬迁、软链接挂接和真实 ref mutation均不在本轮授权内 | scope / authorization / rollback / gate | 用户对话 / CR-051 | confirmed |
| SGQ-AW-003 | 长期项目 worktree、每 CR 分支、source/artifact 双 leg 与共享 `main` 应分别承担什么职责？ | A. 异构双 leg：source 默认分支↔source CR，artifact integration↔artifact CR，shared main 同步在 CR 外；B. 两 leg 都回各自 default/main；C. artifact CR 内刷新 shared main | A：保持项目稳定 integration、隔离跨项目同步责任，并以单一聚合门维持逻辑 CR 完整性 | R2 先确认长期 integration、短期 CR 与 existing-control+sibling-root；R3 用户进一步明确“按照这个方案，实施”：source 从/回源码默认分支，artifact 从/回项目 integration，不接触 artifact main；采用单一聚合门与 create-only integration 初始化 | 当前生效理解为异构双 leg；CP2-DQ-02 的 per-CR merge-main 仅保留历史并由 CP2-DQ-04 supersede；聚合仅全 PASS 完成，shared main↔integration 同步为 CR 外人工维护 | architecture boundary / lifecycle / recovery / validation / authorization | 用户对话 / CR-051 CP2 changes_requested R2-R3 / `CP2-CR051-R3-USER-DECISIONS.json` | confirmed-for-CP2-R3 |

## CR-052 用户可见场景确认证据

**Discussion Log**：`process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md`

**Checkpoint**：`process/checks/CP2-DISCUSSION-CHECKPOINT.json`

| Question ID | 问题 | 选项 / 候选理解 | 推荐方案 | 用户回答 | 复述确认 | 影响面 | 来源 | 状态 |
|---|---|---|---|---|---|---|---|---|
| SGQ-MR-001 | 是否按整改计划把“迁移能力补齐与临时真实演练”放在 CR-052，把真实 meta-flow artifact 布局迁移放在 CR-053？ | A. 拆分 CR-052/CR-053，并要求 CR-052 临时拓扑真实 mutation E2E；B. 合并能力与真实迁移；C. CR-052 仅 dry-run | A：分离能力验证与生产 mutation 的授权/回滚边界 | “按照你的计划实施” | 已确认可按 A 启动 CR-052、建立产品/设计/验证基线并在临时仓执行受控真实 mutation 测试；该原答不等于 DQ52-1..10 或 CP2/CP3/CP5/CP8 的正式批准，也不授权真实仓布局、worktree/ref/link mutation或 push | scope / validation / authorization / gate / rollback | 用户原答；`process/REQUEST.md` CR-052 增量请求；CR-052 §CP2待确认方向 | confirmed-for-scenario-baseline |

## CR-052 vNext R2 用户可见场景确认证据

**Discussion Log**：`process/discussions/CP2-CR052-VNEXT-R2-SCENARIO-DISCUSSION-LOG.md`

**Checkpoint**：`process/checks/CP2-CR052-VNEXT-R2-DISCUSSION-CHECKPOINT.json`

| Question ID | 问题 | 选项 / 候选理解 | 推荐方案 | 用户回答 | 复述确认 | 影响面 | 来源 | 状态 |
|---|---|---|---|---|---|---|---|---|
| SGQ-VNEXT-001 | CR-052 是否应放弃“共享 artifact worktree migration-readiness”为当前目标，改为每项目双库隔离、四层长期治理、G0/G1/G2 与 Work-scoped 资源控制，并以快照方式迁移？ | A. 按 vNext 推荐方向重构产品基线；B. 保持旧 MR 主线；C. 只写概念文档暂不形成正式流程 | A：优先简单、可靠、低治理成本，并把历史转换和批量迁移后置 | 用户明确要求接受推荐方向并开始正式流程 | 当前仅确认 vNext 是应进入正式 CP2 的候选场景基线；发布库复用、main-only publisher/CAS、预算数值、快照兼容期和试点边界仍进入 DQ-VNEXT-01..05，不视为 CP2 全部批准，也不授权实现、迁移或 push | scope / architecture / validation / migration / gate | `process/REQUEST.md`“CR-052 vNext 重解释请求”；`CR-052.vnext-reframe.summary.json` | confirmed-for-scenario-baseline |

## Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-01 | 更丰富的项目规模矩阵或 `regulated` 独立档 | 设计评审否决项 | 当前可复用 runtime-high-risk、authz、evidence、human gate | 真实受监管项目出现现有 gate 无法表达的合规缺口 |
| DEF-02 | 跨仓事务式 roadmap refresh | 设计评审否决项 | 跨仓原子性成本高且回滚边界不清 | 未来有可靠跨仓事务协调器和明确授权 |
| DEF-03 | 长期消费 Markdown register 或 Python 常量作为 capability registry | P1.2a 备选 | 不利于稳定引用和 checker 实现 | 标准 YAML registry 无法覆盖某类能力状态时重新评审 |
| DEF-04 | 复用 CP result checker 校验 roadmap refresh | P1.4 备选 | result 语义不同，复用会污染 CP 检查模型 | roadmap refresh 与 CP result 出现大量共享字段和统一生命周期时 |
| DEF-EI-001 | 跨平台统一加密签名 receipt | SGA-05 | 平台能力并不一致，本轮只能诚实表达 receipt 可用性和证据层级 | 所有目标平台提供稳定可验证签名契约后另行立项 |
| DEF-EI-002 | 用估算 token 建立计费或配额门禁 | SGA-06 | 估算不是平台实测，不能成为强制成本决策依据 | 平台 telemetry 覆盖稳定且误差模型经独立验证后重新评审 |
| DEF-WT-001 | 平台签发的 custom-agent / model / profile receipt 与独立 runtime attestation | CR-045/046 遗留风险、SGA-WT-04 | 当前平台 surface 不提供仓库可验证 receipt，本 CR 不授权真实 runtime 或 SaaS | 平台公开稳定 receipt contract，且用户另行授权真实运行验证 |
| DEF-WT-002 | CR-033 MF-018 runtime trace / SaaS 真实验证 | follow-up candidate | 缺 C0 runtime target、凭据边界和独立运行授权 | 用户正式启动新 CR，提供 runtime target 与最小权限授权 |
| DEF-GB-001 | Forge API/receipt adapter 支持 squash/rebase merge 后的可证明清理 | SGA-GB-04 | 需要 GitHub/GitLab 等平台身份、token、PR receipt 与权限模型，本轮不授权 | 用户启动独立 CR，选择目标 forge，并提供最小权限与 receipt contract |
| DEF-GB-002 | Git Town adapter、stacked branch 和 offline ship 工作流 | SGA-GB-01 | 当前目标是普通 CR 分支；引入 Git Town 会增加配置、同步与升级面 | 出现真实 stacked-branch 需求，且原生 Git adapter 无法满足时重新评审 |
| DEF-GB-003 | 自动 stage/commit 与提交内容规划 | SGA-GB-05 | 自动收集工作树可能夹带无关文件或敏感内容，且双仓提交消息/范围需要独立契约 | 用户明确要求该能力，并批准 path allowlist、secret scan 与提交回滚策略 |
| DEF-AW-001 | 全量自动迁移所有项目 artifact 与软链接 | SGQ-AW-002 | 用户选择能力先行、逐项目人工受控迁移；一次性迁移会扩大回滚和跨项目污染面 | 用户启动具体项目迁移 CR，并确认源/目标 mapping、备份与回滚 |
| DEF-AW-002 | 将现有 control checkout 转为 bare control repo | SGA-AW-04 备选 | 迁移前优先兼容现有 clone，避免本轮先重构仓库存储形态 | sibling worktree 模式被证明确有 control checkout 污染或运维成本问题 |
| DEF-AW-003 | 自动执行 shared main↔project integration 同步，或以 rebase/force-with-lease 改写项目历史 | SGA-AW-03/04 未选备选 | 当前基线把同步明确放在 CR 外人工维护；自动化会扩大跨项目、force 与回滚授权面 | 人工同步成本经真实多项目试点证明不可接受，且用户通过新 CR 批准同步策略、活跃 CR 门与 branch-scoped 高风险授权 |
| DEF-MR-001 | CR-053 执行真实 meta-flow artifact 文件迁移、软链接切换和 project-first route 激活 | SGQ-MR-001 / DQ52-1 | CR-052 只建设并验证能力；真实对象 mutation 有独立授权、回滚和审计边界 | CR-052 CP8 READY/READY_WITH_RISK，全部临时 E2E 证据可读，用户正式启动并批准 CR-053 |
| DEF-MR-002 | 显式 `workspace_root` 作为非同父目录拓扑的 route anchor | SGA-MR-02 / DQ52-2 | 默认 `workspace_parent` 是满足当前三并列拓扑的最小契约 | 真实目标拓扑无法由 `project_root.parent` 唯一定位，且 CP3 证明显式 root 的可移植/安全契约 |
| DEF-MR-003 | shared-main README-only 合规更新 | DQ52-10 | CR-052 不授权 shared-main 内容写入或 repository publication | 用户另行批准 README-only 变更的目标 ref、内容边界和 publication |
| DEF-MR-004 | ptm-team、ptm-atomic、quant-lab 的 project-first 迁移 | 用户明确排除 / DQ52-10 | sibling 项目不在本轮范围，且禁止读取其内容做 discovery | 各项目分别启动迁移 CR，提供自身 mapping、ownership、备份、授权和回滚 |
| DEF-MR-005 | evidence backend 的跨机器共享/复制协议 | SGA-MR-03 / DQ52-4 | CR-052 P0 必须先保证 durable cross-process readback；跨机器持久化方式仍需 CP3 明确边界 | CP3 证明本地 content-addressed store 无法满足 CR-053 恢复，或出现多设备恢复的强制场景 |
| DEF-VNEXT-001 | 将旧共享过程仓历史按项目拆分到新过程库 | vNext 首版范围取舍 | 历史重写成本高且容易损害审计语义，当前快照足以启动隔离治理 | 2×2 试点完成后出现无法通过只读索引解决的强制审计需求 |
| DEF-VNEXT-002 | 旧 CP/CR/Story 无损语义转换器 | vNext 首版范围取舍 | 旧对象版本与语义异构，自动转换会把未知事实伪装为等价 | 至少两个项目证明只读索引无法支持必需恢复，并形成版本化转换契约 |
| DEF-VNEXT-003 | 多 publisher 分布式自动 merge | SGA-VNEXT-02 未选备选 | 单写 publisher + CAS 更简单、可预测；自动 merge 会重新引入冲突治理 | 单 publisher 吞吐在连续两个周期超过已批准 SLO 且人工重试成为主要瓶颈 |
| DEF-VNEXT-004 | 批量迁移所有项目 | SGA-VNEXT-04 | 每项目的路由、快照、回滚和 cutover 不同，批量化会扩大爆炸半径 | 2×2 试点全部通过且用户另行批准项目清单、顺序和逐项目回滚 |
| DEF-VNEXT-005 | 未授权远端 publication 或 production cutover | 当前不授权边界 | CP2 只确认产品基线，不能隐式授予外部写入/运行权限 | 对具体仓、ref、expected OID、目标环境和回滚获得独立授权 |

## CR-052 vNext 历史基线状态映射

| 历史 ID 范围 | vNext R2 状态 | 当前处理 | vNext 承接 |
|---|---|---|---|
| `UC-PG-*`、`UC-EI-*`、`UC-WT-*` | reframed | 保留已验证的状态、证据、路由和检查能力事实，但不再以共享 artifact 拓扑或全量治理链作为当前产品目标 | UC-VNEXT-002..005 |
| `UC-GB-*` | superseded | Git 分支生命周期保留为历史交付能力；vNext 过程库日常 Work 不以成对分支/默认分支 merge 为治理主线 | UC-VNEXT-001, UC-VNEXT-003 |
| `UC-AW-*` | superseded | project-first namespace + shared artifact worktree 方案被“每项目独立过程库”替代；不得恢复为当前默认 | UC-VNEXT-001, UC-VNEXT-006 |
| `UC-MR-*` | superseded | migration-readiness 能力包不再是 CR-052 当前交付目标；其中快照、校验、恢复经验仅作为重构输入 | UC-VNEXT-006 |
| `UC-VNEXT-001..006` | superseded | R2 以系统能力分组，缺少从项目接入、Work 确认到提交、关闭、查询、复盘、自进化和恢复的完整用户旅程；保留正文作为历史，不再作为当前 active 场景 | UC-VNEXT-UJ-001..016，详见下方逐项映射 |
| 旧历史拆分、无损转换、批量迁移、自动 merge | deferred | 不进入 vNext 首版 MVP，保留重启条件 | DEF-VNEXT-001..005 |

### R2 系统场景到 R3 用户旅程映射

| R2 历史场景 | R3 当前承接场景 |
|---|---|
| `UC-VNEXT-001` 双库边界 | `UC-VNEXT-UJ-001` 新项目初始化、`UC-VNEXT-UJ-002` 已有项目接入 |
| `UC-VNEXT-002` 四层治理 | `UC-VNEXT-UJ-003` 轻量长期基线、`UC-VNEXT-UJ-011` Work 关闭与长期投影 |
| `UC-VNEXT-003` Work 生命周期 | `UC-VNEXT-UJ-004` 需求确认、`005` Work/CR 判定、`007` 执行/交接、`011` 关闭、`012` 恢复 |
| `UC-VNEXT-004` 风险路由 | `UC-VNEXT-UJ-005` Work/CR 判定、`006` G0/G1/G2、`008` 评审、`009` 验证 |
| `UC-VNEXT-005` scope/token | `UC-VNEXT-UJ-004`、`006`、`007`、`008`、`009`、`010`、`011`、`013` 的逐场景资源边界 |
| `UC-VNEXT-006` 快照迁移 | `UC-VNEXT-UJ-002` 已有项目接入、`UC-VNEXT-UJ-012` 失败恢复 |

## 使用场景列表

### UC-PG-001：拒绝污染 `STATE.current.json` 的状态写入

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-02 功能 Agent / Skill 作者 |
| 触发条件 | Agent、Skill、CLI 或内部流程准备更新 `process/state/STATE.current.json` |
| 输入 | 待写入 patch、actor、reason、当前 `STATE.current.json` |
| 处理逻辑 | 系统按 allowlist 校验顶层字段，按字段预算校验大小和结构；audit 阶段输出 WARN，enforce 阶段拒绝 unknown / 超预算 / required key 缺失写入 |
| 输出/结果 | 合法写入成功；非法写入产生可追踪错误和检查结果 |
| 前置条件 | allowlist schema、field budgets、受控 update API 已存在 |
| 排除情况 | 不迁移重型状态到 `STATE.current.json`；不允许 agent 直接手工编辑 current state |

### UC-PG-002：以 refs-only 项目状态表达长期项目治理

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 工作流需要表达项目定位、规模、当前项目阶段、活跃目标和 roadmap refs |
| 输入 | 项目 ID、项目定位短字段、project scale、roadmap / milestone / health / deferred index refs |
| 处理逻辑 | 系统在 `STATE.current.json` 仅保留 `project_state_ref`，在 `PROJECT.current.json` 保存短字段和 refs，并校验预算 |
| 输出/结果 | 项目级状态可被下游读取，但不会膨胀 current state |
| 前置条件 | `process/project/` scaffold、`PROJECT.current.json` schema 和预算存在 |
| 排除情况 | 不在 `PROJECT.current.json` 保存 roadmap 全文、deferred 详情或长历史 |

### UC-PG-003：按项目规模提供 gate profile 默认偏好

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 新建或迁移长期项目，需要表达 `lite / standard / full` 项目规模 |
| 输入 | 项目规模、规模原因、gate profile bias refs |
| 处理逻辑 | 系统写入 `PROJECT-SCALE.yaml`，记录默认 gate profile bias 和原因，但不修改 `GATE-PROFILES.json` |
| 输出/结果 | 后续 CR / gate 可读取规模偏好，并保留人工审查边界 |
| 前置条件 | `process/project/PROJECT-SCALE.yaml` 模板和检查逻辑存在 |
| 排除情况 | 不新增五档规模矩阵；不把 project scale 当成自动授权 |

### UC-PG-004：归一 capability / feature 引用来源

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-01 Host Orchestrator 维护者 |
| 触发条件 | CR、roadmap refresh 或迁移流程声明 `capability_refs` / `feature_refs` |
| 输入 | 待引用 capability ID、feature ID、标准 registry |
| 处理逻辑 | 系统校验 capability ID 必须存在于 `CAPABILITY-STATUS.yaml`，feature ID 必须存在于 `FEATURE-REGISTRY.yaml`；缺失时输出 blocked finding 或 FU-RF follow-up |
| 输出/结果 | 引用可解析；无法解析的迁移项不会被自动创造 ID |
| 前置条件 | 标准 capability / feature registry 路径和 checker 已定义 |
| 排除情况 | 不把示例值或历史自由字符串直接升级为正式 ID |

### UC-PG-005：拆分治理影响面、路径和能力引用

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-03 项目迁移执行者 |
| 触发条件 | 新 CR 或迁移报告需要描述影响范围 |
| 输入 | 原始 impact surface 值、文件路径、模块路径、feature / capability 线索 |
| 处理逻辑 | 系统将有限治理面写入 `impact_surface`，路径写入 `affected_paths`，可解析能力写入 `feature_refs` / `capability_refs`；历史漂移进入 migration report |
| 输出/结果 | 冲突检测能基于语义 surface 工作，路径和能力引用各自可检查 |
| 前置条件 | impact surface 枚举、迁移报告格式、新 CR audit / enforce 策略存在 |
| 排除情况 | 不在普通 `cr check` 中对历史 CR 刷屏；不自动创造 feature / capability ID |

### UC-PG-006：执行受边界约束的 roadmap refresh cascade

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 项目定位、milestone、CR 完成或 roadmap 输入发生变化 |
| 输入 | source CR、refresh trigger、当前 project state、roadmap / milestone 对象、发布库 stale 线索 |
| 处理逻辑 | 系统生成独立 ROADMAP-REFRESH result；只自动更新过程归档库机器状态、project 对象、CR index 和 gate ledger；发布库变更只列入 must_check、stale_items、follow_up_candidates |
| 输出/结果 | refresh decision 为 `NO_CHANGE / UPDATED / UPDATED_WITH_DOC_IMPACTS / BLOCKED / FAILED` 之一，并可被 checker 校验 |
| 前置条件 | ROADMAP-REFRESH schema、checker、GATE-LEDGER 事件写入规则存在 |
| 排除情况 | 不复用 CP result checker；不自动修改 quant-lab 发布库 |

### UC-PG-007：迁移 quant-lab 到项目治理基线

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-04 审批者 / Reviewer |
| 触发条件 | P0 / P1 机制实现后，开始真实样本迁移 |
| 输入 | quant-lab 当前 `STATE.current.json`、capability 来源、历史 CR impact surface、roadmap / milestone 线索 |
| 处理逻辑 | 系统清理 current state，归一 capability registry，迁移历史 impact surface，生成 project 状态对象，输出 stale report 和 FU-RF 候选 |
| 输出/结果 | quant-lab 通过 state / capability / feature / capability-claims 检查，发布库只收到 follow-up |
| 前置条件 | P0 / P1 机制和 checker 已可用 |
| 排除情况 | 不与能力实现 CR 混在一起；不自动修改 quant-lab 发布库正式代码或文档 |

### UC-EI-001：重建可信的 gate 与结果时序

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-01 Host Orchestrator 维护者 |
| 触发条件 | 自动检查、条件式批准、人工门、state finalization 或 ledger 事件需要被审计或重放 |
| 输入 | checkpoint result、human checkpoint、gate/state/ledger 事件、check attempt 与输入 artifact hash |
| 处理逻辑 | 校验事件因果顺序、条件式批准前置、check attempt、supersession、最终 dispatch correlation 和跨真相源一致性；compaction/restore 还必须保持 event/dispatch/attempt/run 标识、terminal selection、correction chain 与 workflow-health refs；非法顺序或语义丢失 fail-closed |
| 输出/结果 | 可机器重建的有效时序与 provenance-bearing 机器 audit report，或带对象 ID、冲突字段和路由建议的阻断 finding；报告分开统计 event rows、attempts、threads、terminal outcomes 与 measured/proxy/unavailable token 指标 |
| 前置条件 | 事件时间、attempt ID、correlation refs 与 checker contract 可用 |
| 排除情况 | 不以文件修改时间代替事件因果；不回填伪造的历史批准时间 |

### UC-EI-002：证明平台调度与 attempt 生命周期

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-06 证据生产者 / 成本分析者 |
| 触发条件 | 功能 agent 被 spawn/resume/send_input，发生 retry、supersession 或 terminal closure |
| 输入 | dispatch event、platform receipt（若平台提供）、agent/thread ID、attempt 状态和 supersession refs |
| 处理逻辑 | 区分 platform-attested、session-observed、repository-verifiable 与 unavailable；要求每个 attempt 最终 closed，并让 retry 指向被替代 attempt；仅在当前 session 观察到 agent/thread/tool 且仓库无平台 receipt 时必须披露 `session-observed/repository-unverifiable` |
| 输出/结果 | 可追踪到终态的 dispatch chain；缺 receipt 时显式 `unavailable` 或 `session-observed/repository-unverifiable`，不得升级为 repository-verified/platform-attested |
| 前置条件 | producer 能写 attempt/correlation 字段；平台 receipt 能力按实际报告 |
| 排除情况 | 不合成平台签名或 receipt；handoff 文件本身不等于真实调度 |

### UC-EI-003：用已识别 checker 双口径重放证据

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-03 项目迁移执行者 |
| 触发条件 | 历史 CP/CR 证据需要解释执行时结果，或用当前 checker 评估兼容性 |
| 输入 | evidence bundle、checker version/commit、schema/policy hash、执行时结论与当前 checker |
| 处理逻辑 | 保留 as-executed 结论和 checker identity，再运行 current-replay；对差异使用通用、版本化、append-only post-close correction lifecycle，限制允许修正字段和范围，并记录 author/reason/evidence/supersedes 与独立 audit trail；CR-046 CP1/CP2 原始 null-provenance results 保留为 dogfooding 输入，不静默改写 |
| 输出/结果 | 同时包含 as-executed/current-replay、checker provenance、输入 hash 和差异分类的 replay report；null provenance 在 strict profile 下失败或明确标为 legacy/unavailable；合法 correction 可独立校验 |
| 前置条件 | 历史证据可读；无法识别的 checker 字段可明确标为 unavailable |
| 排除情况 | 不以当前 PASS 改写历史 FAIL/PARTIAL；不让 legacy YAML 干扰 canonical JSON checker |

### UC-EI-004：诚实度量工作流 token 成本

| 字段 | 内容 |
|---|---|
| 使用角色 | P-06 证据生产者 / 成本分析者、P-04 审批者 / Reviewer |
| 触发条件 | agent turn、checker run、context expansion 或阶段交接需要报告成本 |
| 输入 | 平台报告 token usage（若有）、measurement status、proxy estimate、阶段/CR/agent attribution |
| 处理逻辑 | 将 measured、estimated/proxy 和 unavailable 分栏；只把平台报告值计为 measured，并按 CR/阶段/agent/attempt 聚合；机器 audit report 不得以 dispatch event rows 代替 attempt/thread 数或推断 token 占比 |
| 输出/结果 | 可比较且不会把估算冒充实测的 workflow cost report，以及带 checker provenance/input hashes 的机器 audit report |
| 前置条件 | telemetry 字段和 attribution contract 已定义 |
| 排除情况 | 不从文本长度反推“实测 token”；不因 telemetry 缺失阻止如实记录 unavailable |

### UC-EI-005：以 append-only 方式迁移 quant-lab CR-163 证据

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-05 工作流审计者 |
| 触发条件 | Meta Flow 的 evidence-integrity contract 已实现并进入 CR-163 验收试点 |
| 输入 | quant-lab CR-163 既有 process evidence、migration manifest、当前 checker 和 immutable business-code boundary |
| 处理逻辑 | 先消费通用 post-close correction lifecycle，再仅追加 migration/correction/supersession events 与 replay fixture，校验 23 个目标证据，并检查 lineage 业务源码 diff 为 0；pilot 不得自创专用 correction 语义 |
| 输出/结果 | 23/23 current replay PASS 的可重放试点证据，或逐项阻断 finding；原历史行保持不变 |
| 前置条件 | 相应 Story 已通过设计门且迁移路径获得独立授权 |
| 排除情况 | 不修改或重做 quant-lab lineage 业务实现；不访问 credentials、runtime 或 production data |

### UC-WT-001：让 state、current 与 canonical CR index 收敛到同一事实

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-05 工作流审计者 |
| 触发条件 | clone、link、恢复工作流或执行 CR/state 检查 |
| 输入 | `STATE.current.json`、`CURRENT.json`、canonical `CR-INDEX.json`、正式 CR 生命周期状态 |
| 处理逻辑 | 以 State v2 与 JSON index 为机器入口，检查 active CR 存在且未关闭、CURRENT refs 一致、candidate 不被当作 active，并拒绝 legacy YAML 干扰 |
| 输出/结果 | 单一可执行 workflow truth；已关闭 CR 不可能继续作为 active，CR-033 保持 candidate |
| 前置条件 | `process` 路由健康，canonical JSON index 可读 |
| 排除情况 | 不用 Markdown summary 覆盖机器真相；不把 candidate 自动激活 |

### UC-WT-002：在 clean clone 中确定性建立 process 与文档路由

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-03 项目迁移执行者 |
| 触发条件 | 同时 clone `meta-flow` 与 `meta-flow-artifacts` 后首次初始化工作区 |
| 输入 | project root、artifact root、project name、`.meta-flow-process.yaml` |
| 处理逻辑 | 从源码项目执行一次 `workspace link`，使用锚点 + 相对路径记录路由；内部产品/设计/质量文档写入 artifact process 路径，根 `docs/` 保持公开入口 |
| 输出/结果 | `process_link_health=ok`，不同设备看到同一版本的过程真相源，不创建第二套 `docs/product` |
| 前置条件 | 两个仓库存在且 artifact 仓包含 `process/meta-flow` |
| 排除情况 | 不在 artifact root 上创建自指链接；不处理 prelink backup |

### UC-WT-003：让 Doctor 结果区分阻断、warning 与历史审计事实

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 发布前、CP8 前或跨设备恢复后执行 `doctor all` |
| 输入 | artifact budgets、QUALITY-MODEL、read-expansion ledger、Story/Run ledgers、历史 CP result |
| 处理逻辑 | 修复 active budget 与 policy 路径错误；历史证据通过 append-only correction、compact summary、index/hash 或 archive 处理；warning 单列，不把可接受 legacy 状态误报成 blocker |
| 输出/结果 | Doctor 无阻断错误，warning 数量与原因可审计，历史机器结果不被静默改写 |
| 前置条件 | budget / retention / quality policy 可读 |
| 排除情况 | 不截断或原位美化历史 CP result；不伪造缺失授权或 run 时间 |

### UC-WT-004：让 clean-clone guardrail 不依赖 ignored 本机文件

| 字段 | 内容 |
|---|---|
| 使用角色 | P-02 Agent / Skill 作者、P-04 审批者 / Reviewer |
| 触发条件 | clean tree、CI 或发布前运行 delivery guardrail |
| 输入 | tracked canonical rules、generated root wrapper 契约、Git tracked files、cache policy |
| 处理逻辑 | 按 CP2 选择统一规则 source；guardrail 在 clean clone 可独立发现 canonical source，并区分 tracked cache 与 ignored 本机 cache |
| 输出/结果 | `git archive HEAD` 等价环境 guardrail 退出码 0；个人配置不污染共享规则 |
| 前置条件 | canonical-source 决策已批准 |
| 排除情况 | 不把本机个人 Memory Policy 提交为共享规范；不以 ignored 文件作为唯一必需输入 |

### UC-WT-005：把 Ruff 与回归测试纳入确定性质量门

| 字段 | 内容 |
|---|---|
| 使用角色 | P-02 Agent / Skill 作者、P-04 审批者 / Reviewer |
| 触发条件 | 代码变更、发布前检查或 CI |
| 输入 | Python 源码、Ruff 配置、既有 pytest / subtest suite |
| 处理逻辑 | 安全自动修复可机械处理项，人工审查 B/F 语义项；随后运行 Ruff 和完整回归，并把命令纳入发布前组合门 |
| 输出/结果 | Ruff error=0；不少于 377 tests + 70 subtests 通过；失败会阻断发布就绪 |
| 前置条件 | 使用 `uv run --python 3.11` |
| 排除情况 | 不用测试通过替代 lint；不在产品阶段规定具体代码 diff |

### UC-WT-006：为 Agent/CI 提供完整的非交互安装与 cache preflight

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、自动化调用方 |
| 触发条件 | CI、Agent 或非 TTY shell 按 README 执行 project-scope 安装 dry-run |
| 输入 | 平台、component、scope、显式 `--project-dir`、cache policy |
| 处理逻辑 | README 为 Codex/Claude/Qoder 提供等价非交互命令；preflight 对 tracked cache 阻断，对 ignored cache 按批准策略清理或 warning |
| 输出/结果 | 3/3 平台 dry-run 退出码 0；测试后不会因 ignored cache 形成不可解释的永久红灯 |
| 前置条件 | 安装器主体可用，目标目录可读 |
| 排除情况 | 不执行真实用户级安装、外部发布或凭据操作 |

### UC-WT-007：收敛 CR-046 产品状态但保留恢复时序与风险

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-04 审批者 / Reviewer |
| 触发条件 | 产品矩阵、Feature matrix、CR index、CP7 与 release 状态出现陈旧或冲突 |
| 输入 | CR-046 formal CR/index、7 个 Story/LLD/CP6/CP7、产品矩阵与 `READY_WITH_RISK` 风险 |
| 处理逻辑 | 将 current automation 和 closure 状态回写为已实现/已验证带风险，保留 recovered/post-hoc 语义和原始 hash；平台 receipt、独立 QA、token telemetry 与真实 pilot 继续明确 unavailable/未授权 |
| 输出/结果 | 7/7 Story 状态可追踪，产品矩阵不再写 CP2 pending，发布结论仍为 `READY_WITH_RISK` |
| 前置条件 | 正式 CR-046 与 CP7 result 可读 |
| 排除情况 | 不倒填事前批准，不把 fixture rejection 写成真实平台 receipt，不重新打开历史时序 |

### UC-GB-001：从刷新后的远端主分支开启成对 CR 分支

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-03 项目迁移执行者 |
| 触发条件 | 正式 CR 已创建且 route plan 允许进入分支开启动作 |
| 输入 | CR ID、可选 slug、remote、可选 default-branch override、project/artifact workspace 路由 |
| 处理逻辑 | 对两仓执行 Git/route/clean/detached/branch-collision precheck；刷新 remote refs；以 remote HEAD 或显式 override 识别主分支；仅允许 `pull --ff-only`；从 exact remote default tip 创建同名 CR branch 并 `push -u`。任一仓失败即输出 partial/blocking result，不执行 force 补偿 |
| 输出/结果 | 两仓 CR branch、upstream、base OID 和逐仓执行结果可审计；dry-run 只输出计划 |
| 前置条件 | 两仓 Git repo 与 process route 健康；remote 可读写；工作树 clean；不存在分叉或同名 ref 冲突；真实 push 已单独授权 |
| 排除情况 | 不 reset/rebase/force；不自动 stash；不处理 `process/quant-lab/**` 或 prelink backup |

### UC-GB-002：发布已经显式提交的 CR 分支变更

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-02 功能 Agent / Skill 作者 |
| 触发条件 | CR branch 上的源码与过程产物已由显式 Git/Host 操作提交，准备发布到同名远端 branch |
| 输入 | CR ID、预期 branch、两仓 local HEAD/upstream、可选 expected OID |
| 处理逻辑 | 验证两仓当前分支、upstream、clean 状态和非快进风险；只推送已提交 refs；逐仓核验 remote ref 等于 local HEAD。dirty、detached、wrong branch 或 non-fast-forward 均 fail closed |
| 输出/结果 | 两仓 remote CR ref 与本地 HEAD 一致，或结构化 partial/failure 结果与恢复建议 |
| 前置条件 | 提交范围已由显式 `git add`/`git commit` 或等价受控操作决定；repository publication 已授权 |
| 排除情况 | 不隐式 stage/commit/amend；不自动选择提交文件；不 force-push |

### UC-GB-003：仅在合并证明充分后安全清理 CR 分支

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 托管平台或人工 Git 流程已把两仓 CR branch 合入各自远端主分支，准备关闭 CR refs |
| 输入 | CR ID、branch、remote/default branch、recorded/local/remote tip OID、两仓状态 |
| 处理逻辑 | 重新 fetch；验证 default branch 受保护、目标 ref 身份和 tip 未漂移；要求 CR tip 是 remote default tip 的祖先；先保留本地恢复指针并删除 exact remote CR ref，再用 `branch -d` 删除 local ref；刷新主分支只允许 ff-only。远端已自动删除时，仍必须由 local/recorded tip 完成 ancestry 证明 |
| 输出/结果 | 两仓目标 CR refs 不存在、main 未被 destructive 改写、删除证据与逐仓状态可审计；无法证明时 BLOCKED 且 refs 保留 |
| 前置条件 | 合并采用 ancestry-preserving 方式，或未来存在受信 platform receipt adapter；remote delete 已单独授权 |
| 排除情况 | `finish` 不自动或隐式 merge；不删除 main/master/perennial/protected refs；不按 patch-id 猜测 squash/rebase 已合并；不 force-delete |

### UC-GB-004：显式把两仓 CR tip 快进到各自远端默认分支

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | project/artifact 两仓 CR branch 已通过 `publish`，操作者显式请求 merge，并为 default-branch write 提供独立授权 |
| 输入 | CR ID、两仓已发布 exact tip、fresh remote default tip、remote/default branch、授权引用、可选 expected OID |
| 处理逻辑 | 先对两仓完成只读 preflight，证明 remote CR ref 等于已发布 tip、default 未漂移且可 fast-forward；随后按 artifact→project 的确定顺序把 exact CR tip fast-forward 到 remote default。每仓操作后重新读取远端 OID；禁止 merge commit、rebase、force、自动冲突解决和隐式分支删除 |
| 输出/结果 | 2/2 remote default 精确等于各自 CR tip时整体 PASS；第一仓成功、第二仓被并发推进/保护策略/权限拒绝时整体 PARTIAL，保留两仓 CR branch、记录逐仓 before/after OID 和恢复入口，并阻断 `finish` |
| 前置条件 | 两仓 preflight 全部通过；`publish` 证据可读；真实 default-branch write 已独立授权；远端策略允许 fast-forward direct update |
| 排除情况 | 不由 `publish`/`finish` 自动触发；不创建 merge commit；不绕过 branch protection/review/merge queue；不自动回滚已经发生的 default-branch fast-forward |

### UC-AW-001：按项目身份解析 project-first artifact 路由

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-03 项目迁移执行者 |
| 触发条件 | 初始化、恢复、检查或 Git 生命周期需要定位当前项目的 docs/process artifact |
| 输入 | project root、project name、artifact control repo、layout version、worktree registry 与 legacy routing metadata |
| 处理逻辑 | 先验证 project identity，再按显式 layout version 解析当前项目 worktree 内 `<project>/docs` 和 `<project>/process`；未迁移项目可 dual-read `docs/<project>` / `process/<project>`，但写目标不得在无显式配置时静默切换；冲突或多解时 fail closed |
| 输出/结果 | 唯一、可移植、可审计的 project docs/process 路由，或包含冲突候选和修复入口的 BLOCKED 结果 |
| 前置条件 | control repo、project metadata 与当前设备路径可读；真实 link/migration 另行授权 |
| 排除情况 | 不自动移动文件、创建软链接或把 artifact repo root 当作当前项目工作区 |

### UC-AW-002：管理长期常驻的每项目 artifact worktree

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-03 项目迁移执行者 |
| 触发条件 | 为项目准备、检查、列举、修复提示或安全移除独立 artifact working tree |
| 输入 | control repo、project name、可配置 sibling worktree parent、长期 integration branch、短期 CR branch、layout/sparse/owned-path policy 与 dry-run 标志 |
| 处理逻辑 | 验证 common Git dir、目标路径、project identity、branch 占用和禁止嵌套规则；保留现有 control checkout，在 sibling root 创建/登记只暴露当前 project namespace 的长期 worktree；仅当远端 `projects/<project-name>/integration` 不存在时，才从 fresh `origin/main` exact OID 以 create-only 语义初始化，已存在时禁止 recreate/reset/orphan；空闲驻留 integration，CR 激活时切换到短期分支；check/list 返回 branch role、健康和 expected-OID 漂移状态；finish/abort 后回到同一 integration；remove 仅在精确身份、clean 与 ref/恢复条件满足时执行 |
| 输出/结果 | 可被源码项目稳定引用的 project worktree、机器可读 registry/health，或零副作用的失败/修复建议 |
| 前置条件 | CP2 已冻结分支职责与拓扑边界，CP3 冻结 attach/switch/finish/abort 的具体状态机；真实 worktree mutation 另行授权 |
| 排除情况 | 不删除未知目录、不复用 sibling project branch、不在 control checkout 内创建嵌套工作树 |

### UC-AW-003：在当前项目作用域执行源码与 artifact Git 周期

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-02 功能 Agent / Skill 作者 |
| 触发条件 | 当前项目 CR 进入 open、publish、merge、finish 或等价检查阶段 |
| 输入 | project identity、source repo、当前 project artifact worktree、长期 integration branch、短期 CR branch、shared-main OID、expected OID 与操作授权 |
| 处理逻辑 | 将一个逻辑 CR 拆为异构且关联的 source/artifact legs：source leg 从 fresh 源码 `main/master` 创建 CR branch，完成后进入同一源码默认分支；artifact leg 从 fresh 项目 integration 创建短期 CR branch，完成后只进入同一 integration，不 refresh、不 merge、不直接更新 artifact shared `main`；两 leg 共享 CR ID/attempt correlation，分别输出终态；只检查当前项目 dirty/branch/ref/path，sibling dirty 不阻断，任何跨项目路径/ref/index 访问则 fail closed |
| 输出/结果 | 当前项目两条异构 Git leg 的逐 leg 计划、终态与恢复入口；并行项目之间无 checkout、index、branch 或 shared-main 污染 |
| 前置条件 | 当前 project worktree identity 和 owned paths 校验通过；真实 remote mutation仍需操作级授权 |
| 排除情况 | 不读取、stage、commit、merge、clean 或删除 sibling project namespace/ref；不宣称跨仓原子事务 |

### UC-AW-004：聚合异构 source/artifact legs 并隔离 shared main 同步

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者 / Reviewer |
| 触发条件 | 一个逻辑 CR 的任一必需 leg 更新状态，或协调者准备判定整体完成 |
| 输入 | CR ID、attempt correlation、source leg result、artifact leg result、artifact integration expected/current OID、聚合优先级与不授权边界 |
| 处理逻辑 | 单一协调者验证两 leg 归属同一 CR/attempt，并按 `BLOCKED > FAIL > IN_PROGRESS > PASS` 聚合；仅全部必需 leg PASS 才整体完成；`PARTIAL` 只记录进度/影响，失败不自动关闭 CR，也不回滚已成功 leg；artifact expected integration OID 漂移阻断 artifact finish，而 shared main/integration divergence 本身不是单个 CR blocker；shared main↔integration 双向同步只能由 CR 外人工流程执行，默认要求该项目无活跃 artifact CR |
| 输出/结果 | 两 leg 独立结果、aggregate decision、progress/effect、阻断原因、correlation refs 与安全恢复入口 |
| 前置条件 | leg result 可验证且归属一致；artifact finish 使用 fresh integration expected OID；任何真实同步/remote mutation均另行授权 |
| 排除情况 | 不在单个 CR 内 refresh/merge artifact main；不声明跨仓原子性；不自动 rollback、merge、rebase、force、解冲突或关闭失败 CR |

### UC-AW-005：生成逐项目迁移前检查与交接包

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-04 审批者 / Reviewer |
| 触发条件 | CR-051 能力完成后，用户准备逐个项目搬迁 artifact 并重挂软链接 |
| 输入 | legacy/new 路径映射、project worktree health、文件清单/hash、link target、branch/ref 状态与迁移授权状态 |
| 处理逻辑 | 只读检查源/目标冲突、project ownership、worktree/branch readiness、软链接计划、备份/回滚入口和验证命令；生成 per-project manifest/checklist；未获得独立迁移授权时 mutation plan 保持 blocked/dry-run |
| 输出/结果 | 可由用户逐项目执行的迁移交接包和可复跑验收清单；本 CR 结束时真实迁移数为 0 |
| 前置条件 | 路由/worktree/Git 周期能力已验证；每个项目迁移单独启动并确认 |
| 排除情况 | 不批量搬迁所有项目、不自动重挂软链接、不自动提交或发布迁移结果 |

### UC-MR-001：解析 external route 并诚实判定布局模式

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-03 项目迁移执行者、P-07 迁移准备协调者 |
| 触发条件 | workspace link/check、迁移 preflight 或恢复流程需要解析位于源码项目外部 sibling worktree 的 docs/process route |
| 输入 | project root、project identity、schema version、external anchor、layout metadata、legacy symlink/STATE 线索 |
| 处理逻辑 | 对 v1/v2/legacy 执行版本化 dual-read；新写入只产生 schema v2；external anchor 只允许声明的可移植锚点和受 containment 校验的相对路径；health 依据事实区分 `legacy-compatible-not-migrated`、`project-first-migrated`、`route-conflict`，不得用兼容可读性推断已迁移 |
| 输出/结果 | 唯一 route、布局模式、稳定退出码和冲突修复入口；非法 traversal、绝对路径或多解 fail closed |
| 前置条件 | route metadata 与 project identity 可读；真实布局切换未授权时只做 read/check/dry-run |
| 排除情况 | 不全局放宽 `..`，不接受设备绝对路径作为 canonical route，不自动移动文件或重挂链接 |

### UC-MR-002：持久化 leg / aggregate evidence 并安全写入 audit tail

| 字段 | 内容 |
|---|---|
| 使用角色 | P-05 工作流审计者、P-06 证据生产者、P-07 迁移准备协调者 |
| 触发条件 | source/artifact leg 产生计划或终态，aggregate 需要跨进程重算、投影或在工作树内留下审计 receipt |
| 输入 | CR/attempt correlation、leg result、aggregate result、content digest、parent/target OID、opaque evidence handle |
| 处理逻辑 | 先把执行事实写入 out-of-band content-addressed durable store，再由独立 reader readback；aggregate 只消费可验证 handle；需要 project-local 审计时追加带 parent/target OID 的 audit-tail receipt，并区分执行完成事实与 receipt 写入造成的工作树变化 |
| 输出/结果 | 可跨进程重读、可校验归属与完整性的 leg/aggregate evidence、state projection 和 audit-tail receipt，或明确 BLOCKED/FAILED 与恢复入口 |
| 前置条件 | evidence backend、digest/ownership contract 和单写边界已定义；跨机器 durability 在 CP3 冻结 |
| 排除情况 | 不用进程内 memory 冒充 durable store，不让 audit-tail 自引用改变已完成 target OID，不以 Markdown 摘要替代机器 evidence |

### UC-MR-003：通过 transitional bootstrap 协调首次迁移并可续跑

| 字段 | 内容 |
|---|---|
| 使用角色 | P-01 Host Orchestrator 维护者、P-04 审批者、P-07 迁移准备协调者 |
| 触发条件 | 目标项目尚未 project-first migrated，但 route/ref/worktree/seed normalization 必须先发生才能进入 native CP0 |
| 输入 | immutable migration plan、pre-migration intent、fresh OID、typed authorization、已存在 receipt/third-state |
| 处理逻辑 | 在 mutation 前持久化 intent 和 plan digest；按受控步骤执行 bootstrap；每步写 immutable receipt；失败时进入可枚举 third-state，resume 重新观察 OID/route/receipt 后只执行缺失步骤；完成后由 CP0 import receipt，而不是伪造 native-first 历史 |
| 输出/结果 | 可审计 bootstrap receipt、当前 third-state、已执行/未执行步骤和幂等 resume/abort 路由 |
| 前置条件 | coordinator 已由 CR-052 在临时真实拓扑验证；真实 CR-053 另有明确授权 |
| 排除情况 | 不在半成品状态提交伪 CP0 PASS，不把 transitional bootstrap 宣称为 first-write native，不自动跨 leg 回滚 |

### UC-MR-004：先规范 seed，再激活 steady ownership

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-04 审批者、P-07 迁移准备协调者 |
| 触发条件 | 新 project integration 从 shared main 继承 sibling seed，而目标 project worktree 只能长期拥有自身 `docs`/`process` leaf |
| 输入 | frozen manifest、seed path/hash/count、project ownership、fresh source/target OID、prune authorization |
| 处理逻辑 | normalize 阶段只删除 manifest 精确列出的 inherited sibling seed，并验证 shared-main/sibling ref/OID/hash 不变；unexpected add/modify/delete 立即阻断；prune receipt 完成后才激活 steady `docs`/`process` ownership |
| 输出/结果 | manifest-bound prune result、unexpected=0 证明、steady ownership activation receipt 和失败恢复入口 |
| 前置条件 | manifest 已在 mutation 前冻结；prune 与 route activation 使用独立 typed authorization |
| 排除情况 | 不扫描 sibling 真实内容生成 manifest，不扩大 deletion allowlist，不在 prune 前启用会导致 split-brain 的 steady route |

### UC-MR-005：以 typed authorization 执行通用 migration plan

| 字段 | 内容 |
|---|---|
| 使用角色 | P-07 迁移准备协调者、P-04 审批者 |
| 触发条件 | migration engine 准备执行 ref、worktree、prune、link 或 push 类动作，或操作者先请求 dry-run |
| 输入 | immutable plan、action type、repo/ref/path、fresh expected OID、plan digest、expiry、single-use authorization |
| 处理逻辑 | runner 默认只生成确定计划；execute 默认禁用；每个动作分别校验授权对象、OID、digest、有效期和消费状态；执行后逐步 post-check 并持久化 terminal evidence；CLI 只做参数解析和呈现，不复制迁移状态机 |
| 输出/结果 | 零副作用 dry-run，或带逐动作 before/after、terminal status、evidence handle 和 resume route 的执行结果 |
| 前置条件 | CP3/CP5 已冻结动作状态机和授权 schema；真实仓操作还需目标级授权 |
| 排除情况 | 不使用总授权覆盖多个高风险动作，不执行 force/tag/reset/rebase/orphan/自动 merge，不绕过 branch protection |

### UC-MR-006：在三并列临时拓扑真实演练并恢复故障

| 字段 | 内容 |
|---|---|
| 使用角色 | P-07 迁移准备协调者、P-05 工作流审计者、P-04 审批者 |
| 触发条件 | CR-052 需要证明 migration-ready，而 dry-run 或 mock 无法覆盖真实 worktree/ref/link/filesystem mutation 与进程重启 |
| 输入 | 临时 source repo、artifact control repo、project worktree sibling root、local bare remote、故障注入矩阵 |
| 处理逻辑 | 在临时三并列目录执行 schema/health、bootstrap、seed prune、worktree/ref/link、leg/aggregate evidence 和 scoped check 的真实 mutation；在 intent、mutation、evidence-tail、partial push/readback 等边界注入失败，并从新进程 resume；全过程验证 fixture 外路径/ref 不变 |
| 输出/结果 | 每条正向/负向/边界/权限/失败恢复路径的 terminal evidence、清理/恢复证明和 migration-ready 判定 |
| 前置条件 | 临时目录与 local bare remote 可安全销毁；真实 meta-flow/artifact 仓只允许只读快照或 dry-run |
| 排除情况 | 不对真实 meta-flow-artifacts 创建 integration/worktree/ref/link，不 push 真实 remote，不读取 sibling 项目内容 |

### UC-MR-007：生成 scoped readiness 结论并交接 CR-053

| 字段 | 内容 |
|---|---|
| 使用角色 | P-04 审批者、P-05 工作流审计者、P-07 迁移准备协调者 |
| 触发条件 | CR-052 的能力、临时真实 E2E、guardrail、文档与 workflow eval 已完成，准备判定是否允许启动真实迁移 CR |
| 输入 | CR-052 scoped check、冻结的全局 error fingerprint、能力版本、E2E/evidence refs、remaining risks、不授权项 |
| 处理逻辑 | 只要求 CR-052 scoped contract PASS，并验证全局新增 fingerprint=0；保留历史 catalog 错误为非阻断已知事实；汇总 CR-053 必须消费的能力版本、receipt、denylist、授权/回滚前置和 follow-up，不在 handoff 中现场新增核心执行逻辑 |
| 输出/结果 | `READY / READY_WITH_RISK / NOT_READY` readiness 包、CR-053 启动前置清单和明确不授权项 |
| 前置条件 | 所有 CR-052 Story 通过独立 CP6/CP7，CP8 尚需用户人工终验 |
| 排除情况 | 不修复全部 CR-001..033 历史错误，不把 readiness 结论当作真实迁移、publication 或 shared-main 写入授权 |

### UC-VNEXT-001：为每个项目建立独立发布库与过程库

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者、P-01 Host Orchestrator 维护者 |
| 触发条件 | 新项目接入 vNext，或既有项目准备从共享过程仓切换 |
| 输入 | project identity、现有源码/交付仓、候选独立过程仓、路由元数据 |
| 处理逻辑 | 发布库优先复用现有源码/交付仓；为该项目绑定恰好一个独立过程库；验证两个路由唯一且不共享 working tree、index、branch 或过程写入面 |
| 输出/结果 | 每项目恰好 1 个发布库路由和 1 个过程库路由；切换其他项目时本项目状态与文件不变 |
| 前置条件 | project identity 唯一；两个仓库的 ownership 和路由可只读检查 |
| 排除情况 | 不新增第三个 artifact 仓；不让多个项目共享过程库或工作树；不在 CP2 执行真实仓创建/push |

### UC-VNEXT-002：用 Project/Roadmap/Phase/Work 治理长期项目

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者、P-04 审批者 / Reviewer |
| 触发条件 | 项目需要表达长期方向、阶段目标或启动一项可交付工作 |
| 输入 | Project identity、Roadmap outcome、Phase objective、Work goal/owner/risk profile |
| 处理逻辑 | 严格按 Project→Roadmap→Phase→Work 四层建立唯一父子关系；Roadmap/Phase 保存长期真相，Work 保存单次执行上下文并在关闭时只回写最小进度/结果引用 |
| 输出/结果 | 4/4 层可追踪；Work 不把全文复制进长期对象，审批者可从长期视图定位当前和已完成 Work |
| 前置条件 | Project 已绑定独立过程库；Roadmap/Phase identity 可唯一解析 |
| 排除情况 | 不新增第五层并行治理对象；不把 Roadmap/Phase 降格为每次 Work 的临时副本 |

### UC-VNEXT-003：按受控生命周期启动、阻塞、完成和归档 Work

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者、P-01 Host Orchestrator 维护者 |
| 触发条件 | 用户创建新 Work、恢复中断 Work、声明完成/取消或归档证据 |
| 输入 | Work goal、父 Phase、owner、scope、risk profile、expected process-main OID |
| 处理逻辑 | Work 使用 `planned→active→completed|cancelled→archived` 主路径和 `active↔blocked` 恢复路径；非法跳转 fail closed；每次写入由单一 publisher 以 expected-OID CAS 提交并留下 receipt |
| 输出/结果 | Work 状态唯一、恢复入口明确；stale OID 不覆盖他人结果；关闭时仅产生必要长期投影 |
| 前置条件 | Work scope 完整且过程库 main 当前 OID 可读取 |
| 排除情况 | 不为每个 Work 自动创建过程分支；不在冲突时自动 merge 或 force |

### UC-VNEXT-004：按 G0/G1/G2 为 Work 选择足够的流程

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者、P-04 审批者 / Reviewer |
| 触发条件 | Work 创建、范围扩大、风险升高或预算即将超限 |
| 输入 | 变更范围、权限/外部写入/迁移/生产风险、读写文件数、检查组数、token 预算 |
| 处理逻辑 | 低风险且落在默认上限内路由 G0；跨文件/多阶段但无高风险操作路由 G1；权限、安全、外部写入、迁移、production/publication 或超过 G1 默认上限路由 G2；风险上升只允许升级，不静默降级 |
| 输出/结果 | 每个 Work 恰好 1 个 risk profile，并得到与风险匹配的 gate、检查和审批要求 |
| 前置条件 | 风险判定输入和推荐默认上限可读；待用户决策项已分类 |
| 排除情况 | 不用 G0 绕过高风险人工门；不把 G2 当成无限上下文或无限检查许可 |

### UC-VNEXT-005：用 Work scope 与 token budget 限制读写检查

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者、P-02 功能 Agent / Skill 作者、P-05 工作流审计者 |
| 触发条件 | Agent/CLI 准备读取文件、修改文件、执行检查或扩展上下文 |
| 输入 | Work `allowed_reads`、`allowed_writes`、`required_checks`、risk profile、token budget 和当前 usage |
| 处理逻辑 | 默认 deny 未声明读取/写入/检查；G0 使用 8读/8写/3检查/32k token 上限，G1 使用 20读/24写/8检查/96k token 上限；超限前停止并记录原因，按规则扩 scope 或升级 profile |
| 输出/结果 | 实际访问与声明 scope 一致；token measured/proxy/unavailable 诚实记录；无静默全文扩读或无关全量回归 |
| 前置条件 | Work scope 在执行前持久化；平台 usage 可用性被明确表达 |
| 排除情况 | 不把估算 token 冒充 measured；不以“可能有用”为由读取全仓或运行所有检查 |

### UC-VNEXT-006：以当前快照迁移并保留旧共享仓只读历史

| 字段 | 内容 |
|---|---|
| 使用角色 | P-03 项目迁移执行者、P-04 审批者、P-08 项目负责人 / Work 发起者 |
| 触发条件 | 目标项目通过 vNext preflight，准备初始化独立过程库并进入试点 |
| 输入 | 当前 Project/Roadmap/Phase/active Work 快照、旧共享仓只读来源、目标过程仓、expected OID、回滚计划 |
| 处理逻辑 | 只复制当前有效快照和来源索引；不拆 Git 历史、不无损转换旧 CP/CR/Story；cutover 后新写入只进入目标过程库，旧共享仓保持只读；先完成 2 个项目×每项目 2 个周期试点 |
| 输出/结果 | 新过程库可从快照恢复；旧历史可按引用查询但新增写入为 0；试点至少 4 个 Work 周期且每项目覆盖 G0/G1 |
| 前置条件 | 目标项目有独立迁移授权、快照清单、备份和回滚入口；CR-052 当前仅定义产品契约 |
| 排除情况 | 不批量迁移；不重写旧历史；不自动 publish；不把只读兼容解释为双写 |

> **历史状态**：`UC-VNEXT-001..006` 是 R2 的系统能力分组，已由用户指出“不是用户走的真实旅程”，现已被下列 `UC-VNEXT-UJ-001..016` supersede。历史正文保留用于追溯，不再作为当前设计输入。

## CR-052 vNext R3 用户旅程场景（当前确认基线）

> 本节是当前唯一 active 的 vNext 用户场景。用户已确认 `UC-VNEXT-UJ-001..016` 并授权本地、可逆实现；真实迁移/软链接切换、commit/push/tag、远端/production/凭据和破坏性 Git 仍需独立授权。每个未来项目仍可按自身情况选择“适用 / 修改后适用 / N/A”，但不得恢复共享 working tree 或合并四类授权语义。

### UC-VNEXT-UJ-001：初始化新项目的发布库与过程库

**用户故事**：作为项目负责人，我要在开启项目时快速建立彼此独立的发布库和过程库，以便项目从第 0 步就与其他项目物理隔离，不因任何项目切分支而看到错误版本的过程文档。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；涉及远端创建或高风险路径覆盖时由 P-04 审批者确认 |
| 触发条件 | 用户说“新建项目 X”，或在空目录中启动 Meta Flow |
| 输入 | 项目名、项目根目录；可选的现有源码仓、发布仓/过程仓名称与远端地址 |
| 处理逻辑 | 系统先问发布库是否复用现有源码仓（默认复用；不存在则创建），再确认过程库名称（默认 `<project>-process`）；只读校验路径、仓库身份和已有内容，确保两仓不共享 working tree、index 或 branch；生成相对软链接 `<release-repo>/process -> <process-repo>`；只初始化最小 `PROJECT.yaml`，不生成八份产品基线 |
| 输出/结果 | 用户得到两仓路径、Git 身份、相对软链接目标、`project_id` 和初始化结果；切换其他项目后本项目文件/ref/hash 变化数为 0 |
| 前置条件 | 目标路径可解析；已有非空目录或远端写入必须先获得用户确认；同名项目 ID 不得歧义 |
| 排除情况 | 不创建共享过程工作树；不默认创建第三个归档仓；不生成 Roadmap、需求矩阵、Story Map 或全套检查点 |
| 典型交互 | 用户：“新建项目 X。”系统：“检测到源码仓，建议复用为发布库；过程库使用 `X-process`，可以吗？”用户确认后，系统展示 dry-run，再执行本地初始化并报告结果 |
| 解决痛点 / 用户价值 | 从一开始消除跨项目分支联动；初始化足够小，用户无需先理解重型流程 |
| 适用性选择 | `适用 / 修改仓库或目录默认值后适用 / N/A（仅管理既有项目）` |
| 风险与资源边界 | 默认 G1；只读 preflight 不超过 G0 上限。已有目录写入、远端仓创建、首次 push、覆盖软链接或权限变更升级 G2；所有读取/写入计入本次初始化预算 |

### UC-VNEXT-UJ-002：把已有项目安全接入双仓治理

**用户故事**：作为已有项目的负责人，我要把当前项目的有效过程快照接入独立过程库，同时保留旧仓可审计，以便获得项目隔离而不重写历史、不丢失正在进行的工作。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者、P-04 审批者 / Reviewer |
| 触发条件 | 既有项目仍使用本地过程目录或共享过程仓，用户明确提出“接入新治理”或“迁移当前项目” |
| 输入 | 现有发布仓、旧过程来源、当前有效 Project/Phase/Work 状态、目标过程仓、快照清单、回滚点和目标 OID |
| 处理逻辑 | 系统先只读盘点并生成快照迁移计划；请用户确认迁移对象、排除项和回滚；只复制当前有效状态与旧来源索引，不拆 Git 历史、不把旧 CP/CR/Story强行转换；切换后只写新过程仓，旧仓保持只读 |
| 输出/结果 | 新过程仓可从快照继续工作；旧历史仍可按引用审计；跨项目共享写入数和双写数均为 0 |
| 前置条件 | 用户明确选择迁移时点；旧仓和目标仓可校验；存在备份、expected OID 和原子恢复入口 |
| 排除情况 | 不批量迁移其他项目；不重写旧历史；不删除旧仓；不把兼容读取解释成继续双写 |
| 典型交互 | 用户：“把项目 Y 接入新流程。”系统先报告“将迁移 1 个 Project、1 个当前 Phase、2 个 active/blocked Work，旧仓只读保留”，用户确认后才允许执行 |
| 解决痛点 / 用户价值 | 既有项目也能获得隔离，但迁移爆炸半径限定在单项目、单快照，可随时回查旧事实 |
| 适用性选择 | `适用 / 修改快照范围后适用 / N/A（全新项目）` |
| 风险与资源边界 | 强制 G2；盘点可在批准前只读执行，任何文件复制、软链接切换、commit 或远端写入均需单独授权和独立预算 |

### UC-VNEXT-UJ-003：建立并维护轻量长期治理基线

**用户故事**：作为项目负责人，我要用最少的信息记录项目长期目标和当前阶段，以便大型项目持续知道“往哪里走、现在做到哪”，小项目又不必背负无用层级。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；重大 Roadmap 改向时由 P-04 审批者确认 |
| 触发条件 | 项目初始化完成、开始首个阶段、阶段结束/暂停，或长期方向发生变化 |
| 输入 | 一句话 Project 目标；可选的一句话 Roadmap outcome；当前 Phase objective；项目规模与是否需要跳过 Roadmap/Phase |
| 处理逻辑 | 系统一次只问一个长期治理问题；大型/长期项目建立 `PROJECT.yaml`、`ROADMAP.yaml` 与 `phases/<id>/PHASE.yaml`，每个对象只保留目标、状态和引用；小项目可由用户选择 `Project -> Work`，不强行补空 Roadmap/Phase；阶段转换或项目暂停时只更新状态和必要引用 |
| 输出/结果 | 用户可快速看到项目目标、当前阶段、当前/下一项 Work；长期对象不复制 Work 全文 |
| 前置条件 | 项目已绑定独立过程仓；层级选择和当前阶段由用户确认 |
| 排除情况 | 不强制所有项目四层齐全；不一开始生成 Story Map/MVP/八份基线；不让单个 Work 重写整个 Roadmap |
| 典型交互 | 系统：“这个项目长期要达到什么结果？”用户回答后，系统再问：“第一阶段先完成什么？这是长期项目，需要 Roadmap 层吗？”用户可选择保留或跳过 |
| 解决痛点 / 用户价值 | 长期方向和阶段不丢失，同时层级可按项目复杂度伸缩，不把治理模板当成果 |
| 适用性选择 | `完整 Project/Roadmap/Phase / 轻量 Project/Phase / 最小 Project/Work / N/A` |
| 风险与资源边界 | 建立/小改基线通常 G0；跨阶段重排、目标废弃或影响多个活跃 Work 为 G1；重大方向改变或风险接受进入正式 CR/G2。读取与写入只限长期对象和直接引用 |

### UC-VNEXT-UJ-004：通过问询和头脑风暴确认单个 Work 需求

**用户故事**：作为项目负责人，我要在系统动手前通过简短问询和头脑风暴把当前要解决的问题聊清楚，以便避免理解偏差和返工，又不为一个 Work 生成大量模板文档。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；需要风险接受时加入 P-04 审批者 |
| 触发条件 | 用户说“我想做 X”“修一下 X”或提供一段尚未确认的需求 |
| 输入 | 用户自然语言目标、痛点、目标用户/使用时机、已知约束；必要时当前 Project/Phase 的短引用 |
| 处理逻辑 | 系统不得直接实现；一次只问一个高价值问题，先确认痛点、用户和使用时机，再提出 2-3 个真实候选场景供用户选择/补充；复述“你的意思是……影响面是……”并等待确认；只把确认结果写成几段话的最小 `REQUEST.md` 和确认记录 |
| 输出/结果 | 一份用户确认过的最小 Work 请求：目标、适用场景、不做什么、初步影响和待分类风险；不产生八份产品基线 |
| 前置条件 | 用户可参与至少一次确认；已有明确需求也至少需要一次复述确认，不能静默推断 |
| 排除情况 | 不单方面批量生成 UC/REQ/TC/Story；不一次抛出长问卷；用户纠正后不继续强推预设选项 |
| 典型交互 | 用户：“我想优化归档。”系统：“你最想解决的是跨项目版本错乱、操作太多，还是 token 太高？”用户选择后，系统提出 2-3 个使用情景并复述，直到用户确认 |
| 解决痛点 / 用户价值 | token 用在减少误解的对话上，而不是用在未经确认的模板填充上 |
| 适用性选择 | `适用 / 修改提问深度后适用 / N/A（仅当请求已由用户显式确认且仍完成一次复述）` |
| 风险与资源边界 | 默认按 G0 discovery 预算：最多读取 8 个当前 Work 相关输入、只写 1 份 REQUEST/确认记录、最多 1 组一致性检查；计入 Work 总 token 预算，不读取全项目历史。需要扩读时先说明原因并重新分类 |

### UC-VNEXT-UJ-005：判定日常 Work 还是正式 CR

**用户故事**：作为项目负责人，我要让系统说明这项变化应当作为普通 Work 还是正式 CR，以便日常开发不背 CR 全流程，真正重大的变化又留下足够审计和决策记录。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者、P-04 审批者 / Reviewer |
| 触发条件 | UC-VNEXT-UJ-004 已确认请求，或执行中发现目标/架构/风险发生实质变化 |
| 输入 | 已确认 REQUEST、影响的 Phase/模块、架构和兼容影响、安全/迁移/发布/风险接受需求 |
| 处理逻辑 | 系统按可解释准则判断：局部日常交付进入 Work；跨阶段重构、公共架构/契约变化、强审计、不可逆迁移、安全权限或风险接受进入正式 CR；展示判定理由和反例；用户可要求升级，降级必须重新证明不存在重大触发项 |
| 输出/结果 | 唯一治理容器 `Work` 或 `CR`、判定理由、触发的审批门和可申诉入口 |
| 前置条件 | 请求已经过用户复述确认；关键风险信号可回答为是/否/未知 |
| 排除情况 | 不把所有开发默认包装成 CR；不因文件数量多就自动判 CR；未知高风险不得静默按 Work 放行 |
| 典型交互 | 系统：“这是 3 个文件内的兼容修复，不改变公共契约，建议普通 Work；如果要改变公共 API，则升级正式 CR。接受吗？” |
| 解决痛点 / 用户价值 | CR 回归“重大变化治理机制”，普通工作走短路径，用户仍知道为什么被升级 |
| 适用性选择 | `接受 Work / 升级为 CR / 修改影响判断 / 暂停` |
| 风险与资源边界 | 分类本身只读取 REQUEST、直接影响摘要和必要政策引用，按 G0 控制；一旦命中重大触发项，后续治理直接进入 G2，不能靠扩大 G1 预算规避 |

### UC-VNEXT-UJ-006：自动判定 G0/G1/G2 和资源预算

**用户故事**：作为项目负责人，我要系统根据当前 Work/CR 的真实风险自动推荐流程档位和成本上限，以便简单事情走轻流程，复杂事情得到足够控制，而且我能看懂和申诉判定。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；G2 与降级请求由 P-04 审批者确认 |
| 触发条件 | Work/CR 创建、范围扩大、预算将超限，或执行中出现新的风险信号 |
| 输入 | 文件/模块范围、公共契约影响、权限/安全/数据/迁移/生产风险、验证要求、预计 reads/writes/checks/token |
| 处理逻辑 | 单文件文档/配置等可逆低风险改动推荐 G0；跨文件、多步骤但无高风险推荐 G1；权限、安全、不可逆迁移、生产写入、正式发布/部署、强审计或正式 CR 推荐 G2。系统显示命中规则、候选预算和升级原因；用户可升级或申诉，但不能静默降级；超限前必须暂停并申请新档位/预算 |
| 输出/结果 | `G0/G1/G2`、理由、允许读取/写入/检查范围、token 上限、必要门禁和重新分类条件 |
| 前置条件 | REQUEST 和影响摘要可读；预算度量不可用时必须显式标记 `telemetry-unavailable` |
| 排除情况 | 不把 G2 当无限预算；不因常规 Git push 自动把所有 Work 升 G2；不允许执行后补做风险分类 |
| 典型交互 | 系统：“当前只改 1 个文档且可回退，建议 G0；上限为当前 G0 候选预算。若改到权限规则或生产发布，将立即升级 G2。” |
| 解决痛点 / 用户价值 | 用户不用手工估流程，却保留最终知情权；流程成本与风险成比例 |
| 适用性选择 | `接受推荐档 / 主动升级 / 提交降级申诉 / 修改范围后重判` |
| 风险与资源边界 | Work 总上限：G0 `8 reads / 8 writes / 3 check groups / 32k token`，G1 `20 / 24 / 8 / 96k`；G2 单独批准。该数值已随 R3.2.1 场景确认成为当前 vNext 规则 |

### UC-VNEXT-UJ-007：执行、暂停、恢复和交接当前 Work

**用户故事**：作为项目负责人，我要随时知道当前 Work 做到了哪里，并能在会话中断、换人或换设备后从最小上下文继续，以便不反复读取全部历史，也不靠聊天记忆恢复工作。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者 |
| 触发条件 | Work 开始执行、用户要求暂停/继续、会话即将结束、执行者变化或设备切换 |
| 输入 | 当前 Work 的 REQUEST、scope、risk profile、当前 OID、已完成步骤、阻塞项、下一步和最小证据引用 |
| 处理逻辑 | 系统只装载当前 Work 允许的最小上下文；执行时持续检查 scope 和预算；暂停/交接时写一份短状态，包含已完成、未完成、阻塞、下一步和 OID；恢复时先核对 OID/工作区，再只读必要引用，不重新扫描全项目或复制完整会话 |
| 输出/结果 | 可继续执行的最小 Work 状态；用户能一眼看到进度、风险和下一动作；重复全文读取数为 0 |
| 前置条件 | Work 已完成 UJ-004..006；允许读写清单和恢复锚点可用 |
| 排除情况 | 不把完整 transcript 当交接；不默认读取全部 HLD/历史 CR/所有测试报告；OID 或 scope 冲突时不继续写入 |
| 典型交互 | 用户：“先停一下，明天继续。”系统只记录“已完成 A，B 被 C 阻塞；下一步 D；代码 OID=x、过程 OID=y”，下次先核对两 OID 再继续 |
| 解决痛点 / 用户价值 | 中断和交接不再触发昂贵的全量上下文恢复，长期项目也能稳定续跑 |
| 适用性选择 | `适用 / 修改交接字段后适用 / N/A（单轮即可完成的极小 Work）` |
| 风险与资源边界 | 完全继承当前 Work 的 G0/G1/G2 总预算；恢复预检计入原预算，不自动获得新预算。需要扩读时先说明文件和原因，超限则升级或拆 Work |

### UC-VNEXT-UJ-008：按风险做足够而不过量的开发评审

**用户故事**：作为项目负责人，我要当前改动得到与风险相称的设计和代码评审，以便小改动快速完成，复杂改动又不会因为缺少设计而返工。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者、P-04 审批者 / Reviewer |
| 触发条件 | Work 准备实现、影响面扩大、实现完成准备验证，或 G2 到达设计门 |
| 输入 | 当前 Work REQUEST、影响文件/接口、diff 或轻量设计说明、risk profile 和待决策项 |
| 处理逻辑 | G0 默认跳过独立设计评审，直接按范围实现并自检；G1 只在涉及接口/多模块/复杂失败路径时进行一次当前 Work 范围的轻量设计评审；G2 强制 HLD/ADR、人工设计门和高强度架构评审。Reviewer 默认只读当前 scope，发现范围外风险才提出明确扩读请求 |
| 输出/结果 | `通过 / 修改后通过 / 阻断 / 升级档位` 的评审结论、少量具体问题和可执行下一步 |
| 前置条件 | Work 风险档和 scope 已确定；待评审对象存在且没有未披露的范围外修改 |
| 排除情况 | 不为 G0 自动生成 HLD/ADR；不为 G1 扫描全项目历史；不以形式性文档数量代替评审质量 |
| 典型交互 | 系统：“这是 G1 且改变两个内部模块接口，建议一次轻量设计评审，只看 REQUEST、接口说明和 6 个相关文件；是否开始？” |
| 解决痛点 / 用户价值 | 评审真正用于降低当前风险，而不是成为固定的 token 消耗仪式 |
| 适用性选择 | `按推荐评审 / 主动升级 / 修改评审范围 / G0 跳过` |
| 风险与资源边界 | G0 独立设计评审数为 0；G1 默认最多 1 次且计入 G1 read/check/token 总预算；G2 的 HLD、ADR、独立评审预算必须单独批准 |

### UC-VNEXT-UJ-009：按风险验证当前成果

**用户故事**：作为项目负责人，我要系统只运行足以证明当前 Work 正确的验证，以便简单改动快速得到信心，复杂改动获得充分证据，又不在每次变更都跑全量检查。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；G2 独立验收由 P-04 审批者 / Reviewer 复核 |
| 触发条件 | 实现完成、缺陷修复、评审回修完成，或准备提交/推送 |
| 输入 | risk profile、当前 Work 验收目标、changed paths、项目最小测试入口和必要的构建/检查命令 |
| 处理逻辑 | G0 执行针对性测试或项目配置的最小测试入口，并检查 `git status`/diff；G1 增加必要构建和当前 Work 定向检查；G2 执行经批准的全量回归、独立 QA 和发布/迁移验证。系统先展示计划，失败时只报告与当前 Work 相关的证据和恢复入口 |
| 输出/结果 | 可复核的 PASS/FAIL/BLOCKED、实际执行命令/范围、失败点和是否允许进入提交/推送 |
| 前置条件 | 验收目标和检查范围在执行前确定；测试环境不会隐式写生产数据 |
| 排除情况 | 不因“更保险”默认跑所有检查；不把未执行项报告为 PASS；不在 G0/G1 未授权地访问 runtime/production |
| 典型交互 | 系统：“这是 G0 文档生成修复，建议运行 1 个目标测试和 `git status`；不跑全仓回归。确认执行吗？” |
| 解决痛点 / 用户价值 | 验证时间和 token 与真实风险成比例，同时保留清晰的通过证据 |
| 适用性选择 | `接受计划 / 增加检查 / 缩小范围后重判 / 暂停` |
| 风险与资源边界 | 检查组计入 Work 总上限：G0 最多 3 组、G1 最多 8 组、G2 单独批准；任何新增检查需说明它覆盖的具体风险，不得把一次命令拆成多组规避预算 |

### UC-VNEXT-UJ-010：独立、安全地提交和推送两仓成果

**用户故事**：作为项目负责人，我要在验证通过后分别、安全地提交和推送发布库与过程库，以便成果可共享和恢复，但不承受双 leg、aggregate 或伪原子配对的复杂度。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；受保护分支、生产发布或新远端写入由 P-04 审批者确认 |
| 触发条件 | 当前 Work 验证通过，用户明确要求 commit/push，或批准的自动发布策略到达触发点 |
| 输入 | 两仓目标 remote/ref、当前 HEAD、expected remote OID、dirty/index 状态、待提交路径和验证结果 |
| 处理逻辑 | 系统分别展示发布库和过程库计划；每仓独立检查 changed-path allowlist、clean/index、exact OID 和 fast-forward 条件；只提交当前 Work 声明路径；按用户确认的顺序独立 push，不做双 leg 配对或 aggregate；一侧失败时保留另一侧真实结果并交给 UJ-012 恢复 |
| 输出/结果 | 每仓 `not-requested / committed / pushed / failed / blocked` 状态、commit/ref/OID 和下一步；部分成功不得报告整体 PASS |
| 前置条件 | UJ-009 已通过；目标仓/ref 已明确；用户或预授权策略明确允许 commit/push |
| 排除情况 | 不隐式 `git add -A`；不自动 merge、force、tag、rebase 或删除分支；不宣称跨仓原子性；不自动回滚已成功一侧 |
| 典型交互 | 系统：“发布库将 push `feature/x`，过程库将 push `main`；二者独立执行。若第二步失败，第一步不会自动回滚。继续吗？” |
| 解决痛点 / 用户价值 | 推送行为简单、透明、可恢复；用户不会因配对编排而面对更多隐藏状态 |
| 适用性选择 | `只提交不推送 / 推送发布库 / 推送过程库 / 两仓依次推送 / 暂停` |
| 风险与资源边界 | 已预授权仓/ref 的常规成果 push 继承原 Work 的 G0/G1，不自动升级 G2；新远端、受保护主分支、生产发布/部署、tag、权限变化或不可逆外部发布强制 G2。Git 状态/OID 检查计入当前 Work 检查预算 |

### UC-VNEXT-UJ-011：关闭 Work 并最小回写 Phase/Roadmap

**用户故事**：作为项目负责人，我要在 Work 完成后用一份短摘要关闭它，并只更新必要的阶段/路线图进度，以便长期治理保持最新，但不会被每次执行的细节淹没。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；阶段完成或 Roadmap 方向变化时由 P-04 审批者确认 |
| 触发条件 | Work 已验证并完成、取消，或确认无法继续；相关提交/推送状态已知 |
| 输入 | Work 目标、最终状态、结果摘要、关键证据/commit refs、remaining risks、所属 Phase/Roadmap |
| 处理逻辑 | 系统展示“目标是否达成、留下什么、还欠什么”；用户确认后关闭 Work；只把状态、结果引用和必要进度回写所属 Phase，必要时更新一个 Roadmap 投影；不复制 REQUEST、完整 diff、测试日志或会话历史。阶段完成/暂停/切换时再单独确认长期对象变化 |
| 输出/结果 | 已关闭/取消的 Work、可审计结果引用、更新后的当前 Phase 和可选 Roadmap 进度；遗留项进入下一 Work/Backlog，而不是无限延长当前 Work |
| 前置条件 | 验证和推送状态已明确；未解决风险已由用户接受、转交或阻断关闭 |
| 排除情况 | 不因关闭 Work 生成全套发布文档；不把未完成写成完成；不自动改变其他 Phase/Roadmap；不复制完整执行证据进长期对象 |
| 典型交互 | 系统：“目标已完成并验证；建议把 PH-001 进度从 40% 更新为 60%，Roadmap outcome 不变。另有问题 Z 建议开新 Work。确认关闭吗？” |
| 解决痛点 / 用户价值 | 长期项目保持方向和阶段可见，日常 Work 又能快速结束，不形成过程文档雪球 |
| 适用性选择 | `关闭并回写 / 关闭但不改 Roadmap / 转为 blocked / 拆出后续 Work / 取消` |
| 风险与资源边界 | 普通关闭为 G0，并计入当前 Work 剩余预算；跨阶段重排为 G1；改变长期目标、接受重大剩余风险或关闭正式 CR 为 G2/人工门。默认最多写 Work、一个 Phase 和一个 Roadmap 投影 |

### UC-VNEXT-UJ-012：从检查失败、OID 漂移、半推送或中断中恢复

**用户故事**：作为项目负责人，我要在执行或推送失败时准确知道已经发生了什么、什么没有发生，以及有哪些安全恢复选项，以便不靠猜测重跑，也不因自动回滚或自动合并造成二次损害。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；涉及覆盖、回滚、重新发布或风险接受时由 P-04 审批者确认 |
| 触发条件 | 检查失败、预算超限、工作区 dirty、expected OID/CAS 漂移、会话中断、commit 后 push 失败，或两仓仅一侧成功 |
| 输入 | 当前 Work/恢复摘要、两仓 before/after OID、已执行步骤、terminal status、失败原因、dirty/index 和远端观测 |
| 处理逻辑 | 系统先重新只读观测，不盲目重放；按步骤列出 `已完成 / 未执行 / 失败 / 状态未知`；给出互斥选项：修复后重试失败步骤、接受部分结果并开后续 Work、人工恢复、或停止并保持现场；CAS 漂移时重新规划，不自动 merge；已成功 push 不自动回滚 |
| 输出/结果 | 可枚举恢复状态、用户选择、下一安全动作和新的 expected OID；重复 mutation、静默覆盖和误报 PASS 数为 0 |
| 前置条件 | 执行步骤能提供最小 receipt/OID；状态未知时必须先停止写入并请求人工确认 |
| 排除情况 | 不执行 force/reset/rebase/orphan；不自动解决冲突；不跨仓自动回滚；不把 dirty 工作区静默清理掉 |
| 典型交互 | 系统：“发布库已 push 到 OID A；过程库因 remote OID 从 B 变为 C 未执行。建议重新读取 C 后只重试过程库，或保留部分结果并暂停。请选择。” |
| 解决痛点 / 用户价值 | 用户面对的是清晰事实和少量可恢复选项，而不是隐藏状态、重复执行或不可解释的半成品 |
| 适用性选择 | `重试失败步骤 / 接受部分结果 / 人工恢复 / 保持现场并暂停` |
| 风险与资源边界 | 只读诊断继承原 Work 档位；任何会覆盖现状、重写历史、修改授权边界或恢复生产状态的动作升级 G2 并单独授权。恢复不自动重置原 Work 预算；超限需新预算或拆出恢复 Work |

### UC-VNEXT-UJ-013：按需查看项目、阶段和当前 Work 真相

**用户故事**：作为项目负责人，我要随时用自然语言查询项目现在做到哪、当前 Work 为什么阻塞、最近做过什么决定，以便快速获得可信答案，而不必让系统每次读取全部过程文档和历史记录。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；P-04 审批者 / Reviewer 可按权限查看待审批事项和证据摘要 |
| 触发条件 | 用户询问“项目进展如何”“当前阶段是什么”“这个 Work 为什么 blocked”“上次决定了什么”或请求查看某项证据 |
| 输入 | 自然语言问题、当前项目身份；可选的 Phase/Work/decision ID 和期望详细程度 |
| 处理逻辑 | 系统先从 Project/Phase/Work 的轻量索引和当前引用回答；默认只读取能直接回答问题的对象，输出结论、状态、下一步和少量证据链接；只有用户追问细节或索引冲突时才逐层扩读，并在扩读前说明将读取什么、为什么；不同项目的查询严格切换到各自独立过程仓 |
| 输出/结果 | 一份短、可追溯的项目/阶段/Work 状态摘要，以及必要的 canonical refs；默认全文档扫描数为 0，跨项目读取数为 0 |
| 前置条件 | 当前项目身份唯一；轻量索引和引用可读；查询者具有相应读取权限 |
| 排除情况 | 不默认加载全部 Roadmap、历史 CR、所有 Story/Work、完整测试报告或会话 transcript；不把过期摘要冒充当前事实；不跨项目拼接结果 |
| 典型交互 | 用户：“项目 X 现在卡在哪里？”系统：“当前 Phase=PH-002；Work W-014 因远端 OID 漂移 blocked，建议按 UJ-012 重新观察后重试。依据：Work 摘要和两条 OID receipt。需要展开失败日志吗？” |
| 解决痛点 / 用户价值 | 把“读文档”从全量上下文装载改成面向问题的最小查询，直接降低等待、认知负担和 token 消耗 |
| 适用性选择 | `短摘要 / 展开指定对象 / 只列证据链接 / N/A（无过程仓查询需求）` |
| 风险与资源边界 | 默认查询为 G0：最多读取 5 个直接引用对象、写入 0、最多 1 组一致性检查，并计入当前 Work/会话预算；涉及跨项目汇总、敏感证据、全历史审计或索引冲突时升级 G1/G2 并先批准扩读范围 |

### UC-VNEXT-UJ-014：在项目或关键阶段完成后进行证据化复盘

**用户故事**：作为项目负责人，我要在项目完成、重大 Roadmap 发布切片完成或关键 Phase 收尾后，基于可验证证据复盘交付结果、流程规范符合度、Meta Flow 适配性、推进效率和各阶段 token 合理性，以便知道哪些做法应保留、简化、加强或进入后续改进，而不是凭印象评价项目和工作流。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者；重大结论、强审计或高风险项目由 P-04 审批者 / Reviewer 复核 |
| 触发条件 | Project 完成、关键 Roadmap 发布切片完成、Phase 收尾，或用户对一组异常 Work 发起专项复盘；长生命周期项目不必等到整个 Project 永久结束 |
| 输入 | 用户确认的复盘范围和时间窗口、Project/Roadmap/Phase/Work 轻量索引、目标与验收结果、checkpoint/evidence 摘要、风险/恢复记录、read/write/check 使用量、token telemetry 与用户反馈 |
| 处理逻辑 | 系统先让用户在“完整项目 / 最近发布切片（推荐）/ 指定 Phase 或异常 Work”中选择范围并复述确认；随后优先读取 evidence index、结果摘要和 token 汇总，只在证据冲突或用户追问时扩读。复盘分别分析：①目标与用户价值；②流程规范符合度和证据可信度；③质量、风险与恢复；④交付流动效率；⑤token 与上下文合理性；⑥Meta Flow 适配性和改进价值。每项结论标记为 `事实 / 推断 / 待人工判断`，token 标记 `measured / proxy / unavailable`；流程是否被遵守与流程是否合理必须分别结论化 |
| 输出/结果 | 一份结构化项目复盘总结报告，至少包含复盘范围、审批者摘要、目标达成、流程符合度、质量/恢复、阶段效率、阶段 token、Meta Flow 适配性、改进候选、剩余风险和证据质量；报告结论经用户确认后成为事实与候选基线，但不授权修改 Meta Flow |
| 前置条件 | 复盘窗口已结束或已冻结；目标、Work 终态和最小证据索引可读；无法测量的 token/时间数据必须诚实标为 unavailable |
| 排除情况 | 不默认读取完整会话、全量 diff、全部历史 CR/Work 或所有测试日志；不把 unavailable 当 0；不因流程符合率高就推断流程有效；不从报告直接生成并执行代码、规则修改、commit 或 push |
| 典型交互 | 用户：“复盘刚完成的 Phase。”系统：“建议复盘 PH-003 及其 6 个 Work，而不是整个项目；预计读取 10 个摘要对象，分析目标、合规、质量、流动效率、token 和流程适配。确认吗？”用户确认后，系统输出：“验证门全部执行，但两次 G0 被误升 G1；需求阶段 token 占 46%，其中 3 次重复扩读；建议把该问题列为改进候选，而不是自动修改流程。” |
| 解决痛点 / 用户价值 | 用户既能发现“团队没有按 Meta Flow 做”的执行偏差，也能发现“Meta Flow 本身不适合这个项目”的设计问题；token 和流程成本可以按阶段、风险与产出解释，而不是只看总数 |
| 适用性选择 | `完整项目复盘 / 最近发布切片 / 指定 Phase / 指定异常 Work / 暂缓` |
| 风险与资源边界 | 默认 G1，并使用独立复盘预算；单个小 Work 可为 G0；安全事故、权限绕过、生产故障、跨项目污染或强审计调查为 G2。到达预算上限时先输出证据缺口和缩小范围建议，不静默扩读；报告生成不授权任何实现或外部写入 |

**复盘报告的六个核心维度：**

| 维度 | 核心问题 | 候选量化指标 |
|---|---|---|
| 目标与用户价值 | 是否交付了用户需要的结果，而不只是走完流程？ | 成功标准通过率、计划/实际范围差异、未交付 P0/P1、最终确认轮数 |
| 规范符合度与证据可信度 | 必需阶段、门禁和授权是否真实发生，证据能否重放？ | 必需证据覆盖率、缺失证据、越权/绕过、waiver、跨项目意外变化 |
| 质量、风险与恢复 | 缺陷何时发现，失败能否安全恢复？ | 逃逸缺陷、返工轮次、CP 重试、恢复成功率、恢复时长、剩余风险 |
| 交付流动效率 | 时间花在产生价值、等待、返工还是重复交接？ | 各阶段 cycle time、等待占比、handoff、重复读取/检查、一次通过率 |
| Token 与上下文合理性 | 每阶段消耗是否与风险、规模和产出相称？ | 阶段 token/占比、预算使用率、超限、扩读、每个已接受成果 token、unavailable 比例 |
| Meta Flow 适配性 | 哪些步骤帮助交付，哪些造成摩擦或诱发绕行？ | workaround、规则冲突、重复解释、不必要人工门、G 档误判、可执行改进建议占比 |

### UC-VNEXT-UJ-015：审议复盘建议并批准有界进化包

**用户故事**：作为审批者和项目负责人，我要在认可复盘报告事实后，逐项审议改进建议的证据、收益、成本、风险、范围、验证和回退条件，以便只把值得做且边界清楚的建议转化为有界进化包，而不让“报告正确”被误解为“允许系统修改自身”。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-04 审批者 / Reviewer 负责事实与风险审议；P-08 项目负责人 / Work 发起者确认改进价值、实施优先级和是否进入后续 Work/CR |
| 触发条件 | UJ-014 的复盘报告已由用户确认事实部分，且报告中存在一个或多个 Meta Flow 改进候选 |
| 输入 | 已确认复盘报告、单项改进候选、证据强度、适用项目范围、预期收益、影响面、候选 G 档、预计 reads/writes/checks/token、验收指标、回退和试点建议 |
| 处理逻辑 | 系统一次展示一个建议，先区分“项目本地配置问题 / Meta Flow 通用问题 / 证据不足 / 暂不处理”；再提供 `接受为进化包 / 修改后再审 / 延后 / 拒绝` 选项。接受项必须固化唯一目标、来源证据、当前基线/exact OID、适用范围、allowed reads/writes、required checks、G 档、token 预算、不修改项、验收阈值、回退条件、试点范围和 Reviewer。事实确认、建议选择、执行授权、commit/push/production 等外部写入授权四种语义分别记录，不允许用一次 approve 合并 |
| 输出/结果 | 对每项建议形成 `accepted / changed / deferred / rejected` 决定；只有 accepted 项生成一个尚未执行的有界进化包。被延后/拒绝项保留理由和重启条件；进化包本身不等于已经创建 Work/CR，也不等于授权实现或发布 |
| 前置条件 | 复盘事实已确认；建议能追溯到至少一个具体问题和证据；无法证明通用性的单项目摩擦默认只作为项目本地候选 |
| 排除情况 | 不自动接受报告中的全部建议；不把多个低相关建议打包成“优化整体流程”；不根据一个项目的局部偏好直接修改所有项目默认规则；不允许 Meta Flow 同时充当建议者、批准者和最终验证者 |
| 典型交互 | 系统：“建议把 G1 需求确认从八份文档缩为 REQUEST+场景记录。证据来自 2 个项目、7 个 Work，预计 token 降低 35%，风险是追溯信息不足。推荐接受为 G2 进化包，先在 meta-flow 和一个试点项目验证；本次只批准进化包，不批准实现或 push。请选择接受、修改、延后或拒绝。” |
| 解决痛点 / 用户价值 | 把复盘事实、产品决策、执行授权和外部写入授权分开，既能推动真正有价值的改进，又避免“报告生成后系统自动修改自己”的授权偷渡 |
| 适用性选择 | `接受为有界进化包 / 修改范围或验证条件 / 延后 / 拒绝并记录原因` |
| 风险与资源边界 | 建议审议通常为 G1；涉及安全、权限、人工门、公共契约或跨项目默认行为时按 G2 审议。通用效率优化默认要求至少两个项目或多个独立 Work 的证据；安全/正确性缺陷可基于单个重大事件进入 G2，但仍需人工批准 |

### UC-VNEXT-UJ-016：在受控 Work/CR 中实现并验证 Meta Flow 自进化

**用户故事**：作为 Meta Flow 的项目负责人，我要把已经人工审议、具有明确证据、范围、风险等级、验收标准和回退方案的进化包，作为有界 Work 或正式 CR 实施、试点和回测，以便 Meta Flow 从真实项目反馈中持续改进，同时避免无限自修改、越权扩展或为了降低 token 损害质量。

| 字段 | 内容 |
|---|---|
| 使用角色 | P-08 项目负责人 / Work 发起者选择并启动单个进化包；P-04 审批者 / Reviewer 批准高风险设计、验证结论和推广决策 |
| 触发条件 | UJ-015 已生成并批准一个有界进化包；本次执行范围、基线、G 档、预算、验证和回退条件均完整 |
| 输入 | 单个 approved evolution package、来源复盘和证据、当前 Meta Flow 版本/exact OID、allowed reads/writes、required checks、验收阈值、试点/canary 范围、回退和外部写入授权状态 |
| 处理逻辑 | 用户从已批准进化包中选择一项，系统展示目标、基线、范围、预算、风险、验证和“不授权项”；用户确认后按 UJ-005/006 将其作为 G0/G1 Work 或正式 CR/G2，固定可恢复基线，再在 allowlist 内实现。验证必须先重现原问题，再证明目标指标改善，同时检查质量、返工、证据完整性、恢复能力、生产默认行为、其他平台和其他项目无未授权回归；高风险变更先在 fixture/临时项目或受控 dogfood/canary 中验证。用户根据回测决定推广、修改、回退或停止；commit/push/publication 继续按 UJ-010 单独授权。完成后记录假设是否成立和实测结果，但不得自动触发下一轮自进化 |
| 输出/结果 | 一个边界明确的 Meta Flow 改进、风险匹配的验证证据、试点前后指标对比、推广/回退决定和进化结果报告；若未达到阈值或出现回归，则停止推广并保留可审计的失败与回退原因 |
| 前置条件 | 进化包已批准且未过期；基线/tag或 exact OID 可恢复；目标问题可重现或有足够证据；独立 Reviewer 可用于高风险结论 |
| 排除情况 | 不从复盘报告直接生成并执行代码；不一次执行全部建议；不以“自进化”读取/修改全仓；不自动改变安全、权限或人工门；不形成递归后台自修改链；不以 force/history rewrite 或删除证据回退 |
| 典型交互 | 用户：“执行 EVO-003。”系统：“该包将修改需求确认规则和两个 Skill，G2，预算为 18 reads/12 writes/7 checks/80k token；先在 fixture 和 meta-flow dogfood，成功阈值是需求阶段 token P50 降低 30%、返工率不升、追溯覆盖率保持 100%。本次不授权 push。确认启动吗？”验证后系统报告实际改善和回归，再由用户决定是否推广 |
| 解决痛点 / 用户价值 | Meta Flow 能从真实项目数据中进化，但每次进化仍然是可理解、可审查、可验证、可回退的一代变更，而不是失控的自主改写循环 |
| 适用性选择 | `作为 G0/G1 Work 实施 / 作为正式 CR/G2 实施 / 修改进化包后再审 / 暂缓或取消` |
| 风险与资源边界 | 机械、局部、可逆且不改公共契约的变更可为 G0；跨模块但不改公共治理契约可为 G1；修改工作流状态机、Agent/Skill 共享契约、权限/安全/人工门、CP/ledger/State/context 真相源、跨平台安装发布、迁移或生产行为必须为正式 CR/G2。G2 仍有明确 allowlist、token/check 预算和回退阈值；发布库与过程库独立提交、独立 push、独立恢复 |

## 附录：覆盖自检表

| 维度 ID | 维度名称 | 状态 | 涉及场景 | 备注 |
|---|---|---|---|---|
| D1 | 用户维度 | 已补充 | UC-PG-001..007, UC-EI-001..005 | 新增审计者、证据生产者与成本分析者 |
| D2 | 任务维度 | 已补充 | UC-PG-001..007, UC-EI-001..005 | 补充时序、调度证明、重放、telemetry 与 append-only 迁移 |
| D3 | 动机维度 | 已补充 | UC-EI-001..005 | 可信审计、可复现重放、诚实成本和历史不可变 |
| D4 | 时间维度 | 已补充 | UC-EI-001, UC-EI-002, UC-EI-003, UC-EI-005 | 覆盖条件式批准、retry/supersession、as-executed/current-replay 与历史修正 |
| D5 | 环境维度 | 已补充 | UC-EI-002, UC-EI-005 | 区分平台 session、仓库证据与 quant-lab pilot |
| D6 | 方式维度 | 已补充 | UC-EI-001..005 | checker、ledger、receipt、hash、telemetry 与 migration manifest |
| D7 | 异常维度 | 已补充 | UC-EI-001..005 | 非法时序、缺 receipt/telemetry、retry、replay drift、历史不可变性 |
| D8 | 集成维度 | 已补充 | UC-EI-001..005 | 与 CP result、gate/state/dispatch/read ledger、checker 和 CR-163 衔接 |
| D1-WT | 用户维度 | 已补充 | UC-WT-001..007 | 覆盖维护者、审计者、迁移执行者、Reviewer 与非交互调用方 |
| D2-WT | 任务维度 | 已补充 | UC-WT-001..007 | 覆盖 clone/link、状态发现、CR tracking、Doctor、guardrail、lint、安装和历史收敛 |
| D3-WT | 动机维度 | 已补充 | UC-WT-001..007 | 单一 workflow truth、确定性质量门与诚实风险披露 |
| D4-WT | 时间维度 | 已补充 | UC-WT-001, UC-WT-003, UC-WT-007 | 覆盖首次 clone、发布前、closed CR 事后收敛与回退 |
| D5-WT | 环境维度 | 已补充 | UC-WT-002, UC-WT-004, UC-WT-006 | 覆盖不同设备、clean clone、CI/Agent 非 TTY 与本机 cache |
| D6-WT | 方式维度 | 已补充 | UC-WT-001..007 | 通过 JSON truth、symlink metadata、Doctor、Ruff、guardrail、dry-run 和 append-only correction 验证 |
| D7-WT | 异常维度 | 已补充 | UC-WT-001..007 | 覆盖 closed active CR、legacy index、断链、超预算、missing rule、lint error、缺 project-dir 和 receipt unavailable |
| D8-WT | 集成维度 | 已补充 | UC-WT-001..007 | 与 state/current/CR index、artifact repo、quality policy、README、CI 与 CR-046 evidence 链衔接 |
| D1-GB | 用户维度 | 已补充 | UC-GB-001..004 | 覆盖 Host Orchestrator、实现作者、迁移者与审批者 |
| D2-GB | 任务维度 | 已补充 | UC-GB-001..004 | 覆盖开启、发布、显式合并和关闭四个用户 outcome |
| D3-GB | 动机维度 | 已补充 | UC-GB-001..004 | 以跨设备可恢复、远端 ref 安全、完整交付旅程和减少手工漂移为目标 |
| D4-GB | 时间维度 | 已补充 | UC-GB-001..004 | 覆盖 CR 开启、实现提交后、显式 merge、清理和重复执行 |
| D5-GB | 环境维度 | 已补充 | UC-GB-001..004 | 覆盖双仓、远端默认分支、branch protection、并发推进与 forge 自动删 branch |
| D6-GB | 方式维度 | 已补充 | UC-GB-001..004 | 使用原生 Git、ff-only、upstream、exact OID、ancestry 与 dry-run |
| D7-GB | 异常维度 | 已补充 | UC-GB-001..004 | 覆盖 dirty/detached/divergence/collision/partial/non-ancestor/ref drift/默认分支写拒绝 |
| D8-GB | 集成维度 | 已补充 | UC-GB-001..004 | 与 workspace routing、CR route plan、双仓 push/merge、ledger/result 和 release cleanup 衔接 |
| D1-AW | 用户维度 | 已补充 | UC-AW-001..005 | 覆盖维护者、功能作者、迁移执行者与审批者 |
| D2-AW | 任务维度 | 已补充 | UC-AW-001..005 | 覆盖项目路由、worktree 管理、异构双 leg、单一聚合门、integration bootstrap 和迁移交接 |
| D3-AW | 动机维度 | 已补充 | UC-AW-001..005 | 避免共享 checkout/index/branch 污染，同时保留单仓共享与逐项目可管理性 |
| D4-AW | 时间维度 | 已补充 | UC-AW-002..005 | 覆盖 integration 首次 create-only 初始化、CR 短期激活、双 leg 独立终态、聚合、finish/abort 回归、CR 外人工同步和后续逐项目迁移 |
| D5-AW | 环境维度 | 已补充 | UC-AW-001..005 | 覆盖 existing clone、sibling worktree、legacy/new layout、local bare remote fixture 与跨设备路径 |
| D6-AW | 方式维度 | 已补充 | UC-AW-001..005 | 通过 metadata/schema、Git worktree、sparse namespace、namespaced branch、dry-run 和 manifest 验证 |
| D7-AW | 异常维度 | 已补充 | UC-AW-001..005 | 覆盖歧义路由、dirty current、dirty sibling、branch collision、integration expected-OID 漂移、leg 部分失败、断链与 stale registry |
| D8-AW | 集成维度 | 已补充 | UC-AW-001..005 | 与 workspace routing、git lifecycle、CR gates、artifact ownership、迁移手册和后续项目 CR 衔接 |
| D1-MR | 用户维度 | 已补充 | UC-MR-001..007 | 覆盖迁移准备协调者、维护者、审计者、证据生产者、迁移执行者和审批者 |
| D2-MR | 任务维度 | 已补充 | UC-MR-001..007 | 覆盖 external route、mode detection、seed normalization、durable evidence、bootstrap、runner、E2E 和 handoff |
| D3-MR | 动机维度 | 已补充 | UC-MR-001..007 | 以“先证明 migration-ready，再触碰真实布局”为核心用户价值 |
| D4-MR | 时间维度 | 已补充 | UC-MR-002..007 | 覆盖 intent 前、逐步 mutation、证据尾写入、进程重启、resume、CP0 import 与 CR-053 启动前 |
| D5-MR | 环境维度 | 已补充 | UC-MR-001, UC-MR-006 | 覆盖不同 sibling root、临时三并列目录、local bare remote 与真实仓只读边界 |
| D6-MR | 方式维度 | 已补充 | UC-MR-001..007 | 使用 schema v2、manifest/OID/CAS、content-addressed evidence、typed authorization、scoped check 和 workflow eval |
| D7-MR | 异常维度 | 已补充 | UC-MR-001..007 | 覆盖 route conflict、traversal、seed 越界删除、stale OID、evidence tail、half push、expired/replayed auth 和 third-state |
| D8-MR | 集成维度 | 已补充 | UC-MR-001..007 | 与 CR-051 AW 契约、CP0、workspace health、leg/aggregate、state projection、CP7/CP8 和 CR-053 衔接 |
| D1-VNEXT | 用户维度 | 已补充 | UC-VNEXT-001..006 | 覆盖项目负责人、维护者、审批者、执行者和审计者 |
| D2-VNEXT | 任务维度 | 已补充 | UC-VNEXT-001..006 | 覆盖接入、长期规划、Work 生命周期、风险路由、scope 执行与快照迁移 |
| D3-VNEXT | 动机维度 | 已补充 | UC-VNEXT-001..006 | 简单、可靠、低治理成本和跨项目零联动 |
| D4-VNEXT | 时间维度 | 已补充 | UC-VNEXT-002..006 | 覆盖长期 Roadmap/Phase、单次 Work、blocked/resume、cutover 和观察期 |
| D5-VNEXT | 环境维度 | 已补充 | UC-VNEXT-001, UC-VNEXT-005, UC-VNEXT-006 | 覆盖两个项目、两类仓库、不同 token telemetry 能力和只读旧仓 |
| D6-VNEXT | 方式维度 | 已补充 | UC-VNEXT-001..006 | 使用唯一路由、四层对象、G profile、scope allowlist、token budget、expected-OID CAS 和快照索引 |
| D7-VNEXT | 异常维度 | 已补充 | UC-VNEXT-001..006 | 覆盖多解路由、跨项目访问、非法状态跳转、预算超限、CAS 漂移、旧仓写入和迁移回滚 |
| D8-VNEXT | 集成维度 | 已补充 | UC-VNEXT-001..006 | 与现有发布仓、Host Orchestrator、检查器、审计证据和后续项目级迁移授权衔接 |
| D1-VNEXT-UJ | 用户维度 | 已补充 | UC-VNEXT-UJ-001..016 | 当前 active 场景以 P-08 项目负责人为用户故事主体；P-04 只在审批、复盘或评审时参与，系统内部角色不冒充业务用户 |
| D2-VNEXT-UJ | 任务维度 | 已补充 | UC-VNEXT-UJ-001..016 | 覆盖新建、既有接入、长期治理、需求确认、Work/CR、分级、执行、评审、验证、推送、关闭、恢复、查询、复盘、建议审议和自进化 |
| D3-VNEXT-UJ | 动机维度 | 已补充 | UC-VNEXT-UJ-001..016 | 每个场景均明确对应隔离、减少返工、降低流程/token、可靠发布、按需读取、可恢复、持续学习或改进价值 |
| D4-VNEXT-UJ | 时间维度 | 已补充 | UC-VNEXT-UJ-001..003, UC-VNEXT-UJ-007, UC-VNEXT-UJ-011..016 | 覆盖首次建立、既有迁移、日常查询/执行、中断恢复、阶段/项目收尾、复盘、改进试点和推广后观察 |
| D5-VNEXT-UJ | 环境维度 | 已补充 | UC-VNEXT-UJ-001..002, UC-VNEXT-UJ-007, UC-VNEXT-UJ-010..016 | 覆盖本地双仓、不同设备/会话、旧仓只读、远端漂移、弱网络、跨项目查询隔离和 fixture/dogfood/canary |
| D6-VNEXT-UJ | 方式维度 | 已补充 | UC-VNEXT-UJ-001..016 | 覆盖自然语言问询/查询、交互确认、最小 YAML/Markdown、Git、定向检查、复盘报告、进化包和回测；内部接口不冒充用户场景 |
| D7-VNEXT-UJ | 异常维度 | 已补充 | UC-VNEXT-UJ-002, UC-VNEXT-UJ-005..016 | 覆盖未知风险、预算超限、scope 漂移、检查失败、OID 漂移、中断、部分成功、索引冲突、证据不足、改进无效和回归 |
| D8-VNEXT-UJ | 集成维度 | 已补充 | UC-VNEXT-UJ-001..016 | 覆盖发布仓、过程仓、Git remote、Reviewer、测试/构建、长期投影、复盘证据和 Meta Flow dogfood；不采用双 leg/aggregate |
| Dx1-VNEXT-UJ | 成本与流动效率 | 已补充 | UC-VNEXT-UJ-004, UC-VNEXT-UJ-006..009, UC-VNEXT-UJ-013..016 | 同时分析 token、读取、检查、等待、返工和交接；降低 token 必须与质量、返工和证据完整性一起验证 |
| Dx2-VNEXT-UJ | 学习与进化闭环 | 已补充 | UC-VNEXT-UJ-014..016 | 明确 `事实复盘 → 建议审议 → 有界实现 → 试点回测 → 推广/回退`，每一代自进化均需人工确认且不能递归自动触发 |
