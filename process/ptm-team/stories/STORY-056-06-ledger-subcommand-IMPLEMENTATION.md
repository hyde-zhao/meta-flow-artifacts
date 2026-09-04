---
story_id: "STORY-056-06"
story_slug: "ledger-subcommand"
cr_id: "CR-056"
wave: 2
design_evidence_type: "technical-note"
design_evidence_ref: "process/HLD-CR-056.md v0.9 §7.4（ledger 子命令契约）+ §4.1（状态语义与待办口径）+ §7.1（first_submitted_at / verify_case_state 语义源）+ §12 G-10"
implementation_status: "code-complete-ledger-subcommand"
executed_by: "meta-dev"
executed_at: "2026-09-04T00:40:00Z"
revision: "r1（cmd_ledger 骨架 → 实体；case_runner.py 零改动，委托分支复用 056-04 接线）"
---

# STORY-056-06 实现执行证据 — case_ledger.cmd_ledger 实体化（只读聚合状态视图 + --out splice 机器区重写/人工区保留）

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| 设计证据（technical-note，design-already-in-hld）已确认 | PASS | `process/HLD-CR-056.md` v0.9 §7.4 四段视图（计数表/待办/漂移/submit_time 摘要）；host-orchestrator 派发任务 1~4 |
| 依赖门控满足（depends_on=[STORY-056-05]，same-file-serial） | PASS | STORY-056-05 已验收；交接要点确认：main() 委托分支与 parser（`--cases-root` required / `--mark` list / `--out`）已就绪，本 Story 只实体化 `case_ledger.cmd_ledger` |
| 文件所有权无冲突 | PASS | primary=`skills/case-execution/scripts/case_ledger.py`；**case_runner.py 零改动**（git diff --stat 维持 056-04 的 +678/-7，实测确认） |
| splice 机制先读后复用 | PASS | `exec_task.py` 实测：`splice_report(old, parts) -> (content, refuse_reason)`，parts=`{frontmatter, machine, manual_template}`，marker `GEN_BEGIN/GEN_END`（`<!-- case-execution:report-gen:begin/end -->`），marker 缺失/错序拒绝覆盖（fail-closed 防吞人工区）；cmd_ledger 直接调用，零重复实现 |
| AI 任务清单存在 | PASS | 派发任务：实体化（三类待办/交叉计数/first_submitted_at/--out splice/--mark 过滤/exit 5 收敛）+ 验证（py_compile 双版本、88 例回归、冒烟 ≥20 断言、--out 二次运行断言）+ 顺手项判断 |
| 顺手项事实核对 | PASS | 协调者描述"execute 过滤只在 cmd_mark 调用处实现（函数本身未过滤）"**与现状不符**：STORY-056-05 已把 mode 过滤实现于 `resolve_evidence_rounds` **函数体内**（appearances 先滤 `mode=="execute"` 再计连续，docstring 已更新）；直接证据 = 056-05 冒烟 L01~L05 **直接调用该函数**断言（L02 纯 dry-run 容器返回"无 mode=execute"）。不存在双口径，本 Story 零改动该函数（详见设计缺口反馈 #1） |
| 安全校验 | PASS | 只读聚合：台账与全部用例 md 零写入（冒烟 E01/E02 前后逐字节比对）；git 零操作；不触网不触设备；--out 仅写用户指定路径 |

## 实现对象清单

| # | 对象 | 操作 | 说明 |
|---|---|---|---|
| 1 | `skills/case-execution/scripts/case_ledger.py` | 修改 | cmd_ledger 骨架 → 实体 + 视图渲染辅助 5 个（`_LEDGER_MANUAL_TEMPLATE`/`_scoped_entries`/`_accept_evidence_of`/`_resolve_case_file`/`_drift_rows`/`_ledger_view_markdown`/`_ledger_frontmatter`）+ 模块头更新 |
| 2 | `process/stories/STORY-056-06-ledger-subcommand-IMPLEMENTATION.md` | 新建 | 本证据 |
| 3 | `process/checks/CP6-STORY-056-06-ledger-subcommand-CODING-DONE.md` + `.result.json` | 新建 | CP6 门禁 |
| 4 | `process/returns/STORY-056-06-ledger-subcommand-CP6.return.json` + `process/evidence/STORY-056-06-ledger-subcommand-CP6.index.json` | 新建 | Return Packet / Evidence Index |
| 5 | `DEV-LOG.md` | 追加 | 056-06 实现段 |
| 6 | `process/DEVELOPMENT-PLAN-CR-056.yaml` | 更新 | STORY-056-06 status planned → ready-for-verification + execution_note |

禁改边界实测：`resolve_evidence_rounds` / `cmd_mark` 及 056-03/05 全部既有函数零改动（仅文件尾部追加视图段 + 模块头两行）；`git status --porcelain` 其余 modified 均为前序 Story 未提交改动；真实 ptm-cases 与 312 条用例只读探针零写入。

## 设计契约映射

### §7.4 视图四段（stdout 与 --out 同一渲染源 `_ledger_view_markdown`）

| HLD 契约 | 实现 |
|---|---|
| 状态交叉计数表 | mark 5 态（MARK_VALUES 序）× cicd_mark 4 档（无/cicd_verify/cicd_verify_fail/cicd_accept）矩阵 + 行合计/列合计/总计 + 独立 `deprecated 计数` 行（标注 G-10 吸收态语义）；条目缺 mark 按 debug、缺 cicd_mark 按无口径 |
| 待办清单·accept 未提交 CICD | `mark==accept 且 cicd_mark=None`；含"验收证据（稳定轮，DQ-056-02）"列——`_accept_evidence_of` 自 mark_history 倒序找最近 accept 记录，rounds≥3 → "达标（连续 N 轮，evidence）"、rounds=None → "轮数未记录"、<3 → "未达标"、无记录 → "无 accept 打标记录"（覆盖 HLD"accept 且稳定轮达标待提交 CICD"而不引入额外扫描） |
| 待办清单·待整改 | `mark==verify_fail 或 cicd_mark==cicd_verify_fail`（含状态与 path 列） |
| 待办清单·漂移/异常 | `_drift_rows`：① 条目侧经 `_resolve_case_file`（候选序：cases_root 父目录/DQ-056-04 口径 → cases_root 相对 → 文件名兜底）定位 md 后调 `verify_case_state`（mark/cicd_mark 不一致 + content_sha256 失配，判定语义单一来源）；② 文件缺失 / 读取失败显式成行；③ 磁盘扫描 `_iter_case_files(cases_root)` + `_extract_case_id`（仅认 frontmatter 用例编号）补"台账无登记"——tde 设计源/README 无 frontmatter 天然跳过（冒烟 A12） |
| submit_time 摘要 | `first_submitted_at` 共 N 例 + 最早/最近 + 全列表（DQ-056-09：保留于台账，frontmatter 四字段不参与） |

### 命令面其余条目

| 契约 | 实现 |
|---|---|
| `--mark` 过滤 | `_scoped_entries` 工厂域过滤（骨架值域校验保留），对条目侧四段全部生效；"台账无登记"磁盘扫描不受过滤（无条目可过滤，视图头注明口径） |
| `--out` splice | 复用 `exec_task.splice_report`：首生成 = frontmatter（doc/cases_root/generated_at/cases_total/scope）+ GEN marker 包裹机器区 + 人工区占位模板；已存在 → marker 重写机器区、人工区原样保留；marker 缺失/错序 → 拒绝覆盖 **exit 3**（fail-closed）；写入走 tmp+os.replace 原子写（save_ledger 同惯例）+ 父目录 makedirs（generate_report 同惯例）；exec_task 模块缺失 → --out 不可用 **exit 5**（stdout 视图不受影响） |
| 降级 | 无 PyYAML / 台账损坏 → `load_ledger` 异常收敛 **exit 5** 无 traceback（cmd_mark 同款模板）；只读视图不产出 |
| 退出码 | 0 成功；2 参数错误；3 --out 拒绝覆盖/写入失败（模块头已补 3 档说明，同 exec-report 语义）；5 依赖/台账不可用 |

## 单元测试与 Fixture 计划

- 本 Story 验证 = 交互冒烟（`/tmp/smoke_cr056_06.py`，**不落仓库**，29 断言 > 任务要求 20）+ 既有 88 例回归 + 056-04/05 冒烟回归（70/55 断言）；**正式 pytest 文件 N/A（后置 STORY-056-07）**，理由同 056-03/04/05（CR-056 Story 拆解明确测试收口归 056-07）；冒烟 29 断言即 ledger 视图部分用例蓝本。
- Fixture：`/tmp/cr056-06-smoke/cases` = 台账 11 条目（debug×2 / verify×1 / verify_fail×2 / accept×6 遍布 cicd 4 档，含 mark_history rounds=3 与 rounds=None 两种验收证据形态、first_submitted_at 5 例）+ 4 类漂移夹具（md/台账 mark 漂移、指纹失配（台账记篡改前指纹）、用例文件缺失、md 存在但台账无登记）+ tde 设计源与 README（无 frontmatter，应被忽略）。全部经 `case_runner.main(["ledger", ...])` 端到端。

## 最小实现切片

| Slice | 内容 | 结果 |
|---|---|---|
| S1 | 模块头更新（056-06 状态行 + 退出码 3 档） | 完成 |
| S2 | cmd_ledger [P0]/[P1]（骨架校验保留 + exec_task 门控 + load_ledger 收敛 exit 5） | 完成 + 冒烟 B03/B04/D01 |
| S3 | `_ledger_view_markdown` 四段渲染（矩阵/待办三类/提交摘要） | 完成 + 冒烟 A01~A13/B01~B02 |
| S4 | `--out` splice 复用（首生成/重写/拒绝/原子写/父目录） | 完成 + 冒烟 C01~C07 |
| S5 | 降级双路径（exec_task 缺失 --out exit 5 / 无 PyYAML exit 5） | 完成 + 冒烟 C08/C09/D01 |
| S6 | 只读边界验证（台账与 md 零写入前后比对） | 完成 + 冒烟 E01/E02 |
| S7 | 回归（88 例 + 056-04 冒烟 70 + 056-05 冒烟 55）+ py_compile 双版本 + 真实 cases 只读探针 | 完成 |

## 平台差异处理

N/A：纯本地 Python 只读视图；splice 依赖的 exec_task 为同目录兄弟模块（056-03 起既有可选导入），缺失时仅 --out 降级（exit 5）不影响 stdout 视图；4 副本脚本树同步归 STORY-056-07。现场无 PyYAML 环境：视图收敛 exit 5（冒烟 D01 注入实测）。

## 验证结果

| 验证 | 命令 | 结果 |
|---|---|---|
| py_compile | `uv run --python 3.11 python -m py_compile skills/case-execution/scripts/case_ledger.py skills/case-execution/scripts/case_runner.py` | PASS |
| py_compile | `uv run --python 3.12 python -m py_compile skills/case-execution/scripts/case_ledger.py` | PASS |
| 既有测试回归 | `uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q` | **88 passed**（零回归） |
| 交互冒烟（本 Story） | `uv run --python 3.11 --with pyyaml python /tmp/smoke_cr056_06.py` | **29 pass / 0 fail**（A 全量视图 13 + B --mark 过滤 4 + C --out splice 9 + D 无 PyYAML 1 + E 只读边界 2） |
| 前序冒烟回归 | `/tmp/smoke_cr056_05.py` + `/tmp/smoke_cr056_04.py` | **55 pass / 0 fail** + **70 pass / 0 fail**（cmd_mark/run 准入零回归） |
| 真实环境只读探针 | `main(["ledger", "--cases-root", "/home/hyde/projects/ptm-te-manaul/cases"])` | rc=0 正常产出视图（台账尚未建立 → 空台账骨架视图），**零写入** |

任务指定判据逐项复核：交叉计数/待办三类/漂移清单正确（A03~A11：accept 行 3/1/1/1/6、待办① 3 例含达标与轮数未记录证据列、待办② 3 例、漂移 4 例四类全命中且 tde/README 忽略）PASS；--out 生成后二次运行机器区重写（矩阵随台账变更更新）而人工区自定义内容保留（C04/C05，仿 exec-report 惯例）PASS；无台账目录与损坏收敛 exit 5 无 traceback（D01；"无台账目录"场景 = load_ledger 骨架路径 + cases_root 存在性校验 exit 2，损坏场景 exit 5）PASS；断言 29 ≥ 20 PASS。

## 未覆盖项

| 项 | 归属 | 说明 |
|---|---|---|
| archive-check 实体 | STORY-056-08 | cmd_archive_check 占位 exit 5；本 Story 的漂移清单口径（verify_case_state）可复用其 M 类判定 |
| 正式 pytest 文件 | STORY-056-07 | 见单元测试 N/A 理由；4 副本脚本同步、SKILL.md ST-EX-19 深度收口同归 056-07 |
| 台账 `--json` 输出 | 未排期（HLD §9.3"后续可加"） | 对外消费契约本期只固化 schema；视图渲染函数已单体化，加 JSON 输出为小改动 |
| "accept 且稳定轮达标"独立成段 | 行为约定 | 以 2.1 的"验收证据"列承载（达标/未达标/未记录三态），不另设扫描（避免读 runs 目录，保持纯台账只读） |
| 漂移清单分页/上限 | 行为约定 | 全量输出（工厂侧 312 条量级无性能问题）；如需上限属后续 CR |

## 设计缺口反馈

| # | 发现 | 处置 | 建议 |
|---|---|---|---|
| 1 | 协调者"顺手项"前提与事实不符：resolve_evidence_rounds 的 execute 过滤并非"只在 cmd_mark 调用处实现"，056-05 已实现于函数体内（056-05 冒烟 L01~L05 直接调用函数断言 mode 过滤为证） | 本 Story 零改动该函数，无双口径需要统一；此条留痕供 host-orchestrator 销项 | 无需 07 处理；若 07 回归用例要覆盖该函数，直接按"仅 mode=execute 计入"口径书写即可 |
| 2 | `--out` 与 exec-report 共用同一 marker 串（`case-execution:report-gen`）：LEDGER.md 与 REPORT.md 为不同文件无冲突，但若用户把 REPORT.md 内容粘进 LEDGER.md 人工区会引入 marker 导致下次重写边界漂移 | splice_report 本身对错序/缺失 fail-closed；人工区引入 begin 无 end 会被判缺 marker → 拒绝覆盖（安全侧） | 056-07 文档提示：人工区勿粘贴含生成区 marker 的内容 |
| 3 | 漂移清单"台账无登记"扫描以 frontmatter 用例编号为准，编号重复的 md 文件（同号多文件）只会记一行首见路径 | 存量 312 用例编号唯一（056-04 只读校验佐证）；如需暴露重复编号属 GATE-5/编号规范域（CR-053），不属台账视图 | 维持现状；编号唯一性校验归 056-07 回归或 CR-053 既有卡口 |
| 4 | `_resolve_case_file` 候选序依赖 cases_root 目录名为 `cases`（与 056-05 缺口 #4 同源口径） | 候选②（相对 cases_root）可命中 `te/...` 形态 path，缓解目录名差异；三候选均不命中才报"文件缺失" | 同 056-05 建议：056-07 文档统一提示 --cases-root 应指向 cases 目录 |

## 后续交接

- **给 meta-qa（CP7 验证入口）**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（88 passed 基线）+ 冒烟重放 `/tmp/smoke_cr056_06.py`（29 断言，fixture 自建于 /tmp/cr056-06-smoke）+ `/tmp/smoke_cr056_05.py`（55）+ `/tmp/smoke_cr056_04.py`（70）。验证对象 = 四段视图口径（矩阵/待办三类/漂移四类/提交摘要）、--mark 过滤、--out splice 首生成与二次重写（机器区更新 + 人工区保留）、无 marker 拒绝覆盖 exit 3、双降级 exit 5、只读边界。
- **给 STORY-056-07（测试与同步收口）收口清单**：
  - 正式 pytest：`tests/test_cr056_mark_lifecycle.py`（或拆分文件）应覆盖 03~06 冒烟蓝本——指纹 8 / 迁移 33 / verify_case_state / resolve_evidence_rounds（仅 mode=execute，LCQ 已定案）/ cmd_mark 六步与降级 / cmd_ledger 四段视图与 splice 三态（首生成/重写保留/拒绝覆盖）/ run 准入与冻结校验（056-04 冒烟 70 断言为蓝本）。
  - 4 副本脚本同步：ptm-te/.claude、ptm-te-manaul/.claude、ptm-te-manaul/.agents 的 `skills/case-execution/scripts/`（case_runner.py / case_ledger.py / init_cases_repo.py / exec_task.py）与 4 副本一致性校验。
  - SKILL.md 深度收口：ST-EX-18（mark/cicd_mark 过滤与运行准入）、ST-EX-19（mark/ledger 子命令与台账，含 `--out` 人工区勿贴 marker 提示）、ST-EX-20（归档检查）+ R-F-027/028 + Gotchas G-1/G-6 等（HLD §8）。
  - 文档提示项（来自 05/06 缺口反馈）：`--cases-root` 应指向含 case_ledger.yaml 的 `cases` 目录；`--evidence` 建议绝对路径；`--run-scope` 与 `--role` 正交（G-11）；跨域圈选拒绝口径（§4.4 规则 2）。
- **给 STORY-056-08（archive-check）**：接线点 = main() 委托分支已就绪（`archive-check` → `cmd_archive_check(args_ns)`，占位 exit 5 防门禁 fail-open）；parser 参数面 `--cases-root/--author/--date/--fix/--json` 已按 HLD §5.4 备齐；可复用件：`_extract_case_id`/`_FRONTMATTER_RE`（A/M 类 frontmatter 读写）、`compute_case_fingerprint`（M 类"内容实质差异"判定 = 指纹变化，G-15 打标提交不触发）、git 根发现语义（`_git_commit_mark` 内 rev-parse 模式，056-05）；注意规则 17 门禁语义（FAIL → exit 1 阻断 push，非 git → exit 2）。
- **风险提示（整体）**：① LEDGER.md 的 stdout 与 --out 同源渲染，stdout 供人读、文件供留档，二者的 frontmatter 仅存在于文件；② 视图对"无 frontmatter 文件"静默跳过（tde/README），若未来设计源加入 frontmatter 会被误当用例扫描（当前 312 用例库无此形态）；③ 056-04 冒烟 F2b/F4 为跨 Story 演进断言（/tmp 脚本非仓库资产），QA 重放以当前版本为准。
