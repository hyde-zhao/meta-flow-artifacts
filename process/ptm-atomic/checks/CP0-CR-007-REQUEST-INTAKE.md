# CP0 原始请求受理门

| 字段 | 值 |
|------|-----|
| 检查点 | CP0 |
| 状态 | PASS |
| 生成时间 | 2026-06-08T16:14:00+0800 |
| 请求编号 | CR-007 |
| 请求标题 | CLI 架构重构：run 命令嵌套子命令拆分 |

## 原始请求

用户要求将 `ptm-atomic run` 命令从扁平 80+ 参数注册重构为按操作族分组的嵌套子命令结构。这是针对操作快速增长（当前 40+，预期 100+）的预防性架构改进。

## 前置调研

### 问题确认

当前 `src/atomic_ops/commands/run.py`：
- `add_arguments()` 注册了 ~80 个扁平参数
- 所有 op（login、object、policy-route、interface、ospfv2、ospfv3-bfd）的参数混在一起
- 用户 `ptm-atomic run --help` 看到所有参数，无论跑哪个 op
- 参数通过 `op_id` 字符串分发到不同 runner 函数
- 新增操作族（如 BGP）需在 run.py 中追加 20-30 个参数

### 前期分析结论

1. ✅ argparse vs typer 已分析：采用方案 C（渐进式），先重构架构后评估迁 typer
2. ✅ CLI 命令名已从 `atomic-ops` 改为 `ptm-atomic`
3. ✅ 推荐方向：嵌套子命令按操作族分组

### Scope 界定

| 范围 | 说明 |
|------|------|
| **In Scope** | `ptm-atomic run` 拆分为嵌套子命令；消除 80+ 参数扁平注册 |
| **In Scope** | 每个操作族独立命令模块，只暴露自己的参数 |
| **In Scope** | 统一错误输出和 stale warning 的重复实现 |
| **In Scope** | 测试覆盖和文档更新 |
| **Out of Scope** | 框架迁移（typer）—— 列入后续评估 |
| **Out of Scope** | ops 的语义或行为变更 |
| **Out of Scope** | sync/list/show/adapter/packages/validate 命令的变更 |

## Entry Criteria

- [x] 用户已明确请求
- [x] 问题可复现（run --help 显示所有参数）
- [x] 无安全/权限/外部接口边界问题
- [x] 无冲突的活跃 CR

## Exit Criteria

- [x] 需求范围已界定
- [x] 进入 CP1 场景分析

## 结论

PASS — 进入 CP1 用户场景分析。
