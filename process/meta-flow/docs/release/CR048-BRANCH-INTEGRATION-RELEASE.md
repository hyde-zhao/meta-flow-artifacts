# CR-048 Branch Integration Release

## 发布说明

- `meta-flow/main` 通过 merge commit `ad3f6f42b179ec4767d29a07b7e9604e1969b8fe` 同时包含原 main 和 `fix/claude-md-rule-numbering` 的历史。
- `delivery/rules/AGENTS.md` 成为唯一 tracked rules source；Claude 安装仍生成目标 `CLAUDE.md`。
- dispatch result 检查按 attempt 聚合多事件，event identity 不再回退为 dispatch/run identity。
- 新增 handoff dispatch 检查，默认兼容 legacy，显式文件和 `--strict-all` 严格。

## 部署检查

- source `origin/main` 与 merge commit 一致。
- artifact 仓只允许提交 `process/meta-flow/**`，必须排除现有 `process/quant-lab/**` 改动。
- 不删除 fix/integration 远端分支，不创建 tag/release，不执行真实安装。

## 回滚

- 推送前回退点：`3e303215692840ec8ac1920a1d3c21c14ea2907b`。
- 推送后只能通过新的显式 revert CR 回退 merge commit；禁止 force-push 或历史改写。
- artifact 过程证据采用普通 revert/correction，不改写 ledger。

## 迁移

- 源码消费者无需数据迁移。
- 维护者停止编辑 tracked `delivery/rules/CLAUDE.md`；Claude 规则由 installer/package builder 从 AGENTS 源生成。

## 反馈与剩余风险

- 独立 QA/platform receipt 未提供，结论最高 `READY_WITH_RISK`。
- 平台验证为 dry-run-only；真实安装需要新授权。
- 历史 ledger/handoff 缺字段保持 warning，不进行倒填。
