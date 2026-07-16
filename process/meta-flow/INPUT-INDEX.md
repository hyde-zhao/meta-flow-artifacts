# Input Index

| 输入 ID | 类型 | 来源 | 关联 CR | 用途 | 状态 |
|---|---|---|---|---|---|
| IN-ROOT-001 | historical request | `process/REQUEST.md` 初始请求 | CR-037..047 | Meta Flow 自我开发长期背景 | active-reference |
| IN-CR050-001 | user request | 2026-07-15 当前对话，已摘录到 `process/REQUEST.md#cr-050-增量请求2026-07-15` | CR-050 | Git branch lifecycle 场景、需求与 CP2 决策来源 | accepted |
| IN-CR050-002 | repository evidence | `meta_flow/workspace/git_sync.py`、`meta_flow/cli.py`、`tests/test_workspace_git_sync.py` | CR-050 | 识别既有原生 Git 集成边界 | read-only-audited |
| IN-CR050-003 | official contracts | `https://git-scm.com/docs/git-switch`、`git-pull`、`git-fetch`、`git-branch`、`git-merge-base`、`git-push` | CR-050 | Git 命令与失败语义依据 | read-only-audited |
| IN-CR050-004 | external alternative | `https://www.git-town.com/basic-commands.html`、`https://www.git-town.com/commands/ship.html` | CR-050 | Git Town 方案与切换条件评估 | read-only-audited |

## 边界

- 不读取 credential、secret 或 forge token。
- 不把 shell alias `gb` 当作稳定外部工具契约。
- 不处理 `process/quant-lab/**` 或 `/home/hyde/workspace/meta-flow.process-prelink-backup-20260713T100930`。
