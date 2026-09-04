---
doc_type: return-summary
id: HANDOFF-CR047-RC-META-PM-RETURN-001
cr_id: CR-047
stage: requirement-clarification
from_role: meta-pm
to_role: host-orchestrator
delegated_interaction: false
created_at: "2026-08-19"
input_handoff: HANDOFF-CR047-RC-META-PM-001
---

# CR-047 增量需求澄清 Return Summary（meta-pm，阶段零 + CP1/CP2 输入）

## 一、阶段零调研结论

### 1.1 三方一致性核对（CR-047.md ↔ BL-039-01 ↔ 台账 CR-039-FU-047）：一致

| 核对项 | 结论 | 证据 |
|---|---|---|
| 回源锚点双标注 | 一致 | CR-047 表格块 2 双标注「实测锚点 L2521（BACKLOG 卡片原写 L2437-2440）」，并显式说明「CR-039 收敛时实测工作区睡眠行为 L2511 附近，最终确认 L2521」，与台账 FU-047「工作区实测锚点 L2511，卡片原写 L2437-2440 双标注」、BL-039-01「实测锚点 L2511」可互解释；canonical 行号以符号定位为准（CR-036 同款约束），已在 CR-047 声明 |
| 排除边界 | 一致 | CR-047「CR-039 STORY-039-03 回源时已精细排除（grep 零残留 + 源码级守护单测）」与台账 FU-047、BL-039-01 描述逐字对应；已实测核实：canonical `case_runner.py` grep `_parse_post_delay`/`post_delay` 零命中；守护单测 `tests/test_tg_op_mapping.py:622-626`（`test_case_runner_no_post_delay_backport`）当前断言零残留存在 |
| static-only 口径 | 一致 | CR-047 验证方式 = static-only（单测 + 静态断言），runtime_authorization 声明「无」；与 DQ-039-04 真机三不授权、台账不授权项第 5 条（follow_up_tracking，用户决定推进时转正式 CR）口径吻合 |
| EXPECTED_OP_COUNT=45 不变 | 一致 | 已实测核实 `skills/policy-route-execution/scripts/op_mapper.py:1080` `EXPECTED_OP_COUNT = 45`；post_delay 是 case_runner step 级参数、不新增 op，不触碰 op 映射与 op-coverage-matrix 计数（total=124/mapped=21、EXPECTED_OP_COUNT=45 双口径均不变） |

### 1.2 canonical 现状核实（符号定位全部命中）

- `OP_INTERVAL_SEC = 2`（`case_runner.py` L110）。
- execute_steps 主循环睡眠行 L2417，位于 `if idx < n - 1` 分支（「op 间间隔（HLD §13.1，仅 --execute 模式且非最后一步）」注释下方），与 CR-047 声明「当前 L2417，idx < n - 1 分支」一致。
- rollback 清理循环睡眠行 L2513，位于 `if i > 0` 分支（run_cleanup），与 CR-047「不动（工作区同样保持纯 OP_INTERVAL_SEC）」一致。
- dry-run 分支在主循环内 `continue` 先于睡眠行：post_delay 回源后仅 `--execute` 模式生效，dry-run 行为不受影响（既有语义，非新增决策点）。

### 1.3 调研总结论

CR-047.md 声明与来源链（BL-039-01 / 台账 FU-047 / DQ-039-05=B）完全一致，锚点、排除边界、static-only 口径、op 计数不变四项均经源码实测核实，无事实冲突，可进入 CP1/CP2。

## 二、文档增量判定（逐文档）

> 关键事实：当前 `docs/product/` 五文档基线全部属于 **ptm-tse 逆向分析域**（CR-030/031，REQ-RA-* / UC-RA-* / SCN-RA-* / GOAL-RA-*），与 CR-047 所属 **ptm-te 测试执行域**（case-execution skill）不同需求域。同类回源 CR（CR-039、CR-046）均未更新该五文档，其真实文档面是 `skills/case-execution/SKILL.md`（Gotcha + 修订记录）与 op-coverage-matrix。回源型 CR 优先最小增量。

| 文档 | 判定 | 理由 |
|---|---|---|
| docs/product/REQUIREMENTS.md | **N/A** | REQ-RA-* 全部为 ptm-tse 逆向分析需求；post_delay 是 case_runner step 级实现参数（用途：PPPoE 断开后等待会话 tear down），不构成新用户需求条目；CR-039/CR-046 先例均未触碰 |
| docs/product/USE-CASES.md | **N/A** | UC-RA-* 域外；本 CR 不新增用户旅程或 persona 场景（回源已实测改动） |
| docs/product/SCENARIOS.yaml | **N/A** | SCN-RA-* 域外；CR-047 验证走 CR 内 static-only 单测 + CP7，不新增工程验证场景 |
| docs/product/TEST-MATRIX.md | **N/A** | SCN-RA-* 覆盖矩阵域外；不新增 Story/场景行，覆盖统计不变 |
| docs/product/MVP-SCOPE.md | **N/A** | GOAL-RA-* / IN-RA-* / OUT-RA-* 域外；不改变任何 MVP 范围 |
| skills/case-execution/SKILL.md | **待决策（DQ-047-01）** | 量化验收第 4 条本身留了「补一行或写 N/A 理由」两分支；case_steps 转换规则表已有 step 级可选字段（retry/known_issue）文档惯例 |
| op-coverage-matrix（SKILL.md 内） | **N/A** | post_delay 非 op，total=124/mapped=21 与 EXPECTED_OP_COUNT=45 均不变 |

> 若 DQ-047-01 采纳推荐 A（补 SKILL.md），修改发生在 Story 执行阶段按 CR 决策执行，并追加 SKILL.md 修订记录行（同 CR-039 v1.8 追溯补记模式）；本阶段不修改任何正式对象。

## 三、CP2 Decision Brief 输入（决策项）

### DQ-047-01 | scope | SKILL.md 是否补 post_delay step 级参数说明（量化验收第 4 条）

- **推荐 A**：补一行说明（case_steps step 可选字段 `post_delay`，支持 `"5s"` / `"500ms"` / 纯数字秒；叠加在 `OP_INTERVAL_SEC` 之上；仅 `--execute` 模式生效；None/非法值按 0.0 处理）+ SKILL.md 修订记录追加一行。
- **备选 B**：不补，在 CP6/CP8 写 N/A 理由（如「SKILL.md 未系统文档化 step 级执行参数」）。
- **优劣**：A 使参数对用例作者可发现，与既有惯例一致（转换规则表已文档化 step 级可选字段 retry/known_issue，v1.3 修订记录可证），成本一行+一行修订记录；B 零文档成本但形成隐性参数，且 N/A 理由与「retry/known_issue 已文档化」事实相悖，说服力弱。
- **影响/风险**：A 为纯文档增量，无代码风险，需遵守修订记录规则；B 的风险是后续用例作者无法发现该能力，回源价值打折。
- **回退/切换**：A 与 B 可随时互切（纯文档）；若 A 采纳后认为文档面噪声大，可在下一 CR 移除。

### DQ-047-02 | implementation | workflow_mode 判定（fast-lane vs standard）

- **推荐 A**：standard（规则默认值；同 CR-039 / CR-046 回源型 CR 先例；CP3 预判 N/A 仅审计确认、CP5 走 technical-note，全链路 CP0-CP8 保留但各 CP 轻量，与 CR-047.md Checkpoint Index 已枚举的 CP 结构一致）。
- **备选 B**：fast-lane（本 CR 单文件+单测、无运行授权、无架构变更，形式上符合「低风险轻量实现」）。
- **优劣**：A 审计链完整，且与并行推进的 active CR-036（声明 case_runner.py 影响面，仅剩流程收尾）衔接清晰——CR-036 收尾验证若在本 CR 落地后执行，其基线需注明包含 post_delay 增量，standard 全链路证据使该注明有据可查；B 省时，但「快车道 × 并行 CR 影响面重叠」组合在本仓无先例，且 fast-lane 仍不可跳过 CP6/CP7/CP8，节省有限。
- **影响/风险**：B 的风险是并行 CR 基线注明缺乏标准 CP 证据链支撑；A 的代价是多走几个人工门禁（CP2/CP5/CP8），但均为轻量内容。
- **回退/切换**：A 内 CP3 已预判 N/A、CP5 已预判 technical-note，实际开销低；若推进中出现多 Story 依赖或文件所有权冲突信号，按规则必须留在 standard（本判定不可向 fast-lane 降级）。

### 归入 agent 默认处理（不升级为用户决策，仅审计记录）

1. `_parse_post_delay` 落点（`OP_INTERVAL_SEC` 常量定义之后）与函数内部实现细节。
2. 新增单测的文件组织（新文件 vs 既有文件；守护单测语义反转建议留在 `test_tg_op_mapping.py` 原位）。
3. `test_case_runner_no_post_delay_backport` 语义反转的断言写法（零残留 → 回源后存在：函数定义 + 1 处调用）。
4. 锚点漂移（L2437-2440 / L2511 / L2521）以符号定位为准的执行口径（CR-036 同款约束，CR-047 已声明）。
5. rollback 循环不受影响的守护测试写法。
6. dry-run 不生效（`continue` 先于睡眠行）为既有行为，单测可顺带断言，无需单独决策。

## 四、范围基线草案（供 host-orchestrator 汇总 CP2 Decision Brief）

### In Scope

1. `skills/case-execution/scripts/case_runner.py`：新增 `_parse_post_delay`（解析 `"5s"` / `"500ms"` / 纯数字秒；None / 空串 / 非法值返回 `0.0`）+ execute_steps 主循环睡眠行叠加 `time.sleep(OP_INTERVAL_SEC + _parse_post_delay(step.get("post_delay")))`（`idx < n - 1` 分支，注释标注 CR-047 用途）。
2. 新增单测 ≥ 8 用例：解析边界（None / 数字 / "5s" / "500ms" / 空串 / 非法串 / 负数）+ monkeypatch `time.sleep` 断言叠加值 + rollback 循环不受影响守护。
3. `tests/test_tg_op_mapping.py` post_delay 源码守护单测语义更新为「回源后存在」并通过（目标 3）。
4. SKILL.md `post_delay` 参数说明（按 DQ-047-01 决策；A 则一行 + 修订记录，B 则 N/A 理由）。

### Out of Scope

1. rollback 清理循环（L2513，`i > 0` 分支）的 post_delay 支持——工作区未改，保持纯 `OP_INTERVAL_SEC`，本 CR 同样不动。
2. 真机验证（BL-039-02 独立 runtime_authorization，DQ-039-04 三不授权不变）。
3. sw3 族多 op 扩展（BL-039-04）与 `next_hop_a` 多下一跳数组化（BL-039-05），保持 candidate。
4. op_mapper / op-coverage-matrix 任何变更（post_delay 非 op；EXPECTED_OP_COUNT=45、total=124/mapped=21 不变）。
5. post_delay 语义扩展（pre_delay、全局默认值、yaml 时间单位扩展等）。
6. docs/product 五文档（ptm-tse 域，见第二节 N/A 判定）。

### 排除项确认（CP2 范围基线要点）

- CR-039 已对 post_delay 精细排除（grep 零残留 + 源码级守护单测），**本 CR 是 post_delay 回源的唯一通道**，不存在双通道覆盖风险。
- CR-036 并行推进：其 A/B/C/D 代码已全部落地不再改代码，无文件写冲突；**若 CR-036 收尾验证在 CR-047 落地后执行，其基线需注明包含 post_delay 增量**（CR-047 冲突预检结论，建议写入 CP2 范围基线）。

## 五、open_questions（灰区，交 host-orchestrator 汇总）

1. **台账状态同步时机**：台账 CR-039-FU-047 已标 active（已转正式 CR-047），但 BACKLOG-CR-039 BL-039-01 卡片状态仍为 open（待用户决定启动 CR-047）。建议 host-orchestrator 在 CR-047 CP8 关闭时同步 BACKLOG 卡片状态（open → closed），避免状态悬挂。非阻塞。
2. **工作区锚点行号三处漂移**（卡片 L2437-2440 → 实测 L2511 → 最终 L2521）：CR-047 已用双标注 + 符号定位约束收敛，但若工作区在回源落地前再次漂移，需以符号定位（`OP_INTERVAL_SEC` 常量后定义 / 主循环 `idx < n - 1` 分支睡眠行）为准重锚。非阻塞，属实现期 agent 默认处理口径，提请 host-orchestrator 知悉。
3. **SKILL.md 文档落点**（若 DQ-047-01=A）：post_delay 说明放 case_steps 转换规则表新行（与 retry/known_issue 同位，建议）还是 Gotcha 编号条目——实现细节，agent 默认处理，不需用户决策。

## 六、CP1/CP2 就绪声明

- 阶段零调研完成，CR-047 与来源链零事实冲突。
- 文档增量判定：docs/product 五文档全部 N/A（域外），SKILL.md 待 DQ-047-01。
- 决策项 2 项（DQ-047-01 / DQ-047-02），均已给推荐 + 备选 + 优劣 + 风险 + 回退。
- 范围基线草案（in-scope 4 项 / out-of-scope 6 项 / 排除项确认 2 条）已就绪，供 host-orchestrator 生成 CP2-CR047.md Decision Brief。
- 本阶段未修改任何正式对象文件；未发起 CP2 门禁（host-orchestrator 唯一发起者）。
