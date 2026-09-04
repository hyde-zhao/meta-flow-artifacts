---
doc_type: follow_up_tracking
cr_id: CR-047
created_at: "2026-08-19"
owner: host-orchestrator
status: open
parent_cr: "CR-038"
source_candidate: "CR-039-FU-047（BL-039-01，DQ-039-05=B）"
release_decision: READY_WITH_RISK
source: "CR-047 CP7 PASS_WITH_RISK（R-1/R-2）+ STORY-047-01 open_items（OPI-047-01）+ VERIFICATION-REPORT P-1 + BACKLOG-CR-039"
---

# CR-047 Follow-Up Tracking 台账

> CP8 后续候选台账。所有候选默认 `candidate`（不占执行锁），只有用户决定推进某项时才创建正式 CR。
> 状态取值：`candidate` / `active` / `blocked` / `spike_candidate` / `converted-to-spike` / `closed` / `cancelled` / `superseded`。

## 候选清单（5 项 candidate）

| 候选 ID | 类型 | 状态 | 来源风险/编号 | 摘要 | 触发条件 |
|---|---|:---:|---|---|---|
| CR-047-FU-RUNTIME | 真机回归 | candidate | R-2（CP7） | **post_delay 真机 tear-down 效果实测**：断开 PPPoE 后等待会话 tear down 再发流的真机效果验证；不新建独立项，并入既有真机回归候选 BL-039-02 / CR-039-FU-RUNTIME（trex 发流 / SSH 重启 / sw3 telnet 三不授权，需独立 runtime_authorization） | 用户发起真机回归授权时激活（沿用 CR-039 台账） |
| CR-047-FU-P1 | 存量缺陷 | candidate | P-1（VERIFICATION-REPORT） | test_cr018_p2 GATE-4 2 处存量失败（test_gate4_passes_with_case_steps_atomic_op_contract / test_gate5_pass_sets_completed_phase）；CR-039 CP7 stash 对拍已确认非回源类改动引入，非本 CR 归属；**沿用 CR-039-FU-P1 台账项，不重复建项** | 用户决定修复时（CR-039-FU-P1 激活即覆盖） |
| CR-047-FU-NEG | 语义扩展 | candidate | R-1（RA-047-01 / OPI-047-01） | **负数 post_delay clamp**：当前透传为固化语义（`-1` / `"-1"` -> -1.0，单测 test_parse_post_delay_negative_passthrough 锁定，与工作区实测一致）；\|负值\| >= OP_INTERVAL_SEC 时 `time.sleep` 总和为负抛 ValueError；本 Story 按 SGQ-047-04 回源逐块一致原则不做 clamp | 实战出现负值用例或语义收敛需求时转正式 CR |
| CR-047-FU-SEM | 语义扩展 | candidate | OPI-047-01 / return packet notes | **post_delay 语义扩展 deferred**：负数 clamp / 其他时间后缀（如 `min`）；另工作区参考实现 apply_warming_up 内另一处 `_parse_post_delay` 调用（工作区 L1398，cur_delay 相关）不属本 Story 两块回源范围，canonical 未引入，是否回源待定 | post_delay 语义收敛 CR 或 apply_warming_up 回源需求出现时 |
| CR-047-FU-039FU047 | 台账关闭 | candidate | BL-039-01 / CR-039-FU-047（DQ-039-05=B） | **CR-039 台账项关闭确认**：BL-039-01（open）与 CR-039-FU-047（active，已转正式 CR-047）随本 CR 回源完成满足关闭条件；建议随 CP8 确认后置 closed（本台账登记为关闭动作候选，关闭执行方为 CR-039 台账/host-orchestrator status-sync） | CP8 人工门禁确认（CP8-DQ-047-039FU） |

## 不授权项（runtime_authorization，非台账 candidate 决策范围）

> 以下为独立运行授权决策项（DQ-039-04 沿用 / CP8-DQ-047-RUNTIME），不可 waive；CP8 通过不构成授权。

| # | 事项 | 关联对象 | 决策类型 |
|---|---|---|---|
| 1 | trex 真机发流 | 真机回归（BL-039-02） | runtime_authorization |
| 2 | SSH 重启 trex-stl-te | 真机回归（BL-039-02） | runtime_authorization |
| 3 | sw3 telnet 真机实连（H3C 登录 / undo + ip route-static 下发） | G5 sw3 路径（CR-039 R-2） | runtime_authorization |
| 4 | 凭据明文落盘 / 生产凭据读取 | ADR-02（本 CR 零凭据接触） | security（永久不授权明文形态） |
| 5 | git commit / push | CR-047 改动位于工作区未提交 | implementation（提交动作由用户确认后另行执行） |

## 关闭条件

- 所有 candidate 由用户决定推进（转正式 CR）或关闭（closed/cancelled/superseded）。
- 本台账不阻塞 CR-047 关闭；CR-047 关闭后本台账持续有效，供后续 CR 启动冲突预检消费。
