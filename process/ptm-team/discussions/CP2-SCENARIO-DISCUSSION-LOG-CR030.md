---
cr_id: CR-030
checkpoint: CP2
status: completed
discussion_mode: inline-fallback-approved-by-user
---

# CP2 场景讨论记录 — CR-030

## SGQ-CR030-01：MVP 是否限制为恢复后现网问题，并以人工批准的文件化改进输入流向下游？

| 项目 | 内容 |
|---|---|
| 背景 | 参考材料同时描述现网闭环、自动化演进和内部问题的后续方向；不澄清会改变 scope、安全边界和 HLD。 |
| 推荐方案 | 仅支持恢复后现网问题；P1 必做、P2 可选、P3/P4 不自动处理；只输出文件化的已批准改进输入。 |
| 备选 A | 覆盖所有严重度与内部问题。优点是覆盖广；代价是分析终点、度量和安全模型混杂。 |
| 备选 B | 只生成报告，不定义下游输入。优点是实现最小；代价是无法证明 CA/PA 闭环。 |
| 用户回答 | 用户在 CR-030 已建立、范围/禁止能力明确后回复“批准，继续推进项目到下一个人工门禁”。 |
| 复述确认 | 作为对 CR-030 推荐范围与 inline fallback 的批准：采用推荐方案，不授权 runtime、凭据或自动修复。 |
| 影响面 | USE-CASES、REQUIREMENTS、SCENARIOS、MVP、跨 Agent HLD 契约和未来 Story。 |
| 状态 | resolved-by-user-approval |

## 讨论结论

## SGQ-CR030-REV01-01：用户场景应当以谁的真实目标和旅程为中心？

| 项目 | 内容 |
|---|---|
| 背景 | 原 v1.0 把“创建 RA / 生成 CA/PA / 跟踪关闭”等处理步骤写成场景标题；虽列出 persona，却没有让用户的目标、决策和结果成为主线。 |
| 用户自由反馈 | “你的用户场景不是基于用户来写的，我需要的是基于用户来写的用户场景”。 |
| 复述理解 | v1.1 以五类真实用户为主体：测试经理决定是否投入预防，测试架构师建立事实并解释逃逸，测试设计负责人消费已批准教训，质量负责人依据有效性关闭，安全/合规负责人守住权限边界。每个 UC 均包含目标、触发、输入、旅程、结果、协作和失败体验。 |
| 影响面 | `USE-CASES.md`、`REQUIREMENTS.md`、`SCENARIOS.yaml`、`TEST-MATRIX.md`、`STORY-MAP.md`、`MVP-SCOPE.md`、CP1/CP2；HLD/CP3 在 CP2 后刷新。 |
| 当前状态 | resolved-by-user-approval |

## 当前结论

- 原 SGQ-CR030-01 的范围与权限结论继续有效，但旧场景表达及其 CP2 批准已失效。
- 用户已回复“批准CP2”，确认 v1.2 的七个用户旅程及三项 ITR 决策；BLOCKING 场景问题：0。
- 已完成一次受控 GET schema 探测：HTTP 200、81 条记录；不展示问题单内容，字段映射在 CP3 HLD 中收敛。
- 未确认的实现细节不在 CP2 处理；转入 CP3 Architecture Gray Areas。
- Deferred Ideas 已保留在 `docs/product/BACKLOG.md`。

## SGQ-CR030-REV02-01：如何让用户从真实问题单获得分析价值而不扩大权限？

| 项目 | 内容 |
|---|---|
| 背景 | 用户要求 ptm-tse 使用指定 ITR 请求获取并保存所有现网问题单，再完成分析、输出总结和改进措施。这改变了原有“无外部读取”的用户旅程和安全边界。 |
| 用户自由反馈 | “需要使用这个请求，他的响应信息就是所有的现网问题单。ptm-tse需要具备获取问题单信息，并保存。然后根据这些问题单完成分析，如果需要新增分析字段也可以新增。ptm-tse需要具备问题分析后输出总结的能力，需要具备输出改进措施的能力。” |
| 复述理解 | 新旅程由测试架构师发起：受控 ITR GET → 可追溯保存 → 逐单/批量分析 → 总结与 CA/PA 候选；安全/合规负责人监督读取和保存，reviewer 仍是确认、分发和关闭的唯一授权者。 |
| 推荐边界 | 只允许该固定 URL 的 GET；不读凭据、不推断认证、不写 ITR 或其他系统；在数据分类/保留未确认时不持久化原始响应；不自动执行改进。 |
| 影响面 | `USE-CASES.md`、`REQUIREMENTS.md`、`SCENARIOS.yaml`、`TEST-MATRIX.md`、Story Map、MVP、HLD、存储契约、fixture 和安全验证。 |
| 当前状态 | resolved-by-user-approval |
