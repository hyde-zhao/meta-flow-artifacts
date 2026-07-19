# CP2 Scenario Discussion Log

## CR-036：审批者体验与 CR 拆分场景刷新

| 字段 | 内容 |
|---|---|
| 日期 | 2026-06-24 |
| 来源 | 用户反馈与当前对话 |
| 关联 CR | `process/changes/CR-036.md` |
| 场景基线 | `process/USE-CASES.md` v1.2 |

## 用户反馈摘要

用户反馈 meta-flow 当前 CR 有时拆分得非常细，人工确认项也跟着变细。用户难以判断某个 CR 对整体目标的作用，也难以判断待决策项对整体目标、风险、授权边界和后续路线的影响。

## 候选理解

| 候选 | 说明 | 处理结论 |
|---|---|---|
| A | 将 CR 和人工确认体验作为正式 use-case 刷新，补目标导向 CR 汇总、审批者摘要和风险分级确认场景。 | 用户接受，作为本轮主选 |
| B | 只优化人工确认文案，不新增 use-case。 | 放弃，无法支撑后续需求和设计追溯 |
| C | 保持现状，仅在 CR 模板中补字段。 | 放弃，不能解决用户在 use-case 汇总层面的目标感缺失 |

## Scenario Gray Areas

| 灰区 ID | 问题 | 处理结论 |
|---|---|---|
| SGA-CR036-01 | CR 应按工程对象拆分，还是按用户目标 / 风险 / 授权边界组织？ | 采用目标导向组织，工程对象作为下层证据 |
| SGA-CR036-02 | 哪些决策必须进入人工确认，哪些由 agent 默认处理？ | 采用决策分层，低风险可逆实现细节默认不进入用户主确认表 |
| SGA-CR036-03 | use-case 汇总展示完整细节还是目标影响摘要？ | 汇总页展示目标影响和体验变化，完整证据保留路径 |

## 用户可见确认记录

| Question ID | 问题 | 用户回答 | 复述确认 | 状态 |
|---|---|---|---|---|
| SGQ-CR036-001 | 是否接受将 CR 拆分和人工确认体验作为 meta-flow 的正式用户场景刷新？ | “可以接受你的建议，你先输出use-case即用户场景...” | 用户接受建议，要求先输出 / 刷新 use-case，并说明影响哪些 use-case 以及 use-case 汇总体验变化。 | confirmed |

## 下游衔接

- 正式场景基线已更新：`process/USE-CASES.md` v1.2。
- 本轮只完成 use-case 层刷新，不直接修改 human gate checker、CR 模板、Agent/Skill 或 delivery 产物。
- 后续若继续推进，应进入 requirement extraction 或 change impact refinement，围绕 UC-05 至 UC-07 形成需求与实现设计。

---

## CR-052：Migration Readiness Hardening 场景刷新

| 字段 | 内容 |
|---|---|
| 日期 | 2026-07-19 |
| 来源 | 用户原答“按照你的计划实施”、`process/REQUEST.md`、CR-052 summary/full ref 与八份既有产品基线 |
| 关联 CR | `process/changes/CR-052.md` |
| 场景基线 | `process/docs/product/USE-CASES.md` v2.0 |
| 委托 Agent | `meta-pm` / `/root/cr052_meta_pm` |
| 正式人工门 | 未发起；由 host-orchestrator 独占 |

### 用户真实意图

1. 把 CR-051 已有 project-first/worktree 库级契约补成可在真实拓扑安全执行、失败恢复和持久审计的 migration-ready 能力。
2. 保持能力建设与真实迁移分离：CR-052 只在临时三并列 Git 拓扑执行真实 mutation E2E，CR-053 才触碰真实 meta-flow artifact 布局。
3. 不读取或修改 ptm-team、ptm-atomic、quant-lab，不执行真实仓 ref/worktree/link mutation、shared-main 内容写入或 repository publication。

### 候选理解与取舍

| 候选 | 说明 | 优势 | 代价 / 风险 | 当前处理 |
|---|---|---|---|---|
| A（推荐） | CR-052 完成全部能力和临时真实 E2E；CR-053 执行真实布局迁移 | 授权、回滚、审计边界清晰；可在生产 mutation 前验证恢复 | 多一个正式 CR 和一次 handoff | 用户原答确认按计划启动；仍须 CP2 正式决策 |
| B | CR-052 同时建设能力并迁移真实布局 | 交付链更短 | 未验证能力直接作用真实仓，bootstrap/rollback/partial 风险混合 | 不纳入当前范围；仅用户重开 scope 才切换 |
| C | CR-052 只做 dry-run/mock，不做临时真实 mutation | 运行风险最低 | 无法证明 worktree/ref/link/filesystem/evidence 恢复，不能声称 migration-ready | 不推荐；若环境不允许真实临时测试则 CP8 必须 NOT_READY |

### Scenario Gray Areas

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 处理结论 | 分类 |
|---|---|---|---|---|---|
| SGA-MR-01 | 能力补齐与真实布局迁移是否拆为 CR-052 / CR-053 | 决定临时验证与真实仓 mutation 的授权、回滚、审计是否混杂 | scope / validation / authorization / rollback | 推荐拆分；用户原答确认启动该计划，但不是正式 CP2 批准 | decision-item / DQ52-1；SGQ-MR-001已确认场景基线 |
| SGA-MR-02 | external route 使用 workspace_parent 还是显式 workspace_root | 决定跨设备可移植性、可支持拓扑与配置错误面 | architecture / compatibility / validation / maintenance | 推荐 schema v2 + workspace_parent；非同父目录时切换显式 root | decision-item / DQ52-2 |
| SGA-MR-03 | durable evidence 使用两阶段 store + audit tail 还是纯 tracked store | 决定跨进程/跨机器恢复、dirty tail和OID自引用风险 | architecture / reliability / audit / gate | 推荐两阶段；CP3冻结durability和parent/target OID | decision-item / DQ52-4 |
| SGA-MR-04 | transitional bootstrap、seed prune和typed auth如何约束真实mutation | 决定首次迁移、删除范围和授权重放能否fail closed | security / failure-recovery / complexity / authorization | 推荐显式transitional CP0、manifest-bound prune、receipt后steady ownership和逐动作单次授权 | decision-item / DQ52-3/5/8/9 |

### 用户可见场景确认记录

| Question ID | 问题 | 候选选项 | 推荐 | 用户原答 | 复述确认 | 影响面 | 状态 |
|---|---|---|---|---|---|---|---|
| SGQ-MR-001 | 是否按整改计划把“迁移能力补齐与临时真实演练”放在 CR-052，把真实 meta-flow artifact 布局迁移放在 CR-053？ | A. 拆分且临时真实E2E；B. 合并能力与真实迁移；C. 仅dry-run | A | “按照你的计划实施” | 将原答解释为：允许启动CR-052、生成产品/设计/Story/检查点产物，并在临时仓执行受控真实mutation测试；不表示DQ52-1..10或CP2/CP3/CP5/CP8正式批准，不授权真实仓布局/link/worktree/ref mutation或push。除此之外没有伪造任何用户回答。 | scope / validation / authorization / gate / rollback | confirmed-for-scenario-baseline |

### DQ52-1..10 分类

| 决策 ID | 类型 | 分类 | 推荐产品语义 | 当前用户回答证据 |
|---|---|---|---|---|
| DQ52-1 | scope | decision-item | CR-052能力+临时真实E2E，CR-053真实迁移 | 仅有SGQ-MR-001计划启动确认；无CP2正式回答 |
| DQ52-2 | architecture | decision-item | schema v2 + workspace_parent，非同父目录时显式workspace_root | 未回答 |
| DQ52-3 | architecture | decision-item | CR-053显式transitional CP0并消费coordinator receipt | 未回答 |
| DQ52-4 | architecture | decision-item | out-of-band content-addressed evidence + project-local audit tail | 未回答 |
| DQ52-5 | implementation | decision-item | manifest-bound prune后激活steady ownership | 未回答 |
| DQ52-6 | runtime_authorization | decision-item | local legacy audit persistence继续；commit/push按exact OID另行授权 | 用户启动指令不授权commit/push |
| DQ52-7 | scope | decision-item | scoped CR check + frozen global fingerprint；历史修复follow-up | 未回答 |
| DQ52-8 | risk_acceptance | decision-item | 临时仓真实mutation E2E强制；真实仓mutation禁止 | SGQ确认计划方向；正式风险接受仍待CP2 |
| DQ52-9 | security | decision-item | ref/worktree/prune/link/push逐动作单次typed authorization | 未回答 |
| DQ52-10 | follow_up_tracking | decision-item | CR-053、shared README、其他项目迁移有owner/触发条件地follow-up | 未回答 |

> 所有灰区均已分类；没有阻断产品基线自动预检的“未分类”问题。10项 decision-item 仍必须由 host-orchestrator 在正式 CP2 人工门统一展示和收集，本文不发起审批。

### Deferred Ideas

| ID | 项目 | 延后原因 | 重启条件 |
|---|---|---|---|
| DEF-MR-001 | CR-053真实meta-flow artifact迁移/link swap | 与能力建设的授权/回滚边界不同 | CR-052 CP8 READY/READY_WITH_RISK并由用户正式启动CR-053 |
| DEF-MR-002 | 显式workspace_root非同父目录拓扑 | workspace_parent是当前最小契约 | 真实拓扑无法唯一解析或CP3验证需要 |
| DEF-MR-003 | shared-main README-only变更 | 当前不授权shared-main内容/publication | 独立README内容/ref/OID授权 |
| DEF-MR-004 | ptm-team/ptm-atomic/quant-lab迁移 | 当前明确禁止读取/修改 | 每个项目独立迁移CR |
| DEF-MR-005 | evidence跨机器共享/复制协议 | P0先保证cross-process durability | CR-053多设备恢复成为强制需求 |

### 下游衔接与不授权项

- 正式产品基线已增量更新：`process/docs/product/USE-CASES.md`、`REQUIREMENTS.md`、`SCENARIOS.yaml`、`TEST-MATRIX.md`、`STORY-MAP.md`、`MVP-SCOPE.md`、`RELEASE-SLICES.md`、`BACKLOG.md`。
- 本阶段不创建正式 CP2 checkpoint，不发起人工审批，不生成 CP4 Story/LLD，不修改源码或测试。
- 不授权真实 meta-flow artifact 文件/link/worktree/ref mutation、repository publication、shared-main 内容写入、force/tag/reset/rebase/orphan、credentials/runtime/production/publish/live/trading。
- ptm-team、ptm-atomic、quant-lab 不迁移、不读取、不修改。
