---
status: active
version: "2.0"
created_by: "meta-po"
created_at: "2026-04-24T11:05:36+08:00"
updated_at: "2026-04-24T16:00:00+08:00"
scope: "full-delivery verification snapshot (STORY-01 .. STORY-09)"
supersedes: "v1.0 STORY-03 targeted snapshot"
---

# TEST-STRATEGY v2

## 验证目标

- 覆盖 `STORY-01` ~ `STORY-09` 全部 verified 产物，形成交付态验证快照
- 验证 `scripts/install.py` 在 `claude-code / codex` 两平台 × `project / user` 两 scope 的 dry-run 行为
- 验证 `scripts/mcp_query_client.py` 的只读 staged query 三态契约（mock-only v2.0）
- 验证仓库级护栏（kebab-case / 资产同树 / 无缓存 / manifest 路径）
- 验证 `agents/ptm-tde.md` 与 17 个 Skill 在信息缺失时显式停顿、消费新模型

## 验证范围

- `scripts/install.py`、`scripts/install.sh`、`scripts/install.ps1`
- `scripts/check_delivery_guardrails.py`
- `scripts/mcp_query_client.py`、`scripts/excel_coupling_tool.py`
- `agents/ptm-tde.md`
- `skills/<name>/SKILL.md`（17 个）及各自的 `templates/`
- `rules/CLAUDE.md`、`rules/copilot-instructions.md`
- `doc/INSTALL-MANIFEST.yaml`、`doc/PLATFORM-INSTALL-SPEC.md`
- `delivery/README.md`、`delivery/USER-MANUAL.md`

## 验证方法

1. **仓库护栏**（自动）：运行 `scripts/check_delivery_guardrails.py`，要求零违规。
2. **静态编译**：对 `scripts/*.py` 执行 `python -m py_compile`，确认无语法错误。
3. **安装器 Dry-run**：覆盖 2 平台 × 2 scope 的四个 dry-run 入口，确认输出包含平台、scope、目标根目录、文件列表、规则合并模式。
4. **只读 MCP 三态**：对 `mcp_query_client.py` 执行 `--mock-state success / knowledge_missing / interface_unavailable` 三类入口。
5. **Skill/Agent 静态审查**：按 `process/VALIDATION-ENV.yaml` 的 per-story `focus` 字段，人工比对 SKILL.md 与 HLD v7.1 对应章节。

## 通过标准

- 护栏脚本输出 `[guardrails] OK`
- 所有 `scripts/*.py` py_compile 通过
- 所有 dry-run 命令以 `Dry run completed.` 结束且退出码 0
- `mcp_query_client.py` 在三类 mock 状态下分别返回与契约一致的字段/退出码
- 无 Skill 引用 `.output/` 或 `doc/` 已归档文件作为 SOT
- `agents/ptm-tde.md` 的 Skill 触发词映射覆盖全部 17 个 Skill
- Known limitations 已在 README / USER-MANUAL 中显式披露
