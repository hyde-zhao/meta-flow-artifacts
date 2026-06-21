---
status: draft
version: "1.0"
feature_id: "F-CR093-01"
source_cr: "CR-093"
validation_mode: "static-ledger-and-unit-tests-only"
created_at: "2026-06-18T17:55:00+08:00"
owner: "host-orchestrator"
---

# CR093 Ledger Hygiene TEST-PLAN

## 测试目标

验证 CR093 的实现能消除 CR019 / CR025 历史旧账阻断，同时不隐藏真实当前状态冲突。

## 测试矩阵

| 测试 ID | 场景 | 输入 | 预期结果 | 类型 |
|---|---|---|---|---|
| TP-CR093-01 | active formal CR 识别 | 当前 CR093 active | summary 显示 `active formal CRs: CR-093` | static |
| TP-CR093-02 | CR025 history 误扫 | history 中存在 `active_change: CR-025` 且 formal closed | audit warning，不 exit 1 | unit / fixture |
| TP-CR093-03 | CR019 closed 等价 | CR020/030/040..045 formal status 与 tracking status 字面不同但语义 closed | 不报 mismatch | unit / fixture |
| TP-CR093-04 | cancelled-user-deleted 等价 | CR021..024 tracking 为 cancelled-user-deleted | 分类为 cancelled，不报 invalid | unit / fixture |
| TP-CR093-05 | 真实 current conflict | 顶层 active_change 指向 formal closed CR | exit 1 | unit / fixture |
| TP-CR093-06 | warning-only 范围 | cp8-follow-up missing tracking row | warning-only，不阻断 CR019 / CR025 目标 | static |
| TP-CR093-07 | py_compile | checker 脚本 | 编译通过 | static |
| TP-CR093-08 | workspace health | process symlink | `process_link_health: ok` | static |

## 推荐验证命令

```bash
uv run --python 3.11 python -m py_compile scripts/check_cr_tracking_consistency.py
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr093_cr_tracking_consistency.py
uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab
uv run --python 3.11 meta-flow check cr-tracking --project-root .
```

## 拒收条件

- checker 通过 suppress 已知错误但没有保留真实 current conflict 失败能力。
- 删除 STATE history 或 CR019 历史追溯文本。
- 访问 NAS、凭据、账户、runtime、data/reports 或 provider/lake/publish。
