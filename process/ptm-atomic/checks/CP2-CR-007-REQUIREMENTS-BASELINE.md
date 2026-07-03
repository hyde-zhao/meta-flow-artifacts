# CP2 需求基线门（自动预检）

| 字段 | 值 |
|------|-----|
| 检查点 | CP2 |
| 状态 | PASS（自动预检） |
| 关联 CR | CR-007 |
| 生成时间 | 2026-06-08T16:20:00+0800 |

## 需求结构化

### REQ-001：嵌套子命令结构（P0）

将 `ptm-atomic run <op_id> [--80+参数]` 重构为：

```
ptm-atomic run <family> <action> <target> [--family-specific-params]
```

**操作族划分**（基于现有 `run.py` 中的 op 分组）：

| 族名 | 现有 op 数 | 现有参数数 |
|------|:---:|:---:|
| `auth` | 1 (login) | 共享参数 + username/password-env/change-default-password/new-password-env |
| `object` | 1 (config) | 共享参数 + object-name/ipaddr/mask/object-desc |
| `policy-route` | 5 | 共享参数 + source-network/dst-network/next-hop-ip/in-interface/... +16 个 |
| `interface` | 5 | 共享参数 + id/interface-kind/mode/ip-address/... +20 个 |
| `ospf` | 14 (ospfv2) | 共享参数 + router-id/area/network/interface/cost/... +25 个 |
| `ospfv3` | 17 (ospfv3 + bfd) | 共享参数 + router-id/area/interface/... +10 个 |

### REQ-002：共享参数提取（P0）

所有 run 子命令共享的参数：

```
--base-url       (required)  设备管理 URL
--session-file   (optional)  会话文件路径
--timeout        (default: 10.0) HTTP 超时
--verify-tls     (flag)      验证 TLS 证书
--format         (json|yaml) 输出格式
--execute        (flag)      真实执行 vs dry-run
--auth-header    (Authorization|AuthorizationToken) 认证头
```

这些参数应在 `run` 层级定义，自动继承到所有子命令。

### REQ-003：独立命令模块（P0）

每个操作族一个模块文件：
- `commands/run_auth.py`
- `commands/run_object.py`
- `commands/run_policy_route.py`
- `commands/run_interface.py`
- `commands/run_ospf.py`
- `commands/run_ospfv3.py`

每个模块仅声明自己需要的参数。

### REQ-004：统一错误/警告输出（P1）

当前 `_write_error`、`_write_warning`、`_write_stale_warning`、`_suggestion`、`_details_text` 在 7 个命令模块中重复实现。应提取到 `errors.py` 或新建公共模块。

### REQ-005：向后兼容（P1）

旧格式 `ptm-atomic run <op_id> [--params]` 至少保留 1 个版本：
- 检测 `op_id` 是否为已知的操作族子命令
- 如果不是子命令，回退到旧格式分发
- 输出 deprecation warning 提示新格式

### REQ-006：CLI 注册层重构（P0）

`cli.py` 需支持嵌套子命令注册：
- 当前 `CommandSpec` 仅支持单层子命令
- 新增 `CommandGroup` 概念支持层级结构
- `_register_subcommand` 需支持嵌套 `add_subparsers()`

---

## Scenario Gray Areas

### GA-1：嵌套深度

| 选项 | 示例 | 优点 | 缺点 |
|------|------|------|------|
| A: 二级 | `ptm-atomic run ospf-config-global` | 简单，迁移成本低 | 名称冗长，扩展性差 |
| B: 三级 | `ptm-atomic run ospf config global` | 自然层级，可探索 | 实现复杂度增加 |
| **C: 混合** | `ptm-atomic run ospf config global`（复杂族）+ `ptm-atomic run auth login`（简单族） | **推荐**：灵活匹配复杂度 | 需要设计准则 |

### GA-2：共享参数传递机制

| 选项 | 说明 | 优点 | 缺点 |
|------|------|------|------|
| A: argparse parents | 使用 `parents=[shared_parser]` | argparse 原生支持 | 继承行为不够透明 |
| **B: 全局参数 + 独立定义** | run 层级定义共享参数，子命令用 `getattr` 读取 | 参数定义清晰 | 各子命令需访问父解析器结果 |
| C: 每模块重复声明 | 每个模块 `add_arguments` 中包含共享参数 | 最独立 | 重复代码 |

### GA-3：op_id 兼容策略

| 选项 | 说明 |
|------|------|
| A: 硬切换 | 直接移除 op_id 格式 |
| **B: 兼容过渡** | 保留 op_id 位置参数 1-2 个版本，输出 deprecation warning |
| C: 永久兼容 | 同时支持新旧两种格式 |

---

## 待人工决策清单

| 决策 ID | 类型 | 问题 | 推荐方案 | 备选方案 |
|------|------|------|------|------|
| DQ-1 | architecture | 嵌套深度：二级、三级还是混合？ | **混合**：复杂族三级，简单族二级 | A: 全部三级；B: 全部二级 |
| DQ-2 | implementation | 共享参数传递机制？ | **方案 B**：run 层级定义 + 子命令读取父解析器 | A: argparse parents；C: 每模块重复 |
| DQ-3 | implementation | 旧格式兼容策略？ | **方案 B**：兼容过渡 1-2 版本 + deprecation warning | A: 硬切换；C: 永久兼容 |
| DQ-4 | scope | 是否同时执行错误输出统一（REQ-004）？ | **是**：趁机消除重复代码 | 否：缩小变更面 |

---

## 自动预检结果

| 检查项 | 结果 |
|------|:---:|
| 需求条目 ≥ 4 | ✅ (6) |
| 覆盖核心场景 U-001/002 | ✅ |
| 覆盖维护场景 U-003/004 | ✅ |
| 识别 ≥ 3 个灰区 | ✅ (3) |
| 待决策项 ≥ 2 | ✅ (4) |
| 推荐方案已标注 | ✅ |
| 无安全/权限/外部接口边界问题 | ✅ |
| 已存 REQUIREMENTS.md、USE-CASES.md 可增量更新 | ✅ |

## Exit Criteria

- [x] 需求条目完整（6 条）
- [x] 灰区已识别（3 个）
- [x] 待人工决策清单已生成（4 项）
- [ ] ⏳ 待 CP2 人工确认

## 结论

PASS（自动预检） — 进入 CP2 人工确认。
