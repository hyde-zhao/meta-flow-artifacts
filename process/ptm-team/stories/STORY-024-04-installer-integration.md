---
story_id: STORY-024-04
story_name: 安装器集成与验证
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-024-ptm-te-agent
tier: S
wave: W3
depends_on: [STORY-024-01, STORY-024-02, STORY-024-03]
blocks: []
status: lld-ready
lld_policy: technical-note
feature_design_refs: [ptm-te-agent]
file_ownership:
  - script/ptm_team/install.py
  - skills/README.md
  - docs/ptm-team-blueprint.md
parallel_safe: false
hld_ref: process/HLD-CR-024.md
design_evidence_type: technical-note
---

# STORY-024-04：安装器集成与验证

## 1. 目标

将 ptm-te agent + 3 个 skill（device-management / device-connection / policy-route-execution）集成进 `install.py` 安装器，支持三平台（claude/codex/qoder）投影安装。v1 不注入 rule block（CP2 DQ-03）。

## 2. 范围

- **修改文件**：3 个（install.py + skills/README.md + docs/ptm-team-blueprint.md）
- **不改动**：agents/ptm-te.md（S1）、skill 文件（S2/S3）
- **设计证据类型**：technical-note（写入本卡片，不单独 LLD）

## 3. 验收标准

- [ ] AC01：`AGENT_ALIASES` 含 `"te": "ptm-te"` 和 `"ptm-te": "ptm-te"`
- [ ] AC02：`get_agent_skills("ptm-te")` 返回 `["device-management", "device-connection", "policy-route-execution"]`
- [ ] AC03：v1 不注入 rule block（无 `render_ptm_te_rule_body`，CP2 DQ-03）
- [ ] AC04：三平台 dry-run PASS（claude->`.claude/agents`+`.claude/skills`；codex->`.codex/agents`+`.agents/skills`；qoder->`.qoder/agents`+`.qoder/skills`）
- [ ] AC05：`skills/README.md` 含 ptm-te 3 个 skill 索引
- [ ] AC06：`docs/ptm-team-blueprint.md` ptm-te 状态回写（planned->active）+ 修订记录
- [ ] AC07：真实安装验证（project scope `meta-flow install claude --project-dir <tmp>` 或等价 dry-run，ptm-te agent + 3 skill 正确投影）

## 4. Technical Note（设计证据）

本 technical-note 设计依据为 HLD v1.1 §12 + DEVELOPMENT-PLAN-CR-024.yaml。文件影响范围：3 个文件（install.py + skills/README.md + blueprint）。接口契约：复用 install.py 现有 AGENT_ALIASES / get_agent_skills / render_claude_agent / render_codex_agent / render_qoder_agent / PLATFORM_DIRS 接口，不新增渲染逻辑。数据：AGENT_ALIASES 加 2 键 + 新增 PTM_TE_SKILLS 常量。权限：v1 不注入 rule block（CP2 DQ-03），不授权修改 ptm-tde 基线，不授权 devices.yaml 入库明文。失败处理：install.py 改动破坏 ptm-tde 投影时，dry-run 回归验证拦截；三平台投影路径差异由现有 PLATFORM_DIRS 兜底。测试：三平台 dry-run + 真实安装验证 + ptm-tde 投影回归。风险：见 §6。

### 4.1 install.py 改动（script/ptm_team/install.py）

**改动 1：AGENT_ALIASES（第 60-63 行）**

```python
AGENT_ALIASES = {
    "tde": "ptm-tde",
    "ptm-tde": "ptm-tde",
    "te": "ptm-te",        # 新增
    "ptm-te": "ptm-te",    # 新增
}
```

**改动 2：新增 PTM_TE_SKILLS 常量（在 PMT_TDE_SKILLS 后）**

```python
# ptm-te referenced skills（设备管理 + 策略路由执行）
PTM_TE_SKILLS = [
    "device-management",
    "device-connection",
    "policy-route-execution",
]
```

**改动 3：get_agent_skills（第 407-411 行）加 ptm-te 分支**

```python
def get_agent_skills(agent_name: str) -> list[str]:
    """Get the list of skills referenced by an agent."""
    if agent_name == "ptm-tde":
        return PMT_TDE_SKILLS
    if agent_name == "ptm-te":          # 新增
        return PTM_TE_SKILLS            # 新增
    return []
```

**改动 4：rule block 不注入（CP2 DQ-03）**

- 不新增 `render_ptm_te_rule_body` 函数
- 不在 `PTM_TDE_RULE_FILES` 旁新增 ptm-te rule 文件
- ptm-te 的执行规则由 `agents/ptm-te.md` 自述（S1 产出），不写入 AGENTS.md/CLAUDE.md managed block
- 理由：ptm-te 首期流程简单（设备管理 + 策略路由执行），不需跨项目工作区隔离规则；ptm-tde 的 rule block 因三阶段框架 + 多特性隔离 + Gate 门控才需要

### 4.2 投影路径（复用现有 PLATFORM_DIRS，无需改动）

| 平台 | agents 目录 | skills 目录 |
|------|------------|------------|
| claude | `.claude/agents` | `.claude/skills` |
| codex | `.codex/agents` | `.agents/skills` |
| qoder | `.qoder/agents` | `.qoder/skills` |

install.py 现有 `render_claude_agent` / `render_codex_agent` / `render_qoder_agent` + `get_agent_skills` + `PLATFORM_DIRS` 机制自动处理 ptm-te 投影，无需新增渲染逻辑。

### 4.3 skills/README.md 改动

追加 ptm-te skill 索引段：

```markdown
## ptm-te 测试执行工程师 Skill

| Skill | 职责 | 来源 |
|-------|------|------|
| device-management | 设备清单（devices.yaml）+ 型号映射查表（元数据） | 迁移自 manaul |
| device-connection | SSH/Telnet 双轨连接 + 回退 + 系统快照采集 | 迁移自 manaul collect_sysinfo.py |
| policy-route-execution | op_id->子命令 + args->flag 双层映射 + 干跑/执行/verify + inverse_op 清理 | 按 ptm-atomic 真实 CLI 重写 |
```

### 4.4 docs/ptm-team-blueprint.md 改动

- ptm-te 状态行：`⬜ 未开始` -> `🔄 active（CR-024）`
- 追加 `## 修订记录`：`2026-07-10 CR-024 ptm-te 升格 active（设备管理 + 策略路由执行）`

### 4.5 验证

- **dry-run**：`meta-flow install claude --project-dir <tmp> --component agent`（或 install.py 等价 dry-run），确认 ptm-te.md + 3 skill 投影到 `.claude/agents/` + `.claude/skills/`
- **三平台**：claude/codex/qoder 各 dry-run 一次
- **回归**：ptm-tde 投影不受影响（AGENT_ALIASES 追加 + get_agent_skills 加分支，不改 ptm-tde 既有逻辑）

## 5. 依赖与门控

- **依赖**：STORY-024-01（agent md）+ STORY-024-02（device skill）+ STORY-024-03（policy-route skill）全部完成
- **阻塞**：无（W3 末尾）
- **dev_gate**：S4 technical-note 确认后进入实现

## 6. 风险

| 风险 | 缓解 |
|------|------|
| install.py 改动破坏 ptm-tde 投影 | 追加式修改（AGENT_ALIASES 加键 + get_agent_skills 加分支），不改 ptm-tde 既有逻辑；dry-run 回归验证 |
| 三平台投影路径差异 | 复用现有 PLATFORM_DIRS，无需新增逻辑 |
