---
schema_version: "stage3_research_machine_handoff_context_v1"
project: "quant-lab"
workflow: "RUNNER-QMT-SIMULATION-MULTIFACTOR"
from_stage: "Stage 2 - multifactor research framework upgrade no-lake"
to_stage: "Stage 3 - production mature multifactor strategy on research machine"
status: "ready_for_research_machine_implementation"
created_at: "2026-06-26T15:30:00+08:00"
owner: "host-orchestrator"
runtime_authorization: "not_authorized"
data_lake_authorization_on_current_machine: "not_available"
gateway_authorization: "not_authorized"
small_live_live_authorization: "not_authorized"
---

# Stage 3 Research Machine Handoff

## 1. 交接结论

当前机器无法连接数据湖；Stage 3 将由用户在另一台可以连接数据湖的研究机上实施。

Stage 2 已完成 no-lake 桥接切片，具备进入 Stage 3 的框架入口：

- `engine/mature_multifactor_framework.py` 已提供 `StrategyTypeAdapterContract`、`SignalSet`、项目级 `StrategyCandidate`、`ResearchEvidenceIndex`、`PortfolioRiskPolicy`、`MatureAdmissionSupport`、`Stage2MatureFrameworkBundle` 和 `Stage3ResearchMachineHandoff`。
- CR030 `strategy_admission_package_v1` 与 CR039 `multifactor_strategy_admission_package_v1` 已通过 Stage 2 bridge 合同衔接。
- Stage 3 目标不是运行 fixture / 注入包样例，而是生产一条可解释、可审计、可扩展到真实股票池、可长期模拟盘观察的成熟多因子策略。

本交接文档是 Stage 3 在研究机上的默认入口。研究机实施前先读取本文档，再读取当前 STATE 和 Stage 2 证据。

## 2. 最小读取清单

| 类型 | 路径 | 用途 |
|---|---|---|
| 当前交接 | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STAGE3-RESEARCH-MACHINE-HANDOFF-2026-06-26.md` | Stage 3 入口和约束 |
| 机器状态 | `process/state/STATE.current.json` | 当前阶段、风险、下一步 |
| 人类状态摘要 | `process/STATE.md` | 恢复入口和关键 refs |
| 阶段目标 | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-PHASE-GOALS-AND-PLAN-2026-06-26.md` | 五阶段路线和 Stage 2 / Stage 3 边界 |
| Stage 2 检查 | `process/checks/STAGE2-MULTIFACTOR-FRAMEWORK-NO-LAKE-INITIAL-SLICE-2026-06-26.md` | Stage 2 bridge slice evidence |
| 研究组件说明 | `docs/components/MULTIFACTOR-RESEARCH.md` | 多因子研究合同和 Stage 3 handoff 说明 |
| Stage 2 框架代码 | `engine/mature_multifactor_framework.py` | 合同、builder、validator |
| Stage 2 测试 | `tests/test_stage2_mature_multifactor_framework.py` | 期望行为和 fixture 使用方式 |
| Stage 2 fixture | `tests/fixtures/stage2_multifactor_framework/mature_multifactor_stage2_fixture.json` | no-lake 样例，不得当真实研究结论 |

默认不要读取完整 runtime evidence、历史 trading-window r3/r4/r5/r6 operator evidence、私有 runtime overlay、raw account、raw order、gateway 日志或任何凭据文件。Stage 3 策略研究不需要这些材料。

## 3. 授权边界

| 操作 | 当前状态 | 规则 |
|---|---|---|
| 连接数据湖 | 仅在研究机 Stage 3 范围内执行 | 只能用于真实策略研究数据读取和 evidence 生成；必须记录 release、lineage 和输入版本 |
| provider fetch / lake read | 研究机允许按用户本地环境执行 | 不得在当前机器执行；研究机执行后必须记录数据源、时间、release 和过滤规则 |
| lake write / catalog publish | 未默认授权 | 如需写湖或 publish current pointer，必须另行形成明确数据发布门禁 |
| QMT gateway / simulation runtime | 未授权 | Stage 3 不启动 gateway，不提交 / 撤销订单，不查询账户 |
| `small_live` / `live` | 未授权 | 必须 Stage 5 独立 live switch CR 和人工门禁 |
| 凭据读取 | 不纳入交接 | 不得把 token、账号、cookie、密钥、`.env` 内容写入 evidence 或文档 |

Stage 3 产物可以作为 Stage 4 simulation observation 的输入，但不自动构成 simulation runtime authorization。Stage 4 仍必须逐次授权，并从 P0 health / identity 开始。

## 4. Stage 3 输入清单

研究机实施 Stage 3 时，必须将下列真实输入固化为可追溯引用。缺任一 P0 输入时，成熟策略准入必须 fail-closed 或 typed unavailable，不得伪造 pass。

| 输入 | 要求 | Evidence 字段建议 |
|---|---|---|
| `data_release_ref` | 指向真实数据湖 release / snapshot，不使用临时 current pointer 作为唯一真相 | `data_release_ref` |
| PIT universe | 按 decision date 可用的股票池，处理上市、退市、北交/沪深范围和指数成分可用时点 | `pit_universe_ref` |
| 上市退市 | 可证明每个交易日证券生命周期状态 | `listing_delisting_ref` |
| ST / 退市整理过滤 | 明确 ST、*ST、退市整理、风险警示处理规则 | `st_filter_ref` |
| 停牌过滤 | 明确停牌、长期停牌、临停和无可交易价格处理规则 | `suspension_filter_ref` |
| 涨跌停过滤 | 明确调仓日涨停买入、跌停卖出、封板不可成交处理 | `limit_up_down_filter_ref` |
| 流动性过滤 | 成交额、换手、ADV、最小可交易规模、容量约束 | `liquidity_filter_ref` |
| 行业分类 | 行业版本、变更日期、缺失处理 | `industry_classification_ref` |
| 市值数据 | 总市值 / 流通市值口径和可用时点 | `market_cap_ref` |
| 风格暴露 | beta、size、value、momentum、volatility 等暴露定义或替代代理 | `style_exposure_ref` |
| benchmark | HS300 / CSI500 / CSI1000 或策略基准，需记录版本和成分时点 | `benchmark_ref` |
| 费用滑点模型 | commission、tax、slippage、impact、最小交易单位 | `fee_slippage_model_ref` |

## 5. 必须生成的研究 evidence

Stage 3 的核心产出不是单一收益曲线，而是一组可审计 evidence index。建议输出目录按研究机本地约定选择，但 evidence ref 必须能回填到 `ResearchEvidenceIndex` 和 mature admission package。

| Evidence | 必须包含 | 失败规则 |
|---|---|---|
| run manifest | run_id、strategy_id、config_hash、data_release_ref、factor versions、code version、seed、date range | 缺任一 P0 字段不得进入准入包 |
| factor spec refs | 因子 ID、方向、输入字段、窗口、available_at、预处理、lineage、经济含义 | 因子方向或 available_at 不明时 blocked |
| factor panel | symbol、trade_date、factor_id、factor_value、available_at、quality_status、lineage | 缺 available_at 或 PIT lineage 时 blocked |
| label window | label horizon、start/end、available_at、复权口径、非重叠规则 | 标签泄漏风险时 blocked |
| IC / RankIC | 分窗口、滚动、市场阶段、行业/市值分组统计 | 单一全样本指标不能单独支持成熟准入 |
| 分层收益 | quantile / layered returns、long-short、成本前后 | 缺成本后表现时不得声明 simulation-ready |
| 换手与成本 | turnover、cost bps、capacity、impact proxy | 换手不可控时进入 watch 或 blocked |
| 暴露 | 行业、市值、风格、benchmark active exposure | 暴露无法解释时 blocked 或需要风险限额 |
| 组合版本 | signal version、portfolio version、rebalance rule、top_n、weight cap | 组合不可复现时 blocked |
| 风险版本 | risk policy version、stop conditions、capacity assumptions | 风控未版本化时 blocked |
| mature admission package | mature strategy admission package、blocked claims、unlock conditions | 不得声明 QMT-ready / live-ready |
| runner offline preflight | runner offline / plan-only / preflight-only import check | 只有导入检查通过后才能进入 Stage 4 候选 |

## 6. 推荐实施顺序

1. 在研究机确认仓库代码和 process artifact 已同步到同一版本。
2. 先运行 Stage 2 合同回归，确认研究机 Python 环境一致：

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_stage2_mature_multifactor_framework.py
```

3. 建立 Stage 3 run id，建议格式：

```text
stage3-mature-mf-YYYYMMDD-<short-purpose>
```

4. 冻结本次真实数据 release，记录 `data_release_ref`、可用日期范围、数据字典版本和质量检查结果。
5. 生成真实 PIT universe 和交易可行性过滤结果：ST、停牌、涨跌停、流动性、上市退市、行业、市值、风格。
6. 生产 FactorSpec / FactorRunSpec，并对每个因子记录输入字段、窗口、available_at、预处理、lineage 和经济含义。
7. 生成 factor panel 和 label window，先做 no-lookahead / label leakage 检查。
8. 运行单因子评价：IC / RankIC、分层收益、换手、成本、暴露、滚动和市场阶段稳定性。
9. 运行多因子组合：信号合成、权重约束、行业/风格/市值暴露控制、容量与成本约束。
10. 生成 `SignalSet`、项目级 `StrategyCandidate`、`ResearchEvidenceIndex`、`PortfolioRiskPolicy` 和 mature admission package。
11. 执行 runner offline / plan-only / preflight-only import check，只验证导入和计划，不触发 gateway / QMT / simulation。
12. 形成 Stage 4 simulation observation plan，包括调仓频率、观察指标、停止条件、异常分类、manual takeover 和回滚规则。

## 7. Stage 3 输出包要求

Stage 3 完成后，至少应回传以下摘要文件或等价 evidence refs，供 Stage 4 前置审查使用：

| 输出 | 内容 |
|---|---|
| Stage 3 run manifest | 本次策略研究 run 的机器可读 manifest |
| Mature strategy admission package | 真实数据策略准入包，不含 runtime authorization |
| ResearchEvidenceIndex | 所有关键 evidence ref、metric ref、lineage ref |
| SignalSet | 每次调仓信号集合，含 available_at 和 lineage |
| StrategyCandidate | 项目级策略候选，绑定 CR030 / CR039 / Stage 3 evidence |
| PortfolioRiskPolicy | top_n、max_weight、turnover、industry/style/capacity/stop conditions |
| Runner offline preflight report | runner 离线导入检查结果 |
| Stage 4 observation plan | 模拟盘观察周期、指标、停止条件和调整流程 |

所有输出必须脱敏：

- 不写 token、API key、交易密码、cookie、session。
- 不写 raw account、raw broker order id、raw fund / position 明细。
- 不写未脱敏私有路径；如必须引用路径，使用 root label 或 artifact ref。
- 不写 QMT-ready、simulation-ready、live-ready，除非后续 Stage 4 / Stage 5 门禁单独通过。

## 8. Stage 4 进入门槛

只有同时满足以下条件，才可以回到当前 runtime 机器准备 Stage 4：

| 条件 | 要求 |
|---|---|
| mature admission package | 已生成，且 blocked claims / unlock conditions 明确 |
| runner offline import | offline / plan-only / preflight-only 通过 |
| evidence index | 数据、因子、标签、评价、组合、风险、版本追溯齐全 |
| observation plan | 调仓、观察、停止条件、回滚、manual takeover 明确 |
| runtime authorization | 用户重新逐次授权 simulation runtime |
| gateway lifecycle | 用户在 Windows 手工启动 / 重启 gateway，并从 P0 health / identity 复验 |

Stage 4 不得复用历史 r3/r4/r5/r6 授权；不得把 Stage 3 研究机输出直接变成订单。

## 9. Context Policy

```yaml
context_policy:
  capsule_first: true
  context_ref: "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STAGE3-RESEARCH-MACHINE-HANDOFF-2026-06-26.md"
  read_profile: "compact"
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "Stage 3 handoff is a phase transition document for implementation on a different data-capable research machine."
  must_read:
    - "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STAGE3-RESEARCH-MACHINE-HANDOFF-2026-06-26.md"
    - "process/state/STATE.current.json"
    - "engine/mature_multifactor_framework.py"
    - "docs/components/MULTIFACTOR-RESEARCH.md"
  read_if_needed:
    - "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-PHASE-GOALS-AND-PLAN-2026-06-26.md"
    - "process/checks/STAGE2-MULTIFACTOR-FRAMEWORK-NO-LAKE-INITIAL-SLICE-2026-06-26.md"
    - "tests/test_stage2_mature_multifactor_framework.py"
    - "tests/fixtures/stage2_multifactor_framework/mature_multifactor_stage2_fixture.json"
  do_not_read_by_default:
    - "process/evidence/runner-simulation-runtime-2026-06-26/**"
    - "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-*-EVIDENCE-2026-06-26*.json"
    - "/home/hyde/.quant-lab/runtime/qmt/**"
    - ".env"
```

## 10. 当前禁止事项

- 不在当前机器尝试连接数据湖。
- 不启动 Windows gateway。
- 不执行 simulation runtime。
- 不触发 QMT / MiniQMT / XtQuant。
- 不把 fixture / 2 ETF 样例当作成熟多因子策略。
- 不把 Stage 3 研究机输出自动解释为 `small_live` / `live` 条件。
- 不把研究机本地凭据、账号、原始订单、原始持仓或私有路径写入仓库。
