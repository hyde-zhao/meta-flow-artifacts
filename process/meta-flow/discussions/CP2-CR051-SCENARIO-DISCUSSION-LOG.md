---
discussion_id: "CP2-CR051-SCENARIO"
checkpoint: "CP2"
cr_id: "CR-051"
phase: "requirement-clarification"
status: "completed"
owner: "meta-pm"
created_at: "2026-07-17"
updated_at: "2026-07-18"
checkpoint_ref: "process/checks/CP2-CR051-DISCUSSION-CHECKPOINT.json"
---

# CR-051 CP2 Scenario Gray Areas 讨论日志

## 用户真实意图

1. artifact采用project-first语义：`<project_name>/docs`与`<project_name>/process`。
2. 不同项目使用独立worktree，仍共享同一个artifact Git仓库和remote。
3. 当前先交付Meta Flow的路由、worktree管理和Git周期能力；真实文件迁移与软链接挂接后续逐项目执行。
4. 每项目worktree长期存在：空闲驻留`projects/<project-name>/integration`，每个CR使用`projects/<project-name>/cr/<cr-id>-<slug>`短期分支，`main`保留为共享集成基线。
5. 一个逻辑CR使用异构双leg：source leg从/回源码默认分支，artifact leg从/回项目integration；artifact CR不接触shared `main`。
6. 两leg独立出结果并由单一协调者聚合；shared `main`↔project integration同步完全在CR外人工维护，integration缺失时仅允许create-only初始化。

## 用户可见场景确认

### SGQ-AW-001：共享artifact的隔离模型

- 问题：多项目共享artifact时，是否改为project-first并为每项目提供独立worktree？
- 候选：
  - A（推荐）：`<project>/docs|process` + 每项目独立worktree；共享Git对象与remote。
  - B：继续`docs|process/<project>` + 单working tree；只靠目录隔离。
  - C：每项目独立artifact仓库；失去单仓共享和统一历史。
- 用户回答：选择“项目优先”，并要求artifact目录采用不同worktree管理。
- 复述确认：project identity成为artifact路由、worktree、branch和ownership的第一隔离键；branch还必须带project name，因为不同项目CR ID可重复。
- 影响面：scope、compatibility、validation、delivery、Git concurrency。
- 状态：`confirmed`。

### SGQ-AW-002：能力开发与真实迁移的切分

- 问题：本轮立即迁移所有artifact/软链接，还是先交付能力再逐项目迁移？
- 候选：
  - A（推荐）：先完成能力和fixture验证，后续逐项目迁移/挂链。
  - B：能力与全量迁移一次完成；回滚和跨项目污染面最大。
  - C：只写迁移文档，不实现worktree/Git能力；无法满足当前目标。
- 用户回答：先完成Meta Flow Git周期和artifact worktree能力，后续自己逐项目迁移文件和挂软链接。
- 复述确认：CR-051真实文件搬迁、软链接变化和真实worktree/branch/ref mutation均为0；只交付preflight/manifest/手册。
- 影响面：scope、authorization、rollback、delivery、follow-up tracking。
- 状态：`confirmed`。

### SGQ-AW-003：长期integration、短期CR branch与shared main的职责（R2历史，R3已修订）

- 问题：项目worktree空闲时、CR活动时和跨项目集成时分别应使用什么分支？
- 候选：
  - A（推荐）：每项目长期`projects/<project-name>/integration`；每CR短期`projects/<project-name>/cr/<cr-id>-<slug>`；`main`只作共享集成基线。
  - B：worktree空闲时detached在fresh `origin/main`。
  - C：长期worktree持续占用最近一次CR branch。
- 用户回答：选择A；同时接受DQ-02显式merge fresh `origin/main`和DQ-03 existing-control+sibling-worktree推荐。
- 复述确认：worktree idle/active角色已冻结；CR结束或中止后不得继续滞留已结束CR branch；`main`不被任何项目登记为owned/idle branch。精确attach/switch/finish/abort、OID与清理命令留给CP3。
- 影响面：branch lifecycle、软链接稳定性、多项目并发、失败恢复、结构化结果和验证fixture。
- 状态：`superseded-in-part-by-CP2-R3`；长期integration、短期CR和拓扑继续有效，per-CR merge-main语义由CP2-DQ-04替代。

## CP2 changes_requested R2 决策回填

| 决策ID | 用户回答 | 产品基线影响 | 状态 |
|---|---|---|---|
| CP2-DQ-01 | 每项目长期`projects/<project-name>/integration`；空闲驻留该分支；每CR使用`projects/<project-name>/cr/<cr-id>-<slug>`；`main`为共享集成基线 | 替换R1 idle detached候选；integration/CR/main职责写入UC、REQ、TC、Story、Slice和MVP | resolved-by-user |
| CP2-DQ-02 | 接受推荐 | 独立显式refresh把fresh `origin/main` merge到当前项目CR branch；冲突fail closed；不静默rebase/force/自动解冲突 | resolved-by-user |
| CP2-DQ-03 | 接受推荐 | 保留existing control checkout；项目worktree位于configurable sibling root；采用project namespace/sparse policy与owned-path gate | resolved-by-user |

R2仍保留的CP3细化项仅限：metadata/path schema、owned-path gate落点、worktree attach/switch/finish/abort机制、integration↔CR↔main的精确Git命令和expected OID、冲突abort/resume与branch cleanup、sparse行为。它们不是新的CP2产品决策。

## 关键灰区与状态（R3当前基线）

| 灰区ID | 问题 | 影响面 | 推荐 | 状态 / 路由 |
|---|---|---|---|---|
| SGA-AW-01 | project-first还是type-first | 路由、兼容、ownership、迁移 | project-first | resolved-by-user / SGQ-AW-001 |
| SGA-AW-02 | persistent还是ephemeral project worktree | 软链接稳定、清理、branch占用、恢复 | persistent；idle=`projects/<project>/integration`，active=`projects/<project>/cr/<cr-id>-<slug>`，`main`共享 | resolved-by-user / CP2-R2-DQ-01 |
| SGA-AW-03 | 异构双leg与整体完成门 | base/target、部分失败、跨仓回滚、完成语义 | source default leg + artifact integration leg；单一聚合优先级`BLOCKED > FAIL > IN_PROGRESS > PASS`，仅全PASS完成 | resolved-by-user / CP2-DQ-04..05；DQ-02 superseded |
| SGA-AW-04 | integration初始化与main同步边界 | 既有ref保护、OID、跨项目授权、恢复 | 缺失integration从fresh `origin/main` exact OID create-only；存在时不recreate/reset/orphan；main↔integration仅CR外人工同步 | resolved-by-user / CP2-DQ-03/06 |

## Deferred Ideas

| ID | 内容 | 原因 | 重启条件 |
|---|---|---|---|
| DEF-AW-001 | 全量自动迁移所有项目artifact和软链接 | 用户选择逐项目执行；一次性迁移风险过大 | 用户启动目标项目迁移CR并确认mapping/备份/回滚 |
| DEF-AW-002 | control repo立即转bare | 非能力MVP必需，会扩大迁移面 | existing clone方案出现不可接受branch占用/维护问题 |
| DEF-AW-003 | rebase+force-with-lease线性refresh | 改写project branch历史并扩大force授权 | merge-main历史成本经真实pilot证明不可接受并获新授权 |

## 阶段结论（R2历史）

- 用户可见SGQ交互：3条，前2条`confirmed`，R2新增1条`confirmed-for-CP2-R2`。
- BLOCKING信息缺口：0。
- 待CP2策略决策：0项；DQ-01..03均`resolved-by-user`。
- CP2总体门：R2产品基线仍待用户`approve`。
- 建议：R2产品基线因用户进一步修订保持changes_requested；由下方R3结论替代，不创建/发起门禁，也不授权CP3或实现。

## CP2 changes_requested R3 决策回填（当前生效）

用户在R2人工审查中进一步明确异构source/artifact生命周期、单一聚合门和integration bootstrap边界，并回复“按照这个方案，实施”。本次是对既有`SGQ-AW-003`的继续确认，不新增SGQ/SGA ID。

| 决策ID | 用户确认的当前产品基线 | 状态 |
|---|---|---|
| CP2-DQ-01 | idle=`projects/<project>/integration`；active=`projects/<project>/cr/<cr-id>-<slug>`；shared main保留跨项目基线职责 | resolved-by-user |
| CP2-DQ-02 | R2 per-CR merge fresh `origin/main`语义不再生效 | superseded-by-user / CP2-DQ-04 |
| CP2-DQ-03 | existing control checkout + configurable sibling worktree root + namespace/sparse/owned-path gate | resolved-by-user |
| CP2-DQ-04 | 一个逻辑CR使用异构legs：source从/回源码默认分支，artifact从/回项目integration；artifact CR不接触shared main | resolved-by-user |
| CP2-DQ-05 | 单一aggregate gate按`BLOCKED > FAIL > IN_PROGRESS > PASS`；仅全PASS完成；PARTIAL仅progress/effect；不自动回滚或关闭 | resolved-by-user |
| CP2-DQ-06 | integration缺失时从fresh `origin/main` exact OID create-only初始化；存在时不recreate/reset/orphan；main↔integration仅CR外人工维护 | resolved-by-user |

### R3 分类与授权边界

- pending decision items：0。
- `Q/OPEN/权限/风险/运行授权/publish`：全部分类；DQ-01/03/04/05/06=`resolved-by-user`，DQ-02=`superseded-by-user`，真实Git/remote/migration/main同步=`non-authorized`，CP3精确schema/状态机=`non-blocking-open`。
- main/integration divergence：不是单个CR blocker；artifact finish只以项目integration expected OID漂移作为阻断条件。
- 人工main→integration同步默认前置：该项目无活跃artifact CR；精确precheck与操作授权留CP3/后续独立运行授权。
- 当前用户可见SGQ交互仍为3条，SGA-AW仍为4条；没有新增ID。

### R3 阶段结论

- BLOCKING信息缺口：0。
- 待CP2策略决策：0。
- CP2总体门：R3产品基线等待Host Orchestrator生成并发起新的人工checkpoint。
- 本日志不创建人工checkpoint、不进入CP3、不拆Story、不授权实现或任何真实Git/worktree/link/remote mutation。
