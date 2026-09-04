---
checkpoint: CP6
checkpoint_id: CP6-STORY-056-04-case-runner-mark-admission
cr_id: CR-056
story_id: STORY-056-04
story_slug: case-runner-mark-admission
wave: 2
status: PASS
checked_at: 2026-09-03T22:10:00Z
---

# CP6 编码完成检查 — STORY-056-04 case_runner mark/cicd_mark schema + run 准入 + 冻结校验

## Entry Criteria

- Story `STORY-056-04`（`process/DEVELOPMENT-PLAN-CR-056.yaml`，Wave 2，depends_on=[STORY-056-03]，dependency_type=interface）由 host-orchestrator 派发实现
- 设计证据：`process/HLD-CR-056.md` v0.9 §4.3/§4.4/§7.1/§7.2（technical-note，design-already-in-hld；C-1~C-8 锚点级清单随派发任务给出）
- 文件所有权：primary=`skills/case-execution/scripts/case_runner.py`；forbidden=`case_ledger.py`（接口只 import 不改）
- 上游接口已实物核对：case_ledger.py 594 行实际 API（MARK_VALUES/CICD_MARK_VALUES/ROLE_SCENE_POOLS/ROLE_SCOPE/LEDGER_FILENAME/load_ledger/verify_case_state/compute_case_fingerprint/cmd_mark/cmd_ledger/cmd_archive_check）

## Checklist

| # | 检查项 | 结果 | 证据 |
|---|---|---|---|
| 1 | 实现对象清单可追溯（6 项，工程资产仅 case_runner.py，+678/-7 行） | PASS | IMPLEMENTATION.md §实现对象清单；case_ledger.py/exec_task.py/init_cases_repo.py 零改动（git status 实测） |
| 2 | C-1/C-2：常量导入（降级门控）+ parse_frontmatter [3c] mark 枚举 / [3d] cicd_mark 枚举 / 不变式读点 + 返回键（含 4 个早退分支） | PASS | 冒烟 A1~A11；真实 te 库 312 md 只读解析 311 用例 0 error、mark 全缺省 debug、cicd_mark 全 None |
| 3 | C-3：filter_by_mark / filter_by_cicd_mark（in 并集、none→cicd_mark is None、空列表透传） | PASS | 冒烟 B1~B6 |
| 4 | C-4/C-5：run 5 新参数 + 参数级准入 exit 2（越权/非法组合/值域）+ 默认池与显式池圈选 + cmo mark≠accept 自动排除 + 过滤链叠加 + 无 --role 零变化 | PASS | 冒烟 C1~C11、D1~D13；88 例既有回归零破坏（--tag/--keyword 语义不变） |
| 5 | C-6：台账向上定位（symlink 场景 cases/te→cases/ 命中）+ LEDGER_MISSING fail-closed（factory/cicd exit 2，local WARNING）+ CASE_MARK_STALE 固定话术 G-6 可区分 + local 失配 WARNING 放行 + dry-run 生效（校验在 execute 判定前） | PASS | 冒烟 E1~E14（E1 为真实篡改构造，未 mock） |
| 6 | C-7：case.case_state {mark, cicd_mark, fingerprint_ok} 三态快照（4 个记录点）+ run 级 mark_admission 语境 + 既有字段兼容 | PASS | 冒烟 D2/D3/E4/E7/E12；generate_report/exec_task 按键消费不受影响 |
| 7 | C-8：mark/ledger/archive-check parser + 委托 case_ledger（validate/exec-report 同层、无启动校验/设备配置、模块缺失 exit 5） | PASS | 冒烟 F1~F6 |
| 8 | 任务清单外增量最小化并留痕（deprecated 吸收态不执行 G-8/G-06、mark_admission、跨域圈选 vs G-8 措辞冲突、台账定位口径） | PASS | IMPLEMENTATION.md §设计缺口反馈 #1~#4；均可一行回退，待 host-orchestrator 认可 |
| 9 | 验证：py_compile 3.11/3.12 + pytest 88 passed + 冒烟 69 断言全过 | PASS | `uv run --python 3.11 python -m py_compile …` PASS；`pytest skills/case-execution/tests/ -q` 88 passed；`/tmp/smoke_cr056_04.py` 69 pass / 0 fail（不落仓库） |
| 10 | 正式 pytest 文件 N/A 理由成立（后置 STORY-056-07；冒烟断言即蓝本） | PASS | IMPLEMENTATION.md §单元测试与 Fixture 计划 N/A 理由 + CR-056 Story 拆解 |
| 11 | 安全边界（不触网、不触真实设备、不读凭据、ptm-te-manaul/ptm-cases 零写入） | PASS | 冒烟全部 dry-run（无 --execute --authorized）；te 库仅只读解析 |
| 12 | Agent Dispatch Evidence 存在（meta-dev subagent 真实调度） | PASS | 本次 Task/Subagent 调度；agent_id/thread_id 由 host-orchestrator 回填 AGENT-DISPATCH-LEDGER |

## Exit Criteria

- 全部 BLOCKER/HIGH 项 PASS；Story 状态 `planned → ready-for-verification`
- Return Packet / Evidence Index 已写入，CP7 可直接消费；STORY-056-05 可按 same-file-serial 接力（main() 接线点与 cmd_mark 签名见 IMPLEMENTATION §后续交接）

## Deliverables

- `skills/case-execution/scripts/case_runner.py`
- `process/stories/STORY-056-04-case-runner-mark-admission-IMPLEMENTATION.md`
- `process/checks/CP6-STORY-056-04-case-runner-mark-admission-CODING-DONE.md` + `.result.json`
- `process/returns/STORY-056-04-case-runner-mark-admission-CP6.return.json` + `process/evidence/STORY-056-04-case-runner-mark-admission-CP6.index.json`
- `DEV-LOG.md` 追加段、`process/DEVELOPMENT-PLAN-CR-056.yaml` 状态更新

## 人工审查结果

（待 host-orchestrator / meta-qa CP7 回填）
