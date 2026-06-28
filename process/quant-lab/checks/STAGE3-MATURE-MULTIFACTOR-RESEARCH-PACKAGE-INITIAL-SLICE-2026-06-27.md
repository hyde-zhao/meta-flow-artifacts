---
check_id: "STAGE3-MATURE-MULTIFACTOR-RESEARCH-PACKAGE-INITIAL-SLICE-2026-06-27"
workflow: "RUNNER-QMT-SIMULATION-MULTIFACTOR"
stage: "Stage 3 - production mature multifactor strategy"
status: "PASS"
created_at: "2026-06-27T00:00:00+08:00"
owner: "host-orchestrator"
runtime_authorization: "not_authorized"
data_lake_runtime_on_current_machine: "not_executed"
gateway_authorization: "not_authorized"
small_live_live_authorization: "not_authorized"
---

# Stage 3 Mature Multifactor Research Package Initial Slice

## 1. 结论

Stage 3 已从 handoff-only 推进到初始研究包合同切片：

- 新增 Stage 3 run manifest 合同，用于记录 `run_id`、`strategy_id`、`config_hash`、`data_release_ref`、factor versions、code version、seed 和 date range。
- 新增 Stage 3 mature research package 合同，用于汇总真实 P0 输入 refs、research evidence refs、`SignalSet`、`StrategyCandidate`、`ResearchEvidenceIndex`、`PortfolioRiskPolicy`、mature admission package、runner offline preflight 和 Stage 4 observation plan。
- 新增 fail-closed 校验：空引用、`typed_unavailable:*`、`required`、Stage 2 fixture placeholder 或缺少关键 evidence 时，package 不得进入 Stage 4 审查候选。

本切片没有连接数据湖，没有读取凭据，没有启动 gateway，没有执行 runner simulation，也没有形成任何 `small_live` / `live` 授权。

## 2. 变更对象

| 路径 | 变更 |
|---|---|
| `engine/mature_multifactor_framework.py` | 新增 `Stage3ResearchRunManifest`、`Stage3MatureResearchPackage`、builder、validator 和 Stage 3 fail-closed refs 规则。 |
| `tests/test_stage2_mature_multifactor_framework.py` | 追加 Stage 3 package happy path、缺真实 refs blocked path、run manifest 必填项 blocked path。 |
| `docs/components/MULTIFACTOR-RESEARCH.md` | 补充 Stage 3 初始研究包合同入口和授权边界。 |

## 3. 验证

```bash
uv run --python 3.11 pytest -q tests/test_stage2_mature_multifactor_framework.py
```

结果：

```text
11 passed in 0.06s
```

## 4. 授权边界

| 项目 | 状态 |
|---|---|
| 数据湖读取 | 未在当前机器执行 |
| 数据湖写入 / catalog publish | 未授权 |
| QMT gateway / simulation runtime | 未授权、未执行 |
| 账户 / 订单 / 凭据读取 | 未授权、未执行 |
| `small_live` / `live` | 未授权 |

## 5. 下一步

在数据湖研究机上，用真实 `data_release_ref` 和 P0 输入 refs 填充 Stage 3 package：

1. 冻结真实数据 release / snapshot。
2. 生成 PIT universe、上市退市、ST、停牌、涨跌停、流动性、行业、市值、风格、benchmark、费用滑点 refs。
3. 生成 factor panel、label window、IC / RankIC、分层收益、换手、暴露、组合版本、风险版本、mature admission package、runner offline preflight refs。
4. 用 `validate_stage3_mature_research_package()` 验证 package 进入 Stage 4 审查候选。
