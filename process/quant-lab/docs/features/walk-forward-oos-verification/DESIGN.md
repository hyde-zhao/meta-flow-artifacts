---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-05"
owner: "meta-se-inline"
---

# FEAT-166-05 Fixture/static Verification 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 冻结独立验证编排、12 项 QAC、8 类负向、event N/A、权限与历史失败归因。 |

## 1. 适用性与所有权

本 Feature 因 claim-sensitive 安全边界、跨四个生产 Feature 的 QAC 汇总和 CR165 历史失败归因而 `required`。它拥有 test/fixture/evidence-index 设计，不拥有生产 schema、validator、producer、consumer policy 或独立验证 runtime。

## 2. 文件布局

| 文件 | 职责 |
|---|---|
| `tests/research/test_walk_forward_oos_qac.py` | QAC-01..12 汇总与追踪。 |
| `tests/research/test_walk_forward_oos_authorization.py` | zero-dereference、forbidden counter、static scan、Stage flags。 |
| `tests/research/test_walk_forward_oos_cr155_regression.py` | CR155 blocked/paper false 与 CR165 历史失败触达归因。 |
| `tests/fixtures/walk_forward_oos/` | daily/ML positive、8 类 negative/boundary/tamper fixture；不含真实数据。 |

## 3. 验证矩阵

| 维度 | 精确目标 |
|---|---|
| P0 fail-closed | 8/8：fold、temporal、purge、embargo、metric、lineage、authorization、hash。 |
| strategy compatibility | daily/ML=2/2；event applicability=1/1 N/A；event producer/fixture=0。 |
| input/temporal | 7/7 字段族；temporal/leakage negative=3/3 blocked。 |
| integrity/determinism | lineage false PASS=0、orphan=0、10 runs→1 hash、tamper acceptance=0。 |
| extension/integration | C3/C4 calculator=0；consumer=3/3；unknown mandatory false PASS=0。 |
| authorization/claim | dereference=0；forbidden operations=0；Stage2=true；Stage3/runtime/real-evidence=false。 |
| regression | CR166 新路径 failure=0；触及 CR165 14 项时逐项归因率=100%。 |

## 4. 执行分层

1. contract/unit subsets；2. validator/producer subsets；3. projection/CR155 subsets；4. QAC/authorization static subsets；5. repository full suite。只允许 `uv run` 本地 fixture/static 执行。full suite 若触及 CR165 14 项历史失败，记录测试 ID、触发路径、基线证据与非 CR166 回归理由；不能笼统豁免。

## 5. 失败、授权与 Gotchas

任一 false PASS、外部操作、CR155 promotion、Stage3/real-evidence overclaim、unattributed touched baseline failure 均为 CP7 FAIL/NEEDS_REWORK。禁止 lake/NAS/provider/credential/external framework/runtime/broker/trading/publish/deploy/Git remote write。Gotcha：fixture PASS 只证明 contract/algorithm path；不证明真实 OOS 或策略收益。Gotcha：静态扫描没有命中危险词不等于运行授权。

## 6. 回退与重访

若生产实现回退，验证文件随对应 Feature 停用但保留 evidence refs；不得将失败用删除用例解决。出现真实数据验证授权时必须独立 Stage 3 CR 与新 test strategy，不扩张本 Feature。
