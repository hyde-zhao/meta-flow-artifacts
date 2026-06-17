---
status: "implemented-cp6-ready"
version: "1.0"
story_id: "CR061B"
story_slug: "package-import-layout-batch-b"
feature_id: "cr061-package-import-layout"
implementation_type: "mixed"
source_story: "docs/design/FEATURE-DESIGN-MATRIX-CR061.md"
source_design_evidence: "docs/features/cr061-package-import-layout/DESIGN.md"
created_by: "host-orchestrator-inline-fallback"
created_at: "2026-06-15T08:55:37+08:00"
updated_at: "2026-06-15T08:55:37+08:00"
---

# Implementation: CR061 Batch B Package / Import / Layout

## 1. 实现摘要

| 项目 | 内容 |
|---|---|
| 实现目标 | 将 repo-local distribution metadata 收敛为 `quant-lab`，新增 `quant_lab` 兼容命名空间，并保留 legacy import roots。 |
| 行为变化 | `pyproject.toml` / `uv.lock` 中项目名为 `quant-lab`；`import quant_lab` 和 `import quant_lab.engine` 等兼容 alias 可用；`engine` / `market_data` / `strategies` / `trading` 仍可直接导入。 |
| 范围边界 | 未移动、重命名、删除 legacy roots；未批量改 import；未执行 Git remote、NAS/data lake/provider/runtime/凭据/CR046 recovery。 |
| CP6 证据 | `process/checks/CP6-CR061-BATCH-B-PACKAGE-IMPORT-LAYOUT-CODING-DONE.md` |

## 2. 上游设计引用

| 来源 | 路径 / ID | 本次消费内容 |
|---|---|---|
| CP5 Batch B gate | `process/checkpoints/CP5-CR061-BATCH-B-IMPLEMENTATION-AUTHORIZATION.md` | 用户批准窄实现范围和不授权项。 |
| Feature Matrix | `docs/design/FEATURE-DESIGN-MATRIX-CR061.md` | Story / LLD 策略、设计证据覆盖和执行边界。 |
| Feature Design | `docs/features/cr061-package-import-layout/DESIGN.md` | compatibility-first 设计、集成契约、失败路径。 |
| Test Plan | `docs/features/cr061-package-import-layout/TEST-PLAN.md` | 离线 import smoke 和静态验证范围。 |
| Candidate Manifest | `docs/release/CR061-BATCH-B-CANDIDATE-MANIFEST.yaml` | BATCH-B-ACT-001..005 和 blocked action。 |
| Dirty Worktree Policy | `docs/release/CR061-BATCH-B-DIRTY-WORKTREE-INCLUDE-EXCLUDE.yaml` | include/exclude 文件边界。 |

## 3. 实现前置检查

| 检查项 | 结果 | 证据 / 说明 |
|---|---|---|
| 上游 Feature 设计存在 | PASS | `docs/features/cr061-package-import-layout/DESIGN.md` |
| Story 范围明确 | PASS | `docs/design/FEATURE-DESIGN-MATRIX-CR061.md` |
| 待确认问题已关闭 | PASS | 用户于 2026-06-15T08:55:37+08:00 回复“同意”，CP5 Batch B approved。 |
| 影响范围可定位 | PASS | 仅 `pyproject.toml`、`uv.lock`、`quant_lab/**`、`tests/test_cr061_package_import_layout.py` 和 CR061 evidence。 |
| 验证方式明确 | PASS | smoke pytest、py_compile、pyproject parse、diff check。 |
| 当前 Wave / dev_gate 满足 | PASS | Batch B gate approved；CR046 仍暂停且未恢复。 |
| 文件所有权无冲突 | PASS | 未修改 legacy roots 内容。 |

## 4. 实现对象清单

| 对象 | 类型 | 目标 | 是否必须 | 验证方式 |
|---|---|---|---|---|
| `pyproject.toml` | code metadata | `[project].name` 收敛为 `quant-lab` | yes | TOML parse / pytest |
| `uv.lock` | lock metadata | 由 `uv lock` 再生成项目包名 | yes | `uv lock` 输出 / diff review |
| `quant_lab/__init__.py` | code | 提供最小新命名空间 | yes | import smoke |
| `quant_lab/engine.py` | code | alias 到 legacy `engine` root | yes | import smoke |
| `quant_lab/market_data.py` | code | alias 到 legacy `market_data` root | yes | import smoke |
| `quant_lab/strategies.py` | code | alias 到 legacy `strategies` root | yes | import smoke |
| `quant_lab/trading.py` | code | alias 到 legacy `trading` namespace root | yes | import smoke |
| `tests/test_cr061_package_import_layout.py` | guardrail-test | 锁定 metadata、legacy roots 和 compat aliases | yes | pytest |

## 5. 设计契约映射

| 设计要求 | 来源 | 实现位置 | 实现动作 | 验证 |
|---|---|---|---|---|
| distribution metadata 使用 `quant-lab` | DQ-CP5-CR061B-01 | `pyproject.toml` | modify | `PASS pyproject name quant-lab` |
| lock 只能由 uv 生成 | DESIGN §3 | `uv.lock` | `uv lock` regenerate | `Resolved 149 packages... Removed local-backtest... Added quant-lab...` |
| 保留 legacy roots | DESIGN §3 / DQ-CP5-CR061B-04 | `engine/**`、`market_data/**`、`strategies/**`、`trading/**` | keep | smoke test imports all roots |
| 新增 `quant_lab` 兼容入口 | DQ-CP5-CR061B-01 | `quant_lab/**` | create alias modules | smoke test imports alias modules |
| 禁止 external/runtime 行为 | DQ-CP5-CR061B-04 | validation commands | no provider/lake/runtime commands | only offline pytest / parse / diff |

## 6. 单元测试 / Fixture 计划

| 测试对象 | 测试类型 | 输入 / Fixture | 期望 | 覆盖风险 | 状态 |
|---|---|---|---|---|---|
| project metadata | unit | `pyproject.toml` | `project.name == "quant-lab"` | lock/name mismatch | passed |
| `quant_lab` namespace | unit | Python import | import succeeds and version stable | namespace missing | passed |
| legacy roots | smoke | `engine` / `market_data` / `strategies` / `trading` | direct imports succeed | compatibility break | passed |
| alias modules | smoke | `quant_lab.<root>` imports | alias is same module object as legacy root | duplicate module state | passed |

## 7. 最小实现切片

| Slice ID | 对应设计契约 | 改动对象 | 输出文件 | 局部验证 | 状态 |
|---|---|---|---|---|---|
| IMPL-S1 | metadata convergence | metadata / lock | `pyproject.toml`、`uv.lock` | `uv lock` + pyproject parse | done |
| IMPL-S2 | compatibility namespace | code | `quant_lab/*.py` | py_compile + import smoke | done |
| IMPL-S3 | regression smoke | tests | `tests/test_cr061_package_import_layout.py` | pytest 4 passed | done |

## 8. 变更说明

### 8.1 代码变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `pyproject.toml` | modify | `[project].name` 从 `local-backtest` 改为 `quant-lab`。 |
| `uv.lock` | modify | `uv lock` 再生成，本项目包名从 `local-backtest` 改为 `quant-lab`。 |
| `quant_lab/__init__.py` | create | 新增最小 compat namespace。 |
| `quant_lab/engine.py` | create | alias 到 legacy `engine` root。 |
| `quant_lab/market_data.py` | create | alias 到 legacy `market_data` root。 |
| `quant_lab/strategies.py` | create | alias 到 legacy `strategies` root。 |
| `quant_lab/trading.py` | create | alias 到 legacy `trading` namespace root。 |

### 8.2 Prompt / Skill 变更

N/A，本批次未修改 Prompt / Skill。

### 8.3 模板 / Schema 变更

N/A，本批次未修改模板 / Schema。

### 8.4 Guardrail / 测试变更

| 文件 / 命令 | 动作 | 说明 |
|---|---|---|
| `tests/test_cr061_package_import_layout.py` | create | 新增离线 package/import smoke test。 |
| `uv run --python 3.11 pytest -q tests/test_cr061_package_import_layout.py` | run | 4 passed。 |

### 8.5 文档变更

| 文件 | 动作 | 说明 |
|---|---|---|
| `docs/features/cr061-package-import-layout/IMPLEMENTATION.md` | create | 本实现交接说明。 |
| `process/checks/CP6-CR061-BATCH-B-PACKAGE-IMPORT-LAYOUT-CODING-DONE.md` | create | CP6 编码完成证据。 |

## 9. 平台差异处理

| 平台 | 检查项 | 预期 | 结果 |
|---|---|---|---|
| Claude Code | direct ask agent 有 `AskUserQuestion` | n/a | N/A |
| Claude Code | non-direct agent 不声明 `AskUserQuestion` | n/a | N/A |
| Codex | 不写 Claude-only `tools` schema | yes | PASS |
| Codex | 无 `request_user_input` 时可降级 | n/a | N/A |
| install | dry-run 可执行 | n/a | N/A |

## 10. 验证结果

| 命令 / 检查 | 结果 | 证据 |
|---|---|---|
| `uv lock` | PASS | `Resolved 149 packages... Removed local-backtest v0.1.0; Added quant-lab v0.1.0` |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr061_package_import_layout.py` | PASS | `4 passed in 0.03s` |
| `PYTHONPYCACHEPREFIX=/tmp/cr061-batch-b-pycompile uv run --python 3.11 python -m py_compile ...` | PASS | exit code 0 |
| `uv run --python 3.11 python -c "... pyproject ..."` | PASS | `PASS pyproject name quant-lab` |
| `git diff --check` | PASS | exit code 0 |

## 11. 未覆盖项

| 未覆盖内容 | 原因 | 后续处理 |
|---|---|---|
| physical relayout to `src/quant_lab/**` | 明确不授权 bulk move / rename。 | 后续独立 CR / gate。 |
| bulk import rewrite | 高风险且未授权。 | 后续独立 CR / dry-run。 |
| Git remote / push | 明确不授权。 | 后续 CR062 或等价 Git cutover gate。 |
| NAS / data lake / provider / runtime / credentials | 明确不授权。 | 后续 CR063+ 或 runtime_authorization gate。 |
| Full pytest | 当前 Batch B 授权为离线 import smoke 和必要 subset；全量可能触发外部依赖。 | CP7 可按风险决定是否扩大离线 subset。 |

## 12. 风险与回滚

| Risk ID | 风险 | 影响 | 缓解 | 回滚 / 切换条件 |
|---|---|---|---|---|
| R-CR061B-01 | `quant_lab.market_data` import 继承 legacy `market_data` 初始化成本 | import smoke 可能加载轻量依赖 | 当前测试通过；不执行 CLI normalize/publish | 若后续发现副作用，改为 lazy alias 或只保留最小 namespace。 |
| R-CR061B-02 | `uv.lock` 包段顺序变化 | diff 显示删除旧包段并新增新包段 | diff review 确认依赖字段一致 | 若 lock 出现非预期依赖变化，恢复 lock 并重跑 `uv lock`。 |
| R-CR061B-03 | Full pytest 未运行 | 非 import 行为回归未完全覆盖 | CP7 记录风险，可追加安全离线 subset | 若用户要求 full regression，先确认不会触发 external/runtime。 |

## 13. 设计缺口反馈

| Gap ID | 发现阶段 | 问题 | 应反馈到 | 是否阻塞 | 推荐处理 |
|---|---|---|---|---|---|
| GAP-CR061B-01 | implementation | `trading` 是 namespace package，无 `__init__.py`；当前 alias 只做 package-level import，不验证子模块。 | 后续 relayout CR | no | 后续 bulk relayout 前单独设计 trading package boundary。 |

## 14. QA / Review / Doc 后续交接

### QA 关注点

- 重点验证 `quant_lab.*` alias 不造成重复 module state。
- 保持 `market_data.cli` 现有入口，不执行 normalize/publish。
- 不恢复 CR046，也不进入任何 QMT/MiniQMT runtime。

### Review 关注点

- `sys.modules[__name__] = legacy_module` 是有意的 alias 策略，用于保证 `quant_lab.engine is engine`。
- `uv.lock` diff 应仅为虚拟项目包名变化。

### Doc 关注点

- README / USER-MANUAL 当前无需更新；用户命令仍可使用 legacy roots。
