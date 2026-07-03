# CP1 用户场景完备门

| 字段 | 值 |
|------|-----|
| 检查点 | CP1 |
| 状态 | PASS |
| 关联 CR | CR-007 |
| 生成时间 | 2026-06-08T16:15:00+0800 |

## 场景识别

### U-001：用户按操作族执行命令

**角色**：网络测试工程师
**场景**：用户需要对防火墙执行 OSPF 配置操作

**当前体验**：
```bash
# 用户输入
ptm-atomic run fw_config_ospfv2_global --base-url https://10.0.0.1 --enable --router-id 10.113.55.51

# --help 显示 80+ 个参数，包括 policy-route、interface、bfd 的无关参数
```

**期望体验**：
```bash
# 用户输入
ptm-atomic run ospf config global --base-url https://10.0.0.1 --enable --router-id 10.113.55.51

# --help 只显示 ~10 个 ospf config global 相关参数
```

---

### U-002：开发者新增操作族

**角色**：项目维护者
**场景**：需要新增 BGP 操作族（~15 个 op）

**当前体验**：
1. 在 `run.py` 的 `SUPPORTED_OPS` 元组中添加 15 个 op_id
2. 在 `add_arguments()` 中追加 20-30 个参数
3. 在 `run()` 中新增 elif 分支
4. 所有 BGP 参数与现有 ospf/interface/policy-route 参数混在一起
5. 合并冲突风险高

**期望体验**：
1. 创建 `commands/run_bgp.py`，声明自己的参数
2. 在 `cli.py` 中注册 `run bgp` 子命令组
3. 不影响任何现有命令

---

### U-003：用户探索可用操作

**角色**：新用户
**场景**：用户不知道有哪些操作可用，需要逐层探索

**当前体验**：
```bash
$ ptm-atomic run --help
# 显示所有 80+ 参数，无法看出有哪些操作族
```

**期望体验**：
```bash
$ ptm-atomic run --help
# 显示可用操作族：auth, object, policy-route, interface, ospf, ospfv3

$ ptm-atomic run ospf --help
# 显示 ospf 子命令：config, verify, delete

$ ptm-atomic run ospf config --help
# 显示 config 子命令：global, network, interface, redistribution
```

---

### U-004：向后兼容

**角色**：现有用户
**场景**：已编写脚本使用 `ptm-atomic run <op_id>` 格式

**期望**：至少 1-2 个版本保持旧格式兼容，提供迁移指引

---

## Scenario Coverage Checklist

| 场景 | 类型 | 优先级 |
|------|------|--------|
| U-001 按操作族执行命令 | 核心用户场景 | P0 |
| U-002 开发者新增操作族 | 维护者场景 | P0 |
| U-003 用户探索可用操作 | 发现性场景 | P1 |
| U-004 向后兼容 | 迁移场景 | P1 |

## Exit Criteria

- [x] 至少 3 个独立用户场景
- [x] 覆盖核心使用和维护场景
- [x] 向后兼容已纳入考虑

## 结论

PASS — 场景完备，进入 CP2 需求基线。
