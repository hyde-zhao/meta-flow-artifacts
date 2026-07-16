---
checkpoint_id: "CP2-CR050"
checkpoint_name: "CR-050 Git Branch Lifecycle Product Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-15T15:49:55Z"
reviewed_by: "user"
reviewed_at: "2026-07-15T16:11:32Z"
auto_check_result: "process/checks/CP2-CR050-GIT-BRANCH-BASELINE.result.json"
context_ref: "process/context/CP2-CR050-GIT-BRANCH-CONTEXT.yaml"
decision_brief_profile: "compact"
target:
  phase: "requirement-clarification"
  artifacts: ["process/docs/product/USE-CASES.md", "process/docs/product/REQUIREMENTS.md", "process/docs/product/SCENARIOS.yaml", "process/docs/product/TEST-MATRIX.md", "process/docs/product/STORY-MAP.md", "process/docs/product/MVP-SCOPE.md", "process/docs/product/RELEASE-SLICES.md", "process/docs/product/BACKLOG.md"]
---

# CP2 CR-050 Git 分支生命周期产品基线人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR050-GIT-BRANCH-USE-CASE.result.json` | PASS | 0 | 三个 Use Case、十一场景与三 Story 候选完成追踪 |
| `process/checks/CP2-CR050-GIT-BRANCH-BASELINE.result.json` | PASS | 0 | 五项边界决策等待人工确认 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 为 CR 提供确定、可审计的 `open → publish → finish` 分支生命周期，覆盖源码仓和 artifact 仓。 |
| 推荐动作 | `approve`：接受 CP2-DQ-01..05 推荐方案，进入 CP3 架构设计。 |
| approve 后会发生什么 | 产出 HLD/ADR/蓝图与失败路由，并在 CP3 再次请求人工确认；仍不会实现代码或操作真实远端分支。 |
| approve 不授权什么 | 源码实现、stage/commit/push、真实 branch create/delete、auto merge、force/history rewrite、托管平台 API 与凭据读取。 |
| 不确认会阻塞什么 | CR-050 停留在 requirement-clarification；不得进入架构、Story 设计和实现。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule | `process/context/CP2-CR050-GIT-BRANCH-CONTEXT.yaml` |
| 状态 / profile | ready / compact |
| read_profile | compact |
| 默认读取策略 | capsule first；仅消费 `allowed_reads`，产品全文按需且须有 read-expansion 记录 |
| 全文档读取扩展 | 八个产品基线、CR-050 与 discussion log 的读取已记录到 `READ-EXPANSION-LEDGER.ndjson` |
| 产品范围 | UC-GB-001..003、TC-GB-001..011、ST-GB-001..003 |
| 实现证据 | 仓库已有 `git_sync.py` 原生 Git subprocess 封装和 bare-repo fixture；本阶段只读核验 |
| 外部契约 | Git 官方 `fetch/pull/switch/push/merge-base/branch`；Git Town 官方 `sync/propose/ship` 作为备选对照 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| state | `process/state/STATE.current.json` | scanned | 1 | 0 | 当前应停在 CP2，不形成额外产品选择 |
| inline handoff | `process/handoffs/CR050-CP1-CP2-META-PM.md` | scanned | 5 | 5 | 用户禁用子 Agent，Host inline fallback 已留证 |
| auto checks | CP1/CP2 result | scanned | 0 | 0 | blocker=0、waiver=0 |
| discussion | `process/discussions/CP2-CR050-SCENARIO-DISCUSSION-LOG.md` | scanned | 5 | 5 | SGA-GB-01..05 |
| formal artifacts | 八个 `process/docs/product/**` 文件 | scanned | 5 | 5 | 工具、双仓、merge、squash/rebase、commit 边界 |
| repository evidence | `git_sync.py`、相邻测试 | scanned | 1 | 1 | 已有封装可扩展，无新增依赖的必要性 |
| official evidence | `process/INPUT-INDEX.md` | scanned | 2 | 1 | 原生 Git 与 Git Town/gb 对照 |
| user input | 当前请求、既有 no-subagent 指令 | scanned | 2 | 0 | 目标与调度方式已确认，未替代 CP2 策略决策 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 5 | CP2-DQ-01..05 |
| 高风险策略确认 | 2 | CP2-DQ-03 远端写边界、CP2-DQ-04 删除证明 |
| agent 默认处理 | 4 | 命令结果 schema、错误消息、fixture 目录、文档示例细节 |
| Deferred | 3 | forge receipt adapter、squash/rebase cleanup、Git Town adapter |
| 仅审计记录 | 3 | no-subagent inline fallback、当前 `gb` 未安装、两仓远端已同步 |
| 明确禁止 | 6 | auto merge、implicit commit、force、history rewrite、猜测合并、未经证明删除 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP2-DQ-01 | architecture | 直接使用 Git，还是安装 `gb`/Git Town？ | 扩展现有 Python subprocess 边界，调用标准 Git CLI；不新增运行时依赖。 | A：Git Town adapter；B：把 `gb` 当环境别名。 | 推荐方案与现有 `git_sync.py`、零运行时依赖和 bare-repo fixture 一致；Git Town 功能丰富但增加安装/配置/版本契约；`gb` 名义不唯一。 | 错误工具契约会造成环境不可移植、行为漂移和额外供应链面。 | 若未来多仓 stacked branch、forge PR 编排成为已批准需求，再以可选 adapter 引入 Git Town。 |
| CP2-DQ-02 | scope | 只管理源码仓，还是源码 + artifact 两仓？ | 两仓使用同名 CR 分支并分别报告结果；不声称跨仓原子事务。 | 只管理源码仓，artifact 继续手工。 | 推荐方案保证代码与过程证据可跨设备恢复；代价是部分成功恢复更复杂。源码单仓简单。 | 单仓方案会重现证据漂移；双仓方案必须诚实处理 partial success。 | 若项目没有外置 process，repository discovery 只返回项目仓；不能静默跳过已配置 artifact 仓。 |
| CP2-DQ-03 | security | `finish` 是否自动 merge？ | 不 merge；只验证外部 merge 已完成，再删除 exact CR refs。 | A：本地 `git merge` 后 push；B：调用 forge merge API。 | 推荐方案不绕过 review、保护规则或 merge queue。自动 merge 便利但扩大授权并依赖平台策略。 | 自动 merge 可能绕过审查/保护；verify-only 要求用户或平台先完成 merge。 | 只有后续 CR 明确授权 merge、定义审批/保护契约和 receipt，才增加独立 merge 动作。 |
| CP2-DQ-04 | risk_acceptance | squash/rebase 后 Git ancestry 不成立时怎么办？ | fail closed：保留分支并给出需 platform receipt/follow-up 的结构化结果。 | A：patch-id/内容相似度猜测；B：已配置 forge receipt adapter。 | 推荐方案可能需要人工清理，但不会误删；patch 猜测无法证明审查对象与远端 tip 身份。 | 接受 squash/rebase 在 MVP 中不能自动清理的剩余风险，以避免误删远端 ref。 | 未来 adapter 能验证仓、PR、head SHA、base、merged 状态及时间时，可作为第二种证明策略。 |
| CP2-DQ-05 | implementation | `publish` 是否替用户 stage/commit？ | 只推送调用前已提交 refs；dirty/uncommitted 状态结构化报告，不自动提交。 | A：`git add -A` + 自动 commit；B：显式 path allowlist + 用户提供 message 的独立未来命令。 | 推荐方案权限最小且不会混入未选文件；用户需要先自行提交。A 风险过高；B 是额外产品能力。 | 隐式提交可能带入 secret、缓存、个人文件或无关变更，并模糊责任边界。 | 若后续需求明确要求自动提交，必须独立定义 path allowlist、消息、secret scan、dry-run 与人工确认。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 CP2-DQ-01..05 推荐方案 |
| 备选方案 | 可逐项回复 `修改: CP2-DQ-xx <你的要求>`；或 `reject` 停止当前 CR |
| 影响维度 | Git 工具契约、双仓恢复、远端写权限、删除安全、提交责任边界 |
| 优劣分析 | 见各 DQ 行 |
| 风险与回退 | CP3 前仍可回退 requirement-clarification；未批准不设计、不实现、不写远端 refs |
| 用户需决策事项 | CP2-DQ-01、CP2-DQ-02、CP2-DQ-03、CP2-DQ-04、CP2-DQ-05 |

### 推荐产品契约

1. `open`：所有目标仓先通过 precheck；`fetch --prune`；识别远端默认分支；本地默认分支 `pull --ff-only`；从精确远端 tip 创建同名 CR 分支；`push -u`。任何仓失败时停止后续破坏性动作。
2. `publish`：拒绝 protected/default branch；只 push 当前已提交 tip；不 stage、不 commit、不 amend、不 force。
3. `finish`：重新 fetch；验证 exact remote branch、记录/当前 tip 和 `merge-base --is-ancestor`；证明成功后按远端再本地顺序删除。远端已被平台自动删除时，缺失本身不是证明。
4. 所有动作支持 `--dry-run`，dry-run 的 refs 变化必须为 0；输出逐仓状态、已完成步骤、失败步骤与恢复提示。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP0/CP1 通过 | PASS | CP0/CP1 result |
| 产品基线齐套 | PASS | 八个 `process/docs/product/**` 文件 |
| Context ready | PASS | CP2 context capsule |
| 自动预检 | PASS | CP2 result；blockers=0 |

## Checklist

| # | 检查项 | 当前结果 | 证据 / 待审查意见 |
|---:|---|---|---|
| 1 | UC/REQ/TC/ST 可追溯 | PASS | TEST-MATRIX / STORY-MAP |
| 2 | In/Out/Deferred 明确 | PASS | MVP-SCOPE / BACKLOG |
| 3 | 五项 DQ 均有推荐、备选、风险和切换条件 | PASS | CP2-DQ-01..05 |
| 4 | 远端删除与跨仓部分成功 fail closed | PASS | REQ-GB / TC-GB / CR 风险表 |
| 5 | inline fallback 与不授权边界诚实 | PASS | handoff / dispatch ledger / CR-050 |

## Exit Criteria

| 条目 | 当前结果 | 证据 |
|---|---|---|
| 产品范围无 blocker | PASS | CP2 result |
| 工具与权限策略已决策 | PASS | 用户 `approve`；接受 CP2-DQ-01..05 推荐方案 |
| 人工审查完成 | PASS | 本文 `人工审查结果` |

## Deliverables

| 交付物 | 路径 | 结果 |
|---|---|---|
| 产品基线 | `process/docs/product/**` | ready |
| 自动结果 | `process/checks/CP2-CR050-GIT-BRANCH-BASELINE.result.json` | PASS |
| Context | `process/context/CP2-CR050-GIT-BRANCH-CONTEXT.yaml` | ready |
| Discussion | `process/discussions/CP2-CR050-SCENARIO-DISCUSSION-LOG.md` | complete |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-15T16:11:32Z
- 修改意见：无；接受 CP2-DQ-01..05 的推荐方案。
- 风险接受项：接受 squash/rebase 在 MVP 中无法仅靠 Git ancestry 自动证明、因此必须 fail closed 并保留分支的剩余风险。批准不授权代码实现、stage/commit/push、真实 branch create/delete、auto merge、force/history rewrite、托管平台 API 或凭据读取。
