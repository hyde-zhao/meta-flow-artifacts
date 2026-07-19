---
status: final-for-cp8
version: "1.0"
cr_id: "CR-173"
verification_decision: PASS_WITH_RISK
updated_at: "2026-07-17T12:30:00+08:00"
---

# CR-173 Verification Report

## 1. 结论

CR-173 的 3/3 Story 均取得最终 CP7 `PASS`，blocker=`0`、waiver=`0`。本轮交付可以进入 CP8 人工终验，但结论只覆盖 fixture-only、strategy-agnostic 的离线 participation-ratio estimator、七字段 standalone evidence 与 fail-closed/授权边界；最高声明为 `offline_method_ready`。

`public_effective_trial_count_populatable=false`、`c1_computable=false`、CR-172 自动恢复次数=`0`。真实数据、runtime、Stage 3、admission、public C1 projection/write、publish/deploy 均未执行且未获授权。

## 2. Story 验证摘要

| Story | 最终结果 | 核心证据 | Findings |
|---|---|---|---|
| CR173-S01 contract/evidence/canonicalization | PASS | 60/60 回归；相邻对抗 10/10；public/real/runtime/credential/remote 操作 0 | F-CP7-S01-001/002/003 全部 CLOSED |
| CR173-S02 exact spectral estimator | PASS | 87/87；exact PSD oracle 854/854，mismatch=0；compile 3/3 | 新增 finding 0；waiver 0 |
| CR173-S03 golden/failure/boundary | PASS | authorization 9/9；targeted 124/124；combined public read-only 379/379 | F-CP7-S03-001 CLOSED |

## 3. 需求与场景追踪

| 追踪面 | 覆盖 | 结论 |
|---|---:|---|
| REQ-CR173-001..008 | 8/8 | PASS；estimand、输入、失败语义、七字段 schema、确定性、standalone/public stop、恢复边界均有最终证据 |
| 工程场景 | 8/8 | PASS；正向、负向、边界、权限、失败恢复均覆盖 |
| golden classes | 6/6 × 3/3 repeats | PASS；稳定 computation/evidence identity 与 append-only audit |
| failure classes | F01..F08，8/8 | PASS；缺失/不足 fail-closed，矛盾/篡改 blocked，不发生 raw alias |
| public boundary | 8 个 production path + 12 个 read-only regression path | 修改 0/0；public dependency/call/write 0 |

## 4. Claim ceiling 与授权核验

| 声明/操作 | 最终值 |
|---|---:|
| `offline_method_ready` | `true`（仅在 CP8 批准后成立） |
| `public_effective_trial_count_populatable` | `false` |
| `c1_computable` | `false` |
| `real_evidence_available` / `stage3_started` / `stage3_entry_ready` | `false / false / false` |
| `mature_admission_pass` / `aggregate_orchestration_implemented` | `false / false` |
| CR-172 auto-close / auto-resume / auto-approve | `0 / 0 / 0` |
| real data / credential / provider / runtime / trading / remote write | `0 / 0 / 0 / 0 / 0 / 0` |

## 5. 剩余风险

| Risk ID | 严重度 | 状态 | CP8 处理 |
|---|---|---|---|
| R-CR173-EXACT-INTEGER-GROWTH | MEDIUM | accepted fixture-only | 当前 fixture 最大 n=4，不承诺 production SLA；CR-172 PATH-C/A 或真实规模/性能目标出现前必须重评 |
| R-CR173-SECOND-ORDER-MODEL-BIAS | MEDIUM | accepted claim-limited | participation ratio 仅表示二阶相关结构有效维度，不是 Li–Ji/BH/FWER/Šidák/DSR/admission calibration |
| R-CR173-PUBLIC-MIGRATION | HIGH（延期域） | deferred | public C1 versioned projection 必须由独立 owner/CR 批准；本 CR 不形成双真相 |

## 6. 证据入口

- `process/checks/CP7-CR173-S01-REVERIFY-2.result.json`
- `process/checks/CP7-CR173-S02-VERIFICATION.result.json`
- `process/checks/CP7-CR173-S03-REVERIFY-1.result.json`
- `process/evidence/CR173-S01-contract-evidence-canonicalization.CP7R2.index.json`
- `process/evidence/CR173-S02-exact-spectral-estimator.CP7.index.json`
- `process/evidence/CR173-S03-golden-failure-boundary-verification.CP7R1.index.json`

