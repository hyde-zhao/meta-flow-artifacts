---
story_id: "STORY-056-08"
story_slug: "archive-check"
cr_id: "CR-056"
wave: 3
design_evidence_type: "technical-note"
design_evidence_ref: "process/HLD-CR-056.md v0.9 §5.4（归档推送前检查六步）+ §4.1a（作者元数据字段）+ §12 G-15（打标提交不触发）+ 规则 17"
implementation_status: "code-complete-archive-check"
executed_by: "meta-dev"
executed_at: "2026-09-04T03:30:00Z"
revision: "r1（cmd_archive_check 占位 → 实体 + 薄壳 CLI archive_check.py + 4 副本分发）"
---

# STORY-056-08 实现执行证据 — archive_check 归档推送前检查实体化（规则 17 门禁）

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| 设计证据（technical-note，design-already-in-hld）已确认 | PASS | host-orchestrator 派发任务 1~4（实体化/位置选择/验证/证据）；HLD §5.4 六步语义 + §4.1a 四字段 + G-15 |
| 依赖门控满足（depends_on=[STORY-056-07]，all-waves 收尾） | PASS | 056-07 交接清单确认：main() 委托 + parser 参数面（--cases-root/--author/--date/--fix/--json）已就绪 |
| 文件所有权无冲突 | PASS | 改动 = `case_ledger.py`（cmd_archive_check 实体化）+ 新建薄壳 `scripts/archive_check.py` + 分发副本；case_runner.py canonical 零改动（git diff 维持 056-04 的 +678/-7 实测） |
| 实现位置决策（派发授权自行判断） | PASS | **case_ledger 内实体 + 薄壳 CLI 双入口**：实体承接 case_runner `archive-check` 委托；薄壳 `archive_check.py` 仅 import 委托（argparse 参数面一致），满足 HLD §5.4 CLI 契约 `python archive_check.py --cases-root ...`（规则 17 工厂/CICD 侧可直接调用），检查逻辑零重复实现 |
| 安全校验 | PASS | 只读检查（无 --fix 零写入，冒烟 A13 抽样）；--fix 仅在 /tmp 夹具上执行，**严禁对真实用例库 --fix**（真实探针为仅检查模式 rc=0 零写入）；永不 push |

## 实现对象清单

| # | 对象 | 操作 | 说明 |
|---|---|---|---|
| 1 | `skills/case-execution/scripts/case_ledger.py` | 修改 | cmd_archive_check 占位 → 实体 + 辅助 5 个（`_git_run`/`_FM_META_VALUE_RES`/`_fm_meta_fields`/`_archive_changeset`/`_archive_check_one`/`_archive_fix_one`）+ 模块头状态行 |
| 2 | `skills/case-execution/scripts/archive_check.py` | 新建 | 薄壳独立 CLI（argparse 参数面 = HLD §5.4，import case_ledger 委托） |
| 3 | 三分发目录 `scripts/{archive_check,case_ledger}.py` | 同步 | ptm-te/.claude、ptm-te-manaul/.claude、ptm-te-manaul/.agents；5 脚本 × 3 分发 = 15/15 diff 一致 |
| 4 | `skills/case-execution/SKILL.md` | 微调 | ST-EX-20 占位文案 → 正式（双入口/检查语义/退出码/push 边界） |
| 5 | 过程证据（IMPLEMENTATION / CP6 / return / evidence / DEV-LOG / plan 状态） | 新建/更新 | 本 Story 记录 |

禁改边界实测：case_runner.py canonical 零改动；真实 ptm-cases（/home/hyde/projects/ptm-cases）只读探针 rc=0 零写入；--fix 冒烟仅在 /tmp/cr056-08-smoke 临时 git 仓库执行。

## 设计契约映射

### §5.4 六步落点（cmd_archive_check 主体）

| HLD 步骤 | 实现 |
|---|---|
| ① git 根发现 | `_archive_changeset`：`git rev-parse --show-toplevel` 自 --cases-root 向上发现（symlink 场景自动成立）→ 失败 NOT_GIT_REPO **exit 2**（--json 时同样输出 NOT_GIT_REPO 结构） |
| ② 变更集 | `git status --porcelain`（`-c core.quotepath=false` 保证中文路径原样）+ `git diff HEAD --name-only`，过滤 `cases/` 前缀；`??` → A（新增）、含 "A" → A、含 "D" → D（仅提示）、其余内容变化 → M；**D 不得经 diff HEAD 泄入 M**（工作区无文件可读会误报 READ_ERROR，见设计缺口反馈 #1） |
| ③ A 类校验 | 四字段齐全（MISSING_CREATOR/MISSING_CREATED/MISSING_MODIFIER/MISSING_MODIFIED）+ 日期 `DATE_RE`（INVALID_DATE）+ 创建时间/修改时间 == 操作日期（STALE_MODIFIED） |
| ④ M 类校验 | HEAD 版（`git show HEAD:<path>`）与工作区版 **compute_case_fingerprint 对比**：一致（含仅 mark/cicd_mark 行变化的打标提交，G-15）→ 跳过修改人/时间校验；实质差异 → 修改人非空 + 修改时间 == 操作日期（MISSING_MODIFIER/MISSING_MODIFIED/STALE_MODIFIED/INVALID_DATE） |
| ⑤ --fix | `--fix --author`（author 必填，缺失 exit 2）：A 类四字段统一写 author/date（新建即当日，幂等）；M 类实质差异仅刷新修改人/修改时间（创建字段保持不变）；写后**重校验**，仍失败即 FAIL（FIX_FAILED/保持原问题）；frontmatter 块缺失无法自动修复（显式提示人工）；写失败 → 该文件 FAIL |
| ⑥ 输出 | 逐文件 [PASS]/[FAIL]（问题行 code: message）/[FIXED] + D 类仅提示行 + 汇总（N 通过 / N 失败 / fix 修复 N）+ 阻断/放行话术；`--json` 输出 {cases_root, git_root, date, author, fix, result, summary{total,pass,fail,fixed,deleted_hint}, files[{path,kind,status,fixed,problems[code,message]}]}；**有 FAIL 且无全部通过 → exit 1 阻断**；NOT_GIT_REPO/参数错误 → 2 |

### 其余契约

| 项 | 实现 |
|---|---|
| date 缺省 | 今天（`time.strftime("%Y-%m-%d")`）；显式传入需匹配 DATE_RE，非法 exit 2 |
| --fix --author | author 必填（exit 2）；author 记入输出供审计 |
| 检查日期口径 | 判等以 YYYY-MM-DD 字符串全等（G-14 年月日精度，无秒级时间戳） |
| 无 PyYAML | archive-check **零 PyYAML 依赖**（frontmatter 用 `_FRONTMATTER_RE` + 行正则手写解析，指纹为纯 hashlib）——比 mark/ledger 更强的现场环境容忍 |
| 薄壳 CLI | `archive_check.py main(argv)` argparse 同参数面 → `case_ledger.cmd_archive_check(args)`；`sys.path` 自目录注入；docstring 声明"禁止在本文件重复实现检查逻辑" |
| 退出码 | 0 通过；1 存在 FAIL（阻断推送）；2 参数错误/NOT_GIT_REPO；5 依赖不可用（模块头已同步） |

## 单元测试与 Fixture 计划

- 交互冒烟 `/tmp/smoke_cr056_08.py`（**不落仓库**）：**30 pass / 0 fail**（P0 参数与前置 4 + A 变更集校验 11 + B --json 4 + C --fix 5 + D 薄壳等价 2 + E 只读边界 1）。
- Fixture：`/tmp/cr056-08-smoke/repo`（git init + local 身份 + HEAD 基线 commit：M1~M4/D1 用例 md + cases/case_ledger.yaml + README + docs/other.md）→ 工作区变更：A1~A4 新增（齐全/缺修改人时间/日期格式非法/创建时间过期）、M1 实质修改未刷新、M2 实质修改已刷新、M3 仅 mark 行变化（打标提交）、M4 无元数据实质修改、D1 删除、docs/other.md 非 cases 前缀修改（应过滤）。
- 正式 pytest：**N/A（增量归 056-08 自身冒烟 + 既有 206 基线）**——理由：archive-check 为 CR-056 最后实体化 Story，本 Story 冒烟 30 断言即 `test_cr056_archive_check.py` 用例蓝本，可随后续收口 CR 与 07 测试文件合并；本 Story 以 206 基线零回归 + 冒烟 30 断言为完成门槛（与派发验证要求一致）。

## 最小实现切片

| Slice | 内容 | 结果 |
|---|---|---|
| S1 | cmd_archive_check 实体（①② git 发现 + 变更集） | 完成 + 冒烟 P01/P04/A10/A11 |
| S2 | ③④ A/M 类校验（含 G-15 指纹跳过） | 完成 + 冒烟 A02~A09 |
| S3 | ⑤ --fix（补齐/刷新/重校验/不可修复显式 FAIL） | 完成 + 冒烟 C01~C05 |
| S4 | ⑥ 输出（文本清单 + --json）与退出码 0/1/2 | 完成 + 冒烟 A01/A12/A14/B01~B04 |
| S5 | 薄壳 archive_check.py + 分发同步 | 完成 + 冒烟 D01/D02 + 15/15 diff |
| S6 | SKILL.md ST-EX-20 转正式 + 回归 | 完成 |

## 平台差异处理

分发覆盖 claude/codex 两类目录形态（薄壳 import 同目录 case_ledger，sys.path 自注入与 case_runner §9.1 同惯例）；git 依赖 CLI subprocess（`-c core.quotepath=false` 处理中文路径引号转义——**分发环境与 canonical 同为 Linux/WSL2 实测**）；现场无 PyYAML 不影响本命令（零 yaml 依赖）。

## 验证结果

| 验证 | 命令 | 结果 |
|---|---|---|
| 全量回归 | `uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q` | **206 passed**（零回归；含 07 落盘的 archive-check 委托断言，已随实体化更新为终态口径） |
| py_compile | 3.11 / 3.12 双版本 case_ledger.py + archive_check.py | PASS |
| 交互冒烟 | `uv run --python 3.11 --with pyyaml python /tmp/smoke_cr056_08.py` | **30 pass / 0 fail**（> 任务要求 30 达标） |
| 4 副本分发 | 5 脚本 × 3 分发 `diff -q` | **15/15 一致** |
| 真实环境只读探针 | `main(["archive-check", "--cases-root", "/home/hyde/projects/ptm-te-manaul/cases", "--date", "2026-09-04"])` | rc=0 PASS（git_root 正确解析 /home/hyde/projects/ptm-cases——symlink 场景向上发现成立；工作树干净 → 空变更集）；**零写入、未 --fix** |

任务指定判据逐项复核：四字段齐全/MISSING_* 各错误类/STALE_MODIFIED/INVALID_DATE/仅打标提交跳过（G-15）/--fix 补齐后重跑 PASS/FAIL exit 1/非 git exit 2/--json 输出——冒烟 A01~A14/B01~B04/C01~C05/D01~D02 全覆盖（30 断言 ≥ 30）；A/M/D 变更集 fixture 为临时 git init 仓库 + 真 git add/commit 构造 HEAD 基线 ✓。

## 未覆盖项

| 项 | 归属 | 说明 |
|---|---|---|
| archive_check 正式 pytest 合并 | 后续收口 CR 候选 | 本 Story 冒烟 30 断言为蓝本（见单元测试 N/A 理由） |
| rename（R 类）变更集语义 | 行为约定 | porcelain R 按新路径计入 M 类（git status 默认不展开 R 除非 -M；现实现按内容变化处理）；如需显式 rename 识别属后续 CR |
| 存量 312 用例作者元数据回填 | O-056-06（独立后续 CR 候选） | 本检查只对 A/M 变更集强制，存量未动文件不强制（HLD §4.1a） |

## 设计缺口反馈

| # | 发现 | 处置 | 建议 |
|---|---|---|---|
| 1 | `git diff HEAD --name-only` 会把已删除文件列入（工作区无文件），初版实现泄入 modified → 误报 READ_ERROR FAIL | 修复：diff HEAD 循环排除 `deleted` 集合（porcelain "D" 已捕获，删除仅提示）；冒烟 A10/A12/B01 复验 | 已修复并留痕；该行为由冒烟 A10 断言锁定 |
| 2 | --fix 修复会改变元数据行 → 元数据行参与指纹（G-4 变更证据语义）→ 修复后的文件对 HEAD 仍是"实质差异"，但修改人/时间已刷新 → 重校验 PASS（语义自洽） | 按 HLD 实现并在冒烟 C04 断言"创建字段不变 + 修改字段刷新" | 无需动作 |
| 3 | 薄壳与 case_runner 双入口共享实体，未来参数面变更需两处同步（argparse 定义重复） | 薄壳 docstring 声明参数面镜像关系；当前参数面稳定（HLD §5.4 定稿） | 若后续加参数，优先改 case_runner 并同步薄壳（或后续 CR 把 argparse 定义下沉 case_ledger） |
| 4 | A 类"创建时间/修改时间 != 操作日期"记 STALE_MODIFIED（HLD 未给独立类别，7 类错误码约束下最贴切） | 实现按 STALE_MODIFIED 并在 message 写明字段 | 若 host-orchestrator 认为需独立错误码（如 STALE_CREATED），一行可改 |

## 后续交接

- **给 meta-qa（CP7 验证入口，CR-056 全量）**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（206 passed 基线）+ 冒烟重放 `/tmp/smoke_cr056_08.py`（30 断言）与 03~07 冒烟（103/69/55/29/70）；5 脚本 × 4 副本（canonical + 3 分发）diff；.codex TOML 三方一致性；SKILL.md/执行指导结构。HLD §13 矩阵：G-09 全链（A/M 类元数据校验 + --fix + INVALID_DATE）由本 Story 冒烟覆盖收口。
- **给 host-orchestrator（CR-056 收口建议）**：① 八 Story 全部 ready-for-verification，建议整体进入 CP7（validation_mode 判定：code-project 为主，静态 + 冒烟重放；真机三场景端到端属 runtime_authorization 独立域）；② OQ-056-02-01（07 收口）、LCQ-STORY-056-03-01（05 落地）均可记 RESOLVED；③ 候选台账：GAP-07-01（install.py tools 数组格式）、O-056-06（存量元数据回填）、`ledger --json`、archive-check rename 语义（见本文件 §未覆盖项/缺口反馈）。
- **风险提示**：① archive-check 的 --fix 会写用例文件（元数据行参与指纹），对**已冻结（verify/accept）用例**运行 --fix 属内容修改，将触发冻结校验失配（CASE_MARK_STALE）——操作顺序应为"先 mark 打回 debug → 修改 → 归档检查"，已写入 SKILL.md ST-EX-20 语义链；② porcelain 依赖 `-c core.quotepath=false`（中文路径），极老 git 版本若不支持该配置项会 rc!=0 → NOT_GIT_REPO 误报（现代 git ≥2.10 均支持，现场 git 实测通过）；③ 冒烟 A10/B02 初版为断言笔误（非产品问题），已修正后 30/0。

## 附：CR-056 八 Story 最终交接摘要（供 CP7）

| Story | 交付物 | 关键验证 |
|---|---|---|
| 056-01 | init_cases_repo.py（git 关联/连通探测/幂等/降级） | 冒烟 103 内含 + pytest 88 基线 |
| 056-02 | 规则块 v1.1.0（规则 4/14~17）+ agent md 三副本 + cases/te 路径 | 渲染断言 + 10 pytest |
| 056-03 | case_ledger.py 模块（常量/指纹/迁移矩阵/台账读写/verify_case_state/evidence rounds/入口骨架） | 冒烟 103 + 契约表 |
| 056-04 | case_runner run 侧接线（[3c]/[3d]/[3e]、filter、--role 准入、CASE_MARK_STALE、case_state、mark/ledger/archive-check 委托） | 冒烟 69 + 回归 88 |
| 056-05 | cmd_mark 实体（六步/证据轮数 LCQ 定案/双写/git commit） | 冒烟 55 + 回归 88 |
| 056-06 | cmd_ledger 实体（四段视图/--out splice） | 冒烟 29 + 回归 88 |
| 056-07 | test_cr056_mark_lifecycle.py 118 用例 + 4 副本同步 + TOML 收口 + SKILL.md v2.2 + 执行指导 §9 | 206 passed + 15/15→12/12 diff + 三方一致 |
| 056-08 | cmd_archive_check 实体 + 薄壳 CLI + 分发（5 脚本 × 3 分发 = 15/15 一致）+ ST-EX-20 正式 | 冒烟 30 + 回归 206 + 真实只读探针 |
| 终态 | canonical 5 脚本（case_runner/case_ledger/init_cases_repo/exec_task/archive_check）+ 3 分发目录逐字节一致；agent 三方正文一致；规则块 v1.1.0 三分发 | 206 pytest 全绿 = CR-056 完成门槛 |
