---
cr_id: "CR-091"
status: "active"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 QMT strategy runner、策略包消费、交易主机执行边界、运行授权、安全边界和未来下单接口扩展风险；不得使用 fast-lane。"
rollback_to: "CR091 blocked-research-plan-recorded / CR089 blocked-readiness-approved / CR046 closed-current-delivery recovery point"
approval_result: "approved / CP2-CP3-CP5"
created_at: "2026-06-18T00:28:26+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-18T14:16:02+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-089"
source_checkpoint: "current conversation"
source_decision_id: "USER-20260618-QMT-STRATEGY-RUNNER-RESEARCH-DESIGN-IMPLEMENTATION-PLAN"
follow_up_type: "strategy-runner-gate"
risk_class: "trading-runtime-boundary"
owner: "host-orchestrator"
revisit_condition: "用户已于 2026-06-18T14:16:02+08:00 批准 CR091 CP2/CP3/CP5；当前可进入离线实现切片。任何逐 run runtime authorization 仍需后续独立门禁。"
acceptance_criteria: "完成 runner 参考项目研究和风险矩阵，形成 HLD 推荐方案、LLD、TEST-PLAN、CP2/CP3/CP5 checkpoint 与待决策项；CP5 approve 后才允许实施离线 runner；本 CR 门禁不授权任何 QMT/MiniQMT/XtQuant/gateway/runner runtime、NAS、凭据、账户、下单或模拟/实盘动作。"
close_condition: "CR091 研究、方案、实现、验证和 CP8 均完成并由用户确认；或用户取消 / 合并到 CR089 / 拆分为后续 CR。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR091 QMT Strategy Runner Research / Design / Implementation Plan

## 变更描述

用户要求围绕 QMT strategy runner 制定新的 CR 计划，并明确采用“先研究、再确定方案、最后再实施”的路线。用户随后明确选择启动 CR091，并要求方案优先考虑多因子策略，同时具备其他策略的运行能力。本轮将 CR091 从已记录的 blocked plan 推进到 CP2/CP3/CP5 人工门禁：只完成静态研究、HLD、LLD、测试计划和待决策项，不进入实现或 runtime。

本 CR 不改变 CR046 / CR089 的状态事实：

| 对象 | 当前事实 | 对 CR091 的含义 |
|---|---|---|
| `CR-046` | 已于 2026-06-18T07:59:20+08:00 关闭为 `closed-current-delivery / READY_WITH_RISK` | CR091 可承接 runner 缺口，但不能把 CR046 framework-first 交付解释为 runtime ready |
| `CR-089` | `blocked-readiness-approved`，只读 `query_positions` smoke 和脱敏 collector 可作为经验输入 | CR091 可复用 CR089 的只读网关与脱敏证据模型，但不激活 CR089 runtime |
| `CR-020` | `closed-current-delivery`，用户删除的 QMT gateway 路线已归档 | 仅保留历史审计和当前 typed contract 参考，不得恢复为当前验证入口 |

因此，CR091 当前状态更新为 `active`：研究、HLD、LLD、测试计划和 CP2/CP3/CP5 门禁已由用户批准；下一步仅允许离线实现切片，runtime 仍未授权。

## 冲突预检结论

| 对象 | 影响面 | 冲突判断 | 处理方式 |
|---|---|---|---|
| `CR-046` | 双目标策略交付框架、策略包合同、MiniQMT runner 安装设计、验证框架 | 已关闭但语义重叠 | CR091 只承接 runner 缺口，不恢复 CR046 |
| `CR-089` | QMT 接口验证、交易主机取包、只读 query_positions smoke、运行授权 | 相关但仍 blocked | CR091 可复用 CR089 离线 package / redaction 模型，不自动启动 CR089 |
| `qmt_interface_smoke` package | 离线策略包骨架、manifest、checksum、脱敏 evidence 模板 | 可复用 | 只作为只读 runner 初始测试包输入，不授权 NAS 或 runtime |
| 外部 runner / gateway 项目 | runner、gateway、event engine、dry-run first 设计经验 | 可参考 | 仅研究借鉴，不直接照搬高风险 live/order 能力 |

结论：允许启动 CR091 静态研究和方案门禁；不得进入 runner 实现、NAS package pull/publish、gateway/QMT 启动或账户查询。CP5 未批准前不得写 runner 代码。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-091-QMT-STRATEGY-RUNNER-RESEARCH-DESIGN-IMPLEMENTATION-PLAN-2026-06-18.md` | 原文档更新 | 保留 plan-recorded 历史段，追加本轮启动事实和门禁批准事实 | 本文件 frontmatter 与章节 | approved-cp2-cp3-cp5 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR046 closed 与 CR089 blocked 事实 | CR091 entry | approved-cp2-cp3-cp5 |
| `process/STATE.md` | 原文档更新 | 保留当前 workflow state 与 CR046 closure history | `orchestrator_session` / `cr_tracking` | approved-cp2-cp3-cp5 |
| `docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md` | 新增 | 记录本轮静态研究，不替换上游 CR | 修订记录 | approved-cp2-cp3-cp5 |
| `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md` | 新增 | 形成推荐方案与备选方案 | 修订记录 | approved-cp2-cp3-cp5 |
| `process/stories/CR091-QMT-STRATEGY-RUNNER-LLD.md` | 新增 | CP5 实施准备输入 | 修订记录 | approved-for-offline-implementation |
| `docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md` | 新增 | CP5 测试准备输入 | 修订记录 | approved-cp2-cp3-cp5 |
| `process/checkpoints/CP2-CR091-QMT-STRATEGY-RUNNER-SCOPE-REVIEW.md` | 新增 | CP2 人工门禁 | 人工审查结果 | approved |
| `process/checkpoints/CP3-CR091-QMT-STRATEGY-RUNNER-HLD-REVIEW.md` | 新增 | HLD 人工门禁 | 人工审查结果 | approved |
| `process/checkpoints/CP5-CR091-QMT-STRATEGY-RUNNER-READINESS.md` | 新增 | LLD / TEST-PLAN / 实施边界人工门禁 | 人工审查结果 | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR046 StrategyPackageContract | CR091 runner package intake | 原文保留 | CR091 不改策略包合同，只研究 runner 如何消费 approved package |
| CR046 MiniQMT runner install design | CR091 local readonly runner | 原文保留 | CR091 先限制为交易主机本地 runner，不直接扩展到下单 |
| CR089 qmt_interface_smoke | CR091 readonly runner first package | 原文保留 | 只读 smoke 包可作为 runner 的首个 fixture / 手工包 |
| CR089 redacted collector | CR091 runtime evidence model | 原文保留 | CR091 若进入 runtime，只能输出脱敏摘要 |
| CR020 historical gateway code | CR091 gateway client reference | 审计保留 | 不恢复 CR020，不用 CR020 作为状态入口 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增或重定义需求 | QMT strategy runner、策略包消费、交易主机本地执行 | true | 新增 CR091 研究 / 方案 / 实施门禁需求，不改既有业务需求 |
| 场景层 | 是否改变测试矩阵覆盖范围 | runner package intake、manifest 校验、readonly gateway 调用、脱敏 evidence | true | 未来研究阶段输出场景矩阵；本轮只登记计划 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR046 / CR089 / CR091，未来 runner Story | true | 串行拆为研究、HLD、LLD、实现、runtime smoke、收尾 |
| 安全层 | 是否引入新的高风险动作或权限要求 | NAS、QMT/MiniQMT/XtQuant/gateway、`.env`、账户、订单接口 | true | 默认禁止；任何 runtime 必须逐 run 授权；订单接口独立 CR |
| 交付层 | 是否需要重新生成交付物或回归子集 | runner 研究报告、HLD、LLD、测试计划、只读 runner、collector | true | 未来按 CP2/CP3/CP5/CP6/CP7/CP8 输出，不在本轮实施 |

## 推荐总路线

推荐采用 clean-room lightweight runner，而不是直接改造外部项目或上来接入完整交易框架。

推荐边界：

- runner 运行在交易主机本地。
- runner 不直接 import `xtquant`，只调用本机 `127.0.0.1` gateway 的受控接口。
- 首个 runner 只支持 package intake、manifest/checksum 校验、runner health、gateway health 和 `query_positions` 只读调用。
- 策略包先进入本地不可变缓存，再由 active pointer 指向当前批准版本。
- 首个实现不支持 `submit_order`、`cancel_order`、simulation 或 live。
- NAS pull/publish 不进入 runner 默认职责，未来单独 CR 决策。

## 参考项目研究范围

| 项目 | URL | 可借鉴点 | 禁止直接照搬点 |
|---|---|---|---|
| lite-qmt-executor | `https://github.com/lotey/lite-qmt-executor` | QMT 执行器分层、策略插件、队列、WAL 恢复、HTTP/WS 信号入口 | 默认实盘买卖能力、自动拉起 QMT、真实账户配置 |
| qmt-gateway | `https://github.com/zillionare/qmt-gateway` | gateway 分离、HTTP/WS API surface、Windows QMT 服务化思路 | 账户/订单接口面过大、自动登录/密码相关能力、直接作为 runner 主体 |
| xqshare | `https://github.com/jasonhu/xqshare` | xtquant 远程执行 / 代理思路 | 作为主 runner 方案前必须重新做安全设计 |
| vnpy_qmt | `https://github.com/ruyisee/vnpy_qmt` | QMT adapter 映射和 Gateway 对接 | 不能把完整 vn.py 体系直接引入首版 runner |
| vn.py / VeighNa | `https://github.com/vnpy/vnpy` / `https://www.vnpy.com/` | 事件驱动、Gateway、策略生命周期、风控独立 | 首版过重，不能默认打开交易写接口 |
| freqtrade | `https://github.com/freqtrade/freqtrade` | dry-run first、配置驱动、策略接口、安全文档 | 面向 crypto，不能直接映射 A 股/QMT 账户模型 |
| QuantConnect Lean | `https://github.com/QuantConnect/Lean` | 模块化事件驱动 engine、backtest/live 命令分层 | 体量过大，不适合作为当前最小 runner |
| RQAlpha | `https://github.com/ricequant/rqalpha` | mod 扩展和研究/交易框架隔离思想 | 不是 QMT runner 主体，需避免引入过多依赖 |

## 阶段计划

### Phase 0: Startup Conflict / Preflight

目标：确认 CR091 可从历史 `blocked-research-plan-recorded` 迁移到本轮 CP2/CP3/CP5 人工门禁。

进入条件：

- 用户明确要求形成 CR 计划并落盘。
- `process` 路由健康检查通过。
- 已确认 CR046 已关闭为 `closed-current-delivery / READY_WITH_RISK`，CR089 仍是 `blocked-readiness-approved`。

退出条件：

- CR091 研究、HLD、LLD、测试计划和 CP2/CP3/CP5 checkpoint 创建。
- `CR-INDEX.yaml` 与 `STATE.md` 同步为 CR091 CP2/CP3/CP5 approved，进入离线实现准备态。
- 一致性检查通过。

本阶段不授权：

- 研究执行、HLD、LLD、代码实现、NAS、QMT/MiniQMT/XtQuant/gateway、`.env`、账户、订单接口。

### Phase 1: Research / Spike

目标：研究可借鉴 runner / gateway / engine 项目，形成可审计的参考矩阵。

未来输入：

- CR091 本文件。
- CR089 只读 smoke 记录。
- 本地 `trading/qmt_runtime.py`、`qmt_runtime_cli`、CR089 package checker 和 collector。
- 外部参考项目 README / docs / license / API surface。

未来输出：

- `docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md`
- runner reference matrix
- 风险清单与不采纳项
- clean-room 设计约束清单

退出条件：

- 研究矩阵覆盖 runner、gateway、event engine、dry-run/safety、QMT adapter 五类参考。
- 明确哪些设计可借鉴，哪些能力禁止进入首版。

### Phase 2: HLD / Solution Decision

目标：确定 runner 架构方案。

推荐方案：

- clean-room lightweight runner。
- 交易主机本地执行。
- runner 不直接连接 XtQuant，只调用本机 gateway。
- 默认只读：health、capabilities、query_positions。
- package intake 使用 manifest/checksum 和本地 immutable cache。
- active pointer 控制当前可运行策略包。
- evidence 只输出脱敏摘要。

候选备选：

- 改造 lite-qmt-executor。
- 基于 vn.py gateway / event engine 做完整 runner。
- 只保留手工脚本，不做 runner 模块。

未来输出：

- `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md`
- `process/checkpoints/CP3-CR091-QMT-STRATEGY-RUNNER-HLD-REVIEW.md`
- CP3 Decision Brief 和待人工决策清单

退出条件：

- 用户明确批准 HLD 推荐方案或指定备选。
- 未批准前不得进入 LLD 或实现。

### Phase 3: Implementation Design / LLD / Test Plan

目标：冻结接口、文件边界、测试策略和实现切片。

可能的首版文件边界：

- `trading/strategy_runner/package_loader.py`
- `trading/strategy_runner/manifest_validator.py`
- `trading/strategy_runner/active_pointer.py`
- `trading/strategy_runner/readonly_runner.py`
- `trading/strategy_runner/evidence.py`
- 或更小切片：`scripts/run_cr091_qmt_strategy_runner_readonly_smoke.py`

测试策略：

- fixture package，不读 NAS。
- mock gateway / fake client，不连接 QMT。
- 不读取 `.env`。
- 不 import `xtquant`。
- 禁止计数器必须全为 0：NAS、凭据、原始账户、原始持仓、下单、撤单、simulation、live、provider、lake、publish。
- 只允许脱敏 evidence。

未来输出：

- `process/stories/CR091-QMT-STRATEGY-RUNNER-LLD.md`
- `docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md`
- `process/checkpoints/CP5-CR091-QMT-STRATEGY-RUNNER-READINESS.md`

退出条件：

- CP5 统一人工确认 approved。
- 所有不授权项独立列出。
- 文件 owner 和测试矩阵明确。

### Phase 4: Implementation

目标：实现首版只读 runner。

允许范围，需未来 CP5 approved 后才生效：

- package manifest/checksum 校验。
- 本地 immutable cache 读取。
- active pointer 读取。
- gateway health / capabilities / query_positions 只读调用。
- redacted summary 输出。

不允许范围：

- NAS pull/publish/list/write/delete。
- `.env` 或凭据读取由 agent 执行。
- QMT/MiniQMT/XtQuant/gateway 启动。
- submit/cancel、simulation/live。

未来输出：

- 代码和测试。
- CP6 implementation evidence。
- 本地离线测试报告。

### Phase 5: Trading Host Manual Smoke

目标：在单独 runtime authorization 后，由用户在交易主机本机手工执行只读 runner smoke。

进入条件：

- CP6 / CP7 离线验证通过。
- 用户给出逐 run runtime authorization。
- 明确 scope，例如 `qmt:positions:read`。

只允许输出：

- 脱敏 summary。
- position count bucket。
- digest。
- redaction status。
- forbidden counters。

不得输出：

- 账户、持仓明细、资金、委托、成交、日志原文。
- client/account/nonce/signature 具体引用。
- `.env`、secret、token、session。

### Phase 6: Closure / Follow-up

目标：收敛只读 runner，拆出后续高风险 CR。

关闭条件：

- CP8 approved。
- 首版 runner 仍保持只读边界。
- 所有禁止动作计数为 0。

后续候选：

- `CR092-QMT-ORDER-WRITE-INTERFACE-DESIGN-GATE`：订单写接口设计门禁。
- `CR093-QMT-NAS-PACKAGE-PULL-PUBLISH-GATE`：NAS package 自动拉取 / 发布门禁。
- `CR094-QMT-SIMULATION-LIVE-RUNTIME-GATE`：模拟盘 / 实盘运行授权门禁。

## 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR091-01 | scope | CR091 当前是否启动为静态研究 / 方案 / 实施计划门禁？ | 启动 CR091 门禁，但 CP5 前不实现、不运行 | 暂停；直接 CR047；直接实现 | 推荐方案解决 runner 结构缺口且保留权限边界 | 不证明真实 runtime 可用 | 用户要求先交付策略包时切 CR047 |
| DQ-CP3-CR091-02 | architecture | runner 主体采用什么路线？ | clean-room package-driven runner | 改造 lite-qmt-executor；接入 qmt-gateway/xqshare；接入 vn.py；仅手工脚本 | 推荐方案最小依赖、边界清晰；外部项目实盘接口面过大 | 需要自行实现 package/cache/evidence | 未来复杂事件驱动需求出现时再评估 vn.py / Lean |
| DQ-CP3-CR091-03 | architecture | 多因子与其他策略如何兼容？ | 多因子优先，统一 StrategyAdapter 输出合同 | 多因子专用 runner；每类策略各自 runner | 推荐方案同时满足多因子优先和通用扩展 | adapter contract 需要稳定测试 | 若多因子字段不稳定，先降级 legacy adapter |
| DQ-CP5-CR091-04 | implementation | 首版产物形态是什么？ | `trading/strategy_runner` 薄模块 + 离线 checker + fixtures/tests | 只写脚本；直接完整 runtime | 推荐方案可复用且仍小范围；脚本方案扩展弱，runtime 方案越权 | 实现量高于单脚本 | 若范围过大，可先交付 adapters + evidence 子集 |
| DQ-CP5-CR091-05 | security | runner 是否直接从 NAS 运行策略？ | 不直接运行；只读本地 immutable cache / active pointer | NAS 原地运行；自动 pull/publish | 推荐方案避免 NAS 和发布边界；备选需新 CR | 后续需要单独处理包分发 | 用户要求 NAS 自动取包时启动 CR093 |
| DQ-CP5-CR091-06 | runtime_authorization | 首版是否允许真实 QMT runtime？ | 自动验证只允许 fake transport；真实只读 smoke 需后续逐 run 授权 | 立即连接 QMT；完全取消只读 smoke | 推荐方案保留验证路线但不越权 | 无真实 runtime ready 结论 | 用户给出逐 run 授权后再开 runtime gate |
| DQ-CP5-CR091-07 | follow_up_tracking | 下单 / 撤单是否纳入 CR091？ | 不纳入，拆为 CR092 候选 | 纳入 CR091 后半段 | 推荐方案隔离 order write 风险；纳入会扩大门禁复杂度 | CR091 不能验证下单链路 | 用户明确要求订单写验证时启动 CR092 |
| DQ-CP5-CR091-08 | risk_acceptance | 是否接受 CR091 首版不证明真实可交易？ | 接受；只证明 runner 合同和离线验证 | 要求真实交易机验证后才通过；取消 runner | 推荐方案符合当前权限；真实验证需要额外授权 | CP8 仍可能是 READY_WITH_RISK | 用户拒绝风险时暂停或新增 runtime gate |

## 订单接口明确排除

CR091 不验证下单接口。以下能力全部排除：

- `submit_order`
- `cancel_order`
- order status polling for write flow
- simulation submit
- live trading
- account raw query for order sizing
- fund / cash raw output
- fill / trade raw output

若用户后续要验证下单接口，必须新建独立高风险 CR，建议候选为 `CR092-QMT-ORDER-WRITE-INTERFACE-DESIGN-GATE`。该候选必须重新定义 order intent、pre-trade risk、kill switch、最小下单金额 / 数量、撤单语义、失败恢复、人工确认和逐 run authorization。

## 不授权项

本 CR 创建和落盘不授权以下任何动作：

- 不访问、列出、复制、拉取、发布、写入、删除 NAS 内容。
- 不读取 `.env`、账号、密码、token、cookie、session、QMT 凭据、NAS 凭据、交易密码。
- 不启动 QMT / MiniQMT / XtQuant / gateway。
- 不由 agent 执行账户、资金、持仓、委托、成交或日志查询。
- 不输出账户、持仓、资金、委托、成交或日志原文。
- 不执行 `submit_order`、`cancel_order`、simulation 或 live。
- 不做远程桌面、验证码、二次认证代操作。
- 不执行 Git commit / push / remote governance，除非用户另行授权。
- 不恢复 CR020，不恢复 CR046 CP7，不激活 CR089。

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：不适用
- 授权原文：无
- 回填要求：未来 CP8 必须人工确认，不得自动关闭。

## 处理结论

- 审批结论：`approved / CP2-CP3-CP5`
- [ ] 自动批准（低风险）
- [x] 人工确认通过（高风险）
- [ ] 待人工审批（高风险）

用户于 `2026-06-18T14:16:02+08:00` 回复“同意”，按 CP2/CP3/CP5 `approve` 处理，接受 DQ-CP2-CR091-01、DQ-CP3-CR091-02..03、DQ-CP5-CR091-04..08 的推荐方案。下一步允许进入 CR091 离线实现切片：`trading/strategy_runner` 薄模块、离线 checker、fixtures/tests、fake transport 和脱敏 evidence；仍不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env` / 凭据 / 账户、submit / cancel、simulation / live、provider / lake / publish。

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | `CR-046` | 上游 framework-first 交付已关闭，不恢复 |
| CR | `CR-089` | QMT 接口验证门禁，保持 blocked-readiness-approved |
| RUN-EXEC | `runs/RUN-EXEC-20260618-001.md` | CR089 redacted collector PASS，可作为只读 evidence 模型参考 |
| Package | `packages/qmt_interface_smoke/0.1.0` | 未来 runner 首个只读 package fixture / 手工包候选 |
| External reference | `https://github.com/lotey/lite-qmt-executor` | runner 设计参考，不直接照搬 |
| External reference | `https://github.com/zillionare/qmt-gateway` | gateway 设计参考，不直接照搬 |
