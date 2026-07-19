---
status: baseline
version: "3.0"
created_at: "2026-07-02"
owner: "meta-pm"
cr_ref: "CR-037"
active_change_ref: "CR-052"
current_baseline: "CR-052-VNEXT-R2-candidate"
source_story_map: "process/docs/product/STORY-MAP.md"
source_requirements: "process/docs/product/REQUIREMENTS.md"
---

# Meta Flow 项目治理与状态强制 MVP 范围

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 3.0 | 2026-07-19 | meta-pm | CR-052 vNext R2：新增 IN/OUT/DEF-VNEXT 与 MVP-SM-VNEXT，当前 MVP 改为每项目双库、四层治理、G0/G1/G2、Work scope/token、main CAS 和快照 2×2 试点；旧 PG/EI/WT 能力 reframed，GB/AW/MR 主线 superseded。保留全部历史 Scope/Deferred/Metric ID。 | 原文档增量 reframe；CP2 人工门待批准 |
| 2.1 | 2026-07-19 | host-orchestrator | CP2 开门前将 IN-MR-007/OUT-MR-007/MVP-SM-MR-08 从 CR catalog 专项扩为 CR catalog + event-ledger frozen fingerprint；记录 19+1 历史错误，不新增范围 ID。 | 原文档增量修订；历史修复仍为 follow-up |
| 1.4 | 2026-07-13 | host-orchestrator-inline-fallback | 增量加入 CR-047 workflow-truth、Doctor、guardrail、Ruff、非交互安装和 CR-046 状态收敛范围。 | 原文档增量更新 |
| 1.5 | 2026-07-15 | host-orchestrator inline fallback | 增量加入 CR-050 的双仓 CR branch open、committed-ref publish 与 proof-gated cleanup；将 forge receipt、Git Town、自动 merge/commit 保持在范围外或 deferred。 | 原文档增量更新 |
| 1.6 | 2026-07-16 | host-orchestrator inline fallback | 将独立两仓 fast-forward-only merge 纳入 CR-050 MVP；继续排除隐式 merge、merge commit、rebase/force/自动冲突解决、forge API 与策略绕过；等待 CP2 R2。 | 原文档增量更新；保留 1.5 历史范围 |
| 1.7 | 2026-07-17 | meta-pm | 增量加入CR-051 project-first路由、每项目独立worktree、项目作用域Git周期、共享main显式refresh和migration preflight；明确真实迁移/软链接/真实Git mutation后置。 | 原文档增量更新；保留全部既有Scope/Deferred ID |
| 1.8 | 2026-07-17 | meta-pm | CR-051 CP2 R2：保留全部 Scope/Deferred ID，按用户决策把 worktree/branch MVP 修订为长期项目integration、短期CR branch、shared main、显式merge-main与existing-control+sibling-worktree，并把idle detached明确移出候选基线。 | 原文档增量更新；R2总体门仍待approve |
| 1.9 | 2026-07-18 | meta-pm | CR-051 CP2 R3：保留全部Scope/Deferred/Metric ID，当前MVP改为integration create-only、异构source/artifact双leg、单一aggregate gate与CR外人工main/integration同步；R2 per-CR refresh只保留历史。 | 原文档增量更新；R3总体门仍待approve |
| 2.0 | 2026-07-19 | meta-pm | 为 CR-052 增量新增 IN-MR-001..007、OUT-MR-001..008、DEF-MR-001..005 与 MVP-SM-MR-01..10；保留全部 CR-051 Scope/Deferred/Metric ID，明确 CR-052 能力建设与 CR-053 真实迁移边界。 | 原文档增量更新；CR-052 CP2 人工门待批准 |
| 1.0 | 2026-07-02 | meta-pm | 建立本轮实施的 MVP / Out of Scope / Deferred 范围 | 初始化长期可追踪范围基线 |
| 1.2 | 2026-07-11 | meta-pm | 增量加入 CR-046 evidence-integrity MVP、明确不授权项和 CR-163 append-only pilot；保留全部 PG scope ID | 原文档增量更新 |
| 1.3 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：把 compaction 语义保持、通用 post-close correction、机器 audit report、null-provenance dogfooding 和 dispatch 披露纳入既有 IN-EI scope | 原文档增量更新 |

## MVP 目标

本轮 MVP 的目标是先把 meta-flow 的轻量状态入口和项目治理状态边界变成可执行、可校验、可迁移的机制，再用 quant-lab 作为真实样本验证。MVP 不追求新增第二套治理体系，而是在既有 state、CR、ledger、context、gate 和 checker 体系上补齐缺口。

CR-050 的增量目标是在既有 `workspace git-status/push` 与 CR route plan 上增加四段式 Git branch lifecycle：两仓安全开启、只发布已提交 refs、独立显式 fast-forward-only merge、合并后以 fresh ancestry/tip 证明清理。它不引入 `gb`/Git Town 必需依赖，也不调用 forge API 或创建 merge commit。

CR-051 的增量目标是把CR-050的artifact侧作用域从共享control checkout收紧为当前project独立worktree，并建立project-first `<project>/docs|process` 路由、legacy dual-read、多项目并发隔离、长期 `projects/<project>/integration`、短期 `projects/<project>/cr/<cr-id>-<slug>`、integration create-only初始化、异构source/artifact双leg与单一aggregate gate。source leg从/回源码默认分支，artifact leg从/回项目integration；shared `main`同步完全在CR外人工执行。现有clone继续作为control checkout，项目worktree位于configurable sibling root并执行namespace/sparse/owned-path gate。MVP只开发和验证能力；用户后续逐个项目执行真实artifact迁移与软链接挂接。

CR-052 的增量目标是把 CR-051 的库级契约升级为 migration-ready 能力：在不触碰真实 meta-flow artifact 布局的前提下，补齐 schema v2 external route、health mode、seed-normalization/steady ownership、durable evidence、transitional bootstrap、typed migration runner、scoped CR check，并在隔离的三并列临时拓扑和 local bare remote 中执行真实 mutation、故障注入与 resume。CR-053 才迁移真实文件、软链接、integration/worktree/ref；ptm-team、ptm-atomic、quant-lab、shared main 和 repository publication 都不属于 CR-052。

上述 CR-052 migration-readiness 目标现作为 v2.1 历史基线保留，并已被 vNext R2 当前候选范围 supersede。vNext R2 的 MVP 目标是直接消除共享过程仓和 working tree：每项目恰好一个现有发布库与一个独立过程库；过程库以 main-only 单写 publisher + expected-OID CAS 承载 `Project -> Roadmap -> Phase -> Work`，每个 Work 通过 G0/G1/G2 与 read/write/check/token scope 控制治理成本。迁移首版只复制当前快照，旧共享仓只读；用两个项目、每项目两个完整 Work 周期证明跨项目变化为 0。真实仓创建、迁移、commit/push 和 production cutover 仍需 CP2 之后的设计门与独立运行授权。

## 历史范围状态映射

| 历史范围 | vNext 状态 | 当前处理 |
|---|---|---|
| IN/OUT/DEF-PG、EI、WT | reframed | 保留历史交付事实；后续只作为 vNext Work scoped 能力依赖，不按原全量链路整体进入当前 MVP |
| IN/OUT/DEF-GB、AW、MR | superseded | 保留审计、风险与不授权事实；共享 artifact worktree/migration-readiness 不再属于 CR-052 当前实施范围 |
| IN/OUT/DEF-VNEXT | current-candidate | 当前 CP2 候选范围；CP2 approved 前不授权设计下游、源码或真实迁移 |

## In Scope

| Scope ID | 范围项 | 优先级 | 对应 Story | 推荐理由 |
|---|---|---|---|---|
| IN-PG-001 | `STATE.current.json` allowlist、字段预算、audit/enforce 灰度 | P0 | ST-PG-001 | 当前已出现 v2 约束失效，必须先阻断继续污染。 |
| IN-PG-002 | 受控 current state 更新 API 和内部写入路径收敛 | P0 | ST-PG-002 | 没有受控入口，schema 规则无法成为真实门禁。 |
| IN-PG-003 | Agent / Skill current state 写契约同步 | P0 | ST-PG-003 | 需要让人和 agent 都知道重型状态应写到何处。 |
| IN-PG-004 | `process/project/` scaffold、`PROJECT.current.json`、`PROJECT-SCALE.yaml` | P1 | ST-PG-004, ST-PG-005 | 长期项目治理需要独立 refs-only 项目状态，不能继续塞入 current state。 |
| IN-PG-005 | capability / feature registry 引用校验 | P1 | ST-PG-006 | 防止 migration 把自由字符串升级成新的隐性命名空间。 |
| IN-PG-006 | impact surface normalization 和 migration report | P1 | ST-PG-007 | 恢复冲突检测语义，避免路径和治理面混写。 |
| IN-PG-007 | ROADMAP-REFRESH result/checker、GATE-LEDGER event、过程归档库内 cascade | P1 | ST-PG-008, ST-PG-009 | 支持 roadmap refresh，同时保持跨仓写入边界清晰。 |
| IN-PG-008 | FU-RF / SP-RF / RA-RF tracking 和 project stale check | P1 | ST-PG-010 | 让 refresh 发现的后续影响可持续追踪。 |
| IN-PG-009 | quant-lab P2 迁移验证 | P2 | ST-PG-011 | 用真实长期项目验证机制完整性，但不混入机制实现 CR。 |
| IN-PG-010 | ledger compaction 命令边界和最小 CLI 契约 | P0.5 | ST-PG-012 | 明确 ledger compact 不复用 state compact，先冻结命令职责边界，完整 retention 策略可在 FEAT-PG-002 设计中细化。 |
| IN-PG-011 | second-system guardrail | P0 | ST-PG-013 | 所有 CR-037 子能力必须复用现有 state/context/result/ledger/registry 体系，新增平行机制必须被设计审查阻断。 |
| IN-EI-001 | Gate temporal invariant 与条件式批准重放 | P0 | ST-EI-001 | 关闭“schema PASS 但实际时序不可信”的核心缺口。 |
| IN-EI-002 | Dispatch receipt/attempt/retry/supersession/terminal closure contract，以及 compaction/restore 语义保持和 session/repository 证据层级披露 | P0 | ST-EI-002 | 平台调用必须可证明；平台不提供 receipt 时只能诚实披露 `session-observed/repository-unverifiable` 或 unavailable；压缩不得混淆标识或丢失 attempt chain。 |
| IN-EI-003 | CP check attempt、输入 hash、final correlation 与跨真相源一致性 | P0 | ST-EI-003 | 让最终结论关联真实执行并可检测 result/checkpoint/gate/state/ledger 冲突。 |
| IN-EI-004 | delivered finalization、workflow health、authorized read expansion 与 provenance-bearing machine audit report | P0 | ST-EI-004 | 防止 delivered 残留 active refs、悬空 health ref 和隐式全文读取，并让 event/attempt/thread/outcome/token 指标可机器复核。 |
| IN-EI-005 | 平台 token telemetry 与 unavailable/proxy 分层 | P0 | ST-EI-005 | 提供可信 workflow 成本，不把估算冒充实测。 |
| IN-EI-006 | checker identity、schema/policy hash、as-executed/current-replay、CR-046 CP1/CP2 null-provenance dogfooding | P0 | ST-EI-006 | 使历史证据在 checker 演进后仍可解释、可复现；缺 provenance 的原始结果不被静默美化。 |
| IN-EI-007 | 可复用 post-close correction lifecycle 与 quant-lab CR-163 append-only migration/replay fixture | P1 | ST-EI-007 | 先建立通用 correction schema/audit trail，再由 pilot 消费并达成 23/23 current replay，同时保持 lineage 业务源码 diff 为 0。 |
| IN-WT-001 | State/CURRENT/JSON CR index 一致性与 legacy YAML 迁移 | P0 | ST-WT-001 | 当前 canonical tracking 直接失败。 |
| IN-WT-002 | portable process/docs 路由与 clean-clone 验证 | P0 | ST-WT-002 | 消除跨设备过程真相漂移。 |
| IN-WT-003 | Doctor budget、quality source、read-expansion、future Run ledger 收敛 | P0 | ST-WT-003 | 发布质量门当前为红。 |
| IN-WT-004 | tracked rule source、guardrail clean-clone 与 cache policy | P0 | ST-WT-004 | 当前 ignored root rule 与 guardrail 契约矛盾。 |
| IN-WT-005 | Ruff 0 与完整回归门 | P0 | ST-WT-005 | 当前 90 项 lint 未闭环。 |
| IN-WT-006 | 三平台非交互安装示例与 preflight | P1 | ST-WT-006 | README 首入口在非 TTY 失败。 |
| IN-WT-007 | CR-046 产品矩阵/验证状态收敛 | P0 | ST-WT-007 | 当前文档仍含陈旧状态，但正式 evidence 已存在。 |
| IN-GB-001 | 两仓 CR branch open：precheck、remote default discovery、fetch/prune、ff-only refresh、同名 branch 与 upstream | P0 | ST-GB-001 | 保证源码与过程证据从各自最新主分支开始，并可跨设备一起恢复。 |
| IN-GB-002 | 只发布已显式提交 refs：clean/wrong-branch/non-fast-forward gate、逐仓 push/OID 核验、dry-run 与 partial result | P0 | ST-GB-002 | 防止工具隐式 stage/commit 或把单仓成功冒充双仓发布完成。 |
| IN-GB-003 | proof-gated cleanup：exact branch/tip、ancestry、protected ref、remote-auto-delete 幂等与 local `branch -d` | P0 | ST-GB-003 | 只有充分证明已合并时才删除目标 refs；证明不足时 fail closed。 |
| IN-GB-004 | 显式双仓 merge：preflight-all、artifact→project、fast-forward-only default update、独立授权、逐仓 result、partial retention 与 finish gate | P0 | ST-GB-004 | 完成用户要求的 publish 后合并旅程，同时让 default-branch 写入、分支保护和部分成功保持可审计、可恢复。 |
| IN-AW-001 | project-first路由、layout version、project identity与legacy dual-read | P0 | ST-AW-001 | 让每项目docs/process归属清晰，同时允许未迁移项目继续确定读取而不静默切换写目标。 |
| IN-AW-002 | 长期每项目worktree的create/check/list/safe-remove、registry/health；idle=`projects/<project>/integration`、active=`projects/<project>/cr/<cr-id>-<slug>`；缺失integration从fresh `origin/main` exact OID create-only初始化 | P0 | ST-AW-002 | 避免共享checkout/index/project branch，为后续软链接提供稳定目标，且不覆盖/重建既有integration。 |
| IN-AW-003 | 一个逻辑CR的异构source/artifact双leg、finish/abort、dirty/ownership/control/sibling门；source目标源码default，artifact目标项目integration | P0 | ST-AW-003 | sibling dirty不阻断当前项目，artifact CR不接触shared main，结束CR不继续占用worktree，任何跨项目mutation都fail closed。 |
| IN-AW-004 | 两leg独立结果、单一aggregate gate、`BLOCKED > FAIL > IN_PROGRESS > PASS`、PARTIAL progress/effect、integration expected-OID门与双项目bare-remote fixture | P0 | ST-AW-004 | 仅全PASS完成逻辑CR；部分失败不自动关闭、回滚或伪装完成；main/integration divergence不成为单CR blocker。 |
| IN-AW-005 | per-project migration preflight/manifest、验证/回滚checklist与迁移手册 | P1 | ST-AW-005 | 支持用户后续逐项目执行，同时把本CR真实迁移保持为0。 |
| IN-MR-001 | schema v2 external anchor、legacy/v1/v2 dual-read、v2-only write与health三模式 | P0 | ST-MR-001 | 真实sibling拓扑必须可移植且fail closed，兼容可读不能冒充已迁移。 |
| IN-MR-002 | frozen manifest、exact seed prune、unexpected=0与receipt后steady ownership激活 | P0 | ST-MR-002 | 把一次性normalization与长期ownership分开，避免误删sibling或产生双写。 |
| IN-MR-003 | filesystem/content-addressed durable leg/aggregate evidence、opaque handle、projection与audit-tail OID契约 | P0 | ST-MR-003 | 让执行事实跨进程可恢复，并把audit tail与target completion语义分离。 |
| IN-MR-004 | transitional bootstrap coordinator、intent/receipt、third-state、resume与CP0 import | P0 | ST-MR-004 | 解决route尚未原生化时的bootstrap悖论，不伪造native-first历史。 |
| IN-MR-005 | 通用migration engine、薄CLI、确定plan/dry-run和逐动作单次typed authorization | P0 | ST-MR-005 | execute默认禁用；错对象/OID/digest/expiry/replay必须在mutation前阻断。 |
| IN-MR-006 | 三并列临时目录+local bare remote真实mutation E2E、故障注入、跨进程readback/resume | P0 | ST-MR-006 | 只有真实临时mutation才能证明worktree/ref/link/filesystem与evidence恢复能力。 |
| IN-MR-007 | `cr check --id CR-052`、CR catalog/event-ledger 全局 fingerprint delta、workflow eval、文档和 CR-053 readiness handoff | P0 | ST-MR-006, ST-MR-007 | scoped contract 不被 19+1 历史治理噪声淹没；真实迁移只消费已验证能力。 |
| IN-VNEXT-001 | 每项目恰好1个现有发布库+1个独立过程库、唯一route、互斥ownership和跨项目物理隔离 | P0 | ST-VNEXT-001 | 从拓扑上消除共享过程仓/working-tree/branch联动，新增第三仓数为0。 |
| IN-VNEXT-002 | Project/Roadmap/Phase/Work四层、唯一父子关系、Work生命周期和最小长期投影 | P0 | ST-VNEXT-002, ST-VNEXT-003 | 支持大型项目长期治理，同时避免每次Work把长期对象写胖。 |
| IN-VNEXT-003 | 过程库main-only单写publisher、fresh expected-OID CAS、唯一receipt和幂等重放 | P0 | ST-VNEXT-003 | 以最小并发模型保证可靠，不恢复per-Work分支或自动merge。 |
| IN-VNEXT-004 | G0/G1/G2风险路由，G0=8读/8写/3检查/32k，G1=20读/24写/8检查/96k，高风险/超限进入G2 | P0 | ST-VNEXT-004 | 让简单Work真正轻量，并保留复杂/高风险Work的人工门。 |
| IN-VNEXT-005 | deny-default Work read/write/check scope、token usage诚实记录、超限前停止与升级 | P0 | ST-VNEXT-005 | 同时降低文档读取、归档写入、检查校验和上下文成本。 |
| IN-VNEXT-006 | 当前快照迁移、旧共享仓只读、项目级回滚、2项目×2周期试点和30天观察候选 | P1 | ST-VNEXT-006 | 用小批量可回滚证据证明隔离和预算，不重写历史或批量迁移。 |

## Out of Scope

| Scope ID | 排除项 | 原因 | 影响 |
|---|---|---|---|
| OUT-PG-001 | 第二套上下文治理、影响分析、capability 命名、result 或 ledger 体系 | 已批准方案明确否决，新增体系会扩大维护面 | 复用现有体系并补 checker / schema 缺口 |
| OUT-PG-002 | 跨仓原子事务 | 成本高且授权 / 回滚边界不清 | roadmap refresh 只自动写过程归档库 |
| OUT-PG-003 | 自动修改 quant-lab 发布库代码、测试和正式文档 | 发布库需要独立 CR 和用户授权 | 输出 stale finding / follow-up，由用户后续决策 |
| OUT-PG-004 | 直接修改 `GATE-PROFILES.json` | project scale 只表达默认偏好，不改变全局 gate profile 真相源 | 使用 `PROJECT-SCALE.yaml` |
| OUT-PG-005 | 新增五档项目规模矩阵或 regulated 独立档 | 当前 `lite / standard / full` 与 runtime-high-risk 足够表达 | 需要真实项目证明不足后再开 CR |
| OUT-PG-006 | 在产品阶段定义实现模块、函数签名细节或代码文件 diff | 属于设计 / 实现阶段 | 由后续 HLD / LLD / implementation 处理 |
| OUT-EI-001 | 伪造或追溯补齐历史 platform receipt、签名、token telemetry、checker identity | 会把未知事实伪装为已验证事实 | 缺失项显式 unavailable/unknown，并按证据层级降级 |
| OUT-EI-002 | 修改 quant-lab lineage contract、recorder、producer、consumer 或 admission 业务代码 | pilot 只验证 process evidence migration | 业务源码 diff 必须为 0；若需业务变更另开 CR |
| OUT-EI-003 | credentials、runtime、production write、publish、交易、repository commit/push | 当前产品门不授予这些权限 | 需要独立明确授权和安全/回滚边界 |
| OUT-EI-004 | 把所有 Story QA 合并成单一 batch 结论 | 会丢失 Story 独立结论和路由 | 只允许共享 regression 批量运行，Story verdict 保持独立 |
| OUT-WT-001 | 删除或处理 `meta-flow.process-prelink-backup-20260713T100930` | 用户明确排除 | 保持原状，不读取、不迁移、不删除 |
| OUT-WT-002 | 伪造平台 receipt、独立 QA、token telemetry 或历史 run 时间 | 证据不可恢复 | 保持 unavailable/legacy-unverified/READY_WITH_RISK |
| OUT-WT-003 | 本 CR 内执行 Git commit/push | CP2 只确认范围 | 另需用户明确授权 |
| OUT-GB-001 | 安装或强制使用 `gb`、Git Town、GitPython、GitHub/GitLab CLI | 原生 Git 已是项目依赖且可直接测试；`gb` 名义不唯一 | 可选 adapter 另行立项，不影响默认 CLI |
| OUT-GB-002 | `publish`/`finish` 隐式 merge、merge commit、自动审批 PR、自动冲突解决、绕过 branch protection 或 merge queue | 会模糊授权、制造非快进/冲突处理语义，并把工作流治理与 forge 策略耦合 | 本轮只允许独立显式 fast-forward-only merge；保护策略拒绝作为合法 BLOCKED/PARTIAL |
| OUT-GB-003 | squash/rebase merge 的 patch-id/内容相似度推断 | 不能提供 branch tip 已被主分支包含的确定证明 | 需要未来 forge receipt adapter |
| OUT-GB-004 | 隐式 `git add -A`、自动 commit/amend 或自动选择提交文件 | 可能夹带无关文件、个人配置或 secret | 提交保持显式；本轮只推送 committed refs |
| OUT-GB-005 | 跨仓原子 ref transaction | Git 远端不提供 project/artifact 跨仓事务 | 逐仓结果 + partial/blocking + 幂等恢复 |
| OUT-AW-001 | 搬迁真实`meta-flow-artifacts`中的任何`docs/**`或`process/**`文件 | 用户选择能力先行、迁移后置；共享仓当前还有其他项目活动内容 | 本CR只输出fixture验证和migration manifest |
| OUT-AW-002 | 创建、替换或修复任何现有项目的`process`/`docs`/`checkpoints`软链接 | 链接目标切换属于具体项目迁移和回滚边界 | 用户后续逐项目执行并验证 |
| OUT-AW-003 | 在真实artifact仓创建/删除worktree、branch、commit、tag、stash或remote ref | 本轮只授权能力开发，真实Git状态变更会影响共享项目 | 仅临时repo/bare remote fixture可mutation |
| OUT-AW-004 | 本轮将现有artifact control checkout转换为bare repo | 会扩大迁移与操作风险，非能力MVP必需 | 先兼容现有clone+sibling worktree；真实试点后再判断 |
| OUT-AW-005 | per-CR artifact main refresh/merge/update、自动main↔integration同步、idle detached、自动rebase/force/回滚或自动重建项目branch | 这些行为会把跨项目同步责任塞入单CR、改写历史或覆盖既有integration | 本基线采用异构双leg + 长期integration + 单一聚合门；main↔integration仅在CR外人工维护，任何自动化需新CR和额外授权 |
| OUT-AW-006 | 真实远端多项目pilot或publication | 需要独立repo/ref/OID授权与branch protection边界 | 本轮只用本地bare remote；CP8披露真实remote未验证 |
| OUT-MR-001 | 迁移真实`meta-flow` artifact文件或删除legacy布局 | 与CR-053的授权、备份和回滚边界不同 | CR-052真实布局文件mutation=0，进入DEF-MR-001 |
| OUT-MR-002 | 创建、替换、修复或删除真实`process`/`docs`软链接 | link swap是具体迁移的关键切换点 | CR-053按fresh manifest和独立link授权执行 |
| OUT-MR-003 | 创建/切换/删除真实project integration、worktree、branch/ref、commit/tag/stash | 会影响共享artifact仓和当前工作流恢复 | 仅临时repo/bare remote fixture可mutation |
| OUT-MR-004 | 读取、发现、迁移或修改ptm-team、ptm-atomic、quant-lab | 用户明确排除，sibling项目有独立ownership/审计边界 | 只用合成namespace验证隔离；其他项目逐个新CR |
| OUT-MR-005 | shared-main内容/README写入、main↔integration同步与repository publication | 当前只授权产品/设计/临时验证，不授权shared-main或push | 进入DEF-MR-003或后续独立operation authorization |
| OUT-MR-006 | 全局放宽`..`、任意绝对route、总授权、force/tag/reset/rebase/orphan/自动merge或跨leg自动回滚 | 扩大路径逃逸、重放、历史改写与不可逆风险 | schema v2声明式anchor、逐动作typed auth和evidence-only resume |
| OUT-MR-007 | 修复 CR-001..033 全部历史 catalog 错误及既有 handoff ledger `context_ref` 缺口 | 与 migration readiness 用户价值无直接依赖且会扩大历史治理范围 | scoped PASS + global 新增 fingerprint=0；19+1 历史修复独立 follow-up |
| OUT-MR-008 | credentials、runtime、SaaS、production、publish、live/trading | 不属于本次meta-flow迁移能力产品门 | 需要独立高风险CR、最小权限、回滚和人工授权 |
| OUT-VNEXT-001 | 拆分或重写旧共享过程仓Git历史 | 审计语义风险和迁移成本高，当前快照足以启动隔离治理 | 旧仓只读索引；强制审计缺口出现后另开CR |
| OUT-VNEXT-002 | 旧CP/CR/Story无损自动转换 | 版本/语义异构，自动推断会制造虚假等价 | 首版仅保留来源索引和必要快照 |
| OUT-VNEXT-003 | 多publisher分布式自动merge、per-Work过程分支或force冲突处理 | 会重新引入分支/冲突治理复杂度 | main-only单写publisher+CAS失败后fresh重试 |
| OUT-VNEXT-004 | 批量迁移所有项目 | 每项目路由、快照、回滚和cutover不同，爆炸半径过大 | 2×2试点后仍须逐项目新授权 |
| OUT-VNEXT-005 | 未授权repository publication、remote push、runtime/production/publish/live/trading | CP2只确认产品基线，不能授予外部写入或高风险运行权限 | 后续对具体目标、OID、环境和回滚单独授权 |
| OUT-VNEXT-006 | CP2前正式Feature/Story/LLD、源码实现或真实仓迁移 | 关键产品决策尚未人工批准 | CP2 approved 后依次进入CP3/CP5和操作授权门 |

## Deferred

| ID | 延后项 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-PG-001 | Ledger compaction 高级保留策略扩展 | P0 后段 | MVP 只冻结命令边界和最小 CLI 契约；复杂 retention 窗口、归档层级和索引策略可后续细化 | P0 enforcement 稳定后，ledger 体积或读取成本成为实际瓶颈 |
| DEF-PG-002 | 更细粒度 project scale 或 regulated 档 | SGA-04 | 当前三档足够，避免过早复杂化 | 新项目无法用现有 gate bias 和 high-risk profile 表达 |
| DEF-PG-003 | 跨仓自动修复 stale 发布文档 | SGA-02 | 自动写发布库越权 | 用户批准具体发布库 CR，且有明确回滚策略 |
| DEF-PG-004 | 长期消费非 YAML capability 来源 | SGA-03 | 不利于稳定 checker | 标准 registry 不能表达某些能力状态时重新设计 |
| DEF-EI-001 | 跨平台统一加密签名 receipt | SGA-05 | 平台能力不一致，当前 contract 先表达实际可用性 | 所有目标平台提供稳定签名接口后另行设计 |
| DEF-EI-002 | 基于估算 token 的强制计费/配额门 | SGA-06 | estimate 不能替代平台 measured telemetry | telemetry 覆盖和估算误差模型通过独立验证后重启 |
| DEF-WT-001 | 真实平台 custom-agent receipt producer | CR-045/046 风险 | 当前平台面未提供 repository-verifiable receipt | 平台提供稳定 discovery/selector/receipt 后独立 CR |
| DEF-WT-002 | 独立 runtime/SaaS/pilot 验证 | 授权边界 | 当前未授权 runtime、凭据或外部写入 | 用户单独批准 runtime-high-risk CR |
| DEF-GB-001 | Forge receipt adapter 支持 squash/rebase merge 清理 | SGA-GB-04 | 需要平台 API、identity、token、PR/merge receipt 与最小权限契约 | 用户选择目标 forge 并批准独立高风险 CR |
| DEF-GB-002 | Git Town/stacked branch adapter | SGA-GB-01 | 普通 CR branch 不需要额外同步策略和安装面 | 真实 stacked/offline ship 场景出现且原生 Git 不足 |
| DEF-GB-003 | 自动 stage/commit planner | SGA-GB-05 | 文件选择、secret scan、双仓消息与回滚契约尚未批准 | 用户明确启动后续范围并确认 allowlist/rollback |
| DEF-AW-001 | 用户逐项目执行真实artifact文件迁移 | SGQ-AW-002 | 当前先完成能力开发，实际mapping/备份/回滚必须按项目确认 | CR-051交付READY后，用户为目标项目启动迁移CR/操作授权 |
| DEF-AW-002 | 用户逐项目重挂软链接并切换layout version | SGQ-AW-002 | 需与具体源码项目目录和bootstrap契约一起验证 | 对应项目文件迁移完成且preflight/checklist通过 |
| DEF-AW-003 | bare control repo转换或自动main↔integration同步/rebase/force线性历史策略 | SGA-AW-03/04未选备选 | 用户已选择existing control checkout与CR外人工同步；转换control形态或自动历史改写都会扩大操作面 | 真实多项目试点证明当前拓扑/人工同步成本不可接受，并通过新CR重新决策和授权 |
| DEF-MR-001 | CR-053执行真实meta-flow artifact文件迁移、link swap和project-first route激活 | SGQ-MR-001 / DQ52-1 | CR-052只建设/验证能力，真实对象有独立授权与回滚边界 | CR-052 CP8 READY/READY_WITH_RISK，用户正式启动CR-053并批准目标/OID/manifest/回滚 |
| DEF-MR-002 | 显式workspace_root支持非同父目录拓扑 | DQ52-2 | workspace_parent是当前三并列拓扑的最小可移植contract | 真实拓扑不满足project_root.parent或无法唯一解析时由CP3/新CR切换 |
| DEF-MR-003 | shared-main README-only合规更新 | DQ52-10 | 当前不授权shared-main内容变更或publication | 用户批准具体内容、target ref/OID和README-only publication |
| DEF-MR-004 | ptm-team、ptm-atomic、quant-lab项目迁移 | 用户明确排除 / DQ52-10 | 每个项目mapping、ownership、备份、授权和回滚不同，且本轮禁止读取 | 对应项目逐一启动独立迁移CR |
| DEF-MR-005 | evidence backend跨机器共享/复制协议 | DQ52-4 / SGA-MR-03 | P0先保证cross-process durable readback；跨机器边界尚待CP3冻结 | CR-053多设备恢复成为强制需求，或本地content-addressed store无法满足handoff |
| DEF-VNEXT-001 | 旧共享过程仓历史按项目拆分 | OUT-VNEXT-001 | 首版优先隔离和低风险快照，不重写历史 | 2×2试点后仍存在只读索引无法满足的强制审计需求 |
| DEF-VNEXT-002 | 旧CP/CR/Story版本化语义转换器 | OUT-VNEXT-002 | 先保持事实诚实，避免unknown被转换为confirmed | 至少两个项目证明恢复必须转换且转换schema/差异报告可验证 |
| DEF-VNEXT-003 | 多publisher队列、分片或自动merge | SGA-VNEXT-02未选备选 | 单写publisher最简单可靠，尚无吞吐瓶颈证据 | 连续两个试点周期违反已批准SLO，且CAS重试成为主要成本 |
| DEF-VNEXT-004 | 批量项目迁移编排 | OUT-VNEXT-004 | 试点前批量化会扩大范围和回滚风险 | 2项目×2周期全部通过，用户另批项目清单/顺序/逐项目回滚 |
| DEF-VNEXT-005 | 远端publication或production cutover自动化 | OUT-VNEXT-005 | 当前不授权外部写入/runtime | 具体repo/ref/OID/环境/最小权限和回滚获得独立高风险批准 |

## 范围边界规则

- P0 必须先于 P1 / P2：没有 current state enforcement，项目治理对象会继续被污染。
- P1 中 project state、registry 归一、impact surface、roadmap refresh 可以分 CR 交付，但必须保留依赖顺序。
- P2 quant-lab 迁移是验证切片，不是机制设计切片；迁移不得自动改 quant-lab 发布库。
- Roadmap refresh 的 `UPDATED_WITH_DOC_IMPACTS` 不等于授权修改发布库，只表示产生 must_check / stale_items / follow_up_candidates。
- 缺失 registry ID、无法解析 impact surface 或发布库 stale 均应进入 blocked finding / follow-up，不得由实现静默补全。
- 缺失 platform receipt 或 token telemetry 不构成虚构值的理由；必须输出 unavailable，并保留可用的较低证据层级。
- CR-163 pilot 必须 append-only；任何历史事件改写或 lineage business-code diff 都应阻断并回退设计/实现门。
- CR-051能力MVP必须以至少两个项目的临时worktree/bare-remote fixture验证；其他项目dirty不阻断当前项目，但当前项目dirty/identity/ownership异常必须阻断。
- project worktree branch必须包含project identity；产品层已冻结长期integration、短期CR、异构双leg、单一aggregate gate、create-only bootstrap及existing-control+sibling-root边界。CP3只细化metadata/path schema、sparse/owned-path gate、attach/switch/finish/abort、expected OID、leg correlation、aggregate schema和人工同步precheck，不得把artifact目标改回shared main。
- migration preflight PASS不等于授权搬文件、挂链接或改Git refs；每个真实项目迁移都需独立目标、备份、回滚和验证。
- CR-052 CP2 precheck PASS只表示产品基线可进入人工决策；DQ52-1..10和CP2总体门未批准前，ST-MR不得转为正式Story/LLD或源码实现。
- CR-052必须真实执行临时拓扑mutation；如果安全环境只能dry-run，CP8必须NOT_READY，不得声称migration-ready。
- 真实meta-flow/artifacts路径只允许只读快照或dry-run，ptm-team/ptm-atomic/quant-lab内容发现也禁止；所有隔离/seed fixture必须合成。
- CR-053必须消费CR-052已版本化能力和evidence，不能在生产迁移时现场新增核心route/bootstrap/evidence/runner逻辑。
- vNext R2 当前生效范围以 IN-VNEXT-001..006 为准；IN-MR-001..007 仅为 superseded 历史，不得继续拆 Story 或实现。
- G0/G1 默认上限是 CP2 推荐决策值：G0=8读/8写/3检查/32k token，G1=20读/24写/8检查/96k token；未获 CP2 批准前不得把数值写成已发布运行策略。
- 过程库采用 main-only 单写 publisher + expected-OID CAS；stale OID 只能 fail closed 和 fresh retry，不自动 merge/force。
- 快照迁移不等于历史迁移。cutover 后旧共享仓新增写入数必须为0，任何双写都应阻断并回退。
- 2×2试点必须覆盖每项目至少1个G0和1个G1周期；任一周期跨项目变化非0、scope违规或CAS静默覆盖即不得扩大迁移。

## 成功指标

| 指标 ID | 指标名称 | MVP 验收口径 |
|---|---|---|
| MVP-SM-01 | P0 污染阻断 | unknown / oversized current state 写入在 enforce 模式被拒绝。 |
| MVP-SM-02 | 项目状态瘦身 | `STATE.current.json` 仅通过 `project_state_ref` 引用项目级状态，`PROJECT.current.json` 不超过 16KB。 |
| MVP-SM-03 | 引用可解析 | capability / feature refs 缺失注册时不会通过检查，也不会自动生成 ID。 |
| MVP-SM-04 | 影响面可迁移 | 路径类 impact surface 被拆入 `affected_paths`，历史漂移有 migration report。 |
| MVP-SM-05 | refresh 不越权 | Roadmap refresh 自动写入只发生在过程归档库，发布库影响进入 follow-up。 |
| MVP-SM-06 | quant-lab 可验证 | P2 迁移后检查可复跑，发布库无自动写入。 |
| MVP-SM-EI-01 | 时序可信 | chronology negative fixtures 100% rejected；合法条件式批准可重放。 |
| MVP-SM-EI-02 | Attempt 闭环 | 适用 attempt terminal status、input hash 和 final correlation 覆盖率 100%。 |
| MVP-SM-EI-03 | Telemetry 诚实 | measurement status 覆盖率 100%，estimated/unavailable 不标 measured。 |
| MVP-SM-EI-04 | 重放与 pilot | replay provenance 完整或明确 unavailable；CR-163 current replay 23/23 PASS；业务源码 diff 为 0。 |
| MVP-SM-EI-05 | Compaction 与 correction 语义完整 | compact/restore 前后关系图、terminal selection、correction chain、workflow-health refs 语义差异为 0；非法 post-close 原位改写 100% rejected。 |
| MVP-SM-EI-06 | Audit report 与 dogfooding 可信 | machine audit report 逐维计数 100% 正确并携带 provenance/input hashes；CR-046 R1 null-provenance results 在 strict profile 下失败或标 legacy/unavailable。 |
| MVP-SM-GB-01 | 两仓 branch 起点 | 2/2 CR branch base 等于刷新后的 remote default tip，2/2 upstream 正确。 |
| MVP-SM-GB-02 | 发布边界 | dirty/uncommitted 场景 100% 阻断；合法发布后 2/2 remote CR ref 等于 local HEAD。 |
| MVP-SM-GB-03 | 删除安全 | non-ancestor、ref drift、protected/mismatched branch 删除执行数为 0；合法目标清理率 100%。 |
| MVP-SM-GB-04 | 可恢复 partial | 任一仓注入失败时 overall 非 PASS、逐仓 terminal status/OID/恢复路由覆盖率 100%。 |
| MVP-SM-GB-05 | Dry-run | open/publish/merge/finish 计划覆盖率 100%，local/remote ref、HEAD、index、worktree 变化数为 0。 |
| MVP-SM-GB-06 | 显式 merge | 2/2 remote default 仅以 fast-forward 更新到 exact published tip；merge commit/rebase/force/自动冲突解决为 0；partial 时 2/2 CR branch 保留且 finish 删除数为 0。 |
| MVP-SM-AW-01 | 路由唯一与兼容 | project-first/legacy fixture均按显式版本唯一解析；歧义写目标阻断率100%；设备绝对路径违规0。 |
| MVP-SM-AW-02 | worktree隔离 | 两项目fixture的idle integration与active CR branch均精确匹配命名契约；`main`作为项目working branch次数为0；branch/index/path crossover与index.lock争用均为0。 |
| MVP-SM-AW-03 | dirty与ownership门 | sibling dirty误阻断数0；current dirty、identity mismatch或ownership越界的mutation前阻断率100%。 |
| MVP-SM-AW-04 | 双leg聚合 | leg result/correlation覆盖率100%；聚合优先级组合100%可复算；仅全PASS完成；artifact main per-CR mutation、自动跨leg回滚和误报完成数均为0。 |
| MVP-SM-AW-05 | dry-run与可重入 | route/worktree/bootstrap/双leg/aggregate/migration plans重复执行结果确定；既有integration不被重建，文件/link/worktree/ref副作用0。 |
| MVP-SM-AW-06 | 零真实迁移 | CR-051结束时真实artifact文件搬迁、软链接变更、真实worktree/branch/ref mutation均为0；迁移manifest/checklist齐套。 |
| MVP-SM-MR-01 | External route安全可移植 | 合法v2 route跨目录前缀重放成功率100%；traversal/absolute/undeclared anchor接受数0。 |
| MVP-SM-MR-02 | Health模式诚实 | legacy/migrated/conflict矩阵判定一致率100%；route-conflict退出非零；legacy误报migrated数0。 |
| MVP-SM-MR-03 | Seed normalization精确 | actual prune集合=manifest allowlist；unexpected add/modify/delete=0；receipt前steady route激活数0。 |
| MVP-SM-MR-04 | Evidence跨进程恢复 | leg/aggregate opaque handle新进程readback成功率100%；篡改/错归属拒绝率100%；audit-tail OID可复核。 |
| MVP-SM-MR-05 | Bootstrap可续跑 | 每个intent/mutation/receipt故障点有唯一third-state；重复resume destructive mutation数0；CP0不伪造native-first。 |
| MVP-SM-MR-06 | Typed auth防重放 | 错action/object/OID/digest、过期、重复使用拒绝率100%；单token跨动作复用数0。 |
| MVP-SM-MR-07 | 临时真实E2E | 三并列临时拓扑规定路径terminal evidence覆盖率100%；partial误报PASS数0；fixture外mutation数0。 |
| MVP-SM-MR-08 | Scoped治理不新增历史噪声 | CR-052 scoped blocker=0；CR catalog/event-ledger 全局新增 error fingerprint=0；冻结的 19+1 历史对象原位改写数0。 |
| MVP-SM-MR-09 | CR-053 readiness完整 | handoff migration-critical能力/evidence引用覆盖率100%；缺任一关键引用时结论NOT_READY。 |
| MVP-SM-MR-10 | 真实对象零变化 | CR-052真实artifact文件/link/worktree/ref/publication变化数0；ptm-team/ptm-atomic/quant-lab内容读取/变化数0。 |
| MVP-SM-VNEXT-01 | 双库唯一 | 每个试点项目release route=1、process route=1、新增第三仓=0、归属错误=0。 |
| MVP-SM-VNEXT-02 | 跨项目零联动 | 两项目交替Work后sibling path/ref/index/state/hash变化数=0，共享过程working-tree/index/branch=0。 |
| MVP-SM-VNEXT-03 | 四层完整 | Project/Roadmap/Phase/Work 4/4层可解析；孤儿、多父、跳层、循环接受数=0。 |
| MVP-SM-VNEXT-04 | 投影克制 | 每Work最多更新1个Phase+1个Roadmap投影；Work全文复制数=0；完成证据覆盖100%。 |
| MVP-SM-VNEXT-05 | G0上限 | 每G0 Work≤8读、≤8写、≤3检查、≤32,000 token；高风险误入和静默超限=0。 |
| MVP-SM-VNEXT-06 | G1上限 | 每G1 Work≤20读、≤24写、≤8检查、≤96,000 token；高风险误入和静默超限=0。 |
| MVP-SM-VNEXT-07 | Scope执行 | scope外读/写/检查=0；扩展记录覆盖100%；telemetry status覆盖100%。 |
| MVP-SM-VNEXT-08 | Process-main CAS | 两并发同expected OID恰好1成功1stale；静默覆盖/自动merge/force=0；成功receipt覆盖100%。 |
| MVP-SM-VNEXT-09 | 快照与只读历史 | 当前快照覆盖100%；历史拆分/无损转换/双写/旧仓新增写入=0。 |
| MVP-SM-VNEXT-10 | 2×2试点 | 项目数=2、完整周期≥4、每项目G0≥1且G1≥1、跨项目变化=0。 |
| MVP-SM-VNEXT-11 | 回滚准备 | 每项目snapshot hash/pre-cutover OID/回滚目标/观察期字段覆盖100%；缺失时cutover=0。 |
| MVP-SM-VNEXT-12 | 未授权操作为零 | CP2阶段正式Story/LLD/源码/真实迁移/push/runtime/production执行数均=0。 |
