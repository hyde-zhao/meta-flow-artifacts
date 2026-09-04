---
story_id: "STORY-056-05"
story_slug: "mark-subcommand"
cr_id: "CR-056"
wave: 2
design_evidence_type: "technical-note"
design_evidence_ref: "process/HLD-CR-056.md v0.9 §7.3（六步内部动作）+ §5.3（git 自动 commit）+ §4.1（迁移矩阵与连带清除）+ §10（失败路径）+ §12 G-5/G-7/G-10"
implementation_status: "code-complete-mark-subcommand"
executed_by: "meta-dev"
executed_at: "2026-09-03T23:20:00Z"
revision: "r1（cmd_mark 骨架 → 实体；case_runner.py 零改动，委托分支复用 056-04 接线）"
---

# STORY-056-05 实现执行证据 — case_ledger.cmd_mark 实体化（迁移校验/证据轮数/md+台账双写/批量部分失败/git 自动 commit）

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| 设计证据（technical-note，design-already-in-hld）已确认 | PASS | `process/HLD-CR-056.md` v0.9 §7.3 六步内部动作 + §5.3 git commit 语义 + §4.1 迁移表；host-orchestrator 派发任务 1~5 |
| 依赖门控满足（depends_on=[STORY-056-04]，same-file-serial） | PASS | STORY-056-04 已验收（CP6 PASS / CP7 通过）；其交接要点确认：main() 委托分支与 parser 参数面已就绪，本 Story 只实体化 `case_ledger.cmd_mark` |
| 文件所有权无冲突 | PASS | primary=`skills/case-execution/scripts/case_ledger.py`（056-03 新建、本 Story 实体化）；forbidden 边界遵守：**case_runner.py 零改动**（git diff --stat 仍为 056-04 的 +678/-7，实测确认） |
| AI 任务清单存在 | PASS | 派发任务：实体化六步 / 圈选 / 摘要表 / 验证（py_compile 双版本、88 例回归、冒烟 ≥30 断言、临时 git 仓库验证 commit）/ 证据 |
| LCQ-STORY-056-03-01 已定案消费 | PASS | 协调者确认"resolve_evidence_rounds 只统计 --execute 真实运行 run"。**事实核对**：056-03 代码实际未过滤 mode（docstring 为灰区留痕原样）——本 Story 在 case_ledger.py（本 Story 写入范围）内落地该定案：仅 `mode == "execute"` 的轮参与连续 PASS 计数，dry-run 不计入且打断连续；mark 侧直接消费 |
| 复用事实核对完成 | PASS | exec_task.scan_exec_runs/build_case_records（appearances 含 mode 字段，2026-09-03 源码核对）；init_cases_repo git 根发现语义（`git rev-parse --show-toplevel`）；056-04 可复用件语义（_split_csv/_resolve_ledger_root/filter 口径）因循环 import 与 op_mapper 解耦原因在 case_ledger 本地对齐实现（见设计缺口反馈 #1） |
| 安全校验 | PASS | 不触网、不触真实设备；git 只 add/commit **永不 push/force**；冒烟全部使用 /tmp 临时 git 仓库与临时夹具，ptm-cases 与 312 条真实用例零触碰 |

## 实现对象清单

| # | 对象 | 操作 | 说明 |
|---|---|---|---|
| 1 | `skills/case-execution/scripts/case_ledger.py` | 修改 | cmd_mark 骨架 → 实体 + 10 个私有辅助 + resolve_evidence_rounds mode 过滤（LCQ 定案）+ 模块头/骨架注释更新 |
| 2 | `process/stories/STORY-056-05-mark-subcommand-IMPLEMENTATION.md` | 新建 | 本证据 |
| 3 | `process/checks/CP6-STORY-056-05-mark-subcommand-CODING-DONE.md` + `.result.json` | 新建 | CP6 门禁 |
| 4 | `process/returns/STORY-056-05-mark-subcommand-CP6.return.json` + `process/evidence/STORY-056-05-mark-subcommand-CP6.index.json` | 新建 | Return Packet / Evidence Index |
| 5 | `DEV-LOG.md` | 追加 | 补记 056-03 条目一行（此前漏记，host-orchestrator 授权）+ 056-05 实现段 |
| 6 | `process/DEVELOPMENT-PLAN-CR-056.yaml` | 更新 | STORY-056-05 status planned → ready-for-verification + execution_note |

禁改边界实测：`git status --porcelain` 中 `agents/ptm-te.md` / `script/ptm_team/install.py` / `skills/case-execution/SKILL.md` / `DEV-LOG.md`（本 Story 仅追加）的 modified 为前序 Story 未提交改动；`case_runner.py` / `exec_task.py` / `init_cases_repo.py` 本 Story 零改动；`git diff --stat case_runner.py` 维持 +678/-7（056-04 产物）。

## 设计契约映射

### §7.3 六步内部动作落点（cmd_mark 主体）

| HLD 步骤 | 实现 |
|---|---|
| ① 迁移合法性 | 逐用例：`_fm_mark_fields` 读 md 当前双值（缺省 debug/None）+ 台账条目当前值——**台账为准**，md/台账漂移先 stderr WARNING；台账无登记 → WARNING 并按 md 状态判定 + 本次建档。`validate_transition(cur_mark, cur_cicd, --set)` 非法 → 该用例失败项（原因串含 HLD 示例前缀）；**同值自环预判跳过**（056-03 设计缺口 #5 定案：提示"无状态变化"而非报错，记"跳过"行，rc 0） |
| ② note 规则 | `--set debug`（回退）/ `--set deprecated`（废弃）缺 `--note` → 拒绝该项（失败原因"必须提供 --note"）；`debug→verify` 缺 note → 仅 stderr WARNING（不阻断） |
| ③ 验收证据 | `EVIDENCE_REQUIRED[(cur, target)]` 命中即验收类：缺 `--evidence` → 拒绝；`--evidence` 经 `_evidence_runs_and_id` 拆分 `<runs-dir>/<exec-id>`（无目录部分按 case_runner `--runs-dir` 缺省 `runs`）；轮数门槛边（verify→accept / cicd_verify→cicd_accept）走 `resolve_evidence_rounds` ≥ MARK_STABLE_ROUNDS=3；无轮数门槛边（None→cicd_verify）走 `_evidence_container_has_execute_case`（容器可用 + 该 case 有 mode=execute 轮，审计追溯成立即可）；不达标拒绝该项并附原因描述 |
| ④ 写面 | `compute_case_fingerprint`（打标不改指纹；对写入前文本计算一次）→ `_frontmatter_set_line`：块内 `^mark:`/`^cicd_mark:` 行原位替换、无该行在 **frontmatter 块尾部插入**、连带清除（→debug / →deprecated）删 `cicd_mark` 行；块定位与指纹用同一 `_FRONTMATTER_RE`（写入面与指纹剔除面一致，G-4）；无 frontmatter → 该项失败不动文本。台账侧（全程 `ledger_lock`）：`append_mark_history`（**先**，记录迁移后双域值/by/role/note/evidence/rounds，首次 debug→verify 写 first_submitted_at）→ 更新 entry mark/cicd_mark/content_sha256（**后**）→ `save_ledger`（tmp+os.replace 原子写）；新建条目派生 `path` = `<cases_root 目录名>/<相对路径>`（DQ-056-04 cases/te 前缀口径，既有条目 path 原样保留） |
| ⑤ git 自动 commit | `_git_commit_mark`：`git -C cases_root rev-parse --show-toplevel` 向上发现仓库根（init_cases_repo 同语义）→ `git add -- <md> <case_ledger.yaml>` → `git commit -m "mark(case): <case_id> <旧>→<新>"`（双域串如 `accept/cicd_accept→debug`）；`--no-commit` 关闭；非 git / add 失败 / commit 失败 → 降级跳过 + stderr WARNING（不回滚已写面）；**永不 push / force / 不触碰暂存区之外文件**；commit 在台账锁外执行 |
| ⑥ 无 PyYAML / 台账读写失败 | `load_ledger` RuntimeError/ValueError → stderr 明确错误 + **exit 5**（不写任何 md/台账面，不暴露 traceback）；`save_ledger` 失败 → 该用例失败项 + "md 已写，请人工核对或以 mark --set 回退"提示（HLD §10 不自动改 md） |

### §7.3 契约面其余条目

| 契约 | 实现 |
|---|---|
| 圈选 | `--case-file` 单文件直取（--mark/--cicd-mark WARNING 忽略，§7.3）；`--cases-dir` 递归收集 .md（`_iter_case_files`，确定性排序）后按 `--mark`（in 并集）/`--cicd-mark`（none→is None、并集）圈选（语义对齐 case_runner.filter_by_mark/filter_by_cicd_mark，见设计缺口反馈 #1）；圈选空 → exit 2"圈选后无待打标用例"；圈选参数值域非法 → exit 2（与 run 侧同口径防拼写空选） |
| `--cases-root` | 显式指定优先；缺省自 `--case-file` 所在目录 / `--cases-dir` 向上 ≤2 级精确匹配 `case_ledger.yaml`（`_resolve_ledger_root`，与 case_runner C-6 逐语义对齐）；不可定位 → exit 2 提示显式指定；root 存在但台账文件不存在 → WARNING"将以新台账创建条目"后继续（bootstrap 场景） |
| 入口校验 | 保留骨架既有：--set 值域路由、--role/--scene 值域、feature·acceptance 拦截（exit 2）；新增：--case-file 与 --cases-dir 互斥且必给其一、目标文件/目录存在性（exit 2） |
| 批量部分失败 | 逐用例独立判定：单项失败/跳过不影响其余；摘要表逐条输出；存在失败项 → exit 1，全成功（或 --no-commit 下写面全成功）→ exit 0 |
| 摘要表 | stdout：`== mark 摘要：--set <值>（<域> 域）成功 N / 跳过 M / 失败 K ==` + 逐行 `成功/跳过/失败  case_id  旧→新双域描述（含证据轮数描述）`；WARNING 类（漂移/无登记/note 建议/git 降级）走 stderr |

### resolve_evidence_rounds（LCQ-STORY-056-03-01 定案落地）

| 项 | 实现 |
|---|---|
| mode 过滤 | appearances 先滤 `mode == "execute"` 再计最近连续 PASS；纯 dry-run 容器 → (False, "无 mode=execute 的真实运行轮（LCQ 定案：dry-run 不计入验收证据）")；FAIL 打断连续计数（execute 轮内） |
| 描述口径 | "最近连续 execute PASS N 轮（execute 轮共 M，门槛 MARK_STABLE_ROUNDS=3，DQ-056-02）" |
| 灰区状态 | LCQ-STORY-056-03-01 关闭（RESOLVED-by-user 定案 + 本 Story 实现），docstring 灰区留痕段移除 |

## 单元测试与 Fixture 计划

- 本 Story 验证 = 交互冒烟（`/tmp/smoke_cr056_05.py`，**不落仓库**，55 断言）+ 既有 88 例回归 + 056-04 冒烟回归（70 断言，其中 F2/F2b 过渡断言随 cmd_mark 实体化更新，见验证结果表）；**正式 pytest 文件 N/A（后置 STORY-056-07）**，N/A 理由同 056-03/04（CR-056 Story 拆解明确测试收口归 056-07）；冒烟 55 断言即 mark 子命令部分用例蓝本。
- Fixture：`/tmp/cr056-05-smoke/` = `repo/`（`git init` 临时仓库 + local 身份 + init commit）内 `cases/case_ledger.yaml`（13 用例条目，真实指纹经 compute_case_fingerprint）+ `cases/te/demo/*.md`（16 键 frontmatter，含无 mark 行/含 cicd_mark/台账无登记等形态）+ `runs/exec-*`（伪造 result.json 容器：3 轮 execute PASS / 3 轮 dry-run PASS / PASS-FAIL-PASS execute / 1 轮 execute / 3 轮 execute for cicd）+ `nogit/`（非 git 降级）+ `lonely/`（无台账根）。全部经 `case_runner.main(["mark", ...])` 端到端（覆盖 056-04 委托分支）。

## 最小实现切片

| Slice | 内容 | 结果 |
|---|---|---|
| S1 | 模块头/骨架注释更新 + subprocess 导入 | 完成 |
| S2 | resolve_evidence_rounds mode 过滤（LCQ 定案） | 完成 + 冒烟 L01~L05 |
| S3 | 辅助函数 10 个（_split_csv/_resolve_ledger_root/_iter_case_files/_extract_case_id/_entry_path/_frontmatter_set_line/_evidence_runs_and_id/_evidence_container_has_execute_case/_git_commit_mark/_mark_transition_desc） | 完成 + 冒烟间接覆盖 |
| S4 | cmd_mark [P0] 参数校验（骨架校验 + 互斥/存在性/圈选值域） | 完成 + 冒烟 P01~P06/J01b |
| S5 | cmd_mark [P1]/[P2] cases_root 定位 + 台账加载 + 圈选 | 完成 + 冒烟 K02/J03/I04 |
| S6 | cmd_mark [P3] 六步逐用例判定与写面 | 完成 + 冒烟 A~J 全链 |
| S7 | cmd_mark [P4] 摘要表 + 退出码 | 完成 + 冒烟 A01/I03 |
| S8 | 回归（88 例 + 056-04 冒烟 70 断言）+ py_compile 双版本 | 完成 |

## 平台差异处理

N/A：纯本地 Python 模块改动；git 依赖为 CLI subprocess（POSIX/WSL2 实测），git 缺失或非 git 仓库均降级跳过不阻塞打标；fcntl/yaml 差异沿用 056-03 既有处理；4 副本脚本树同步归 STORY-056-07。现场无 PyYAML 环境：cmd_mark 显式收敛 exit 5（冒烟 K01 注入 `case_ledger.yaml=None` 实测，无 traceback）。

## 验证结果

| 验证 | 命令 | 结果 |
|---|---|---|
| py_compile | `uv run --python 3.11 python -m py_compile skills/case-execution/scripts/case_ledger.py skills/case-execution/scripts/case_runner.py` | PASS |
| py_compile | `uv run --python 3.12 python -m py_compile skills/case-execution/scripts/case_ledger.py` | PASS |
| 既有测试回归 | `uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q` | **88 passed**（零回归；cmd_mark 实体化不影响 run/validate/exec-report 路径） |
| 交互冒烟（本 Story） | `uv run --python 3.11 --with pyyaml python /tmp/smoke_cr056_05.py` | **55 pass / 0 fail**（P0 参数校验 7 + 正例链 A/B/D/E/F/G 17 + 反例 C 4 + 自环/漂移 H 3 + 批量圈选 I 4 + no-commit/非 git/建档 J 4 + 降级 K 2 + resolve 单元级 L 5；断言数 > 任务要求 30） |
| 056-04 冒烟回归 | `uv run --python 3.11 --with pyyaml python /tmp/smoke_cr056_04.py` | **70 pass / 0 fail**（F2 过渡断言更新：cmd_mark 实体化后 `mark --set accept` 无目标文件由"骨架返回 5"变为"缺目标 exit 2"；新增 F2b 实体化端到端探针） |
| git commit 产物 | 临时仓库 `git log --format=%s` / `git show --name-only` | `mark(case): PC-T01-01 debug→verify` 等 commit 存在且**逐用例独立**；commit 恰含 `<md>` + `cases/case_ledger.yaml` 两文件；`--no-commit` 零新 commit；非 git 目录 WARNING 降级跳过；全程无 push/force |

任务指定判据逐项复核：正例链 debug→verify→accept（evidence mock 3 轮 execute PASS）→cicd_verify（容器追溯）→cicd_accept（3 轮）全过（A/B/D）；verify_fail 全链（verify_fail→verify 重提 G01、verify_fail→deprecated F02）全过；accept→debug 连带清 cicd_mark（md 行删除 + 台账 None + commit 双域串 E02~E05）全过；反例（无 note 打回 E01/F01、证据缺失 C01、dry-run 容器 C02、轮数不足 C04、非法迁移 G02/I03、非 accept 设 cicd_mark D03、deprecated 迁出 F03）全过；md+台账双写与 git commit 产物在 /tmp 临时 git 仓库验证（A07~A09/E05），**未触碰真实 ptm-cases**。

## 未覆盖项

| 项 | 归属 | 说明 |
|---|---|---|
| ledger 状态统计视图 | STORY-056-06 | cmd_ledger 仍为骨架（返回 5）；本 Story 的 mark_history/entry 结构即其消费面 |
| archive-check 实体 | STORY-056-08 | cmd_archive_check 占位 exit 5 |
| 正式 pytest 文件 | STORY-056-07 | 见单元测试 N/A 理由；4 副本脚本同步、SKILL.md ST-EX-19 深度收口同归 056-07 |
| `--evidence` 相对路径按 CWD 解析 | 行为约定 | `--evidence runs/exec-x` 相对当前工作目录解析（与 case_runner `--runs-dir` 缺省 `runs` 一致）；绝对路径等价支持；冒烟用绝对路径 |
| `--rounds` 阈值覆盖 | 行为约定 | `--rounds` 为台账 mark_history 审计声明值（缺省：验收类边记 MARK_STABLE_ROUNDS、其余记 None），不降低 resolve_evidence_rounds 的硬门槛（DQ-056-02 N=3 固定）；如需可配置门槛属后续 CR |
| git identity 缺失引导 | 降级路径 | commit 失败降级 WARNING（init_cases_repo 步骤 6 已有配置引导）；mark 侧不代配身份（用户属性） |

## 设计缺口反馈

| # | 发现 | 处置 | 建议 |
|---|---|---|---|
| 1 | 协调者指令"复用 case_runner 的 filter_by_mark/filter_by_cicd_mark + _split_csv"存在架构冲突：case_ledger 被 case_runner 顶层 import，反向 import 形成循环；函数级懒加载则使 mark 功能耦合 op_mapper 可用性 | 在 case_ledger 本地实现对齐版本（`_split_csv`/圈选语义与 case_runner 逐语义一致，`_resolve_ledger_root` 同口径），并以注释标明对齐来源；冒烟双向断言（056-05 冒烟圈选 + 056-04 冒烟 run 圈选）保证语义不分叉 | host-orchestrator 认可该解耦取舍；若坚持单一实现，需 CR 把圈选纯函数下沉到 case_ledger 并让 case_runner 反向消费（机械改动，影响 056-04 已验收面） |
| 2 | 协调者指令"resolve_evidence_rounds case_ledger 已按 LCQ 定案实现"与实际代码不符（056-03 落地为灰区留痕原样、未过滤 mode） | 本 Story 在自身写入范围内落地定案：仅 mode=execute 轮计入连续 PASS 且打断连续；`_evidence_container_has_execute_case`（无轮数门槛边）同口径 | 无需回改；LCQ-STORY-056-03-01 状态建议由 host-orchestrator 记为 RESOLVED（定案 2026-09-03 + 实现 STORY-056-05） |
| 3 | 摘要表失败原因在 stdout（需求 3 单一真相源），stderr 仅承载 WARNING；纯 stderr 消费方需依赖退出码感知失败 | 维持 stdout 摘要表 + rc 0/1 分级 | 若 CI 需要失败行进 stderr，可在 056-07 收口时加 `--fail-stderr` 类开关（不建议，避免双写） |
| 4 | 台账 `path` 派生依赖 cases_root 目录名恰为 `cases`（symlink 方案天然成立）；若用户以非 `cases` 名目录作 --cases-root，新建条目 path 前缀跟随目录名，与 DQ-056-04 `cases/te` 口径偏离 | 实现按"目录名 + 相对路径"派生并在 docstring 声明；既有条目不回改 | 文档（056-07）提示 --cases-root 应指向含 case_ledger.yaml 的 `cases` 目录；如需强校验前缀（非 `cases` 即拒绝）属后续 CR |
| 5 | 056-04 冒烟 F2 为过渡断言（"骨架返回 5"），随本 Story 实体化失效 | 冒烟脚本（/tmp，不落仓库）更新为"缺目标文件 exit 2"并新增 F2b 实体化端到端探针；056-04 冒烟 70/0 复跑全绿 | 无需动作（/tmp 脚本非仓库资产）；正式回归资产由 056-07 pytest 承接 |

## 后续交接

- **给 meta-qa（CP7 验证入口）**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（88 passed 基线）+ 冒烟重放 `/tmp/smoke_cr056_05.py`（55 断言，fixture 自建于 /tmp/cr056-05-smoke，含临时 git 仓库）+ `/tmp/smoke_cr056_04.py`（70 断言回归）。验证对象 = 六步内部动作逐条（迁移矩阵正反例、note 规则、EVIDENCE_REQUIRED 三边、双写顺序、连带清除、git commit 产物与降级）、批量部分失败 rc 语义、圈选口径、无 PyYAML 收敛 exit 5。
- **给 STORY-056-06（ledger 子命令实体化）**：
  - **接线点**：case_runner main() `mark/ledger/archive-check` 委托分支已就绪，`ledger` 已委托 `case_ledger.cmd_ledger(args_ns)`——06 只实体化 cmd_ledger，case_runner 侧零改动。
  - **cmd_ledger 现有签名与消费面**：`cmd_ledger(args) -> int`；args 属性 `cases_root`（required）/`mark`（**list**，parser 已按逗号拆分；骨架已校验值域）/`out`。可复用件：`load_ledger`（骨架校验 + 结构校验）、`verify_case_state`（漂移清单判定 = fingerprint_ok False 的语义源）、`_fm_mark_fields`/`_iter_case_files`（全库扫描 mark/cicd_mark 现值）、`_extract_case_id`（漂移清单定位 case_id）、`compute_case_fingerprint`（指纹漂移比对）；条目结构 = `{path, mark, cicd_mark, content_sha256, mark_history[{at,mark,cicd_mark,by,role,note,evidence,rounds}], first_submitted_at?}`。
  - **建议视图口径**：交叉计数 = mark 5 态 × cicd_mark 4 档（无/cicd_verify/cicd_verify_fail/cicd_accept）+ deprecated 计数；待办清单三类（accept 未提交 CICD / verify_fail+cicd_verify_fail 待整改 / 指纹漂移）；漂移清单 = 逐用例 `_fm_mark_fields` vs 台账 + `compute_case_fingerprint` vs content_sha256；`--out` Markdown 沿用 exec-report splice 惯例（机器生成区 marker 重写 + 人工区原样保留）。
  - **风险提示**：无 PyYAML 环境 load_ledger 收敛 exit 5（cmd_mark 同款 catch 模板可直接复用）；台账含人工区键（submitter/notes/open_issues），`--out` 重写时不得触碰。
- **风险提示（整体）**：① 冒烟曾三次暴露冒烟脚本自身缺陷（摘要断言空格、YAML 误用 json 解析、stdout 变量错绑），与产品代码无关，修正后 55/0 与 70/0 双绿——QA 重放请以当前 /tmp 脚本版本为准；② git commit 依赖操作者身份配置，缺失时降级 WARNING（打标仍成功），审计完整性依赖台账 mark_history（by 字段）；③ `_frontmatter_set_line` 与指纹共用 `_FRONTMATTER_RE`，`--- `（尾随空格）等非规范首行的用例会判"缺 frontmatter"失败而非错写——与 056-04 run 侧 parse_frontmatter 的宽松正则存在理论差异（存量 312 用例实测无此形态，见 056-04 只读校验）。
