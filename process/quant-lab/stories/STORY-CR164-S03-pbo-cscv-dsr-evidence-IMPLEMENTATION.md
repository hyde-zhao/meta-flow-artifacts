# S03 Implementation — PBO / CSCV / DSR

## 实现前置检查

S01 contract available；raw-count/effective non-alias ADR confirmed。

## 实现对象与映射

`engine/overfit_evidence.py` 实现 stable CSCV split manifest/validation、PBO rank/logit aggregation、raw-count DSR probability和 hard non-alias checker。`tests/research/test_overfit_evidence.py` 覆盖 minima、split leak/duplicate、domain、count mismatch、degenerate moments与 effective ceiling。

## 结果与边界

DSR provenance 明确 `dsr_input_method=raw_trial_count`；effective value/ref/method 保持 empty/unavailable。无 estimator、real data、I/O 或 design delta。

