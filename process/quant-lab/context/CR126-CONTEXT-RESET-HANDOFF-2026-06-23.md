---
context_id: "CR126-CONTEXT-RESET-HANDOFF-2026-06-23"
source_cr: "CR-126"
source_checkpoint:
  - "process/checkpoints/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-REVIEW.md"
  - "process/checkpoints/CP5-CR126-RUNNER-ENTRY-DESIGN-AUTHORIZATION-REVIEW.md"
status: "ready-for-context-reset"
created_at: "2026-06-23T14:11:38+08:00"
owner: "host-orchestrator"
read_profile: "minimal"
active_cr: "CR-126"
recommended_next_action: "resume-cr126-human-gate-or-reconcile-condensed-runner-plan"
---

# CR126 Context Reset Handoff

## 用途

本文件用于清空上下文后恢复当前工作。恢复后先读取本文件，再读取最小必要文件；不要依赖旧对话历史，也不要自动启动 CR127 或任何 runtime / NAS / QMT / trading 相关 CR。

## 0. 恢复后第一步

建议先运行只读/静态检查：

```bash
uv run --python 3.11 meta-flow check cr-tracking --project-root /home/hyde/workspace/quant-lab
git status --short
git -C /home/hyde/workspace/meta-flow-artifacts status --short
```

这些命令不读取凭据、不启动 runtime、不访问 NAS。

随后读取最小上下文：

1. `process/context/CR126-CONTEXT-RESET-HANDOFF-2026-06-23.md`
2. `process/changes/CR-126-QUANT-LAB-RUNNER-DEVELOPMENT-READINESS-RUNNER-ENTRY-DESIGN-2026-06-23.md`
3. `process/checkpoints/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-REVIEW.md`
4. `process/checkpoints/CP5-CR126-RUNNER-ENTRY-DESIGN-AUTHORIZATION-REVIEW.md`
5. `process/context/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-CONTEXT.yaml`
6. `process/context/CP5-CR126-RUNNER-ENTRY-DESIGN-AUTHORIZATION-CONTEXT.yaml`
7. `process/changes/CR-INDEX.yaml`
8. `process/STATE.md`，仅用于 legacy 状态确认

## 1. 当前状态

| 字段 | 当前值 |
|---|---|
| 当前 active formal CR | `CR-126` |
| 标题 | `Quant-Lab Runner Development Readiness / Runner Entry Design` |
| 当前阶段 | CP2/CP5 人工门禁准备已完成，等待用户确认或修订 |
| 当前 checkpoint | `process/checkpoints/CP2-CR126-RUNNER-DEVELOPMENT-READINESS-REVIEW.md`、`process/checkpoints/CP5-CR126-RUNNER-ENTRY-DESIGN-AUTHORIZATION-REVIEW.md` |
| 最近校验 | human-gate CP2/CP5 OK；Decision Brief CP2/CP5 PASS；CR tracking OK；YAML OK；diff check OK |
| 是否已 approve | 否。用户尚未批准 CR126 CP2/CP5。 |
| 是否允许源码修改 | 否。当前只允许 process/design gate。 |
| 是否允许 runtime / NAS / QMT / 凭据 | 否。 |

## 2. 最新用户修正

用户明确指出：之前把 runner 后续拆成 4-6 个 CR 有过度拆分倾向。runner 本身是一个功能集中的模块，不应把 `run_spec`、runner entry、result、evidence、安全测试拆成多个正式 CR。

最新规划口径应改为：

```text
CR126: Runner Entry Design / Scope Gate
CR127: Runner Core MVP Implementation

CR128+ 仅在需要触碰外部权限域时作为可选后续 CR。
```

恢复后必须以这个压缩后的规划为准；如果 CR126 既有文件仍体现更细的后续拆分，应先解释为“CR127 内部实现 slice”，不要再把它们自动升级成多个正式 CR。

## 3. Runner Core 的当前边界

runner core 的最短路径目标是 offline-first，不依赖外部 runtime。

| 层级 | 当前建议 | 说明 |
|---|---|---|
| package intake | `trading/strategy_runner/package_loader.py` | 离线读取 manifest / checksum / payload；authorization flags 必须为 false。 |
| adapter dispatch | `trading/strategy_runner/adapters.py` | 将 multifactor / legacy / strategy_package payload 转为 target portfolio / order intent draft。 |
| readonly boundary | `trading/strategy_runner/readonly_gateway.py` | 默认 fake transport；真实 QMT client 必须另行 runtime authorization。 |
| evidence | `trading/strategy_runner/evidence.py` | 脱敏 evidence summary 和 forbidden counters。 |
| offline checker | `scripts/check_cr091_strategy_runner_package.py` | 当前最接近 offline runner entry 的脚本参考。 |

## 4. 修正后的 CR 规划

### 4.1 必须完成的 CR

| 顺序 | CR | 状态 | 目标 | 是否改源码 | 是否 runtime |
|---:|---|---|---|---:|---:|
| 1 | `CR126 Runner Entry Design / Scope Gate` | 当前 active | 冻结 runner MVP 范围、模块边界、禁止项、CR127 实现范围。 | 否 | 否 |
| 2 | `CR127 Runner Core MVP Implementation` | 未启动 | 一次性完成 offline runner core：run spec、runner orchestration、result/evidence、CLI/API、测试、CP7 验证、交付。 | 是 | 否 |

结论：完成 runner 本体，只需要 `CR126 + CR127`。

### 4.2 可选后续 CR

这些 CR 不应阻塞 runner core MVP。只有用户明确要求触碰对应外部权限域时才启动。

| 候选 | 触发条件 | 默认状态 |
|---|---|---|
| `CR128 Readonly Runtime Authorization Gate` | 需要真实 QMT/gateway health、capabilities、query_positions。 | 不启动 |
| `CR129 Readonly Runtime Integration` | CR128 已授权，且需要把 readonly runtime result 接入 runner evidence。 | 不启动 |
| `CR130 NAS Package Exchange Integration` | runner 需要直接消费/同步真实 NAS package exchange。 | 不启动 |
| `CR131 Provider / Lake / Catalog Integration` | runner 需要 provider fetch、lake write、catalog publish/current pointer。 | 不启动 |
| `CR132 Simulation / Live Execution Gate` | 用户明确要求 submit/cancel/buy/sell/simulation/live。 | 不启动，高风险 |

## 5. CR127 建议范围

CR127 应是一个集中实现 CR，不要继续拆小 CR。

| 模块 | 文件候选 | 内容 |
|---|---|---|
| Run Spec | `trading/strategy_runner/run_spec.py` | offline mode、run id、package path、output path、forbidden scope、not_authorization。 |
| Runner Orchestration | `trading/strategy_runner/runner.py` | package loader -> adapter -> target portfolio -> order intent -> evidence -> result。 |
| Run Result | `trading/strategy_runner/result.py` | status、blocked reasons、target count、order intent count、evidence summary、counters。 |
| Public API | `trading/strategy_runner/__init__.py` | 导出正式 runner API。 |
| CLI / Script Entry | 优先评估 `quant_lab/cli.py`，否则新增 `scripts/run_strategy_runner.py` | 暴露最小 offline runner 命令。 |
| Tests | `tests/test_cr127_runner_core_mvp.py` 等 | 正向、负向、安全边界、redaction、no-runtime。 |
| Process Evidence | `process/checks/CP6-*`、`CP7-*`、`CP8-*` | 实现、验证、交付证据。 |

### CR127 内部 slice

以下只是 CR127 内部执行顺序，不是多个正式 CR：

1. `S1 Contract`: `RunSpec` / mode / forbidden scope / result schema。
2. `S2 Orchestration`: package -> adapter -> target portfolio -> order intent。
3. `S3 Evidence`: evidence summary / counters / redaction。
4. `S4 CLI/API`: 暴露 offline runner entry。
5. `S5 Regression`: CR091/CR098 相关回归 + CR127 测试。
6. `S6 Delivery`: CP7/CP8 交付说明。

## 6. CR127 验收标准

CR127 完成时，必须能证明：

1. 本地 fixture package 能跑完整 offline runner。
2. runner 输出统一 `RunResult`。
3. 输出包含 target portfolio、order intent count、adapter status、evidence status、forbidden counters。
4. `not_authorization=true`。
5. `qmt_allowed=false`。
6. 不读取 `.env`。
7. 不访问 NAS。
8. 不启动 QMT / MiniQMT / XtQuant / gateway。
9. 不调用 provider。
10. 不写 lake。
11. 不 publish catalog。
12. 不执行 submit/cancel/buy/sell/simulation/live。
13. 敏感字段 redaction fail closed。
14. 负向输入 fail closed。
15. 最小测试集通过。

## 7. 明确不授权

清空上下文后仍然不授权以下动作：

- 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 `scripts/collect_cr099_runner_readonly_smoke.py` 或 `scripts/run_cr104_qmt_miniqmt_validation.ps1`。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不修改 `.gitignore`。
- 不提交、不 push、不 remote write。

## 8. 恢复后的建议动作

恢复后先判断用户意图：

### 如果用户说 `approve`

解释为：用户批准 CR126 当前 CP2/CP5 门禁，但需将最新修正纳入执行口径：

- runner 本体集中到 CR127。
- `RDS-01 / RDS-02 / RDS-03` 仅作为 CR127 内部 slice。
- 不预创建 CR128+。
- CR126 CP6 只生成压缩版 `Runner Core MVP Design / CR127 Scope`。

### 如果用户要求“继续”

先询问或明确采用推荐路径：

```text
我将按最新修正继续：CR126 只收敛 runner core MVP 设计和 CR127 范围；CR127 集中实现 runner core；不启动 runtime/NAS/trading CR。
```

### 如果用户要求“启动 CR127”

先检查：

1. CR126 CP2/CP5 是否已批准。
2. CR126 是否已完成 CP6/CP7/CP8 或是否允许跳转。
3. `meta-flow check cr-tracking --project-root .` 是否 OK。
4. 是否仍无 runtime/NAS/credential/provider/lake/catalog 授权。

未满足时不要直接创建 CR127。

## 9. 默认不要读取

- 全量历史对话 transcript。
- 全量 `process/stories/`。
- 历史 LLD、失败轮次、旧 QMT gateway route。
- Windows/Linux `.env` 或任何 secret。
- NAS 目录或包内容。
- 原始 QMT 账户、资金、持仓、委托、成交或日志。
- `data/`、`reports/` 内容，除非另有明确授权。

## 10. 恢复后的第一句话建议

建议新上下文读取本文件后回复：

```text
我已读取 process/context/CR126-CONTEXT-RESET-HANDOFF-2026-06-23.md。当前 active CR 是 CR126，CP2/CP5 门禁材料已准备但尚未批准。最新规划已收敛：runner 本体只用 CR126 + CR127 完成，CR128+ 仅作为 runtime/NAS/provider/lake/trading 等外部权限域可选后续。下一步建议先 approve CR126，并在 CP6 生成压缩版 Runner Core MVP Design / CR127 Scope。
```

## 11. 充分性结论

本 handoff 足以在清空上下文后恢复 CR126。恢复后不要直接启动 CR127，不要执行源码修改，不要启动 runtime。先确认 CR126 的人工门禁和最新压缩规划，再继续。
