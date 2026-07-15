---
story_id: "ST-WT-007"
title: "以 protected-object firewall 收敛 CR-046 current status"
story_slug: "cr046-status-firewall"
lld_version: "1.0"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-14"
confirmed_by: "user"
confirmed_at: "2026-07-15T12:05:17Z"
feature_design_refs: ["process/docs/features/cr047-operator-status/DESIGN.md", "process/docs/features/cr047-operator-status/TEST-PLAN.md", "process/docs/features/cr047-operator-status/TASKS.md", "process/docs/features/cr047-truth/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["protected-history", "object-identity", "evidence-ceiling"]}
open_items: 0
---

# LLD: ST-WT-007 — CR-046 Status Firewall

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline/meta-dev | 冻结对象身份manifest、三时点校验、允许写集合与越界子CR路由。 |

## 0. 工程依据与上游设计依据

工程依据为 CP3-DQ-05/06、ADR-WT-005、Feature truth/operator designs与CR-046现有closed/READY_WITH_RISK事实。目标只允许current projection和append-only correction，不重新裁决或倒填CR-046 chronology。

## 1. 目标

在 CP6 pre-implementation 构建CR-046 protected-object manifest，CP6完成首验、CP7再验，确保 protected original hash changes=0；仅把current projection收敛为closed/READY_WITH_RISK。任一原件变化立即阻断并拆子CR。

## 2. 需求（Functional / Non-Functional）

- manifest对象身份必须含 `path+object_type+original_sha256+immutable+allowed_operation`，禁止仅路径前缀识别。
- protected集合覆盖CR-046历史时序字段、既有CP results、Story Implementations、CP7 results、evidence-index originals、ledger event prefix。
- allowed writes仅为CR-047自身证据、current product/status projection、新append-only correction/index/hash。
- CP6 pre生成baseline；CP6 completion和CP7重算；任一缺失/变化/未授权路径均BLOCK。
- 无独立QA时CP7≤PASS_WITH_RISK、CP8≤READY_WITH_RISK；receipt/token/pilot/working-tree风险独立叠加。

## 3. 模块拆分与职责

| 模块/对象 | 职责 |
|---|---|
| `process/evidence/CR047-ST-WT-007.protected-originals.json` | CP6 pre生成的对象身份manifest |
| ST-WT-007 implementation/check result | 记录生成器、输入refs、CP6首验与差异 |
| CP7 verification/evidence index | 再验hash、projection和风险上限 |
| `process/docs/product/TEST-MATRIX.md`等current docs | 允许的当前状态投影，不改历史原件 |
| `meta_flow/checks/cr_tracking.py` | 消费ST-WT-001 truth contract；shared最小接线 |

## 4. 代码结构与文件影响范围

CP5仅批准设计，不创建manifest。CP6 pre创建manifest和相关CR-047 evidence；可能更新current product/status说明与测试。禁止修改任何manifest标记immutable的CR-046对象、原ledger前缀、prelink backup、quant-lab。

## 5. 数据模型与持久化设计

manifest顶层含 schema_version、cr_id、story_id、generated_at、source_index_refs、objects。每个object含path、object_type、original_sha256、immutable=true、allowed_operation=`read|reference`、identity_source_ref。verification含stage、checked_at、observed_sha256、match、finding。manifest生成后本身进入Story evidence index，不得重写baseline；修正需superseding manifest并保留旧hash。

## 6. API / Interface 设计

`build_protected_manifest(project_root, source_refs) -> manifest` 仅在CP6 pre调用；`verify_protected_manifest(manifest, stage) -> findings` 在CP6 completion/CP7调用；`classify_changed_path(path, identity_set, allowed_writes) -> protected|allowed|unauthorized`。路径前缀只可用于候选发现，不能作为身份/豁免结论。

## 7. 核心处理流程

1. CP6 pre从CR-046 evidence/index/CP/Story/ledger refs解析具体对象。
2. 对每个真实文件计算SHA256并写对象身份manifest；解析失败即阻断。
3. 执行只允许的projection/correction变更，收集changed paths。
4. CP6 completion重算全部hash并核对未授权paths；零差异才可进入verification。
5. CP7独立重算并验证projection/risk ceiling。
6. 若原件变化：停止、保留差异证据、状态BLOCKED/NEEDS_DESIGN_CLARIFICATION、创建child CR，不在CR-047修复原件。

## 8. 技术细节与设计细节

对象identity keyed by规范化project-relative path+object_type+original hash；同目录CR-046/047混居不构成身份。ledger保护使用具体event identities/prefix hash与原文件hash，新增append-only event合法但旧prefix byte/hash必须不变。allowed writes为显式对象/模式并由changed-path审计消费，不能覆盖protected identity。

## 9. 安全与性能设计

hash读取只限manifest对象和允许current docs；拒绝symlink逃逸。全程不处理凭据/runtime/SaaS/backup/quant-lab。复杂度为protected bytes线性哈希；每阶段输出对象级差异但不复制完整敏感内容。

## 10. 测试设计

| 场景 | 预期 |
|---|---|
| 完整对象身份manifest | 必填字段100%，prefix-only=0 |
| 同目录CR-047新文件 | allowed，不误伤CR-046 |
| protected文件1字节变化 | CP6/CP7 BLOCK、child-CR route |
| protected对象缺失/identity解析失败 | BLOCK |
| 仅新增append-only correction | 原prefix hash不变、允许 |
| current projection | closed/READY_WITH_RISK |
| 独立QA缺失 | verdict ceiling正确 |
| 任一继承风险仍open | CP8不超过READY_WITH_RISK |

## 11. 实施步骤

1. `TASK-WT-007-01`（CP6 pre）：从显式refs生成manifest、验证字段/边界、登记baseline hash。
2. `TASK-WT-007-02`：仅更新允许projection/correction并记录changed paths。
3. `TASK-WT-007-03`：CP6 completion首验；通过后CP7再验和风险上限检查。
4. 任一protected变化立即停止并按ADR-WT-005建子CR；不得自动restore掩盖越界。

## 12. 风险、难点与预研建议

最大风险是用目录前缀替代对象身份、manifest漏对象、自动restore掩盖违规、独立QA缺失却声称READY。通过显式source refs、required object classes、保留差异、机器上限解决。无OPEN；触碰原件需求本身就是切换条件。

## 13. 回滚与发布策略

projection/correction可通过新append-only supersession回滚，protected original永不作为回滚目标。manifest/checker异常时停止ST-WT-007，不降低验证规则。子CR需独立HLD/审批/原hash/replacement语义。

## 14. DoD（Definition of Done）

- [ ] 0–14 章节完整、`open_items=0`。
- [ ] manifest字段覆盖=100%，prefix-only identities=0。
- [ ] CP6 completion与CP7两次 `protected_original_hash_changes=0`、`unauthorized_cr046_path_changes=0`。
- [ ] projection=closed/READY_WITH_RISK，7/7 Story与五项继承风险未美化。
- [ ] 越界fixture 100%阻断并路由child CR。
- [ ] CP5批准前manifest不存在且 `confirmed=false`；实现未开始。

## 人工确认区

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-15T12:05:17Z
- 修改意见：
- 风险接受项：inline fallback；CP7/CP8 风险上限。
