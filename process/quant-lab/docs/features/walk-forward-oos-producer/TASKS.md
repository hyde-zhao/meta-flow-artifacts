---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-03"
---

# FEAT-166-03 任务清单

| Task ID | Story | 任务 | 依赖 | 完成准则 |
|---|---|---|---|---|
| F166-03-K01 | S03 | 实现 metric/fold decision 纯函数 | S01/S02 | 只接受 validated input；no caller-passed truth。 |
| F166-03-K02 | S03 | 实现 declared denominator aggregate | K01 | count/pass-rate 100% 可重算。 |
| F166-03-K03 | S03 | 实现 component/envelope hash 与 provenance | K01-K02 | 10→1；无 clock/path/runtime。 |
| F166-03-K04 | S03 | 实现 self-validation 与 typed failures | K03 | tamper acceptance=0；false present=0。 |
| F166-03-K05 | S03/S05 | 完成 producer fixtures | K01-K04 | TEST-PLAN 8/8；operation=0。 |

所有任务写 `engine/walk_forward_oos_evidence.py` 或对应新测试；S03 在 W3 独占该生产文件。CP5 批准前禁止实现。
