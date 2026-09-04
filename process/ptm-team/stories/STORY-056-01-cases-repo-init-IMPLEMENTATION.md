---
story_id: "STORY-056-01"
story_slug: "cases-repo-init"
cr_id: "CR-056"
wave: 1
design_evidence_type: "technical-note"
design_evidence_ref: "process/HLD-CR-056.md §5.2/§10（v0.8）+ process/changes/CR-056.md IMP-056-04"
implementation_status: "code-complete-real-link-resolved-repo-subdir"
executed_by: "meta-dev"
executed_at: "2026-09-03T12:20:00Z"
revised_at: "2026-09-03T13:10:00Z"
revision: "r2（NEEDS_REWORK 增量收口：向上发现 git 仓库 / mode=repo-subdir / 真实对齐已由用户 symlink 方案解决）"
---

# STORY-056-01 实现执行证据 — init_cases_repo.py 用例库 git 关联脚本 + 执行初始化

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| 设计证据（technical-note，design-already-in-hld）已确认 | PASS | `process/HLD-CR-056.md` v0.8 §5.2/§10 + `process/changes/CR-056.md` IMP-056-04；host-orchestrator 以其派发实现 |
| 依赖门控满足（depends_on=[]，Wave 1 与 STORY-056-02 并行且文件所有权零交集） | PASS | `process/DEVELOPMENT-PLAN-CR-056.yaml` waves[0] |
| 文件所有权无冲突 | PASS | primary=`skills/case-execution/scripts/init_cases_repo.py`；forbidden `ptm-te-manaul/cases/tde/**` 零触碰（314 个内容文件校验和前后一致） |
| AI 任务清单存在 | PASS | host-orchestrator 派发任务 1~6（真相源脚本 / 分发副本 / 实际执行 / 幂等验证 / 冒烟与 pytest / py_compile） |
| 安全校验（不 push / 不 force / 不改已存在 remote / 不触凭据 / 无 traceback） | PASS | 脚本安全边界（docstring）+ 测试 `_assert_no_push_like_commands` 全场景断言 + `GIT_TERMINAL_PROMPT=0` / ssh BatchMode |

## 实现对象清单

| # | 对象 | 操作 | 说明 |
|---|---|---|---|
| 1 | `skills/case-execution/scripts/init_cases_repo.py` | 新建 | 真相源脚本（HLD §5.2 执行序 7 步全实现；CLI `--cases-dir/--remote/--branch main/--json`；退出码 0/1/2） |
| 2 | `ptm-te-manaul/scripts/init_cases_repo.py` | 新建 | 分发副本（与真相源 `diff` 逐字节一致） |
| 3 | `skills/case-execution/tests/test_cr056_init_cases_repo.py` | 新建 | mock subprocess 单测 20 例（不真跑 ssh 与 git switch） |
| 4 | `ptm-te-manaul/cases/.git` | 脚本产物 | 实际执行初始化产生的半初始化态（origin 已配 / `main` unborn / 索引空 / pack 已 fetch）；314 个内容文件零改动 |
| 5 | `process/stories/STORY-056-01-cases-repo-init-IMPLEMENTATION.md` | 新建 | 本证据 |
| 6 | `process/checks/CP6-STORY-056-01-cases-repo-init-CODING-DONE.md` + `.result.json` | 新建 | CP6 门禁 |
| 7 | `process/returns/STORY-056-01-cases-repo-init-CP6.return.json` + `process/evidence/STORY-056-01-cases-repo-init-CP6.index.json` | 新建 | Return Packet / Evidence Index |
| 8 | `process/state/QUESTION-LEDGER.ndjson` | 追加 | LCQ-STORY-056-01-01（cases 布局映射决策，阻塞实际对齐） |
| 9 | `DEV-LOG.md` | 追加 | 实现摘要 + 决策项 |

## 设计契约映射

| HLD §5.2 契约 | 实现 | 验证 |
|---|---|---|
| 1 幂等前置：`.git` 存在且 origin 匹配 → `already-initialized` exit 0 零副作用 | `init_cases_repo()` 步骤 1（加强：还须 HEAD 已落地，防中途 abort 误报完成） | 单测 `test_already_initialized_zero_side_effects`（只读命令白名单断言）+ 沙箱二次运行 HEAD/status/branch 前后一致 |
| 2 工作区安全校验：目录存在且含 `te/`；已有 `.git` 但 origin 不符 → abort 不改 remote | 步骤 2 前置校验 + origin 不符分支（永不 `remote set-url`） | 单测 `test_missing_te_dir_aborts` / `test_origin_mismatch_aborts_and_never_changes_remote`；沙箱 wt7 实测 remote 保持原值 |
| 3 SSH 探测：`-o BatchMode=yes -o ConnectTimeout=5 -o StrictHostKeyChecking=accept-new -T git@<host>`，以输出含 `Welcome to GitLab` 判定，禁止 exit code（G-1） | `ssh_reachable()`（stdout+stderr 合并匹配） | 单测 4 例含「exit 1 但欢迎输出→通」「exit 0 无欢迎输出→不通」+ `test_ssh_probe_options_match_hld` |
| 4 连通路径 mode=linked：init → remote add → fetch → `git diff --stat origin/main` 为空才 `git switch -c main origin/main`，不空 abort 绝不覆盖（G-2） | 步骤 4（关键实现细节：空索引下 `git diff` 看不到未跟踪文件，必须先 `git add -A` 再 diff；abort/失败路径 `_restore_index()` 还原暂存；`git init -b main` 确定性初始分支） | 沙箱全流程实测（与远端逐字节一致时 `SWITCH_OK`、status 干净、自动设 upstream）；单测 `test_linked_flow_success` / `test_tree_mismatch_aborts_without_switch`（不 switch、恢复暂存） |
| 5 降级 mode=local-only：init + add + commit（不 fetch）；重跑补对齐；本地领先提交 abort | 步骤 5 + 幂等补齐分支（local-only 重跑幂等 / 补对齐遇本地提交 abort 提示 merge 或 push） | 单测 `test_local_only_degradation_on_ssh_unreachable` / `test_local_only_rerun_is_idempotent` / `test_local_ahead_aborts_on_relink`；沙箱 wt6 实测（commit 文案逐字一致） |
| 6 git 身份缺失仅 WARNING，不代配不阻塞 | `check_git_identity()` 只读检查 | 单测 `test_git_identity_missing_is_warning_not_blocker` |
| 7 摘要 mode/branch/commit/remote/cases_count（te 下 .md 数）；`--json` 机器可读 | `render_summary()` / payload JSON | 单测 `test_main_json_output_contract` / `test_main_human_summary`；沙箱/真实运行输出实测 |
| §10 失败路径 | SSH 不通→local-only；树不一致→abort 不覆盖；身份缺失→WARNING | 全部对应单测 + 沙箱 |
| 安全边界 | 永不 push/force/改 remote；`GIT_TERMINAL_PROMPT=0`；OSError/异常全收敛不暴露 traceback | `_assert_no_push_like_commands` 全场景；`main()` 兜底 except |

## 单元测试与 Fixture 计划

- Fixture：`cases_dir`（tmp_path 真实目录 + te/ 嵌套 .md）、`fake_runner`（`FakeRunner` 替身 `init_cases_repo._run`，维护伪 git 状态机：init/remote/fetch/add/diff/switch/commit/config，ssh 返回预设输出；不真跑 ssh 与 git switch）。
- 20 例覆盖：连通判据与 exit code 解耦（4 组）、探测参数逐项、URL host 解析 6 形态、linked 成功、自定义分支、树失配 abort、半初始化补齐、幂等零副作用（只读命令白名单）、origin 不符不改 remote、缺 te/ 与目录不存在、local-only 降级/重跑幂等/领先提交 abort、身份 WARNING、cases_count 口径（仅 te 下 .md）、`--json` 契约、abort JSON、human 摘要、全流程禁 push/force/set-url。

## 最小实现切片

| Slice | 内容 | 结果 |
|---|---|---|
| S1 | 命令封装 + SSH 探测 + host 解析 | 完成（py_compile + 单测） |
| S2 | 幂等前置 / 安全校验 / origin 不符 abort | 完成 |
| S3 | linked 路径（add -A 后 diff、restore、`init -b`） | 完成 + 沙箱真 git 验证 |
| S4 | local-only 降级与幂等补齐分支 | 完成 + 沙箱真 git 验证 |
| S5 | 摘要输出（human/JSON）+ 兜底异常 | 完成 |
| S6 | 单测 20 例 + 真实执行 + 分发副本 | 完成（真实对齐阻塞，见「未覆盖项」） |

## 平台差异处理

N/A：纯本地脚本，无平台目录/安装结构差异；分发为单文件 `cp`（已 diff 逐字节一致）。WSL2 + git 2.43 + Python 3.11/3.12 双版本 py_compile/pytest 均通过。

## 验证结果

| 验证 | 命令 | 结果 |
|---|---|---|
| py_compile | `uv run --python 3.11 python -m py_compile skills/case-execution/scripts/init_cases_repo.py` | PASS |
| 本 Story 单测 | `uv run --python 3.11 pytest skills/case-execution/tests/test_cr056_init_cases_repo.py -q` | **20 passed** |
| 回归（case-execution 全套） | `uv run --python 3.11 pytest skills/case-execution/tests/ -q` | **86 passed**（含 CR-053/CR-049 既有套件） |
| 沙箱 linked 全流程 | 合成 bare 远端 + 逐字节一致工作区 | exit 0，`switch -c main origin/main` 成功、status 干净、自动 upstream |
| 沙箱幂等 | 二次运行 | exit 0 `already-initialized`，HEAD/status/branch 前后一致 |
| 沙箱降级/abort 各分支 | 不可达 host / 失配工作区 / origin 不符 / 半初始化 / local-only 重跑 | 全部符合 §5.2/§10 |
| 真实 SSH 探测 | `ssh -T git@10.113.53.21` | 输出含 `Welcome to GitLab, @zhaohaibo!` → 连通 |
| 真实远端核对 | `git ls-remote` | 仅 `refs/heads/main = 74073e633be4...`（与 HLD 一致） |
| 真实内容比对 | 本地 314 文件 `git hash-object` vs 远端 `ls-tree -r` blob | **远端 `cases/**` 314 文件与本地 `te/+tde/` 逐字节一致（0 差异）**；远端根另有 `.gitignore`/`README.md`/`docs/execution-guide.md` 三个本地没有的文件 |
| 真实执行 run1/run2 | 真相源与分发副本各跑一次 `--json` | 均 exit 2 abort「树不一致」（确定性、可重跑）；**314 内容文件校验和前后一致（零副作用）**；未 push、未改 remote |

## 未覆盖项

| 项 | 状态 | 说明 |
|---|---|---|
| 真实 cases 目录 mode=linked 对齐 74073e6 | **OPEN（阻塞于用户决策 LCQ-STORY-056-01-01）** | HLD 前提「工作区已与 74073e6 一致」在内容层成立（`cases/**` 子树 314 文件 blob 全同），但在仓库根布局层不成立：远端根 = `.gitignore + README.md + cases/{tde,te} + docs/execution-guide.md`（README 明确 `cases/te` 为 `--cases-dir` 指向层），本地 worktree 根 = `tde/ + te/`。`git diff --stat origin/main` 必然非空 → 按 §5.2 步骤 4 / §10 abort（设计内安全行为）。远端仅 main 一个分支，无其他布局可选。 |
| mode=local-only 分支在真实环境触发 | N/A | 真实 SSH 连通，降级路径未在真实环境触发（沙箱 + 单测已覆盖）。 |

## 设计缺口反馈

1. **HLD §5.2 步骤 4 前提失实**：「工作区已与 74073e6 一致，直接接上远端历史」——实测应为「工作区内容与远端 `cases/` 子树逐字节一致；远端仓库根另含 README/.gitignore/docs 且内容嵌套 `cases/` 前缀」。需 host-orchestrator 修订 HLD 并由用户定案布局映射。
2. **HLD §6 台账 path 口径失实**：「台账 path 相对 cases 仓库根（`te/...` 前缀），与 ptm-cases 入库路径一致」——远端实际入库路径前缀为 `cases/te/...`，按现远端布局该口径不成立（STORY-056-02 已按 `te/...` 固化规则 4 文案，受本决策影响）。
3. **HLD §5.2 步骤 1 幂等前置不完备**（已在本 Story 代码内加强并留痕）：origin 匹配但 HEAD unborn（上次运行中途 abort 的半初始化态）若直接判 `already-initialized`，abort 后重跑将永远空转；已改为「origin 匹配且 HEAD 已落地才判完成，否则继续补齐」。另补 `git init -b <branch>`（用户环境未配 `init.defaultBranch`，否则 local-only 提交会落在 `master`）。
4. **空索引 diff 盲区**（G-2 补充实现要点）：`git diff --stat origin/main` 在空索引下对所有远端文件显示「删除」，必须先 `git add -A` 再比对；abort/失败路径以 `read-tree --empty`（unborn HEAD）/`reset`（born HEAD）还原暂存。

## 实现灰区与取舍记录

- 取舍：abort 后**保留**半初始化态（origin 已配 + pack 已取 + unborn main），不自动清理——这是 §10 设计内状态，保留可使用户决策后一条命令补齐；同时留清理口径（`rm -rf /home/hyde/projects/ptm-te-manaul/cases/.git`）供用户选择彻底回原态。风险：若在决策前运行 STORY-056-05 mark 自动 commit，会在 unborn main 上产生 root commit（触发「本地领先提交」abort 分支）——已在本节与 CP6 风险中显式提示。

## 偏离记录

| 偏离 | 依据 |
|---|---|
| 步骤 1 增加「HEAD 已落地」判定；`git init -b <branch>`；diff 前先 `git add -A`；abort 还原暂存 | 均为实现层稳健性加强，不改变 §5.2 对外契约与安全边界；已记入设计缺口反馈 3/4 |
| 实际执行未达成 mode=linked/local-only，而是设计内 abort | §5.2 步骤 4「不空 → abort，绝不覆盖本地修改」按契约执行；冲突根因见设计缺口反馈 1/2，决策项 LCQ-STORY-056-01-01 |

## 风险与重访条件

- R-056-01-A：决策前对 cases 运行 mark 自动 commit → 产生 root commit 使后续补齐复杂化。重访条件：用户定案布局后按方案重跑（或清理 .git 重来）。
- R-056-01-B：决策若选「本地适配远端布局」，`cases/te` 调用方路径与台账 path 前缀、规则 4 文案（STORY-056-02 已固化 v1.1.0）、agent 三副本需联动修订。

## 后续交接

- meta-qa（CP7）：验证入口 = `uv run --python 3.11 pytest skills/case-execution/tests/test_cr056_init_cases_repo.py -q`（20 例）+ 回归 `skills/case-execution/tests/`（86 例）+ 沙箱 dry-run（见本文件「验证结果」）；脚本 CLI `--json` 契约与退出码 0/1/2；安全边界断言在测试内固化。风险提示：真实 cases 目录当前为半初始化态（origin 已配、main unborn、内容零改动），不要在其上执行 push/commit/merge 类操作，待 LCQ-STORY-056-01-01 定案。
- host-orchestrator：LCQ-STORY-056-01-01（cases 布局映射三选一）待用户决策；HLD §5.2/§6 口径修订与 STORY-056-02 规则 4 文案联动，需在决策后统一处理。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| r1 | 2026-09-03 | meta-dev（STORY-056-01 首轮） | 初版：脚本 + 20 例单测 + 分发副本 + 沙箱七分支实测；真实对齐因远端根布局差异 abort（LCQ-STORY-056-01-01 待决策）；CP6 PASS（1 项 OPEN） |
| r2 | 2026-09-03 | meta-dev（NEEDS_REWORK 增量收口） | §实现对象清单 1/3、§设计契约映射步骤 1/2、§验证结果、§未覆盖项按本轮增量更新；详见下节 |

## 增量修订 r2（NEEDS_REWORK 收口，2026-09-03 用户拍板 symlink 方案）

**背景**：用户已定案并落地 symlink 方案——`ptm-te-manaul/cases` → symlink → `/home/hyde/projects/ptm-cases/cases`（.git 在 `/home/hyde/projects/ptm-cases` 仓库根，HEAD=main 74073e6 已对齐 origin、工作树 0 变更；cases 下半初始化 .git 已删除，实体备份 `/tmp/ptm-te-manaul-cases-bak-20260903`）。LCQ-STORY-056-01-01 据此 resolved（用户自建方案，非 A/B/C 原选项）。

**增量需求（本轮 only 改 init_cases_repo.py 及其测试）**：

1. **幂等前置改造为「向上发现 git 仓库」**：以 `git rev-parse --show-toplevel` 从 `--cases-dir` 起向上发现（允许 cases-dir 是仓库内子目录，如 symlink 目标 `ptm-cases/cases` 而 .git 在 `ptm-cases` 根）——发现仓库且 origin == 期望且 HEAD 已落地 → `already-initialized`（toplevel != cases-dir 时 `mode=repo-subdir`，否则 `mode=linked`）exit 0 零副作用；发现仓库但 origin 不符 → abort exit 2（不自动改 remote）；**新增保护**：上级仓库未配置 origin → abort（脚本不自动为非本脚本创建的仓库添加 remote）；未发现仓库 → 走原 init 流程（目录即根，r1 逻辑不变，含 `rev-parse` 失败但 `.git` 存在的 legacy 兜底）。
2. **树一致性预检保留（已对齐仓库）**：跳过 diff/switch（已对齐无需动作）；`git status --porcelain` 非空且涉及 cases 下文件（porcelain 路径相对仓库根，cases 前缀由 `realpath(cases_dir)` 与 toplevel 换算，正确处理 symlink）→ WARNING 提示提交前检查（不阻塞）；不涉及 cases 下文件不告警。
3. **单测**：新增 2 例 `test_repo_subdir_already_initialized_zero_side_effects`（mock `rev-parse --show-toplevel`，断言 mode/repo_root/只读命令白名单/零变更动作）、`test_repo_subdir_dirty_status_warning_non_blocking`（cases 前缀命中告警 / 其他路径不告警）；FakeRunner 增加 `toplevel` / `porcelain` 两个模拟状态与对应命令分发；只读白名单补 `status` / `-c`（core.quotepath=false 为 git 级选项）。
4. **输出契约**：payload 与 human 摘要新增 `repo_root` 字段；mode 值域扩展为 `linked / repo-subdir / local-only`。

**实现 diff 摘要（init_cases_repo.py）**：

- 新增 `_git_toplevel()`（rev-parse --show-toplevel 向上发现）、`_dirty_cases_entries()`（porcelain 解析 + realpath 前缀匹配 + rename 条目取新路径）、`_warn_dirty_cases()`（WARNING 组装）。
- 主流程：`has_git` 改由 toplevel 发现驱动（legacy `.git` 兜底）；幂等前置分支按 `is_repo_subdir` 区分 `mode=repo-subdir/linked` 并写 `repo_root`；origin 不符 abort 文案携带仓库根；新增上级仓库无 origin 的 abort 分支；payload 全分支补 `repo_root`。
- docstring 步骤 1/2/7 同步改写（执行序「HLD §5.2 + STORY-056-01 增量修订 2026-09-03」）。

**r2 验证结果**：

| 验证 | 结果 |
|---|---|
| py_compile（真相源 + 分发副本） | PASS |
| 本 Story 单测 | **22 passed**（20 + 2 新增） |
| case-execution 全套回归 | **88 passed** |
| 真实 symlink 环境运行（真相源） | exit 0：`mode=repo-subdir / status=already-initialized / repo_root=/home/hyde/projects/ptm-cases / commit=74073e633be4... / cases_count=312 / warnings=[]` |
| 真实 symlink 环境二次运行（分发副本） | exit 0，输出与首跑 `diff` 完全一致（幂等零副作用）；`ptm-cases` 仓库 HEAD/porcelain/origin 实测未变 |
| 既有 20 例回归 | 全过（FakeRunner 升级保持原语义） |

**r2 未覆盖项**：真实环境的 dirty-warning 分支未实测（需制造未提交变更，不允许改动用户仓库内容），由 mock 单测覆盖；沙箱 linked/local-only 各分支 r1 已实测且本轮未改动该部分逻辑（88 例回归佐证）。
