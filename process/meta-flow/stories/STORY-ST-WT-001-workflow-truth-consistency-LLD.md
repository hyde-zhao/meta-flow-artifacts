---
story_id: "ST-WT-001"
title: "校验 State、CR 与 CURRENT workflow truth"
story_slug: "workflow-truth-consistency"
lld_version: "1.0"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-14"
confirmed_by: "user"
confirmed_at: "2026-07-15T12:05:17Z"
feature_design_refs: ["process/docs/features/cr047-truth/DESIGN.md", "process/docs/features/cr047-truth/TEST-PLAN.md", "process/docs/features/cr047-truth/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-module", "state-lifecycle", "shared-contract"]}
open_items: 0
---

# LLD: ST-WT-001 — Workflow Truth Consistency

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline/meta-dev | 冻结 State/CR/CURRENT 只读关系校验、失败码与回归边界。 |

## 0. 工程依据与上游设计依据

工程依据为 accepted HLD、ADR-WT-001、Feature truth design、Story 卡和当前 `cr_tracking.py`/`state/current.py`。State、CR catalog 与 CURRENT 各自单写；本 Story 只新增关系校验，不创建总状态或反向 writer。

## 1. 目标

对 `STATE.current.json`、canonical `CR-INDEX.json`/CR frontmatter、`CURRENT.json` 建立确定性一致性检查，使 active change 指向 closed/cancelled/superseded CR、缺失 CR、或 CURRENT 与 State 漂移的 fixture 100% 被拒绝。

## 2. 需求（Functional / Non-Functional）

- 优先读取 `process/state/STATE.current.json`；`STATE.md` 仅为人类摘要/fallback。
- active change 必须存在于 JSON index，且 lifecycle 不得为 closed/cancelled/superseded；idle 可为空。
- `process/current/CURRENT.json` 的 change/checkpoint/context refs 必须与 State 可推导投影一致；缺失或断链为结构化 finding。
- legacy `CR-INDEX.yaml` 与 canonical JSON 同目录并存时 fail-closed，不自动删除。
- finding 排序、退出码和同输入输出保持 100% 确定；只读检查不改 State/CR/CURRENT。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| `meta_flow/checks/cr_tracking.py` | 加载 canonical CR catalog/lifecycle 并执行 active-change 关系规则 |
| `meta_flow/state/current.py` | 解析 State v2 与 CURRENT 投影，返回 typed snapshot/finding |
| `meta_flow/workflow/cr_lifecycle.py` | 复用 lifecycle 枚举/归一化，不增加 writer |
| `meta_flow/cli.py` | 复用 `check cr-tracking`/`state check` 输出和退出码 |
| `tests/test_cr047_truth_consistency.py` | 正向、负向、迁移与确定性 fixture |

## 4. 代码结构与文件影响范围

修改 `cr_tracking.py`、`state/current.py`；必要时最小接线 `cr_lifecycle.py`/`cli.py`。创建 `tests/test_cr047_truth_consistency.py`。不修改历史 CR-046 证据、ledger 或 archive；不创建新 truth 文件。

## 5. 数据模型与持久化设计

`TruthSnapshot` 含 `active_change`、`phase`、`pending_gate`、`cr_lifecycle`、`current_refs`；`TruthFinding` 含稳定 code、severity、source_ref、actual、expected。错误码至少包括 `ACTIVE_CR_MISSING`、`ACTIVE_CR_TERMINAL`、`CURRENT_MISSING`、`CURRENT_STATE_MISMATCH`、`DUAL_CR_INDEX`。对象均为内存派生，无新增持久化。

## 6. API / Interface 设计

`load_truth_snapshot(project_root) -> TruthSnapshot`；`validate_truth_relations(snapshot) -> list[TruthFinding]`；现有 CLI 将 finding 渲染为文本并以非零退出。writer 仍由 `state write`、`current-refresh`、`cr status-sync` 各自拥有；checker 不调用 writer。

## 7. 核心处理流程

1. 解析 State v2 schema；识别 idle/active。
2. 仅加载 canonical JSON index及 active CR frontmatter，检测 dual-index。
3. 解析 CURRENT.json 和 `.ref` 目标，不跟随到 project root 外。
4. 对 lifecycle、gate、projection refs 做关系校验。
5. 按 `(severity, code, source_ref)` 稳定排序，返回结果；不自动修复。

失败路径：输入缺失/坏 JSON直接 BLOCKER；legacy-only 状态进入显式迁移 finding；任何写入尝试视为测试失败。

## 8. 技术细节与设计细节

active 状态定义由 CR lifecycle 既有枚举提供，不用字符串包含猜测。CURRENT 是可重建投影，允许 writer 刷新，但 checker 只能给出预期 ref。symlink 解析后必须仍位于健康的 project/process 路由。为 CR-037 closed-but-active 建专门回归 fixture。

## 9. 安全与性能设计

所有路径以 `project_root.resolve()` 为锚并拒绝越界；不读取凭据/runtime/quant-lab/backup。复杂度为 CR index `O(n)` 加固定 current refs；同一输入重复 10 次的序列化结果必须一致。

## 10. 测试设计

| 场景 | 预期 |
|---|---|
| active CR 存在且非终态、CURRENT 一致 | 0 finding、exit 0 |
| CR-037 closed 仍 active | `ACTIVE_CR_TERMINAL`、exit 非零 |
| active CR 不在 JSON index | `ACTIVE_CR_MISSING` |
| CURRENT 缺失/断链/指向其他 change | 专用 finding，拒绝 |
| JSON+YAML 双 index | `DUAL_CR_INDEX` |
| idle 且无 active change | 合法 |
| check 两次 | finding 顺序/JSON 完全相同 |
| 既有 state/cr tests | 无回归 |

## 11. 实施步骤

1. `TASK-WT-001-01`：提取 typed snapshot 与 lifecycle 关系规则。
2. `TASK-WT-001-02`：接入 CURRENT/state-v2 优先读取与稳定 finding。
3. `TASK-WT-001-03`：补 CR-037 漂移、dual-index、断链、idle 和回归测试。
4. 运行定向 pytest、`state check --mode enforce`、`check cr-tracking`；写 Implementation/Return/Evidence，不修改输入证据。

## 12. 风险、难点与预研建议

风险是兼容模式被误判为 canonical、checker 误调用 writer、symlink 越界。以显式 schema version、依赖方向测试、临时目录 fixture 缓解。无 OPEN/Spike；若既有 lifecycle 无法表达状态，回 CP5 设计澄清。

## 13. 回滚与发布策略

先在测试中冻结错误码，再接现有检查入口。若合法 workspace 被误拒绝，回滚调用接线并保留 fixture；不回写任何 State/CR/CURRENT。发布不涉及迁移、外部写或 commit/push。

## 14. DoD（Definition of Done）

- [ ] 0–14 章节完整、`open_items=0`。
- [ ] 五类错误码和 idle 正例均有测试，非法拒绝率=100%。
- [ ] CR-037 closed-active 回归 fixture 通过。
- [ ] State/CR/CURRENT 输入 hash 前后不变，新增 truth source=0。
- [ ] 定向与既有回归通过，finding 确定性=100%。
- [ ] CP5 未批准前 `confirmed=false`，不开始实现。

## 人工确认区

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-15T12:05:17Z
- 修改意见：
- 风险接受项：inline fallback；CP7/CP8 风险上限。
