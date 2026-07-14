---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-01"
---

# FEAT-168-01 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 定义 C3 contract、数值、hash、availability 和 10 类 fail-closed 的 static tests。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：给 producer orchestration、exact basis/net return 与 10 个精确 reason code 增加可判定断言。 |

## 测试范围

| Test ID | 覆盖 | 精确断言 |
|---|---|---|
| EC-T01 | 9-family happy path / entry orchestration | raw→normalize→validate(issues=0)→calculate(仅 1 次)→present；9/9 rules resolved；component=1；schema v1=1。 |
| EC-T02 | fee/tax/spread/slippage | fee=`traded_notional×fee_rate+fee_fixed_amount`；tax=`sell_notional×tax_rate+tax_fixed_amount`；spread/slippage 各为 `traded_notional×effective_rate`；impact 使用 traded notional；五分项差异=0。 |
| EC-T03 | square-root impact | proxy 0、1、>1 均有限可算；负值/NaN/Inf acceptance=0。 |
| EC-T04 | rounding | raw components 先 sum；total/net final quantize；逐项先舍入实现不得通过 golden。 |
| EC-T05 | minor unit | missing/zero/negative minor unit 3/3 blocked；default fallback=0。 |
| EC-T06 | 10-class negative matrix | N01..N10 精确映射 Domain §7 的 10 个 code；10/10 blocked/unavailable per contract；calculator invocations=0；false PASS=0。 |
| EC-T07 | hash determinism | 同 semantic projection 10 runs → distinct component hash=1。 |
| EC-T08 | A1 identity separation | 不同 attachment identity + 同 families 2-9 → same component hash；identity 不出现在 component body。 |
| EC-T09 | tamper | 成本字段/hash 篡改 100% blocked；unsupported/rebate present=0。 |
| EC-T10 | authorization | opaque refs zero dereference；forbidden operation counters 全 0。 |

`EC-T02/EC-T04` 还必须断言 `net_return=quantized_net_pnl/performance_notional`，`performance_notional` 缺失或非正时 blocked；不存在任何 implied performance basis 或 default currency fallback。

## 风险驱动与失败路径

| Risk | 测试方式 | 通过值 |
|---|---|---:|
| cost understatement | static assumptions/limitations/status pairing | missing limitation false PASS=0 |
| unit/currency drift | CNY/USD 无 conversion、calendar/price basis 混用 | 3/3 blocked |
| floating nondeterminism | reject binary float/nonfinite + 10 reruns | accepted float=0；hash distinct=1 |
| identity collision | daily/ML differing subject | component=1 hash；envelope由 FEAT-166-01 验证 distinct=2 |

只运行 repo-local `uv run --python 3.11 pytest ...`；不接真实数据或 external runtime。CP5 批准前本文件只定义计划，不执行新增测试。
