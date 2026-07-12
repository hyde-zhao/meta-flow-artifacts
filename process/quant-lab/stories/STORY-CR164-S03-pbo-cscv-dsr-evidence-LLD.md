---
story_id: "CR164-S03-pbo-cscv-dsr-evidence"
title: "PBO CSCV and raw-count DSR evidence"
story_slug: "pbo-cscv-dsr-evidence"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-12T20:25:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-12T20:50:00+08:00"
feature_design_refs: ["docs/features/overfit-deflation-calculators/DESIGN.md", "docs/features/overfit-deflation-calculators/TEST-PLAN.md", "docs/features/overfit-deflation-calculators/TASKS.md", "docs/features/statistical-evidence-contract/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["statistical-algorithm", "split-leakage", "raw-effective-non-alias"], rationale: "Claim-sensitive overfit/deflation evidence."}
open_items: 0
---

# LLD: CR164-S03 — PBO/CSCV and raw-count DSR evidence

## 0. 上游设计依据（工程依据）

HLD §5、ADR-001/004、FEAT-24/26、CP2 minima/QAC。

## 1. 目标（Goal）

创建 stable CSCV split validator、PBO calculator、raw-count DSR calculator 和 non-alias guards。

## 2. 需求（Requirements）

PBO：candidate≥4、valid splits≥4、train/test non-empty、stable split ids、explicit rank/loss/tie direction。DSR：raw trials≥2、sample≥30、finite Sharpe/skew/kurtosis、variance>0、sealed raw ref；effective fields unavailable/empty。

## 3. 模块拆分

CSCV split manifest/validator、PBO calculation、moment validation、DSR calculation/non-alias projection；物理文件 `engine/overfit_evidence.py`。

## 4. 代码结构与文件影响

创建 `engine/overfit_evidence.py`、`tests/research/test_overfit_evidence.py`；禁止修改 CR163 lineage 与 effective fields in consumer。

## 5. 数据模型

`CSCVMembership/Split` 含 stable ids/train/test ids；PBO evidence 含 split count/ranks/logits/PBO；DSR config 含 `dsr_input_method=raw_trial_count`；effective value/ref/method 为 null/empty，availability typed_unavailable。

## 6. API

`validate_cscv_splits`、`calculate_pbo`、`calculate_raw_count_dsr`、`validate_dsr_count_non_alias`。调用者为 S04；输出统一 MethodEvidence。

## 7. 流程

validate S01 identity → validate split/moments/count → calculate → finite/domain validate → wrap evidence。Leak/duplicate/mismatch/non-finite/alias=blocked；minima不足=unavailable；policy miss=fail。

## 8. 技术细节

Split id 由 sorted train/test ids + schema version hash；每 observation 不得同时出现在同 split train/test。PBO/DSR exact equations、tail、kurtosis convention、tie behavior 与 golden values必须在实现 docstring/test 引 primary source；不得用 library default 隐藏语义。

## 9. 安全与性能

No I/O/global RNG/runtime/data/network；combinatorial size 只允许 fixture-policy 上限，超限行为在实现中 typed/block 明确，不声明生产容量。

## 10. 测试

valid 4x4 fixture；split duplicate/leak/empty/invalid count；PBO [0,1]；DSR minima/moments/variance；raw ref mismatch；raw copied to effective；consumer-requires-effective remains unavailable。

## 11. 实施步骤（TASKS）

S03-T01 split validator；T02 PBO；T03 DSR/non-alias；T04 golden/negative tests。

## 12. 风险与灰区

无 LCQ。风险为 combinatorial explosion、rank convention、DSR formula drift；以 explicit policy/version/golden fixtures 缓解。Effective estimator OUT。

## 13. 回滚

停用 PBO/DSR producer，相关 evidence 回 unavailable；不改变 raw lineage/effective claim。无发布授权。

## 14. DoD / Definition of Done

- [ ] minima/split leak/non-alias 100% tested
- [ ] PBO/DSR finite/domain/golden fixtures 固定
- [ ] effective alias=0；CP5 前不实现
