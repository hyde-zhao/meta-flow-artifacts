---
checkpoint: CP6
checkpoint_id: CP6-STORY-056-06-ledger-subcommand
cr_id: CR-056
story_id: STORY-056-06
story_slug: ledger-subcommand
wave: 2
status: PASS
checked_at: 2026-09-04T00:50:00Z
---

# CP6 编码完成检查 — STORY-056-06 case_ledger.cmd_ledger 实体化（只读聚合状态视图）

## Entry Criteria

- Story `STORY-056-06`（`process/DEVELOPMENT-PLAN-CR-056.yaml`，Wave 2，depends_on=[STORY-056-05]，same-file-serial）由 host-orchestrator 派发实现
- 设计证据：`process/HLD-CR-056.md` v0.9 §7.4（计数表/待办清单/漂移清单/submit_time 摘要）（technical-note，design-already-in-hld）
- 文件所有权：primary=`skills/case-execution/scripts/case_ledger.py`；056-05 交接确认 case_runner 委托分支与 parser 已就绪，本 Story case_runner 零改动
- splice 机制先读后复用：exec_task.splice_report + GEN marker（禁止重复实现）

## Checklist

| # | 检查项 | 结果 | 证据 |
|---|---|---|---|
| 1 | 实现对象清单可追溯（6 项，工程资产仅 case_ledger.py，既有函数零改动） | PASS | IMPLEMENTATION.md §实现对象清单；git diff --stat case_runner.py 维持 +678/-7 实测 |
| 2 | §7.4 ① 状态交叉计数：mark 5 态 × cicd_mark 4 档矩阵 + 行/列/总计 + deprecated 计数（G-10 语义标注） | PASS | 冒烟 A03~A06 |
| 3 | §7.4 ② 待办三类：accept 未提交 CICD（含稳定轮证据三态列，覆盖"稳定轮达标待提交"）/ verify_fail+cicd_verify_fail 待整改 / 漂移清单（verify_case_state 单一判定语义 + 文件缺失 + 台账无登记磁盘扫描，tde/README 忽略） | PASS | 冒烟 A07~A12（漂移 4 例四类全命中） |
| 4 | §7.4 ③ first_submitted_at 提交记录摘要（N 例 + 最早/最近 + 全列表） | PASS | 冒烟 A13 |
| 5 | --out splice 复用 exec_task.splice_report：首生成人工区模板 / 机器区 marker 重写（矩阵随台账更新）/ 人工区自定义内容逐字保留 / 无 marker 拒绝覆盖 exit 3 内容未变 / 原子写无 .tmp 残留 / 父目录自动创建 | PASS | 冒烟 C01~C07 |
| 6 | --mark 过滤（条目侧四段生效 + 值域校验）与降级双路径（exec_task 缺失 → --out exit 5 而 stdout 可用；无 PyYAML/损坏 → exit 5 无 traceback） | PASS | 冒烟 B01~B04/C08/C09/D01 |
| 7 | 只读边界：台账与全部用例 md 零写入（前后逐字节比对）、git 零操作、真实 cases 目录只读探针 rc=0 | PASS | 冒烟 E01/E02 + 真实探针记录 |
| 8 | 验证：py_compile 3.11/3.12 + pytest 88 passed + 冒烟 29/0 + 前序冒烟回归 70/55 全绿 | PASS | IMPLEMENTATION.md §验证结果；056-04 冒烟 F4 过渡断言随实体化更新（预期演进，留痕） |
| 9 | 正式 pytest 文件 N/A 理由成立（后置 STORY-056-07；冒烟 29 断言为蓝本）+ 07/08 收口清单已交接 | PASS | IMPLEMENTATION.md §未覆盖项 + §后续交接（pytest 蓝本清单/4 副本同步/SKILL.md ST-EX-18~20/文档提示项/archive-check 复用件） |
| 10 | 顺手项销项：resolve_evidence_rounds mode 过滤已在函数内实现（056-05），协调者描述与现状不符已留痕 | PASS | IMPLEMENTATION.md §实现前置检查 + 设计缺口反馈 #1；本 Story 零改动该函数 |
| 11 | 安全边界（只读聚合零写入、不触网不触设备、--out 仅写用户指定路径、真实 ptm-cases 零触碰） | PASS | IMPLEMENTATION.md §实现前置检查 + 冒烟 E 段 |
| 12 | Agent Dispatch Evidence 存在（meta-dev subagent 真实调度） | PASS | 本次 Task/Subagent 调度；agent_id/thread_id 由 host-orchestrator 回填 AGENT-DISPATCH-LEDGER |

## Exit Criteria

- 全部 BLOCKER/HIGH 项 PASS；Story 状态 `planned → ready-for-verification`
- Return Packet / Evidence Index 已写入，CP7 可直接消费；STORY-056-07/08 收口清单见 IMPLEMENTATION §后续交接

## Deliverables

- `skills/case-execution/scripts/case_ledger.py`
- `process/stories/STORY-056-06-ledger-subcommand-IMPLEMENTATION.md`
- `process/checks/CP6-STORY-056-06-ledger-subcommand-CODING-DONE.md` + `.result.json`
- `process/returns/STORY-056-06-ledger-subcommand-CP6.return.json` + `process/evidence/STORY-056-06-ledger-subcommand-CP6.index.json`
- `DEV-LOG.md` 追加段、`process/DEVELOPMENT-PLAN-CR-056.yaml` 状态更新

## 人工审查结果

（待 host-orchestrator / meta-qa CP7 回填）
