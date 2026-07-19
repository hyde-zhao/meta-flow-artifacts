---
handoff_id: CR173-CP5-META-DEV-LLD
cr_id: CR-173
from: host-orchestrator
to: meta-dev
codex_agent_name: meta-dev
reasoning_profile: default
status: returned
created_at: "2026-07-16T15:40:10+08:00"
dispatch_trigger: CP4-PASS-auto-continue-to-CP5-design-evidence
context_ref: process/context/CP5-CR173.context.json
route_plan_ref: process/checks/CP0-CR173.route-plan.json
cp4_result_ref: process/checks/CP4-CR173-STORY-DAG-PARALLEL-SAFETY.result.json
dispatch_ref: AD-CR173-CP5-META-DEV-SPAWN-20260716T154010+0800
agent_id: /root/dev_yang_cr173_cp5
completed_at: "2026-07-16T16:16:13+08:00"
return_ref: process/handoffs/CR173-CP5-META-DEV-LLD-RETURN-SUMMARY.md
---

# CR-173 CP5 Meta-Dev LLD Handoff

## 目标

按 S01→S02→S03 的已批准设计顺序，为三个 `full-lld` Story 编写独立 LLD，使全部设计证据达到 CP5 人工审查就绪。不得使用 batch-lld 替代三个 full LLD，不得实现源码、测试或 fixture。

## 上下文策略

- 必须先读 `process/context/CP5-CR173.context.json`，并通过 capsule-first 最小读取集工作。
- 仅在算法、failure mapping 或 public split 细节不足时，按 `read_if_needed` 展开 CR173 HLD/ADR；扩展读取须写 `READ-EXPANSION-LEDGER.ndjson` 或在 return 中给出 event ref。
- `process/archive/**`、无关 Story/LLD/测试报告不得默认读取。

## 允许写入

- `process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md`
- `process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md`
- `process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md`
- 三张 Story 卡的 `lld_gate.status` / 设计证据元数据，只能更新为 `ready-for-cp5-review`，不得标记 approved/confirmed。
- `process/state/QUESTION-LEDGER.ndjson`：仅在出现真正阻塞澄清时追加；不得直接向用户提问。
- `process/handoffs/CR173-CP5-META-DEV-LLD-RETURN-SUMMARY.md`

## 三份 LLD 必须冻结的核心合同

1. S01：七字段 `7/7`、present/null/ref 规则、8 failure reason、canonical numeric/bytes/hash、append-only recovery；raw alias、strategy identity、public write 均为 0。
2. S02：deterministic exact-rational decimal/parser/matrix validator、唯一 pivot comparator、fraction-free `LDLᵀ`、zero-pivot residual coupling、`n²/ΣRij²`、未舍入与舍入后 `2/2` invariant、half-even 一次、float/tolerance/clamp=0。
3. S03：6 类 golden × 3/3、8/8 failure、7-field mutation/orphan、8+12 public inventory、zero-operation/public-diff/call/overclaim、append-only recovery；不得创建 public adapter。

## 设计评审规则

- 每份 LLD 必须含：修订记录、模块/文件影响、数据与接口、逐步流程/伪代码、异常与失败映射、测试设计、任务映射、发布/回滚、风险、Gotchas、开放项状态。
- 成功标准全部量化；Feature/Story/Wave/Task 数必须保持 1/3/3/12。
- 调用方向、时机、输入、输出、降级、后续衔接、调用方同步修改面必须完整；public 同步修改面固定为 0。
- S02 若无法证明 pivot comparator 和 singular PSD 行为唯一，返回 `NEEDS_DESIGN_CLARIFICATION`，不得硬写实现方案。
- S03 必须区分 canonical result identity 与 append-only attempt audit ref，避免“重复三次 hash 唯一”和“每次 attempt ref 新增”自相矛盾。

## 禁止项

- 不创建/修改 `engine/*.py`、`tests/**/*.py` 或 fixture 实现。
- 不运行新增实现测试；只允许文档/结构/静态 schema 检查。
- 不触碰 public C1、Gate1、DSR、admission production contract，不读取真实数据/credential/provider，不执行 runtime/trading/publish/deploy/Git remote write。
- 不发起 CP5 人工门禁，不修改 STATE/GATE/CHECKPOINT ledger。

## 机器可验证出口

- full LLD=`3/3`；`meta-flow story lld-check`=`3/3 OK`。
- clarification blocking=`0`；若非 0，返回 queue refs 并停止。
- CP5 审查前 `design_evidence_confirmed=false`、实现数=`0`。
- public projection Feature/Story/Task/production write=`0/0/0/0`。
