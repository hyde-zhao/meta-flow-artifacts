---
story_id: "STORY-CR048-S01"
cr_id: "CR-048"
title: "Integrate the single-source rules branch"
status: "ready-for-verification"
design_evidence_type: "technical-note"
lld_policy: "technical-note"
owner: "host-orchestrator-inline"
---

# STORY-CR048-S01 Branch Integration

## 目标与验收

在一个保留双亲的 merge commit 中合并 `origin/fix/claude-md-rule-numbering`，解决三处冲突，并使规则单源、安装生成、dispatch/handoff 检查与 current evidence 语义同时成立。

- tracked `delivery/rules/CLAUDE.md` 删除，Claude dry-run 仍生成目标 `CLAUDE.md`。
- `event_id`、`dispatch_id`、`attempt_id` 身份分离；CP result 按 attempt 聚合生命周期证据。
- 新格式 handoff 严格，默认目录扫描兼容无 dispatch block 的历史文件。
- 全量 pytest、Ruff、guardrail、clean snapshot 和三平台 dry-run 通过。

## 技术说明

- 设计依据：CR048-DQ-01..04 和 CR-046 typed evidence 契约。
- 文件影响：`delivery/rules`、安装器、package builder、guardrail、CLI、dispatch/cp checker、测试和说明文档。
- 接口：新增 `meta-flow check handoff-dispatch`；目录模式支持 `--strict-all`。
- 数据：ledger 中 `event_id` 是事件身份，`dispatch_id + attempt_id` 是尝试聚合键。
- 权限：只操作两个已授权 Git 仓库；禁止 runtime、凭据、quant-lab 和 prelink backup。
- 失败：任一语义冲突、测试、Ruff、guardrail 或 dry-run 失败即阻断 merge commit。
- 测试：定向 103+8、全量 434+70、Ruff、clean snapshot guardrail、三平台 dry-run。
- 风险：没有独立 QA 子 Agent，CP7/CP8 最高 `PASS_WITH_RISK` / `READY_WITH_RISK`。
