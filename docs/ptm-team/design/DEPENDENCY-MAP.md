# PTM Team 依赖地图

> 版本：v1.0 · 更新：2026-06-08 · 覆盖：全部 6 个 Agent 及外部依赖

---

## Agent 间依赖关系

```
                    ┌─────────┐
                    │ ptm-tm  │ 测试经理（调度核心，Step 3）
                    └────┬────┘
                         │ 调度/任务下发
            ┌────────────┼────────────┐
            ▼            ▼            ▼
      ┌─────────┐  ┌─────────┐  ┌─────────┐
      │ ptm-tse │  │ ptm-tde │  │ ptm-qa  │
      │ 架构师  │  │ 设计    │  │ 质量    │
      │ Step 3  │  │ ✅ v1.0 │  │ Step 3  │
      └────┬────┘  └────┬────┘  └─────────┘
           │ 评审       │ 用例
           ▼            ▼
      ┌──────────────────────┐
      │       ptm-te         │ 测试执行（Step 2）
      └──────────┬───────────┘
                 │ 执行记录 + 工具需求
                 ▼
      ┌──────────────────────┐
      │       ptm-tae        │ 自动化工程师（Step 1-2）
      │  🔄 Step 1 进行中    │
      └──────────────────────┘
                 │
                 │ 公共 Skill + 工具
                 ▼
            全部 Agent
```

---

## 依赖类型

| 上游 | 下游 | 依赖类型 | 状态 | 说明 |
|---|---|---|---|---|
| ptm-tm | ptm-tse | runtime | ⬜ planned | Step 3 调度 |
| ptm-tm | ptm-tde | runtime | ⬜ planned | Step 3 调度 |
| ptm-tm | ptm-te | runtime | ⬜ planned | Step 3 调度 |
| ptm-tm | ptm-qa | runtime | ⬜ planned | Step 3 调度 |
| ptm-tse | ptm-tde | contract | ⬜ planned | 评审用例覆盖度 |
| ptm-tse | ptm-te | contract | ⬜ planned | 评审执行策略 |
| ptm-tse | ptm-tae | contract | ⬜ planned | 评审工具框架 |
| ptm-tde | ptm-te | contract | ✅ verified | 用例输出格式（16 列 PC） |
| ptm-te | ptm-tae | runtime | ⬜ planned | 执行记录 + 工具需求 |
| ptm-tae | 全部 | contract | 🔄 in-progress | 公共 Skill + 原子能力 |

---

## 外部依赖

| 依赖 | 类型 | 提供方 | 状态 | 说明 |
|---|---|---|---|---|
| Claude Code | runtime | Anthropic | ✅ | Agent/Skill 运行平台 |
| Codex | runtime | OpenAI | ✅ | Agent/Skill 运行平台 |
| Qoder | runtime | Qoder | ✅ | Agent/Skill 运行平台 |
| atomic-ops CLI | runtime | ptm-tae | ✅ | 79 个原子操作 |
| 因子库 YAML | data | ptm-tde/ptm-tae | 🔄 | 策略路由因子等 |
| 禅道 API | runtime | 外部平台 | ⬜ | 任务管理集成 |
| 钉钉 API | runtime | 外部平台 | ⬜ | 通知推送 |
| BPS/IXIA 仪表 | runtime | 硬件 | 🔄 | 流量生成 |
| 防火墙硬件 | runtime | 硬件 | 🔄 | 测试目标设备 |

---

## 文件所有权（ptm-tde 部分）

| 文件/目录 | primary | shared | forbidden |
|---|---|---|---|
| `agents/ptm-tde.md` | ptm-tde | — | 其他 Agent |
| `skills/m-analyzer/` | ptm-tde | ptm-tae（atomic-ops 集成） | ptm-te |
| `skills/f-analyzer/` | ptm-tde | — | — |
| `skills/q-analyzer/` | ptm-tde | — | — |
| `skills/ppdcs/` | ptm-tde | — | — |
| `docs/ptm-tde/` | ptm-tde | meta-po（文档维护） | — |
| `resource/` | ptm-tae（维护） | ptm-tde（消费） | — |

---

## 关键路径

```
Step 1 完成（当前）：
  ptm-tae 原子能力就绪 → ptm-te 执行可用 → tde→te→tae 闭环

Step 2 完成：
  ptm-te 执行就绪 → 自动化翻译 → 回归运行 → 最小闭环验证

Step 3 完成：
  ptm-te 闭环成熟 → ptm-tm 调度 + ptm-tse 评审 + ptm-qa 审计 → 六 Agent 协同

Step 4 完成：
  六 Agent 协同成熟 → 全自动 → 人工监督
```

---

*本地图跟踪 ptm-team 的全部依赖关系。随项目进展动态更新。*
