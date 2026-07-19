---
handoff_id: CR173-CP4-META-SE-STORY-PLANNING
cr_id: CR-173
from: host-orchestrator
to: meta-se
codex_agent_name: meta-se-critical
reasoning_profile: critical
status: returned
created_at: "2026-07-16T15:11:03+08:00"
completed_at: "2026-07-16T15:34:41+08:00"
return_ref: process/handoffs/CR173-CP4-META-SE-STORY-PLANNING-RETURN-SUMMARY.md
dispatch_trigger: CP3-approved-auto-continue-to-CP4
context_ref: process/context/CP3-CR173.context.json
route_plan_ref: process/checks/CP0-CR173.route-plan.json
---

# CR-173 CP4 Story Planning Handoff

## 任务目标

在已批准 CP3 基线下，将 **estimator-only** HLD 收敛为一个最小 Feature、全部必要 Story、Feature 级 DESIGN / TEST-PLAN / TASKS、`DEVELOPMENT-PLAN.yaml` 增量，以及 CP4 Story/DAG/文件所有权预检输入。不得把 public C1 projection、真实数据 activation 或 CR-172 范围拆成 CR-173 Story。

## 已批准的硬边界

- 采用 `spectral_participation_ratio`，claim 仅为 sealed-trial 二阶相关结构的 effective dimensionality。
- 本 CR 只交付 standalone 七字段 evidence、exact-rational deterministic estimator、synthetic/fixture/golden vectors 与 fail-closed 语义。
- public C1 projection Story 数=`0`，公共 C1/Gate-1/admission 生产修改数=`0`。
- 真实数据、strategy identity、provider、credential、runtime、trading、publish/deploy、Git remote write 均未授权。
- CP5 批准前 source/test implementation 数=`0`。

## 上下文策略

- `read_profile=compact`，先读 `process/context/CP3-CR173.context.json`。
- must_read：CR-173、CR173 HLD/ADR/Blueprint/Domain Map/Dependency Map、CR173 当前产品 Story Map/MVP/Release Slice 增量、当前 `FEATURE-DESIGN-MATRIX.md` 与 `DEVELOPMENT-PLAN.yaml` 的结构约定。
- read_if_needed：与拟修改文件直接相关的现有代码/测试目录结构；只读，用于文件布局和所有权，不得实现。
- do_not_read_by_default：`process/archive/**`、无关历史 discussion、完整旧 CR 验证证据。
- 如 capsule 字段不足而展开全文，须把原因写入 read-expansion ledger 或 return summary。

## 允许写入

- `docs/design/FEATURE-DESIGN-MATRIX.md`：CR-173 增量行。
- `docs/features/`：CR-173 estimator Feature 的 DESIGN / TEST-PLAN / TASKS。
- `process/DEVELOPMENT-PLAN.yaml`：CR-173 Story/Wave/DAG/文件所有权增量，保留既有内容。
- `process/stories/STORY-CR173-*.md`：全部 Story 卡片及 design policy；不得写 implementation evidence。
- `process/handoffs/CR173-CP4-META-SE-STORY-PLANNING-RETURN-SUMMARY.md`：返回摘要。

## 机器可验证出口

- Feature 数、Story 数、Wave 数在 DESIGN、TASKS、DEVELOPMENT-PLAN 三处一致。
- REQ-CR173-001..008 和 SCN-CR173-* 覆盖率=`100%`。
- 所有 Story 有 `feature_design_refs`、`lld_policy`、依赖、文件所有权、验收与失败路由。
- DAG 无环、无悬空依赖；同 Wave 文件冲突=`0`。
- public C1 projection / real activation Story=`0`。
- 为 CP4 自动结果给出 PASS / FAIL 建议；若发现必须越界，返回 blocker，不得自行扩 scope。
