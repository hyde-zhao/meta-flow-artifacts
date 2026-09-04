---
cr_id: "CR-033"
phase: "requirement-clarification"
status: "resolved"
sgq_count: 5
created_at: "2026-07-28T09:00:00+08:00"
resolved_at: "2026-07-28T10:30:00+08:00"
discussion_mode: "exact-text"
revision: "v1.1 (CP2 scope modification: SGQ-05 added)"
---

# CR-033 CP2 Scenario Gray Areas 讨论日志

## 概要

CR-033（ptm-te TG 设备建模 + case-execution 执行引擎 + 12 条改进）在 CP2 前识别 4 个影响交付的灰区。用户通过 host-orchestrator relay 确认全部 4 项选择，所有灰区已 resolved。

---

## SGQ-01：case-execution 的 dry-run 默认门与 runtime_authorization 边界

| 字段 | 内容 |
|------|------|
| **问题** | case_runner.py 默认 dry-run（不实际执行写操作），`--execute` 是授权门。但 24 用例中大量步骤是写操作，dry-run 模式下只打印命令不实际执行，无法验证用例逻辑。如何平衡"安全默认"与"可验证性"？ |
| **候选 A（推荐）** | dry-run 是默认门，`--execute` 需用户显式授权；dry-run 模式下校验命令构建和参数映射正确性，不校验业务结果；`--execute` 模式下才校验 PASS/FAIL |
| **候选 B** | 分两级授权：`--dry-run`（只构建命令）/ `--validate`（实际执行只读 op 如 verify，跳过写 op）/ `--execute`（全量执行） |
| **候选 C** | 默认 `--execute`，通过 `VALIDATION-ENV.yaml` 控制是否允许运行时 |
| **推荐项** | A |
| **用户回答** | A |
| **复述确认** | dry-run 是默认门，校验命令构建和参数映射正确性，不校验业务结果；--execute 需显式授权才校验业务 PASS/FAIL。安全默认与可验证性分离。 |
| **影响面** | 范围 / 验证方式 / CP7 验证层 / runtime_authorization 决策项 |
| **状态** | resolved |

**trade-off 分析**：
- A 优势：安全默认（不触发写操作），dry-run 仍有价值（校验命令构建/参数映射），授权边界清晰
- A 劣势：dry-run 模式下无法验证业务逻辑正确性，需要用户显式 `--execute` 才能跑全量
- B 优势：更细粒度控制
- B 劣势：增加引擎复杂度，三态授权门对用户认知负担大
- C 优势：默认可验证
- C 劣势：安全风险高，与 NO_PRODUCTION_WRITE 策略冲突

---

## SGQ-02：跨仓库路由--ptm-team(canonical 源) ↔ ptm-te(workspace) 的产物归属与回填

| 字段 | 内容 |
|------|------|
| **问题** | case-execution skill 源在 ptm-team，但运行环境是 ptm-te workspace。24 用例 md、devices.yaml、runs/ 都在 ptm-te。规范化后的 skill 是否安装回 ptm-te？产物文档写哪个仓库？ |
| **候选 A（推荐）** | skill 源在 ptm-team `skills/case-execution/`（canonical），通过 install.py 安装回 ptm-te `.claude/skills/`；过程文档写 ptm-team `docs/product/`；24 用例 md 留 ptm-te 不迁入 |
| **候选 B** | skill 源和过程文档都在 ptm-team，但 24 用例 md 也迁入 ptm-team `cases/` 目录统一管理 |
| **候选 C** | skill 源在 ptm-team，过程文档写 ptm-te workspace `docs/` |
| **推荐项** | A |
| **用户回答** | A |
| **复述确认** | skill 源在 ptm-team `skills/case-execution/`（canonical），过程文档写 ptm-team `docs/product/`，24 用例 md 留 ptm-te 不迁入，install.py 安装回 ptm-te `.claude/skills/`。 |
| **影响面** | 交付出口 / 文件所有权 / install.py 规则 / 重装一致性 |
| **状态** | resolved |

**trade-off 分析**：
- A 优势：canonical 源单一（ptm-team），install.py 回填保证重装一致性，24 用例 md 留 workspace 不污染源仓库
- A 劣势：跨仓库需要 install.py 维护回填逻辑
- B 优势：统一管理
- B 劣势：24 用例 md 是运行时数据，迁入源仓库增加管理负担
- C 优势：过程文档离运行环境近
- C 劣势：ptm-te workspace 不是 git 仓库，过程文档无法版本控制

**文档文件决策**：现有 `docs/product/USE-CASES.md` / `REQUIREMENTS.md` 等文件属于 CR-030/CR-031（ptm-tse 逆向分析，scenario_subject_id=ptm-tse）。CR-033 是不同产品线（ptm-te 执行引擎，scenario_subject_id=ptm-te-exec）。为避免混合不同产品线的场景和需求，CR-033 的产品文档使用 `-PTM-TE-EXEC` 后缀新建文件（如 `USE-CASES-PTM-TE-EXEC.md`），不修改既有 ptm-tse 文档。

---

## SGQ-03：known_issue 的 DUT 行为差异判定如何与脚本 bug 区分

| 字段 | 内容 |
|------|------|
| **问题** | M4-01-09 STEP-007 删除被引用的地址对象，DUT 行为是"被阻止"（符合异常用例预期）；M3-01-02 入接口指定 GE0_3 但流量从 port3 进，DUT 行为是"不命中"（符合异常用例预期）。case_runner 如何区分"DUT 行为差异导致的预期失败"（KNOWN_FAIL）和"脚本 bug 导致的意外失败"（FAIL）？ |
| **候选 A（推荐）** | 用例 md 中通过 `known_issue` 字段显式标记 DUT 行为差异步骤；case_runner 遇到 known_issue 标记的步骤失败时输出 KNOWN_FAIL，未标记的失败输出 FAIL |
| **候选 B** | case_runner 通过 `expected_result` 文本语义自动推断（如"被阻止"/"不命中"出现在 expected_result 中则视为 KNOWN_FAIL） |
| **候选 C** | 不区分，所有失败都是 FAIL，由人工后处理判断 |
| **推荐项** | A |
| **用户回答** | A |
| **复述确认** | 用例 md 中通过 `known_issue` 字段显式标记 DUT 行为差异步骤；case_runner 遇标记步骤失败输出 KNOWN_FAIL，未标记输出 FAIL。显式可审计。 |
| **影响面** | 结果分级 / 用例 md 改造量 / 24 用例 ARP 预热整改 / 自动化程度 |
| **状态** | resolved |

**trade-off 分析**：
- A 优势：显式标记可审计，判定逻辑清晰，不依赖文本语义推断
- A 劣势：需要人工在用例 md 中标注 known_issue 字段
- B 优势：自动化程度高
- B 劣势：文本语义推断不可靠，"被阻止"等措辞可能歧义
- C 优势：引擎最简
- C 劣势：无法自动区分，人工后处理负担重

---

## SGQ-04：ARP 预热与正式流使用相同 template 的强制方式

| 字段 | 内容 |
|------|------|
| **问题** | 24 用例中 ARP 预热（warming_up: true）和正式流使用相同的 `tg_apply_traffic_template` + `tg_start_traffic_stream` op，只是 name 后缀不同。#3 要求"24 用例 ARP 预热批量整改"。如何强制 ARP 预热的一致性？ |
| **候选 A** | 规则固化--install.py ptm-te-workflow 规则块中写明"ARP 预热必须使用 warming_up: true + post_op: tg_stop_traffic_stream"，用例 md 校验脚本检查合规性 |
| **候选 B** | 引擎强制--case_runner 识别 warming_up: true 时自动执行 post_op，不依赖用例 md 手动写 post_op |
| **候选 C（推荐）** | 两者结合--规则定义 + 引擎强制（warming_up: true 时 case_runner 自动补 post_op，即使 md 未写） |
| **推荐项** | C |
| **用户回答** | C |
| **复述确认** | install.py ptm-te-workflow 规则块定义 ARP 预热要求 + case_runner 识别 `warming_up:true` 时自动补 post_op（即使 md 未写）。规则与引擎双重保障，防重装回退。 |
| **影响面** | 规则块 / case_runner 引擎逻辑 / 24 用例 md 整改量 / 重装回退风险 |
| **状态** | resolved |

**trade-off 分析**：
- A 优势：规则可读，校验脚本独立
- A 劣势：只靠规则不靠引擎，重装后规则可能被绕过
- B 优势：引擎强制可靠
- B 劣势：规则不可见，新用例作者不知道要求
- C 优势：规则定义 + 引擎强制双重保障，防重装回退
- C 劣势：需要同时维护规则块和引擎逻辑

---

## Deferred Ideas（未深入讨论但记录的候选）

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-EX-01 | HTML 报告生成 | CR-033 非目标 | result.json + report.md 足够，HTML 进 BACKLOG | 用户明确要求或 CP8 后 |
| DEF-EX-02 | `--validate` 中间授权模式（只读 op 实际执行，写 op 跳过） | SGA-01 候选 B | 增加引擎复杂度，dry-run + --execute 二级已足够 | 需要更细粒度验证时 |
| DEF-EX-03 | 24 用例 md 迁入 ptm-team 统一管理 | SGA-02 候选 B | 24 用例是运行时数据，留 workspace 更合适 | 需要版本控制用例 md 时 |
| DEF-EX-04 | expected_result 文本语义自动推断 KNOWN_FAIL | SGA-03 候选 B | 文本推断不可靠，显式标记更可审计 | NLP 能力成熟后 |
| DEF-EX-05 | ptm-atomic CLI 本体扩展（如 fw_delete_object 暴露） | CR-033 非目标 | 只改消费侧，不改 ptm-atomic 本体 | ptm-atomic 升级 CR |
| DEF-EX-06 | pydantic devices.yaml schema 校验 | CR-033 非目标 | 不引入 pydantic，保持轻量 | 设备模型复杂度显著增加时 |

---

## SGQ-05：CP2 范围修改-用例结构化与执行入口调整（v1.1 新增）

| 字段 | 内容 |
|------|------|
| **问题** | CP2 门禁 pending 时用户回复"修改"，提出 5 点范围修改：(1) 执行入口 --case-files 替换为 --tag/--keyword；(2) 用例目录结构改为三级；(3) 用例 md frontmatter 16 列；(4) 用例命名 <编号>-<名称>.md；(5) 标签执行关键词列+tags 列。是否全部纳入 CR-033 范围？ |
| **候选 A（推荐）** | 全部纳入 CR-033，新增 R-F-022..026，R-F-021 扩大整改范围，UC-EX-10 新增用例结构化场景 |
| **候选 B** | 仅纳入执行入口调整（--tag/--keyword），用例结构化（目录/命名/frontmatter）作为后续独立 CR |
| **候选 C** | 全部纳入但分两期：P1 先做执行入口 + frontmatter 定义，P2 再做 24 用例目录迁移 |
| **推荐项** | A |
| **用户回答** | A（全部纳入，已与用户确认 3 个决策点：目录迁移范围 / frontmatter 必填字段 / 命名冲突处理） |
| **复述确认** | 全部 5 点范围修改纳入 CR-033。执行入口改为单用例/按目录/按标签或关键字；用例目录三级结构；frontmatter 16 列；用例命名 <编号>-<名称>.md；24 用例全量整改（目录迁移+重命名+frontmatter+tags+ARP 预热）。新增 DQ-04（目录迁移范围）、DQ-05（frontmatter 必填字段）、DQ-06（命名连字符冲突）。 |
| **影响面** | 范围 / 需求 R-F-004/R-F-021 更新 + R-F-022..026 新增 / UC-EX-10 新增 / SCN-EX-21..25 新增 / 里程碑 M3/M4 扩大 / 24 用例整改工作量增加 |
| **状态** | resolved |

**trade-off 分析**：
- A 优势：一次性完成避免半迁移状态；用例结构化与执行引擎同步设计减少返工
- A 劣势：24 用例整改工作量增加（目录迁移+重命名+frontmatter 补全+tags 标注+ARP 预热）
- B 优势：减少当前 CR 范围
- B 劣势：执行入口 --tag/--keyword 依赖 frontmatter tags 列，不先做结构化则 --tag 无数据源
- C 优势：分两期降低单次工作量
- C 劣势：半迁移状态增加协调成本

**新增 DQ**：
- DQ-04：24 用例目录迁移是否全部纳入 CR-033 范围 → 推荐 A（全部纳入 M4 一次性完成）
- DQ-05：frontmatter 16 列哪些必填 → 推荐 8 必填 + 8 可选
- DQ-06：用例名称连字符与文件名分隔符冲突 → 推荐以 frontmatter 用例编号列为唯一标识
