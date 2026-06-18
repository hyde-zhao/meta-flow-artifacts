# CR091 Deploy Checklist

> release_decision: `READY_WITH_RISK`
> profile: `compact`
> generated_at: `2026-06-18T15:24:12+08:00`

## 部署结论

CR091 当前没有真实部署动作。CP8 只确认离线 runner 合同和本地代码 / fixture / checker 可交付。

| 项 | 结论 | 说明 |
|---|---|---|
| 安装脚本 | N/A | 未交付 install script、Agent、Skill 或平台扩展。 |
| 服务部署 | N/A | 不启动 QMT / MiniQMT / XtQuant / gateway / runner runtime。 |
| 配置变更 | N/A | 不读取 `.env`，不新增凭据或账户配置。 |
| 数据迁移 | N/A | 不迁移用户数据、账户数据、交易日志或 NAS 内容。 |
| 本地代码可验证 | PASS | CP6 / CP7 已通过离线验证。 |

## 发布前检查

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | process route health | PASS | `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` |
| 2 | CP6 implementation evidence | PASS | `process/checks/CP6-CR091-QMT-STRATEGY-RUNNER-CODING-DONE.md` |
| 3 | CP7 verification evidence | PASS_WITH_RISK | `process/checks/CP7-CR091-QMT-STRATEGY-RUNNER-VERIFICATION-DONE.md` |
| 4 | pytest contract tests | PASS | `13 passed` |
| 5 | offline checker | PASS | `passed=true` |
| 6 | static boundary scan | PASS | 未发现真实 runtime / sensitive read 入口 |
| 7 | release context | PASS | `process/release/RELEASE-CONTEXT-CR091.yaml` |
| 8 | CP8 context capsule | PASS | `process/context/CP8-CR091-DELIVERY-CONTEXT.yaml` |

## 不执行矩阵

| 项 | 状态 | 说明 |
|---|---|---|
| QMT / MiniQMT / XtQuant / gateway / runner runtime | NOT AUTHORIZED | 需要独立 runtime authorization gate。 |
| NAS package exchange | NOT AUTHORIZED | 需要独立 NAS package exchange gate。 |
| `.env` / 凭据 / 账户读取 | NOT AUTHORIZED | 当前 CP8 不授权。 |
| submit / cancel / simulation / live | NOT AUTHORIZED | 需要独立高风险 CR。 |
| provider / lake / publish | NOT AUTHORIZED | 当前 CP8 不授权。 |

## 升级 / 重复执行 / 回滚

| 场景 | 判断 |
|---|---|
| 全新安装 | N/A；无安装脚本。 |
| 升级 | N/A；仅本地代码和过程账本交付。 |
| 重复执行 | 可重复运行离线 pytest 和 checker；不得重复解释为 runtime 授权。 |
| 回滚 | 见 `process/docs/release/ROLLBACK-CR091.md`。 |
