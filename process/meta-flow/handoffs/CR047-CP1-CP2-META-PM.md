---
handoff_id: "CR047-CP1-CP2-META-PM"
workflow_id: "meta-flow-self-dev"
change_id: "CR-047"
from_agent: "host-orchestrator"
to_agent: "meta-pm"
stage: "requirement-clarification"
context_ref: "process/context/CP0-CR047.context.json"
status: "completed"
dispatch:
  mode: "inline-fallback"
  canonical_role: "meta-pm"
  codex_agent_name: "meta-pm"
  reasoning_profile: "default"
  dispatch_trigger: "CR-047 product-baseline refresh and CP2 preparation"
  tool_name: "spawn_agent"
  agent_id: "/root/pm_wu"
  thread_id: "/root/pm_wu"
  spawned_at: "2026-07-13T03:05:17Z"
  replacement_agent_id: "/root/pm_zheng"
  interruption_reason: "Three bounded product-baseline turns produced only partial USE-CASES/REQUIREMENTS updates and no CP1/CP2 return."
  approved_by: "user"
  approved_at: "2026-07-13T03:40:39Z"
  fallback_reason: "User explicitly instructed: 不使用子agent，继续推进."
  completed_at: "2026-07-13T03:40:39Z"
---

# CR-047 CP1 / CP2 meta-pm Handoff

## 目标

把已确认的治理整改范围增量写入产品基线，形成可验证的 CP1 与 CP2 输入；不得实现代码、修改架构或提前打开 CP2 人工门。

## 必读

- `process/state/STATE.current.json`
- `process/changes/summaries/CR-047.summary.json`
- `process/checks/CP0-CR047.route-plan.json`
- `process/context/CP0-CR047.context.json`
- `process/policies/READ-POLICY.json`

## 按需读取

- `process/changes/CR-047.md`：仅为五维影响和文档处理决策；记录 `full_doc_read_reason=deep_review`。
- `docs/product/*`：仅更新 CR-047 关联章节，保留旧基线和修订记录。
- 本次用户审计摘要：事实已固化在 CR-047 scope，不需要读取会话 transcript。

## 固定范围

1. canonical JSON CR tracking 与 legacy YAML 迁移。
2. process / artifact / docs 路由和 clean-clone 一致性。
3. Doctor budget、policy、read-expansion 治理。
4. 根规则 canonical source 与 guardrail clean-clone 契约。
5. Ruff 静态质量门。
6. README 非交互安装示例与 cache preflight。
7. CR-046 产品矩阵/验证状态收敛，但不得改写历史时序。

## 不授权

- credentials、runtime、SaaS、production write、publish、trading、repository commit/push。
- 不得把 fixture rejection 写成真实平台 receipt。
- 不得删除或处理 `/home/hyde/workspace/meta-flow.process-prelink-backup-20260713T100930`。

## 交付

- 增量更新 `docs/product/USE-CASES.md`、`REQUIREMENTS.md`、`SCENARIOS.yaml`、`TEST-MATRIX.md`、`STORY-MAP.md`、`MVP-SCOPE.md`、`RELEASE-SLICES.md`、`BACKLOG.md`。
- `process/discussions/CP2-CR047-SCENARIO-DISCUSSION-LOG.md` 和 `process/checks/CP2-CR047-DISCUSSION-CHECKPOINT.json`；本次用户“开始推进这个 CR”可作为范围启动确认，但不得伪造未发生的选择题回答。
- CP1 result/summary、CP2 自动预检 result/summary、CP2 context capsule、CP2 人工审查稿草案。
- `process/handoffs/CR047-CP1-CP2-META-PM-RETURN-SUMMARY.md`，列出 Decision Brief 候选项与所有未决灰区。

## Context Policy

- `read_profile=compact`
- capsule first；默认禁止 archive、全部历史 CR、全部历史 discussion。
- 若全文展开，追加 `process/state/READ-EXPANSION-LEDGER.ndjson`，不得仅在摘要中声称已记录。

## 调度结果

- `/root/pm_wu`：两次 turn 均在部分产品基线写作后卡住，已中断，未声称完成。
- `/root/pm_zheng`：替代 turn 只追加读取记录，未交付剩余产物，已中断。
- 保留的部分产物：`docs/product/USE-CASES.md`、`docs/product/REQUIREMENTS.md` 增量草稿。
- 用户随后明确批准不使用子 agent；Host 按同一 handoff 合同完成剩余产品基线和 CP1/CP2 草案。
- CP1 结论：`PASS`；CP2 自动预检：`PASS`，等待人工范围决策。
