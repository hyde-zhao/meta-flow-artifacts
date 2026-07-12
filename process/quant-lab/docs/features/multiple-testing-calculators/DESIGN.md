# FEAT-25 BH + WRC/SPA Calculators — DESIGN

## 修订记录

| 版本 | 日期 | 修订人 | 变更 |
|---|---|---|---|
| 0.1 | 2026-07-12 | host inline meta-se | 初始实现设计。 |

BH 与 WRC/SPA 是无 I/O 的独立 calculator，输入只来自 FEAT-24 validated envelope，输出统一 MethodEvidence。BH 冻结 p-value ordering、adjusted q-values、alpha 和 membership hash；旧 anomaly helper 只能复用经验证的 BH primitive，不能复用其 normal-approximation来源或 Bonferroni/BH OR 最终标志作为 CR164 proof。

WRC/SPA 冻结 `stationary_bootstrap`、`block_length_mode=fixed_window`、positive integer window、explicit seed/replications/benchmark/null。automatic selector 不实现。return matrix 必须按 candidate/time 对齐、finite 且 candidate≥2。

失败：非法 p/domain、shape/hash冲突、window<1、缺 seed/config 为 blocked；单纯输入不足为 typed_unavailable。calculator 不读取文件、不写 consumer。

## Gotchas

- Reality Check 与 SPA 必须分别标识算法/version，不能合并成模糊 “WRC/SPA passed”。
- stationary bootstrap 来源不等于 automatic block selection 已定义。
- 排序/tie/tail 规则必须进入 S02 LLD 与 golden values。

