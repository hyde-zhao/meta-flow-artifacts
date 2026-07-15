---
status: confirmed
version: "6.0"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-04-23T20:18:07+08:00"
created_by: "meta-se"
created_at: "2026-04-23T20:18:07+08:00"
hld_version: "6.0"
---

# 架构决策记录 v6：ptm-tde

## ADR-1：保持单 Agent 编排，新增 `case-retriever`

**决策**：继续采用单 Agent 架构，运行态能力仍由 `ptm-tde` 编排；新增 `case-retriever` 作为独立检索 Skill。

**原因**：
- 用户把“生成用例”和“检索用例”视为两类独立动作。
- 场景链、动作源、设计过程和检索索引共享同一追踪链，不适合拆为多 Agent。

**后果**：
- Agent 入口不变；
- Skill 数量从 16 扩展到 17；
- 检索能力从交付副产物升级为正式运行能力。

## ADR-2：动作源采用显式建模

**决策**：将 `REST API / CLI Tool / Tool Method` 抽象为统一的 Action Source 模型，而不是只在场景文本中隐式描述。

**原因**：
- 原子操作需要直接指导后续逻辑用例和物理用例；
- 用户明确要求支持外部 API 配置和 CLI 化工具接口；
- 现有工具能力不足时，需要输出工具抽象草案而不是停留在描述层。

**后果**：
- `scenario-discovery` 需要消费动作源输入；
- 交付物中需要保留动作源缺口和工具抽象说明；
- M/F/Q 与 integrator 需要保留 Action Source trace。

## ADR-3：知识治理收敛为只读 MCP staged query

**决策**：知识库能力当前只做只读查询，采用“领域场景 → 特性场景 → 特性主功能”的 staged query，不做知识入库、远端索引维护或写回。

**原因**：
- 用户已明确当前阶段不做知识入库；
- 先把知识引用链做稳，避免过度设计；
- 只读查询更利于与现有知识库联合开发。

**后果**：
- `mcp_query_client.py` 需要返回成功 / 缺失 / 不可用三态；
- HLD 中的知识索引治理链路被删除；
- 场景链与交付物需保留来源引用与查询时间。

## ADR-4：安装由 `install.py` 统一托管，project 先落地

**决策**：Claude Code / Codex 的安装统一由 `scripts/install.py` 托管；首阶段优先落地 `project scope`，`user scope` 在具备规则文件合并能力后补齐。

**原因**：
- 当前仓库已存在 installer 基础能力；
- 用户明确需要 `project scope / user scope` 双范围；
- `user scope` 存在 `CLAUDE.md` / `AGENTS.md` 覆盖风险，需先设计 marker merge。

**后果**：
- Story 规划必须拆出 project / user 两个安装 Story；
- `user scope` 需要 merge report；
- Codex agent 投影文件统一生成为 `.codex/agents/*.toml`；
- Codex Skill 投影文件按官方仓库/用户发现规则分别生成为 `<project>/.agents/skills/<name>/SKILL.md` 与 `~/.agents/skills/<name>/SKILL.md`；
- 未实现合并前不得静默覆盖现有全局规则文件。

## ADR-5：检索保持轻量结构化索引

**决策**：检索首版仅支持 `需求编号 / 逻辑用例编号 / 功能分类标签` 三类入口，不扩展关键字全文检索和复杂排序。

**原因**：
- 用户明确不需要复杂检索；
- 轻量结构更适合先把 trace 和标签产出做完整；
- 能避免把实现重心从设计质量偏到检索算法。

**后果**：
- `case-index.yaml` 字段收敛；
- `deliverable-renderer` 必须生成 feature tags；
- `case-retriever` 只做精确过滤和命中展示。

## ADR-6：设计 Skill 按“图形链”与“规则/数据链”成组推进

**决策**：Story 层面将 `process-design + state-design` 视为图形链设计包，将 `parameter-design + data-design + combination-design` 视为规则/数据链设计包。

**原因**：
- 两组技能内部共享建模方式和输出骨架；
- 便于在 Story 级 LLD 中复用共享片段，降低设计重复；
- 仍保留 Skill 级实现边界，不把多个方法混成单一实现对象。

**后果**：
- Story 数量控制在可评审范围内；
- LLD 中必须显式拆分子模块职责；
- `shared_fragments` 需要在多个 LLD 中复用。
