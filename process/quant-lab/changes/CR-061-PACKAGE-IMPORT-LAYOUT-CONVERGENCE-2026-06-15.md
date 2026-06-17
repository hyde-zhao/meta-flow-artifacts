---
cr_id: "CR-061"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 package/import/layout、文件所有权、测试入口、兼容别名和潜在 bulk rewrite，必须保持 standard。"
rollback_to: "pre-CR061 package/import/layout baseline"
approval_result: "cp5-approved-gate-only"
created_at: "2026-06-15T07:29:19+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-15T07:52:05+08:00"
source: "USER-20260615-START-CR061-PACKAGE-IMPORT-LAYOUT-CONVERGENCE"
parent_cr: "CR-060"
source_checkpoint: "process/checkpoints/CP8-CR060-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR060-05"
follow_up_type: "migration-sequence"
risk_class: "high"
owner: "host-orchestrator / meta-dev / meta-qa"
revisit_condition: "用户批准 CR061 Batch B CP5 实现授权后，才能进入窄实现；真实 move / rename / bulk import rewrite 仍需后续二次授权。"
acceptance_criteria: "CR061 必须冻结 package/import/layout candidate list、compatibility strategy、preserve-audit allowlist、rollback_ref、dry-run plan 和 no-runtime boundary。"
close_condition: "package/import/layout convergence 完成验证并通过 CP8，或用户选择只关闭设计门禁并分拆后续执行 CR。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-061 Package / Import / Layout Convergence

## 变更描述

CR061 从 CR060 的 docs-only identity convergence 继续推进，目标是为 `quant-lab` 建立生产级 package/import/layout 收敛方案。

本 CR 当前只启动正式 CR、冻结影响分析、candidate list、CP2/CP3/CP5 门禁和执行前置条件；不在本次创建动作中执行任何真实源码移动、目录重命名、批量 import 改写、`pyproject.toml` / `uv.lock` 修改或测试入口替换。

当前状态：用户已对 CR061 CP2/CP3/CP5 门禁回复“同意”，按 `approve` 处理；CR061 gate-only 门禁包批准为 `cp5-approved-gate-only`。随后用户再次回复“同意”，已按继续准备 Batch B 实现授权门禁处理；当前状态为 `active-batch-b-authorization-review-pending`。该状态不授权立即执行 `pyproject.toml`、`uv.lock`、源码目录、import 路径、Git remote、NAS/data lake、runtime、凭据或 CR046 recovery；只有新的 Batch B CP5 人工门禁 approved 后，才允许窄实现。

更新：用户于 2026-06-15T08:55:37+08:00 再次回复“同意”，按 CR061 CP5 Batch B `approve` 解析。当前状态切换为 `active-batch-b-implementation-in-progress`，仅授权窄实现：`pyproject.toml` metadata、`uv.lock` 再生成、可选 `quant_lab` 兼容命名空间和离线 smoke test。仍不授权 physical move / rename / delete、bulk import rewrite、删除 legacy roots、Git remote、NAS/data lake/provider/catalog、凭据、QMT/MiniQMT runtime 或 CR046 recovery。

实现更新：CR061 Batch B 已完成授权内窄实现并通过 CP6 PASS / CP7 PASS_WITH_RISK。`pyproject.toml` project name 已改为 `quant-lab`，`uv.lock` 由 `uv lock` 再生成，新增 `quant_lab` 兼容命名空间和 `tests/test_cr061_package_import_layout.py` 离线 smoke test。当前等待 CP8 delivery readiness，需在 CP8 中显式接受 full pytest 未运行和 bulk relayout 未完成风险。

CP8 更新：CR061 Batch B delivery readiness 已生成，release_decision=`READY_WITH_RISK`，当前等待用户人工终验。CP8 approve 只接受本批交付就绪与风险，不授权真实 release、Git remote、NAS/data lake/provider/catalog、凭据、QMT/MiniQMT runtime、CR046 recovery、physical move 或 bulk import rewrite。

关闭更新：用户于 2026-06-15T09:23:26+08:00 回复“同意，下一步是做什么？”，按 CP8 `approve` 解析。CR061 Batch B 当前交付关闭为 `closed-current-delivery`，release_decision=`READY_WITH_RISK`。下一步推荐启动 CR062 Git remote cutover / repository publication gate；CR061 CP8 不授权真实 release、Git remote、NAS/data lake/provider/catalog、凭据、QMT/MiniQMT runtime、CR046 recovery、physical move 或 bulk import rewrite。

推荐路线是 staged compatibility-first：

- 先保留现有顶层 import roots：`engine`、`market_data`、`strategies`、`trading`。
- 后续经 CP5 批准后，再考虑新增 `quant_lab` compatibility namespace 和 package metadata 收敛。
- 真实 bulk move 到 `src/quant_lab/**` 只作为高风险候选，必须在本 CR 内二次冻结 candidate manifest、rollback ref、dirty worktree include/exclude 和 full regression 后单独授权。

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| 当前 active formal CR | `CR-046` paused | CR046 仍保持用户暂停的 CP6 PASS / ready-for-verification；CR061 不恢复 CR046。 |
| 与 CR046 影响面重叠 | partial / controlled | CR061 会识别 `trading/**` 为 package/import 候选，但当前只做静态 candidate 和 no-runtime design，不触发 runtime、凭据、账户、策略导入或交易动作。 |
| 与 CR060 关系 | follow-up | CR060 关闭 docs-only identity；CR061 承接其 deferred package/import boundary。 |
| 与 CR062 关系 | excluded | Git remote add / push / branch rename / tag 仍归 CR062。 |
| 与 CR063 / CR064 / CR065 关系 | excluded | NAS 数据迁移、data lake root/catalog、QMT/MiniQMT runtime 均不进入 CR061。 |
| 顶层 `active_change` | preserved as `CR-046` | 既有 consistency 脚本要求 CR046 active 时顶层字段保持 CR046；CR061 通过 `cr_tracking.active_crs` 表达为迁移并行推进对象。 |

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-060` |
| 来源检查点 | `process/checkpoints/CP8-CR060-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR060-05` |
| follow-up 类型 | `migration-sequence` |
| 风险等级 | `high` |
| owner | `host-orchestrator / meta-dev / meta-qa` |
| 重访条件 | 完成 CR061 CP2/CP3/CP5 后，用户可批准进入 package/import/layout 实现。 |
| 验收标准 | package/import/layout candidate list、compatibility strategy、rollback_ref、dry-run plan 和 no-runtime boundary 可审计。 |
| 关闭条件 | 通过 CR061 CP8 或分拆后续执行 CR。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `pyproject.toml` | 本轮不变；后续候选 | Git diff + rollback_ref + CR061 manifest | 不适用 | pending |
| `uv.lock` | 本轮不变；后续候选 | `uv lock` 重生成前后 diff + rollback_ref | 不适用 | pending |
| `engine/**` | 本轮不变；后续候选 | 现有 import root 保留为 legacy public root | 不适用 | pending |
| `market_data/**` | 本轮不变；后续候选 | 现有 CLI `python -m market_data.cli` 保留 | 不适用 | pending |
| `strategies/**` | 本轮不变；后续候选 | 现有 import root 保留 | 不适用 | pending |
| `trading/**` | 本轮不变；高风险候选 | CR046 paused boundary + no-runtime guard | 不适用 | pending |
| `tests/**` | 本轮不变；后续候选 | 现有 tests 作为 regression baseline | 不适用 | pending |
| `README.md` / `docs/USER-MANUAL.md` | 本轮不变；后续按实现需要更新 | CR060 已收敛为 quant-lab canonical | CR060 / 后续 CR061 修订段 | pending |
| `process/**` historical evidence | 不变 | preserve-audit | 不适用 | accepted-policy |
| Git remote / NAS / data lake / runtime docs | 不变 | out-of-scope | 不适用 | accepted-policy |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| `local-backtest` project metadata | `quant-lab` distribution metadata | 原文保留至实现批准前 | 只在 CP5 后修改 `pyproject.toml`，并用 `uv lock` 验证。 |
| `engine` import root | optional `quant_lab.engine` compatibility namespace | 保留 legacy root | 后续新 namespace 不应破坏旧测试和脚本。 |
| `market_data` import root | optional `quant_lab.market_data` compatibility namespace | 保留 legacy CLI | `python -m market_data.cli` 继续有效；新入口需独立 smoke。 |
| `trading` import root | optional `quant_lab.trading` compatibility namespace | 保留 no-runtime guard | 只做静态 import/layout，不连接 QMT/MiniQMT。 |
| historical `process/**` references | N/A | preserve-audit | 不批量替换历史事实。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增、删除或重定义迁移需求 | package/import/layout migration | true | 将 CR060 deferred package/import boundary 转为 CR061 正式 CR。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | import smoke、offline pytest、CLI smoke | true | 新增 package/import compatibility smoke 场景；不运行 provider / lake / runtime。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR061 CP2/CP3/CP5/CP6/CP7/CP8 | true | 当前生成 CP2/CP3/CP5 门禁，后续实现必须另有 CP6/CP7。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | `.env`、runtime、Git remote、NAS、lake、trading | true | 全部列为不授权项；`trading/**` 只允许静态 import/layout review。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | release docs、manifest、checks、checkpoints | true | 新增 CR061 candidate list、layout plan、rollback gate、preserve-audit 和 launch messages。 |

## 回退决策

- 影响范围：全局 repo-local package/import/layout。
- 回退到阶段：`pre-CR061 package/import/layout baseline`。
- 需要重新确认的对象：`pyproject.toml`、`uv.lock`、`engine/**`、`market_data/**`、`strategies/**`、`trading/**`、`experiments/**`、`scripts/**`、`tests/**`、README / USER-MANUAL 相关命令。
- 当前 rollback refs：CR059 git bundle / cold backup 仅能支撑 planning 和 docs gate；真实实现前必须刷新 include/exclude manifest，必要时刷新 git bundle。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | package/import/layout 影响代码和测试入口。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 包边界、兼容 namespace、运行边界需要架构决策。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | import roots 和 CLI entrypoint 是跨模块契约。 |
| 需要 HLD / LLD 才能解释影响 | true | 需要 staged compatibility-first 设计。 |
| 是否保持 fast-lane | false | CR061 必须 standard。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR061-PACKAGE-IMPORT-LAYOUT-BATCH-A`
- 批次范围来源：CR061 影响分析 / 用户启动迁移序列
- 批次内 Story：
  - `CR061-S01-package-metadata-and-uv-lock-gate`
  - `CR061-S02-quant-lab-compatibility-namespace-design`
  - `CR061-S03-import-root-and-cli-compatibility-dry-run`
  - `CR061-S04-tests-and-offline-regression-gate`
  - `CR061-S05-bulk-move-deferred-authorization-boundary`
- 批次人工确认稿：`process/checkpoints/CP5-CR061-PACKAGE-IMPORT-LAYOUT-READINESS.md`
- 开发启动条件：
  - [ ] 批次内全部 Story 设计证据已输出或在 CP5 中明确 waived。
  - [ ] CP5 自动预检通过。
  - [ ] CP5 人工确认结论为 `approved`。
  - [ ] 真实 move / rename / bulk import rewrite 已通过二次执行授权。
  - [ ] dirty worktree include/exclude manifest 已冻结。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR061 并冻结门禁包 | CR060 handoff / STATE / CR-INDEX | 本 CR、candidate list、CP2/CP3/CP5 | 当前已完成文件创建 | 等待人工确认 |
| 2 | `meta-dev` 或 approved inline fallback | 设计 package/import/layout 实现批次 | CP5 approved + candidate list | Story 设计证据 / 实现计划 | 不得直接 move / rewrite | 交回 host |
| 3 | `meta-dev` | 执行批准范围内的 package/import changes | 二次授权 / rollback_ref / dirty manifest | 代码 / package metadata / tests | CP6 | 交回 QA |
| 4 | `meta-qa` | 离线验证 | CP6 evidence | CP7 verification | 不运行 provider / lake / runtime | 交回 host |
| 5 | `host-orchestrator` | 发布准备与关闭 | CP7 / release docs | CP8 | 用户 approve | 关闭或分拆后续 CR |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：CP8
- 自动通过条件：N/A
- 授权原文：N/A
- 授权时间：N/A
- 回填要求：必须等待人工确认。

## 后续事项台账

- 是否存在后续事项：true
- 台账路径：`process/changes/CR-053-FOLLOW-UP-TRACKING-2026-06-14.md`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .`
- 状态取值：`active` / `candidate` / `closed` / `blocked`

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| CR-061 | Package / Import / Layout Convergence | active | CR | 1 | `process/changes/CR-061-PACKAGE-IMPORT-LAYOUT-CONVERGENCE-2026-06-15.md` | CR046 paused; no runtime overlap allowed | CP5 approved gate-only | 真实实现授权未完成 | 准备实现授权 gate / Batch B 设计。 |

## 处理结论

- 审批结论：`cp5-approved-gate-only`
- [ ] 自动批准（低风险）
- [x] 人工确认已通过（门禁包）
- [x] 待后续实现授权（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-060` | docs-only identity convergence 已关闭，package/import deferred。 |
| Handoff | `process/context/POST-CR060-MIGRATION-HANDOFF-CONTEXT.yaml` | 清上下文恢复包。 |
| Candidate list | `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md` | CR061 候选对象与动作分类。 |
| Layout plan | `docs/release/REPO-LOCAL-PACKAGE-LAYOUT-PLAN-CR061.md` | staged compatibility-first package/import/layout 方案。 |
| Rollback ref | `docs/release/ROLLBACK-REF-CR061.md` | 实现前 rollback / dirty worktree gate。 |
