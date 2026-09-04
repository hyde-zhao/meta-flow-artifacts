# 交还摘要 — requirement-clarification（meta-pm）→ host-orchestrator

- CR：CR-039（ptm-te 工作区已验证代码回源 canonical：G1 静态路由回滚兜底 + G4 next_hop_a 占位符 + G5 sw3 族）
- 阶段：requirement-clarification（增量澄清）
- 委托：host-orchestrator 委托 meta-pm 直连用户
- 交还时间：2026-08-18
- 下一动作：host-orchestrator 汇总并发起 CP2 正式人工确认（meta-pm 不自行发起）
- 审核模式：本 CR 为同步/修复类，走 **static-only** 验证（单测对拍 + dry-run），工作区已 26/28 step PASS 实测

---

## 1. 用户真实意图

1. 将 **ptm-te-manaul 工作区已验证通过**的 3 块能力改动（G1 静态路由回滚 verify 兜底、G4 `${ENV.dut.next_hop_a}` 第 10 类占位符、G5 sw3 族六处映射 + telnet 旁路）**回源 canonical**，避免重装 ptm-atomic / 重新部署丢失修复。
2. 同步修复 **SKILL.md 计数漂移**（Gotcha#8 `EXPECTED_OP_COUNT=40`→45）与 **op-coverage-matrix 旁路说明**，满足方案评审规则第 1 条（文档-代码一致性）。
3. 补记 **CR-043 / CR-044 / CR-046 追溯链**（canonical 零正式记录，代码有标注但无变更单）。
4. 守住安全边界：真机验证（trex 发流 / SSH 重启 trex-stl-te）沿用 CR-038 模式，独立 runtime_authorization，本 CR 默认不授权。

## 2. 核对结论（canonical vs 工作区）

| CR-039.md 声明 | 工作区实际实现 | 核对结论 |
|---|---|---|
| G1 新增 `_query_static_route_id` + `handle_rollback` 静态路由特殊分支 | op_mapper.py L2616-2669 `_query_static_route_id`（按 dst_network，可选 next_hop_addr 匹配，经 `fw_verify_static_route` 查询）；`handle_rollback` L2728-2750 对 `fw_config_static_route` **无条件**走 verify 查询 | 一致，但**实现为无条件优先 verify**，非"缺失时兜底"（DQ-039-01 关键事实） |
| G4 第 10 类占位符 `${ENV.dut.next_hop_a}` | resolve_env_refs L1562-1566 新增 `dut.next_hop_a` 分支；L1484 注释 9→10 类；L1596 未知路径报错 9→10 类 | 一致 |
| G5 sw3 族六处映射表 + `_execute_sw3_op` + `_exec_sw3_telnet` | 六处映射确认：OP_ID_TO_SUBCOMMAND(L130-132)、ARGS_TO_FLAGS(L500-503)、REQUIRED_ARGS(L576-577)、ROLLBACK_STRATEGY(L829-833)、OP_ID_METADATA(L1070-1075)、EXPECTED_OP_COUNT(L1078-1079)=45；另有 execute_op sw3 旁路(L1956-1964)、`_execute_sw3_op`(L2122)、`_exec_sw3_telnet`(L2279)、`_parse_sw3_pppoe_route`(L2324)、validate sw3 子命令校验(L3135-3142) | 一致（六处映射 + 执行 + 校验均确认） |
| case_runner sw3 dry-run envelope | `_build_sw3_dry_run_envelope`(L925-952) + dry-run 分支(L2321-2334) + import map_op_id_to_subcommand(L54) | 一致，但**还含 CR-047 post_delay**（L113-134/L2438-2440），CR-039 未覆盖 → DQ-039-05 |
| SKILL.md Gotcha#8 40→45 | L315=45，修订记录 v1.7（CR-046）已存在 | 一致 |
| op-coverage-matrix 新增 sw3 旁路章节 | 工作区**无该文件**；canonical total=124/mapped=21/gap=6/unmapped=97 | 该改动是 canonical 新增内容（非回源），需在 canonical 文档侧新增 |
| 追溯 CR-043/044/046 | canonical 代码有 CR-043（L79-82/L529-530）、CR-044（L114-117）标注，无正式变更单；CR-046 完全零记录 | 与声明一致 |

## 3. G1 / G4 / G5 精确改动集清单（回源依据）

### G1 静态路由回滚兜底（`skills/policy-route-execution/scripts/op_mapper.py`）

| 改动点 | 工作区行号 | 内容 |
|---|---|---|
| 新增 `_query_static_route_id()` | L2616-2669 | 按 `dst_network`（可选 `next_hop_addr`，`${ENV.*}` 占位符跳过匹配）经 `fw_verify_static_route` 查询路由 id；多路由命中取首个；无匹配返回 None |
| `handle_rollback()` 静态路由分支 | L2728-2750 | `rtype=="inverse_op"` 且 `op_id=="fw_config_static_route"` 时无条件调用 `_query_static_route_id`，再 `fw_delete_static_route --id`；未匹配返回 EXEC_FAILED envelope |

### G4 next_hop_a 占位符（`skills/policy-route-execution/scripts/op_mapper.py`）

| 改动点 | 工作区行号 | 内容 |
|---|---|---|
| `resolve_env_refs` 新增分支 | L1562-1566 | `path=="dut.next_hop_a"` → `nodes.dut1.next_hop_a`，缺失抛 `_missing` |
| 占位符表注释 | L1484 | 第 10 类 `${ENV.dut.next_hop_a}` |
| 未知路径报错文案 | L1596 | "不在 9 类" → "不在 10 类" |

### G5 sw3 族（`op_mapper.py` + `case_runner.py` + `SKILL.md` + `op-coverage-matrix.md`）

| 改动点 | 工作区行号 | 内容 |
|---|---|---|
| OP_ID_TO_SUBCOMMAND | L130-132 | `sw3_sync_pppoe_route` → `("sw3", "sync-pppoe-route")` |
| ARGS_TO_FLAGS | L500-503 | 形式 flag `vpn_instance` → `--vpn-instance`（实际不走 ptm-atomic） |
| REQUIRED_ARGS | L576-577 | `sw3_sync_pppoe_route: []`（vpn_instance 有默认值 hyde） |
| ROLLBACK_STRATEGY | L829-833 | `type=irreversible`（修复性同步，回滚无意义） |
| OP_ID_METADATA | L1070-1075 | `side_effect=state_mutation` / `rollback=irreversible` / `idempotent=True` |
| EXPECTED_OP_COUNT | L1078-1079 | 42 + 2 + 1 = **45** |
| execute_op sw3 旁路 | L1956-1964 | `family=="sw3"` 走 `_execute_sw3_op`，不经 build_command/subprocess |
| `_execute_sw3_op` | L2122-2276 | 定位 SW 节点（优先 role=pppoe_server）→ dry-run 返回命令计划 → 实连 telnet 同步默认路由下一跳 |
| `_exec_sw3_telnet` | L2279-2322 | telnetlib 登录（H3C Login/Password 大小写不敏感匹配）、screen-length 0 关分页、逐条命令执行 |
| `_parse_sw3_pppoe_route` | L2324-2356 | 从 `display ip routing-table vpn-instance` 解析 (client_ip, default_next_hop) |
| validate sw3 子命令校验 | L3135-3142 | 校验 sw3 族子命令集 = {sync-pppoe-route} |
| case_runner `_build_sw3_dry_run_envelope` | L925-952 | dry-run 时 sw3 命令计划包装为 dry_run envelope（非 ptm-atomic 命令） |
| case_runner sw3 dry-run 分支 | L2321-2334 | dry-run 时 family=="sw3" 调 execute_op(dry_run=True) 获取计划 |
| case_runner import | L54 | 新增 `map_op_id_to_subcommand` |
| SKILL.md Gotcha#8 | L315 | 40→45，补 CR-043/044/046 来源 |
| SKILL.md 修订记录 | L332 | v1.7（CR-046 sw3 族 + 计数 44→45） |
| op-coverage-matrix | — | canonical 侧新增 sw3 旁路章节（total=124/mapped=21 不变） |

## 4. Scenario Gray Areas 处理结果

| 灰区 ID | 主题 | 用户选择 | 处理状态 | 讨论日志 / checkpoint |
|---|---|---|---|---|
| SGA-039-01 | G1 回滚 verify 兜底 vs step-refs 优先级 | 待 CP2（SGQ-039-01） | open | `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR039.md` |
| SGA-039-02 | EXPECTED_OP_COUNT=45 校验口径 | 待 CP2（SGQ-039-02） | open | 同上 |
| SGA-039-03 | CR-043/044/046 追溯补记范围 | 待 CP2（SGQ-039-03） | open | 同上 |
| SGA-039-04 | CR-047 post_delay 范围缺口（新增） | 待 CP2（SGQ-039-04） | open | 同上 |

- 讨论日志：`process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR039.md`
- SGQ 证据：SGQ-039-01~04 已记录（问题、候选选项、推荐项、影响面）；用户回答待 CP2 回填。本 CR 为同步回源，无新用户场景，故灰区集中在回源语义与校验口径而非场景行为。
- 说明：meta-pm 当前工具面无 `AskUserQuestion`，SGQ 记录为"待 host-orchestrator 在 CP2 统一发起"，未直接向用户提问。

## 5. 需求摘要

- **N/A 判定**：CR-039 为同步/修复类 CR，工作区已实现并实测，**不新增需求语义**。需求基线以 CR-039.md §需求理解框架六条（G1 语义 / G4 契约 / G5 映射 / SKILL.md 回写 / op-coverage-matrix / 追溯补记）为准。
- 若 host-orchestrator 需要正式需求基线条目，建议在 CP2 后由 meta-se/实现阶段以"回源核对单"形式固化，无需新建 REQUIREMENTS-CR-039.md。
- BLOCKING 未决项：**5 个决策项**（DQ-039-01~05），均为 scope/runtime_authorization 类，需用户确认。
- `ready_for_design`：**false**（待 CP2 用户确认后置 true）。

## 6. SCENARIOS 覆盖摘要

- **N/A**：CR-039 验证模式为 `static-only`（单测对拍 + dry-run），无场景展开需求。工作区 26/28 step PASS 为实测证据，回源验证用：
  - G1：`test_op_mapper_rollback.py` 新增 verify 兜底单测（`_query_static_route_id` 匹配/未匹配/多路由/占位符跳过）
  - G4：`test_tg_op_mapping.py` 或新建新增 `${ENV.dut.next_hop_a}` 解析单测
  - G5：`test_tg_op_mapping.py` 断言改 45 + sw3 六处映射断言 + dry-run envelope 单测
- 不产出 `SCENARIOS-CR-039.yaml`。

## 7. TEST-MATRIX 缺口摘要

- **N/A**：无用例场景矩阵增量；验证缺口即上述单测断言更新（canonical `test_tg_op_mapping.py` L167/L396 断言 44 需改 45，`test_op_mapper_rollback.py` 需补 G1 verify 兜底用例）。
- 风险：canonical **无 sw3 相关单测**，回源时须补齐，否则 CP6/CP7 无法机器验证。

## 8. STORY-MAP 摘要

- **N/A**：CR-039 为回源同步，不拆 Story（直接以实现+单测形式回源）。不产出 `STORY-MAP-CR-039.md`。

## 9. MVP 范围

- **N/A**：范围已由 CR-039.md frontmatter `impact_*_refs` 定义，无需 MVP-SCOPE 文档。
- 隐含 In：G1 回滚兜底、G4 第 10 类占位符、G5 sw3 族、EXPECTED_OP_COUNT=45、SKILL.md 回写、op-coverage-matrix 旁路章节、CR-043/044/046 追溯补记、canonical 新增 env-file（case-execution 格式，含 next_hop_a）。
- 隐含 Out/Deferred：CR-047 post_delay（待 DQ-039-05 决策）、真机验证（DQ-039-04 不授权）、sw3 族多 op 扩展（DEF-039-02）。

## 10. 发布切片与 backlog

- **RELEASE-SLICES.md**：**N/A**（无发布切片，回源随下一个交付批次合入）。
- **BACKLOG.md**：**建议增量**（候选后续项）：
  - CR-047 post_delay 回源（若 DQ-039-05 选择独立 CR）
  - 真机验证回归（trex 发流 / SSH 重启 trex-stl-te）—— 独立 runtime_authorization
  - sw3 族多 op 扩展 / `${ENV.dut.next_hop_a}` 多下一跳数组（DEF-039-02/03）
  - op-coverage-matrix 旁路章节落地核对
  - 若 host-orchestrator 决定创建，建议命名 `BACKLOG-CR-039.md`（或并入既有 BACKLOG）。

## 11. CP1 / CP2 证据

- **CP1**（场景完备门）：本 CR 无新场景，`USE-CASES.md` 判定 N/A；CP1 建议结论 **WAIVED**（理由：同步回源 CR，场景基线不变），由 host-orchestrator 在 `process/checks/CP1-CR039.md` 落自动检查。
- **CP2**（需求/场景/范围基线门）：待用户确认 5 个决策项后由 host-orchestrator 写入 `process/checks/CP2-CR039.md`；当前建议结论 **BLOCKED**（5 个 DQ 未确认）。
- 讨论恢复点：`process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR039.md`。

## 12. 待人工决策项（供 CP2 Decision Brief）

| 决策 ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 优劣 | 影响/风险 | 回退/切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-039-01 | scope | G1 回滚 verify 兜底与 step-refs 优先级 | A：无条件优先 verify 查询（与工作区一致） | B：仅 step-refs/声明缺失时兜底；C：改 `id_source=query` 声明纳入 4 模式统一路径 | A：与已实测行为一致、最可靠、实现最简单；B：多一层分支但 static_route config 不返回 id 时实际必走 verify，与 A 等价且更复杂；C：最规范但需动 decl 声明 + build_inverse_args，改动面大 | A 影响：每次 static_route 回滚多一次只读 verify（性能可忽略）；不回源或改 B 会导致与工作区行为漂移 | 若发现 verify 查询在特定拓扑（大路由表）超时，可降级 B（先 step-refs 后 verify） |
| DQ-039-02 | scope | G5 `EXPECTED_OP_COUNT=45` 校验口径 | A：代码常量 + validate 三表 + `test_tg_op_mapping.py` 断言 + SKILL.md Gotcha#8 + op-coverage-matrix 旁路章节五处同步 | B：仅代码 + 单测 + SKILL.md（op-coverage-matrix 不改） | A：满足方案评审规则第 1 条内部一致性；op-coverage-matrix 需说明 sw3 走 telnet 旁路避免读者疑惑；B：改动最小但文档缺 sw3 说明 | A 影响：5 处改动同步成本低；缺任一处即一致性缺陷，CP6 校验可能 FAIL | 若 op-coverage-matrix 结构不允许旁路章节，则降级 B + 在 README/RELEASE 说明 sw3 边界 |
| DQ-039-03 | scope | CR-043/044/046 追溯补记范围 | B：补正式变更单 + SKILL.md 修订记录补 v1.7/v1.8 来源行 | A：仅变更单留痕；C：变更单 + SKILL.md + DEV-LOG + BACKLOG 追溯表 | B：SKILL.md 缺 CR-043/044/046 来源行本身就是一致性缺陷，补齐最小且有效；C 最完整但工作量大 | B 影响：三份最小 CR 档案 + SKILL.md 修订记录；若只做 A，SKILL.md 仍缺来源行（一致性缺陷残留） | 若审计要求 DEV-LOG 明细，升级 C |
| DQ-039-04 | runtime_authorization | 真机验证（trex 发流 / SSH 重启 trex-stl-te）授权 | 仅审计记录，不授权（沿用 CR-038 模式，本 CR static-only） | 在 CP2 一并授权真机回归 | 推荐：真机操作影响测试环境，独立授权边界清晰；备选：一次授权省后续门禁但风险集中 | 不授权影响：回源后仅 dry-run/单测验证，真机回归待用户发起；误授权风险：发流/重启服务影响测试环境 | 用户发起真机回归时，单独发起 runtime_authorization |
| DQ-039-05 | scope | CR-047 post_delay 是否并入本 CR 回源（新增） | B：独立 CR-047 另行回源；或 C：剔除并记 BACKLOG | A：并入 CR-039 一并回源 | 推荐：CR-047 是独立语义（PPPoE 断开后等待），独立 CR 审计边界清晰；A 会扩大 CR-039 影响面 | 若 B/C：case_runner 回源后与工作区仍差 post_delay（记录到 BACKLOG 防遗漏）；若 A：本 CR 需补 impact 说明 | 若用户确认 CR-047 低风险且与 G5 同批部署，选 A |

## 13. N/A / WAIVED 项

| 项 | 原因 | 影响范围 | 后续触发条件 |
|---|---|---|---|
| `USE-CASES-CR-039.md` | 同步回源 CR，无新用户场景；G1/G4/G5 均为既有 ptm-te 测试执行能力的修复/补全 | 场景基线不变 | 未来新场景 CR 需增量时 |
| `REQUIREMENTS-CR-039.md` | 无新需求语义；需求基线以 CR-039.md §需求理解框架六条为准 | 范围确认依赖 CR-039.md + 本交还摘要 | 若需正式需求条目，实现阶段以回源核对单固化 |
| `SCENARIOS-CR-039.yaml` | 验证模式 static-only（单测对拍 + dry-run），无场景展开 | 回源验证以单测断言为准 | 真机回归时另行授权验证 |
| `TEST-MATRIX-CR-039.md` | 无用例场景矩阵增量；验证缺口=单测断言更新 | canonical 单测需补 G1/G4/G5 断言 | 单测在 CP6/CP7 落地 |
| `STORY-MAP-CR-039.md` | 回源同步不拆 Story | 无 Story 规划增量 | 后续功能 CR 拆 Story 时 |
| `MVP-SCOPE-CR-039.md` | 范围已由 CR-039.md impact_*_refs 定义 | 无独立 MVP 文档 | — |
| `RELEASE-SLICES-CR-039.md` | 无发布切片 | 随下一交付批次合入 | 发布批次规划时 |
| `CP1-CR039.md` | 无新场景 → 结论 WAIVED（建议） | 场景完备门 | 由 host-orchestrator 落自动检查 |
| 产物 N/A 合计 | 8/9 项 N/A + BACKLOG 建议增量 | 见上 | — |

## 14. CP2 Decision Brief 输入（建议 host-orchestrator 使用）

- **推荐决策**：approve（接受 5 项推荐方案：DQ-039-01=A、DQ-039-02=A、DQ-039-03=B、DQ-039-04=不授权、DQ-039-05=B 或 C）。
- **不授权项**：approve 不隐含真机下发授权（trex 发流 / SSH 重启 trex-stl-te 为独立 runtime_authorization）；不授权凭据明文落盘（sw3 密码走 `SW3_PASSWORD` 环境变量）。
- **approve 后发生**：host-orchestrator 回填 CP2 人工结果，消费 CR-039 route_plan 推进至实现（meta-dev 回源代码 + 单测 + SKILL.md + op-coverage-matrix + 追溯档案 + env-file）。
- **不确认阻塞**：CP2 未通过前不得进入 solution-design / Story 拆解 / LLD / 实现。
- **成功指标**：
  - SM-039-01：回源后 `validate_mapping_consistency()` PASS，`EXPECTED_OP_COUNT=45` 五处一致（代码/单测/SKILL.md/op-coverage-matrix）。
  - SM-039-02：G1 静态路由回滚 verify 兜底单测覆盖匹配/未匹配/多路由/占位符跳过四分支。
  - SM-039-03：G4 `${ENV.dut.next_hop_a}` 解析单测 PASS（含缺失报错）。
  - SM-039-04：sw3 dry-run envelope 单测 PASS（命令计划非 ptm-atomic 命令）。
  - SM-039-05：CR-043/044/046 正式变更单 + SKILL.md 修订记录补齐。
- **风险**：
  - RA-039-01：G1 verify 兜底在真实大路由表场景超时或匹配歧义（dst_network 非唯一）——缓解：next_hop_addr 参与匹配 + 单测覆盖多路由。
  - RA-039-02：sw3 telnet 依赖 `SW3_PASSWORD` 环境变量，真机验证前无法完整验证——缓解：static-only + dry-run 计划断言。
  - RA-039-03：CR-047 post_delay 若被静默遗漏，case_runner 回源不完整——缓解：DQ-039-05 显式决策 + BACKLOG 记录。
  - RA-039-04：canonical 无 sw3 单测，回源后 CP6/CP7 无机器验证——缓解：回源时补齐单测。
  - RA-039-05：op-coverage-matrix 结构限制旁路章节——缓解：降级方案 B（仅代码/单测/SKILL.md 同步 + README 说明 sw3 边界）。

## 15. 阶段产物路径清单

- `process/CLARIFICATION-LOG.md`（追加 CR-039 调研发现段落，round 15→16）
- `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR039.md`（灰区讨论日志 + SGQ-039-01~04）
- `process/handoffs/requirement-clarification-meta-pm-RETURN-SUMMARY-CR039.md`（本交还摘要）
- 建议 host-orchestrator 后续创建：`process/checks/CP1-CR039.md`、`process/checks/CP2-CR039.md`、`process/checkpoints/CP2-REQUIREMENTS-BASELINE-CR039.md`、可选 `BACKLOG-CR-039.md`
