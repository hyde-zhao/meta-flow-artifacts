---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-02"
---

# FEAT-168-02 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 初始 S04 执行任务。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：任务锁定 B01 精确 claims 与 public-callable DI double。 |

| TASK-ID | Story | 顺序 | 动作 | 输出文件 | 完成准则 |
|---|---|---:|---|---|---|
| G4-K01 | S04 | 1 | 创建 typed outcome 与 public forbidden-key constant | `engine/economic_cost_gate4_projection.py` | 4/4 allowlist、8/8 denylist。 |
| G4-K02 | S04 | 2 | 创建 pre-call decision/operation guard | 同上 | escape/C3 unavailable/C4 present calls=0。 |
| G4-K03 | S04 | 3 | 创建 fixed candidate-release canonical call | 同上 | canonical public call site=1；private imports=0。 |
| G4-K04 | S04 | 4 | 创建双 postcondition reason | 同上 | PASS/claim violation 分离；B01 精确三 claim；aggregate=0。 |
| G4-K05 | S04/S05 | 5 | 创建 integration + public-double tests | `tests/research/test_economic_cost_gate4_projection.py` | keyword-only `gate4_validator=` DI fake；不 monkeypatch canonical/private helper；TEST-PLAN 9/9。 |

阻塞项：无。无法在 source-change=0 下满足 postcondition 时回退 component-only，不得偷改 canonical。
