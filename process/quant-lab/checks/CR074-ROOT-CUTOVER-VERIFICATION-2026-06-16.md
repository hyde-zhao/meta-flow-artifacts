---
check_id: "CR074-ROOT-CUTOVER-VERIFICATION"
cr_id: "CR-074"
type: "verification-evidence"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-16T22:51:21+08:00"
checked_at: "2026-06-16T22:51:21+08:00"
source_root: "/home/hyde/workspace/local_backtest"
target_root: "/home/hyde/workspace/quant-lab"
authoritative_root: "/home/hyde/workspace/quant-lab"
legacy_root: "/home/hyde/workspace/local_backtest"
---

# CR074 Root Cutover Verification

## Verification Summary

| 检查项 | 结果 | 证据 | 说明 |
|---|---|---|---|
| local CR tracking | PASS | `python3 -B scripts/check_cr_tracking_consistency.py --project-root .` | 输出 `CR tracking consistency: PASS`。 |
| target CR tracking | PASS | `python3 -B /home/hyde/workspace/quant-lab/scripts/check_cr_tracking_consistency.py --project-root /home/hyde/workspace/quant-lab` | 输出 `CR tracking consistency: PASS`。 |
| target root authority marker | PASS | `rg root_authority /home/hyde/workspace/quant-lab/process/STATE.md /home/hyde/workspace/quant-lab/process/changes/CR-INDEX.yaml` | target STATE / CR-INDEX 均包含 `authoritative_root: /home/hyde/workspace/quant-lab`、`legacy_root: /home/hyde/workspace/local_backtest` 和 `approved_gate: CP2/CP3/CP5-CR074`。 |
| CR074 status | PASS | target STATE / CR-INDEX / formal CR | `status: active-logical-cutover-achieved`。 |
| scoped evidence equality | PASS | 逐文件 `cmp` 白名单 | 无 `DIFF` 输出。 |
| old root retained | PASS | `test -d /home/hyde/workspace/local_backtest` | old root 仍存在。 |
| target root present | PASS | `test -d /home/hyde/workspace/quant-lab` | target root 仍存在。 |
| target forbidden root find | PASS | `find /home/hyde/workspace/quant-lab -maxdepth 1 ...` | 空输出。 |
| target forbidden git status | PASS | `git -C /home/hyde/workspace/quant-lab status --short -- reports data .venv node_modules .env` | 空输出。 |
| target forbidden git ls-files | PASS | `git -C /home/hyde/workspace/quant-lab ls-files -- reports data .venv node_modules .env` | 空输出。 |

## Target Marker Evidence

`rg` 在 target 中返回：

```text
/home/hyde/workspace/quant-lab/process/changes/CR-INDEX.yaml:98:root_authority:
/home/hyde/workspace/quant-lab/process/changes/CR-INDEX.yaml:100:  authoritative_root: /home/hyde/workspace/quant-lab
/home/hyde/workspace/quant-lab/process/changes/CR-INDEX.yaml:101:  legacy_root: /home/hyde/workspace/local_backtest
/home/hyde/workspace/quant-lab/process/changes/CR-INDEX.yaml:103:  approved_gate: CP2/CP3/CP5-CR074
/home/hyde/workspace/quant-lab/process/changes/CR-INDEX.yaml:200:  status: active-logical-cutover-achieved
/home/hyde/workspace/quant-lab/process/STATE.md:15:root_authority:
/home/hyde/workspace/quant-lab/process/STATE.md:17:  authoritative_root: "/home/hyde/workspace/quant-lab"
/home/hyde/workspace/quant-lab/process/STATE.md:18:  legacy_root: "/home/hyde/workspace/local_backtest"
/home/hyde/workspace/quant-lab/process/STATE.md:108:    status: active-logical-cutover-achieved
/home/hyde/workspace/quant-lab/process/STATE.md:1270:  approved_gate: CP2/CP3/CP5-CR074
```

## Forbidden Path Verification

| 检查命令 | 结果 |
|---|---|
| `find /home/hyde/workspace/quant-lab -maxdepth 1 \( -name reports -o -name data -o -name .venv -o -name node_modules -o -name .env \) -print` | 空输出 |
| `git -C /home/hyde/workspace/quant-lab status --short -- reports data .venv node_modules .env` | 空输出 |
| `git -C /home/hyde/workspace/quant-lab ls-files -- reports data .venv node_modules .env` | 空输出 |

## Boundary Confirmation

| 边界 | 结果 |
|---|---|
| 当前会话 cwd 自动切换 | 未执行 |
| shell / IDE / cron / service / systemd 修改 | 未执行 |
| 旧根删除 / 重命名 / 移动 | 未执行 |
| `reports/`、`data/`、`.venv/`、`node_modules/`、`.env` 复制 / 读取 / 比对 | 未执行 |
| 凭据读取 / 打印 / 迁移 | 未执行 |
| target env rebuild / dependency sync / smoke runtime | 未执行 |
| 远端 Git 写入 / branch / tag / default branch governance | 未执行 |
| NAS 新写入 / 删除 / promote | 未执行 |
| data lake / provider fetch / lake write / catalog publish | 未执行 |
| QMT / MiniQMT runtime / 交易动作 | 未执行 |
| CR046 recovery | 未执行 |

## Conclusion

- 结论：PASS
- CR074 状态：`active-logical-cutover-achieved`
- authoritative root：`/home/hyde/workspace/quant-lab`
- legacy root：`/home/hyde/workspace/local_backtest`
- 后续候选：CR075 target env rebuild、CR076 data/reports access policy、CR077 old root retirement policy、CR078 remote Git governance；均未自动启动。
