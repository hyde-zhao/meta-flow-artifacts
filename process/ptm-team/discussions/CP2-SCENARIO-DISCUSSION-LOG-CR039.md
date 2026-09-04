# CP2 Scenario Gray Areas 讨论日志 — CR-039

- CR：CR-039（ptm-te 工作区已验证代码回源 canonical）
- 阶段：requirement-clarification（meta-pm 委托）
- 讨论范围：G1 静态路由回滚兜底语义 / G4 next_hop_a 占位符契约 / G5 sw3 族回源口径 / 追溯补记范围 / 范围缺口
- 生成时间：2026-08-18
- 状态：open（推荐方案已记录，用户确认待 host-orchestrator 发起 CP2）

---

## 1. 灰区识别（SGA）

CR-039 为同步/修复类 CR，无新用户场景，灰区集中在回源语义与校验口径，共识别 4 个关键灰区 + 1 个范围缺口。

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 用户选择 | 状态 |
|---|---|---|---|---|---|
| SGA-039-01 | G1 回滚 verify 兜底与 step-refs 优先级 | 工作区实现是**无条件 verify 查询**，非"缺失时兜底"；回源后行为取决于此语义 | 回滚可靠性 / 单测断言 / 代码复杂度 | 待 CP2（SGQ-039-01） | open |
| SGA-039-02 | G5 `EXPECTED_OP_COUNT=45` 校验口径 | 三表 + 单测断言 + SKILL.md + op-coverage-matrix 四处需一致，缺一即内部一致性缺陷（方案评审规则第 1 条） | 校验 / 文档一致性 / 测试 | 待 CP2（SGQ-039-02） | open |
| SGA-039-03 | CR-043/044/046 追溯补记范围 | canonical 对三 CR 零正式记录（代码有标注但无变更单，CR-046 完全零记录），影响审计链条 | 追溯 / 变更管理 / SKILL.md 修订记录 | 待 CP2（SGQ-039-03） | open |
| SGA-039-04 | CR-047 post_delay 范围缺口（新增） | 工作区 case_runner.py 含 CR-047 post_delay，canonical 无 CR-047 变更单，CR-039 未覆盖；若仅回源 sw3，case_runner 回源后仍与工作区不一致 | 范围 / 回源完整性 | 待 CP2（SGQ-039-04） | open |

## 2. SGQ 用户可见场景确认交互

标准模式要求至少 1 条 `SGQ-*` 用户可见场景确认交互；本 CR 提供 4 条（均待 host-orchestrator 在 CP2 统一向用户发起确认，meta-pm 不自行发起）。

### SGQ-039-01：G1 回滚兜底与 step-refs 优先级

- **问题**：`fw_config_static_route` 回滚时，`_query_static_route_id`（verify 按 dst_network 匹配）与 step-refs / 声明优先路径（`build_inverse_args`）的优先级如何定？
- **候选选项**：
  - A（工作区现状）：无条件优先 verify 查询（当前实现）
  - B：仅当 step-refs / result_envelope / args 无 id 时兜底 verify
  - C：声明式 —— 将 static_route 改为 `id_source=query`（query_op=fw_verify_static_route，match=dst_network），纳入 `build_inverse_args` 4 模式统一路径
- **推荐项**：A（无条件 verify，与工作区已实测一致）。理由：config POST 不返回 id（id_source=None），step-refs 无法从 config 步骤获得 id；verify 按 dst_network 匹配最可靠且已实测通过；性能开销仅一次只读查询。
- **用户回答**：待 CP2。
- **复述确认**：待用户确认后回填。
- **影响面**：`handle_rollback` 分支结构、`test_op_mapper_rollback.py` 新增单测断言、回滚可靠性。

### SGQ-039-02：EXPECTED_OP_COUNT=45 校验口径

- **问题**：G5 回源后 `EXPECTED_OP_COUNT=45` 需要在哪些载体上同步并保持一致性？
- **候选选项**：
  - A：代码常量 + validate 三表 + `test_tg_op_mapping.py` 断言 + SKILL.md Gotcha#8 + op-coverage-matrix 旁路章节（五处同步）
  - B：仅代码 + 单测 + SKILL.md（op-coverage-matrix 不改）
- **推荐项**：A。理由：op-coverage-matrix 需说明 sw3 族走 telnet 旁路（非 ptm-atomic），避免读者疑问 sw3 为何缺失；total=124/mapped=21 不变。SKILL.md Gotcha#8 需 40→45（补 CR-043/044/046）。
- **用户回答**：待 CP2。
- **影响面**：校验一致性、文档-代码一致性（方案评审规则第 1 条）、测试基线。

### SGQ-039-03：CR-043/044/046 追溯补记范围

- **问题**：canonical 对 CR-043/044/046 零正式记录，补记到什么粒度？
- **候选选项**：
  - A：仅变更单留痕（三份最小 CR 档案，标注来源=代码标注/工作区实测）
  - B：变更单 + SKILL.md 修订记录补 v1.7/v1.8 来源行
  - C：变更单 + SKILL.md + DEV-LOG 条目 + BACKLOG 追溯表（最完整）
- **推荐项**：B（变更单 + SKILL.md 修订记录）。理由：SKILL.md 是 case-execution 的公开修订记录，缺 CR-043/044/046 来源行本身就是一致性缺陷；DEV-LOG 条目属附加可选项。
- **用户回答**：待 CP2。
- **影响面**：变更管理档案、SKILL.md 修订记录、审计链条。

### SGQ-039-04：CR-047 post_delay 范围缺口（新增）

- **问题**：工作区 case_runner.py 含 CR-047 post_delay 改动（`_parse_post_delay` + `time.sleep(OP_INTERVAL_SEC + post_delay)`），canonical 无 CR-047 变更单，CR-039 未覆盖。如何处置？
- **候选选项**：
  - A：并入 CR-039 一并回源（扩大 impact_module_paths，补 post_delay 说明）
  - B：独立 CR-047 另行回源（本 CR 只回源 sw3 部分）
  - C：明确从 CR-039 范围剔除，仅记录到 BACKLOG（post_delay 后续处理）
- **推荐项**：B 或 C 均可（取决于 CR-047 的优先级），但必须**显式决策**，不得静默遗漏。若用户确认 CR-047 是低风险独立修复，建议 B（独立 CR 保持审计边界清晰）。
- **用户回答**：待 CP2。
- **影响面**：case_runner.py 回源完整性、CR 审计边界、PPPoE 断开后等待语义。

## 3. Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-039-01 | 真机验证（trex 发流 / SSH 重启 trex-stl-te） | 工作区实测已覆盖，回源用 static-only | 真机操作需独立 runtime_authorization（沿用 CR-038 模式） | 用户发起真机回归时授权 |
| DEF-039-02 | sw3 族扩展（多 op / 更多 SW 厂商命令） | G5 当前仅单 op `sw3_sync_pppoe_route` | 当前用例只需此 op | 未来 SW 操作需求 |
| DEF-039-03 | `${ENV.dut.next_hop_a}` 之外的多下一跳占位符（next_hop_b/数组） | G4 仅补单值占位符 | 当前用例只用 GW-A 单下一跳 | 负载分担多下一跳用例需要时 |

## 4. 讨论 checkpoint

- 恢复点：`process/checks/CP2-DISCUSSION-CHECKPOINT.json`（CR-038 遗留文件；CR-039 建议新增 `CP2-DISCUSSION-CHECKPOINT-CR039.json`，由 host-orchestrator 决定）
- 决策项载体：`process/handoffs/requirement-clarification-meta-pm-RETURN-SUMMARY-CR039.md`
