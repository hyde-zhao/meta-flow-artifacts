---
change_id: CR-018-ptm-tde-workflow-compliance-and-workspace-isolation
workflow_id: WF-PTM-TEAM-20260520-001
created_at: "2026-06-11T00:00:00+08:00"
created_by: meta-po（po-zhao via current Codex session）
status: active
impact_level: high
complexity: standard
rollback_to: solution-design
approval_source: user-request
source_request:
  - "分析 ptm-tde 多次未按流程执行的原因并提出整改"
  - "安装 ptm-tde agent 时，Codex 写入 AGENTS.md，Claude 写入 CLAUDE.md；文件不存在则创建"
  - "支持一个项目分析多个特性：以 .input 所在目录判断特性工作区，分析过程放入 .input 同级目录，目录间状态隔离"
supersedes:
  - "CR-015-FOLLOW-UP T-01 Codex 平台 AskUserQuestion 等价交互整改"
related_changes:
  - CR-011-ptm-tde-kym-phase
  - CR-012-ptm-tde-mfq-phase
  - CR-013-ptm-tde-ppdcs-phase
  - CR-015-ptm-tde-ask-user-question-interaction
  - CR-016-atomic-ops-consumption-gap
  - CR-017-factor-library-discovery-gap
---

# CR-018 — ptm-tde 流程合规、安装投影与多特性工作区隔离

## 变更请求摘要

本 CR 将本轮分析发现的 ptm-tde 流程执行问题与用户新增的两个需求统一纳入正式整改：

1. 修复 ptm-tde 不按流程执行的结构性原因：安装副本漂移、Gate 骨架检查误放行、文档多真相源、Skill 执行证据缺失、Codex/Claude 交互协议混写。
2. 安装 `ptm-tde` agent 时自动向目标项目平台规则文件写入工作流程：
   - Codex：写入或更新目标项目 `AGENTS.md`
   - Claude Code：写入或更新目标项目 `CLAUDE.md`
   - 文件不存在时创建；文件存在时只替换 ptm-team managed block，不覆盖用户手写内容。
3. 支持同一项目内多个特性并行分析：
   - 标准输入目录为 `.input/`
   - `feature_workspace_root = parent(.input)`
   - 若 `.input` 不在仓库根目录，`kym/`、`mfq/`、`ppdcs/`、`process/` 必须写入 `.input` 同级目录
   - 每个 `feature_workspace_root/process/STATE.yaml` 独立维护状态，不同特性目录之间不得共享或覆盖状态。

## 当前根因分析

| 根因 | 证据 | 影响 | 整改方向 |
|---|---|---|---|
| 安装副本漂移 | 多个已安装项目中的 `.codex/agents/ptm-tde.toml` 仍保留旧 `.input/.output` 或旧 CP 流程；源仓库 `agents/ptm-tde.md` 与已安装副本不一致 | 用户实际调用的 agent 可能不是最新流程 | 安装器写入平台规则托管块，后续增加 manifest hash / drift check |
| 共享资源卸载越界 | 临时验证发现 uninstall agent 会删除 `~/.ptm-team/resource/*` 共享资源目录 | 一个项目卸载 ptm-tde 可能破坏其他项目运行环境 | resource entry 从 manifest 解除关联，但默认保留共享资源目录 |
| Gate 骨架检查误放行 | `run_checkpoint.py` GATE-2/3/4/5 多处只检查目录存在；GATE-2 原先 mission 缺失也可能 PASS | Agent 可在关键产物缺失时推进阶段 | GATE-2 先修阻断；后续逐 Gate 升级真实校验 |
| 文档多真相源 | `agents/ptm-tde.md`、`docs/ptm-tde/*`、部分 Skill 仍混用 `input/`、`.output/`、`doc/STATE.yaml`、AskUserQuestion/ask_user | Agent 和用户看到的流程不一致 | 明确 `.input` 与 `feature_workspace_root` 为主契约，全仓 grep 清理旧口径 |
| Skill 调用证据不足 | Agent 声明 Skill 已执行时缺少统一 `call_id`、输入、输出、状态、证据路径 | 无法区分“真实 Skill 执行”和“Agent 代写产物” | 新增 Skill execution evidence schema，并让 Gate 消费 |
| 平台交互协议混写 | CR-015 主要覆盖 Claude AskUserQuestion；Codex 场景仍可能引用不可用工具 | 人工确认点被跳过或表达不一致 | 平台协议统一为“结构化选择优先，否则 exact text 三选项” |

## 五维度影响分析

### 1. 需求影响

| 项 | 影响 |
|---|---|
| 新增需求 | 安装投影、`.input` 工作区根、状态隔离、多 `.input` 阻断选择 |
| 变更需求 | 原 `input/` 运行约定调整为 `.input/` 主路径；legacy `input/` 仅可作为兼容 fallback，不再作为新文档主路径 |
| 需求冲突 | 若旧文档仍指导用户使用 `input/`，会与新安装规则冲突；本 CR 必须分类清理 |

### 2. 场景影响

| 场景 | 期望行为 |
|---|---|
| 根目录单特性 | `<repo>/.input` 存在时，输出写入 `<repo>/kym`、`<repo>/mfq`、`<repo>/ppdcs`、`<repo>/process` |
| 子目录特性 | `<repo>/features/A/.input` 存在时，输出写入 `<repo>/features/A/*`，不得写入 `<repo>/*` |
| 多特性 | 发现多个 `.input` 且未指定目标时阻断，并提示用户指定 `--project-root` 或 `--input-dir` |
| 已有 AGENTS.md / CLAUDE.md | 保留原内容，只新增或替换 managed block |
| 重复安装 | 同一个 managed block 原地更新，不追加重复块 |
| 卸载 | 只移除 managed block，不删除规则文件 |
| Codex 无结构化 UI | 使用 exact text：`approve`、`修改: <具体修改点>`、`reject` |

### 3. 计划影响

本 CR 采用 standard 模式，不走 fast-lane。meta-po 子线程建议拆成 CR-018~CR-021 四个 CR；本轮按用户要求合并为一个 umbrella CR，但内部按四个实施主题拆分 Story/Wave。

| Story | 主题 | 状态 | 说明 |
|---|---|---|---|
| STORY-018-01 | `.input` 运行根与状态隔离 | in_progress | 已开始修改 checkpoint resolver 与 agent 路径契约 |
| STORY-018-02 | 安装器 managed block 投影 | in_progress | 已开始实现 `AGENTS.md` / `CLAUDE.md` upsert 与 uninstall 清理 |
| STORY-018-03 | 文档与 Skill 路径口径收敛 | in_progress | 已覆盖 `docs/ptm-tde/*` 主路径、checkpoint-manager、scenario-discovery、change-impact、bug-gap；仍需后续审查全量 AskUserQuestion 平台措辞 |
| STORY-018-04 | Gate 真实校验升级 | done | GATE-2/3/4/5 已从 skeleton 升级为 machine-baseline；GATE-2/3/4 会生成独立 `*-manual.md` 人工审查稿；P2 已补字段级结构检查 |
| STORY-018-05 | Skill 执行证据与平台交互协议 | in_progress | 已定义 `process/execution/SKILL-CALLS.yaml` 最小 schema，新增 `--record-skill-call` 写入器，并接入 GATE-2/3/4 结构化阻断；高频 Skill 已收敛 Claude/Codex 交互分支 |
| STORY-018-06 | 安装漂移检测与测试 | done | manifest 已记录 agent/skill/rule/resource source_hash 与 installed_hash；`check <platform> --agent ptm-tde` 检查 installed/source drift、managed block 缺失、目标文件缺失和 resource 内容漂移；P2 已沉淀 unittest |

### 4. 安全/权限影响

| 风险 | 控制 |
|---|---|
| 安装器覆盖用户规则文件 | 仅替换 `<!-- ptm-team:managed:ptm-tde-workflow:begin ... -->` 到 `end` 的 managed block |
| 卸载误删规则文件 | uninstall 只删除 managed block，不删除 `AGENTS.md` / `CLAUDE.md` |
| 卸载误删共享资源 | uninstall 不删除用户级 `~/.ptm-team/resource/*`，只更新当前 workspace manifest；后续如需清理共享资源，应单独提供显式 resource prune 命令 |
| 多 `.input` 静默选错特性 | resolver 发现多个 `.input` 必须返回错误并要求显式选择 |
| 状态串目录 | `process/STATE.yaml` 写入 `feature_workspace_root` 和 `input_root`，不同目录独立 |
| Gate 误判质量 | 结构化字段和 trace 做硬校验；语义质量进入 manual draft，不让脚本假装理解质量 |

### 5. 交付影响

| 交付面 | 影响 |
|---|---|
| 安装器 | `script/ptm_team/install.py` 与 `script/install.py` 必须保持一致或收敛为单一 canonical 入口 |
| Agent | `agents/ptm-tde.md` 必须明确 `.input`、`feature_workspace_root`、状态隔离、Skill 证据 |
| Skills | `checkpoint-manager`、`feature-parser`、`kym`、`scenario-discovery`、MFQ/PPDCS 关键 Skill 需同步路径和交互协议 |
| Docs | `docs/ptm-tde/README.md`、`USER-MANUAL.md`、`gate-spec.md`、`runtime-artifacts.md` 需统一新路径和 Gate 语义 |
| Tests | 至少覆盖安装托管块、卸载托管块、多 `.input` 阻断、子目录 `.input` 输出隔离、GATE-2 mission 缺失阻断 |

## 决策项

| 决策 ID | 类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-018-01 | implementation | 是否保留 legacy `input/` fallback | `.input` 为唯一主路径；`input/` 仅 legacy-read fallback + warning，不进入新文档示例 | 彻底移除 `input/` fallback | 推荐方案兼容旧项目，但需 grep 标注 legacy |
| DQ-018-02 | runtime_authorization | 多 `.input` 未指定时是否允许自动选择 | 不允许，必须询问用户或要求 `--input-dir` | 默认选择最近或第一个 | 默认选择风险高，可能污染其他特性状态 |
| DQ-018-03 | security | 是否允许安装器改写规则文件非 managed 内容 | 不允许，只能改 managed block | 覆盖整文件 | 覆盖整文件会破坏团队自定义规则 |
| DQ-018-04 | implementation | 两个安装入口如何治理 | 短期保持同步；后续将 `script/install.py` 收敛为 thin wrapper | 继续长期双副本 | 长期双副本会继续制造安装漂移 |
| DQ-018-05 | implementation | Skill execution evidence 路径 | 推荐 `process/execution/SKILL-CALLS.yaml`，由 feature workspace root 隔离 | 分散写入各阶段目录 | 集中文件更便于 Gate 消费和审计 |
| DQ-018-06 | risk_acceptance | Gate 中哪些项允许 WARN | 资源缺失可 WARN；人工确认、trace 断链、状态串目录、核心产物缺失必须 BLOCKING | 维持目录存在性检查 | 目录检查会继续误放行 |

## 已启动实施切片

| 文件 | 当前改动 |
|---|---|
| `script/ptm_team/install.py` | 新增 ptm-tde 平台规则 managed block 渲染、upsert、manifest rule entry、uninstall 清理；卸载 agent 时保留共享 resource |
| `script/install.py` | 与包内安装器同步，避免入口漂移 |
| `script/ptm_team/install.py` / `script/install.py` | P1 新增 `check <platform> --agent ptm-tde`：基于 `.ptm-team-manifest.json` 校验 agent、skill、rule installed_hash；校验源资产 source_hash；校验 `AGENTS.md` / `CLAUDE.md` 中 managed block 是否存在 |
| `script/ptm_team/install.py` / `script/install.py` | P2 新增 resource 文件列表和内容 hash，`check` 对共享 resource 执行内容漂移检查，不再跳过 resource |
| `skills/checkpoint-manager/scripts/run_checkpoint.py` | 新增 `.input` resolver、`--input-dir`、多 `.input` 阻断、状态写入 `feature_workspace_root/input_root`；GATE-2 mission 缺失不再 PASS |
| `skills/checkpoint-manager/scripts/run_checkpoint.py` | P2 新增字段级机器基线：GATE-2 场景字段、GATE-3 CAE/trace/binding/PPDCS 字段、GATE-4 PC 16 列和交付 trace 字段 |
| `agents/ptm-tde.md` | 运行目录契约切换为 `.input` + `feature_workspace_root` + 状态隔离 |
| `skills/feature-parser/SKILL.md` | 修正 `.input`、`process/STATE.yaml`、平台通用确认协议 |
| `docs/ptm-tde/*` | 主用户文档、Gate spec、runtime artifacts、component manual 已切换 `.input`、`feature_workspace_root`、machine-baseline 和 drift check 说明 |
| `skills/scenario-discovery/SKILL.md` | TGFW topo 输入从 `input/` 改为 `.input/` |
| `skills/change-impact-analyzer/SKILL.md` / `skills/bug-gap-analyzer/SKILL.md` | 旧 `analysis/design/checkpoints/delivery/doc/STATE.yaml` 基线改为 `kym/mfq/ppdcs/process` 基线；P1 补充 `.input` → `feature_workspace_root` 工作区隔离契约和扩展分支内部输出约束 |
| `skills/kym/SKILL.md` / `skills/f-analyzer/SKILL.md` / `skills/q-analyzer/SKILL.md` / `skills/test-point-integrator/SKILL.md` / `skills/design-planner/SKILL.md` / `skills/deliverable-renderer/SKILL.md` | 高频人工交互点收敛为 Claude Code `AskUserQuestion` 可用时结构化选择，Codex 或不可用时 exact text / STOP-05 文本协议 |
| `tests/test_cr018_p2.py` | 新增正式 unittest，覆盖 resource drift、GATE-3 字段级阻断、GATE-4 PC 16 列阻断 |

## 验收标准

| 编号 | 标准 |
|---|---|
| AC-01 | `uv run python script/install.py install codex --agent ptm-tde --dry-run` 显示将写入 `AGENTS.md` managed block |
| AC-02 | `uv run python script/install.py install claude --agent ptm-tde --dry-run` 显示将写入 `CLAUDE.md` managed block |
| AC-03 | 已有规则文件重复安装只更新 managed block，不追加重复块，不改用户内容 |
| AC-04 | uninstall 只删除 managed block，不删除 `AGENTS.md` / `CLAUDE.md` |
| AC-04a | uninstall agent 不删除用户级共享 resource 目录，只解除当前 manifest 关联 |
| AC-05 | 子目录 `.input` fixture 运行 GATE-1 后，`process/STATE.yaml`、`kym/`、`mfq/`、`ppdcs/` 均位于 `.input` 父目录 |
| AC-06 | 多 `.input` fixture 未指定目标时，checkpoint 脚本返回非 0 并打印显式选择提示 |
| AC-07 | GATE-2 在缺少 `kym/mission-understanding/mission-statement.md` 时返回 BLOCKED |
| AC-08 | 全仓 `rg "doc/STATE.yaml|\\.output|ask_user|input/"` 分类审查完成，保留项必须标注 legacy / historical reason |
| AC-09 | 新增或更新测试覆盖安装投影、工作区隔离、Gate 阻断和 manifest rule entry |
| AC-10 | `uv run python script/install.py check <platform> --agent ptm-tde` 能在安装后 PASS，删除 managed block 或改动已安装资产时返回非 0 |
| AC-11 | resource 文件被修改后，`check <platform> --agent ptm-tde` 返回非 0 并报告 `INSTALLED_DRIFT resource:` |
| AC-12 | GATE-3 缺 CAE/trace/binding 字段或 GATE-4 缺 PC 16 列时返回非 0 |
| AC-13 | `uv run python -m unittest tests.test_cr018_p2` 通过 |

## 回滚方案

1. 安装器回滚：删除 manifest 中 `kind=rule` 的处理逻辑；目标项目可通过 uninstall 移除 managed block。
2. 工作区 resolver 回滚：恢复 `--project-root` 直接作为运行根，但必须同步回滚 agent/docs 中 `.input` 契约。
3. Gate 阻断回滚：仅允许回滚到更严格的人工阻断，不允许回滚到 mission 缺失也 PASS。
4. 文档回滚：保留 CR-018 记录旧基线，不用新草案覆盖旧文档。

## 修订记录

| 日期 | 修订人 | 版本 | 说明 |
|---|---|---|---|
| 2026-06-11 | meta-po / current Codex session | v0.1 | 创建 CR，纳入流程合规整改、安装投影和 `.input` 多特性隔离；启动第一批实现切片 |
| 2026-06-11 | meta-po / current Codex session | v0.2 | 继续实施：GATE-2/3/4/5 machine-baseline、Skill execution evidence、manifest hash、文档路径收敛和临时 fixture 验证 |
| 2026-06-11 | meta-po / current Codex session | v0.3 | P0 整改：新增 `--record-skill-call` 结构化 evidence 写入器，Gate evidence 结构化校验，独立 manual Gate 草稿，高频 Skill 平台交互协议收敛 |
| 2026-06-11 | meta-po / current Codex session | v0.4 | P1 整改：新增安装 drift check 命令，Gate/CP 文档从 skeleton 残留收敛到 machine-baseline + Skill evidence，change/bug 扩展 Skill 补齐工作区隔离契约；临时 fixture 验证 PASS 路径和 managed block 缺失阻断 |
| 2026-06-11 | meta-po / current Codex session | v0.5 | P2 整改：resource content hash + drift check、Gate 字段级机器基线、正式 unittest 夹具落地 |
