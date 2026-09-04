---
doc_type: backlog
id: BACKLOG-CR-039
cr_ref: CR-039
created_at: "2026-08-19"
owner: host-orchestrator
status: "open"
purpose: CR-039 收敛后不并入本 CR 的待回源项与后续候选，防静默遗漏（DQ-039-05=B）
---

# BACKLOG-CR-039：CR-039 范围外待办与后续候选

> 来源：CR-039 CP2 澄清门（approve 2026-08-19）。本台账只登记不执行；推进任一项需用户拍板后转正式 CR 或并入既有 CR。

## 1. 待回源项（工作区已实现、canonical 未回源，**禁止在 CR-039 回源 case_runner 时整文件覆盖引入**）

| 编号 | 项目 | 工作区位置（排除行锚点） | 说明 | 状态 |
|------|------|------------------------|------|------|
| BL-039-01 | **CR-047 post_delay**（step 级执行后延迟） | case_runner.py `_parse_post_delay` 函数（L114-133：解析 "5s"/"500ms"/纯数字）+ `time.sleep(OP_INTERVAL_SEC + _parse_post_delay(step.get("post_delay")))` 改动（睡眠行，实测锚点 L2511） | 用途：断开 PPPoE 后等待会话 tear down 再发流。DQ-039-05=B：**独立另行回源（候选 CR-047），不并入本 CR**；CR-039 STORY-039-03 回源 case_runner 时必须排除这两块，不得整文件覆盖引入 | closed（2026-08-19 随 CR-047 回源完成关闭，CP8-DQ-047-039FU） |

## 2. 真机验证回归候选（独立 runtime_authorization）

| 编号 | 项目 | 说明 | 状态 |
|------|------|------|------|
| BL-039-02 | G1/G4/G5 真机回归 | trex 发流 / SSH 重启 trex-stl-te / sw3 telnet 真实同步，均属独立 runtime_authorization（DQ-039-04 明确本 CR 不授权，仅 static-only）。CR-039 交付后如需真机对拍，另发起授权决策 | candidate |
| BL-039-03 | 工作区 KNOWN_FAIL 2 项流量回退验证 | 工作区实测 26/28 step PASS 中剩余 2 处流量回退验证已标 KNOWN_FAIL 归档；是否追验由用户决定 | candidate |

## 3. 后续 CR 候选（Deferred / DEF-039 项）

| 编号 | 项目 | 来源 | 说明 | 状态 |
|------|------|------|------|------|
| BL-039-04 | sw3 族多 op 扩展 | DEF-039-02 | 当前仅 `sw3_sync_pppoe_route` 单 op（telnet 旁路）；SW3 上其他配置动作（如 PPPoE Server 侧查询/会话管理）待实战需求定义后扩展 | candidate |
| BL-039-05 | `next_hop_a` 多下一跳数组 | DEF-039-03 | 当前 `nodes.dut1.next_hop_a` 为单值（80.0.0.2，策略路由 A 下一跳 1 GW-A）；策略路由 A 下一跳 2（GW-B）等多下一跳场景需数组化扩展（`next_hop_a[0]` 占位符语法待定） | candidate |

## 4. 文档核对项

| 编号 | 项目 | 说明 | 状态 |
|------|------|------|------|
| BL-039-06 | op-coverage-matrix sw3 旁路章节落地核对 | CR-039 STORY-039-04 已新增 sw3 旁路章节（total=124/mapped=21 不变，DQ-039-02 两套口径）；CP7/CP8 验证时核对章节与回源后代码行为一致（`_execute_sw3_op` 走 telnet 旁路、不进 OP_ID_TO_SUBCOMMAND 的 ptm-atomic CLI 语义） | open（随 CR-039 CP7 核对） |
