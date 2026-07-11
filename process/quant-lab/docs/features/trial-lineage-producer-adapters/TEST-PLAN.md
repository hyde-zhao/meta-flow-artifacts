---
status: draft-for-cp5
version: "1.0"
feature_id: "FEAT-21"
---

# Feature Test Plan: Trial-lineage Producer Adapters

## 测试范围

| Scope | Mapping/scenario | 测试 | 量化出口 |
|---|---|---|---|
| TP21-01 | CPI-001/003, P01 | public chain fixture/static call trace | 2/2 mappings；declaration before first trial；session count=1 |
| TP21-02 | CPI-002/004, P01 | legacy chain fixture/static call trace | 2/2 mappings；declaration before first trial；session count=1 |
| TP21-03 | P02/B01/F01 | params/seed/retry/terminal matrix | seed A/B raw=2；3 retries are attempts；terminal retention=100% |
| TP21-04 | N01/N02 | missing session/mapping/duplicate conflict | missing included mapping 100% unavailable/blocked；conflict 100% blocked |
| TP21-05 | coverage | inventory coverage report | CPI-001..004=4/4；deduplicated chains=2/2；double-count=0 |

## 权限/安全

Tests use fixture inputs and patched writers only. Network/provider/lake/NAS/credential/runtime/trading/external registry/Git remote counters must all equal 0.

## 手工验收

- Review four mapping call sites against frozen inventory; excluded anomaly/adapter/consumer paths remain excluded/N/A with reasons.
- Review S03 LLD for exactly one selection writer per chain and one session owner per chain.

