---
context_id: "CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24"
source_cr: "CR-137"
source_recent_crs:
  - "CR-128"
  - "CR-133"
  - "CR-134"
  - "CR-135"
  - "CR-136"
  - "CR-137"
recommended_next_cr: "CR-138"
recommended_next_action: "start-runner-qmt-operational-use-case-baseline"
status: "ready-for-context-reset"
created_at: "2026-06-24T11:08:09+08:00"
owner: "host-orchestrator"
read_profile: "minimal"
active_cr: null
source_repo_commit: "c9a87d0"
artifact_repo_commit_before_handoff: "5ec3428"
---

# CR137 Closure -> Runner / QMT Use-Case Context Reset Handoff

## 用途

本文件用于清空上下文后恢复 Runner / QMT 网关下一步规划。恢复后先读取本文件，再按 `process/state/STATE.current.json` 和本文件的最小读取集合恢复。

不要读取完整历史对话，不要从 CR128-CR137 的离线实现自动推导 online / QMT / MiniQMT / XtQuant / credential / trading 授权。当前任务重点已经从“继续补离线小功能”调整为“先冻结真实用户如何使用 Runner 与 QMT 网关的运营场景”。

## 0. 恢复后第一步

先运行只读 / 静态检查：

```bash
uv run --python 3.11 meta-flow check cr-tracking --project-root /home/hyde/workspace/quant-lab
uv run --python 3.11 meta-flow cr check --project-root /home/hyde/workspace/quant-lab
uv run --python 3.11 meta-flow state check --project-root /home/hyde/workspace/quant-lab
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_process_artifact_hygiene.py --json
git status --short --branch
git -C process status --short --branch
```

这些命令不读取凭据、不启动 runtime、不访问 NAS、不连接 QMT。

## 1. 当前真实状态

| 字段 | 当前值 |
|---|---|
| active formal CR | none |
| blocked formal CR | none |
| 刚关闭 CR | `CR-137 Offline Runner Run Registry` |
| 当前阶段 | delivered / context-reset-ready |
| 默认机器状态入口 | `process/state/STATE.current.json` |
| 当前 context handoff | `process/context/CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24.md` |
| runner 长期 feature authority | `process/docs/features/strategy-runner-core/DESIGN.md` |
| 已验证 CR137 回归 | CR137/136/135/134/133/128 + hygiene，`50 passed` |
| 仍未授权 | runtime / NAS / QMT / MiniQMT / XtQuant / credentials / provider / lake / catalog / trading |

`meta-flow check cr-tracking` 当前结果：

- active formal CRs: none
- blocked formal CRs: none
- follow-up / spike candidates 仅是 backlog，不占执行锁
- result: OK

`meta-flow cr check` 当前结果：

- `CR Lifecycle Check: OK`

## 2. 当前未提交工作区

源码仓库仍有 CR137 相关未提交改动：

- `scripts/check_process_artifact_hygiene.py`
- `tests/test_cr132_process_artifact_hygiene.py`
- `trading/strategy_runner/__init__.py`
- `trading/strategy_runner/cli.py`
- `trading/strategy_runner/run_spec.py`
- `trading/strategy_runner/runner.py`
- `tests/test_cr137_runner_run_registry.py`
- `trading/strategy_runner/run_registry.py`

artifact / process 仓库仍有 CR137 相关未提交改动：

- `changes/CR-INDEX.json`
- `changes/CR-INDEX.yaml`
- `docs/features/strategy-runner-core/*`
- `state/CR-LEDGER.ndjson`
- `state/STATE.current.json`
- `archive/CR-137/`
- `changes/CR-137-OFFLINE-RUNNER-RUN-REGISTRY-2026-06-23.md`
- `changes/summaries/CR-137.summary.json`
- CR137 CP6 / CP7 / CP8 checks, checkpoints, context, release docs and release context

用户尚未授权 `git add` / `git commit` / `git push`。恢复后不得假设这些改动已经提交。

## 3. 已关闭 CR137 结论

CR137 已由用户 `approve` 后关闭为 `READY_WITH_RISK`：

- 新增 `trading/strategy_runner/run_registry.py`
- `RunSpec` 增加 `run_registry_output_path`
- runner pass run 可追加 validated bundle registry entry
- runner blocked run 可登记 diagnostic entry，不链接 pass bundle
- CLI 增加 registry output / append bundle / list / inspect
- registry 不复制完整 `RunResult`、target portfolio、order intents 或 raw evidence
- registry / bundle / output 路径中的 `.env`、token、secret、credential、credentials fail closed

已通过验证：

```bash
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_cr132_process_artifact_hygiene.py tests/test_cr137_runner_run_registry.py tests/test_cr136_runner_bundle_validation.py tests/test_cr135_runner_artifact_bundle.py tests/test_cr134_runner_evidence_index.py tests/test_cr133_runner_spec_cli.py tests/test_cr128_runner_core_mvp.py
# PASS, 50 passed

PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_process_artifact_hygiene.py --json
# PASS

uv run --python 3.11 meta-flow cr check --project-root /home/hyde/workspace/quant-lab
# PASS

uv run --python 3.11 meta-flow check cr-tracking --project-root /home/hyde/workspace/quant-lab
# PASS, active formal CRs: none
```

## 4. 本轮用户纠偏结论

用户明确纠正了 Runner 规划方式：

1. CR 拆分不能继续按小能力碎片推进，应按大的功能模型拆。
2. use-case 不是 `RunSpec`、bundle、registry、validation 这些规格或设计约束。
3. use-case 必须描述用户如何使用 Runner。
4. online use-case 不能只写检查 / 准入 / smoke，还必须覆盖真实运行场景：
   - 多因子策略怎么运行
   - 日常怎么检查和运维
   - 事件型策略怎么运行
   - 日志怎么检查和运维
   - 策略如何调整、发布、暂停、回滚
5. 用户提供两份 QMT 生态研究报告，要求参考后重新整理 Runner 和 QMT 网关用户场景。

## 5. 参考输入

恢复后如需继续 Runner/QMT use-case 规划，优先读取以下输入：

1. `process/context/CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24.md`
2. `process/state/STATE.current.json`
3. `process/release/RELEASE-CONTEXT-CR137.yaml`
4. `process/docs/features/strategy-runner-core/DESIGN.md`
5. `process/docs/features/strategy-runner-core/TEST-PLAN.md`
6. `process/docs/features/strategy-runner-core/TASKS.md`
7. `/home/hyde/workspace/llm-wiki/llm-wiki/work/studies/quant-trading/sources/platforms/qmt/lite-qmt-executor、BulletTrade、EasyXT 源码深度研究报告.md`
8. `/home/hyde/workspace/llm-wiki/llm-wiki/work/studies/quant-trading/sources/platforms/qmt/quant-qmt-proxy 与 qmt-bridge 深度研究报告.md`
9. `process/USE-CASES.md`，仅用于读取 UC-28..UC-32 的既有双目标策略交付框架基线
10. `process/docs/design/BLUEPRINT.md`，仅用于读取 FEAT-05 / FEAT-06 / FEAT-09 的边界

默认不要读取：

- 完整历史对话 transcript
- 全量旧 CR 正文
- `process/STATE.md`，除非 state v2 缺失或人工审计
- `.env`、token、secret、credential、account、session、cookie
- NAS 目录
- 原始 QMT 账户、资金、持仓、委托、成交或日志

## 6. 研究报告提炼结论

### Runner 侧启发

来自 `lite-qmt-executor` / `BulletTrade` / `EasyXT`：

- Runner 不应只是离线检查器，而应被理解为策略运行控制面。
- `lite-qmt-executor` 强在信号接入、任务状态机、WAL 恢复、订单跟踪和柜台对账。
- `BulletTrade` 强在统一 runtime、`Context/Portfolio`、事件总线、订单队列、风控和 QMT guard。
- `EasyXT` 强在实际策略样本，包括多因子 live、双均线、条件单、网格、跟单、轮询策略。
- 多因子策略、事件型策略、组合再平衡策略、执行型策略对运行时要求不同，不能只用一个 bundle/registry 检查模型覆盖。

### QMT 网关侧启发

来自 `quant-qmt-proxy` / `qmt-bridge`：

- QMT 网关不是普通 HTTP 包装器，而是服务化运行层。
- `quant-qmt-proxy` 值得参考的是 REST/gRPC/WebSocket、多环境、会话和订阅的协议分层。
- `qmt-bridge` 值得参考的是服务生命周期、交易管理器、通知、WebSocket、独立调度器、xtdata 串行化保护和常驻 QMT 运行边界。
- QMT / xtquant 的真实瓶颈在单机、内网、线程受限、客户端常驻、xtdata 并发和 QMT 状态损坏，不在 API 路由本身。
- 机构化网关需要统一交易入口、行情入口、会话/账户编排、风控闸门、故障隔离、恢复、审计和可观测性。

## 7. 推荐下一步 CR

推荐启动：

```text
CR138 Runner & QMT Gateway Operational Use-Case Baseline
```

目标不是实现代码，而是冻结真实用户如何使用 Runner 和 QMT 网关：

- 多因子策略日常调仓运行
- 多因子策略盘前确认
- 多因子策略执行与跟踪
- 事件型策略触发运行
- 信号型策略接入执行
- 组合再平衡运行
- 盘中运行总览
- 单次 run 日志和证据查询
- 盘后复盘
- 异常处理与恢复
- 策略参数调整
- 当天运行计划人工修订
- 策略暂停、下线和回滚
- QMT 网关启动、健康、账户/持仓/订单/成交查询
- 行情订阅和缓存
- 下单、撤单、订单回报
- QMT 连接异常后的降级和恢复
- 网关审计、日志、升级、回滚和配置变更

CR138 不应直接做 online runtime 实现，也不应继续只补离线小功能。应先产出 use-case baseline，再进入功能模型、边界设计和可行性验证。

## 8. 推荐 use-case 版图

### Runner 用户场景

| 场景 | 用户怎么使用 Runner | 目标 |
|---|---|---|
| 多因子日常调仓 | 配置策略任务，Runner 定时读取数据、因子、股票池、持仓和风险规则，生成调仓计划。 | 形成可审查的目标组合和订单意图。 |
| 多因子盘前确认 | 用户检查调仓计划、QMT 网关状态、持仓、可交易性、风险和 kill switch。 | 决定今天 shadow / dry-run / simulation / blocked。 |
| 多因子执行跟踪 | 用户批准 run plan 后，Runner 跟踪 order intents 到订单状态。 | 看到 submitted / partial / filled / cancelled / rejected。 |
| 事件型策略运行 | 用户或系统提交公告、财报、价格突破、风控预警、人工事件。 | Runner 匹配规则并生成建议动作。 |
| 信号型策略接入 | 外部信号源向 Runner 投递信号。 | Runner 去重、幂等、检查未完成单并生成任务。 |
| 组合再平衡 | Runner 基于目标组合和当前组合计算净买卖。 | 处理现金、最小交易单位、部分成交和未完成单。 |
| 日常运维 | 用户查看策略、run、事件、订单、队列、QMT 状态和告警。 | 识别异常、等待确认和人工接管项。 |
| 日志审计 | 用户按 run_id / strategy_id / order_intent_id 查看执行证据。 | 复盘为什么生成某动作。 |
| 策略调整 | 用户修改参数、因子、风险阈值或策略版本。 | 生成新版本、差异、验证和发布/回滚决策。 |

### QMT 网关用户场景

| 场景 | 用户怎么使用 QMT 网关 | 目标 |
|---|---|---|
| 启动网关 | 运维在 Windows QMT 机器启动 gateway。 | 确认 QMT 客户端、API key、监听地址、health。 |
| 能力探测 | Runner 调用 health / capabilities。 | 判断可只读、可交易、degraded 或 unavailable。 |
| 账户查询 | Runner 查询账户、持仓、订单、成交。 | 支撑盘前确认、盘中跟踪和盘后对账。 |
| 行情订阅 | Runner 订阅 tick/bar 或查询历史数据。 | 支撑实时策略和事件策略。 |
| 下单撤单 | Runner 提交标准化 order intent，网关执行 broker adapter。 | 统一下单、撤单、回报和拒单。 |
| 交易事件流 | 网关推送订单、成交、持仓、连接状态。 | Runner 不靠散乱轮询猜状态。 |
| 故障恢复 | QMT 断连、xtdata 崩溃、订阅失效时进入 degraded/cooldown。 | 快速失败、只读降级、恢复订阅和接管未完成单。 |
| 审计日志 | 运维按 request_id / strategy_id / broker_order_id 查询日志。 | 全链路追踪与问题定位。 |

## 9. Runner / QMT 网关边界

| 边界 | Runner | QMT 网关 |
|---|---|---|
| 主要职责 | 策略如何运行、如何调整、如何观察和如何复盘。 | 如何安全稳定触达 QMT / MiniQMT / xtquant。 |
| 核心对象 | StrategyRun、StrategyTask、Context、Portfolio、OrderIntent、RunLog。 | TradingSession、AccountProfile、BrokerOrder、ExecutionReport、GatewayHealth。 |
| 对用户输出 | 调仓计划、事件处理结果、订单意图、运行状态、复盘报告。 | 行情、账户、持仓、订单、成交、健康状态、审计事件。 |
| 不应负责 | 直接暴露 xtquant 原语或绕过风控下单。 | 理解策略逻辑、决定调仓或修改策略参数。 |

一句话恢复原则：

```text
Runner 负责“策略如何运行”；QMT 网关负责“如何安全稳定地触达 QMT”。
```

## 10. 明确不授权

清空上下文后仍然不授权以下动作：

- 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不把 CR137 或此前 offline runner 能力解释为 runtime-ready 或 trading-ready。

## 11. 恢复后的第一句话建议

```text
我已读取 process/context/CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24.md。当前无 active formal CR；CR137 已关闭为 READY_WITH_RISK，但源码和 process 仍有未提交 CR137 改动。下一步推荐启动 CR138 Runner & QMT Gateway Operational Use-Case Baseline，先冻结真实用户如何使用 Runner 和 QMT 网关，再做功能模型、边界设计和可行性验证；仍不授权 runtime/NAS/QMT/credentials/provider/lake/catalog/trading。
```

## 12. 充分性结论

本 handoff 足以在清空上下文后恢复下一步规划。后续应优先启动 CR138 的 Runner / QMT 网关运营场景基线；除非用户明确另行授权，否则不得触碰任何外部运行域、数据域或交易域。
