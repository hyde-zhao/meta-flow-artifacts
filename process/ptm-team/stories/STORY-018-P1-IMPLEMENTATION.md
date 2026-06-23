# STORY-018 P1 Implementation Evidence

## Scope

CR-018 P1整改覆盖安装漂移检查、Gate/CP 文档口径收敛，以及 ptm-tde 扩展 Skill 的多特性工作区隔离契约。

## Implementation Objects

| Object | Files | Change |
|---|---|---|
| Installer / guardrail | `script/ptm_team/install.py`, `script/install.py` | 新增 `check <platform> --agent ptm-tde`，校验 manifest、installed hash、source hash 和规则 managed block |
| User docs | `docs/ptm-tde/README.md`, `docs/ptm-tde/USER-MANUAL.md` | 增加 install/check 命令、AGENTS/CLAUDE 托管块说明、卸载不删共享 resource 说明、多 `.input` 工作区说明 |
| Gate spec | `docs/ptm-tde/gate-spec.md` | 移除当前流程中的 skeleton 误导口径，补充 Skill evidence 和 manual Gate 草稿契约 |
| Extension skills | `skills/change-impact-analyzer/SKILL.md`, `skills/bug-gap-analyzer/SKILL.md` | 增加 `.input` -> `feature_workspace_root` 隔离契约，修正旧 `analysis/*` / `design/*` 输入路径 |

## Contract Mapping

| Contract | Implementation |
|---|---|
| 安装后可诊断投影漂移 | `check_installed_drift()` 比较 `installed_hash`、`source_hash`，并检查 managed block 是否存在 |
| AGENTS.md / CLAUDE.md 只管理托管块 | drift check 对 rule 只提取 managed block body，不读取用户自定义规则 |
| 多特性状态隔离 | change/bug Skill 明确只消费当前 `feature_workspace_root` 的 `kym/mfq/ppdcs/process` |
| Gate 主体系清晰 | `gate-spec.md` 明确 CP 只是兼容路由，Gate 才是主状态机 |

## Verification Plan

| Check | Command |
|---|---|
| Python syntax | `uv run python -m compileall script/ptm_team/install.py script/install.py skills/checkpoint-manager/scripts/run_checkpoint.py` |
| Drift check PASS fixture | temp workspace install + `uv run python script/install.py check codex --agent ptm-tde` |
| Drift check missing block fixture | 删除 temp `AGENTS.md` managed block 后 check 返回非 0 |
| Text guardrail | grep old skeleton / old analysis path / project-root wording |
| Whitespace | `git diff --check` |

## Verification Results

| Check | Result |
|---|---|
| Python syntax | PASS |
| Text guardrail | PASS；旧 `当前为骨架`、`analysis/integration`、`design/pc`、`doc/STATE.yaml`、旧 AskUserQuestion 强制措辞均未命中 |
| Whitespace | PASS |
| Drift check PASS fixture | PASS；`/tmp/ptm-tde-p1.5EflZs` 真实安装后 `check codex --agent ptm-tde` 返回 0，agent/rule/20 skills 全部 OK |
| Drift check missing block fixture | PASS；临时移走 `AGENTS.md` 后 `check codex --agent ptm-tde` 返回 2，并报告 `MISSING rule:ptm-tde:ptm-tde-workflow` |

## Remaining Risk

P1 未实现 resource content hash 的安装漂移校验；共享 resource 仍按用户级目录保留策略处理。字段级 Gate parser 和正式 unittest 夹具归入 P2。

## P2 Closure

上述 P1 剩余风险已由 `process/stories/STORY-018-P2-IMPLEMENTATION.md` 关闭：resource content hash、字段级 Gate parser 和正式 unittest 夹具均已落地。
