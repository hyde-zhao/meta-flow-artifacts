---
checkpoint: CP6
checkpoint_id: CP6-STORY-056-05-mark-subcommand
cr_id: CR-056
story_id: STORY-056-05
story_slug: mark-subcommand
wave: 2
status: PASS
checked_at: 2026-09-03T23:30:00Z
---

# CP6 编码完成检查 — STORY-056-05 case_ledger.cmd_mark 实体化

## Entry Criteria

- Story `STORY-056-05`（`process/DEVELOPMENT-PLAN-CR-056.yaml`，Wave 2，depends_on=[STORY-056-04]，same-file-serial）由 host-orchestrator 派发实现
- 设计证据：`process/HLD-CR-056.md` v0.9 §7.3（六步内部动作）+ §5.3（git 自动 commit）+ §4.1（迁移矩阵/连带清除）+ §10（失败路径）（technical-note，design-already-in-hld）
- 文件所有权：primary=`skills/case-execution/scripts/case_ledger.py`；056-04 交接确认 case_runner 委托分支与 parser 参数面已就绪，本 Story case_runner 零改动
- LCQ-STORY-056-03-01 已由用户定案（仅 mode=execute 轮计入验收轮数）

## Checklist

| # | 检查项 | 结果 | 证据 |
|---|---|---|---|
| 1 | 实现对象清单可追溯（6 项，工程资产仅 case_ledger.py） | PASS | IMPLEMENTATION.md §实现对象清单；git diff --stat case_runner.py 维持 056-04 的 +678/-7 实测 |
| 2 | ① 迁移合法性：md+台账双读（台账为准，漂移 WARNING）、validate_transition、同值自环预判跳过（056-03 缺口 #5 定案）、台账无登记建档 | PASS | 冒烟 H01~H03/D03/G02/F03/J03 |
| 3 | ② note 规则：回退 debug / 废弃 deprecated 缺 --note 拒绝该项；debug→verify 缺 note 仅 WARNING | PASS | 冒烟 E01/F01（拒绝）+ A01（带 note 主链） |
| 4 | ③ EVIDENCE_REQUIRED 三边 + LCQ 定案（仅 mode=execute 轮计入连续 PASS，dry-run 打断计数）；无轮数门槛边容器追溯检查 | PASS | 冒烟 B01/C01/C02/C04/D01/L01~L05；056-03 未实现 mode 过滤的事实偏差已在设计缺口反馈 #2 留痕并落地 |
| 5 | ④ 写面：指纹不变式（打标后 verify_case_state ok）/ md 行原位替换与块尾插入 / 连带清除（md 删行 + 台账 None）/ append_mark_history 先行 → entry 更新后行 / ledger_lock + 原子写 / 新建条目 path 派生 cases/te 前缀 | PASS | 冒烟 A02~A06/E02~E06/J03 |
| 6 | ⑤ git 自动 commit：逐用例独立 commit、message `mark(case): <id> <旧>→<new>`（双域）、--no-commit、非 git/失败降级 WARNING、永不 push/force、锁外执行 | PASS | 临时 git 仓库产物实测 A07~A09/E05/H02/J01/J02 |
| 7 | ⑥ 降级路径：无 PyYAML exit 5 无 traceback（load 前置收敛）/ save_ledger 失败不整体回滚并提示人工回退 / 圈选空与不可定位台账根 exit 2 | PASS | 冒烟 K01/K02/I04 |
| 8 | 圈选与入口：--case-file 直取（圈选参数忽略 WARNING）/ --cases-dir 圈选（mark in 并集 + cicd none 语义）/ 互斥与存在性 / 圈选值域校验 / 摘要表 stdout 逐条 + rc 0/1/2 | PASS | 冒烟 P01~P06/I01~I04/J01b/A01 |
| 9 | 验证：py_compile 3.11/3.12 + pytest 88 passed + 冒烟 55/0 + 056-04 冒烟回归 70/0 | PASS | 见 IMPLEMENTATION.md §验证结果；F2 过渡断言更新属预期行为演进（冒烟脚本不落仓库） |
| 10 | 正式 pytest 文件 N/A 理由成立（后置 STORY-056-07；冒烟 55 断言为蓝本） | PASS | IMPLEMENTATION.md §单元测试与 Fixture 计划 |
| 11 | 安全边界（不触网、不触真实设备、永不 push/force、ptm-cases 与 312 条真实用例零触碰、冒烟全 /tmp 临时夹具） | PASS | IMPLEMENTATION.md §实现前置检查 + git 产物验证均在 /tmp/cr056-05-smoke |
| 12 | Agent Dispatch Evidence 存在（meta-dev subagent 真实调度） | PASS | 本次 Task/Subagent 调度；agent_id/thread_id 由 host-orchestrator 回填 AGENT-DISPATCH-LEDGER |

## Exit Criteria

- 全部 BLOCKER/HIGH 项 PASS；Story 状态 `planned → ready-for-verification`
- Return Packet / Evidence Index 已写入，CP7 可直接消费；STORY-056-06 接力要点见 IMPLEMENTATION §后续交接

## Deliverables

- `skills/case-execution/scripts/case_ledger.py`
- `process/stories/STORY-056-05-mark-subcommand-IMPLEMENTATION.md`
- `process/checks/CP6-STORY-056-05-mark-subcommand-CODING-DONE.md` + `.result.json`
- `process/returns/STORY-056-05-mark-subcommand-CP6.return.json` + `process/evidence/STORY-056-05-mark-subcommand-CP6.index.json`
- `DEV-LOG.md` 追加段（含补记 056-03 条目）、`process/DEVELOPMENT-PLAN-CR-056.yaml` 状态更新

## 人工审查结果

（待 host-orchestrator / meta-qa CP7 回填）
