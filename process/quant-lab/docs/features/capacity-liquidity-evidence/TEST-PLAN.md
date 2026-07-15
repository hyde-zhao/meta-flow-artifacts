---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-01"
---

# FEAT-169-01 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 定义 C4 contract、proxy、hash、12 类 fail-closed 与 no-real-op 静态测试。 |

## 测试矩阵

| Test ID | 覆盖 | 精确断言 |
|---|---|---|
| CL-T01 | valid producer flow | normalize→validate(0 issues)→calculate(1)→produce；component/schema=1/1；3 refs present。 |
| CL-T02 | proxy math | participation、capacity amount、headroom 与公式逐项相等；Decimal precision=28。 |
| CL-T03 | quantization | capacity/headroom 最终按显式 minor unit HALF_EVEN；默认 minor unit=0。 |
| CL-T04 | cap boundary | ratio=cap 可通过；ratio>cap blocked；cap<=0 或 cap>1 全拒绝。 |
| CL-T05 | 12-class negative | N01..N12 各 1 个确定性 case；12/12 fail-closed；issue path calculator=0。 |
| CL-T06 | hash determinism | 同 normalized computational body 10 runs → distinct component hash=1。 |
| CL-T07 | identity separation | 同计算 body、不同 subject → component hash 相同；envelope hash 由 S03 断言不同。 |
| CL-T08 | tamper | body/ref/component/envelope hash 篡改 false PASS=0。 |
| CL-T09 | no-real claims | real ADV/liquidity/capacity-ready 全 false；外部 operation counters 全 0。 |
| CL-T10 | alpha absence | schema/body/output 中 alpha calculator/result 字段数=0。 |

仅允许 repository-local synthetic fixture 与 `uv run --python 3.11 pytest ...`；CP5 批准前不创建或执行新增测试。
