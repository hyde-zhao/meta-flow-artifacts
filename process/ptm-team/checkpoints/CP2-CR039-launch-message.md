请审查人工门禁 `CP2-CR039`（修订 v2，按用户评审意见整改）。

checklist 路径: `process/checkpoints/CP2-CR039.md`
自动预检结论: CP1 自动检查 **PASS**（同步回源 CR 场景基线豁免）；Decision Brief 已按评审整改重新生成并通过 `meta-flow check human-gate` 校验。

审批者摘要:
- 本次确认服务的整体目标: 锁定 CR-039 需求/场景/范围基线（G1/G4/G5 回源 + SKILL.md 回写 + op-coverage-matrix sw3 旁路章节 + CR-043/044/046 追溯补记），批准后进入实现（meta-dev 以 canonical HEAD 为基底逐块回源）。
- 推荐动作: 默认推荐 approve，接受下列 3 项推荐方案 + 2 项 agent 默认处理语义声明。
- approve 后会发生什么: 接受本轮 Decision Brief 推荐方案，回填 CP2 结果并推进至实现阶段（meta-dev 逐块回源 + 单测 + SKILL.md 回写 + 追溯档案），随后 CP6/CP7 验证（static-only）。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；真机下发（trex 发流 / SSH 重启）为独立 runtime_authorization，本 CR 不授权。
- 不确认会阻塞什么: 实现、单测、CP6/CP7 验证推进保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: `process/context/CP2-CR039-CONTEXT.yaml`（read_profile=compact，默认读 CR-039.md + checkpoint + meta-pm 交还摘要）。
决策收集覆盖: 见 checkpoint `### Decision Collection Coverage`（6 来源已扫描，5 候选问题 → 3 纳入待决策，2 降级 agent 默认处理，0 N/A 缺失）。
决策分层:
- 必须用户决策: 3（DQ-039-03 / DQ-039-04 / DQ-039-05）
- 高风险策略确认: 0
- agent 默认处理: DQ-039-01（含语义声明）/ DQ-039-02（含口径澄清）+ SM-039-01~05 + RA-039-01~05 + 实施补充（G6 台账、sw3 用例来源、逐块合并、post_delay 精细排除）
- 仅审计记录: 0
本轮待人工决策项: 3
blocking / high-risk 决策摘要: DQ-039-03（追溯补记范围）、DQ-039-04（真机授权边界确认）、DQ-039-05（CR-047 范围缺口）

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-039-03 | scope | CR-043/044/046 追溯补记范围？ | **B：补正式变更单 + SKILL.md 修订记录补 v1.7/v1.8 来源行** | A：仅变更单留痕；C：变更单 + SKILL.md + DEV-LOG + BACKLOG 追溯表 | B 最小且修复 SKILL.md 缺来源行的一致性缺陷；C 最完整但工作量大 | B：三份最小 CR 档案 + SKILL.md 修订记录；若只做 A，SKILL.md 仍缺来源行（缺陷残留） |
| DQ-039-04 | runtime_authorization | 本 CR 是否授权真机验证（trex 发流 / SSH 重启 trex-stl-te）？ | **仅审计记录，不授权**（沿用 CR-038 模式，本 CR static-only；后续真机回归单独发起 runtime_authorization） | 在 CP2 一并授权真机回归 | 推荐：真机操作影响测试环境，独立授权边界清晰，符合验证锁；备选：一次授权省后续门禁但风险集中 | 不授权影响：回源后仅 dry-run/单测验证，真机回归待用户发起；误授权风险：发流/重启服务影响测试环境 |
| DQ-039-05 | scope | CR-047 post_delay（工作区 case_runner 含、canonical 零记录）是否并入本 CR 回源？ | **B：独立 CR-047 另行回源；或 C：剔除并记 BACKLOG**（须显式决策，不得静默遗漏） | A：并入 CR-039 一并回源 | 推荐：CR-047 是独立语义（PPPoE 断开后等待），独立 CR 审计边界清晰；A 会扩大 CR-039 影响面 | 若 B/C：case_runner 回源时精细排除 post_delay 行（L114-115/L2438-2440）+ 记 BACKLOG；若 A：本 CR 需补 impact 说明 |

**agent 默认处理语义声明（approve 即接受，不占用决策位）**:

1. **DQ-039-01（G1 回滚优先级）**：方案 A = 无条件优先 verify 查询，**放弃 canonical 现有"step-refs 优先、verify 兜底"语义**（op_mapper.py L706-707）；verify 匹配失败即返回 `EXEC_FAILED`，无 fallback 到 `${STEP-N.static_route_id}`。已与工作区实测一致（26/28 step PASS）。
2. **DQ-039-02（EXPECTED_OP_COUNT 口径）**：计数同步载体为代码常量 + validate 三表 + `test_tg_op_mapping.py` 断言 + SKILL.md Gotcha#8（四处改 45）；**op-coverage-matrix 保持 `total=124/mapped=21` 不变，仅新增 sw3 族旁路章节**（sw3 走 telnet 旁路，不计入 ptm-atomic atoms 映射），不改计数。

如果你回复 approve，表示你接受以上 3 项推荐方案 + 2 项 agent 默认处理语义声明，不表示授权以下不授权项。
不授权项:
1. **真机下发**：trex 发流 / SSH 重启 trex-stl-te —— 独立 `runtime_authorization`（本 CR 不授权，DQ-039-04 确认边界）。
2. **凭据明文落盘**：sw3 密码走 `SW3_PASSWORD` 环境变量（ADR-02）。
3. **CR-047 post_delay**：若选 B/C，不在本 CR 实现（回源时精细排除相关行并记 BACKLOG）。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
