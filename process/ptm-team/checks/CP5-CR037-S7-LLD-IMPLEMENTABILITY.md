---
doc_type: cp_check_result
id: CP5-CR037-S7-LLD-IMPLEMENTABILITY
story_id: "CR037-S7"
cr_id: "CR-037"
stage: CP5
checkpoint_type: auto
status: PASS
created_at: "2026-08-05"
owner: "meta-dev"
evidence_type: "full-lld"
evidence_path: "process/stories/STORY-CR037-S7-LLD.md"
---

# CP5-CR037-S7：设计证据可实现性自动预检

| 条目 | 结果 |
|------|:---:|
| lld_policy = full-lld 匹配 | PASS |
| 14 章节完整（0-15） | PASS |
| Goal 明确可量化 | PASS |
| Requirements 12 FR + 7 NFR 覆盖 | PASS |
| 模块拆分与职责清晰（SKILL.md + topo_plan_cli.py + templates 文件 + CLI 内部 12 函数划分） | PASS |
| 代码结构与文件影响范围明确（SKILL.md / topo_plan_cli.py / templates 新增；S1-S6 与 topo-config 只读消费；与 S8 边界） | PASS |
| 数据模型定义完整（CLI 参数归一化 dict + env-file/失败报告透传 S5 契约 + 模板副本参考 + 持久化 dry-run/execute 差异） | PASS |
| API/Interface 契约完整（CLI 命令格式 + 12 参数明细含 --execute/--authorized + _extract_topo_id_from_case + match 接线 + 退出码约定 + RuntimeAuthorizationError） | PASS |
| 核心处理流程有流程图 + 向导 5 步输出表 + 错误路径表 | PASS |
| 技术细节（统一 PYTHONPATH import 集成 S1 P2-4 + env-name 生成 + dry-run would-allocate 与预览副本 + SKILL.md 契约结构 + 模板内容约束） | PASS |
| 安全与性能分析（凭据禁止 ADR-02 + 默认 dry-run 门 + --authorized 显式门 + YAML 安全 + 路径安全；性能 <1s） | PASS |
| 测试设计（19 用例 + 3 fixture + 集成链路） | PASS |
| 实施步骤 12 TASK + 19.5h 预估 | PASS |
| 风险 6 项 + 缓解 | PASS |
| 实现灰区 7 项 + 取舍记录（topo-id vs case-file 优先级/dry-run 不写台账/NLP→CLI 归属/env-file 落盘时机/模板副本源/--authorized 授权门/dry-run 预览副本） | PASS |
| 回滚与发布策略 | PASS |
| DoD 18 项可验证 | PASS |
| 输出文件与 DEVELOPMENT-PLAN 一致（SKILL.md + scripts/topo_plan_cli.py + templates/{physical_pool,physical_ledger}.yaml.example） | PASS |
| 文件所有权无冲突（Wave 3 串行，S7 primary = SKILL.md + topo_plan_cli.py；与 S8 ptm-te.md/install.py 互斥） | PASS |
| 依赖 S7=[S1,S2,S3,S5,S6] precedence 满足 | PASS |
| tier="A" 合理 | PASS |
| open_items 无 | PASS |
| HLD §4.7 对齐（skill 向导 5 步 + CLI topo-plan + 默认 dry-run） | PASS |
| skill 向导 5 步（展示池 → 选/建逻辑 topo → 映射 → 导出 → 部署/验证）逐步有输出与下一步提示 | PASS |
| CLI 参数覆盖（--topo-id/--case-file/--limit/--limit-file/--devices-yaml/--pool-path/--ledger-path/--output/--report-out/--dry-run/--execute/--authorized） | PASS |
| NLP→CLI 转换由 agent 实现（G5/DQ-037-06），CLI 只收结构化参数；limit 语法规范引用 S2 docs/limit-syntax.md | PASS |
| 与 topo-config 引擎 import 集成（统一 PYTHONPATH，对齐 S1 P2-4，无 importlib 临时代码） | PASS |
| 默认 dry-run 门（不写台账/不落真实路径/不部署；--execute 必须配 --authorized 显式门，未授权抛 RuntimeAuthorizationError；真机部署由 S9 承接） | PASS |
| lld-check: LLD Structure Check | OK |

## 开放项

无。

## 修订说明（2026-08-05 产出）

- **向导 5 步**：对齐 HLD §4.7 与 topo-config SKILL 既有 5 步模式，每步输出人读摘要 + 「下一步提示」。
- **CLI 契约**：`topo-plan` 12 参数齐全（含 `--execute` / `--authorized`）；`--case-file` 双源提取 topo_id（E2，与 S8 编排语义一致）；`--limit`/`--limit-file` 互斥归一化。
- **dry-run 门**：默认 `--dry-run`（S4 `match` 传 `env_name=None` 不写台账，env-file 到 stdout 不落真实路径，打印 would-allocate 清单）；`--execute` 必须配 `--authorized` 显式门（S7-B，DQ-037-04 分层），未授权抛 `RuntimeAuthorizationError`（exit 3）。
- **预览副本（S8-B 联动）**：dry-run + `--output` 指向 run 目录 → 落预览 env-file（`metadata.executable:false`），下游 load_env_file 可解析，不写真实 `topology/` 路径。
- **PYTHONPATH**：import 集成走 S8 install 统一注入（S1 P2-4），CLI 顶部 sys.path 过渡降级，无 importlib 临时代码。
- **模板副本参考**：S7 templates 供人工参考与 S10 校验；S8 install 以 `skills/topo-config/config/`（P-1 模板化）为单一源，避免双套模板漂移（灰区 5）。

## 修订说明（2026-08-05，v1.1 CP5 NEEDS_REWORK 修订）

- S7-A：FR8 与向导 Step 5 流程一致性修正——CLI `--execute` 边界 = 写台账+落盘 env-file；真机部署由 S9 `deploy_bridge` 独立入口承接（S7 不直接调 deployer）。
- S7-B：新增 `--authorized` 显式门（FR12/NFR7）——`--execute` 无 `--authorized` 抛 `RuntimeAuthorizationError`（exit 3）；`--authorized` 无 `--execute` 报参数错误（exit 2）；dry-run 0 次真机写。
- S7-C：保持 dry-run 默认 / NLP→CLI agent 转换（G5/DQ-037-06）/ PYTHONPATH 集成（S1 P2-4）不变。
- S8-B 联动：dry-run 落预览 env-file 到 run 目录（`executable:false`），保证 dry-run 全链路下游可解析。
- lld-check 复跑 OK。
