---
cr_id: "CR-033"
artifact_type: "feature-design-matrix"
version: "1.0"
created_at: "2026-07-28T11:50:00+08:00"
author: "meta-se"
status: "draft"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_blueprint: "docs/design/BLUEPRINT-PTM-TE-EXEC.md"
---

# CR-033 ptm-te 执行引擎 Feature 设计矩阵

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CR-033 Feature 设计矩阵初稿：3 Feature / 8 Epic / 16 Story |
| 1.1 | 2026-07-28 | meta-se | CP3 评审反馈 P1/P4/P5 落实：P1 full-lld 统计 7->8 并改写说明；P4 核对 ST-EX-03 trigger_reasons 含 _build_exec_env 签名扩展；P5 ST-EX-02 拆分（规则块留 Wave 1，case-execution 安装验证并入 ST-EX-04），FE-EX-02/03 DESIGN.md 说明同步 |
| 1.2 | 2026-07-28 | meta-se | CP3 二轮评审 R4 落实：FE-EX-03 关联 Story 移除 ST-EX-14（ST-EX-14/EP-EX-08 归 FE-EX-02，消除双重归属） |
| 1.3 | 2026-07-28 | meta-se | CP3 评审范围扩展：环境文件驱动 resolve_env_refs。新增 ST-EX-17（环境解析层 + DUT 接口预配置，full-lld，FE-EX-02）；ST-EX-03/04/13 trigger_reasons 扩展（resolve_env_refs/load_env_file/${ENV.*} 改写）；lld_policy 统计 full-lld 8->9，总数 16->17 |

## Feature 设计适用性矩阵

| Feature ID | 名称 | 来源 | 是否需要 Feature 级设计 | 触发原因 | 产物路径 | 关联 Story |
|---|---|---|---|---|---|---|
| FE-EX-01 | TG 设备建模 | BLUEPRINT §1.1 / HLD §3 | waived | 单 Story 小改（ST-EX-01 SKILL.md + reference 增量）；ST-EX-03 op_mapper 改动由 Story LLD 承载 | N/A | ST-EX-01, ST-EX-03 |
| FE-EX-02 | case-execution 执行引擎 | BLUEPRINT §1.2 / HLD §3 | **required** | cross-module-contract（case_runner <-> op_mapper import）; data-model（frontmatter 16 列 + case_steps YAML）; concurrency（共享 session）; shared-story-boundary（12 Story 共享 case_runner.py） | docs/features/case-execution/DESIGN.md + TEST-PLAN.md + TASKS.md | ST-EX-04..12, ST-EX-14, ST-EX-15, ST-EX-16 |
| FE-EX-03 | 规则固化与执行改进 | BLUEPRINT §1.3 / HLD §3 | **required** | migration（24 用例目录迁移）; rollback（exec_v4.py 废弃）; cross-module（install.py 规则块 + 24 用例 md） | docs/features/rule-fix-and-improvement/DESIGN.md + TEST-PLAN.md + TASKS.md | ST-EX-02, ST-EX-13 |

## Feature 级设计产物清单

### FE-EX-02: case-execution 执行引擎

| 产物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| DESIGN.md | docs/features/case-execution/DESIGN.md | required（CP3 后生成） | case_runner.py 函数签名/类结构/异常处理/frontmatter 16 列解析/case_steps YAML 解析/extract_payload 字段提取表/四态分级判定矩阵/幂等容错规则/report.md 模板/case-execution 安装验证（PTM_TE_SKILLS） |
| TEST-PLAN.md | docs/features/case-execution/TEST-PLAN.md | required（CP3 后生成） | 25 场景 -> Story -> 测试用例覆盖矩阵 |
| TASKS.md | docs/features/case-execution/TASKS.md | required（CP3 后生成） | case_runner.py 实现任务清单 |

### FE-EX-03: 规则固化与执行改进

| 产物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| DESIGN.md | docs/features/rule-fix-and-improvement/DESIGN.md | required（CP3 后生成） | install.py 规则块文本/24 用例目录迁移映射表/重命名规则/frontmatter 补全规则/exec_v4.py 废弃标记 |
| TEST-PLAN.md | docs/features/rule-fix-and-improvement/TEST-PLAN.md | required（CP3 后生成） | install.py 重装一致性验证/24 用例整改校验 |
| TASKS.md | docs/features/rule-fix-and-improvement/TASKS.md | required（CP3 后生成） | 规则块 + 整改任务清单 |

## Story 与 lld_policy 矩阵

| Story ID | 名称 | Feature | lld_policy.required_level | trigger_reasons | rationale | feature_design_refs |
|---|---|---|---|---|---|---|
| ST-EX-01 | TG 设备建模 | FE-EX-01 | technical-note | 单 Story 小改，SKILL.md + reference 增量 | HLD 已承载边界，Story 内技术说明足够 | N/A（FE-EX-01 waived） |
| ST-EX-02 | 规则固化 | FE-EX-03 | technical-note | install.py 规则块增量，模式既有 | render_ptm_te_rule_body 既有，新增规则文本 | docs/features/rule-fix-and-improvement/DESIGN.md |
| ST-EX-03 | TG 路由参数化 | FE-EX-01 | full-lld | op_mapper 改动（_build_exec_env 签名扩展）；TREX_API_URL 注入；resolve_env_refs + env_topology 参数 | op_mapper 是核心映射层，改动需完整 LLD | N/A（FE-EX-01 waived，Story LLD 承载） |
| ST-EX-04 | case_runner 核心 | FE-EX-02 | full-lld | 新建 case_runner.py（三入口 + 取址 + 解析）；跨模块 import op_mapper；case-execution 安装验证（PTM_TE_SKILLS）；load_env_file 加载 --env-file | 核心引擎，需完整 LLD | docs/features/case-execution/DESIGN.md |
| ST-EX-05 | dry-run/授权门 | FE-EX-02 | full-lld | 安全边界（dry-run 默认门 + --execute 授权）；runtime_authorization 审计 | 安全相关，需完整 LLD | docs/features/case-execution/DESIGN.md |
| ST-EX-06 | 用例清理 | FE-EX-02 | full-lld | 逆序清理 mutation ops；step-refs 读取；回滚逻辑 | 涉及状态管理和回滚，需完整 LLD | docs/features/case-execution/DESIGN.md |
| ST-EX-07 | fw_logout 会话清理 | FE-EX-02 | full-lld | op_mapper 新增 fw_logout 映射；EXPECTED_OP_COUNT 改；降级逻辑 | op_mapper 改动 + 降级策略，需完整 LLD | docs/features/case-execution/DESIGN.md |
| ST-EX-08 | ARP 预热引擎 | FE-EX-02 | full-lld | warming_up/post_op 引擎强制；双重保障（ADR-06） | 引擎核心逻辑，需完整 LLD | docs/features/case-execution/DESIGN.md |
| ST-EX-09 | retry 轮询 | FE-EX-02 | technical-note | retry 字段解析 + 轮询逻辑 | 逻辑简单，Story 内技术说明足够 | docs/features/case-execution/DESIGN.md |
| ST-EX-10 | 四态分级 | FE-EX-02 | technical-note | known_issue 标记 + 四态判定 | 逻辑简单，Story 内技术说明足够 | docs/features/case-execution/DESIGN.md |
| ST-EX-11 | 诊断与报告 | FE-EX-02 | full-lld | 失败诊断字段提取 + report.md 模板 + 幂等容错记录 | 涉及错误分析和报告生成，需完整 LLD | docs/features/case-execution/DESIGN.md |
| ST-EX-12 | extract_payload | FE-EX-02 | technical-note | 统一解析函数，按 op_id 提取字段 | 逻辑简单，Story 内技术说明足够 | docs/features/case-execution/DESIGN.md |
| ST-EX-13 | 24 用例全量整改 | FE-EX-03 | full-lld | 24 用例目录迁移 + 重命名 + frontmatter 16 列 + tags + ARP 预热校验 + ${ENV.*} 引用改写 | 大批量迁移 + 多维度整改，需完整 LLD | docs/features/rule-fix-and-improvement/DESIGN.md |
| ST-EX-14 | verify_loss 消费侧 | FE-EX-02 | technical-note | verify_loss 提取 tx/rx/loss_ratio | 逻辑简单，Story 内技术说明足够 | docs/features/case-execution/DESIGN.md |
| ST-EX-15 | 用例结构化约定 | FE-EX-02 | technical-note | 目录结构 + 命名 + frontmatter 16 列约定 | 约定文档，Story 内技术说明足够 | docs/features/case-execution/DESIGN.md |
| ST-EX-16 | 标签/关键字执行 | FE-EX-02 | technical-note | --tag 精确过滤 + --keyword 模糊匹配 | 逻辑简单，Story 内技术说明足够 | docs/features/case-execution/DESIGN.md |
| ST-EX-17 | 环境解析层 + DUT 接口预配置 | FE-EX-02 | full-lld | env_topology 契约 + ${ENV.*} 解析集成 + DUT 接口自动预配置 + TREX_API_URL 环境文件优先 | 环境解析层是跨环境执行核心，涉及契约/解析/预配置/清理，需完整 LLD | docs/features/case-execution/DESIGN.md |

## lld_policy 统计

| lld_policy | Story 数 | Story ID |
|---|---|---|
| full-lld | 9 | ST-EX-03,04,05,06,07,08,11,13,17 |
| technical-note | 8 | ST-EX-01,02,09,10,12,14,15,16 |
| waived | 0 | - |

**说明**：17 个 Story 按 lld_policy 分布为 9 个 full-lld（ST-EX-03,04,05,06,07,08,11,13,17）+ 8 个 technical-note（ST-EX-01,02,09,10,12,14,15,16），与上表 Story 矩阵逐行一致。

## 关键决策项

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣/影响/风险/回退 |
|---|---|---|---|---|---|
| DQ-CP3-01 | architecture | ADR-08 TG 设备数据归属 | A：devices.yaml 元数据 + topology yaml 接口 | B：devices.yaml 含接口 | 推荐 A 与 firewall 块对称；备选 B 单文件但冗余。回退：topology yaml 不存在时切 B |
| DQ-CP3-02 | implementation | ADR-04 fw_logout op 暴露验证 | 安装前验证 + 降级 session 清理 | 强制 ptm-atomic 升级 | 推荐 DQ-02 降级方案；备选超出范围。回退：降级为 session 文件清理 |
| DQ-CP3-03 | scope | 24 用例 known_issue 标注完整性 | 纳入 R-F-021 同步检查 | 单独 CR | 推荐同步整改；备选增加协调成本。回退：只做 ARP 预热，known_issue 进 BACKLOG |

## waived / not-applicable 理由

| Feature | 判定 | 理由 | 影响 | 重访条件 |
|---|---|---|---|---|
| FE-EX-01 | waived | 单 Story 小改（ST-EX-01 SKILL.md + reference 增量）；ST-EX-03 op_mapper 改动由 Story full-lld 承载 | 无影响，HLD 已承载边界 | op_mapper TREX_API_URL 注入复杂度增加时重访 |
