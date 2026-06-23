# STORY-018 P2 Implementation Evidence

## Scope

CR-018 P2 整改覆盖 P1 留下的三个工程化缺口：resource content hash 漂移检查、Gate 字段级机器基线、正式自动化测试沉淀。

## Implementation Objects

| Object | Files | Change |
|---|---|---|
| Installer / manifest | `script/ptm_team/install.py`, `script/install.py` | resource entry 记录 `source_files / resource_files / source_hash / installed_hash`；`check` 对 resource 执行内容漂移检查 |
| Checkpoint guardrail | `skills/checkpoint-manager/scripts/run_checkpoint.py` | 新增字段级检查函数；GATE-2/3/4 从“文件存在”升级为“关键字段存在且结构上可消费” |
| Docs / Skill contract | `docs/ptm-tde/README.md`, `docs/ptm-tde/USER-MANUAL.md`, `docs/ptm-tde/gate-spec.md`, `skills/checkpoint-manager/SKILL.md`, `script/README.md` | 文档同步 resource drift 与字段级 machine-baseline 语义 |
| Tests | `tests/test_cr018_p2.py` | 新增 unittest，覆盖 resource drift、GATE-3 字段缺失阻断、GATE-4 PC 16 列阻断 |

## Contract Mapping

| Contract | Implementation |
|---|---|
| resource 安装后可诊断漂移 | manifest 记录每个 resource 的文件列表和 hash；`check_installed_drift()` 不再跳过 resource |
| Gate 不再只看目录/文件存在 | `add_required_fields_check()` 和 `add_table_columns_check()` 接入 GATE-2/3/4 |
| 自动检查不替代人工语义确认 | 字段级检查仍标记为 machine-baseline；manual Gate 草稿继续承载语义确认 |
| 临时 fixture 固化 | `uv run python -m unittest tests.test_cr018_p2` 成为正式回归入口 |

## Verification Results

| Check | Result |
|---|---|
| Python syntax | PASS：`uv run python -m compileall script/ptm_team/install.py script/install.py skills/checkpoint-manager/scripts/run_checkpoint.py tests/test_cr018_p2.py` |
| P2 unittest | PASS：`uv run python -m unittest tests.test_cr018_p2`，3 tests |
| Covered fixture 1 | resource 文件被篡改后 `check codex --agent ptm-tde` 返回 2，并报告 `INSTALLED_DRIFT resource:` |
| Covered fixture 2 | GATE-3 的 M 分析缺 CAE 字段时返回 2，并在 Gate 文件写入 `M1: M 分析 CAE/trace 字段完整` BLOCKING |
| Covered fixture 3 | GATE-4 的 PC 文件缺 16 列 Markdown 表格时返回 2，并在 Gate 文件写入 `P2: PC 16 列格式检查` BLOCKING |

## Remaining Risk

process 文件提交策略仍需团队层面决定：当前 `process/*` 在仓库中被 ignore，适合作为本地恢复与审计记录；若需要纳入正式交付，应单独修改 `.gitignore` 或导出到非 ignored 路径。此项是仓库治理策略，不阻塞 CR-018 P2 工程整改。
