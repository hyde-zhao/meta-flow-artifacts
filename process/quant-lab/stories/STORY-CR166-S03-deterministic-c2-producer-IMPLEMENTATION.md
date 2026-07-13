# CR166-S03 Implementation — Deterministic C2 Producer

## 实现前置检查

S01/S02 contract 与 validator 测试通过；producer 仅接收 `WalkForwardInputValidation`。

## 实现对象与设计契约映射

| 对象 | 文件 | 设计契约 | 验证 |
|---|---|---|---|
| metric/fold decision | `engine/walk_forward_oos_evidence.py` | RULE-166-07 | 调用方无直接 passed 真相入口 |
| declared denominator aggregate | 同上 | ADR-CR166-006 | 2/3 fixture pass_rate=2/3 |
| input/config/component/envelope identity | 同上 | ADR-CR166-007 | 10 次→1 hash、tamper blocked |

## 最小切片与结果

validated input → mandatory metric 重算 → fold outcome → declared denominator → component/envelope canonical identity → self-validation。非 validated input 不产生 present component；missing fold 不从分母消失。

## 边界与设计差异

current clock、物理 path、I/O、gate policy 和 admission threshold 均未进入 producer。无设计差异。

## CP7 评审回修

初轮 CP7 语义审查发现 self-validation 只校验 fold outcome 与 hash，未独立重算 `MetricDecision.passed` 和 aggregate outcome。已回到本 Story 增加 semantic re-evaluation；新增 tamper fixture 后目标/相邻回归 78/78、最终 full suite 1986/1986 通过。设计契约未变化。
