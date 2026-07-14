---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-03"
---

# FEAT-168-03 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 初始 S05 执行任务。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：authorization task 明确验证 capability registry missing 的 N/A-with-reason 处置。 |

| TASK-ID | Story | 顺序 | 动作 | 输出文件 | 完成准则 |
|---|---|---:|---|---|---|
| EV-K01 | S05 | 1 | 创建 2 个 synthetic/static fixture 族 | `tests/fixtures/economic_cost/` | 2/2；真实/credential 数据=0。 |
| EV-K02 | S05 | 2 | 创建 QAC/negative/determinism suite | `tests/research/test_economic_cost_cr168_qac.py` | 15/15、10/10、10→1。 |
| EV-K03 | S05 | 3 | 创建 authorization/forbidden-operation suite | `tests/research/test_economic_cost_authorization.py` | 所有 forbidden counters=0；registry missing 只能是 Feature/module refs 的 N/A-with-reason，persistent/parallel registry=0。 |
| EV-K04 | S05 | 4 | 创建 CR155/C1/C2 regression suite | `tests/research/test_economic_cost_cr155_regression.py` | promotion=0；paper_candidate=false；existing regression=0。 |
| EV-K05 | S05 | 5 | 执行 targeted/full suite 与 failure attribution | CP7 evidence/quality docs | introduced/unknown failures=0。 |

阻塞项：独立 verifier 未提供不是 CP5 blocker；若 CP7 仍 inline，CP8 显式披露。CP5 前禁止执行实现测试。
