---
status: baseline
version: "1.7"
created_at: "2026-07-11"
owner: "meta-pm"
active_change_ref: "CR-051"
source_story_map: "process/docs/product/STORY-MAP.md"
---

# Meta Flow 产品发布切片

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.2 | 2026-07-13 | host-orchestrator-inline-fallback | 增量追加 CR-047 SL-WT-01..03，保留 CR-046 与既有切片。 | 原文档增量更新 |
| 1.3 | 2026-07-15 | host-orchestrator inline fallback | 增量追加 CR-050 SL-GB-01..03，保留全部既有切片和 Story 回链。 | 原文档增量更新 |
| 1.4 | 2026-07-16 | host-orchestrator inline fallback | 增量追加 SL-GB-04 显式两仓 fast-forward merge，并把既有 SL-GB-03 cleanup 的前置改为 SL-GB-04 2/2 PASS；不重编号历史切片。 | 原文档增量更新；等待 CP2 R2 |
| 1.5 | 2026-07-17 | meta-pm | 增量追加CR-051 SL-AW-01..04，覆盖project-first兼容、per-project worktree、项目作用域Git周期/shared-main refresh和零迁移交接；保留全部既有切片。 | 原文档增量更新；等待CP2 |
| 1.6 | 2026-07-17 | meta-pm | CR-051 CP2 R2：Slice ID不变；修订SL-AW-02/03为长期项目integration、短期CR branch、shared main、显式merge-main与finish/abort回归，并记录DQ-01..03已resolved-by-user。 | 原文档增量更新；等待CP2 R2总体approve |
| 1.7 | 2026-07-18 | meta-pm | CR-051 CP2 R3：保持SL-AW-01..04不变，当前切片改为integration create-only、异构source/artifact双leg、单一aggregate gate与CR外人工main/integration同步；DQ-02标记superseded。 | 原文档增量更新；等待CP2 R3总体approve |
| 1.0 | 2026-07-11 | meta-pm | 建立独立 release-slice 目录基线；引用既有 SL-PG 切片，并增量记录 CR-046 SL-EI 切片 | 缺失产物初始化；不改变 STORY-MAP 中既有 ID |
| 1.1 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：扩展既有 SL-EI 切片的 compaction/correction/audit/dogfooding/dispatch 验证入口，不改变 Slice 或 Story ID | 原文档增量更新 |

## 既有切片引用

`SL-PG-00..05` 的正式定义继续保留在 `docs/product/STORY-MAP.md`；本文件不复制或重新编号既有切片。后续产品变更应逐步把新切片增量登记在此文件，并保持对 Story Map 的回链。

## CR-046 推荐切片

| Slice ID | 用户价值 | 包含 Story | 前置依赖 | 验证入口 | 风险 / 门控 |
|---|---|---|---|---|---|
| SL-EI-01 | 审计者可以机器证明 gate 时序、真实 dispatch、final CP correlation、compaction 语义与 state/read 一致性 | ST-EI-001..004 | CP2、CP3、全量 CP5 | TC-EI-001..009, TC-EI-014, TC-EI-018 | receipt 缺失必须披露 session/repository 证明上限；compaction 不得丢失关系；不得隐式授权 runtime/credentials |
| SL-EI-02 | 审批者可以区分实测/估算/不可用成本，用机器 audit report 和双口径 replay 复核历史证据 | ST-EI-005..006 | SL-EI-01 | TC-EI-010..012, TC-EI-016..017 | checker provenance 缺失需诚实降级；R1 null-provenance 不得静默改写；legacy YAML 不得干扰 canonical JSON |
| SL-EI-03 | 以通用 post-close correction lifecycle 和真实 CR-163 样本证明 append-only migration/current replay | ST-EI-007 | SL-EI-02；pilot 独立授权 | TC-EI-013, TC-EI-015 | 23/23 PASS；历史不可改写；quant-lab lineage business diff 为 0 |

## CR-047 推荐切片

| Slice ID | 用户价值 | 包含 Story | 前置依赖 | 验证入口 | 风险 / 门控 |
|---|---|---|---|---|---|
| SL-WT-01 | 不同设备得到同一 workflow truth 与 CR-046 当前事实 | ST-WT-001, ST-WT-002, ST-WT-007 | CP2、CP3 | TC-WT-001,002,007 | 不改写历史；不处理 prelink backup |
| SL-WT-02 | 发布前 Doctor/guardrail/Ruff/pytest 可确定判定 | ST-WT-003..005 | SL-WT-01、CP5 | TC-WT-003..005 | warning 语义和规则 source 需 CP2/CP3 决策 |
| SL-WT-03 | CI/Agent 可按 README 完成三平台 dry-run | ST-WT-006 | SL-WT-02 | TC-WT-006 | 不执行真实用户级安装或发布 |

## CR-050 推荐切片

| Slice ID | 用户价值 | 包含 Story | 前置依赖 | 验证入口 | 风险 / 门控 |
|---|---|---|---|---|---|
| SL-GB-01 | 源码与过程证据从各自最新主分支进入同名、可恢复 CR branch | ST-GB-001 | CP2、CP3、全量 CP5 | TC-GB-001..003, TC-GB-010 | remote push 需独立授权；不 reset/rebase/force；双仓不宣称原子 |
| SL-GB-02 | 只发布显式提交的 refs，并逐仓核验远端 OID | ST-GB-002 | SL-GB-01 | TC-GB-004..005, TC-GB-009..010 | 不隐式 stage/commit；partial 不得冒充 PASS |
| SL-GB-04 | 通过独立授权把两仓 exact published tip 以 fast-forward-only 更新到 remote default | ST-GB-004 | SL-GB-02；default-branch write 独立授权 | TC-GB-012..017, TC-GB-009..010 | artifact→project；不创建 merge commit/force/策略绕过；partial 保留两仓 branch并阻断 finish |
| SL-GB-03 | merge 后重新证明 exact tip/ancestry，再清理目标 branch | ST-GB-003 | SL-GB-04 2/2 PASS；delete 独立授权 | TC-GB-006..011, TC-GB-015, TC-GB-017 | merge 结论不是删除授权；squash/rebase 无 receipt 时 BLOCKED；protected ref 不删除 |

## CR-051 推荐切片

| Slice ID | 用户价值 | 包含 Story | 前置依赖 | 验证入口 | 风险 / 门控 |
|---|---|---|---|---|---|
| SL-AW-01 | 每个项目可唯一解析自己的docs/process，并在迁移前兼容legacy布局 | ST-AW-001 | CP2、CP3、全量CP5 | TC-AW-001..003,010,012 | 多写目标fail closed；不移动文件/链接；layout切换需显式metadata |
| SL-AW-02 | 每项目拥有稳定、可检查的独立artifact worktree，缺失integration可从fresh `origin/main` exact OID create-only初始化 | ST-AW-002 | SL-AW-01；CP2-DQ-01/03/06 resolved；待CP3/CP5 | TC-AW-004..005,007,011,014..015 | existing control+sibling root；既有integration不recreate/reset/orphan；真实worktree mutation不授权 |
| SL-AW-03 | 多项目可并行执行异构source-default/artifact-integration双leg，并以单一aggregate gate完成逻辑CR | ST-AW-003, ST-AW-004 | SL-AW-02；CP2-DQ-04/05 resolved，DQ-02 superseded；待CP3/CP5 | TC-AW-005..010,012,014 | 仅全PASS完成；PARTIAL仅进度；integration OID漂移阻断；per-CR artifact main mutation与自动回滚为0 |
| SL-AW-04 | 用户获得逐项目迁移manifest、验证和回滚清单，而本CR不发生真实迁移 | ST-AW-005 | SL-AW-03 | TC-AW-012..013,015 | 真实文件/link/ref mutation为0；后续项目迁移需独立授权 |

## 切片边界

- 每个 Story 保持独立 CP6/CP7 verdict；共享 regression 可以批量运行，但不合并 Story 结论。
- CP2 approval 只确认产品/场景/范围，不授权实现、runtime、credentials、publish、commit/push 或 quant-lab business-code 修改。
- `SL-EI-03` 失败时保留 Meta Flow 已验证切片，pilot 回退到设计澄清或实现回修，不覆盖历史证据。
- CR-047 使用用户批准的 inline fallback；该授权不等于 CP2、CP3、CP5 或 CP8 批准。
- CR-050 延续用户“不使用子 Agent”的约束；CP7/CP8 结论上限应显式披露独立 QA 风险。
- CP2 R2 只确认 CR-050 产品边界，不授权真实 commit/push/default-branch update/delete；真实 Git remote mutation 必须在后续门禁和对应操作的独立显式授权后执行。
- CR-051 R3 pending decision items=0：DQ-01/03/04/05/06 resolved-by-user，DQ-02 superseded-by-user；CP2 R3总体基线仍待approve。即使approve也只确认产品/场景/范围，不授权HLD门禁跳过、源码实现、真实artifact迁移、main↔integration同步、软链接变更或真实Git/worktree/ref操作。
- SL-AW-04是“迁移交接能力”而不是“迁移执行切片”；实际逐项目迁移不计入CR-051完成范围。
