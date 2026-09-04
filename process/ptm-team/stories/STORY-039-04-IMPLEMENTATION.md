---
story_id: STORY-039-04
story_slug: docs-and-traceability
cr_id: CR-039
evidence_type: implementation
status: dev-complete
created_at: "2026-08-19"
taker: meta-dev
design_evidence_ref: process/stories/STORY-039-04.md ## 技术说明（technical-note，CP5 approve 2026-08-19）
---

# STORY-039-04 实现执行证据：文档与追溯

## 实现前置检查

| 检查项 | 结果 |
|--------|------|
| Story 状态与设计证据 | CP5 已 approve（process/checkpoints/CP5-CR039.md，2026-08-19），technical-note 为实现基线；本 Story 无代码依赖（`depends_on=[]`，W1 与 STORY-039-01 并行、文件互斥） |
| 文件所有权 | primary 7 项全部本 Story 独占（SKILL.md / op-coverage-matrix.md / CR-043/044/046.md / BACKLOG-CR-039.md / env-file 样例），无 shared、无 forbidden 冲突 |
| 代码零触碰约束 | 本 Story 仅写文档/档案/样例文件；`op_mapper.py` / `case_runner.py` / 单测文件均未修改（git status 中 op_mapper.py / test_op_mapper_rollback.py 的改动属 STORY-039-01 G1 区块，非本 Story） |
| 凭据约束 | env-file 样例 SW3 密码走 `password_env: SW3_PASSWORD` 环境变量引用，无明文（ADR-02）；静态正则核对通过 |

## 实现对象清单

| # | 文件 | 动作 | 内容 |
|---|------|------|------|
| 1 | `skills/case-execution/SKILL.md` | 修改 | Gotcha#8 `EXPECTED_OP_COUNT` 40->45（含 CR-043/044/046 来源行：40+2+2+1=45）；修订记录追加 v1.7（对齐工作区 CR-046 sw3 族）+ v1.8（CR-039 追溯补记来源行） |
| 2 | `docs/ptm-te/op-coverage-matrix.md` | 修改 | 新增「sw3 族（telnet 旁路，不计入 ptm-atomic atoms 映射）」章节（含 DQ-039-02 口径说明）；修订记录 v1.3；**total=124 / mapped=21 / unmapped=97 计数不变** |
| 3 | `process/changes/CR-043.md` | 新建 | 追溯补记档案：object update/verify（+2）；背景含 canonical 代码标注行号（L79-82/L239-249/L528-529/L1059 注释）；Checkpoint Index 标注追溯补记 N/A |
| 4 | `process/changes/CR-044.md` | 新建 | 追溯补记档案：service-pool config/delete（+2）；标注行号（L114-117/L422-430/L552-554/L736-746/L998-1005/L1059 注释） |
| 5 | `process/changes/CR-046.md` | 新建 | 追溯补记档案：sw3_sync_pppoe_route（+1，telnet 旁路）；canonical 代码回源锚点指向工作区行号（由 STORY-039-03 落码） |
| 6 | `process/BACKLOG-CR-039.md` | 新建 | CR-047 post_delay 独立回源候选（BL-039-01）+ 真机回归候选 + DEF-039-02/03 候选 + 旁路章节核对项 |
| 7 | `skills/case-execution/templates/env-file.pppoe-sw3.next_hop_a.example.yaml` | 新建 | PPPoE + SW3 + 双下一跳 env-file 样例（dut1.next_hop_a=80.0.0.2 + sw1 节点契约） |

## 设计契约映射

| 技术说明契约 | 落地位置 | 验证 |
|--------------|---------|------|
| Gotcha#8 40->45 + 来源行（40 + CR-043 object update/verify 2 + CR-044 service-pool config/delete 2 + CR-046 sw3_sync_pppoe_route 1 = 45） | SKILL.md Gotcha#8（与工作区 v1.7 文本同构 + 显式算式） | 正则核对 = 45 |
| 修订记录 v1.7（对齐工作区 CR-046，44->45） | SKILL.md 修订记录 v1.7（文本与工作区 v1.7 一致，末尾注明 canonical 由 CR-039 追溯补记） | 行存在 |
| 修订记录 v1.8（CR-043/044/046 来源行 + CR-039 追溯补记） | SKILL.md 修订记录 v1.8（含档案路径、DQ-039-03=B、DQ-039-02 口径） | 行存在 |
| op-coverage-matrix sw3 旁路章节 + total=124/mapped=21 不变（DQ-039-02） | op-coverage-matrix.md「sw3 族（telnet 旁路…）」小节 + 口径说明两段 | 数字核对不变 |
| CR-043/044/046 最小档案（DQ-039-03=B：背景含代码标注位置、变更内容、验证方式、来源、Checkpoint Index 标注追溯补记、与 CR-039 关系） | process/changes/CR-043.md / CR-044.md / CR-046.md 六节齐备 | 逐份核对 |
| BACKLOG CR-047 post_delay（DQ-039-05=B） | BACKLOG-CR-039.md BL-039-01（排除行锚点 `_parse_post_delay` L114-133 + 睡眠改动行） | 行存在 |
| env-file 样例字段契约（next_hop_a=80.0.0.2 / SW 节点 node_type=SW / role=pppoe_server / host / username / password_env=SW3_PASSWORD） | env-file.pppoe-sw3.next_hop_a.example.yaml | YAML 解析 + 字段断言通过 |

## 单元测试与 Fixture 计划

- **N/A（文档/追溯类 Story，validation_mode=static-only + review-only）**。无新增代码，无单测 fixture。
- 等价验证：Python 静态核对脚本（本次已执行，全部 PASS）：
  1. SKILL.md Gotcha#8 `EXPECTED_OP_COUNT=45` 正则断言 + v1.7/v1.8 存在
  2. op-coverage-matrix total=124 / mapped=21 / unmapped=97 不变 + sw3 章节存在
  3. CR-043/044/046 / BACKLOG 档案存在且非空
  4. env-file 样例 `yaml.safe_load` 解析成功 + dut1.next_hop_a=80.0.0.2 + sw1 节点契约字段 + 无明文密码正则
- 最终一致性由 STORY-039-03（W3）收口：canonical 代码 `EXPECTED_OP_COUNT` 当前为 44（G1 已合入，G5 未回源），回源后 =45 与 SKILL.md 对齐（SM-039-01 终验）。

## 最小实现切片

| 切片 | 内容 | 验证 |
|------|------|------|
| S1 | SKILL.md Gotcha#8 + 修订记录 v1.7/v1.8 | PASS |
| S2 | op-coverage-matrix sw3 旁路章节 + v1.3 | PASS |
| S3 | CR-043/044/046 三份追溯档案 | PASS |
| S4 | BACKLOG-CR-039.md | PASS |
| S5 | env-file 样例 | PASS |

## 平台差异处理

N/A。本 Story 全部为 canonical 仓库内文档/档案/样例文件，无平台安装目录、无跨平台路径推断；SKILL.md 与模板随 ptm-atomic 既有安装链路分发，无新增平台适配点。

## 验证结果

- 静态核对脚本全部 PASS（见「单元测试与 Fixture 计划」4 项断言）。
- 代码零触碰确认：git 工作区中本 Story 改动仅为 SKILL.md、op-coverage-matrix.md、env-file 样例（+process/ 档案，不入 canonical git 跟踪）；`op_mapper.py` / `test_op_mapper_rollback.py` 的既有改动属 STORY-039-01（G1），非本 Story 产物。
- 已知中间态（非缺陷，按计划）：canonical 代码 `EXPECTED_OP_COUNT=44`，SKILL.md 已写 45——文档先行落基线（Story 卡片「依赖」节明确：值=45 已由 DQ-039-02 锁定，本 Story 先行落文档基线，最终与 STORY-039-03 代码回源一致）。CP7 验收时以「代码回源后 = 45」为准。

## 未覆盖项

- 真机验证（trex 发流 / SSH 重启 / sw3 telnet 真实同步）：独立 runtime_authorization，DQ-039-04 不授权（BACKLOG BL-039-02）。
- CR-047 post_delay 回源：独立 CR（DQ-039-05=B，BACKLOG BL-039-01）。
- sw3 族多 op 扩展 / next_hop_a 多下一跳数组：DEF-039-02/03（BACKLOG BL-039-04/05）。
- 工作区 SKILL.md 修订记录中 v1.7 之外是否存在其他 canonical 未同步版本行：已核对，工作区修订记录止于 v1.7（2026-08-17），canonical 补 v1.7/v1.8 后与工作区语义对齐。

## 设计缺口反馈

- 无阻断缺口。一处行号偏差记录：Story 卡片写工作区 case_runner 睡眠改动在「L2437-2440」，实测定位为 L2511（`time.sleep(OP_INTERVAL_SEC + _parse_post_delay(step.get("post_delay")))`；工作区文件与 Story 拆解时行号漂移）。已在 BACKLOG-CR-039 BL-039-01 中以实测锚点 + Story 原始区间双标注，不影响回源排除语义。
- 建议（仅记录，不阻塞）：后续 CR 涉及「代码标注行号」时，档案中同时标注函数名锚点（如 `_parse_post_delay`），降低行号漂移导致的追溯失效。

## G6 M9 归属补记（security_constraints 留痕，T-3 台账项）

- **M9（静态路由回退多故障场景，DUT2 TE0_9/dut.port3 出接口 + TG TE2_2/tg.port2 收流）归属为 T-3 台账项（用例侧跟踪）**：M9 场景用例的编写与执行归 ptm-tde 用例侧后续跟踪，不在 CR-039 范围。
- 本 Story 已落 M9 所需**环境字段承载**：env-file 样例含 `dut1.port3`（TE0_9，静态路由回退出接口）与 `dut1.next_hop`（110.0.0.2，回退下一跳 GW-C），M9 用例可直接经 `${ENV.dut.port3}` / `${ENV.dut.next_hop}` 引用。此留痕不阻塞本 CR 门禁。

## 后续交接

- **交接 meta-qa（CP7）**：验证入口为本文档「验证结果」4 项静态断言 + SM-039-01（文档侧 45）/ SM-039-05（档案 + 修订记录）核对；风险提示：canonical 代码当前 44，需在 STORY-039-03 完成后统一终验（避免在 W3 前误判文档-代码不一致为 FAIL）。
- **交接 STORY-039-03**：`EXPECTED_OP_COUNT=45` 文档基线已锁定（SKILL.md Gotcha#8 + v1.8 来源行）；回源后无需再改 SKILL.md 计数。
- **交接 host-orchestrator**：CR-043/044/046 档案已落地，CR-INDEX / ledger 同步由 host-orchestrator 收口；BACKLOG-CR-039 待 CP8 分流。
