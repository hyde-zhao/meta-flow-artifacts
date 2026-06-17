---
context_id: NEXT-SESSION-CR073-STARTUP-CONTEXT
created_at: '2026-06-16T09:29:09+08:00'
created_by: host-orchestrator
recommended_next_cr: CR-073
recommended_title: "MIG-B Target Root Convergence / Cutover Readiness Gate"
status: ready-for-next-session
read_profile: compact
---

# Next Session Context: CR073 / MIG-B

## 推荐下一步

建议启动 `CR073 / MIG-B：Target Root Convergence / Cutover Readiness Gate`。

不要直接做 root cutover，也不要删除旧根。当前更稳妥的下一步是先把 `/home/hyde/workspace/quant-lab` 收敛为可审计、可验证的候选工作根：补齐 CR071 CP8 / CR072 后续证据和 CR tracking ledger，确认 target 没有排除项残留，再决定是否进入真正 root cutover。

## 新会话可直接使用的启动语句

```text
启动 CR073 / MIG-B：目标根收敛与 cutover readiness gate。基于 CR071 已关闭的 copy-first shadow root 和 CR072 NAS 归档结果，在不切换日常工作根、不删除/重命名/移动旧根、不复制 reports/、data/、.venv/、node_modules/、.env、不读取凭据、不做远端 Git 写入、不切 data lake、不触发 QMT/MiniQMT runtime、不恢复 CR046 的前提下，先对 /home/hyde/workspace/quant-lab 做 target evidence convergence：

1. 扫描 /home/hyde/workspace/local_backtest 与 /home/hyde/workspace/quant-lab 的 CR071/CR072 证据差异；
2. 生成 CR073 CP2/CP3/CP5 门禁，确认 scoped local sync 方案；
3. CP2/CP3/CP5 通过后，才允许把 CR071 CP8、CR072、STATE、CR-INDEX 和 release/context/checkpoint 证据从旧根同步到 target；
4. 同步必须排除 .git、reports/、data/、.venv/、node_modules/、.env，必须先 dry-run 和冲突分类，不使用 --delete；
5. 验证 target 具备最新 CR tracking / CP8 证据，且 target 对 reports data .venv node_modules .env 的 git status、git ls-files 和根目录查找均为空；
6. CR073 只做到 cutover readiness，不做真正 root cutover。root cutover、target env rebuild、data/reports access、old root retirement、remote Git governance 继续作为后续 CR。
```

## 当前事实快照

| 项目 | 事实 |
|---|---|
| 当前旧根 | `/home/hyde/workspace/local_backtest` |
| 目标根 | `/home/hyde/workspace/quant-lab` |
| 当前 canonical project name | `quant-lab` |
| legacy alias | `local_backtest` |
| active formal CR | `CR-046`，但用户暂停，不恢复 |
| 最近关闭 | `CR071` copy-first shadow root；`CR072` NAS archive local assets |
| CR tracking check | `python3 -B scripts/check_cr_tracking_consistency.py --project-root .` PASS |
| source current head | `45a426d` |
| target current head | `2aaf21b` |
| remote master read-only | `3ade165e8b1edad031a911490cf6c1cee942616e` |
| target 排除项 | `reports data .venv node_modules .env` 的 `git status` / `git ls-files` / 根目录查找均为空 |
| target 缺口 | 当前 target 未包含新生成的 CR071 CP8 关闭证据；`test -e /home/hyde/workspace/quant-lab/process/checkpoints/CP8-CR071-DELIVERY-READINESS.md` 返回 false |

## 关键边界

| 类型 | 边界 |
|---|---|
| 允许候选 | 本机 scoped evidence sync、dry-run、差异扫描、静态验证、CR tracking 一致性检查 |
| 不允许 | root cutover、删除/重命名/移动旧根、`--delete` 同步、复制 `.git`、复制 `.env`、读取凭据、复制或恢复 `reports/` / `data/`、重建 `.venv` |
| 外部系统 | 不做远端 Git 写入、不做 NAS 新写入或删除、不做 data lake root switch、不做 provider fetch / lake write / catalog publish |
| runtime | 不连接 QMT/MiniQMT，不查账户，不下单，不 simulation/live |
| CR046 | 保持 paused，除非用户明确说恢复 CR046 |

## 必读文件

新会话启动后优先读这些文件，不要读全量历史：

| 优先级 | 路径 | 用途 |
|---|---|---|
| must | `AGENTS.md` | 项目规则、语言、memory、CR 门禁 |
| must | `process/STATE.md` | 当前 active/closed CR 与 next_action |
| must | `process/changes/CR-INDEX.yaml` | CR tracking 索引 |
| must | `process/context/CP8-CR071-DELIVERY-CONTEXT.yaml` | CR071 关闭上下文 |
| must | `process/release/RELEASE-CONTEXT-CR071.yaml` | CR071 release scope / non-authorized items / follow-up |
| must | `docs/release/FEEDBACK-CR071.md` | CR071 follow-up candidates |
| must | `process/changes/CR-072-NAS-ARCHIVE-LOCAL-ASSETS-2026-06-16.md` | CR072 归档事实与边界 |
| must | `docs/release/LOCAL-ASSETS-NAS-ARCHIVE-MANIFEST-CR072.yaml` | NAS 归档 manifest |
| must | `process/checkpoints/CP8-CR071-DELIVERY-READINESS.md` | CP8 DQ 和不授权项 |
| if needed | `docs/release/MIGRATION-CR071.md` | migration 兼容性和后续项 |
| if needed | `docs/release/ROLLBACK-CR071.md` | 回退路径 |
| if needed | `docs/quality/TEST-REPORT-CR071.md` / `docs/quality/REVIEW-CR071.md` | CP7 风险与 CP8 缓解状态 |

## 不默认读取

- 不读取完整历史 `process/checkpoints/CP2..CP8` 全部旧 CR 文件。
- 不读取完整 `reports/`、`data/`。
- 不读取 `.env`、token、cookie、私钥、账户或交易事实。
- 不读取完整对话历史。
- 不把 `CR046` 当作默认恢复目标。

## CR073 建议范围

### In Scope

- 创建正式 `CR-073-*` 文件。
- 生成 `CP2 / CP3 / CP5` 门禁和 context capsule。
- 比对 source / target 中以下证据路径的差异：
  - `process/STATE.md`
  - `process/changes/CR-INDEX.yaml`
  - `process/changes/CR-071-*`
  - `process/changes/CR-072-*`
  - `process/context/CP8-CR071-*`
  - `process/release/RELEASE-CONTEXT-CR071.yaml`
  - `process/checks/CP8-CR071-*`
  - `process/checkpoints/CP8-CR071-*`
  - `docs/release/*-CR071.md`
  - `docs/release/*-CR072.yaml`
  - `docs/quality/*-CR071.md`
  - `docs/quality/*-CR072.md`
- CP2/CP3/CP5 通过后执行 scoped local sync。
- 验证 target evidence convergence。

### Out Of Scope

- 真正 root cutover。
- 删除旧根或删除 target。
- 复制 `.git`。
- 复制 `.env`、`reports/`、`data/`、`.venv/`、`node_modules/`。
- target 环境重建或 full pytest。
- 远端 Git 写入。
- NAS 新写入 / 删除 / promote。
- data lake / provider / catalog。
- QMT/MiniQMT runtime。
- CR046 recovery。

## CR073 推荐验收标准

| 验收项 | 预期 |
|---|---|
| CR tracking | `python3 -B scripts/check_cr_tracking_consistency.py --project-root .` PASS |
| target CP8 evidence | `/home/hyde/workspace/quant-lab/process/checkpoints/CP8-CR071-DELIVERY-READINESS.md` 存在 |
| target release context | `/home/hyde/workspace/quant-lab/process/release/RELEASE-CONTEXT-CR071.yaml` 存在 |
| target CR index | target 中 `CR-INDEX.yaml` 记录 CR071 / CR072 closed，CR046 paused |
| target excluded paths | `git -C /home/hyde/workspace/quant-lab status --short -- reports data .venv node_modules .env` 为空 |
| target tracked excluded paths | `git -C /home/hyde/workspace/quant-lab ls-files reports data .venv node_modules .env` 为空 |
| filesystem excluded paths | target 根目录没有 `reports data .venv node_modules .env` |
| remote write | 未执行 |
| destructive action | 未执行 |

## 推荐后续顺序

| 顺序 | 候选 | 说明 |
|---|---|---|
| 1 | `CR073 / MIG-B target root convergence + cutover readiness` | 先补齐 target 证据，避免双根审计分叉。 |
| 2 | `CR074 root cutover / daily workflow switch` | 只有 CR073 通过后，再决定是否把日常工作根切到 target。 |
| 3 | `CR075 target env rebuild and smoke` | 若需要在 target 运行测试，再单独授权 `.venv` 重建和 smoke。也可与 CR074 合并，但需要明确授权。 |
| 4 | `CR076 data/reports access policy` | 设计 target 使用本地保留副本、NAS、挂载或重建方式。 |
| 5 | `CR077 old root retirement` | 只有 target 成为 authoritative root 且恢复路径确认后才考虑。 |

## 不推荐作为下一步

| 候选 | 原因 |
|---|---|
| 直接 root cutover | target 缺少 CR071 CP8 / CR072 后续证据，且未做 authoritative root 判定。 |
| old root 删除 / 退休 | 破坏性操作，且 target 尚未通过 cutover readiness。 |
| target env rebuild | 可以做，但应在 target evidence convergence 后再授权，避免运行态动作掩盖证据分叉。 |
| remote Git governance / push | 外部状态写入，和本机 root 收敛不是同一风险面。 |
| CR046 recovery | CR046 是 trading/runtime 线，用户已暂停，不应被迁移线自动恢复。 |

## 启动后第一批命令建议

这些命令只读或静态检查，不执行写入：

```bash
python3 -B scripts/check_cr_tracking_consistency.py --project-root .
test -d /home/hyde/workspace/local_backtest
test -d /home/hyde/workspace/quant-lab
test -e /home/hyde/workspace/quant-lab/process/checkpoints/CP8-CR071-DELIVERY-READINESS.md
git -C /home/hyde/workspace/quant-lab status --short -- reports data .venv node_modules .env
git -C /home/hyde/workspace/quant-lab ls-files reports data .venv node_modules .env
git ls-remote git@github.com:hyde-zhao/quant-lab.git refs/heads/master
```

同步命令必须等 CR073 CP2/CP3/CP5 approved 后再执行，且要先 dry-run。

