---
story_id: "ST-GB-001"
cr_id: "CR-050"
stage: "CP6"
status: "implemented"
execution_mode: "inline-fallback"
implemented_at: "2026-07-16"
---

# ST-GB-001 Implementation

## 实现对象

- `meta_flow/workflow/git_branch_lifecycle.py`：新增 typed intent、repository target、ref snapshot、operation authorization、deterministic plan、per-repository outcome 与 paired branch-open planner/executor。
- `meta_flow/workspace/git_sync.py`：新增无 shell、带 timeout 的 typed Git runner、remote symbolic HEAD/exact ref probe 与本地 repo fingerprint；旧 `push_workspace` 默认行为不变。
- `meta_flow/workflow/cr_lifecycle.py`：注册显式 `meta-flow cr branch-open` 命令族入口；既有 `cr bootstrap` 不产生新的隐式远端写。
- `tests/test_git_branch_lifecycle.py`：用临时 bare remote 覆盖 branch naming、dry-run 零 mutation、2/2 exact base/upstream、dirty preflight fail-closed。

## 契约映射

Open 先对全部 routed repository 执行只读观察和 typed authz/OID 校验；actual 才按 argv-only `fetch --prune`、`pull --ff-only`、`switch -c`、`push -u` 执行。任一全局 preflight 失败不会创建 local/remote CR ref；写入阶段失败保留逐仓事实并返回 `PARTIAL`，不执行 reset/force/自动清理。

新 CR 的 coordinator 只在 local branch prepare 后调用既有 bootstrap writer。bootstrap 产物保持未提交，工具不 stage/commit；操作者必须在 publish 前显式审查并提交。已存在 formal CR 时不重复 bootstrap。

## 验证与边界

- Open/branch contract fixtures：4 passed，7 deselected。
- 既有 workspace Git 与 CR lifecycle 回归：36 passed。
- touched-path Ruff 与 `git diff --check`：PASS。
- 测试只写 pytest 临时目录中的 bare remote；没有调用当前 project/artifact `origin`，没有 commit/push 当前仓库，没有读取凭据。

## 设计差异与交接

无设计差异。CLI actual 强制 append-only `--output` 与 typed `--authorization`；dry-run 可无授权且只输出计划。inline fallback 是用户授权的执行模式，不构成独立 meta-dev 或真实平台远端证明。
