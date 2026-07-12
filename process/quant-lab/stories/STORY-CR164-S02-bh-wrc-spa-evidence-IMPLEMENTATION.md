# S02 Implementation — BH / WRC / SPA

## 实现前置检查

S01 contract available；fixed-window ADR confirmed；automatic selector excluded。

## 实现对象与映射

`engine/multiple_testing_evidence.py` 实现 stable BH、seeded stationary-bootstrap indices、WRC/SPA pure calculators。`tests/research/test_multiple_testing_evidence.py` 覆盖 golden q-values、membership、fixed window、determinism、shape/config failures。Legacy anomaly OR flag 未被复用或修改。

## 结果与边界

Explicit seed/window/replications/benchmark/null 全进入 provenance；invalid config blocked；insufficient inputs unavailable。无真实 returns、I/O、external package 或 design delta。

