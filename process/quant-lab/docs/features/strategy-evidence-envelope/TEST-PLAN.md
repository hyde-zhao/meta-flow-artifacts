---
status: "ready-for-cp5-review"
version: "0.2"
cr_id: "CR-166"
feature_id: "FEAT-166-01"
change_refs: ["CR-166", "CR-168"]
---

# FEAT-166-01 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 定义 schema、canonical、C1 golden、catalog 与 tamper 的 fixture/static 验证。 |
| 0.2 | 2026-07-14 | host-orchestrator inline meta-se | 增补 CR168 `economic_cost@v1` catalog、attachment identity、daily/ML hash 分域与 C1/C2 零回归验证。 |

## 测试范围与追踪

| Test ID | 场景 / 要求 | 断言 |
|---|---|---|
| F166-01-T01 | REQ-001/005；SC-P01 | envelope/C2 contract 字段完整，round-trip 后语义相同。 |
| F166-01-T02 | QAC-07 | 同一 payload 10 次 component/envelope hash distinct count=1。 |
| F166-01-T03 | C1 compatibility | 现有 public import、默认 domain、canonical bytes 与 golden hash 100% 不变。 |
| F166-01-T04 | SC-H01 | optional component 重排 hash 相同；canonical field 篡改沿用旧 hash 100% blocked。 |
| F166-01-T05 | REQ-006/QAC-09 | active C2=1；C3/C4 calculator=0；mandatory unknown blocked；optional unknown PASS 贡献=0。 |
| F166-01-T06 | numeric safety | NaN、Inf、非字符串 key、set 全部被拒绝，false PASS=0。 |
| F166-01-T07 | CR168 A1 / SIM-02 | 字段族 2-9 相同、strategy/package subject 不同：component hash distinct=1，envelope hash distinct=2。 |
| F166-01-T08 | CR168 catalog activation | `economic_cost@v1` active=1；C1/C2 serialized bytes/hash 差异=0；capacity remains reserved。 |
| F166-01-T09 | CR168 identity tamper | attachment identity 或 package auth 篡改且沿用旧 envelope hash 100% blocked。 |

## Fixture 与执行方式

只使用 repo-local Python value/JSON fixture；无临时外部服务。golden C1 payload 复用 `tests/research/test_statistical_evidence_contracts.py`，新增 C2 contract 用 `tests/research/test_walk_forward_oos_contracts.py`。所有 ref 均为 opaque `fixture://` 或 logical ref，不检查存在性。

## 通过标准

9/9 test groups 通过；C1/C2 差异=0；component 10→1；daily/ML envelope distinct=2；identity tamper false PASS=0；non-finite acceptance=0；unknown mandatory false PASS=0；外部操作=0。任一 C1/C2 golden 差异为 blocker，不允许更新期望值规避。
