---
story_id: "STORY-056-07"
story_slug: "tests-and-sync-closure"
cr_id: "CR-056"
wave: 3
design_evidence_type: "technical-note"
design_evidence_ref: "process/HLD-CR-056.md v0.9 §13（验收矩阵 G-01~G-09 + 全局回归）+ §8（规则与文档同步面）+ §2 F-01/F-06（副本清单）+ CR-056 验证方式段"
implementation_status: "code-complete-closure"
executed_by: "meta-dev"
executed_at: "2026-09-04T02:10:00Z"
revision: "r1（正式测试落盘 + 4 副本脚本同步 + OQ-056-02-01 收口 + SKILL.md 深度 + 执行指导 §9）"
---

# STORY-056-07 实现执行证据 — 测试与同步收口（正式 pytest / 4 副本同步 / .codex TOML / SKILL.md / 执行指导）

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| 设计证据（technical-note，design-already-in-hld）已确认 | PASS | host-orchestrator 派发任务 1~5（pytest / 副本同步 / TOML 收口 / SKILL.md / 执行指导）；蓝本 = 056-03~06 冒烟（103 + 69 + 55 + 29 断言） |
| 依赖门控满足（depends_on=01~06，all-waves） | PASS | 056-01~06 全部 ready-for-verification（CP6 PASS）；交接清单（测试蓝本 / 4 副本清单 / SKILL 提示项 / 08 复用件）逐项消费 |
| 文件所有权无冲突 | PASS | primary = `tests/test_cr056_mark_lifecycle.py` + `SKILL.md` + `docs/ptm-te/执行指导.md`；副本同步与 TOML 为派发任务显式清单；`case_runner.py` canonical 零改动（forbidden：只同步副本不改逻辑） |
| 复用事实核对 | PASS | install.py `render_codex_agent`/`toml_multiline`/`MANAGED_VERSION` 先读后直调；exec_task.splice marker 口径已核对；agent 副本 audit 行口径（version=1.0.0 commit=ad89670）取自 claude 副本实测 |
| 安全校验 | PASS | 单测全部 tmp_path + mock exec 容器 + 临时 git 仓库；真实 ptm-cases / 312 条用例只读；分发同步只覆盖派发清单 4 文件；永不 push |

## 实现对象清单

| # | 对象 | 操作 | 说明 |
|---|---|---|---|
| 1 | `skills/case-execution/tests/test_cr056_mark_lifecycle.py` | 新建 | 正式 pytest（118 用例，蓝本 = 03~06 冒烟） |
| 2 | `ptm-te/.claude`、`ptm-te-manaul/.claude`、`ptm-te-manaul/.agents` 三处 `skills/case-execution/scripts/{case_runner,case_ledger,init_cases_repo}.py` | 同步覆盖 | 以 canonical 为准；`exec_task.py` 三处本就一致零改动；同步前备份 /tmp/cr056-07-backup |
| 3 | `ptm-te-manaul/.codex/agents/ptm-te.toml` | 同步刷新 | OQ-056-02-01 收口：正文 v2.5 → v2.6 + header 刷新（直调 render_codex_agent） |
| 4 | `skills/case-execution/SKILL.md` | 修改 | R-F-027/028 + ST-EX-18/19/20 + Gotchas #12~#15 + 命令示例路径 cases/te + 目录树 + 修订记录 v2.2 |
| 5 | `docs/ptm-te/执行指导.md` | 修改 | 新增 §9（mark 状态流转与三场景运行） |
| 6 | 过程证据（IMPLEMENTATION / CP6 / return / evidence / DEV-LOG / plan 状态） | 新建/更新 | 本 Story 记录 |

禁改边界实测：`agents/ptm-te.md`（canonical）与 2 份 claude agent 副本本 Story 零改动（三方正文一致性为既有状态复核）；`case_runner.py`/`case_ledger.py` canonical 逻辑零改动（git diff --stat case_runner.py 维持 056-04 的 +678/-7）；真实 ptm-cases 只读。

## 设计契约映射

### 1. 正式 pytest（118 用例，全绿）

| 蓝本 | 测试类 | 覆盖 |
|---|---|---|
| 056-03 冒烟 | TestFingerprint（8）/ TestValidateTransition（27：15 合法 + 12 非法参数化）/ TestVerifyCaseState（6）/ TestResolveEvidenceRounds（6） | 指纹 8 例（打标不变/双行不变/内容变/正文伪键不误剔/元数据行参与/无 frontmatter/块尾恒等/sha256）；迁移矩阵含值域路由与自环/吸收态/不变式前置；verify_case_state 六分支；resolve 仅 mode=execute 计入（dry-run 被过滤 → 连续 2<3） |
| 056-05 冒烟 | TestCmdMarkGuards（7）/ TestCmdMarkHappyPath（16） | 值域/入口互斥/存在性/角色组合/圈选值域/无台账根 exit 2；正例链（块尾插入+first_submitted_at+history+path 前缀+git commit 文件对+双写不变式）、证据三边（3 轮达标/缺 evidence/纯 dry-run 容器/1 轮不足）、None→cicd_verify 容器追溯、非 accept 拒绝、连带清 cicd_mark（md 删行+台账 None+commit 双域串）、deprecated 吸收态、自环跳过零 commit、漂移以台账为准、--no-commit、非 git 降级、无 PyYAML exit 5、批量圈选 |
| 056-06 冒烟 | TestCmdLedger（12） | 矩阵计数（accept 行 3/1/1/1/6、debug 1、verify_fail 2、deprecated 0）、待办三类、漂移四类、first_submitted 摘要、--mark 过滤、参数错误、--out splice 三态（首生成/重写保留人工区/无 marker 拒绝 exit 3）、exec_task 缺失 exit 5、无 PyYAML exit 5、只读边界 |
| 056-04 冒烟 | TestParseFrontmatterMark（7）/ TestFilterByMark（4）/ TestRolePoolGuards（17）/ TestFreezeCheck（6）/ TestSubcommandDelegation（2） | parse [3c]/[3d]/[3e]/早退键形状；filter 并集与 none 语义；role×scene 越权 10 例参数化 exit 2 + 合法组合 + 四角色默认池与 scope 推导 + 显式覆盖（G-11）+ 无 --role 零变化 + 圈选链 + 单文件 WARNING；CASE_MARK_STALE 阻塞（dry-run 生效、固定话术、case_state=False）/ local 放行 / LEDGER_MISSING 双场景 / 快照 ok / deprecated 卡口；子命令委托与模块缺失 exit 5 |

运行结果：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q` → **206 passed**（88 既有 + 118 新增，零回归）。单测不触真实设备 / 真实 ptm-cases（全部 tmp_path + mock + 临时 git）。

### 2. 4 副本脚本同步

| 文件 | 处置 | 校验 |
|---|---|---|
| case_runner.py / case_ledger.py / init_cases_repo.py | canonical 覆盖分发至 ptm-te/.claude、ptm-te-manaul/.claude、ptm-te-manaul/.agents（case_ledger/init_cases_repo 为**新增分发**——056-03/01 新模块此前未分发） | 同步后 9 文件 `diff -q` 全部一致 |
| exec_task.py | 三处本就一致，**零改动跳过**（按派发指令） | diff -q 一致 |
| 同步前备份 | `/tmp/cr056-07-backup/`（6 个被覆盖文件） | 可回滚 |

### 3. OQ-056-02-01 收口（ptm-te-manaul/.codex/agents/ptm-te.toml）

- **机制核实**：TOML 由 `install.py render_codex_agent` 生成（name/color/tools_raw 直传/description/developer_instructions）；规则块（ptm-te-workflow）走 CLAUDE.md/AGENTS.md 托管块，**TOML 只承载 agent 定义正文，不塞规则块**（核实既有机制后确认）
- **收口方式**：直调 `render_codex_agent`（importlib 挂载 install.py），instructions = canonical `agents/ptm-te.md` 剥 frontmatter 正文（v2.6），name/description/color/tools 取 canonical frontmatter（与原 TOML 值逐一相同），header 与 claude 副本**同口径**：`version=1.0.0（MANAGED_VERSION）canonical-commit=ad89670 generated=<本次时间>`
- **校验**：① header 正确；② developer_instructions 与 canonical 正文**同口径转义后逐字节一致**（body 含 1 个反斜杠，`toml_multiline` 转义正确）；③ 正文标记全命中（cases/te/、mark 准入章节、v2.6 修订行）；④ `cases/<协议族>` 仅存于 v1.6 修订记录行（历史语义合法保留）；⑤ tools 数组无引号为安装器 `tools_raw` 直传**既有格式**（新旧文件同格式，非本次引入）
- **三方正文一致性终态**：canonical / 2 份 claude 副本（规范化首部空行后）与 .codex TOML（同口径转义后）四方一致

### 4. SKILL.md 深度收口（v2.2）

| 项 | 内容 |
|---|---|
| ST-EX-15 区域 | 标题补 R-F-027/R-F-028；16 列清单附加列扩展（tags 第 17 / mark+cicd_mark 第 18/19 / 作者元数据四字段）；新增小节「mark / cicd_mark 状态列（R-F-027）」与「作者元数据四字段（R-F-028）」 |
| 新增 ST-EX-18 | mark/cicd_mark 圈选与运行准入：--mark/--cicd-mark/--role/--scene/--run-scope 参数表 + 角色×场景圈池表 + 准入规则 5 条 + CASE_MARK_STALE/LEDGER_MISSING 语义 |
| 新增 ST-EX-19 | mark 子命令（值域路由/note 边界/证据 3 轮仅 execute 计入/写序列/git commit 边界/批量部分失败）+ ledger 子命令（四段视图/--out splice/台账 path 口径） |
| 新增 ST-EX-20 | 归档推送前检查命令形态 + 规则 17 文案 + **STORY-056-08 实体交付占位声明** |
| Gotchas | 新增 #12（双域与指纹 G-4/G-12/G-13）、#13（INVALID vs CASE_MARK_STALE，G-6）、#14（role 与 scope 正交，G-11）、#15（台账路径/--evidence 绝对路径/人工区勿贴 marker） |
| 路径修正 | 命令格式示例 4 处（IPv4策略路由 ×3、二层转发 ×2）→ `cases/te/...`，旧路径残留 grep = 0（`cases/upload` 为转换规则表历史语义保留）；R-F-022 目录树改 `cases/te` + `cases/tde` 只读 + `case_ledger.yaml` |
| 修订记录 | v2.2 行（变更要点精确到章节） |

### 5. 执行指导.md §9（新增）

「mark 状态流转与三场景运行（CR-056）」：9.1 状态字段表（双域 + 不变式）；9.2 三场景典型命令序列（feature 本地 / factory 日常+验收含 3 轮打标 / cmo 提交+回归+验收）+ 失败打回全链；9.3 台账查看（ledger/--out）与归档检查一句话（引用 056-08）。§8 历史实战记录（cases/upload）保留并加注现入口为 cases/te。

## 单元测试与 Fixture 计划

- 正式 pytest 已落盘（本 Story 交付物）；fixture 全部 tmp_path 动态构造（用例 md 生成器 / 台账 save_ledger / mock exec result.json / `git init` 临时仓库 + local 身份 / monkeypatch 注入 yaml=None 与 exec_task=None / case_runner.case_ledger=None）。
- 冒烟脚本（/tmp/smoke_cr056_03~06）保留为蓝本原件；056-04 冒烟 F2b/F4 与 056-05/06 冒烟为跨 Story 演进断言的过渡版本，正式资产以本 pytest 文件为准。

## 最小实现切片

| Slice | 内容 | 结果 |
|---|---|---|
| S1 | test_cr056_mark_lifecycle.py 编写 + 调试（118 用例全绿） | 完成（初版 3 类问题：stdout/stderr 断言位、夹具路径拼接、计数口径——均为测试侧修正，产品零改动） |
| S2 | 4 副本脚本同步 + 备份 + diff 校验 | 完成（9 文件覆盖 + 3 文件跳过） |
| S3 | .codex TOML 收口（render_codex_agent 直调 + 三方一致性校验） | 完成 |
| S4 | SKILL.md 深度收口（v2.2） | 完成 |
| S5 | 执行指导.md §9 | 完成 |
| S6 | 一致性总校验 + 证据落盘 | 完成 |

## 平台差异处理

分发覆盖 claude（.claude）与 codex（.agents / .codex）两类目录形态；TOML 严格 tomllib 解析对 tools 数组无引号格式不适用——该格式为安装器 `tools_raw` 直传**既有口径**（新旧 TOML 同格式，与 claude 副本 tools 行逐字一致），非本 Story 引入，保持兼容不动（设计缺口反馈 #2）。单测环境差异：fcntl 锁/git 身份在 Linux/WSL2 实测通过，测试内 git 显式配置 local 身份规避环境依赖。

## 验证结果

| 验证 | 命令 | 结果 |
|---|---|---|
| 全量回归 | `uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q` | **206 passed**（88 既有 + 118 新增，零回归）——完成门槛达标 |
| 新增文件用例数 | `pytest test_cr056_mark_lifecycle.py --co -q` | 118 collected |
| 4 副本脚本 | 逐文件 `diff -q`（canonical vs 3 分发 × 4 脚本） | 12/12 一致 |
| agent 三方正文 | canonical vs 2 claude（规范化）vs TOML（同口径转义） | 全部一致 |
| TOML header | `# ptm-team-managed: version=1.0.0 canonical-commit=ad89670 generated=2026-09-03T14:43:16Z` | 与 claude 副本同口径 |
| SKILL.md 结构 | ST-EX-18/19/20、R-F-027/028 命中 13 处；旧路径 grep = 0 | PASS |
| py_compile | 3.11/3.12 双版本 case_runner/case_ledger | PASS |

## 未覆盖项

| 项 | 归属 | 说明 |
|---|---|---|
| archive-check 实体 | STORY-056-08 | SKILL.md ST-EX-20 已按占位文案交付（命令形态 + 规则 17），实体实现后无需回改 SKILL.md |
| 真机三场景端到端 | 运行授权域 | 单测为 dry-run/mock；真实设备执行属 runtime_authorization（DQ-01/CR038-R-AUTHZ 边界不变） |
| 存量 312 用例作者元数据回填 | O-056-06（独立后续 CR 候选） | HLD §14 既定 |
| `ledger --json` | 未排期 | HLD §9.3"后续可加" |

## 设计缺口反馈

| # | 发现 | 处置 | 建议 |
|---|---|---|---|
| 1 | 协调者交接清单中"056-04 冒烟 70 断言蓝本"含两条跨 Story 演进过渡断言（F2b/F4，分别随 05/06 实体化失效并已更新） | 正式 pytest 以终态行为为准（cmd_mark 缺目标 exit 2、cmd_ledger rc 0），过渡断言不落入仓库资产 | 无需动作；/tmp 冒烟脚本非仓库资产，正式回归以 pytest 文件为准 |
| 2 | install.py `render_codex_agent` 的 tools 数组无引号（`tools = [Bash, ...]`）不满足严格 TOML 1.0（tomllib 解析失败）；Codex 实际容忍该格式（既有 TOML 同格式在用） | 保持格式 parity 不动（新旧一致）；若 Codex 未来收紧解析属安装器 CR | 独立后续 CR 候选：tools_raw 输出补引号（一行改动 + 4 处分发刷新） |
| 3 | canonical agents/ptm-te.md 与 claude 副本正文存在 1 个首部空行差异（安装器渲染惯例，056-02 交付时即如此） | 规范化后一致，语义零差异；不回改（回改需触碰 056-02 交付面） | 无需动作；如追求逐字节一致属安装器 CR |
| 4 | 分发目录存在既有 `__pycache__`（ptm-te/.claude 与 ptm-te-manaul/.claude），为运行时生成非交付物 | 未触碰（仓库缓存禁入库存规则针对 ptm-team 仓库；外部工作区缓存不属本 Story 清单） | 056-08 或运维清理时可顺带清除 |

## 后续交接

- **给 meta-qa（CP7 验证入口）**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（206 passed = 88 既有 + 118 新增）；4 副本 diff 清单与 TOML 三方一致性校验命令见本文件 §验证结果（可重放）；HLD §13 验收矩阵映射——G-01（parse 全过 + mark 缺省）、G-02（CASE_MARK_STALE 卡口）、G-03（迁移矩阵）、G-06（deprecated 行为）、G-07（角色场景池）、G-08（不变式双点）、G-09 部分（pytest 覆盖元数据指纹语义，A/M 类归档检查行为归 056-08）、全局回归（206 全绿）已由测试资产覆盖；G-04（git 关联）与 G-05（目录语义）由 056-01 交付 + 本 Story 副本同步承接。
- **给 STORY-056-08（archive-check 实体化）**：接线点 = main() 委托分支已就绪（`archive-check` → `cmd_archive_check(args_ns)`，占位 exit 5 防门禁 fail-open）；parser 参数面 --cases-root/--author/--date/--fix/--json 已按 HLD §5.4 备齐；可复用件 = `_extract_case_id`/`_FRONTMATTER_RE`/`compute_case_fingerprint`（M 类实质差异 = 指纹变化，G-15）/`_git_commit_mark` 的 rev-parse 模式/git 根向上发现；SKILL.md ST-EX-20 占位文案已就位（实体交付后无需回改）；规则 17 门禁语义 FAIL → exit 1、非 git → exit 2；副本同步注意：08 交付 archive_check.py 后需按本 Story 同清单分发 3 处 + canonical。
- **风险提示**：① 分发副本同步为覆盖式，外部工作区若有本地改动已被覆盖（备份在 /tmp/cr056-07-backup，仅本次会话有效）；② TOML tools 数组格式为既有安装器口径（缺口反馈 #2）；③ pytest 中 TestFreezeCheck 依赖 env/devices fixture 走 dry-run 真实 main() 路径，op_mapper 缺失环境会失败（与既有 88 例同前提）。
