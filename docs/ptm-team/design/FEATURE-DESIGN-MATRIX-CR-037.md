# Feature 设计矩阵 — CR-037（ptm-te topo 管理）

> 版本：v0.1 · 更新：2026-08-05 · CR：CR-037 · source_hld：`process/changes/CR-037-HLD.md`

## Feature 总览

| Feature ID | Feature 名称 | Agent | 适用性 | lld_policy | 状态 |
|---|---|---|---|---|---|
| **F-CR037** | ptm-te 集成 topo 管理（逻辑 topo → 物理 topo 映射 + env-file 产物 + 可选部署） | ptm-te | **waived**（HLD 已承载模块设计） | Story 级判定（见 §Story） | 🔄 planned（CR-037） |

## 适用性判定：F-CR037 = waived

**理由**：`process/changes/CR-037-HLD.md` §4 已覆盖全部 6 个模块设计（topology_collection / limit_parser / pool_merge / topo_mapper 增强 / exporter / install 集成）+ 产物契约 + 决策；Feature 级 DESIGN/TEST-PLAN 与 HLD 高度冗余，判定 waived 以避免重复设计。

**重访条件**：当出现以下任一情况时恢复 required 并生成 `docs/features/ptm-te-topo-management/DESIGN.md` / `TEST-PLAN.md` / `TASKS.md`：
1. HLD §4 模块设计在 CP5 评审中被判不充分（open_items 阻塞 Story 实现）
2. 新增模块超出 HLD §4 范围（如跨 Agent 共享拓扑服务）
3. 需要与 ptm-tde/topo-config 之外系统的新集成契约

## Story 级 lld_policy 判定

| Story | 模块 | lld_policy.required_level | 依据 |
|---|---|---|---|
| P-1 | 物理池模板化 | technical-note | 模板改造，模式既有 |
| S1 | topology_collection.py | full-lld | 新建解析模块 + 跨资源格式 |
| S2 | limit_parser.py | full-lld | NLP 边界 + 平台别名 + 语法规范交付物 |
| S3 | pool_merge.py | full-lld | 多数据源归并 + 冲突消歧 |
| S4 | topo_mapper per-link | full-lld | 核心引擎回溯剪枝改动 |
| S5 | exporter.py | full-lld | env-file 契约 + 命名 + 失败报告 |
| S6 | IP 规划 | technical-note | 复用 IPAM |
| S7 | topo-planning skill + CLI | full-lld | 新 skill + Prompt-Skill 边界 |
| S8 | ptm-te 集成 + install.py | full-lld | 跨 skill 编排 + 安装流程 |
| S9 | 可选部署桥接 | technical-note | 复用 EnvironmentDeployer |
| S10 | 测试 | technical-note | 验证覆盖 |
| S11 | 文档 | waived | 内容回填 |

> 无 runtime/security/credential/production-write 高风险标记的 batch 场景；S4 涉及引擎核心、S8 涉及安装流程，单独 full-lld，不使用 batch-lld。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v0.1 | 2026-08-05 | meta-se | 初稿：F-CR037 waived 判定 + Story lld_policy 表 |