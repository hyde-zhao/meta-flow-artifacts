---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-03"
---

# FEAT-168-03 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 定义完整 QAC、fixture、security、claim 与 suite attribution。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：给 S05 authorization suite 分配 capability registry missing 的 N/A-with-reason 校验。 |

| Scope ID | 覆盖 | 通过标准 |
|---|---|---|
| EV-T01 | REQ/QAC/scenario trace | 9/9、15/15、17/17；P0=16/16、P1=1/1。 |
| EV-T02 | two fixtures | daily 1/1；multi-strategy 1/1；event producer=0。 |
| EV-T03 | negatives/determinism | 10/10；10→1；false PASS=0。 |
| EV-T04 | Gate4 containment | B01/B02 2/2；8/8 reject；capacity/aggregate PASS=0。 |
| EV-T05 | regressions | C1/C2 regression=0；CR155 BLOCKED + paper_candidate=false；promotion=0。 |
| EV-T06 | authorization | credential/data/provider/NAS/lake/runtime/broker/trading/store/registry/publish/Git writes 各 0；capability registry missing 必须经 existing Feature/module refs 给出 N/A-with-reason，persistent registry write=0、parallel registry=0。 |
| EV-T07 | claim/doc guard | real TCA/calibration/runtime/Stage3 claims=0；wrong quality refs=0。 |
| EV-T08 | targeted/full suite | CR168 introduced failures=0；unknown attribution=0。 |

执行均使用 `uv run --python 3.11`。CP5 批准前只允许静态检查现有设计文件，不运行新增实现测试；CP7 full suite 才形成最终证据。
