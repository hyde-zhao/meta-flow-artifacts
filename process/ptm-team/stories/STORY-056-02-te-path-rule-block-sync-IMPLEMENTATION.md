---
story_id: STORY-056-02
story_slug: te-path-rule-block-sync
cr_id: CR-056
wave: 1
design_evidence_type: technical-note
lld_ref: process/HLD-CR-056.md（§4.3/§4.4/§6/§8/§12，v0.8）+ process/changes/CR-056.md（IMP-056-06/07/08）
implementation_ref: process/stories/STORY-056-02-te-path-rule-block-sync-IMPLEMENTATION.md
executed_by: meta-dev
executed_at: 2026-09-03
---

# STORY-056-02 实现执行证据 — tde/te 路径约定固化（规则块 v1.1.0 + agent 三副本）

## 实现前置检查

| 条目 | 结果 | 证据 |
|---|---|---|
| 设计证据已确认（technical-note：design-already-in-hld） | PASS | `process/HLD-CR-056.md` v0.8 §6/§8/§12 逐条列出规则文案与同步面；`process/DEVELOPMENT-PLAN-CR-056.yaml` STORY-056-02 `lld_policy.rationale` 明示；host-orchestrator 以 HLD §8 为设计证据派发实现 |
| 依赖门控（depends_on=[]，dependency_type=none） | PASS | 同 Wave 仅 STORY-056-01（文件所有权 `skills/case-execution/scripts/init_cases_repo.py`，与本 Story 零交集），并行安全 |
| 文件所有权无冲突 | PASS | primary=`script/ptm_team/install.py` + `agents/ptm-te.md`；forbidden=`skills/case-execution/scripts/case_runner.py`（零触碰）；SKILL.md 按 host-orchestrator 指令仅做路径文字级同步（深度收口归 STORY-056-07） |
| AI 可执行任务清单存在 | PASS | host-orchestrator 派发任务 1~5（install.py 渲染器 / agent 三副本 / 安装副本刷新 / 一致性校验 / SKILL.md 路径同步） |
| 平台目标明确 | PASS | claude（CLAUDE.md + .claude/agents）与 codex（AGENTS.md）；平台路径以 `RULE_FILES` registry 为真相源，未用目录类比推断 |
| 前置事实核对 | PASS | 安装副本现状实测（三份规则文件 + 两份 .claude agent 副本 + manaul .codex/agents/ptm-te.toml），副本 frontmatter 与 `render_claude_agent` 输出逐字节同构，确认既有刷新走 installer 渲染函数路径 |

## 实现对象清单

| # | 对象 | 文件 | 变更 |
|---|---|---|---|
| 1 | 规则块独立版本 registry | `script/ptm_team/install.py` | 新增 `RULE_BLOCK_VERSIONS`（ptm-te-workflow=1.1.0，ptm-tde-workflow=1.0.0，未登记回退 `MANAGED_VERSION`）；`managed_block_markers` 改按 block_id 取版本 |
| 2 | `render_ptm_te_rule_body` 规则 4 修订 | 同上 | 执行入口 `cases/<协议族>/...` → `cases/te/<协议族>/<功能域>/<子域>/<用例编号>.md`（`--cases-dir` 递归 glob 指向 `cases/te`）+ `cases/tde/` ptm-tde 设计源只读；`cases/upload/` 废弃注记保留 |
| 3 | `render_ptm_te_rule_body` 新增规则 14~17 | 同上 | 14 冻结用例禁改 / 15 mark 唯一入口 + 台账人工区保留 + CICD⊆工厂不变式 / 16 验收打标 exec 容器证据 + 连续 3 轮 PASS / 17 作者元数据四字段 + archive_check 归档推送前检查 FAIL 阻断 push |
| 4 | renderer docstring | 同上 | 补 v1.1.0 变更要点（CR-056） |
| 5 | canonical agent 路径同步 | `agents/ptm-te.md` | 9 处 `cases/<协议族>` → `cases/te/...`（角色表上游/执行入口行、编排图、[1] 输入、[1] 异常路径、PC 消费契约输入路径、运行时目录树、目录表、`ppdcs/delivery/` 行、异常路径汇总）+ 目录树/目录表补 `cases/tde/` 只读行 |
| 6 | canonical agent 新增小节 | 同上 | `### 用例状态 mark 与运行准入（CR-056 简述）`：mark/cicd_mark 双字段值域与不变式 + 三角色×场景圈池表（5 行）+ 冻结禁改/mark 唯一入口/3 轮 PASS 证据/作者元数据与归档检查一句话约定 |
| 7 | canonical agent 修订记录 | 同上 | 追加 v2.6 行（2026-09-03，meta-dev，CR-056 STORY-056-02） |
| 8 | 安装副本刷新（规则文件 ×3） | `ptm-te/CLAUDE.md`、`ptm-te-manaul/CLAUDE.md`、`ptm-te-manaul/AGENTS.md` | 经 `RULE_BLOCK_RENDERERS` + `upsert_managed_rule_block` 刷新 ptm-te-workflow 块至 v=1.1.0（commit=ad89670，generated=2026-09-03T11:24:08Z） |
| 9 | 安装副本刷新（agent 副本 ×2） | `ptm-te/.claude/agents/ptm-te.md`、`ptm-te-manaul/.claude/agents/ptm-te.md` | 保留副本 frontmatter（逐字节不变），正文整体替换为 canonical 正文，managed 行刷新（version=1.0.0 / canonical-commit=ad89670 / generated=2026-09-03T11:24:08Z），与 `render_claude_agent` 布局同构 |
| 10 | SKILL.md 路径文字级同步 | `skills/case-execution/SKILL.md` | 「旧格式 -> 新格式转换规则」表头新格式列 `cases/<协议族>/.../` → `cases/te/<协议族>/.../`（CR-056 路径约定 + tde 设计源只读注记）；仅此 1 处，深度收口归 STORY-056-07 |

## 设计契约映射

| 设计契约（来源） | 实现落点 | 验证入口 |
|---|---|---|
| HLD §6 规则 4 文案：入口 `cases/te/<协议族>/<功能域>/<子域>/`、tde 设计源只读、`discover_cases` 递归 glob 机制零代码改动 | install.py 渲染器规则 4 + canonical agent 9 处路径 + SKILL.md 表头 | 本文件「验证结果」第 1/3 项 + `tests/test_install_mapping.py` |
| HLD §4.1 冻结态（verify/verify_fail/accept/cicd_verify/cicd_verify_fail/cicd_accept 冻结） | 规则 14 文案（值域与 HLD §4.1 逐字一致，deprecated 不在冻结集——吸收态语义另列） | 渲染输出断言（规则 14 全文比对） |
| HLD §4.1/§7.3 mark 唯一写入路径（`--set` 值域路由）+ §9.3 台账人工区保留 + CICD⊆工厂不变式（G-13 双点校验） | 规则 15 文案 | 渲染输出断言（规则 15 全文比对） |
| HLD §4.1 迁移附加条件（verify→accept、cicd_verify→cicd_accept 需 --evidence + 同容器连续 3 轮 PASS，MARK_STABLE_ROUNDS=3） | 规则 16 文案 | 渲染输出断言（规则 16 全文比对） |
| HLD §4.1a 作者元数据四字段（YYYY-MM-DD）+ §5.4 archive_check FAIL 阻断 push + G-15（仅打标提交不刷新） | 规则 17 文案 | 渲染输出断言（规则 17 全文比对） |
| HLD §4.3/§4.4 三角色场景池（feature/factory/cmo × daily/acceptance）+ 不运行池 | canonical agent「用例状态 mark 与运行准入」小节表格 | agent 正文一致性 diff + 小节内容比对 |
| HLD §8 agent 同步面（canonical + 2 平台副本，既有同步方式） | 对象 9（保留 frontmatter + 正文整体替换 + managed 行刷新） | 本文件「验证结果」第 3/5 项 |
| HLD §8 版本 1.0.0→1.1.0（managed 块 header 同步） | `RULE_BLOCK_VERSIONS` + `managed_block_markers` | 本文件「验证结果」第 2 项 + ptm-tde 块 v=1.0.0 不变断言 |

## 单元测试与 Fixture 计划

- 既有载体：`tests/test_install_mapping.py`（`PtmTeRuleBlockTests`：registry 键、skill 列表、`render_ptm_te_rule_body` 关键规则断言、`RULE_FILES` 共用映射）——本 Story 改动后回归。
- 新增 fixture：无新测试文件。本 Story 是规则文案 + 提示词 + 安装副本同步（无运行时逻辑新增），验证以「渲染输出逐条断言 + 三副本逐字节 diff」承担（见「验证结果」）；规则 14~17 的机器行为（迁移矩阵/证据轮数/归档检查）由 STORY-056-03~06、08 的 case_ledger/case_runner/archive_check 单测承接，STORY-056-07 收口回归。
- 渲染输出断言脚本（一次性，uv run 内联，见 DEV-LOG）：规则 ID 列表 == 1..17、规则 4/14/15/16/17 全文比对、`managed_block_markers` v= 值（te=1.1.0 / tde=1.0.0）。

## 最小实现切片

| 切片 | 内容 | 状态 |
|---|---|---|
| S1 | install.py：`RULE_BLOCK_VERSIONS` + `managed_block_markers` 按块取版本 + 渲染器规则 4 修订 + 新增 14~17 + docstring | 完成 |
| S2 | canonical `agents/ptm-te.md`：路径 9 处 + tde 只读 + mark 准入小节 + 修订记录 v2.6 | 完成 |
| S3 | 安装副本刷新：3 份规则文件（installer 渲染函数路径）+ 2 份 agent 副本（保留 frontmatter） | 完成 |
| S4 | `skills/case-execution/SKILL.md` 路径文字级同步（1 处表头） | 完成 |
| S5 | 一致性校验（三副本规则块 / 三副本 agent 正文 / 托管块外内容 / 连带影响面） | 完成 |

## 平台差异处理

| 平台 | 规则文件 | agent 副本 | 差异点 |
|---|---|---|---|
| claude | `CLAUDE.md`（平台标签句「Claude Code」） | `.claude/agents/ptm-te.md` | 仅平台标签句差异，块正文其余逐字节一致 |
| codex | `AGENTS.md`（平台标签句「Codex / Qoder」，`RULE_FILES` 共用映射） | `.codex/agents/ptm-te.toml`（**本 Story 未触碰**，见「未覆盖项」） | 同上 |

平台路径判定全部经 `RULE_FILES` / `PLATFORM_DIRS` registry，无目录类比推断；未新增平台特有字段。

## 验证结果

1. **渲染器输出断言**（`uv run --python 3.11` 内联脚本）：`render_ptm_te_rule_body` 三平台规则 ID 列表 == 1..17；规则 4/14/15/16/17 与 HLD §8 契约逐字一致；`managed_block_markers("ptm-te-workflow")` → `v=1.1.0`，`("ptm-tde-workflow")` → `v=1.0.0`。PASS
2. **既有回归**：`uv run --python 3.11 pytest tests/test_install_mapping.py -q` → **10 passed**。PASS
3. **三份规则文件副本一致性**：`ptm-te/CLAUDE.md` ≡ `ptm-te-manaul/CLAUDE.md`（ptm-te-workflow 块剔除 header 后逐字节一致，True）；`manaul/CLAUDE.md` vs `manaul/AGENTS.md` 仅平台标签句差异（归一化后一致，True）；三份 header 均 `v=1.1.0 commit=ad89670`。PASS
4. **三份 agent md 正文一致性**：canonical 正文 ≡ `ptm-te/.claude/agents/ptm-te.md` 正文 ≡ `ptm-te-manaul/.claude/agents/ptm-te.md` 正文（剔除 frontmatter 与 managed 行后逐字节一致，均 True）；两副本 frontmatter 与刷新前逐字节相同（diff 0 行）；managed 行已刷新（canonical-commit=ad89670 / generated=2026-09-03T11:24:08Z）。PASS
5. **无连带影响面**：两外部项目 19:20 后被改动文件清单 == 本 Story 5 个目标文件（find -newermt 实测）；三份规则文件托管块外内容不变（managed pattern 剔除后对比 True）；`ptm-tde-workflow` 块正文不变（True）。PASS
6. **所有权遵守**：`skills/case-execution/scripts/case_runner.py` 零触碰；REQUIREMENTS / HLD / ARCHITECTURE-DECISION / MVP-SCOPE / BLUEPRINT 零触碰。PASS

## 未覆盖项

| # | 项 | 归属 |
|---|---|---|
| 1 | `ptm-te-manaul/.codex/agents/ptm-te.toml`（codex 平台 agent 副本）未刷新——不在本 Story「agent 三副本」声明同步面（HLD §8 只列 canonical + 2 平台副本），且非 `DEVELOPMENT-PLAN-CR-056.yaml` STORY-056-02/07 `output_files`；其正文仍为 v2.5（2026-09-03T01:09:38Z 刷新）。建议 host-orchestrator 在 STORY-056-07 副本同步清单中显式纳入或记录 waived，否则 manaul 内 codex 入口与 claude 入口提示词不一致 | host-orchestrator 决策 / STORY-056-07 |
| 2 | 规则 14~17 的机器行为（迁移矩阵、3 轮证据、归档检查）未在本 Story 实测——本 Story 只固化规则文案 | STORY-056-03~06、08（实现）+ 07（回归） |
| 3 | SKILL.md ST-EX-18/19/20、R-F-027/028 深度收口、`docs/ptm-te/执行指导.md` 三场景章节、4 副本脚本同步 | STORY-056-07 |
| 4 | `cases/te` 路径约定的运行时行为验证（`--cases-dir cases/te` 递归 glob 命中 312 条）| STORY-056-07（G-01 冒烟） |
| 5 | `process/state/GATE-LEDGER.ndjson` 无 CR-056 gate 事件记录（CP5 确认事件未见落账） | host-orchestrator 账务补记 |

## 设计缺口反馈

- 无设计缺口。HLD §8 表逐条可落地，规则 14~17 文案值域与 HLD §4.1 完全一致，未发现冲突或缺失字段。
- 一处事实修正类反馈：HLD §8 写「安装刷新 ptm-te/CLAUDE.md 与 ptm-te-manaul/{CLAUDE.md,AGENTS.md}」未提 `.codex/agents/ptm-te.toml`，与 manaul 实际安装足迹（codex 平台记录）存在覆盖面缺口 → 已按「未覆盖项 #1」升级为决策项，不擅自扩大范围。

## 实现灰区与取舍记录

| 灰区 | 处置 | 依据 | 重访条件 |
|---|---|---|---|
| 刷新方式：全量 `install claude --agent ptm-te`（CLI 唯一入口）会连带重拷 7 个 skill + 资源到两外部项目，超出本 Story 所有权并引入 Wave-2 WIP 拷贝风险 | 采用 installer 渲染函数直调（`RULE_BLOCK_RENDERERS` + `upsert_managed_rule_block` + `render_claude_agent` 布局），与既有刷新痕迹一致（manaul manifest `installed_at=2026-09-02T02:38:59Z` ≠ 块 `generated=2026-09-03T01:09:45Z`，证明上次刷新即函数级） | 任务允许「记录受阻点后退回手工同步」；函数直调比手写更保证与 CLI 逐字节一致 | 若后续提供规则块-only CLI 子命令，改走 CLI |
| 灰区问询：并行 LLD 期未直接问用户 | 未产生 `blocks_lld=true` 灰区；`.codex/agents/ptm-te.toml` 归属问题以「未覆盖项 + 设计缺口反馈」上报 host-orchestrator，不阻塞本 Story | meta-dev clarification queue 协议 | host-orchestrator 决策后回写 |

## 偏离记录

1. `agents/ptm-te.md` 修订记录 v1.6 历史行中的 `cases/<协议族>/.../` 字样**有意保留**（修订记录是不可改写的历史基线，方案评审规则第 10 条）。
2. SKILL.md 改动仅 1 处表头文字（host-orchestrator 授权的路径文字级同步），未触碰 STORY-056-07 的 ST-EX-18/19/20 与 R-F-027/028 范围。

## 风险与重访条件

- R-056-02-A：规则块 v1.1.0 落到两项目后，存量用户若仍按 `cases/<协议族>` 传 `--cases-dir`，规则与文档已统一指向 `cases/te`，运行时 `discover_cases` 对任意目录仍可用（零代码改动），回归风险低；重访条件：STORY-056-07 G-01 冒烟发现 312 条用例未被 `cases/te` 命中。
- R-056-02-B：`.codex/agents/ptm-te.toml` 滞后（见未覆盖项 #1）；重访条件：host-orchestrator 决策同步或 waived。

## 后续交接

- **meta-qa（CP7 入口）**：
  - 结构验证：`uv run --python 3.11 pytest tests/test_install_mapping.py -q`（10 passed）。
  - 副本一致性验证：三份规则文件 ptm-te-workflow 块 diff（仅 header 时间戳与平台标签句可异）+ 三份 agent md 正文 diff；入口脚本 `uv run --python 3.11 python /tmp/cr056-refresh.py`（幂等可重跑，输出应零 diff——generated 时间戳除外）。
  - 风险提示：R-056-02-A/B；真机执行不在本 CR 范围（static-only + dry-run 口径）。
- **STORY-056-07**：SKILL.md 深度收口、执行指导.md、4 副本脚本同步、回归；建议一并裁决 `.codex/agents/ptm-te.toml`。
- **host-orchestrator**：Story 状态已置 `ready-for-verification`；`dev_gate` 字段维持 `cp5-pending` 由门禁账务统一回填；GATE/AGENT-DISPATCH ledger 事件待补记。
