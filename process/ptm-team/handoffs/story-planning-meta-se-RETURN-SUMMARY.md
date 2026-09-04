# story-planning 交还摘要 — meta-se（CR-038）

- 阶段：story-planning（CP3 已 approve 方案 A + ADR-CR038-01~05）
- 交还时间：2026-08-15
- 交还方：meta-se（蓝图与架构设计师）
- 接收方：host-orchestrator（后续计算 LLD 证据队列 / 发起 CP5）

## 1. 本阶段结论

CP3 通过后，已完成正式 Story 拆解与 Feature 级设计，CP4 自动预检 PASS。Story 边界、优先级、依赖类型、输出文件、文件所有权、Feature 设计引用、LLD 策略、Wave 分组与并行策略已收敛为调度草案，交还 host-orchestrator。

## 2. Story 拆解矩阵摘要

- 13 Story / 4 Wave，全部 `status=lld-ready`，`dev_gate=cp5-approved`。
- 无新增用户决策项；实现层决策 FD-CR038-01~04 已确定性落地于 Feature DESIGN。

| Wave | Story | lld_policy |
|---|---|---|
| 1 地基 | S01（SW 显式映射，full-lld）、S02（SW 归并，technical-note）、S03（PPPoE 命令族+数据源，full-lld）、S08（devices sw 块，waived）、S09（device-reference SW 别名，waived）、S12（topology-collection 建模，waived） | full-lld×2 / tn×1 / waived×3 |
| 2 能力 | S04（PPPoE 下发，technical-note）、S05（PPPoE Client op，full-lld）、S06（exporter MVP+环回，full-lld） | full-lld×2 / tn×1 |
| 3 透传 | S07（case_runner SW，technical-note）、S13（多实例 TRex+interface_kind，full-lld） | full-lld×1 / tn×1 |
| 4 验证/集成 | S10（trex 验证，technical-note）、S11（ptm-te 编排，technical-note） | tn×2 |

**lld_policy 总量**：full-lld×5（S01/S03/S05/S06/S13）、technical-note×5（S02/S04/S07/S10/S11）、waived×3（S08/S09/S12）、batch-lld×0。

**batch-lld 判定**：无低风险同质共享实现面 Story 群，故未启用 batch-lld；涉及 security/credential 的 S03（cipher 密文）与跨模块契约的 S13 均按 full-lld 处理。

## 3. Feature 设计产物

| Feature | 适用性 | 产物 |
|---|---|---|
| F-CR038-E1 SW 映射与台账 | required | `docs/features/cr038-sw-mapping/{DESIGN,TEST-PLAN,TASKS}.md` |
| F-CR038-E2 PPPoE 配置 | required | `docs/features/cr038-pppoe-config/{DESIGN,TEST-PLAN,TASKS}.md` |
| F-CR038-E3 环回 env-file | required | `docs/features/cr038-loopback-envfile/{DESIGN,TEST-PLAN,TASKS}.md` |
| F-CR038-E4 SW 设备管理 | waived | N/A（重访条件见矩阵 §2） |
| F-CR038-E5 验证与集成 | waived | N/A（重访条件见矩阵 §2） |

## 4. 正式化产物

- `process/DEVELOPMENT-PLAN-CR-038.yaml`：status `draft → active`，file_ownership 升级为 primary/shared/merge_owner/forbidden，S13 补对 S03 硬依赖，S05 op_mapper 路径修正为 `skills/policy-route-execution/scripts/op_mapper.py`，S03 补 physical_pool.py 所有权。
- `docs/design/FEATURE-DESIGN-MATRIX-CR-038.md`：status `draft → active`，补每 Story `feature_design_refs` 列，修正 E1 触发原因（interface_kind 归 E3）。
- `process/stories/STORY-038-01.md` ~ `STORY-038-13.md`：13 张正式卡片，frontmatter 含协调者要求字段，正文含目标/依赖/设计证据引用占位/验收标准/输入输出/dev_context/validation_context。

## 5. 调度草案（LLD 证据队列输入）

host-orchestrator 按 `max_parallel_lld=3` 分轮调度 meta-dev 产出设计证据，建议批次：

- **full-lld（14 段，5 个）**：S01、S03、S05、S06、S13 → 分 2 轮（3+2）。
- **technical-note（5 个）**：S02、S04、S07、S10、S11 → Story 卡片内补齐 `## 技术说明`。
- **waived（3 个）**：S08、S09、S12 → 核验 waived 理由 + 重访条件。

全部目标 Story 的设计策略统一确认后，host-orchestrator 一次性发起 CP5 全量确认（CP4 摘要 + 全部设计证据 + CP5 自动预检）。

## 6. 文件所有权与串行化

共享文件（多 Story 写）均通过硬依赖串行，无并行冲突：

- `pool_merge.py`：S02(W1) → S13(W3)，S13 depends_on S02。
- `physical_pool.py`：S03(W1) → S13(W3)，S13 depends_on S03。
- `exporter.py`：S06(W2) → S13(W3)，S13 depends_on S06。

其余文件均为单 Story 独占（primary）。

## 7. CP4 自动预检

- 路径：`process/checks/CP4-STORY-DAG-PARALLEL-SAFETY.md`
- 结论：PASS（DAG 无环、Wave 方向正确、文件所有权串行化、lld_policy 合规）。
- 未豁免 FAIL 数量：0。

## 8. 待决策项

- 无新增用户决策项。CP2 已锁 7 项方向决策，CP3 已 approve 5 条 ADR；Feature 级实现决策 FD-CR038-01~04 已确定性落地。

## 9. 风险与未决项

| 项 | 状态 | 说明 |
|---|---|---|
| Q-038-001 ptm-atomic PPPoE op 是否已暴露（RA-038-001） | non-blocking-open | S05 full-lld 三选一闭环（默认 atomic-skip），不阻塞 LLD |
| Q-038-002 H3C 真机命令差异 | non-blocking-open | dry-run diff + 真机独立 runtime_authorization |

## 10. 约束遵守确认

- 未修改 REQUIREMENTS-CR-038.md / HLD-CR-038.md / ARCHITECTURE-DECISION-CR-038.md / 蓝图三件套。
- 未生成 LLD 设计证据正文（Story 卡片仅留占位）。
- 未自行发起 CP4/CP5 人工门禁，未自行调度 meta-dev。
- 凭据 `${ENV_VAR}` 占位、cipher 密文、默认 dry-run、GE1_1~4 禁改、只操作 TE 实例，均写入 DEVELOPMENT-PLAN `security_constraints` 与 Story `dev_context`。

## 11. 下一步（建议 host-orchestrator 动作）

1. 消费本摘要与 CP4 自动预检，计算覆盖 13 个目标 Story 的 `lld_design_batch`。
2. 按 `max_parallel_lld=3` 分轮拉起 meta-dev，产出 full-lld（5）/ technical-note（5）/ waived（3）设计证据。
3. 收敛 LLD clarification（如有）后，汇总 CP4 摘要 + 全部设计证据 + CP5 自动预检，一次性发起 CP5 全量确认。
