---
status: draft
version: "1.0"
feature_id: "FEAT-EI-CORRECTION"
related_stories: ["ST-EI-007"]
lld_policy_summary: "1 full-lld"
---

# Feature Design: Correction Lifecycle and Pilot Adapter

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se | 冻结versioned append-only correction、allowlist、supersession、CR-163 adapter和legacy D3迁移边界。 |

## 目标与边界

本 Feature提供通用 post-close correction event与 checker，再由CR-163 adapter消费。它不原位修改历史、不运行真实pilot、不访问runtime/credentials、不修改quant-lab lineage业务源码，也不为平台补造receipt/model。

## 设计合同

CorrectionEvent包含schema version、target typed ref、allowlisted patch、author、reason、evidence、supersedes；链必须无环，原对象byte/hash不变。错误 correction通过新的 superseding correction修正。PilotManifest必须包含独立 authorization ref、immutable input hashes、append set、23-target replay、protected business paths和rollback append set。

legacy migration读取 `codex_agent_name` 时只生成：`source_level=D3`、`classification=self-declared-unverifiable`、`resolved_agent_profile/model/reasoning_effort=null|unavailable`；保守审计假设与事实字段分离。不得把历史任务标签、handoff或ledger自报提升为resolved。

## 现有位置与计划面

| 路径 | 计划 |
|---|---|
| `meta_flow/workflow/cr_lifecycle.py`, `story_evidence.py` 或新 correction模块（LLD定名） | correction schema/checker/lifecycle |
| `meta_flow/cli.py` | proposed correction/replay/pilot dry-run接口 |
| `tests/test_cr_lifecycle.py` + 新fixtures | allowlist/cycle/immutability/D3/pilot authz |

## 失败、权限与回滚

非allowlist、缺author/reason/evidence、环、target缺失一律拒绝追加。无独立pilot授权只允许fixture/dry-run设计。执行中业务diff!=0或replay!=23/23则不接受pilot，并只通过append-only补偿/隔离回滚，不改原历史。

## Story LLD 合同与 Gotchas

ST-EI-007 LLD必须下沉schema、allowlist、命令边界、事务/部分失败、pilot manifest、protected paths、D3 migration fixture和授权门。append-only不是“可任意追加”；adapter不得反向依赖quant-lab业务模块。
