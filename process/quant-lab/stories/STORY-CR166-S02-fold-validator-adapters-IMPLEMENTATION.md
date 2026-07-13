# CR166-S02 Implementation — Fold Validator and Adapters

## 实现前置检查

S01 contract 已通过目标测试；CP5 LLD 已确认。只消费显式传入 value/ref，不解析路径或外部资源。

## 实现对象与设计契约映射

| 对象 | 文件 | 设计契约 | 验证 |
|---|---|---|---|
| common validator | `engine/walk_forward_oos_evidence.py` | RULE-166-01..06/13..16 | 7 字段族、8 类 fail-closed |
| daily / ML adapters | 同上 | ADR-CR166-004/005 | 2/2 compatibility fixture |
| event applicability | 同上 | ADR-CR166-008 | N/A reason、producer/feed=0 |

## 最小切片与结果

授权预检 → manifest/inventory → half-open temporal → purge/embargo → metric → lineage。缺失返回 `typed_unavailable`，矛盾、非法、未授权和非零操作返回 `blocked`。daily/ML 正向、event N/A、缺 fold/逆序/purge/embargo/metric/lineage/authorization 负向均通过。

## 边界与设计差异

未 import path/env/network/provider/runtime；external dereference=0。无设计差异。
