---
status: "draft-for-batch-b-authorization"
version: "1.0"
change_id: "CR-061"
feature_id: "cr061-package-import-layout"
created_at: "2026-06-15T08:41:42+08:00"
created_by: "host-orchestrator"
implementation_allowed: false
---

# CR061 Package / Import / Layout Test Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-15 | host-orchestrator | 新增 Batch B 离线验证计划；所有命令为后续授权后执行。 |

## 1. 验证目标

| 目标 | 成功标准 |
|---|---|
| metadata 收敛 | `pyproject.toml` `[project].name` 为 `quant-lab`，TOML 可解析。 |
| lock 一致 | `uv.lock` 由 `uv lock` 生成，diff 可解释且不手工编辑。 |
| legacy import 不破坏 | `engine`、`market_data`、`strategies`、`trading` 可导入或按现有依赖约束给出明确失败原因。 |
| new namespace 可用 | 若新增 `quant_lab`，`import quant_lab` 必须通过；若新增 alias modules，对应 import smoke 必须通过。 |
| runtime 禁止 | 验证不读取 `.env`、不连接 provider、NAS、lake、QMT/MiniQMT，不写数据。 |

## 2. 验证矩阵

| test_id | 层级 | 对象 | 方法 | 预期 | 当前状态 |
|---|---|---|---|---|---|
| TP-CR061B-001 | static | `pyproject.toml` | Python `tomllib` parse | PASS | PENDING authorization |
| TP-CR061B-002 | lock | `uv.lock` | `uv lock` 后审查 diff | PASS / expected metadata diff | PENDING authorization |
| TP-CR061B-003 | import | legacy roots | pytest smoke imports | PASS 或风险化记录 | PENDING authorization |
| TP-CR061B-004 | import | `quant_lab` | pytest smoke import | PASS if implemented | PENDING authorization |
| TP-CR061B-005 | forbidden runtime | validation command review | 命令不含 provider/lake/runtime | PASS | PENDING authorization |
| TP-CR061B-006 | regression subset | new smoke test | `uv run --python 3.11 pytest -q tests/test_cr061_package_import_layout.py` | PASS | PENDING authorization |
| TP-CR061B-007 | broader regression | existing tests | approved offline subset or full `pytest -q` | PASS / PASS_WITH_RISK | PENDING authorization |

## 3. 授权后候选命令

这些命令只允许在 CP5 Batch B 人工门禁 approved 后执行，并且不得使用 `.env`、provider、NAS、lake、QMT/MiniQMT：

```bash
uv lock
```

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr061_package_import_layout.py
```

```bash
git diff --check
```

如需全量回归，应优先使用离线 pytest；若全量测试会触发 provider/lake/runtime，应缩小为离线 subset 并在 CP7 写明风险。

## 4. 禁止验证

| forbidden_id | 禁止内容 | 原因 |
|---|---|---|
| FT-CR061B-001 | `python -m market_data.cli normalize/publish` | 可能 provider fetch / lake write。 |
| FT-CR061B-002 | 读取 `.env` 或打印 token | 凭据边界。 |
| FT-CR061B-003 | NAS copy / move / delete / archive promote | CR061 不处理 NAS 数据迁移。 |
| FT-CR061B-004 | QMT / MiniQMT 连接、账户查询、下单、撤单 | CR046 暂停且 runtime 未授权。 |
| FT-CR061B-005 | Git remote / push / tag / branch rename / history rewrite | 属于后续 Git cutover CR。 |

## 5. 失败分级

| 失败类型 | 处理 |
|---|---|
| `pyproject.toml` parse 失败 | 立即回退 metadata diff。 |
| `uv lock` 引入非预期依赖变化 | 停止实现，提交 CP6 BLOCKED evidence。 |
| legacy import 失败 | 回退新 namespace / metadata diff，或拆出独立修复 CR。 |
| `quant_lab` alias 触发副作用 | 保留最小 namespace，移除 alias modules。 |
| 任一命令需要凭据/runtime | 停止，另起 runtime authorization gate。 |
