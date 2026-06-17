---
project_id: "meta-flow"
release_scope: "meta-flow-artifact-routing-and-eval-governance"
created_at: "2026-06-17T13:49:25+08:00"
---

# Feedback

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-11 | host-orchestrator | 新增 workflow eval governance 反馈入口 |
| 1.1 | 2026-06-17 | host-orchestrator | 增加 artifact routing、symlink health 和 advanced eval runner 观察项 |

## 反馈分流

| 信号 | 分流 |
|---|---|
| `process/` 或 `docs/` symlink 缺失、断链或指向错误项目 | hard-stop，用户提供 artifact root 后创建恢复 CR |
| eval schema 无法覆盖新 workflow 类型 | 创建新 CR，更新 `evals/contracts/*` |
| runner 无法解析复杂 YAML | 创建 runner CR，考虑引入正式 parser |
| expected failure case 影响 suite health 判定 | 创建 eval runner regression CR |
| external adapter 需要真实运行 | 创建 runtime_authorization CR 或 Spike |
| suite health 出现 blocking failure | 写入 `docs/quality/FAILURE-BACKLOG.md` 并路由回实现或评估修复 |

## 当前后续项

无 open follow-up candidate。

## 观察阈值

| 观察项 | 阈值 | 动作 |
|---|---|---|
| workspace health failure | 任意一次 `process_link_health != ok` | 停止工作流并要求用户确认 artifact root |
| eval runner unsupported grader/config | 任意新增正式 fixture 失败 | 创建 CR 或扩展 schema 后再合并 |
| external adapter request | 需要凭据、网络或 SaaS 执行 | 不执行，先走 runtime_authorization |
