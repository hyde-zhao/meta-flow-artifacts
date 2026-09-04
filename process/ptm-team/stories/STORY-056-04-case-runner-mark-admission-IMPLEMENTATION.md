---
story_id: "STORY-056-04"
story_slug: "case-runner-mark-admission"
cr_id: "CR-056"
wave: 2
design_evidence_type: "technical-note"
design_evidence_ref: "process/HLD-CR-056.md v0.9 §4.3/§4.4（准入矩阵与角色场景池）+ §7.2 C-1~C-7 锚点 + §7.1 常量 + §12 G-6/G-8/G-11/G-12/G-13"
implementation_status: "code-complete-runner-wiring"
executed_by: "meta-dev"
executed_at: "2026-09-03T22:05:00Z"
revision: "r1（case_runner.py 首轮 CR-056 接线；mark/ledger/archive-check 子命令实体由 056-05/06/08 填充）"
---

# STORY-056-04 实现执行证据 — case_runner mark/cicd_mark schema + 圈选过滤 + run 准入 + 冻结校验 + case_state 审计

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| 设计证据（technical-note，design-already-in-hld）已确认 | PASS | `process/HLD-CR-056.md` v0.9 §4.3/§4.4/§7.1/§7.2（C-1~C-7 锚点级清单由 host-orchestrator 派发任务逐条给出）；锚点按符号定位不用行号 |
| 依赖门控满足（depends_on=[STORY-056-03]，dependency_type=interface） | PASS | STORY-056-03 已 ready-for-verification（CP6 PASS 10/10）；case_ledger.py 实际 API 先读后接线：`MARK_VALUES`/`CICD_MARK_VALUES`/`ROLE_SCENE_POOLS`/`ROLE_DEFAULT_SCENE`/`ROLE_SCOPE`/`LEDGER_FILENAME`/`load_ledger`/`verify_case_state`/`compute_case_fingerprint`/`cmd_mark`/`cmd_ledger`/`cmd_archive_check`（2026-09-03 源码 594 行核对） |
| 文件所有权无冲突 | PASS | primary=`skills/case-execution/scripts/case_runner.py`（056-03 完成后无并行写者）；forbidden=`case_ledger.py` 只 import 不改（git status 实测未触碰）；Wave 2 为同文件串行（056-05 待本 Story 合入） |
| AI 任务清单存在 | PASS | host-orchestrator 派发任务 1~8（C-1~C-8）+ 验证清单（py_compile / 88 例回归 / 交互冒烟 / dry-run 冻结校验接线） |
| 既有行为兼容面核对完成 | PASS | `parse_frontmatter` 既有消费点仅 filter 链（本 Story 内扩展）；88 例既有测试不直接断言 mark 键；run 主流程新增参数全部 default None/缺省，无 --role 时条件分支不激活 |
| 安全校验 | PASS | 不触网、不触真实设备（冒烟全部 dry-run，无 `--execute --authorized`）、不读凭据；台账只读（load_ledger），写路径属 056-05 |

## 实现对象清单

| # | 对象 | 操作 | 说明 |
|---|---|---|---|
| 1 | `skills/case-execution/scripts/case_runner.py` | 修改（+678/-7 行） | C-1~C-8 全部锚点，唯一工程资产改动 |
| 2 | `process/stories/STORY-056-04-case-runner-mark-admission-IMPLEMENTATION.md` | 新建 | 本证据 |
| 3 | `process/checks/CP6-STORY-056-04-case-runner-mark-admission-CODING-DONE.md` + `.result.json` | 新建 | CP6 门禁 |
| 4 | `process/returns/STORY-056-04-case-runner-mark-admission-CP6.return.json` + `process/evidence/STORY-056-04-case-runner-mark-admission-CP6.index.json` | 新建 | Return Packet / Evidence Index |
| 5 | `DEV-LOG.md` | 追加 | 实现摘要 + 交接 |
| 6 | `process/DEVELOPMENT-PLAN-CR-056.yaml` | 更新 | STORY-056-04 status planned → ready-for-verification + execution_note |

禁改边界实测：`git status --porcelain` 中 `DEV-LOG.md`（本 Story 仅追加段落）/ `agents/ptm-te.md` / `script/ptm_team/install.py` / `skills/case-execution/SKILL.md` 的 modified 为 Wave 1 前序 Story 未提交改动，本 Story 未触碰；`case_ledger.py` / `exec_task.py` / `init_cases_repo.py` / `install.py` 零改动；ptm-te-manaul / ptm-cases 仓库零写入（312 条 te 用例仅只读解析校验）。

## 设计契约映射

### C-1 常量（锚点：exec_task import 块之后 + `_FM_TAGS_KEYS` 附近）

| HLD 契约 | 实现 |
|---|---|
| `_FM_MARK_KEY` / `_FM_CICD_MARK_KEY`（C-1） | `_FM_MARK_KEY = "mark"`、`_FM_CICD_MARK_KEY = "cicd_mark"`，置于 `_FM_TAGS_KEYS` 后，注释标明第 18/19 字段与缺省口径 |
| 值域常量 import 自 case_ledger | `from case_ledger import MARK_VALUES as _MARK_VALUES, CICD_MARK_VALUES as _CICD_MARK_VALUES, ROLE_SCENE_POOLS as _ROLE_SCENE_POOLS, ROLE_SCOPE as _ROLE_SCOPE, ROLE_DEFAULT_SCENE as _ROLE_DEFAULT_SCENE`（try/except ImportError 降级：模块缺失时置 None / 空 tuple，能力门控见下） |
| 委托先例（§9.1） | case_ledger import 失败仅在使用 `--mark/--cicd-mark/--role/--run-scope≠local` 或 mark/ledger/archive-check 子命令时报错（exit 5，同 exec-report 委托先例），不影响既有 run/validate/exec-report |

### C-2 parse_frontmatter（锚点：[3] tags 块后新增 [3c]/[3d]/[3e]）

| HLD 契约 | 实现 |
|---|---|
| [3c] mark 枚举校验（工厂域 5 值） | 缺省 `"debug"`；显式值经 `str().strip().strip('"').strip("'")` 归一后不在 `_MARK_VALUES` → `errors.append("mark 值非法: …（合法值域 …）")`；值域为空（模块缺失）时校验降级跳过、键仍按缺省提取 |
| [3d] cicd_mark 枚举校验（CICD 域 3 值） | 缺省 `None`；非法 → `errors.append("cicd_mark 值非法: …（合法值域 无/…）")` |
| 不变式读点（G-13 双点之一） | `cicd_mark != None and mark != "accept"` → `errors.append("不变式违规: cicd_mark=… 要求 mark=accept（CICD 池 ⊆ 工厂 accept 池…）")`；写点在 case_ledger.validate_transition（056-03 已落） |
| 返回 dict 增键 | 正常返回与 4 个早退分支（frontmatter 缺失/PyYAML 缺失/YAML 解析失败/非 dict）统一补 `"mark": "debug"`、`"cicd_mark": None`，保持键形状一致 |
| 既有 16 列 + tags 解析零破坏 | 88 例既有测试全过（含 parse_frontmatter 间接消费路径）；真实 te 库 312 个 md 只读解析：311 用例 errors=0（1 个 README 非 case 文件按既有语义判 frontmatter 缺失），mark 全部缺省 debug、cicd_mark 全部 None |

### C-3 过滤函数（锚点：`filter_by_keyword` 之后）

| 函数 | 契约 |
|---|---|
| `filter_by_mark(cases, marks)` | case.mark ∈ marks（in 语义并集）；空列表透传；mark 键缺失按缺省 debug 参与匹配（G-12 圈选面多选/写入面单值区分已写入 docstring） |
| `filter_by_cicd_mark(cases, marks)` | marks 含 `"none"` → 匹配 cicd_mark is None；其余值 ∈ 匹配；多值并集；空列表透传 |

### C-4 run parser 增参（锚点：`--plan-file` 之后）

| 参数 | 实现 |
|---|---|
| `--mark` | 逗号列表字符串（--tag 同口径），main 内 `_split_csv` 拆分 |
| `--cicd-mark` | 逗号列表，支持 `none` |
| `--role` | `choices=["feature","factory","cmo"]`，default None |
| `--scene` | `choices=["daily","acceptance"]`，default `"daily"` |
| `--run-scope` | `choices=["local","factory","cicd"]`，**default None**（argparse 层无法区分"显式 local"与"缺省"，故缺省 None + main 内解析：`args.run_scope or ROLE_SCOPE.get(role) if role else "local"`，显式传参覆盖角色推导，G-11 正交） |

### C-5 run 过滤链（锚点：[3] 用例发现 + ST-EX-16 过滤段）

| 契约 | 实现 |
|---|---|
| ① 参数级准入（快速失败） | main() run 段最前（exec-report/mark/ledger/archive-check 委托之后、[0a] exec-id 预校验之前）新增 `[0-pre]`：`_validate_role_pool_args(args_ns)` 非法 → stderr + **exit 2**，不触发 ptm-atomic 探测/设备配置/目录副作用。规则：值域合法性（有无 --role 均校验，防拼写静默空选）；feature·acceptance 非法组合；显式 --mark ⊆ 池 mark 域、显式 --cicd-mark ⊆ 池 cicd_mark 域；**跨域圈选拒绝**（--role cmo --mark \* / --role factory|feature --cicd-mark \*，对齐 HLD §4.4 规则 2 示例）；verify_fail/deprecated/cicd_verify_fail 经池子集校验天然越权拒绝 |
| ① 默认池 | `apply_role_pool(cases, role, scene)` 按 `ROLE_SCENE_POOLS[role][scene]` 圈池；cmo 圈池时 mark≠accept 自动排除（子集语义不报错，§4.4 规则 3，防手改 frontmatter 绕过不变式的防御读点）；stderr 输出 `角色场景池准入: --role X --scene Y before -> after（圈池条件 …）`（`describe_role_pool` 渲染） |
| ② 圈选过滤 | ST-EX-16 段扩展为 `--tag → --keyword → --mark → --cicd-mark → --role 池` 链式 AND，仅 `--cases-dir` 批量模式生效；`--case-file` 时过滤参数 WARNING 忽略（同 --tag 惯例，提示文案扩展） |
| 向后兼容 | 无 `--role` 且无 `--mark/--cicd-mark` 时过滤链激活条件与原实现等价（`tag or keyword` ⊂ 新条件），--tag/--keyword 行为零变化（88 例回归 + 冒烟 D1/D3 佐证） |

### C-6 执行循环冻结校验（锚点：parse_case_file 成功后、plan env 重载与环境检查之前）

| 契约 | 实现 |
|---|---|
| 台账加载与定位 | main() 新增 `[0-pre2]`：`_resolve_ledger_root(start)` 自 `--cases-dir`（或 `--case-file` 所在目录）向上最多 2 级定位含 `case_ledger.yaml` 的目录（symlink 场景 `--cases-dir cases/te` → 上级 `cases/` 命中，HLD §5.1）；`load_ledger` 异常显式收敛为不可用（绝不静默重置）；root 与不可用原因记入审计 |
| LEDGER_MISSING | `--run-scope != local` 且台账不可用（未找到或读取失败）→ stderr `LEDGER_MISSING: … 按 HLD §10 阻塞执行` + **exit 2**（fail-closed，在启动校验前、任何设备动作前）；local 场景仅降级（case_state.fingerprint_ok=None） |
| 逐用例校验 | `_case_state_snapshot(case_path, frontmatter, case_id, ledger)`：调用 `case_ledger.verify_case_state(md_text, case_id, ledger)`（mark/cicd_mark 双值 + 指纹 vs 台账）；校验位于 execute 判定与环境检查之前 → **dry-run 同样生效** |
| 失配阻塞 | factory/cicd 场景失配 → 该用例不执行，`_case_mark_stale_record` 记 ERROR，`steps[0].diagnosis.error_type="CASE_MARK_STALE"`、`reason="状态已失效（用例被修改），需重新走流程"`（固定话术），比对细节落 `details.verify_detail`（G-6：与 INVALID 可区分）；overall=ERROR → run 退出码 1 |
| local 放行 | 失配仅 stderr WARNING（含 verify 原因）放行执行；case_state.fingerprint_ok=False 留审计 |
| 吸收态（G-8/G-06 增量） | `case_state["mark"] == "deprecated"` → 不执行（dry-run 同样生效），记 ERROR 非执行结果 `error_type="CASE_MARK_DEPRECATED"`，stderr 提示无机器回退边（G-10）。**任务清单未列此项，属 HLD §4.1/G-8/G-06 验收准则对 run 准入的明确要求且无其他 Story 承接，按最小实现补齐并在本证据留痕**（见设计缺口反馈 #2） |

### C-7 result.json 审计快照（锚点：执行循环 4 个 results.append 点 + result_json 顶层）

| 契约 | 实现 |
|---|---|
| case 记录增 case_state | 正常执行 / ENV_UNSUPPORTED（2 处）/ CASE_MARK_STALE / CASE_MARK_DEPRECATED 记录统一附 `"case_state": {mark, cicd_mark, fingerprint_ok}`（执行时点快照；fingerprint_ok 三态 True/False/None）；CaseParseError 记录不附该键（frontmatter 不可读，快照无意义） |
| 既有字段兼容 | 仅新增键，不改既有键；generate_report / exec_task.build_case_records 按键消费不受影响（88 例 + exec-report 冒烟佐证） |
| run 级语境（增量） | result_json 顶层新增 `"mark_admission": {role, scene, run_scope, mark_filter, cicd_mark_filter, ledger_root}`（审计口径同 runtime_authorization——case_state 的解释语境），HLD 未显式列出，见设计缺口反馈 #3 |

### C-8 main() 子命令注册（锚点：exec-report parser/委托同层）

| 契约 | 实现 |
|---|---|
| mark parser | `--case-file/--cases-dir/--set(required)/--mark/--cicd-mark/--note/--evidence/--rounds(int)/--operator/--role/--scene/--no-commit/--cases-root`，契约面 = HLD §7.3 全参数 |
| ledger parser | `--cases-root(required)/--mark（逗号列表 → list，适配 cmd_ledger 对 list 的迭代校验）/--out` |
| archive-check parser | `--cases-root(required)/--author/--date/--fix/--json`，契约面 = HLD §5.4 |
| 委托 | `command ∈ {mark, ledger, archive-check}` → case_ledger 未加载则 stderr + exit 5；否则委托 `cmd_mark/cmd_ledger/cmd_archive_check(args_ns)`；置于 validate/exec-report 同层，不经过启动校验与设备配置（HLD §9.1） |

## 单元测试与 Fixture 计划

- 本 Story 验证 = 交互冒烟（`/tmp/smoke_cr056_04.py`，**不落仓库**）+ 既有 88 例回归；**正式 pytest 文件 N/A（后置 STORY-056-07）**，N/A 理由同 056-03：CR-056 Story 拆解明确测试收口归 056-07；冒烟 69 断言即 `test_cr056_mark_lifecycle.py` 中 run 准入/冻结校验部分的用例蓝本。
- Fixture：`/tmp/cr056-04-smoke/` 合成树 = `cases/`（台账根，经 case_ledger.save_ledger 构造 3 条目真实指纹）+ `cases/te/demo/`（3 条 16 列 + tags + mark/cicd_mark 用例 md）+ env.yaml/devices.yaml（取形 cr049 测试 fixture，dry-run 可达 PASS）+ `noledger/te/`（无台账树）。真实侧只读校验 = ptm-te-manaul/cases/te 312 个 md 全量 parse。

## 最小实现切片

| Slice | 内容 | 结果 |
|---|---|---|
| S1 | C-1 import + 常量（降级门控） | 完成 |
| S2 | C-2 parse_frontmatter [3c]/[3d]/[3e] + 返回键 | 完成 + 冒烟 A1~A11 |
| S3 | C-3 filter_by_mark / filter_by_cicd_mark | 完成 + 冒烟 B1~B6 |
| S4 | C-4 run parser 5 参数 + C-5 参数级准入（_validate_role_pool_args/_split_csv） | 完成 + 冒烟 C1~C11 |
| S5 | C-5 过滤链扩展 + apply_role_pool/describe_role_pool + --case-file WARNING 扩展 | 完成 + 冒烟 D1~D13 |
| S6 | C-6 台账定位/加载/LEDGER_MISSING + 冻结校验 + CASE_MARK_STALE/CASE_MARK_DEPRECATED | 完成 + 冒烟 E1~E14 |
| S7 | C-7 case_state 快照 + mark_admission 顶层语境 | 完成 + 冒烟 D2/D3/E4/E7/E12 |
| S8 | C-8 mark/ledger/archive-check parser + 委托 | 完成 + 冒烟 F1~F6 |
| S9 | 回归 + py_compile 双版本 + 真实 te 库只读校验 | 完成 |

## 平台差异处理

N/A：纯本地 Python 改动（case_runner.py 单文件），不涉及平台目录/安装结构；4 副本脚本树同步（ptm-te/.claude、ptm-te-manaul/.claude、ptm-te-manaul/.agents）归 STORY-056-07（file_ownership 未含副本路径，本 Story 只改 canonical 源）。fcntl/yaml 等平台差异均在 case_ledger（056-03 已处理）；case_ledger 模块缺失时 run 既有功能零影响（能力门控降级，冒烟未注入该分支——与 056-03 降级路径同构，风险低）。

## 验证结果

| 验证 | 命令 | 结果 |
|---|---|---|
| py_compile | `uv run --python 3.11 python -m py_compile skills/case-execution/scripts/case_runner.py skills/case-execution/scripts/case_ledger.py` | PASS |
| py_compile | `uv run --python 3.12 python -m py_compile …（同上）` | PASS（3.12 下的 SyntaxWarning 为既有 `_extract_topo_constraint` docstring `\d` 转义，非本 Story 引入，编译通过） |
| 既有测试回归 | `uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q` | **88 passed**（与 056-01/056-03 基线一致，零回归） |
| 交互冒烟 | `uv run --python 3.11 --with pyyaml python /tmp/smoke_cr056_04.py` | **69 pass / 0 fail**（A parse 11 + B filter 6 + C role 准入 11 + D 过滤链/零变化 13 + E 冻结校验 14 + F 子命令委托 6 + G 辅助函数 8） |
| 真实库只读校验 | parse_frontmatter 全量扫 ptm-te-manaul/cases/te | 312 个 md：311 用例 0 error、mark 全缺省 debug、cicd_mark 全 None（1 个 README.md 非 case 文件按既有语义判 frontmatter 缺失，不入池） |

任务指定判据逐项复核：parse 非法值/不变式报错（A6/A7/A8）PASS；filter --mark/--cicd-mark/none（B2~B5）PASS；role 越权 exit 2——feature·acceptance（C1）、factory --mark verify_fail（C2）、cmo --mark accept（C4）PASS；无 --role 行为不变（D1/D3 + 88 例回归）PASS；dry-run 冻结校验接线（E1：篡改正文 + `--role factory` dry-run → CASE_MARK_STALE 阻塞 rc=1，校验在 execute 判定前）PASS——真实构造成功，未使用 mock。

## 未覆盖项

| 项 | 归属 | 说明 |
|---|---|---|
| mark 子命令实体（迁移校验/证据轮数/md+台账双写/批量部分失败/git commit） | STORY-056-05 | 本 Story 只落 parser + 委托；cmd_mark 骨架路由校验通过后返回 5 |
| ledger 状态统计视图 | STORY-056-06 | cmd_ledger 骨架返回 5；parser 的 `--mark` 已按 list 适配其校验 |
| archive-check 实体 | STORY-056-08 | 占位 exit 5 防归档门禁 fail-open |
| 正式 pytest 文件 | STORY-056-07 | 见单元测试 N/A 理由；另 4 副本脚本同步、SKILL.md ST-EX-18/19/20 深度收口均归 056-07 |
| `--case-file` 单文件 + `--run-scope factory` 的台账定位 | 已实现但依赖目录约定 | `_resolve_ledger_root` 自用例文件目录向上定位；无台账时按 LEDGER_MISSING 阻塞（fail-closed），与批量模式同语义 |
| cmo·acceptance 对 `cicd_verify` 用例的真实端到端 | 数据不可构造 | 不变式使 cicd_verify 用例必须 mark=accept；冒烟 D12 以合法 3 用例库验证池口径，跨态合成用例仅单元级（G1）验证防御排除 |

## 设计缺口反馈

| # | 发现 | 处置 | 建议 |
|---|---|---|---|
| 1 | HLD §7.2 C-5 与 G-8 对"显式圈选另一域"的口径不一致：§4.4 规则 2 示例明确 `--role factory --cicd-mark cicd_verify`、`--role cmo --mark accept` 拒绝，而 G-8 "--mark accept,cicd_accept 是工厂+CMO 回归池的标准显式表达" 为 v0.7 前单字段残留措辞 | 按 §4.4 规则 2 实现：--role 模式下跨域圈选参数一律 exit 2（无 --role 时 --mark/--cicd-mark 仍可叠加圈选，G-8 场景在无角色模式下成立） | HLD 修订 G-8 措辞（指明该表达仅适用于无 --role 的本地工具性圈选）（过程文档，不阻塞） |
| 2 | G-8/G-06 "deprecated 用例被 --mark 显式选中仍不执行（吸收态 FORBIDDEN）" 在任务 C-1~C-8 清单中无对应锚点，且无其他 Story 承接 run 侧行为 | 按最小实现补齐：执行循环内 mark=deprecated → 不执行（dry-run 同样生效），记 ERROR `error_type=CASE_MARK_DEPRECATED`（G-6 可区分口径），stderr 提示无机器回退边 | host-orchestrator 确认该增量归属本 Story；若需独立 error_type 命名评审（如改 ENV_FORBIDDEN 类），一行可改 |
| 3 | C-7 只定义 case 级 case_state，run 级缺少圈池/冻结严格度语境，离线审计 result.json 时无法还原当时准入条件 | result_json 顶层新增 `mark_admission`（role/scene/run_scope/两个圈选参数/ledger_root），仅新增键不影响既有消费 | 若 host-orchestrator 认为超范围可移除（一处 dict），case 级快照不受影响 |
| 4 | 台账根定位 HLD 未对 run 路径显式定义（仅 §7.3 mark 契约定义了 --cases-root 上溯默认） | `_resolve_ledger_root` 自 --cases-dir / --case-file 目录向上最多 2 级精确匹配 `case_ledger.yaml` 文件名（非模糊匹配），命中即用；找不到按 LEDGER_MISSING 语义处理 | HLD §4.3 或 §7.2 C-6 补一行台账定位口径（过程文档，不阻塞） |
| 5 | 056-03 遗留：`resolve_evidence_rounds` 计入 dry-run 轮灰区（LCQ-STORY-056-03-01）在本 Story 冻结校验路径不涉及（run 只读 verify_case_state），仍待 056-05 收口前定案 | 不变，引用不重复登记 | 维持 056-03 建议（仅 mode=execute 轮计入） |

## 后续交接

- **给 meta-qa（CP7 验证入口）**：`uv run --python 3.11 --with pytest --with pyyaml pytest skills/case-execution/tests/ -q`（88 passed 基线）+ 冒烟重放 `/tmp/smoke_cr056_04.py`（需 `--with pyyaml`，69 断言，重建 /tmp/cr056-04-smoke fixture 由脚本自带）。验证对象 = parse_frontmatter 新键与校验、filter 两函数、role 场景池准入（参数级 + 池圈选）、冻结校验三态（阻塞/WARNING 放行/LEDGER_MISSING）、子命令委托退出码、无 --role 零变化。
- **给 STORY-056-05（same-file-serial 依赖）**：
  - **main() 接线点**：`main()` 内 `if args_ns.command in ("mark", "ledger", "archive-check"):` 分支（exec-report 委托之后、run 段之前）——`mark` 已委托 `case_ledger.cmd_mark(args_ns)`，056-05 只需把 cmd_mark 骨架替换为实体（case_ledger.py 侧），case_runner 侧无需再改；parser 参数面已按 HLD §7.3 备齐。
  - **cmd_mark 签名**：`cmd_mark(args: Any) -> int`（args_ns 属性：`case_file/cases_dir/set/mark/cicd_mark/note/evidence/rounds/operator/role/scene/no_commit/cases_root`；`--set` → `args.set`，`--cicd-mark` → `args.cicd_mark`，`--no-commit` → `args.no_commit`）。现有骨架已实现 --set 值域路由 + --role/--scene 值域 + feature·acceptance 拦截（返回 5 前置校验），实体化时保留这组前置校验。
  - **本 Story 可复用件**：`_split_csv`（圈选参数拆分）、`_resolve_ledger_root`（--cases-root 缺省上溯定位——mark 契约 §7.3 "默认 case-file 上溯到 te/ 的父目录" 可直接复用，注意其 max_up=2 口径）、`filter_by_mark/filter_by_cicd_mark`（--mark/--cicd-mark 圈选子集）、`_case_state_snapshot`（md 当前 mark/cicd_mark 提取口径）；写序列契约（append_mark_history 先行等）见 056-03 IMPLEMENTATION §后续交接。
  - **风险提示**：现场无 PyYAML 环境下 load_ledger 抛 RuntimeError——mark 实体需 catch 收敛为退出码（056-03 建议走 LEDGER_MISSING 类路径）；run 侧已按"台账不可用即不可用"降级处理（mark/ledger 子命令侧 case_ledger 模块缺失统一 exit 5）。
- **给 STORY-056-06（ledger 视图）**：parser 已备 `--cases-root/--mark(list)/--out`；cmd_ledger 骨架返回 5，实体化在 case_ledger.py 侧；漂移清单可直接消费 verify_case_state 语义。
- **风险提示（整体）**：① run 准入默认池/越权口径为 exact 语义，错误提示均已含 HLD 章节引用，CP7 建议逐条对照 §4.4 规则 1~5；② `--run-scope` 显式覆盖角色推导（G-11）已冒烟（D13），文档收口时需在 USER 可见面提示两者正交；③ 冒烟曾暴露冒烟脚本自身两处缺陷（runs 目录同名秒排序取错 result.json、误传不存在参数触发 argparse SystemExit），与产品代码无关，已修正后全绿——QA 重放时若复现请先核对脚本版本。
