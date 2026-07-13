# CR166-S04 Implementation — Existing Consumer Projections

## 实现前置检查

S01–S03 self-validated component 可用；三个既有 consumer policy owner 保持不变。

## 实现对象与设计契约映射

| 对象 | 文件 | 设计契约 | 验证 |
|---|---|---|---|
| CR151 plan projection | `engine/walk_forward_oos_projections.py`、`engine/strategy_admission_statistical_gate.py` | ADR-CR166-007 | legacy validator + identity |
| CR154 Gate 2 projection | `engine/walk_forward_oos_projections.py`、`engine/cross_strategy_reliability_gates.py` | 同上 | Gate 2 PASS fixture + identity |
| package attach | `engine/strategy_admission_package.py` | RULE-166-11/12 | worse-state + 4 flags unchanged |

## 最小切片与结果

三条 projection 携带同一 component ref/hash/availability/outcome/reasons。tamper 三条全部 blocked；present+pass 不提升已有 blocked package；CR155 `paper_candidate=false` 保持。

## 边界与设计差异

未新增 gate、全局 admission enum、raw fold 重算或 runtime flag。无设计差异。

## CP7 评审回修

初轮 CP7 语义审查发现 CR151 compatibility projection 将显式 embargo 数值简化为 0，且 non-present component 的 trust 判定会把合法 `typed_unavailable` 过度收紧为 `blocked`。已回到本 Story：component 增加 leakage unit/purge/embargo applied facts，legacy plan 投影真实 day 值；projection 以结构完整性区分可信 non-present 与 tamper。新增 2 项回归后全量 1986/1986 通过。
