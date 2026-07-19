---
checkpoint_id: "CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2"
checkpoint_name: "CR-052 vNext R2 需求/场景/范围基线预检"
type: "auto_precheck"
status: "PASS"
owner: "meta-pm"
created_at: "2026-07-19T09:04:51Z"
checked_at: "2026-07-19T09:04:51Z"
result_ref: "process/checks/CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2.result.json"
manual_checkpoint: "由 host-orchestrator 另行生成，不在本 handoff 单写范围"
target:
  phase: "requirement-clarification"
  artifacts:
    - "process/docs/product/REQUIREMENTS.md"
    - "process/docs/product/SCENARIOS.yaml"
    - "process/docs/product/TEST-MATRIX.md"
    - "process/docs/product/STORY-MAP.md"
    - "process/docs/product/MVP-SCOPE.md"
    - "process/docs/product/RELEASE-SLICES.md"
    - "process/docs/product/BACKLOG.md"
---

# CP2 CR-052 vNext R2 需求 / 场景 / 范围基线预检

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP1 通过 | PASS | `CP1-CR052-USE-CASE-COMPLETENESS-VNEXT-R2.result.json` | decision=PASS |
| 需求草案存在 | PASS | `REQUIREMENTS.md` | 32 条 REQ-VNEXT；ready_for_design=false |
| 工程场景/矩阵存在 | PASS | `SCENARIOS.yaml` / `TEST-MATRIX.md` | 20 个 TC-VNEXT；P0/P1 未覆盖=0 |
| 产品规划输入存在 | PASS | `STORY-MAP.md` / `MVP-SCOPE.md` / `RELEASE-SLICES.md` / `BACKLOG.md` | 6 个候选 Story + A-D 切片 |
| 场景讨论证据存在 | PASS | discussion log/checkpoint | 4 个灰区 + SGQ-VNEXT-001 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 功能需求完整 | PASS | 21 个功能 + 6 个约束 | UC-VNEXT-001..006 全覆盖 |
| 2 | 非功能需求量化 | PASS | 5 个 NFR + SM-VNEXT-01..12 | route=1、change=0、预算、CAS、试点均可检验 |
| 3 | 范围清晰 | PASS | IN/OUT/DEF-VNEXT | 首版不拆史、不无损转换、不自动merge、不批量迁移 |
| 4 | 验收标准明确 | PASS | REQ-VNEXT + TC-VNEXT | 32/32 需求有场景回链 |
| 5 | 约束条件记录 | PASS | REQ-VNEXT-C001..006 | CP2 不授权实现、迁移、push/runtime/production |
| 6 | 依赖和风险识别 | PASS | RA-VNEXT-001..006 | 路由、单写吞吐、预算 underfit、快照/投影风险均有缓解 |
| 7 | 需求无未分类冲突 | PASS | DQ-VNEXT-01..05 | 5 项待用户决策；未分类阻断问题=0 |
| 8 | 变更机制明确 | PASS | 八份文档修订记录 | 所有历史 ID/版本保留，vNext 为增量 reframe |
| 9 | 追溯矩阵建立 | PASS | TEST-MATRIX vNext 索引 | REQUEST→6 UC→32 REQ→20 TC→6候选Story→A-D |
| 10 | Scenario Gray Areas 已处理 | PASS | discussion log | 4/4 灰区完成场景级处理并路由决策项 |
| 11 | Deferred Ideas 已隔离 | PASS | BACKLOG / MVP | 5 项均有延后原因和重启条件 |
| 12 | 用户可见场景确认已完成 | PASS | SGQ-VNEXT-001 | 只确认场景方向，不伪造 CP2 批准 |
| 13 | 8 维扫描后台化 | PASS | USE-CASES 附录 | D1-VNEXT..D8-VNEXT 全部处理 |
| 14 | 工程验证场景可追踪 | PASS | 20 TC + 矩阵 | 五类场景覆盖；所有 REQ-VNEXT 有 TC |
| 15 | MVP 范围可确认 | PASS | Story Map/MVP/Release/Backlog | A-D 当前顺序；MR 历史切片 superseded |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 致命问题=0、阻塞问题=0 | PASS | result JSON blockers=[] | 产品基线具备人工决策条件 |
| 待人工决策已完整收集 | PASS | DQ-VNEXT-01..05 | 5 项都有推荐、备选、影响和切换条件 |
| 人工确认完成 | BLOCKED | host-orchestrator 后续 checkpoint | 自动预检 PASS 不等于 CP2 approved |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| 八份产品基线 | `process/docs/product/*` | PASS | 旧 ID 保留；vNext 当前候选完整 |
| CP1 机器结果/摘要 | `process/checks/CP1-CR052-USE-CASE-COMPLETENESS-VNEXT-R2.*` | PASS | 新文件，不覆盖旧 v1.1 |
| CP2 机器结果/摘要 | `process/checks/CP2-CR052-REQUIREMENTS-BASELINE-VNEXT-R2.*` | PASS | 自动预检；人工门 pending |
| Discussion log/checkpoint | `process/discussions/CP2-CR052-VNEXT-R2-SCENARIO-DISCUSSION-LOG.md` / `process/checks/CP2-CR052-VNEXT-R2-DISCUSSION-CHECKPOINT.json` | PASS | SGQ 和 DQ 可审计 |

## 待人工决策摘要

| 决策 ID | 类型 | 推荐方案 |
|---|---|---|
| DQ-VNEXT-01 | architecture | 复用现有发布仓，每项目只新增独立过程仓 |
| DQ-VNEXT-02 | architecture | 过程 main-only 单写 publisher + expected-OID CAS |
| DQ-VNEXT-03 | scope | 只迁当前快照，旧仓永久只读索引，不拆史/不无损转换 |
| DQ-VNEXT-04 | implementation | G0=8/8/3/32k；G1=20/24/8/96k；高风险/超限进G2 |
| DQ-VNEXT-05 | scope | 2项目×2周期、每项目G0/G1；旧仓观察期推荐30天 |

## 结论

- 自动预检：`PASS`
- 阻断项：0
- 豁免项：0
- 人工门：`pending`（5 个 DQ + 总体基线）
- 下一步：交还 host-orchestrator 生成 `process/checkpoints/*VNEXT-R2*` Decision Brief 并发起 CP2；本 Agent 不修改 checkpoint/state/ledger。
