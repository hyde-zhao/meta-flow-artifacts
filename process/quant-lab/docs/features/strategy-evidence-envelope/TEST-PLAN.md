---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-01"
---

# FEAT-166-01 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 定义 schema、canonical、C1 golden、catalog 与 tamper 的 fixture/static 验证。 |

## 测试范围与追踪

| Test ID | 场景 / 要求 | 断言 |
|---|---|---|
| F166-01-T01 | REQ-001/005；SC-P01 | envelope/C2 contract 字段完整，round-trip 后语义相同。 |
| F166-01-T02 | QAC-07 | 同一 payload 10 次 component/envelope hash distinct count=1。 |
| F166-01-T03 | C1 compatibility | 现有 public import、默认 domain、canonical bytes 与 golden hash 100% 不变。 |
| F166-01-T04 | SC-H01 | optional component 重排 hash 相同；canonical field 篡改沿用旧 hash 100% blocked。 |
| F166-01-T05 | REQ-006/QAC-09 | active C2=1；C3/C4 calculator=0；mandatory unknown blocked；optional unknown PASS 贡献=0。 |
| F166-01-T06 | numeric safety | NaN、Inf、非字符串 key、set 全部被拒绝，false PASS=0。 |

## Fixture 与执行方式

只使用 repo-local Python value/JSON fixture；无临时外部服务。golden C1 payload 复用 `tests/research/test_statistical_evidence_contracts.py`，新增 C2 contract 用 `tests/research/test_walk_forward_oos_contracts.py`。所有 ref 均为 opaque `fixture://` 或 logical ref，不检查存在性。

## 通过标准

6/6 test groups 通过；C1 差异=0；10→1；non-finite acceptance=0；unknown mandatory false PASS=0；外部操作=0。任一 C1 golden 差异为 blocker，不允许更新期望值规避。
