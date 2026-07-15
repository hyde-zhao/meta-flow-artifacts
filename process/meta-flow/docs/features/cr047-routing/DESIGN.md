---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-02"
feature_name: "Artifact Routing"
source_blueprint: "process/docs/design/CR047-WORKFLOW-TRUTH-BLUEPRINT.md"
source_hld: "process/docs/design/CR047-WORKFLOW-TRUTH-HLD.md"
source_adr: "process/docs/design/CR047-WORKFLOW-TRUTH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR047-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-WT-002"]
lld_policy_summary: "full-lld=1"
---

# Feature Design: Artifact Routing

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline / meta-se | 冻结 portable link、relative route metadata 与内部 docs 单一 canonical 契约。 |

## 摘要与边界

| 项目 | 内容 |
|---|---|
| 目标 | clean clone 从源码根执行一次 link 后获得健康 `process` 路由和唯一内部文档写入口。 |
| 推荐方案 | 扩展现有 workspace routing/check，所有 refs 使用 project-relative `process/docs/**`。 |
| 下游 Story | ST-WT-002 |
| 非目标 | 处理 prelink backup、建立 ignored 根内部 docs symlink、读取 quant-lab 状态。 |

## 上游依据与现有代码

| 来源/区域 | 路径 | 消费内容 / 当前职责 |
|---|---|---|
| ADR | `ADR-WT-002` | artifact process/docs 唯一 canonical；根 docs 只保留 public tracked docs |
| Workspace | `meta_flow/workspace/routing.py` | link/check、metadata 与 route health |
| CLI | `meta_flow/cli.py` | workspace 命令入口 |
| Contracts | `delivery/rules/DIRECTORY-CONTRACT.md`, `.yaml` | current/process 目录职责 |
| Tests | `tests/test_workspace_routing.py` | link、metadata、health fixtures |

## 推荐方案与接口

| 设计点 | 做法 | 失败行为 |
|---|---|---|
| link root | 仅从 project root 解析 `link_path=process` | 在 artifact repo 根执行导致 regular path 冲突时给出明确诊断 |
| route metadata | `artifact_root` 相对 project_root，`project_process_root` 相对 artifact_root | 设备绝对路径或 anchor 冲突为 BLOCKER |
| docs canonical | 检查内部 writable copy count=1，目标为 `process/docs/**` | 根内部 docs 可写副本出现则失败 |
| clean clone fixture | Git tracked source + artifact fixture + link/check | 依赖 ignored 本机文件即失败 |

route check 输出 `project_root,link_path,routing_mode,expected_project_name,actual_target,artifact_root,project_process_root,metadata_path,state_path`；dirty 只作为信息，不改变 link health。

## 流程、失败与回退

1. 从 source project root 解析 artifact root 和 project name。
2. 若 `process` 缺失，创建 symlink 与 project process root；若 regular path 存在则停止并要求迁移。
3. 写相对 metadata，检查 project_name、target 与 State routing ref。
4. 扫描内部 docs canonical copy，禁止静默复制。

失败时不重建 State、不移动现有 regular path、不碰 backup；回退只删除本次新建且未承载数据的 link，保留 artifact 内容。

## 测试、任务与下游契约

- clean source/artifact snapshot 的 link/check 集成 fixture。
- regular path、断链、project_name mismatch、absolute metadata、duplicate docs negative fixtures。
- TASK 与文件范围见 `TASKS.md`，测试见 `TEST-PLAN.md`。
- lld-designer 必须明确绝对路径只用于运行时诊断，任何持久化 ref 必须相对。

## 风险与 Gotchas

- `artifact_git_dirty` 不等于 route 不健康。
- 命令必须在源码仓根执行；artifact 仓已有 `process/` 是正常数据目录。
- 不得通过兼容 symlink 恢复“本机有、clean clone 无”的隐式真相。
