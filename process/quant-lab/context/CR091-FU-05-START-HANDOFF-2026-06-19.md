---
context_id: "CR091-FU-05-START-HANDOFF-2026-06-19"
candidate_id: "FU-CR091-005"
legacy_candidate_id: "CR091-FU-05"
source_tracking: "process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md"
status: "superseded-by-cr101-closed"
created_at: "2026-06-19T19:35:00+08:00"
updated_at: "2026-06-20T10:45:00+08:00"
owner: "host-orchestrator"
read_profile: "minimal"
formal_cr_created: true
formal_cr_path: "process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md"
superseded_by: "CR-101"
superseded_context: "process/context/CR101-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-20.md"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
cr_index_schema_version: 2
validation_status: "superseded; do_not_use_as_current_start_entry"
---

# FU-CR091-005 启动交接：跨平台策略交付与适配层重对齐

## 0. 清上下文恢复入口

本文件是历史启动交接，已被 `CR-101` 正式承接并关闭。清除上下文后不要再从本文件启动 `FU-CR091-005`，应改读 `process/context/CR101-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-20.md`。

第一步必须运行：

```bash
uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab
```

随后读取最小上下文：

1. `process/context/CR101-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-20.md`
2. `process/STATE.md`
3. `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml`
4. `process/changes/CR-INDEX.yaml`
5. `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md`

恢复后需要再次运行：

```bash
uv run --python 3.11 meta-flow check cr-tracking --project-root /home/hyde/workspace/quant-lab --strict-warnings
```

当前最后一次校验结果：

- `workspace check`: `process_link_health: ok`
- `cr-tracking --strict-warnings`: `OK`
- 本文件状态：`superseded-by-cr101-closed`

## 1. 当前状态

- 当前无 active formal CR。
- `CR-101` 已关闭为 `closed-current-delivery / READY_WITH_RISK`。
- 本文件只作为历史审计入口；不得据此重新启动 `FU-CR091-005`。
- 当前 CR 管理已迁移为 schema v2。`FU-CR091-005` 已由 `CR-101` 承接并关闭，历史别名是 `CR091-FU-05`。
- 当前需求基线是 `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml`；若旧文档与该基线冲突，以当前需求基线和最新 `CR-INDEX.yaml` 为准。

## 2. 用户最新架构决策

| 决策 | 结论 | 影响 |
|---|---|---|
| runner 归属 | runner 是 quant-lab 的 runner，不是 MiniQMT runner | `strategy_runner` 不能直接绑定 MiniQMT；必须通过 adapter protocol 接外部接口 |
| 当前接口适配 | 当前只适配 MiniQMT gateway | MiniQMT 是首个 adapter target，后续可扩展到掘金量化等接口 |
| 策略交付目标 | 策略交付按跨平台 target 模型设计，当前仅支持 QMT | manifest / entrypoint / validation / package layout 必须保留 target 扩展位 |

## 3. 推荐正式 CR

| 字段 | 推荐值 |
|---|---|
| 候选编号 | `FU-CR091-005` |
| 历史别名 | `CR091-FU-05` |
| 推荐标题 | `Cross-platform Strategy Delivery and Adapter Layer Realignment` |
| workflow_mode | `standard` |
| impact_level | `medium-high` |
| rollback_to | `CR091 closed-current-delivery / CR100 closed-current-delivery / CR046 historical framework-first baseline` |
| 启动前置 | 先做 CR 冲突预检，再分配下一个正式 CR ID；不得假定一定是 CR101 |

## 4. 推荐范围

| 范围项 | 推荐处理 |
|---|---|
| target taxonomy | 定义 `delivery_targets[]`，当前 implemented 仅 `qmt_terminal_direct`，future/deferred 可保留但不实现 |
| strategy core | 平台无关，不 import QMT / MiniQMT / XtQuant / 掘金 SDK |
| QMT direct-run target | 当前唯一策略交付 target；提供 entrypoint / contract / smoke checklist / 脱敏 evidence |
| quant-lab runner adapter layer | 定义 runner -> broker/execution adapter protocol；runner 只依赖 protocol |
| MiniQMT gateway adapter | 当前唯一 runner adapter；只作为 API gateway adapter，不承载策略运行 |
| Goldminer / other adapters | 只保留 future extension point，不实现、不运行、不安装 |
| manifest / checker | 从 `miniqmt_runner` 强制项改为 target/adapter 可扩展 schema；校验当前 QMT target 和 MiniQMT adapter contract |
| evidence redaction | 修复否定性审计字段误触发风险，例如 `account` 字段名白名单或安全改名 |
| package generation | 重新生成本地验证包；不再默认包含 `targets/miniqmt_runner/entry.py` |

## 5. 推荐后续 CR 顺序

| 顺序 | 候选 | 目标 | 前置 / 说明 |
|---:|---|---|---|
| 1 | `FU-CR091-005` / legacy `CR091-FU-05` | 设计/schema/fixture/checker/包结构重对齐 | 已由 `CR-101` 关闭 |
| 2 | `RA-CR101-001` / legacy `QMT-DIRECT-RUN-VALIDATION-FU` | 用户在 QMT 环境手工验证 direct-run entrypoint | 需要逐 run 授权和脱敏 evidence |
| 3 | `RA-CR101-002` / legacy `MINIQMT-GATEWAY-ADAPTER-VALIDATION-FU` | quant-lab runner 通过 MiniQMT gateway adapter 做只读验证 | 不把策略放到 MiniQMT runner |
| 4 | `FU-CR101-001` / legacy `CR091-FU-03` | order-write / submit-cancel / simulation-live 设计门禁 | 等 adapter protocol 和只读 evidence 稳定后再评审 |

## 6. 明确不授权

- 不访问、列取、读取、复制、写入、发布或删除真实 NAS。
- 不读取 Windows/Linux `.env`、NAS 凭据、token、secret、账号、账户、资金、持仓、委托、成交或原始日志。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不由 agent 代跑 QMT direct-run 或 MiniQMT gateway runtime 验证。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。

## 7. 上下文读取策略

### 7.1 启动时必读

- `process/context/CR091-FU-05-START-HANDOFF-2026-06-19.md`
- `process/STATE.md`
- `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml`
- `process/changes/CR-INDEX.yaml`
- `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md`

### 7.2 按需读取

仅在冲突预检、CP2 scope/risk/runtime-authorization Decision Brief、CP3 架构设计或 CP5 实现设计需要证据时读取：

- `process/changes/CR-046-FOLLOW-UP-TRACKING-2026-06-13.md`
- `process/changes/CR-098-FOLLOW-UP-TRACKING-2026-06-19.md`
- `process/changes/CR-098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-SMOKE-2026-06-19.md`
- `process/changes/CR-100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-GATE-2026-06-19.md`
- `process/context/CR091-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-18.md`
- `process/context/CP2-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml`
- `process/context/CP3-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml`
- `process/context/CP5-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml`
- `process/context/CP8-CR098-DELIVERY-CONTEXT.yaml`
- `process/context/CP2-CR100-NAS-PACKAGE-EXCHANGE-CONTEXT.yaml`
- `process/context/CP3-CR100-NAS-PACKAGE-EXCHANGE-CONTEXT.yaml`
- `process/context/CP5-CR100-NAS-PACKAGE-EXCHANGE-CONTEXT.yaml`
- `process/context/CP6-CR100-NAS-PACKAGE-EXCHANGE-CONTEXT.yaml`
- `process/context/CP7-CR100-NAS-PACKAGE-EXCHANGE-CONTEXT.yaml`
- `process/context/CP8-CR100-DELIVERY-CONTEXT.yaml`
- `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md`
- `docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md`
- `docs/qmt/CR100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-HLD.md`
- `trading/strategy_runner/__init__.py`
- `trading/strategy_runner/cache.py`
- `trading/strategy_runner/package_loader.py`
- `trading/strategy_runner/package_exchange.py`
- `trading/strategy_runner/readonly_gateway.py`
- `trading/strategy_runner/adapters.py`
- `trading/strategy_runner/evidence.py`
- `trading/strategy_runner/target_portfolio.py`

### 7.3 默认不要读取

- 不默认读取完整历史 `process/STATE.md` 中的旧 `agent_lifecycle.active_agents` 明细，除非需要审计子 agent 调度。
- 不默认读取全量 `process/stories/`、历史 LLD、历史 CP6/CP7 失败轮次或完整 transcript。
- 不默认读取 Windows/Linux `.env`、凭据、token、secret、账号、账户、资金、持仓、委托、成交或原始日志。
- 不访问、列取、读取、复制、写入、挂载、发布或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish。
- 不把 CP2 设计通过解释为运行授权；真实 NAS、QMT/MiniQMT/XtQuant/gateway、simulation/live 或 publish 类动作必须另行逐 run 授权。

## 8. 启动前检查命令

启动正式 CR 前至少运行：

```bash
uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab
uv run --python 3.11 meta-flow check cr-tracking --project-root /home/hyde/workspace/quant-lab --strict-warnings
```

如需分配正式 CR ID，先扫描 `process/changes/CR-*.md` 和 `process/changes/CR-INDEX.yaml`，不要假定下一个 ID 是 `CR101`。

## 9. 当前 CP2 决策草案

| 决策 ID | 类型 | 问题 | 推荐方案 | 备选方案 | 风险 / 回退 |
|---|---|---|---|---|---|
| DQ-FU-CR091-005-01 | scope | 是否启动跨平台策略交付与 adapter layer 重对齐？ | 启动 standard CR，仅做设计/schema/fixture/checker/包结构，不运行真实 runtime | 暂缓；只改文档；直接做 QMT 验证 | 推荐方案先修正架构边界；直接验证会延续 `miniqmt_runner` 混淆 |
| DQ-FU-CR091-005-02 | architecture | runner 与 MiniQMT 的关系如何冻结？ | runner 固定为 quant-lab-side；MiniQMT 是 adapter | MiniQMT runner-hosted；双 runner | 推荐方案可扩展到掘金等接口；runner-hosted 会耦合平台 |
| DQ-FU-CR091-005-03 | implementation | 当前策略交付 target 支持哪些平台？ | 当前 implemented 仅 QMT direct-run；manifest 保留 future targets | 同时实现 MiniQMT target；只做抽象不交付 QMT | 推荐方案符合“当前仅支持 QMT”；同时实现会扩大范围 |
| DQ-FU-CR091-005-04 | runtime_authorization | 本 CR 是否授权真实验证？ | 不授权；真实 QMT/MiniQMT 验证后续逐 run gate | 授权用户手工验证；授权 agent 代跑 | 推荐方案保持离线重对齐；真实验证需单独 evidence schema |

## 10. 充分性检查结论

本 handoff 仅作为历史审计证据。清上下文后应使用 `process/context/CR101-CLOSURE-CONTEXT-RESET-HANDOFF-2026-06-20.md`，不得以 `FU-CR091-005` 作为下一轮启动主键。

恢复后第一项交付应是 CP2 scope/risk/runtime-authorization Decision Brief，而不是直接实现、连接 NAS、连接 QMT/MiniQMT、读取凭据或生成真实运行证据。
