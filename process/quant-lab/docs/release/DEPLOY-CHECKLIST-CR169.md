---
title: "CR-169 Deploy Checklist"
status: "cp8-draft-no-deploy"
version: "1.0"
cr_id: "CR-169"
created_at: "2026-07-15T10:15:00+08:00"
owner: "host-orchestrator"
---

# CR-169 Deploy Checklist

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator | 明确 repository-local closure、提交后复跑和 no-deploy 边界。 |

## CP8 前

- [x] 5/5 Story CP6/CP7 证据齐备。
- [x] Stage2 7/7 result 为 PASS，Stage3 ready=false。
- [x] provenance 与 design surface finding 已修复。
- [x] 两仓工作区与未授权项已记录。
- [ ] 用户接受 CP8 风险和收尾顺序。

## CP8 批准后的推荐本地收尾

仅在用户明确授权本地提交后：

1. 分别审查 quant-lab 与 meta-flow-artifacts 的 staged scope，不包含无关工作流内容。
2. 分别创建本地 CR-169 提交；不得 push。
3. 在已提交状态运行 `uv run --python 3.11 pytest -q`。
4. 只有 0 failed 才可同步 CR-169 closed/READY_WITH_RISK；否则回 CP7。

## 禁止操作

- 不 push/force-push/tag/GitHub release。
- 不 deploy/publish/runtime/trading。
- 不读取真实数据、凭据或 provider。
- 不启动 Stage3、不提升 CR155。
