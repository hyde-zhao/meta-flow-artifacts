# FEAT-26 TEST-PLAN

| 方法 | 正向 | 失败 | 量化 |
|---|---|---|---|
| PBO/CSCV | >=4 candidates / >=4 valid splits | leak、duplicate、empty side、NaN | PBO in [0,1] |
| DSR | >=2 trials / sample>=30 / variance>0 | raw ref mismatch、effective alias、degenerate moments | finite DSR domain |

必须含 raw/effective non-alias fixture 和 consumer-requires-effective fixture。

