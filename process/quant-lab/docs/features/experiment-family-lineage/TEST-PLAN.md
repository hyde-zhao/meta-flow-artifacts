---
status: draft-for-cp5
version: "1.0"
feature_id: "FEAT-20"
---

# Feature Test Plan: Experiment-family Lineage Core

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-11 | meta-se-critical | 初始 core contract/storage/seal/validation fixture plan。 |

## 测试范围

| Scope | 内容 | 场景 | 层级 | 量化出口 |
|---|---|---|---|---|
| TP20-01 | pre-search declaration/state | P01,N01 | unit/contract | 2/2 chains ordering可表达；post-hoc 1/1 blocked |
| TP20-02 | trial/attempt/selection/count | P02,N02,B01,F01 | unit | orphan=0；seed A/B raw=2；1 trial 3 attempts raw=1；terminal retention=100% |
| TP20-03 | canonical seal/hash | P03,T01 | unit/golden | identical fixture 10 seals→1 hash；tamper 1/1 blocked |
| TP20-04 | supersession/recovery | R01 | unit/integration | v1 hash保留；v2 link有效；broken/cyclic chain 100% blocked |
| TP20-05 | availability/permission | B02,A01,G01 | contract/static | effective available=0；forbidden counters=0；CR155 backfill=0 |

## 风险与失败路径

| Risk | 测试方式 | 证据入口 |
|---|---|---|
| canonical drift | golden canonical bytes + hash vector，多次 randomized input key order | planned `tests/test_experiment_family_lineage.py` |
| identity conflict | same event id same/different payload pair | planned core tests |
| shrinkage | selected-only vs full declared membership recount | planned core tests |
| mutable seal | mutate sealed fixture and resolver chain | planned store tests |
| unauthorized I/O | monkeypatch/counter guard for lake/NAS/provider/credential/runtime/external writes | planned S05 tests |

## 手工验收

| Check | 操作 | 期望 | Owner |
|---|---|---|---|
| MAN20-01 | review object/state/error tables against HLD/ADR | 6/6 objects、全部 transitions、无 effective computation | meta-qa |
| MAN20-02 | inspect golden manifest and v1→v2 refs | old bytes/hash不变、chain完整 | meta-qa |

所有测试为 fixture/static；不读取真实 lake/data/credentials，不执行 runtime/trading/external registry。

