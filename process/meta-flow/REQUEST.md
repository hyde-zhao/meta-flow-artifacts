---
request_id: "REQ-meta-flow-retro-20260424"
submitted_at: "2026-04-24T09:56:41+08:00"
submitted_by: "user"
engagement_mode: meta-self-dev
scenario_subject_type: implementation-carrier
scenario_subject_id: "meta-flow"
---

# REQUEST：meta-flow 当前项目追溯补录

> 本文件根据当前仓库已有 HLD、Story、交付物和本轮会话上下文追溯补录，用于恢复当前项目在 `process/` 层的最小必要请求上下文。

## 用户目标

1. 将当前仓库沉淀为一个可持续演进的 **SCOPE-Pack 元工作流产物工厂**。
2. 让该工作流能够覆盖：场景发现、需求结构化、HLD、Story 拆解、LLD、实现、验证、文档与安装交付。
3. 使交付结果可同时面向 **GitHub Copilot / Claude Code / Codex / OpenClaw** 安装与运行。
4. 在不破坏生产模式默认行为的前提下，支持对 **meta 工作流自身**做结构化优化、整改和规则收敛。
5. 通过规则、Skill、Agent、模板和仓库护栏，减少路径漂移、命名漂移、资产归属错误和安装后不可执行等问题。

## 目标平台

- [x] GitHub Copilot
- [x] Claude Code
- [x] Codex
- [x] OpenClaw

## 交付预期

1. `delivery/` 下具备可安装的 Agent、Skill、rules、scripts、README 和 USER-MANUAL。
2. `process/` 下具备支撑工作流运行的 HLD、架构决策、Story Backlog、Development Plan、LLD 方法、验证与状态文档。
3. 关键运行契约通过正式文件显式表达，而不是依赖会话中的隐式推理。
4. 交付资产遵守输出隔离、Skill 私有资产同树安装、跨平台安装和 guardrail 检查约束。

## 补充约束

1. 当前请求属于 **meta 工作流自我开发 / 优化**，因此：
   - `engagement_mode = meta-self-dev`
   - `scenario_subject_type = implementation-carrier`
   - `scenario_subject_id = meta-flow`
2. Python 命令统一使用 `uv run --python 3.11 python ...` 作为仓库默认入口。
3. `delivery/scripts/` 仅保留安装器入口；Skill 运行时脚本和模板必须下沉到各自 Skill 目录。
4. Story 与 LLD 文档命名必须带上名称 slug：
   - `STORY-{id}-{story_slug}.md`
   - `STORY-{id}-{story_slug}-LLD.md`
5. 若用户未显式声明 meta 工作流优化，后续新请求默认仍应回到 `production`，而不是继续沿用当前模式。

## CR-050 增量请求（2026-07-15）

用户要求为 Meta Flow 实现 Git 分支管理：开启 CR 时从远端主分支拉取最新代码并创建 CR branch，提交后推送到远端 branch，在远端 branch 合并后删除 branch；同时要求分析应直接使用 Git 命令还是安装 `gb`。

本轮继续属于 `meta-self-dev`。原始请求确认了完整用户旅程，但没有授权 force、自动解决冲突、自动 merge、forge credential/API，也没有明确 `gb` 的具体实现。相关选择进入 CR-050 CP2 Decision Brief；CP2 前只允许产品基线与检查点产物，不允许源码实现或真实远端 ref mutation。

## CR-052 增量请求（2026-07-19）

用户要求按整改评审后的方案实施 Migration Readiness Hardening，并接受将能力补齐与真实迁移拆分为 CR-052 / CR-053：

1. CR-052 补齐 schema v2 external anchor、health mode detection、durable leg/aggregate evidence、transitional bootstrap coordinator、seed-prune/steady ownership、通用 migration runner、scoped CR check、三并列临时拓扑真实 mutation E2E、故障恢复、文档和 workflow eval。
2. CR-052 不迁移真实 meta-flow artifact 布局，不创建真实 integration/worktree/artifact CR ref，不修改 ptm-team、ptm-atomic 或 quant-lab。
3. CR-053 才执行真实 meta-flow 迁移；其 CP0 使用明确 transitional bootstrap，并且只调用 CR-052 已实现和验证的能力，不得在生产迁移时现场补核心执行逻辑。
4. 当前 `process` 仍写入 legacy `meta-flow-artifacts/process/meta-flow/**`，因此 CR-052 保留 legacy artifact audit persistence；是否 commit/push 必须在运行授权门按 exact OID 另行确认。
5. 临时仓/local bare remote 的真实 mutation 测试属于 CR-052 验证范围；真实仓 mutation、repository publication、shared-main 内容变更、force/tag/reset/rebase/orphan、凭据/runtime/production/publish/trading 不在本次启动授权内。

用户原文：“按照你的计划实施”。该指令授权启动 CR-052 和推进到正式人工门；不替代 CP2/CP3/CP5/CP8 的检查点审计，也不授权真实仓 migration 或 push。

## CR-052 vNext 重解释请求（2026-07-19）

用户进一步明确真实目标并要求按推荐方案开始实施：

1. 每个项目设置两个独立仓库：发布库管理项目发布相关代码和文档，过程库管理过程文档；发布库优先复用现有项目源码/交付仓，不新增第三个仓库。
2. 多项目并行时不再共享过程归档仓和 working tree，必须彻底消除“一个项目切分支导致其他项目看到的文档版本变化”。
3. 过程库必须支持大型项目长期治理，采用 `Project -> Roadmap -> Phase -> Work` 四层；Roadmap 和 Phase 是长期真相源，单次 Work 只回写必要投影。
4. 当前归档、文档读取和检查校验都过重，新的工作流按风险采用 G0/G1/G2，并对每个 Work 限定 read/write/check scope 和 token budget。
5. 总体取向是简单、可靠、后续治理成本低；允许不受现有 CR-051 接口和 artifact worktree 方案约束，旧体系只保留必要只读兼容。
6. 用户要求先为当前源码建立整改前 tag，再启动整改。已创建本地 annotated tag `pre-vnext-simplification-20260719`，指向 `435434bfde536edcf5eabae95985062e1060c02f`。

该请求将 CR-052 从“Migration Readiness Hardening”重解释为“Meta Flow vNext 简化治理整改”。原 CP2 尚未人工批准，因核心目标变化必须标记 `changes_requested` 并重新执行 CP1/CP2；不得把旧自动 `PASS` 当作新基线通过。
