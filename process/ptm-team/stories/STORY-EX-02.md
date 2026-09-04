---
cr_id: "CR-033"
story_id: "STORY-EX-02"
title: "规则固化（install.py ptm-te-workflow 规则块 ≥4 条）"
wave: 1
priority: "P0"
lld_policy: "technical-note"
feature_design_refs:
  - "docs/features/rule-fix-and-improvement/DESIGN.md"
depends_on: []
dependency_type: "none"
status: "lld-ready"
design_evidence_type: "technical-note"
lld_policy_required_level: "technical-note"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_requirements: "docs/product/REQUIREMENTS-PTM-TE-EXEC.md"
created_at: "2026-07-28T16:00:00+08:00"
author: "meta-dev"
---

# ST-EX-02 规则固化（install.py ptm-te-workflow 规则块 ≥4 条）

## 设计证据类型

technical-note（install.py 规则块增量，`render_ptm_te_rule_body` 模式既有，新增规则文本）。

## 设计依据

| 依据类型 | 引用 | 关联点 |
|---|---|---|
| HLD | §12.4 install.py 规则块渲染流程 | render_ptm_te_rule_body 新增 ≥4 条规则 |
| HLD | §16.1 改进 #2（max_loss 参数化）/ #3（ARP 预热双重保障）/ #4（session 生命周期） | 新增规则来源 |
| HLD | §16.1 改进 #1（TREX_API_URL 注入） | TG 路由规则来源 |
| ADR | ADR-06（ARP 预热双重保障，规则+引擎） | ARP 预热规则 |
| ADR | ADR-05（TREX_API_URL 注入） | TG 路由规则 |
| ADR | ADR-04（fw_logout 降级 session 清理） | session 生命周期规则 |
| 需求 | R-F-019（规则固化防重装回退）/ R-NF-003（重装一致性） | 成功标准 SM-EX-05/09 |
| 开发计划 | DEVELOPMENT-PLAN.yaml Wave 1 ST-EX-02 | file_ownership |

## 文件影响范围

| 文件 | 归属段 | 变更类型 | 说明 |
|---|---|---|---|
| `script/ptm_team/install.py` | `render_ptm_te_rule_body` | 修改函数体 | 在现有 8 条规则后新增 ≥4 条规则文本 |

**file_ownership 声明**（与 DEVELOPMENT-PLAN.yaml 一致）：
- `script/ptm_team/install.py#render_ptm_te_rule_body`

**不触碰文件**：
- `skills/policy-route-execution/scripts/op_mapper.py`（ST-EX-03 拥有 `_build_exec_env` / `resolve_env_refs`）
- `skills/case-execution/scripts/case_runner.py`（ST-EX-04 拥有，本 Story 只写规则文本不写引擎代码）
- `PTM_TE_SKILLS` 列表（ST-EX-04 拥有 `case-execution` 安装验证）

**注意**：ST-EX-02 只做规则块文本新增（`render_ptm_te_rule_body` 函数体），不修改 `PTM_TE_SKILLS` 列表（case-execution 安装验证已并入 ST-EX-04，HLD v1.1 P5 拆分）。

## 接口设计要点

### render_ptm_te_rule_body 既有结构

`render_ptm_te_rule_body(platform: str) -> str` 返回 `## ptm-te 工作流程规则` Markdown 块，经 `upsert_managed_rule_block` 写入平台规则文件（CLAUDE.md / AGENTS.md）。当前含 8 条编号规则（dry-run 门 / 凭据安全 / session 路径 / 执行入口 / op_id 阻塞 / id 来源 / 清理回滚 / 重装检查）。

### 新增 4 条规则（编号 9-12）

| 编号 | 规则标题 | 规则文本要点 | 关联改进/ADR |
|---|---|---|---|
| 9 | **TG 路由与 TREX_API_URL 注入** | TG 操作经 `ptm-atomic run tg trex <action>` 下发，框架禁止直接调 TG REST API；`case_runner` 从 `devices.yaml` 读 `tg.api_server`（或环境文件 `${ENV.tg.url}` 优先）传入 `op_mapper._build_exec_env`，注入为子进程 `TREX_API_URL` 环境变量；不得在 op_mapper 或 case_runner 中硬编码 TG 地址 | 改进 #1 / ADR-05 |
| 10 | **max_loss 参数化** | `tg_verify_traffic_loss` 的 `--max-loss` 必须由用例 `case_steps` 显式传参，禁止在脚本或规则中硬编码默认值；缺失 `max_loss` 参数时 `validate_args` 报 `PARAM_INVALID` | 改进 #2 |
| 11 | **ARP 预热双重保障** | `warming_up: true` 的 step 执行后，case_runner 引擎强制执行 `post_op`（如 `tg_stop_traffic_stream`），即使 md 未写 `post_op` 也自动补充；规则块固化此约束作为重装回退兜底（ADR-06 规则+引擎双重保障） | 改进 #3 / ADR-06 |
| 12 | **session 生命周期** | 用例执行结束后必须调用 `fw_logout` 登出并清理 session 文件；`ptm-atomic` 未暴露 `fw_logout` 时降级为 `os.remove(session_file)`，`result.json` 记录 `logout=fallback_session_cleanup`；session 文件路径必须在 `~/.local/state/ptm-atomic/` 下 | 改进 #4 / ADR-04 |

### 规则文本风格约束

- 与现有 8 条规则风格一致：编号 + **标题** + 冒号 + 规则正文
- 规则正文使用确定性动词（必须/禁止/不得），不用模糊表述（应/可以/建议）
- 每条规则可量化（如"≥4 条"、"禁止硬编码"、"必须调用"）

## 实施步骤

1. **定位修改点**：`script/ptm_team/install.py` 第 460-478 行 `render_ptm_te_rule_body` 函数体，现有 8 条规则文本
2. **新增规则 9（TG 路由）**：在规则 8（重装检查）后追加规则 9 文本，引用 ADR-05 TREX_API_URL 注入约束 + ptm-atomic 约束（禁止直接调 TG REST API）
3. **新增规则 10（max_loss 参数化）**：追加规则 10 文本，引用改进 #2，约束 `--max-loss` 必须显式传参
4. **新增规则 11（ARP 预热）**：追加规则 11 文本，引用 ADR-06 双重保障，约束 `warming_up:true` 强制 `post_op`
5. **新增规则 12（session 生命周期）**：追加规则 12 文本，引用 ADR-04 fw_logout 降级，约束登出 + session 清理
6. **验证规则块渲染**：执行 `python script/ptm_team/install.py install claude --component rules` 后，检查 CLAUDE.md 含 12 条规则
7. **重装一致性验证**：执行 install -> uninstall -> install 循环，确认规则块 12 条不丢失（SM-EX-09）

## 回滚策略

- 本 Story 只修改 `render_ptm_te_rule_body` 函数体（规则文本），不修改函数签名或安装逻辑
- 回滚方式：git revert 相关 commit，规则块恢复为 8 条
- 重装后旧规则块（8 条）会被 uninstall 清除，不影响 workspace

## 测试要点

| 测试项 | 验证方法 | 预期结果 |
|---|---|---|
| 规则数量 ≥4 条新增 | grep render_ptm_te_rule_body 返回文本中编号列表 | 12 条（原 8 + 新增 4），SM-EX-05 |
| TG 路由规则含 ptm-atomic 约束 | grep 规则 9 文本 | 含"ptm-atomic run tg trex"和"禁止直接调 TG REST API" |
| max_loss 规则含禁止硬编码 | grep 规则 10 文本 | 含"禁止硬编码"和"--max-loss" |
| ARP 预热规则含双重保障 | grep 规则 11 文本 | 含"warming_up"和"post_op"和"强制" |
| session 规则含 fw_logout + 降级 | grep 规则 12 文本 | 含"fw_logout"和"fallback_session_cleanup" |
| 规则块渲染正确 | install.py install claude --component rules 后读 CLAUDE.md | 含 `<!-- ptm-team:managed:ptm-te-rule-block:begin -->` 标记 + 12 条规则 |
| 重装一致性 | install -> uninstall -> install 循环 | 规则块 12 条不丢失，SM-EX-09 |
| 确定性语言 | grep 规则文本 | 无"应/可以/建议"模糊表述 |


## 权限与风险

- **权限**：NO_CREDENTIAL_READ（不读凭据）/ NO_PRODUCTION_WRITE（dry-run 默认门，不写生产）/ NO_EXTERNAL_PUBLISH（不发布）
- **失败行为**：install.py 规则块渲染失败 -> 安装中断并报错
- **风险**：规则块覆盖或丢失导致 ptm-te-workflow 规则不生效，需重装
- **数据**：规则块文本数据写入 install.py（render_ptm_te_rule_body），重装覆盖；不读用户数据，不写仓库外数据
## 开放项

无。4 条规则来源已在 HLD §16.1 改进追溯表和 ADR 中明确定义，无阻塞问题。

## 与 HLD/ADR 追溯一致性

| HLD/ADR 条目 | 本 Story 落实点 |
|---|---|
| HLD §12.4（render_ptm_te_rule_body 新增 ≥4 条） | 规则 9-12 |
| HLD §16.1 改进 #1（TREX_API_URL 注入） | 规则 9（TG 路由） |
| HLD §16.1 改进 #2（max_loss 参数化） | 规则 10 |
| HLD §16.1 改进 #3（ARP 预热双重保障） | 规则 11 |
| HLD §16.1 改进 #4（session 生命周期） | 规则 12 |
| ADR-05（TREX_API_URL 注入） | 规则 9 引用 ptm-atomic 约束 |
| ADR-06（ARP 预热双重保障） | 规则 11 引用规则+引擎 |
| ADR-04（fw_logout 降级） | 规则 12 引用降级 session 清理 |
| SM-EX-05（≥4 条规则） | 新增 4 条（规则 9-12） |
| SM-EX-09（重装一致性 0 丢失） | install/uninstall/install 循环验证 |
| R-F-019 / R-NF-003 | 规则固化防重装回退 |

## implementation_context（CP6 实现执行证据摘要）

**evidence_type**: technical-note（install.py 规则块文本增量，render_ptm_te_rule_body 既有模式，新增规则文本）

### 实现对象清单

| 文件 | 函数/锚点 | 变更 | 验证 |
|---|---|---|---|
| `script/ptm_team/install.py` | `render_ptm_te_rule_body` | 在规则 8 后新增规则 9-12（4 条），规则块从 8 条增至 12 条 | 12 条编号（1-12），规则 9-12 关键词全 PASS |

### 新增规则（编号 9-12）

| 编号 | 规则标题 | 关联改进/ADR | 关键词验证 |
|---|---|---|---|
| 9 | TG 路由与 TREX_API_URL 注入 | 改进 #1 / ADR-05 | `ptm-atomic run tg trex` + `禁止直接调 TG REST API` + `TREX_API_URL` + `${ENV.tg.url}` |
| 10 | max_loss 参数化 | 改进 #2 | `--max-loss` + `禁止` + `PARAM_INVALID` |
| 11 | ARP 预热双重保障 | 改进 #3 / ADR-06 | `warming_up` + `post_op` + `强制` |
| 12 | session 生命周期与 fw_logout 登出 | 改进 #4 / ADR-04 | `fw_logout` + `fallback_session_cleanup` + `~/.local/state/ptm-atomic/` |

### 设计契约映射

| LLD 契约 | 实现 | 一致性 |
|---|---|---|
| 新增 ≥4 条规则（SM-EX-05） | 新增 4 条（规则 9-12） | 一致 |
| 规则文本风格（编号+标题+确定性动词） | 12 条统一风格，无"应/可以/建议"模糊表述 | 一致 |
| 规则 9 引用 ADR-05 + ptm-atomic 约束 | 含 ptm-atomic run tg trex + 禁止直接调 TG REST API | 一致 |
| 规则 10 引用改进 #2 max_loss | 含 --max-loss + 禁止硬编码 + PARAM_INVALID | 一致 |
| 规则 11 引用 ADR-06 双重保障 | 含 warming_up + post_op + 强制 + ADR-06 | 一致 |
| 规则 12 引用 ADR-04 fw_logout 降级 | 含 fw_logout + fallback_session_cleanup + session 路径 | 一致 |
| f-string ${ENV.tg.url} 转义 | `${{ENV.tg.url}}` 渲染为 `${ENV.tg.url}` | 一致 |

### 本地验证

| 验证项 | 命令 | 结果 |
|---|---|---|
| install.py 语法 | `python3 -m py_compile install.py` | OK |
| 规则数量 | `render_ptm_te_rule_body('claude')` | 12 条（1-12），SM-EX-05 |
| 规则 9-12 关键词 | 关键词检查 | 全 PASS |
| ${ENV.tg.url} 转义 | 渲染输出含 `${ENV.tg.url}` | PASS（f-string 双花括号转义正确） |
| 确定性语言 | 无"应/可以/建议"模糊表述 | PASS |

### 任务描述差异说明

任务描述提到"更新规则 4 执行入口"。LLD §接口设计要点（CP5 approved 真相源）只要求新增 4 条规则（9-12），未要求修改规则 4。规则 4 现状正确描述用例读取入口（`cases/upload/<特性名>特性测试用例.md`），TG 路由执行入口约束已在规则 9 体现（`ptm-atomic run tg trex <action>`）。按 LLD 实现，不改规则 4。

### 平台差异

- install.py render_ptm_te_rule_body 是平台无关函数，规则文本经 upsert_managed_rule_block 写入 CLAUDE.md（claude）/ AGENTS.md（codex/qoder）
- 规则块 managed block 标记 `<!-- ptm-team:managed:ptm-te-rule-block:begin -->` 隔离，重装覆盖

### Agent Dispatch Evidence

inline-fallback（主进程直接执行），technical-note 低风险无需 subagent 调度。

### 回滚策略

git revert 即可恢复为 8 条规则（规则块文本增量，uninstall 清除 managed block，不影响 workspace）。重装一致性（SM-EX-09）依赖 install/uninstall/install 循环，本 Story 范围内规则文本已固化。
