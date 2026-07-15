---
story_id: "STORY-CR048-S01"
cr_id: "CR-048"
stage: "CP6"
status: "implemented_with_risk"
implemented_by: "host-orchestrator-inline"
---

# STORY-CR048-S01 Implementation

## 实现对象清单

- 合并功能分支的单一规则源、Claude 目标生成、package builder、guardrail 和文档变化。
- 保留 main 的 CP result correlation/hash/role/checkpoint 检查，并把真实 dispatch 完整性改为 attempt 聚合。
- 保持 `event_id` 唯一，不以 `dispatch_id` 或 `run_id` 冒充事件身份。
- 新增 handoff dispatch validator；显式文件检查严格，默认目录扫描只检查声明新契约的文件，`--strict-all` 用于迁移审计。

## 设计契约映射

| 决策 | 实现 |
|---|---|
| CR048-DQ-01 | 删除 tracked `delivery/rules/CLAUDE.md`；安装器从 AGENTS 生成 Claude 目标 |
| CR048-DQ-02 | `_validate_dispatch_refs` 保留 current 语义并聚合同 attempt 多事件 |
| CR048-DQ-03 | ledger 只以 `event_id` 去重；typed terminal 强制 closure/completed_at |
| CR048-DQ-04 | handoff 默认 legacy-aware，单文件和 `--strict-all` 严格 |

## 单元测试 / Fixture

- 新增 complete/incomplete subagent、inline fallback、handoff-only、legacy scan 和 strict-all fixture。
- 新增 split typed attempt fixture，证明 running 与 completed 分行时不会 first-match 误判。
- 更新 CR-046 identity fixture，证明 dispatch/event/attempt 三类身份保持分离。

## 最小实现切片与验证

1. 解决三处文本冲突并删除第二规则源。
2. 定向回归：103 passed、8 subtests。
3. 全量回归：434 passed、70 subtests；Ruff 0。
4. 当前 workspace 与无根 AGENTS 的 clean snapshot guardrail 均通过。
5. Codex、Claude、Qoder project/full/noninteractive dry-run 3/3 通过。

## 平台差异与风险

- Claude 的目标文件名仍为 `CLAUDE.md`，仅其内容源统一为 tracked `AGENTS.md`。
- 本轮按用户既有要求不使用子 Agent；实现和验证是 host inline，不声称独立 QA/platform attestation。
