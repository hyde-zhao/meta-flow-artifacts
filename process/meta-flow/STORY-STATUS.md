---
last_updated: "2026-07-03T00:00:00+08:00"
---

## Story 状态汇总

| Story ID | 标题 | Wave | 状态 | 负责人 | 阻塞 |
|----------|------|------|------|--------|------|
| UCD-01 | use-case-discovery 核心增强 | W1 | verified | meta-qa | 否 |
| UCD-02 | use-case-discovery 模板与引用增强 | W1 | verified | meta-qa | 否 |
| LLD-01 | lld-designer 六阶段管道与输入输出契约 | W1 | verified | meta-qa | 否 |
| UCD-03 | UCD 上下游契约同步 | W2 | verified | meta-qa | 否 |
| RG-01 | meta-po 评审协调核心 | W2 | verified | meta-qa | 否 |
| LLD-02 | STORY-LLD 模板增强（14 章节固定） | W2 | verified | meta-qa | 否 |
| RG-02 | review 产物协议与静态前置检查 | W3 | verified | meta-qa | 否 |
| LLD-03 | LLD 生产/消费契约对齐 | W3 | verified | meta-qa | 否 |
| RG-03 | review mode 第一组扩展 | W4 | verified | meta-qa | 否 |
| RG-04 | review mode 第二组扩展 | W4 | verified | meta-qa | 否 |
| RG-05 | 分派表与灰度落地 | W5 | verified | meta-qa | 否 |
| LLD-04 | LLD 方法试点 Story | W6 | verified | meta-qa | 否 |

## Wave 进度

| Wave | 总数 | done | verified | in-dev | blocked |
|------|------|------|----------|--------|---------|
| W1 | 3 | 0 | 3 | 0 | 0 |
| W2 | 3 | 0 | 3 | 0 | 0 |
| W3 | 2 | 0 | 2 | 0 | 0 |
| W4 | 2 | 0 | 2 | 0 | 0 |
| W5 | 1 | 0 | 1 | 0 | 0 |
| W6 | 1 | 0 | 1 | 0 | 0 |

## 阻塞项清单

- 当前无阻塞项

## CR-037 Story 状态汇总

| Story ID | 标题 | Wave | 状态 | 负责人 | 阻塞 |
|----------|------|------|------|--------|------|
| CR037-S00 | second-system guardrail | CR037-W0 | verified-with-risk | meta-dev | 否 |
| CR037-S01 | current-state schema and budgets | CR037-W1 | verified-with-risk | meta-dev | 否 |
| CR037-S02 | controlled update API and writer refactor | CR037-W1 | verified-with-risk | meta-dev | 否 |
| CR037-S03 | agent contract and guardrail sync | CR037-W1 | verified-with-risk | meta-dev | 否 |
| CR037-S04 | ledger compaction policy and CLI | CR037-W1A | verified-with-risk | meta-dev | 否 |
| CR037-S05 | project scaffold and PROJECT.current | CR037-W2 | verified-with-risk | meta-dev | 否 |
| CR037-S06 | PROJECT-SCALE and roadmap objects | CR037-W2 | verified-with-risk | meta-dev | 否 |
| CR037-S07 | feature/capability registry and resolver | CR037-W3 | verified-with-risk | meta-dev | 否 |
| CR037-S08 | impact surface field split and migration report | CR037-W3 | dev-ready | meta-dev | 否 |
| CR037-S09 | roadmap refresh result schema and checker | CR037-W4 | dev-ready | meta-dev | 否 |
| CR037-S10 | process-only cascade and Gate Ledger event | CR037-W4 | dev-ready | meta-dev | 否 |
| CR037-S11 | FU-RF tracking support | CR037-W4 | dev-ready | meta-dev | 否 |
| CR037-S12 | project stale-check | CR037-W4 | dev-ready | meta-dev | 否 |
| CR037-S13 | quant-lab migration dry-run and reports | CR037-W5 | dev-ready | meta-dev | 否 |

## CR-037 Wave 进度

| Wave | 总数 | lld-ready | lld-review | dev-ready | implementing | ready-for-verification | verified-with-risk | blocked |
|------|------|-----------|------------|-----------|--------------|------------------------|--------------------|---------|
| CR037-W0 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| CR037-W1 | 3 | 0 | 0 | 0 | 0 | 0 | 3 | 0 |
| CR037-W1A | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| CR037-W2 | 2 | 0 | 0 | 0 | 0 | 0 | 2 | 0 |
| CR037-W3 | 2 | 0 | 0 | 1 | 1 | 0 | 0 | 0 |
| CR037-W4 | 4 | 0 | 0 | 4 | 0 | 0 | 0 | 0 |
| CR037-W5 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |

## CR-037 实施期风险跟踪

| Risk ID | Story | 状态 | 实施期处理要求 |
|---------|-------|------|--------------------|
| R-CR037-S13-LONG-CHAIN | CR037-S13 | 实施期跟踪中，非 CP5 阻断 | S13 作为 P2 quant-lab migration 验证切片依赖 8 个 P0/P1 Story。真实 migration 必须等上游 Story verified / dev_gate 满足；若上游延期，只允许保持 blocked，或缩窄为只读 inventory/report。 |
