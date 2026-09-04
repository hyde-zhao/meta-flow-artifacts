---
story_id: "STORY-056-03"
story_slug: "case-ledger-module"
cr_id: "CR-056"
wave: 2
design_evidence_type: "technical-note"
design_evidence_ref: "process/HLD-CR-056.md §7.1（v0.9，含完整常量与函数签名注释）+ §4.1/§4.2/§4.4 + process/changes/CR-056.md IMP-056-02/07"
implementation_status: "code-complete-module-skeleton"
executed_by: "meta-dev"
executed_at: "2026-09-03T16:30:00Z"
revision: "r1（新模块首次落地；case_runner 接线由 STORY-056-04/05/06 承接）"
---

# STORY-056-03 实现执行证据 — case_ledger.py 新模块（双字段指纹/分域迁移矩阵/台账读写/状态校验/evidence rounds/不变式）

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| 设计证据（technical-note，design-already-in-hld）已确认 | PASS | `process/HLD-CR-056.md` §7.1 v0.9（常量与函数签名注释逐条落地）+ `process/changes/CR-056.md` IMP-056-02/07；host-orchestrator 派发实现任务 |
| 依赖门控满足（depends_on=[]，dependency_type=none；Wave 2 内 056-04 起串行于本 Story 之后） | PASS | `process/DEVELOPMENT-PLAN-CR-056.yaml` waves[1] |
| 文件所有权无冲突 | PASS | primary=`skills/case-execution/scripts/case_ledger.py`（实现前实测不存在，全新文件）；未触碰 case_runner.py / exec_task.py / 其他文件 |
| AI 任务清单存在 | PASS | host-orchestrator 派发任务 1~5（常量/函数/path 口径/基础验证/实现执行证据） |
| 复用事实核对完成（禁止重复实现） | PASS | 先读 `exec_task.py` 确认实际签名：`scan_exec_runs(exec_dir) -> (entries, skipped)`、`build_case_records(run_entries) -> Dict[case_key, record]`（record.appearances 含 started_at/run_id/mode/overall）、`_exclusive_lock` 为 fcntl 尽力锁惯例；`resolve_evidence_rounds` 仅 import 复用 |
| 安全校验 | PASS | 模块不触网、不触设备、不读凭据；exec_task 只读复用；台账 path 只存储不解析 |

## 实现对象清单

| # | 对象 | 操作 | 说明 |
|---|---|---|---|
| 1 | `skills/case-execution/scripts/case_ledger.py` | 新建 | 真相源模块（约 560 行）：常量 11 组 + 函数 10 个（核心纯函数 7 个全实现 + 子命令入口骨架 3 个）+ 降级路径（yaml/exec_task 缺失显式收敛） |
| 2 | `process/stories/STORY-056-03-case-ledger-module-IMPLEMENTATION.md` | 新建 | 本证据 |
| 3 | `process/checks/CP6-STORY-056-03-case-ledger-module-CODING-DONE.md` + `.result.json` | 新建 | CP6 门禁 |
| 4 | `process/returns/STORY-056-03-case-ledger-module-CP6.return.json` + `process/evidence/STORY-056-03-case-ledger-module-CP6.index.json` | 新建 | Return Packet / Evidence Index |
| 5 | `process/state/QUESTION-LEDGER.ndjson` | 追加 | LCQ-STORY-056-03-01（dry-run PASS 计入验收轮数灰区，blocks_lld=false 非阻塞） |
| 6 | `DEV-LOG.md` | 追加 | 实现摘要 + 决策项 |
| 7 | `process/DEVELOPMENT-PLAN-CR-056.yaml` | 更新 | STORY-056-03 status planned → in-development → ready-for-verification + execution_note |

禁改边界实测：`git status --porcelain` 中 `agents/ptm-te.md` / `script/ptm_team/install.py` / `skills/case-execution/SKILL.md` / `DEV-LOG.md` 的 modified 状态为 Wave 1 前序 Story 未提交改动（会话开始前已存在），本 Story 仅追加 DEV-LOG 段落，未触碰其余三者；case_runner.py / exec_task.py / init_cases_repo.py 零改动。

## 设计契约映射

### 常量（HLD §7.1 v0.9 注释逐条对应，冒烟 §9 全表断言）

| HLD 契约 | 实现 | 冒烟验证 |
|---|---|---|
| MARK_VALUES 5 态 / CICD_MARK_VALUES 3 态（v0.7 双字段） | 模块常量原样 | 值与序逐一断言 |
| META_FIELDS 四字段 + DATE_RE 年月日精度（v0.8，G-14） | 模块常量原样（DATE_RE 保持 HLD 原始串口径，消费方 re.compile） | 断言 |
| TRANSITIONS 工厂域判定表（accept→debug 连带清 cicd_mark；deprecated 吸收态） | 常量原样 + 注释说明连带清除属写入面（调用方 056-05 执行） | 全表断言 |
| CICD_TRANSITIONS 判定表（None→cicd_verify；cicd_accept 吸收态） | 常量原样 | 全表断言 |
| 通用边：任意活动态→debug/→deprecated；deprecated 迁出、cicd_mark 并存 deprecated 拒绝 | `validate_transition` 显式实现→deprecated 通用边（HLD 判定表为摘要视图，未含该目标值）；deprecated 迁出与 CICD 并存拒绝在写点拦截 | 正反例 21 条 |
| ROLE_HINTS 11 边（M1 不强制校验，DQ-056-10） | 常量原样（(cur, nxt) 键，工厂域与 CICD 域同表） | 全表断言 |
| EVIDENCE_REQUIRED 三边 + MARK_STABLE_ROUNDS=3（DQ-056-02） | 常量实现；**MARK_STABLE_ROUNDS 定义置于 EVIDENCE_REQUIRED 之前**（HLD 伪代码列表顺序会 NameError，见设计缺口反馈） | 断言 |
| ROLE_SCENE_POOLS / ROLE_DEFAULT_SCENE / ROLE_SCOPE（§4.4） | 常量原样（池准入消费在 case_runner C-5，STORY-056-04） | 断言 |
| LEDGER_FILENAME | 常量原样；另增私有常量 `.case_ledger.lock`（HLD ledger_lock 行内口径） | 断言 |

### 函数（HLD §7.1 签名与行为，注释转 docstring）

| HLD 契约 | 实现 | 冒烟验证 |
|---|---|---|
| `validate_transition(cur_mark, cur_cicd, nxt) -> Optional[str]`：值域路由；CICD 域前置 cur_mark=="accept"；合法 None / 非法原因串 | 分域路由实现：工厂域（值域/吸收态迁出/判定表/通用边→deprecated/自环拒绝）+ CICD 域（mark=accept 前置优先、当前值合法性、判定表、cicd_accept 吸收态）；原因串确定性中文，含 HLD 示例前缀 "cicd_mark 迁移需 mark=accept"（G-13 写点） | 12 正 + 9 反（工厂域）+ 4 正 + 8 反（CICD 域/路由） |
| `compute_case_fingerprint(md_text) -> str`：frontmatter 块内仅剔 `^mark:`/`^cicd_mark:` 行后全文 sha256；无 frontmatter 对原文哈希 | `\A---\n(.*?)\n---` DOTALL 定位；块内整行剔除（含行分隔符），保证打标前后逐字节等价；`^mark:` 前缀精确匹配不误剔 "marked:" 类键 | 打标前后不变 / 改 case_steps 变 / 作者元数据行变更变（变更证据）/ 正文 `mark:` 字样不误剔（G-4）/ 块尾单行剔除恒等 / 无 frontmatter 原文哈希 / 无 mark 行时 == 原文 sha256 |
| `load_ledger(cases_root) -> dict`：不存在 → 骨架 {metadata:{schema_version:'1.0'}, cases:{}} | 骨架返回；文件存在时结构校验（顶层/metadata/cases 须 mapping），非法抛 ValueError 绝不静默重置（审计真相源）；yaml 不可用且文件存在 → RuntimeError 中文收敛 | 骨架断言 + save→load roundtrip；yaml=None 双分支（不存在→骨架 / 存在→RuntimeError） |
| `save_ledger(cases_root, ledger) -> None`：中文原样输出（G-7）、sort_keys=False、刷新 updated_at、人工区不增不删 | `allow_unicode=True`（**HLD 注释 ensure_ascii=False 为 json 口径，PyYAML 无该参数**，见设计缺口反馈）+ tmp+os.replace 原子写；只刷新 metadata.updated_at | 中文不转义 `\u` / 人工区 submitter/notes/open_issues 原样保留 / updated_at 刷新 / 无 .tmp 残留 / 临界区二次写一致 |
| `ledger_lock(cases_root)`：fcntl 文件锁 `.case_ledger.lock`，参照 exec_task._exclusive_lock | `@contextmanager` 逐行对应 _exclusive_lock 惯例（尽力锁、非 POSIX 静默降级、open("a")、finally unlock+close） | 锁临界区内 save 双次成功 |
| `verify_case_state(md_text, case_id, ledger) -> (ok, reason)` | 台账无登记 / md 状态非法 / mark 漂移 / cicd_mark 漂移（台账为准）+ 指纹失配（需先状态一致）；缺省口径 mark=debug、cicd_mark=None | 7 例（含缺省口径与缺 content_sha256 分支） |
| `append_mark_history(entry, *, mark, cicd_mark, by, role, note, evidence, rounds)`：首次 debug→verify 写 first_submitted_at | 追加式写（不改写既有项）；记录 8 字段；以调用时刻 `entry["mark"]` 为迁移前态判定 first_submitted_at（契约：调用方先 append 后更新 entry 字段，与 HLD §7.3 步骤 4 顺序一致）；不写 META_FIELDS（归档检查职责，DQ-056-11） | 首次写入 / 二次不覆盖 / 字段齐全 / 非 debug→verify 不写 |
| `resolve_evidence_rounds(runs_dir, exec_id, case_id) -> (ok, desc)`：复用 exec_task.scan_exec_runs + build_case_records 只读 | sys.path 注入同目录 import exec_task（模式同 case_runner §9.1）；exec_dir=<runs_dir>/<exec_id>；按容器内该 case **最近连续 PASS run 数** ≥ MARK_STABLE_ROUNDS 判定；描述含轮数与门槛 | 3 轮 True / 1、2 轮 False / 末轮 FAIL 打断 / FAIL 后重连续不计累计 / 容器不存在 / 用例不存在 / dry-run 计入（HLD 字面口径，灰区留痕） |
| `cmd_mark(args) -> int` 骨架 | --set 值域路由校验（域外 → 2）+ --role/--scene 值域 + feature+acceptance 非法组合拦截（§4.4 规则 1）；路由通过后提示 STORY-056-05 接线并返回 5 | 6 例 |
| `cmd_ledger(args) -> int` 骨架 | --cases-root 可用性 + --mark 工厂域值域校验；统计视图由 STORY-056-06 接线 | 3 例 |
| `cmd_archive_check(args) -> int` 骨架 | 占位 exit 5（任务允许 exit 0，取 5 防归档门禁 fail-open，见设计缺口反馈） | 1 例 |

## 单元测试与 Fixture 计划

- 本 Story 基础验证为交互冒烟（临时脚本 `/tmp/smoke_case_ledger_056_03.py` + `/tmp/smoke_yaml_none_056_03.py`，**不落仓库**）：95 + 8 = **103 条断言**，覆盖指纹 8 例、迁移判定 33 例、状态校验 7 例、历史追加 4 例、台账读写 7 例、evidence rounds 8 例（真实 result.json fixture 构造于 /tmp，走 exec_task 真实 scan/build 路径）、入口骨架 10 例、常量表 11 例、降级路径 8 例。
- **正式 pytest 文件 N/A（后置）**：`tests/test_cr056_mark_lifecycle.py` 属 STORY-056-07 测试与同步收口范围（CR-056 Story 拆解与验证方式章节明确）；本 Story 冒烟断言即该测试文件的用例蓝本，已按可迁移断言书写。
- Fixture 计划（移交 STORY-056-07）：真实 te 用例 frontmatter 形态样例（含中文键/引号值/tags 行）、mark/cicd_mark 行三位置（块首/块中/块尾）、正文伪键、/tmp 合成 runs 容器（PASS/FAIL/dry-run 序列）、无 PyYAML 环境分支。

## 最小实现切片

| Slice | 内容 | 结果 |
|---|---|---|
| S1 | 模块头 + sys.path 注入 + exec_task/yaml 可选导入 + 常量 11 组 | 完成 |
| S2 | validate_transition 分域路由（含通用边/吸收态/自环/不变式写点） | 完成 + 冒烟 |
| S3 | compute_case_fingerprint + _fm_mark_fields 最小读点 | 完成 + 冒烟 |
| S4 | load/save_ledger + ledger_lock（原子写 + 人工区保留） | 完成 + 冒烟（yaml 双路径） |
| S5 | verify_case_state + append_mark_history | 完成 + 冒烟 |
| S6 | resolve_evidence_rounds（exec_task 只读复用） | 完成 + 冒烟（真实 result.json fixture） |
| S7 | cmd_mark/cmd_ledger/cmd_archive_check 入口骨架 | 完成 + 冒烟 |

## 平台差异处理

N/A：纯本地 Python 模块（3.11/3.12 双版本 py_compile 通过），不涉及平台目录/安装结构；分发至 4 副本脚本树由 STORY-056-07 统一同步（file_ownership 未含副本路径）。fcntl 锁在非 POSIX 平台静默降级（exec_task 同惯例），仓库验证环境为 Linux/WSL2。

## 验证结果

| 验证 | 命令 | 结果 |
|---|---|---|
| py_compile | `uv run --python 3.11 python -m py_compile skills/case-execution/scripts/case_ledger.py` | PASS |
| py_compile | `uv run --python 3.12 python -m py_compile skills/case-execution/scripts/case_ledger.py` | PASS |
| 交互冒烟（yaml 可用路径） | `uv run --python 3.11 --with pyyaml python /tmp/smoke_case_ledger_056_03.py` | **95 pass / 0 fail** |
| 交互冒烟（yaml 不可用降级路径） | `uv run --python 3.11 python /tmp/smoke_yaml_none_056_03.py` | **8 pass / 0 fail** |
| 既有测试回归 | `uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q` | **88 passed**（与 STORY-056-01 基线一致，零回归；新模块未改任何既有文件） |
| 缓存卫生 | `git check-ignore __pycache__` | ignored（缓存不入库） |

任务指定判据逐项复核（coordinator 冒烟指令 ②）：打标前后指纹不变 PASS；改 case_steps 指纹变化 PASS；正文含 `mark:` 字样不误剔 PASS；debug→verify 合法 PASS；verify→`cicd_*` 拒绝（值域路由：verify 为 mark 域当前态、cicd_verify 为 CICD 域目标 → CICD 前置拦截）PASS；非 accept 设 cicd_verify 拒绝 PASS；accept→debug 合法且连带清 cicd_mark 语义 PASS（拓扑合法 + 清除由调用方写入面执行，docstring 契约注明）；cicd_verify→cicd_accept 合法 PASS；deprecated 迁出拒绝 PASS；--set 值域路由判据 PASS。

## 未覆盖项

| 项 | 归属 | 说明 |
|---|---|---|
| case_runner 接线（argparse/双写/批量/git commit） | STORY-056-05 | cmd_mark 骨架返回 5；接口消费点见 `validate_transition`/`EVIDENCE_REQUIRED`/`resolve_evidence_rounds`/`append_mark_history`/`load_ledger`/`save_ledger`/`ledger_lock` |
| run 准入 + CASE_MARK_STALE + case_state 审计 | STORY-056-04 | 消费 `verify_case_state`/`compute_case_fingerprint`/`ROLE_SCENE_POOLS`；parse_frontmatter [3c]/[3d] 为不变式读点 |
| ledger 状态统计视图 | STORY-056-06 | cmd_ledger 骨架返回 5 |
| archive-check 实体实现 | STORY-056-08 | cmd_archive_check 占位 exit 5 |
| 正式 pytest 文件 | STORY-056-07 | 见"单元测试与 Fixture 计划"N/A 理由 |
| 台账 `--json` 输出 | 未排期（HLD §9.3 "后续可加"） | 对外消费契约本期只固化 schema |

## 设计缺口反馈

| # | 发现 | 处置 | 建议 |
|---|---|---|---|
| 1 | HLD §7.1/G-7 注释 `yaml.safe_dump(ensure_ascii=False)`：PyYAML 无 `ensure_ascii` 参数（json 口径），实测 TypeError | 按事实修正为 `allow_unicode=True`（G-7 真实意图 = 中文不转义，且与 exec_task._dump_task_yaml 先例逐参一致），已冒烟验证中文原样输出 | host-orchestrator 对 HLD §7.1 save_ledger 注释与 G-7 措辞做一次口径修订（过程文档，不阻塞） |
| 2 | HLD §7.1 列表顺序中 `EVIDENCE_REQUIRED`（引用 MARK_STABLE_ROUNDS）先于其定义，Python 下会 NameError | 实现代码将 `MARK_STABLE_ROUNDS = 3` 前置，并加注释说明 HLD 为伪代码序 | 无需回改 HLD（伪代码语义无损） |
| 3 | `resolve_evidence_rounds` 按 HLD 字面口径计入容器内全部 run（含 dry-run）；dry-run 无设备动作，作为"验收通过"证据偏宽松 | 按 HLD 实现并在 docstring 留痕；已写 `LCQ-STORY-056-03-01`（blocks_lld=false，non-blocking-open） | STORY-056-05 收口前由 host-orchestrator 批量询问用户：是否收紧为仅 mode=execute 轮（推荐收紧） |
| 4 | 任务允许 `cmd_archive_check` 骨架 exit 0 占位；归档检查是规则 17 的 push 前门禁，若 case_runner 先于 STORY-056-08 注册子命令，exit 0 会被误读为"检查通过"（fail-open） | 骨架取 exit 5 + stderr 占位说明 | 若 host-orchestrator 坚持 exit 0 语义请明示，一行可改 |
| 5 | mark 同值自环（如 verify→verify）：HLD 未显式定义；按判定表 strict 语义判非法（无状态变化，调用方应跳过） | 返回确定性原因串含"自环"字样 | STORY-056-05 接线时对同值 --set 预判跳过（提示"无状态变化"而非报错） |

## 后续交接

- **给 meta-qa（CP7 验证入口）**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（88 passed 基线）+ 冒烟脚本重放（/tmp 两脚本，断言 103 条）；本 Story 无新增 pytest 文件（后置 056-07），验证对象为模块常量/纯函数契约与骨架退出码。
- **给 STORY-056-04（interface 依赖）**：消费 `verify_case_state` / `compute_case_fingerprint` / `ROLE_SCENE_POOLS` / `ROLE_SCOPE` / `MARK_VALUES` / `CICD_MARK_VALUES`；parse_frontmatter [3c]/[3d] 与本模块 `validate_transition` 构成 G-13 不变式双点校验的读点。
- **给 STORY-056-05**：mark 子命令写序列必须为"迁移校验 → 证据校验 → `append_mark_history`（先）→ 更新 entry mark/cicd_mark/content_sha256（后）→ `save_ledger`（全程 `ledger_lock`）"；accept→debug 与 →deprecated 连带清 cicd_mark 属写入面；cicd_mark=none 删除 frontmatter 行。
- **风险提示**：目标现场环境无 PyYAML（pyproject 仅 inquirerpy）——台账读写在该环境显式 RuntimeError（中文可读），case_runner 接线时需 catch 并收敛为退出码（建议 5 / LEDGER_MISSING 路径），已冒烟验证双分支。
