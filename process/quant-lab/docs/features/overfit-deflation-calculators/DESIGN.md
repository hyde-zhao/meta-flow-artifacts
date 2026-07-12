# FEAT-26 PBO/CSCV + DSR Calculators — DESIGN

## 修订记录

| 版本 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| 0.1 | 2026-07-12 | host inline meta-se | 初始实现设计。 |

PBO/CSCV 接收稳定 candidate ids、time partitions、train/test ranking/loss definition，生成 split manifest、valid split count、logit/rank outcomes 与 PBO evidence。至少 4 candidates、4 valid splits；train/test 非空；split overlap/leak/duplicate 为 blocked。

DSR 接收 observed Sharpe、sample_length、skew、kurtosis、finite returns/moments 与 sealed raw trial count。schema 强制 `dsr_input_method=raw_trial_count`。`effective_trial_count=null`、ref/method empty、availability typed_unavailable；DSR method evidence 与 effective-count claim 分开。

两 calculator 均 pure/no I/O。精确 formula、tail、kurtosis convention、ranking ties 与 golden values由 S03 LLD 冻结并引用 primary source。

## Gotchas

- CSCV split 数量不能用无效 split 充数。
- raw trial count 是保守输入模式，不是 effective count 估计器。
- DSR present 不能自动解锁 CR154 C1 或 admission-ready。

