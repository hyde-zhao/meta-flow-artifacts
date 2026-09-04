---
doc_type: follow_up_tracking
cr_id: CR-039
created_at: "2026-08-19"
owner: host-orchestrator
status: open
parent_cr: "CR-038"
release_decision: READY_WITH_RISK
source: "CR-039 CP7 PASS_WITH_RISK（R-1~R-5）+ BACKLOG-CR-039（BL-039-01~06）+ VERIFICATION-REPORT P-1"
---

# CR-039 Follow-Up Tracking 台账

> CP8 后续候选台账。所有候选默认 `candidate`（不占执行锁），只有用户决定推进某项时才创建正式 CR。
> 状态取值：`candidate` / `active` / `blocked` / `spike_candidate` / `converted-to-spike` / `closed` / `cancelled` / `superseded`。

## 候选清单（9 项 candidate + 1 项 closed）

| 候选 ID | 类型 | 状态 | 来源风险/编号 | 摘要 | 触发条件 |
|---|---|---|:---:|---|---|
| CR-039-FU-047 | 待回源项 | closed | BL-039-01（DQ-039-05=B） | **CR-047 post_delay 独立回源**：case_runner `_parse_post_delay`（解析 "5s"/"500ms"/纯数字）+ 睡眠行（工作区实测锚点 L2511，卡片原写 L2437-2440 双标注）；用途为 PPPoE 断开后等待会话 tear down 再发流。本 CR 回源时已精细排除（grep 零残留 + 源码级守护单测） | 已随 CR-047 回源完成并 CP8 approve 关闭（2026-08-19，CP8-DQ-047-039FU） |
| CR-039-FU-RUNTIME | 真机回归 | candidate | BL-039-02（DQ-039-04） | **真机回归候选**：G1 大路由表 verify + sw3 telnet 端到端（H3C 登录 / undo+ip route-static 下发）+ 2 KNOWN_FAIL 流量回退追验；需独立 runtime_authorization（真机三不授权：trex 发流 / SSH 重启 trex-stl-te / sw3 telnet 实连） | 用户发起真机回归授权时激活 |
| CR-039-FU-R1 | 风险观测 | candidate | R-1（RA-039-01） | G1 真机 verify 超时 / dst_network 歧义 / 分页>100 单页查询未覆盖；超时按 DQ-039-01 备选 B（step-refs 优先）重评，不静默改 | 随 CR-039-FU-RUNTIME 真机回归时跟踪 |
| CR-039-FU-R3 | 环境回归 | candidate | R-3（CP7） | telnetlib py>=3.13 不可用分支无环境验证（仓内 uv 锁 3.11，惰性导入注释已声明） | 升级 Python 版本时回归 |
| CR-039-FU-R4 | 单测补齐 | candidate | R-4（CP7） | sw3 经 case_runner execute_steps 全链路 dry-run（retry/step-refs 包装）未单测；envelope 构建函数与 execute_op(dry_run=True) 两侧已覆盖 | 下一轮 case-execution 测试补齐 CR 或发现包装异常时 |
| CR-039-FU-R5 | 追验 | candidate | R-5（BL-039-03） | 工作区实测 26/28 step PASS 中剩余 2 处 KNOWN_FAIL 流量回退验证未追验（已归档工作区） | 用户决定追验时（可并入 CR-039-FU-RUNTIME） |
| CR-039-FU-P1 | 存量缺陷 | candidate | P-1（VERIFICATION-REPORT） | test_cr018_p2 GATE-4 2 处存量失败（test_gate4_passes_with_case_steps_atomic_op_contract / test_gate5_pass_sets_completed_phase）；stash 对拍确认非本 CR 引入，非本 CR 归属，另行跟踪 | 用户决定修复时（独立任务或并入相关 CR） |
| CR-039-FU-DEF02 | 功能扩展 | candidate | DEF-039-02（BL-039-04） | sw3 族多 op 扩展：当前仅 `sw3_sync_pppoe_route` 单 op（telnet 旁路）；SW3 其他配置动作（PPPoE Server 侧查询/会话管理等）待实战需求定义 | 实战需求出现时转正式 CR |
| CR-039-FU-DEF03 | 功能扩展 | candidate | DEF-039-03（BL-039-05） | `next_hop_a` 多下一跳数组化：当前单值（80.0.0.2，策略路由 A 下一跳 1 GW-A）；GW-B 等多下一跳场景需数组化（`next_hop_a[0]` 占位符语法待定） | 多下一跳策略路由场景需求出现时 |
| CR-039-FU-DOC06 | 文档核对 | closed | BL-039-06 | op-coverage-matrix sw3 旁路章节与回源后代码行为一致性核对：CP7-039-09 已核验（章节存在、total=124/mapped=21 不变、两套口径声明无矛盾） | 已随 CR-039 CP7 关闭（2026-08-19） |

## 不授权项（runtime_authorization，非台账 candidate 决策范围）

> 以下为独立运行授权决策项（CP8-DQ-039-RUNTIME / DQ-039-04），不可 waive；CP8 通过不构成授权。

| # | 事项 | 关联对象 | 决策类型 |
|---|---|---|---|
| 1 | trex 真机发流 | 真机回归（BL-039-02） | runtime_authorization |
| 2 | SSH 重启 trex-stl-te | 真机回归（BL-039-02） | runtime_authorization |
| 3 | sw3 telnet 真机实连（H3C 登录 / undo+ip route-static 下发） | G5 sw3 路径（R-2） | runtime_authorization |
| 4 | 凭据明文落盘 / 生产凭据读取 | sw3 密码仅 SW3_PASSWORD 环境变量（ADR-02） | security（永久不授权明文形态） |
| 5 | CR-047 post_delay 回源 | BL-039-01 / DQ-039-05=B | follow_up_tracking（用户决定推进时转正式 CR） |

## 关闭条件

- 所有 candidate 由用户决定推进（转正式 CR）或关闭（closed/cancelled/superseded）。
- 本台账不阻塞 CR-039 关闭；CR-039 关闭后本台账持续有效，供后续 CR 启动冲突预检消费。
