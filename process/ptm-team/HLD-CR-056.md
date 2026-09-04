# HLD-CR-056 ptm-te 执行场景化方案设计（mark 生命周期选例 + 用例库 git 关联 + tde/te 目录树）

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v0.1 | 2026-09-03 | host-orchestrator（meta-se 职责代执行，待 CP3 确认） | 初稿：§1~§14 全量；函数级设计基于 case_runner.py @ad89670（3844 行）实测结构 |
| v0.2 | 2026-09-03 | host-orchestrator | 对齐用户提供的用例标签规范表：§4.1 mark 重定义为 9 态流程词汇（debug/verify/verify_fail/accept/unstable/cicd_verify/cicd_verify_fail/cicd_accept/remove）+ 正交 xstate（xfail/skip）；§4.3 移除 --mark-mode min；§7.1/7.2/7.3 同步函数级设计；§11 新增 DQ-056-08~10；§14 新增 O-056-05 |
| v0.3 | 2026-09-03 | host-orchestrator | 用户指令"标签多选/互斥需明确定义 + 运行池按特性责任人/工厂责任人/CMO 三角色分别制定"：新增 §4.4 角色运行池（ROLE_RUN_POOLS 机器准入）、§4.5 标签多选与互斥矩阵；§7.1 常量、§7.2 C-4/C-5 锚点同步；CR-056 IMP-056-01/03、PLAN STORY-056-04 同步 |
| v0.4 | 2026-09-03 | host-orchestrator | 用户纠正运行池与验收流程：工厂责任人区分日常运行（仅 accept）与验收场景（accept+verify，跑完按 3 次验收通过打标）；verify_fail 任何场景不入池；CICD 用例集 ⊆ 工厂用例集且同样分日常（cicd_accept）/验收（cicd_accept+cicd_verify）两场景；run 新增 --scene 参数；unstable→accept 直迁取消（恢复走打回全链），unstable 产生场景明确为 accept 用例回归/验收失败；§4.1/4.3/4.4/7.1/7.2/7.3/§13 同步 |
| v0.5 | 2026-09-03 | host-orchestrator | 用户四点修正：① verify_fail 归属特性责任人（整改后重新提交工厂验收，验收执行方仅打标）；② §4.5 新增 mark × xstate × tags 共存矩阵；③ unstable 状态取消（mark 缩为 8 态，accept 用例回归/验收失败直接 accept→debug 打回）；④ remove=deprecated 废弃态（无专属责任人、永不执行）；§4.1/4.3/4.5/7.1/7.2/§11/§13 与 CR-056/PLAN 同步 |
| v0.6 | 2026-09-03 | host-orchestrator | 用户简化指令：xstate（xfail/skip）不建模；功能标签（特性名称）/环境状态（vdutex/vtgex）不做约定；remove 改名 deprecated（8 态，废弃态无责任人不执行）；tags 仅保留作者属性（含优先级 P0~P4）与 mark 正交共存；§4.1/4.2/4.3/4.5/7.1/7.2/7.3/7.4/§11/§12/§13 全面去 xstate/remove，§4.4 角色场景运行池按特性责任人/工厂·日常+验收/CMO·日常+验收 五场景保持；DQ-056-08 取消；与 CR-056/PLAN 同步 |
| v0.7 | 2026-09-03 | host-orchestrator | 用户概念纠正："mark 内部互斥（8 选 1）不正确，工厂运行是全量、CICD 是工厂运行的子集"→ mark 拆双字段：mark（工厂域 debug/verify/verify_fail/accept/deprecated）+ cicd_mark（CICD 域 无/cicd_verify/cicd_verify_fail/cicd_accept），组内单值互斥、组间并存；机器不变式 cicd_mark≠无⇒mark=accept；factory·daily 全量覆盖已进 CICD 用例；§4.1/4.3/4.4/4.5/7.1/7.2/7.3/7.4/§12/§13 全面双字段化；迁移表分工厂域/CICD 域 |
| v0.8 | 2026-09-03 | host-orchestrator | 用户新需求：① frontmatter 作者元数据四字段（创建人/创建时间/修改人/修改时间，YYYY-MM-DD，创建人/时间创建定稿、修改人/时间随内容修改刷新）——DQ-056-09 台账派生方案作废；② gitlab 归档推送前检查技能（archive_check.py：git diff A/M 校验四字段，未刷新 FAIL 阻断，--fix 自动补齐，规则 17）；新增 §4.1a/§5.4；§7/§13/§14 同步 |
| v0.9 | 2026-09-03 | host-orchestrator | 实施期事实修订（STORY-056-01 实测 + 用户定案 symlink）：远端 ptm-cases.git 根含 cases/ 层（方案 A 内嵌 git 被推翻）→ §5.1 决策表改选方案 B symlink（已落地：ptm-te-manaul/cases → ptm-cases/cases，实体备份 /tmp/…-bak-20260903）；台账 path 前缀 te/→cases/te；init_cases_repo 新增"向上发现仓库（repo-subdir）"幂等前置；archive_check git 范围=clone 根（变更集过滤 cases/ 前缀）；O-056-04 RESOLVED；DQ-056-04 更新 |

## 1. 背景与目标

### 1.1 需求来源

- 用户指令（2026-09-03，见 CR-056 需求基线）：三执行场景（本地 / 自动化工厂 / CICD）按 `mark` 圈选用例；用例库 `ptm-te-manaul/cases` 关联 `git@10.113.53.21:tgfw/tgfw-ptm/ptm-cases.git`，初始化时探测 GitLab 连通性并自动完成配置；目录树区分 `tde` / `te`；ptm-te 生成用例落 `te/`。
- 输入设计基线：`ptm-te-manaul/docs/case-lifecycle-tracking-design.md`（2026-09-03 草稿，下称"原设计"）。其 §一（mark 不占 tags）、§二（状态机）、§三（case_ledger）、§四（准入矩阵）结论采纳并收编于本 HLD；其 §九 开放决策点转入本 HLD §11（DQ-056-01~07）。
- 标签规范输入（2026-09-03 用户第二条指令）：现行用例标签规范表（用例开发 debug/verify、工厂验收状态 debug/verify_fail/verify/accept/unstable、异常标签 xfail/skip、功能标签、环境状态 vdutex/vtgex、其他 remove、优先级 P0~P4、cicd 验收状态 cicd_verify_fail/cicd_verify/cicd_accept、提交标签 submit_time，含维护责任人与互斥约束）。mark v0.2 状态词汇直接对齐该规范（§4.1），不得自创同义词汇。

### 1.2 量化目标

| 编号 | 目标 | 度量 |
|------|------|------|
| G-01 | 三场景可按 mark 机器圈定运行池 | `--mark <list> --run-scope <scope>` 单命令圈池；312 条 te 用例 100% 可解析出 mark（缺省 draft） |
| G-02 | 冻结态不可改可机器卡口 | 实质修改后 factory/cicd 场景 100% 报 `CASE_MARK_STALE` 并阻塞该用例；local 场景 0 阻塞仅 warning |
| G-03 | 状态迁移唯一入口 + 证据绑定 | 非法迁移 100% 拒绝；verified/cicd-passed 打标无 exec 证据或轮数不足 100% 拒绝 |
| G-04 | 用例库版本化 | `ptm-te-manaul/cases` 成为 git 仓库（origin=ptm-cases.git），对齐远端 74073e6；mark 动作产生可审计 commit |
| G-05 | 目录语义固化 | 可执行用例 100% 位于 `cases/te/`；`cases/tde/` 设计源零改动 |

### 1.3 非目标（Out of Scope）与相邻对象边界

- **台账运营视图高级能力**（趋势/跨库聚合）：非目标，属后续 CR；本 CR 的 `ledger` 子命令只做单库统计视图。
- **CICD 无人值守授权实现**（预授权凭据 / 设备白名单）：非目标，DQ-056-06 留 M3 拍板；本 CR 不改 DQ-01 授权语义。
- **ptm-tde / 工厂系统侧消费接口实现**：非目标；本 CR 只固化台账 schema 供其只读消费（§9.3），对方系统改造由各自 CR 承接。与"扩展/澄清"类相邻职责的归属：用例内容生成的编号与格式规则归 ptm-tde（CR-053 既有 GATE-5 卡口），本 CR 不触碰。
- **存量用例 mark 批量初始化**：312 条全部自然落 debug（缺省），无需迁移脚本。
- **tags 静态属性迁移**：tags 机制保留，承载作者声明的属性（含优先级 P0~P4），不迁移进 mark；`--tag/--keyword` 行为零变化。v0.6 用户指令：功能标签（特性名称）/环境状态（vdutex、vtgex）/异常标签（xfail、skip）均不建模，不为它们做任何 frontmatter/tags 约定（规范边界见 §4.1/§4.5）。
- **"验证通过/CICD 通过"不再使用 tags 表达**：tags 保持纯静态属性（协议族/功能域/P0/P1/正向反向/CR 号），现有 `--tag/--keyword` 行为零变化。

## 2. 现状事实（2026-09-03 实测）

| # | 事实 | 证据 |
|---|------|------|
| F-01 | case_runner.py 真相源 `ptm-team/skills/case-execution/scripts/case_runner.py`（3844 行），4 副本（ptm-team canonical、ptm-te/.claude、ptm-te-manaul/.claude、ptm-te-manaul/.agents）当前逐字节一致 | `diff -q` 无差异 |
| F-02 | 现有子命令：run / validate / exec-report；过滤链 `--tag`（ST-EX-16，精确）+ `--keyword`（模糊，append AND）；frontmatter 16 列（R-F-024）+ tags 第 17 字段（R-F-025） | case_runner.py:3069-3163、358-471 |
| F-03 | exec 容器机制（CR-053）：`--exec-id exec-YYYYMMDD-<slug>` → `runs/<exec-id>/runs/<run-*>/`；task.yaml 机器区/人工区；`exec_task.py` 独立模块被 main() 委托 | case_runner.py:3127-3138、exec_task.py |
| F-04 | `ptm-te-manaul/cases/{tde,te}` 已建树：te 312 条原子用例按 `特性/子域/用例.md`，tde 2 份设计源 md；manaul 与 cases 均非 git 仓库 | 目录实测 |
| F-05 | `/home/hyde/projects/ptm-cases` 已 clone（origin=ptm-cases.git，SSH），main 含 74073e6"归档 ptm-te 用例库"，其 `cases/` 与 `ptm-te-manaul/cases` 逐字节一致 | `git remote -v`、`diff -rq` 无差异 |
| F-06 | ptm-te-workflow 规则块由 install.py `RULE_BLOCK_RENDERERS` 渲染，现 v1.0.0，已分发 ptm-te/CLAUDE.md 与 ptm-te-manaul/{CLAUDE.md,AGENTS.md} | 副本头部 managed 标记 |
| F-07 | ST-EX 编号已用至 ST-EX-17，R-F 已用至 R-F-026 | SKILL.md |

## 3. 总体架构

```
ptm-team（工厂仓库，canonical）
  skills/case-execution/scripts/case_runner.py   ── 4 副本同步 ──►  ptm-te/.claude/skills/...
  skills/case-execution/scripts/case_ledger.py   │（新）            ptm-te-manaul/.claude/skills/...
  skills/case-execution/scripts/init_cases_repo.py │（新）          ptm-te-manaul/.agents/skills/...
  skills/case-execution/SKILL.md                 ── ST-EX-18/19、R-F-027
  agents/ptm-te.md（canonical）                  ── 3 副本同步（正文）──►  ptm-te、ptm-te-manaul
  script/ptm_team/install.py（ptm-te-workflow v1.1.0）── 安装刷新 ──►  CLAUDE.md / AGENTS.md 副本

ptm-te-manaul（工作目录，非 git）
  cases/  ── git init + origin=ptm-cases.git（STORY-056-01，内嵌 git 仓库）
    ├─ case_ledger.yaml（新：mark 机器真相源，入库）
    ├─ te/<协议族>/<功能域>/<子域>/<用例编号>.md   （312 条可执行用例，mark 写入面）
    └─ tde/*.md                                    （ptm-tde 设计源，只读，无 frontmatter）
  scripts/init_cases_repo.py（薄分发副本）
  runs/<exec-id>/runs/<run-*>/result.json         （case_state 审计快照落点；runs 不入库）
```

分工正交：**mark 池决定"跑哪些"，--exec-id 容器决定"证据归哪"，topo-planning 链决定"在哪跑"**（与既有 ADR-05/08/09、CR-049 plan-file 链不冲突，全部正交叠加）。

## 4. mark 生命周期 v2（对齐用例标签规范）

### 4.1 状态机与迁移规则

用户提供的标签规范表是现行 tags 体系的事实词汇表；mark 直接采用其"工厂验收状态 / cicd 验收状态 / 其他"三类**流程词汇**。v0.7 起（用户概念纠正 2026-09-03）："工厂验收状态"与"cicd 验收状态"是规范表中**两个独立标签类别，可并存**（工厂运行是全量、CICD 是工厂子集）→ **mark 拆为两个正交状态字段，各自组内单值互斥、组间可并存**；不再是一条 8 选 1 的互斥状态机。v0.6 简化保留：xstate/功能标签/环境状态不建模；tags 保留作者属性（含优先级 P0~P4）。

**双状态字段（用例 frontmatter）**：

| 字段 | 值域（组内互斥） | 缺省 | 语义 |
|---|---|---|---|
| `mark`（工厂验收状态） | `debug` / `verify` / `verify_fail` / `accept` / `deprecated` | `debug` | 用例在工厂域的生命周期；`deprecated` 为用例级废弃吸收态（置位时 cicd_mark 强制清除） |
| `cicd_mark`（cicd 验收状态） | （无）/ `cicd_verify` / `cicd_verify_fail` / `cicd_accept` | （无） | 用例进入 CICD 域后的状态；**机器不变式：cicd_mark ≠ 无 ⇒ mark == accept**（CICD 池恒为工厂 accept 池子集） |

**状态含义与责任人**：

| 字段/值 | 含义 | 归属/维护责任人 | 可否修改 |
|---|---|---|---|
| mark=debug（缺省） | 开发/脚本修改中 | 特性责任人 | 可改（local） |
| mark=verify | 已提交工厂待验收 | 特性责任人（提交） | 冻结 |
| mark=verify_fail | 工厂验收失败 | **特性责任人（整改与重提）**；验收执行方仅打标 | 冻结 |
| mark=accept | 工厂验收通过（"验证通过"） | 工厂责任人 | 冻结 |
| mark=deprecated | 用例级废弃（吸收态，无迁出、不执行、无专属责任人） | N/A | — |
| cicd_mark=（无） | 未提交 CICD | — | — |
| cicd_mark=cicd_verify | CICD 待验收 | 工厂责任人（提交） | 冻结 |
| cicd_mark=cicd_verify_fail | CICD 验收失败 | CMO（打标）；整改归特性责任人（经工厂域打回全链） | 冻结 |
| cicd_mark=cicd_accept | CICD 已交付（"CICD 通过"） | CMO | 冻结 |

**互斥与共存规则**：mark 组内互斥（5 选 1）；cicd_mark 组内互斥（无 + 3 值选 1）；**两组之间可并存**（accept 用例同时打 cicd_accept = 同时在工厂池与 CICD 池）；deprecated 为用例级吸收态（与 cicd_mark 并存非法——置位 mark=deprecated 连带清除 cicd_mark）；mark/cicd_mark 与 tags（作者属性）正交共存（§4.5）。

**迁移规则（validate_transition 分域判定，每条标注责任角色，exact 语义）**：

工厂域（`mark`）迁移：

| 迁移 | 责任角色 | 附加条件 |
|---|---|---|
| debug→verify | 特性责任人 | 首次迁移记台账 `first_submitted_at` |
| verify→verify_fail | 验收执行方（工厂）打标 | 验收场景失败；建议 --note 失败原因 |
| verify→accept | 工厂责任人 | **3 次验收通过**：需 --evidence（验收场景 exec 容器）且同容器内该用例连续 3 轮 PASS（MARK_STABLE_ROUNDS=3，DQ-056-02） |
| verify→debug | 工厂责任人 | --note 必填（打回） |
| verify_fail→debug | **特性责任人（整改）** | --note 必填 |
| verify_fail→verify | **特性责任人（整改完成后重新提交工厂验收）** | 无需 evidence（重提） |
| accept→debug | 工厂责任人 / 特性责任人 | **accept 用例在工厂日常回归或验收场景中失败时打回修复**；--note 必填；**连带：cicd_mark 清除**（重走全链） |
| 任意态→deprecated | N/A（废弃不归属任何人） | --note 必填；吸收态不可迁出；**连带：cicd_mark 清除** |

CICD 域（`cicd_mark`）迁移（前置校验：当前 mark == accept，违反即拒绝——CICD ⊆ 工厂不变式）：

| 迁移 | 责任角色 | 附加条件 |
|---|---|---|
| （无）→cicd_verify | 工厂责任人 | 需 --evidence（引用其验收通过的容器，审计追溯）；不变式 mark=accept 检查 |
| cicd_verify→cicd_accept | CMO | **3 次验收通过**：需 --evidence（CICD 验收场景 exec 容器）且连续 3 轮 PASS |
| cicd_verify→cicd_verify_fail | 验收执行方（CMO）打标 | CICD 验收失败；建议 --note |
| cicd_verify_fail→cicd_verify | CMO | 重提 CICD 验收（失败根因非用例内容时；若涉用例内容修改必须先走工厂域打回 debug 全链——指纹失配自动强制） |

其余一律拒绝（越权 `debug→accept`、`verify_fail→accept`（必须经重提验收）、非 accept 状态设置 cicd_mark（不变式拦截）、`deprecated→*` 迁出、mark=deprecated 时 cicd_mark 并存等）。

**用例集包含不变量（机器强制）**：CICD 池 ⊆ 工厂 accept 池——`cicd_mark ≠ 无 ⇒ mark == accept` 在 mark 子命令与 parse_frontmatter 双点校验；accept→debug 连带清 cicd_mark，保证脱离工厂池的用例不可能留在任何 cicd 池。

### 4.1a 作者元数据字段（v0.8，用户指令 2026-09-03）

用例 frontmatter 新增 4 个可选作者元数据字段（作者域，与 tags 同权，16 列表格不动）：

| 字段 | 格式 | 写入时机 | 维护责任人 |
|---|---|---|---|
| `创建人` | 名称 | 用例创建时写入，此后不变 | 用例作者（特性责任人） |
| `创建时间` | `YYYY-MM-DD` | 用例创建时写入，此后不变 | 同上 |
| `修改人` | 名称 | **每次内容修改时刷新** | 本次修改者 |
| `修改时间` | `YYYY-MM-DD` | **每次内容修改时刷新** | 同上 |

- 时间精度仅到年月日（用户指令），不用秒级时间戳；刷新判等以日期为准。
- 元数据行参与指纹（不作为剔除键）——内容修改必然带修改人/时间刷新 → 指纹失配 → 冻结用例必须先打回 debug（与内容变更语义一致，不额外放宽）。
- 缺失容错：存量 312 条 te 用例均无此 4 字段 → parse_frontmatter 缺省视缺失（不计 INVALID），归档检查（§8.3）对新建/修改文件强制补齐，存量未动文件不强制（避免一次性全量回填；批量回填为独立后续 CR 决策）。
- 唯一写入路径：用例创建/修改流程（作者手工或生成器）与归档检查 `--fix`（§8.3）；mark 子命令不写这 4 个字段。

### 4.2 内容指纹（原设计 §一附带决策采纳）

`compute_case_fingerprint(md_text) = sha256(剔除 frontmatter 块内 mark 行与 cicd_mark 行后的全文)`：

- 打标（mark / cicd_mark 写入）不改变指纹 → 状态迁移不触发失配；
- 任何实质修改（case_steps/预期/参数/其他 frontmatter 列，含创建人/修改人等作者元数据行）→ 指纹失配 → 冻结场景 `CASE_MARK_STALE`；
- 指纹记台账不记 md；**剔除范围仅限 frontmatter `--- ... ---` 块内的 mark、cicd_mark 两键行**，正文出现 `mark:`/`cicd_mark:` 字样不剔除（防误剔，见 §12 Gotcha G-4）。

### 4.3 运行准入矩阵（角色 × 场景，双字段圈池）

| 场景（--role + --scene） | 圈池条件（mark × cicd_mark） | 修改权 | 冻结校验（--run-scope 推导） |
|---|---|---|---|
| feature · daily（本地开发调试，默认） | `mark=debug` | 可改 | local：不做；指纹失配仅 WARNING 按 debug 放行 |
| factory · daily（工厂日常回归） | **`mark=accept`（cicd_mark 任意）——工厂全量** | 不可改 | factory：指纹 vs 台账失配 → 该用例 ERROR `CASE_MARK_STALE` 不执行 |
| factory · acceptance（工厂验收） | `mark ∈ {accept, verify}` | 不可改 | 同上 |
| cmo · daily（CICD 持续回归） | **`cicd_mark=cicd_accept`（工厂 accept 池的子集）** | 不可改 | cicd：同上（严格：失配计入 FAIL 汇总） |
| cmo · acceptance（CICD 验收） | `cicd_mark ∈ {cicd_accept, cicd_verify}` | 不可改 | 同上 |
| 无 --role（本地工具性圈选，向后兼容） | `--mark`/`--cicd-mark` 任意枚举 | 可改 | local（可显式 --run-scope 覆盖） |

- **不运行池**：`mark=verify_fail`、`mark=deprecated`、`cicd_mark=cicd_verify_fail` 不属于任何场景运行池（verify_fail 日常与验收都不跑——用户指令，整改后经 verify 重提）。`--role` 传入时圈选它们 → exit 2 拒绝。
- **工厂全量语义**：已进 CICD 的用例 mark 仍为 accept（两字段并存）→ factory·daily 照常覆盖它们，CICD 回归是叠加在工厂日常之上的子集回归（v0.7 核心修正）。
- `--run-scope` 是冻结严格度开关，由 `--role` 推导（feature→local / factory→factory / cmo→cicd；无 --role 默认 local），可显式覆盖；不另设 `--frozen` 别名。
- 圈选面：`--mark`（工厂域，逗号列表 in 语义）+ `--cicd-mark`（CICD 域，逗号列表，`none` 表示未提交 CICD 的用例）可叠加 `--tag`/`--keyword`；min 模式已移除。
- md 与台账不一致（mark 或 cicd_mark）→ 按状态漂移处理（台账为准 + 告警）。
- DQ-01 单次授权语义不变：`--execute --authorized` 一轮一授权；CICD 无人值守冲突留 DQ-056-06（M3）。

### 4.4 角色场景池（特性责任人 / 工厂责任人 / CMO 分治，双字段圈池）

规范表"维护责任人"列上升为**运行准入维度**：`run --role` 声明运行者角色，`--scene` 区分日常运行与验收场景，机器校验圈池是否越权（exact 语义）。

| 角色（--role） | 场景（--scene） | 圈池条件（mark × cicd_mark） | 推导 --run-scope | 典型命令 |
|---|---|---|---|---|
| `feature`（特性责任人） | `daily`（仅此场景，acceptance 不适用） | `mark=debug` | local | `run --cases-dir cases/te --role feature` |
| `factory`（工厂责任人） | `daily`（日常回归） | `mark=accept`（cicd_mark 任意，**工厂全量**） | factory | `run --cases-dir cases/te --role factory` |
| `factory` | `acceptance`（验收） | `mark ∈ {accept, verify}` | factory | `run --role factory --scene acceptance --exec-id <验收容器>` |
| `cmo`（CICD 责任人） | `daily`（持续回归） | `cicd_mark=cicd_accept` | cicd | `run --role cmo` |
| `cmo` | `acceptance`（验收） | `cicd_mark ∈ {cicd_accept, cicd_verify}` | cicd | `run --role cmo --scene acceptance --exec-id <CICD验收容器>` |

准入规则（run 子命令，`--role` 传入时启用）：

1. `--scene` 默认 `daily`；`--role feature --scene acceptance` 报参数错误（feature 无验收场景）；
2. `--mark`/`--cicd-mark` 未传 → 圈池条件 = 上表默认（角色场景 → mark 域条件；cmo 角色 → cicd 域条件）；显式传入 → 必须满足上表圈池条件，越权（如 `--role factory --mark verify_fail`、`--role factory --cicd-mark cicd_verify`、`--role cmo --mark accept` 均拒绝）exit 2 并列出越权项，不进入执行；
3. `--role cmo` 时若用例 mark ≠ accept → 自动排除（子集语义，不视为越权仅说明）；cmo 域圈池基于 cicd_mark；
4. `--run-scope` 缺省按角色推导，显式传入以显式值为准（role+scene 管池、scope 管冻结严格度，正交，Gotcha G-11）；
5. 无 `--role` → 行为与现状完全一致（向后兼容）——供本地工具性圈选。

**验收场景收尾操作卡**（工厂/CMO 同构，3 轮 = MARK_STABLE_ROUNDS）：

```bash
# 工厂验收：同一 exec 容器跑 3 轮 → verify 全 PASS 打 accept；失败打 verify_fail
case_runner.py run --cases-dir cases/te --role factory --scene acceptance \
    --exec-id exec-20260903-verify --execute --authorized
case_runner.py mark --cases-dir cases/te --mark verify --set accept \
    --evidence runs/exec-20260903-verify --rounds 3
case_runner.py mark --case-file <失败用例.md> --set verify_fail --note "<失败原因>"

# CICD 验收（前提 mark=accept）：cicd_mark 无→cicd_verify（工厂提交）→ 3 轮 → cicd_accept
case_runner.py mark --cases-dir cases/te --mark accept --set cicd_verify \
    --evidence runs/exec-20260903-verify      # 引用验收容器，审计追溯
case_runner.py run --cases-dir cases/te --role cmo --scene acceptance \
    --exec-id exec-20260903-cicd --execute --authorized
case_runner.py mark --cases-dir cases/te --cicd-mark cicd_verify --set cicd_accept \
    --evidence runs/exec-20260903-cicd --rounds 3
```

### 4.5 标签多选与互斥

**写入面（用例 frontmatter，v0.7 双字段）**：

| 字段 | 可否多选 | 互斥规则 | 机器校验（M1） |
|---|---|---|---|
| `mark`（工厂验收状态） | **否，单值** | `debug/verify/verify_fail/accept/deprecated` 组内互斥（5 选 1） | parse_frontmatter 枚举校验 + 迁移矩阵 |
| `cicd_mark`（cicd 验收状态） | **否，单值** | （无）/`cicd_verify`/`cicd_verify_fail`/`cicd_accept` 组内互斥（3 选 1）；与 mark **可并存**（唯一不变量：cicd_mark ≠ 无 ⇒ mark=accept） | 枚举校验 + 不变式双点校验 |
| 作者元数据（创建人/创建时间/修改人/修改时间） | 否（各一值） | 无互斥；创建人/创建时间创建后不变，修改人/修改时间随内容修改刷新 | 归档推送前检查（§8.3） |
| `tags`（静态属性） | **是，多值** | 优先级 P0~P4 组内互斥（取一）为规范惯例；其余作者属性（协议族/功能域/CR 号等）并存 | M1 不做 tags 互斥校验（作者域，写入手工；机器化校验留后续 CR，挂 validate 层） |

- v0.6 起不再有 xstate / 功能标签 / 环境状态任何字段或约定（用户指令：均不建模）。
- **三个维度正交**：mark（工厂域）× cicd_mark（CICD 域）× tags/作者元数据（作者属性）互不占用；筛选时 `--mark` × `--cicd-mark` × `--tag`/`--keyword` 叠加圈池。

**圈选面（命令行）**：

| 参数 | 可否多选 | 组合规则 |
|---|---|---|
| `--mark` | **是**，逗号列表（in 语义，任一命中） | 圈工厂域状态；与 `--role`×`--scene` 组合受 §4.4 池子集约束（verify_fail/deprecated 被拒） |
| `--cicd-mark` | **是**，逗号列表（`none` = 未提交 CICD） | 圈 CICD 域状态；`--role cmo` 时默认按此域圈池；用例 mark≠accept 自动排除（子集语义） |
| `--tag` | **是**，逗号列表 | `--tag-mode and/or`（现状零变化） |
| `--keyword` | **是**，可多次指定 | 多关键字 AND（现状零变化） |
| mark 子命令 `--set` | **否**，单值 | 值域路由：`debug/verify/verify_fail/accept/deprecated` → mark 字段；`cicd_verify/cicd_verify_fail/cicd_accept` → cicd_mark 字段（校验不变式）；`--mark`/`--cicd-mark` 圈选待打标子集 |

## 5. 用例库 git 关联（STORY-056-01）

### 5.1 方案对比与决策

| 方案 | 描述 | 优劣 | 结论 |
|------|------|------|------|
| ~~A. cases 内嵌 git 仓库~~ | `cases/` 自身 git init + origin=ptm-cases.git，fetch 后对齐 origin/main | **实测推翻（STORY-056-01）**：远端 ptm-cases.git 根布局为 `README.md + cases/{tde,te} + docs/`（README 明确 `cases/te` 为 --cases-dir 指向层），本地 `ptm-te-manaul/cases` 根={tde,te} 缺 cases/ 一层 → 树无法零冲突对齐；加层（cases/cases）则污染工具路径 | 否决（用户 2026-09-03 拍板改 B） |
| **B. symlink 指向既有 clone（主选，用户已定）** | `ptm-te-manaul/cases` → symlink → `/home/hyde/projects/ptm-cases/cases`；git 仓库本体=ptm-cases clone（origin=ptm-cases.git，HEAD main 74073e6 已对齐，工作树 0 变更） | 优：工具路径 `--cases-dir cases/te`、规则 4 文案、台账/用例路径全部零改动；git 历史/远程天然就绪，授权后直接 push；内容与远端 cases/** 314 文件逐字节一致零迁移。劣：.git 在 symlink 上级（clone 根），`git -C cases` 向上解析生效但 status 显示全 clone 根状态；WSL symlink 兼容性已实测可用（git rev-parse/log/status/read 全通过，2026-09-03） | **主选（已落地 2026-09-03）** |
| C. 双区 rsync 同步 | manaul/cases 工作区 + ptm-cases 归档库手工同步 | 劣：双份漂移、不可追溯 | 否决 |

> 落地记录（2026-09-03，用户定案）：半初始化 .git 已清理，实体目录备份 /tmp/ptm-te-manaul-cases-bak-20260903；symlink 建立后 `git -C cases log` 显示 74073e6、status 0 变更。台账 path 前缀相应为**相对 clone 仓库根 `cases/te/...`**（修正 HLD §6 旧 `te/...` 假设）；init_cases_repo 幂等前置升级为向上发现仓库（repo-subdir，STORY-056-01 增量）。

### 5.2 init_cases_repo.py 设计（幂等初始化脚本）

调用形态：`python init_cases_repo.py --cases-dir <dir> --remote <url> [--branch main] [--json]`
真相源 `skills/case-execution/scripts/init_cases_repo.py`（与 Skill 同树，遵循"Skill 资产同树安装"规则），分发副本落 `ptm-te-manaul/scripts/`。

执行序（每步失败即终止并输出原因，不暴露 traceback）：

0. **向上发现仓库（v0.9 增量，symlink 场景）**：`git rev-parse --show-toplevel` 从 --cases-dir 向上查找——发现上级仓库（如 symlink 目标 ptm-cases/cases 而 .git 在 ptm-cases 根）且 `remote get-url origin` == 期望 URL → 输出 `already-initialized`（mode=repo-subdir）exit 0 零副作用；origin 不符 → abort；无仓库 → 继续步骤 1。
1. **幂等前置**：`cases/.git` 存在且 `remote get-url origin` == 期望值 → 输出 `already-initialized` exit 0，零副作用。
2. **工作区安全校验**：目标目录存在且含 `te/`；若已有 `.git` 但 origin 不符 → 报错要求人工确认，不自动改 remote。
3. **SSH 连通性探测**：`ssh -o BatchMode=yes -o ConnectTimeout=5 -o StrictHostKeyChecking=accept-new -T git@10.113.53.21`。**判定规则：GitLab 对 `-T` 正常返回 exit 1（无 shell），以 stdout/stderr 含 `Welcome to GitLab` 为成功判据，禁止用 exit code 判定**（Gotcha G-1）。
4. **连通路径（mode=linked）**：`git init` → `git remote add origin <url>` → `git fetch origin` → 对齐前先校验工作区与 `origin/main` 树一致（`git diff --stat origin/main` 为空）→ `git switch -c main origin/main`（工作区已与 74073e6 一致，直接接上远端历史，零 merge commit）；树不一致 → abort 提示人工处理，不硬切（防覆盖本地修改，Gotcha G-2）。
5. **降级路径（mode=local-only）**：SSH 不通 → `git init` + `git add -A && git commit -m "用例库本地初始化（待关联 ptm-cases.git）"`；输出提示：网络可达后重跑本脚本自动补 remote 对齐（幂等补齐分支：步骤 1 检测到 local-only 状态时进入步骤 3→4，本地有领先提交时 abort 提示用户决策 merge 或 push）。
6. **git 身份检查**：`user.name` / `user.email` 任一缺失 → WARNING 提示配置命令，不代配全局身份（用户属性），不阻塞 init。
7. **输出摘要**：`mode / branch / commit / remote / cases_count(te 下 .md 数)`；`--json` 时输出机器可读 JSON。

安全边界：脚本只做 init/fetch/switch/commit（降级路径），**永不 push、永不 force、永不改 remote**（除首次 remote add）。

### 5.3 mark 自动 commit（可选，默认开）

`case_runner mark` 成功后：`git add <case.md> cases/case_ledger.yaml` + `git commit -m "mark(case): <case_id> <old>→<new>"`；`--no-commit` 关闭；cases 非 git 仓库时降级跳过并 WARNING（指纹告警兜底，即原设计"未入库"行为）。push 永不自动（用户单独发起）。

### 5.4 gitlab 归档推送前检查技能（v0.8，用户指令 2026-09-03）

**技能归属**：case-execution skill 内新增脚本 `scripts/archive_check.py` + SKILL.md 新增"归档推送前检查"章节 + 规则 17（见 §8）；面向"把用例库 push/归档到 gitlab 前必须自检"场景。

**CLI 契约**：

```bash
python archive_check.py --cases-root <cases> [--author <操作人>] [--date YYYY-MM-DD]
    [--fix] [--json]
```

**执行逻辑**：

1. **前置**：`--cases-root` 必须是 git 仓库或其内子目录（git rev-parse 向上解析，symlink 场景自动成立；找不到仓库 → exit 2 提示先 init/关联）；
2. **取变更集**：`git -C <仓库根> status --porcelain` + `git diff HEAD --name-only`，**过滤到 `cases/` 前缀路径**（仓库根含 README/docs，只校验用例库），得到 `A`（新增）/ `M`（已跟踪且内容变化）文件集（`D` 删除仅提示，不校验）；
3. **A 类（新增文件）校验**：frontmatter `创建人/创建时间/修改人/修改时间` 四字段齐全且时间字段匹配 `^\d{4}-\d{2}-\d{2}$`（年月日）；`创建时间`==`修改时间`==`--date`（新文件同日创建/修改）；
4. **M 类（修改文件）校验**：读 HEAD 版本 frontmatter 与工作区版本对比——**内容有实质差异（指纹变化）的用例，要求 `修改人` 非空且已刷新、`修改时间` == `--date`**；HEAD 已含四字段且本次未改内容仅改 mark/cicd_mark（打标提交）→ 跳过修改人/时间校验（打标非内容修改）；
5. **--fix**：`--author` 必填时自动补齐/刷新缺失或过期的四字段（A 类补全 + M 类刷新修改人/时间为 `--author`/`--date`），写 frontmatter（缺行插入 frontmatter 块尾部）；
6. **输出**：逐文件问题清单（错误类别：`MISSING_CREATOR`/`MISSING_CREATED`/`MISSING_MODIFIER`/`MISSING_MODIFIED`/`STALE_MODIFIED`/`INVALID_DATE`/`NOT_GIT_REPO`），PASS/FAIL 汇总；有 FAIL 且无 `--fix` → exit 1（阻断推送）；`--json` 机器输出。

**失败路径**：非 git → exit 2；git 命令失败 → 该文件标 ERROR 不中断其余；`--fix` 写失败（只读权限等）→ 该文件 FAIL 并提示。

**与既有机制关系**：打标产生 commit（§5.3）不含四字段刷新需求；归档推送（人工执行 git push）前必须经本检查 PASS——push 本身仍不授权自动执行（延续 CR 不授权项），技能只做门禁与提示。规则 17 文案（§8）：归档推送前检查 + 修改用例必须刷新修改人/修改时间（时间格式 YYYY-MM-DD）。

## 6. tde/te 目录树与生成落位（STORY-056-02）

| 目录 | 语义 | 写入方 |
|------|------|--------|
| `cases/te/<协议族>/<功能域>/<子域>/<用例编号>.md` | 可执行原子用例（frontmatter 16 列 + tags + mark） | ptm-te（生成/转换落位面）；mark 仅经 `case_runner mark` |
| `cases/tde/*.md` | ptm-tde 设计源（`特性测试用例.md`，无 frontmatter 16 列） | ptm-tde 交付；ptm-te 侧只读 |

- 规则 4 文案修订：执行入口从 `cases/<协议族>/<功能域>/<子域>/` 改为 **`cases/te/<协议族>/<功能域>/<子域>/`**；`discover_cases` 递归 glob 机制不变（零代码改动），调用方统一传 `cases/te`。
- 防呆：`--cases-dir` 误指 cases 根时会扫到 tde 设计源（无 frontmatter → parse_frontmatter 判 INVALID → 过滤链跳过，天然安全），但规则/提示词/文档统一约定指向 `cases/te`，不依赖该兜底。
- 台账 `path` 字段相对 **clone 仓库根**（`cases/te/...` 前缀，v0.9 修正——远端布局仓库根含 cases/ 层），与 ptm-cases 入库路径一致，工厂/CICD 侧 checkout 后可直接定位。

## 7. case_runner / case_ledger 函数级设计

### 7.1 新模块 case_ledger.py（真相源 skills/case-execution/scripts/，委托模式同 exec_task.py）

```python
# v0.7 双字段：工厂域 mark + CICD 域 cicd_mark（组内单值互斥、组间并存，不变式 cicd_mark≠无⇒mark==accept）
MARK_VALUES = ("debug", "verify", "verify_fail", "accept", "deprecated")
CICD_MARK_VALUES = ("cicd_verify", "cicd_verify_fail", "cicd_accept")
META_FIELDS = ("创建人", "创建时间", "修改人", "修改时间")   # 作者元数据，YYYY-MM-DD（v0.8）
DATE_RE = r"^\d{4}-\d{2}-\d{2}$"                            # 年月日精度

# 工厂域迁移判定表（validate_transition 按目标值路由到工厂域/CICD 域）
TRANSITIONS = {
    "debug":       {"verify"},
    "verify":      {"verify_fail", "accept", "debug"},
    "verify_fail": {"debug", "verify"},            # 特性责任人整改后重提
    "accept":      {"debug"},                      # 回归/验收失败打回；连带 cicd_mark 清除
    "deprecated":  set(),                          # 用例级废弃吸收态，不可迁出
}
# CICD 域迁移判定表（前置：mark == accept，违反即拒绝）
CICD_TRANSITIONS = {
    None:                    {"cicd_verify"},
    "cicd_verify":           {"cicd_accept", "cicd_verify_fail"},
    "cicd_verify_fail":      {"cicd_verify"},      # CMO 重提（用例内容未改时）
    "cicd_accept":           set(),                # 已交付；内容缺陷整改走工厂域打回全链
}
# 通用边：mark 任意活动态→debug / →deprecated（--note 必填）；accept→debug 连带 cicd_mark=None；
# deprecated 迁出、cicd_mark 并存 deprecated 一律拒绝

ROLE_HINTS = {  # 责任角色标注（写入台账 mark_history.role 供审计；M1 不强制校验，DQ-056-10）
    ("debug", "verify"): "feature",       ("verify", "verify_fail"): "factory",
    ("verify", "accept"): "factory",      ("verify", "debug"): "factory",
    ("verify_fail", "debug"): "feature",  ("verify_fail", "verify"): "feature",
    ("accept", "debug"): "factory|feature",
    (None, "cicd_verify"): "factory",     ("cicd_verify", "cicd_accept"): "cmo",
    ("cicd_verify", "cicd_verify_fail"): "cmo", ("cicd_verify_fail", "cicd_verify"): "cmo",
}

MARK_STABLE_ROUNDS = 3   # DQ-056-02（定义在 EVIDENCE_REQUIRED 之前）

EVIDENCE_REQUIRED = {  # 验收类迁移证据门槛：(需 evidence, 需同容器连续 PASS 轮数或 None)
    ("verify", "accept"):           (True, MARK_STABLE_ROUNDS),  # 工厂验收 3 次通过
    (None, "cicd_verify"):          (True, None),                # 提交 CICD：引用验收容器作审计追溯
    ("cicd_verify", "cicd_accept"): (True, MARK_STABLE_ROUNDS),  # CICD 验收 3 次通过
}

ROLE_SCENE_POOLS = {     # §4.4 角色 × 场景圈池条件（--role 传入时启用池准入）
    # feature 圈 mark=debug；factory 圈 mark 域；cmo 圈 cicd_mark 域（并要求 mark==accept 自动成立）
    "feature": {"daily": {"mark": {"debug"}}},
    "factory": {"daily": {"mark": {"accept"}},
                "acceptance": {"mark": {"accept", "verify"}}},
    "cmo":     {"daily": {"cicd_mark": {"cicd_accept"}},
                "acceptance": {"cicd_mark": {"cicd_accept", "cicd_verify"}}},
}
ROLE_DEFAULT_SCENE = "daily"
ROLE_SCOPE = {"feature": "local", "factory": "factory", "cmo": "cicd"}

LEDGER_FILENAME = "case_ledger.yaml"

def validate_transition(cur_mark: str, cur_cicd: Optional[str], nxt: str) -> Optional[str]
    # nxt ∈ MARK_VALUES → 路由工厂域迁移；nxt ∈ CICD_MARK_VALUES → 路由 CICD 域迁移
    # （前置校验：cicd 域迁移要求 cur_mark == "accept" 且 cur_cicd 合法）
    # 合法返回 None；非法返回原因字符串（如 "cicd_mark 迁移需 mark=accept"）

def compute_case_fingerprint(md_text: str) -> str
    # re.DOTALL 定位 ^---\n(...)\n--- frontmatter 段；段内仅剔除 ^mark:.* 与 ^cicd_mark:.* 行；
    # 返回 sha256(处理后全文).hexdigest()；无 frontmatter 时对原文哈希
    # 作者元数据行（创建人/修改人等）不剔除——内容修改必然刷新它们，属变更证据

def load_ledger(cases_root: str) -> dict
    # cases_root/case_ledger.yaml 不存在 → 返回骨架 {metadata:{schema_version:'1.0'}, cases:{}}

def save_ledger(cases_root: str, ledger: dict) -> None
    # yaml.safe_dump(ensure_ascii=False, sort_keys=False)；刷新 updated_at；
    # 人工区 submitter/notes/open_issues 逐字段保留（调用方负责传入原值，本函数不新增不删除人工键）

def ledger_lock(cases_root: str) -> ContextManager
    # 文件锁 cases/.case_ledger.lock（fcntl），参照 exec_task._exclusive_lock 惯例（单写者）

def verify_case_state(md_text: str, case_id: str, ledger: dict) -> Tuple[bool, str]
    # 返回 (ok, reason)：md mark/cicd_mark vs 台账一致 + compute_case_fingerprint vs 台账 content_sha256 一致
    # 台账无该 case 条目 → ok=False reason="台账无登记"

def append_mark_history(entry: dict, *, mark: str, cicd_mark: Optional[str],
                        by: str, role: str = "", note: str = "",
                        evidence: str = "", rounds: Optional[int] = None) -> None
    # 追加 {at, mark, cicd_mark, by, role, note, evidence, rounds}；
    # 首次 debug→verify 时同步写 entry["first_submitted_at"]

def resolve_evidence_rounds(runs_dir: str, exec_id: str, case_id: str) -> Tuple[bool, str]
    # 复用 exec_task.scan_exec_runs + build_case_records（只读）；
    # 仅统计 --execute 的真实运行 run（dry-run 不计，LCQ-056-03-01 收紧）；
    # 返回 (满足与否, 描述)："连续 N 轮 PASS"按容器内该 case 最近连续 PASS run 数计

def cmd_mark(args) -> int      # mark 子命令入口（见 7.3）
def cmd_ledger(args) -> int    # ledger 子命令入口（见 7.4）
def cmd_archive_check(args) -> int  # 归档推送前检查入口（见 5.4，脚本 archive_check.py）
```

### 7.2 case_runner.py 改动（锚点按符号定位，不用行号）

| # | 锚点 | 改动 |
|---|------|------|
| C-1 | 常量区（`_FM_TAGS_KEYS` 附近） | 新增 `_FM_MARK_KEY = "mark"`、`_FM_CICD_MARK_KEY = "cicd_mark"`、`_MARK_VALUES`/`_CICD_MARK_VALUES`/`_META_FIELDS`（import 自 case_ledger） |
| C-2 | `parse_frontmatter`（358-471） | 新增 [3c] mark 枚举校验（工厂域：debug/verify/verify_fail/accept/deprecated）与 [3d] cicd_mark 枚举校验（cicd_verify/cicd_verify_fail/cicd_accept）+ 不变式校验（cicd_mark≠无 且 mark≠accept → error）；返回 dict 增加 `"mark"`（缺省 `"debug"`）与 `"cicd_mark"`（缺省 `None`）键 |
| C-3 | `filter_by_mark` / `filter_by_cicd_mark`（新函数，`filter_by_keyword` 之后） | `filter_by_mark(cases, marks)`：case.mark ∈ marks；`filter_by_cicd_mark(cases, marks)`：marks 含 `none` 时匹配 cicd_mark=None，否则 case.cicd_mark ∈ marks；空列表透传全部 |
| C-4 | run parser（3070-3144） | 增参：`--mark`（工厂域逗号列表）、`--cicd-mark`（CICD 域逗号列表，支持 `none`）、`--role {feature,factory,cmo}`、`--scene {daily,acceptance}`（默认 daily）、`--run-scope {local,factory,cicd}`（默认 local，--role 传入时按 ROLE_SCOPE 推导） |
| C-5 | run 过滤链（3321-3343 ST-EX-16 段后） | ① `--role` 场景池准入：`--role feature --scene acceptance` 非法组合；显式 `--mark`/`--cicd-mark` 必须满足 `ROLE_SCENE_POOLS[role][scene]` 圈池条件（cmo 用 cicd 域条件且自动排除 mark≠accept 用例；factory/feature 用 mark 域条件），越权（verify_fail/deprecated/cicd_verify_fail 等）exit 2 列出越权项；未传时按角色场景默认池。② 追加 `--mark`/`--cicd-mark` 过滤（链式叠加 --tag/--keyword，仅 --cases-dir 批量模式生效）；`--case-file` 指定时过滤参数 WARNING 忽略 |
| C-6 | run 用例执行循环（解析 frontmatter 后、execute_steps 前） | `--run-scope != local` 时逐用例 `verify_case_state`：失配 → 该用例不执行，step_results 记 ERROR `error_type=CASE_MARK_STALE`、reason"状态已失效（用例被修改），需重新走流程"；local 时仅 WARNING 放行。台账缺失时：factory/cicd 场景报 `LEDGER_MISSING` 阻塞，local 场景 WARNING |
| C-7 | result.json case 记录（generate_report 参数链） | 每条 case 增 `case_state: {mark, cicd_mark, fingerprint_ok}`（执行时点快照，审计口径同 runtime_authorization） |
| C-8 | main() 子命令注册（3147-3163 同层） | 新增 `mark` / `ledger` / `archive-check` 子命令 parser 并委托 case_ledger.cmd_mark / cmd_ledger / cmd_archive_check（无需启动校验/设备配置，置于 validate/exec-report 分支同层） |

### 7.3 mark 子命令契约（v0.7 双字段 + 值域路由）

```
case_runner.py mark (--case-file <path> | --cases-dir <dir>) --set <mark|cicd_mark 值>
    [--mark <工厂域圈选>] [--cicd-mark <CICD 域圈选>]
    [--note <原因>] [--evidence runs/<exec-id>] [--rounds N]
    [--operator <name>] [--role feature|factory|cmo]
    [--no-commit] [--cases-root <cases根目录，默认 case-file 上溯到 te/ 的父目录>]
```

- `--set` 值域路由（exact）：`debug/verify/verify_fail/accept/deprecated` → 工厂域 `mark`；`cicd_verify/cicd_verify_fail/cicd_accept` → CICD 域 `cicd_mark`（CICD 域动作校验前置：当前 mark == accept，否则拒绝）。
- 圈选：`--mark`/`--cicd-mark`（含 `none`）圈定待打标子集（缺省按 --cases-dir 全部）；`--case-file` 单文件时忽略圈选。
- `--role` 记录进台账 mark_history.role（对齐规范"维护责任人"列，审计用；M1 不强制校验，DQ-056-10）。
- `--set deprecated` 废弃动作（--note 必填；吸收态不可迁出，无专属责任人；连带清 cicd_mark）。
- mark/cicd_mark 目标值均不写作者元数据四字段（创建人/修改人/时间由归档检查 §5.4 维护）。

内部动作（逐用例独立判定，批量部分失败输出清单不整体回滚）：

1. 迁移合法性：读 md 当前 mark/cicd_mark + 台账当前值（台账为准，不一致先告警）→ `validate_transition`（双域路由 + CICD 域 mark=accept 前置 + 不变式）；非法 → 该用例失败项。
2. 回退（mark→debug）与废弃（mark→deprecated）必须 `--note`；`debug→verify` 提交动作建议 `--note`（缺失仅 WARNING）。
3. 验收类打标证据（EVIDENCE_REQUIRED）：`verify→accept` 要求 `--evidence`（验收场景容器）且 `resolve_evidence_rounds` ≥ 3（MARK_STABLE_ROUNDS=3）；`(无)→cicd_verify` 要求 `--evidence`（引用验收容器，无轮数门槛）；`cicd_verify→cicd_accept` 同 verify→accept（3 次验收通过）。不达标 → 拒绝该项。
4. 指纹计算（剔除 mark/cicd_mark 行）→ 写 md frontmatter 目标行（mark 或 cicd_mark，无则 frontmatter 块尾部插入；cicd_mark=none 删除该行）→ 追加台账 mark_history（含 by/role/新旧双域值）+ 更新 content_sha256/mark/cicd_mark + 首次 `debug→verify` 写 first_submitted_at + 刷新 updated_at（全程 ledger_lock）。
5. git 自动 commit（§5.3，除非 --no-commit 或非 git 仓库）。
6. 输出摘要表：成功/失败逐条（case_id、旧→新（双域）、失败原因）。

### 7.4 ledger 子命令契约

```
case_runner.py ledger --cases-root <dir> [--mark <逗号列表>] [--out LEDGER.md]
```

只读聚合：各状态计数表（mark 5 态 × cicd_mark 4 档交叉计数 + deprecated 计数）、待办清单（"accept 未提交 CICD"、accept 且稳定轮达标待提交 CICD、verify_fail/cicd_verify_fail 待整改）、指纹失配漂移清单、submit_time 摘要（first_submitted_at 派生自 mark_history，DQ-056-09）；`--out` 时生成 Markdown（机器生成区 marker 重写 + 人工区原样保留，复用 exec-report splice 惯例）。

## 8. 规则与文档同步面（STORY-056-02 / 07）

| 对象 | 改动 |
|------|------|
| install.py `RULE_BLOCK_RENDERERS["ptm-te-workflow"]` | 规则 4 文案（cases/te 路径 + tde 只读）+ 新增规则 14（冻结态禁改，需改先打回 debug）/ 15（mark 唯一入口，台账人工区原样保留）/ 16（验收打标必须 exec 证据，轮数 N 默认 3）/ 17（**归档推送前检查：修改用例必须刷新修改人/修改时间（YYYY-MM-DD），push 前运行 archive-check 门禁**）；版本 1.0.0→1.1.0；安装刷新 ptm-te/CLAUDE.md 与 ptm-te-manaul/{CLAUDE.md,AGENTS.md} |
| agents/ptm-te.md（canonical + 2 平台副本） | 用例入口路径改 `cases/te/...`；新增 mark/准入操作指引一段；按既有同步方式刷新 managed 时间戳 |
| SKILL.md | 用例结构化约定（§ST-EX-15）补 `mark`/`cicd_mark` 字段行（R-F-027：可选，枚举，缺省 debug/无）+ 作者元数据四字段（R-F-028：创建人/创建时间/修改人/修改时间，YYYY-MM-DD）；新增 ST-EX-18（mark/cicd_mark 过滤与运行准入）、ST-EX-19（mark/ledger 子命令与台账）、ST-EX-20（归档推送前检查 archive-check + 元数据字段维护）；Gotchas 补 2 条（§12 G-1/G-6） |
| docs/ptm-te/执行指导.md（白名单交付文档） | 三场景操作章节：local/factory/cicd 典型命令序列 + mark 流转操作卡 |

## 9. 集成契约（评审规则 3）

### 9.1 case_runner ↔ case_ledger（模块内）

- 方向/时机：case_runner main() 在子命令分发层委托 cmd_mark/cmd_ledger（解析 argparse 后、启动校验前）；run 主流程在用例循环内调用 verify_case_state/compute_case_fingerprint。
- 输入：args namespace / md 文本 / ledger dict。输出：int 退出码（0 成功 1 部分失败 2 参数错误）/ 校验二元组。
- 降级：case_ledger.py import 失败 → mark/ledger 子命令不可用（print stderr exit 5，同 exec_task 委托先例）；run 冻结校验在 ledger 缺失时按 C-6 的 LEDGER_MISSING 路径。

### 9.2 init_cases_repo ↔ 用例库 / GitLab

- 方向/时机：用户初始化工作区时手动执行一次；幂等可重跑。
- 输入：--cases-dir/--remote/--branch。输出：mode 摘要（文本或 --json）。
- 失败路径：SSH 不通 → local-only 降级（不中断）；树不一致 → abort（人工处理）；remote 已存在且不符 → abort。
- 调用方同步修改范围：无（新工具，无既有调用方）。

### 9.3 台账对外消费契约（工厂 / CICD，只读）

- 消费方：ptm-tde 工厂验收系统、CICD 流水线（各自 CR 承接实现，本 CR 只固化 schema）。
- 接口：`cases/case_ledger.yaml`（git 仓库内）+ `case_runner ledger --json`（后续可加）。
- 消费语义：只认 `mark` 池 + `content_sha256`，不需要理解用例内容；工厂取 `factory` 池，CICD 取 `cicd`/`cicd-passed` 池；checkout 指定 commit 即得冻结副本。

## 10. 失败路径与回退（评审规则 5）

| 阶段 | 失败 | 行为 |
|------|------|------|
| init | SSH 不通 | 降级 local-only（git init + 本地 commit），提示重跑补对齐 |
| init | 工作区与 origin/main 树不一致 | abort，人工确认（不覆盖） |
| init | git 身份缺失 | WARNING 继续（后续 mark commit 前再校验） |
| mark | 迁移非法 / 证据不足 / 指纹失配 | 该用例记失败项，其余用例继续（批量不整体回滚） |
| mark | 台账写失败 | md 已写则告警并输出回滚提示（人工 `mark --set <旧值>` 回退），不自动改 md |
| run 冻结校验 | 台账缺失 | factory/cicd：`LEDGER_MISSING` 阻塞；local：WARNING |
| 整体回退 | CR 交付缺陷 | git revert 工厂仓库提交；用例库侧 mark 历史为追加式，回退 = mark --set draft --note "回退 CR-056"，台账链完整保留 |

## 11. 待人工决策项（Decision Queue）

| 决策 ID | 类型 | 待确认问题 | 推荐方案 | 备选 | 影响/回退 |
|---------|------|-----------|---------|------|----------|
| DQ-056-01 | scope | 双字段状态模型（工厂域 mark 5 态 + CICD 域 cicd_mark 4 档，组间并存、CICD⊆工厂子集不变式）是否采纳 | **采纳**（用户 v0.7 概念纠正已定：工厂全量/CICD 子集须可并存；词汇对齐规范表） | v0.1~0.6 单值 8 选 1 状态机（被用户否决：进 CICD 的用例脱离工厂池） | 字段已按 v0.7 拆分，无兼容负担 |
| DQ-056-02 | implementation | "3 次验收通过"轮数 N（verify→accept 与 cicd_verify→cicd_accept 的打标门槛） | **3 轮连续同容器 PASS**（常量 MARK_STABLE_ROUNDS=3，后续可配置化） | 2（宽松）/5（严格） | 改常量即可，无迁移成本 |
| DQ-056-03 | implementation | 指纹口径 | **剔除 mark 行后全文 sha256**（预期/组网约束变更可检） | 仅 case_steps 哈希（过窄，漏检 frontmatter 变更，不推荐） | 口径变更需全量重算台账指纹（一次性 CR） |
| DQ-056-04 | architecture | 用例库 git 关联方式 | **方案 B symlink 指向既有 clone**（§5.1，用户 2026-09-03 定案；方案 A 内嵌 git 被实测推翻——远端根含 cases/ 层） | C rsync（否决） | 已落地：symlink 建立、实体备份 /tmp/…-bak-20260903、台账前缀改 cases/te |
| DQ-056-05 | implementation | mark 成功后自动 git commit | **默认开，--no-commit 可关；永不自动 push** | 关闭（指纹告警兜底，可追溯性降级） | 行为开关，无回退成本 |
| DQ-056-06 | runtime_authorization | CICD 无人值守 vs DQ-01 单次授权冲突 | **M3 前拍板**：预授权凭据（env 凭据 + 设备白名单双约束）或纯设备白名单 | 每轮人工授权（不可无人值守） | 独立 CR，不影响 M1 |
| DQ-056-07 | risk_acceptance | verified（accept）用例缺陷修复是否重走全链 | **严格重走全链**（修复提交=新 content_sha256 自动强制） | 仅重走工厂段（宽松，CICD 惯例不符） | 打回 debug 后自然强制 |
| DQ-056-09 | implementation | submit_time（首次提交/最后修改时间）载体 | **已由用户 v0.8 指令定案作废台账派生方案** → frontmatter 作者元数据字段（创建人/创建时间/修改人/修改时间，YYYY-MM-DD）承载；mark_history 不再派生 submit_time 输出（保留 first_submitted_at 于台账供审计） | —（用户已拍板） | 无 |
| DQ-056-10 | implementation | 维护责任人（规范表角色列）处理 | **--role 记录台账审计 + ROLE_HINTS 标注，M1 不强制校验**（自报角色无认证价值） | 强制校验（--role 不匹配迁移表拒绝；无身份系统支撑，形式大于实效） | 后续可升级为强制，无需 schema 变更 |
| DQ-056-11 | implementation | 作者元数据字段维护责任人 | **创建人/创建时间由用例创建流程（作者/生成器）写入；修改人/修改时间由每次内容修改者刷新 + 归档检查 --fix 兜底**；mark 子命令不写四字段 | 由 mark 命令统一代写（无法获知操作人姓名，需 --operator 且与内容修改流程脱节，不推荐） | 无 schema 变更风险 |

> DQ-056-08（xfail/skip 异常修饰建模）已随 v0.6 取消——用户指令"xstate 不需要了"，异常标签不建模。
> DQ-056-09 已随 v0.8 由用户直接定案（frontmatter 四字段），不再待决。
> DQ-056-01~05、07、10、11 用户回复 `approve` 即按推荐方案执行；DQ-056-06 不阻塞 M1，M3 前单独决策。

## 12. Gotchas

- **G-1**：GitLab `ssh -T` 正常返回 exit 1（无 shell 分配），必须按输出含 `Welcome to GitLab` 判定连通，用 exit code 判定会永远判"不连通"。
- **G-2**：`git switch origin/main` 前必须校验工作树与目标一致，否则静默覆盖本地未提交修改——这是不可逆损失。
- **G-3**：`tde/` 设计源 md 无 frontmatter，`parse_frontmatter` 判 INVALID 被过滤链跳过——但不得因此把 `--cases-dir` 默认指向 cases 根，规则/文档统一 `cases/te`。
- **G-4**：指纹剔除 mark/cicd_mark 行必须限定在 frontmatter 块内（`^---\n...\n---` 段内 `^mark:`/`^cicd_mark:` 行），正文出现同名键（如步骤描述）不得剔除，否则指纹漂移；作者元数据行（创建人/修改人/时间）不剔除——内容修改必然刷新它们，属变更证据。
- **G-5**：台账并发写需文件锁（ledger_lock，参照 exec_task._exclusive_lock）；同一 exec 容器并发写已有单写者约定，台账沿用。
- **G-6**：冻结场景要区分两种非执行结果：`INVALID`（frontmatter 校验失败，过滤链跳过，ST-EX-15 既有语义）与 `CASE_MARK_STALE`（指纹/状态失配，准入阻塞）——result.json 中 error_type 必须可区分，否则审计无法归因。
- **G-7**：`yaml.safe_dump` 必须 `allow_unicode=True`（PyYAML 无 ensure_ascii 参数，实测 TypeError；与 exec_task `_dump_task_yaml` 先例一致），否则台账中文（note/notes）转义为 `\uXXXX`，人工区不可读。
- **G-8**：`verify_fail` / `cicd_verify_fail` / `deprecated` 不在典型运行池内，`--mark` 显式枚举它们时允许选中但 `deprecated` 用例仍不执行（吸收态 FORBIDDEN）；`--mark accept,cicd_accept` 是工厂+CMO 回归池的标准显式表达（min 模式已移除，池交集用显式枚举）。
- **G-9**（已随 v0.6 移除——xstate 不建模）。
- **G-10**：`--set deprecated` 是吸收态废弃动作（--note 必填、无专属责任人、永不运行、不可迁出）；误废弃无自动恢复路径，须人工手工处理（ledger 漂移清单与 mark_history 留痕辅助审计定位），机器不提供 deprecated→* 回退边（吸收态语义，与 TRANSITIONS 一致）。
- **G-11**：`--role/--scene` 与 `--run-scope` 是两个正交维度（role+scene 管池准入、scope 管冻结严格度），显式传 `--run-scope` 可覆盖角色推导值（如 `--role factory --scene acceptance --run-scope local` 允许，但池校验仍按 acceptance 场景池）；不得把二者合并成一个参数，否则"工厂责任人本地复跑"类场景被误杀。
- **G-12**：`--mark`（工厂域）/`--cicd-mark`（CICD 域）是圈选面（多选并集）、frontmatter mark/cicd_mark 字段是写入面（各自单值）——文档与错误提示必须区分两个语境，避免"mark 支持多选"被误解为用例可打多个状态。
- **G-13**：CICD ⊆ 工厂不变式（cicd_mark ≠ 无 ⇒ mark=accept）必须双点校验——parse_frontmatter（读）与 validate_transition（写）；只校验一侧会漏（如直接手改 frontmatter 绕过 mark 命令）。
- **G-14**：作者元数据时间精度为年月日（YYYY-MM-DD），刷新判等以日期为准；勿在用例 frontmatter 引入秒级时间戳（与指纹/台账时间口径冲突，且与用户精度约定不符）。
- **G-15**：归档检查 M 类判定用"内容实质差异（指纹变化）"，mark/cicd_mark 打标提交（仅状态行变化）不要求刷新修改人/时间——否则每次验收打标都被归档检查拦截。

## 13. 验收准则（对应 §1.2 目标）

| G | 验收项 | 度量 |
|---|--------|------|
| G-01 | 312 条 te 用例 parse 全过、mark 缺省 debug | 冒烟输出 0 error；`--mark debug` 命中 312 |
| G-02 | 指纹卡口 | 打标前后指纹不变 ≥1 实测；修改 case_steps 后 factory/cicd 100% CASE_MARK_STALE、local 0 阻塞 |
| G-03 | 迁移矩阵 | 工厂域非法迁移（debug→accept、verify_fail→accept）与 CICD 域非法迁移（非 accept 设置 cicd_mark、deprecated 并存 cicd_mark）单测 100% 拒绝；verify→accept / cicd_verify→cicd_accept 轮数不足 3 轮 100% 拒绝 |
| G-04 | git 关联 | 连通路径对齐 74073e6（git log 确认）；断网降级 local-only；幂等二跑 exit 0 零副作用；mark 产生 commit |
| G-05 | 目录语义 | 台账 path 100% `cases/te/` 前缀（相对 clone 仓库根）；tde 内容零 diff；symlink 链路（--cases-dir 访问 + git -C 向上解析）实测可用 |
| G-06 | deprecated 行为 | `--set deprecated` 记台账 mark_history 且 --note 缺失时拒绝；deprecated 用例被 --mark 显式选中仍不执行（吸收态）；`--set deprecated` 后任何迁出（deprecated→*）100% 拒绝 |
| G-07 | 角色场景池 | `--role factory`（缺省 daily）默认池={mark=accept}；`--role factory --scene acceptance` 默认池={mark ∈ accept,verify}；`--role cmo --cicd-mark cicd_verify` 圈 CICD 验收池且自动排除 mark≠accept；`--mark verify_fail` 在 --role factory 下 100% exit 2；`--role feature --scene acceptance` 报参数错误；无 --role 行为与现状零差异 |
| G-08 | 双字段不变式 | cicd_mark≠无 的用例 mark 100% == accept（parse + validate 双点校验单测）；accept→debug 后 cicd_mark 连带清除 100% 生效；factory·daily 全量覆盖含 cicd_accept 用例（--mark accept 命中数 == accept 总数，不因进 CICD 减少） |
| G-09 | 作者元数据 | A 类文件四字段（创建人/创建时间/修改人/修改时间）缺失 100% 报错；M 类内容修改未刷新修改时间 100% 报 STALE_MODIFIED；仅打标提交不要求刷新；--fix --author 补齐后重跑 PASS；非法日期格式 100% 报 INVALID_DATE |
| 全局 | 回归 | 既有测试全过；4 副本脚本 / 3 副本规则文件 / 3 副本 agent md 一致性校验通过；新增 pytest ≥ 30 个断言用例 |

## 14. 开放问题状态表

| 编号 | 问题 | 状态 |
|------|------|------|
| O-056-01 | 原设计 6 开放点（§九） | RESOLVED→收编 DQ-056-01~07（2026-09-03，HLD §11） |
| O-056-02 | case-lifecycle-tracking-design.md 草稿处置 | RESOLVED：结论已收编本 HLD，原文件保持原位不动（2026-09-03） |
| O-056-03 | CICD 无人值守授权形态 | OPEN→DQ-056-06（M3 前拍板，不阻塞 M1） |
| O-056-04 | /home/hyde/projects/ptm-cases 既有 clone 处置 | RESOLVED（2026-09-03 v0.9）：clone 升格为工作仓库（symlink 目标），origin 已 gitlab、HEAD 74073e6 对齐；不再有"保留/删除"取舍 |
| O-056-05 | 功能标签（特性名称）与 accept 联动 | RESOLVED（2026-09-03 v0.6）：用户指令功能标签不建模，无联动要求；tags 机制原样保留，`mark --set accept` 不涉及任何 tags 写入 |
| O-056-06 | 存量 312 条 te 用例作者元数据四字段回填 | OPEN：archive-check 不强制存量未动文件补齐（避免一次性全量回填）；批量回填（按 git 历史推导创建人/创建时间）为独立后续 CR 候选，见 CR 台账 |
