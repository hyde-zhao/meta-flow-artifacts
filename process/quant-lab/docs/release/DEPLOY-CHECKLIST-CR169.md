---
title: "CR-169 Deploy Checklist"
status: "cp8-approved-git-delivery"
version: "1.2"
cr_id: "CR-169"
created_at: "2026-07-15T10:15:00+08:00"
owner: "host-orchestrator"
---

# CR-169 Deploy Checklist

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator | 明确 repository-local closure、提交后复跑和 no-deploy 边界。 |
| 1.1 | 2026-07-15 | host-orchestrator | 回填 CP8 批准、双仓 scoped commit、2159/0 和受控推送授权。 |
| 1.2 | 2026-07-15 | host-orchestrator | 回填两个授权 ref 的成对推送、远端一致性复核与 no-deploy 结论。 |

## CP8 前

- [x] 5/5 Story CP6/CP7 证据齐备。
- [x] Stage2 7/7 result 为 PASS，Stage3 ready=false。
- [x] provenance 与 design surface finding 已修复。
- [x] 两仓工作区与未授权项已记录。
- [x] 用户接受 CP8 风险和收尾顺序。

## CP8 批准后的推荐本地收尾

仅在用户明确授权本地提交后：

1. 分别审查 quant-lab 与 meta-flow-artifacts 的 staged scope，不包含无关工作流内容。
2. [x] 分别创建本地 CR-169 提交：quant `c22e9f9`、artifacts `aa05c76`。
3. [x] 在已提交状态运行 `uv run --python 3.11 pytest -q`：2159 passed / 0 failed。
4. [x] 0 failed 条件满足，可同步 CR-169 closed/READY_WITH_RISK。
5. [x] 将关闭态提交成对推送到用户明确授权的两个远端 ref，并复核 0 ahead / 0 behind：quant `c22e9f9`，artifacts 关闭态 `dc8d281`。

机器证据：`process/checks/CR169-PAIRED-GIT-DELIVERY.result.json`。该动作只是 Git 源码/过程产物交付，没有执行 tag、GitHub release、publish 或 deploy。

## 禁止操作

- 只允许本次推送到 `origin/work/cr169-capacity-liquidity-adv-evidence` 与 `origin/main`；不允许其他 ref、force-push、tag 或 GitHub release。
- 不 deploy/publish/runtime/trading。
- 不读取真实数据、凭据或 provider。
- 不启动 Stage3、不提升 CR155。
