# CP4 Story 拆解与并行安全门（自动预检，汇入 CP5）

| 字段 | 值 |
|------|-----|
| 检查点 | CP4 |
| 关联 CR | CR-007 |
| 生成时间 | 2026-06-08T16:35:00+0800 |
| 状态 | PASS（自动预检） |

## Story 清单

| Story ID | 标题 | 涉及文件 | 估时 |
|------|------|------|:---:|
| STORY-007 | 重构 cli.py + run.py 支持嵌套子命令 | `cli.py`, `commands/run.py` | M |
| STORY-008 | 提取公共错误输出到 errors.py | `errors.py`, 7 个 commands/*.py | S |
| STORY-009 | 新增 run_auth.py + run_object.py | `commands/run_auth.py`, `commands/run_object.py` | S |
| STORY-010 | 新增 run_policy_route.py | `commands/run_policy_route.py` | M |
| STORY-011 | 新增 run_interface.py | `commands/run_interface.py` | M |
| STORY-012 | 新增 run_ospf.py | `commands/run_ospf.py` | M |
| STORY-013 | 新增 run_ospfv3.py | `commands/run_ospfv3.py` | M |
| STORY-014 | 更新测试覆盖 | 测试文件 | M |
| STORY-015 | 更新文档 | `CLAUDE.md`, `README.md`, `docs/*.md` | S |

## Story DAG

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Wave 1 (可并行)                                             │
│  ┌──────────────────────┐  ┌──────────────────────┐        │
│  │ STORY-007            │  │ STORY-008            │        │
│  │ cli.py + run.py 重构  │  │ errors.py 增强       │        │
│  └────────┬─────────────┘  └────────┬─────────────┘        │
│           │                         │                       │
│           └──────────┬──────────────┘                       │
│                      │                                      │
│  Wave 2 (全部并行)     │                                      │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐             │
│  │S-009 │ │S-010 │ │S-011 │ │S-012 │ │S-013 │             │
│  │auth  │ │p-rt  │ │intf  │ │ospf  │ │ospfv3│             │
│  └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘             │
│     └────────┴────────┴────────┴────────┘                   │
│                      │                                      │
│  Wave 3               │                                      │
│               ┌──────┴──────┐                               │
│               │ STORY-014   │                               │
│               │ 测试覆盖     │                               │
│               └──────┬──────┘                               │
│                      │                                      │
│  Wave 4               │                                      │
│               ┌──────┴──────┐                               │
│               │ STORY-015   │                               │
│               │ 文档更新     │                               │
│               └─────────────┘                               │
└─────────────────────────────────────────────────────────────┘
```

## 依赖关系

| Story | 依赖 | 类型 | 理由 |
|------|------|------|------|
| STORY-008 | — | 无依赖 | 独立修改 errors.py |
| STORY-009 | STORY-007 | `runtime` | 需要 cli.py 的新注册机制 |
| STORY-010 | STORY-007 | `runtime` | 同上 |
| STORY-011 | STORY-007 | `runtime` | 同上 |
| STORY-012 | STORY-007 | `runtime` | 同上 |
| STORY-013 | STORY-007 | `runtime` | 同上 |
| STORY-014 | STORY-007..013 | `runtime` | 需要完整实现 |
| STORY-015 | STORY-014 | `runtime` | 测试通过后才更新文档 |

## 文件所有权矩阵

| 文件 | 负责 Story | 冲突检查 |
|------|------|:---:|
| `cli.py` | STORY-007 | ✅ 单 owner |
| `commands/run.py` | STORY-007 | ✅ 单 owner |
| `errors.py` | STORY-008 | ✅ 单 owner |
| `commands/sync.py` | STORY-008（引用 errors.py 新 API） | ⚠️ 与 STORY-007 并行安全 |
| `commands/show.py` | STORY-008（引用 errors.py 新 API） | ⚠️ 与 STORY-007 并行安全 |
| `commands/list_ops.py` | STORY-008（引用 errors.py 新 API） | ⚠️ 与 STORY-007 并行安全 |
| `commands/validate.py` | STORY-008（引用 errors.py 新 API） | ⚠️ 与 STORY-007 并行安全 |
| `commands/packages.py` | STORY-008（引用 errors.py 新 API） | ⚠️ 与 STORY-007 并行安全 |
| `commands/adapter.py` | STORY-008（引用 errors.py 新 API） | ⚠️ 与 STORY-007 并行安全 |
| `commands/run_auth.py` | STORY-009 | ✅ 新文件 |
| `commands/run_object.py` | STORY-009 | ✅ 新文件 |
| `commands/run_policy_route.py` | STORY-010 | ✅ 新文件 |
| `commands/run_interface.py` | STORY-011 | ✅ 新文件 |
| `commands/run_ospf.py` | STORY-012 | ✅ 新文件 |
| `commands/run_ospfv3.py` | STORY-013 | ✅ 新文件 |

## 并行安全分析

| 检查项 | 结果 |
|------|:---:|
| DAG 无循环依赖 | ✅ |
| Wave 2（5 个 Story）全部创建不同新文件，无冲突 | ✅ |
| Wave 1（STORY-007 + STORY-008）修改不同文件（cli.py + run.py vs errors.py） | ✅ |
| STORY-008 修改现有 commands/*.py 的 import，与 STORY-007 无文件冲突 | ✅ |
| `runtime` 依赖均为逻辑依赖（需要新注册机制），非文件冲突 | ✅ |
| 无 `file-conflict` 依赖 | ✅ |

## 退出条件

- [x] Story 清单完整（9 个）
- [x] DAG 无循环依赖
- [x] 文件所有权无冲突
- [x] Wave 划分合理（4 个 Wave）
- [x] 可汇入 CP5 Decision Brief

## 结论

PASS — 进入 CP5 Story LLD。
