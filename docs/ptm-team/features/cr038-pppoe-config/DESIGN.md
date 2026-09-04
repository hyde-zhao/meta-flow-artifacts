---
doc_type: FEATURE-DESIGN
id: cr038-pppoe-config
feature_id: F-CR038-E2
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
source_hld: docs/design/HLD-CR-038.md §10.4
---

# Feature 设计 — PPPoE 配置（F-CR038-E2）

## 1. 目标与边界

- **目标**：SW 节点 PPPoE Server 数据源契约 + 三厂商命令族 + H3C 真机下发（dry-run 默认）+ PPPoE Client op 映射。
- **范围**：STORY-038-03（commands + schema）、STORY-038-04（switch_configurator）、STORY-038-05（op_mapper）。
- **非目标**：NGFW Web / trex 真机下发（独立 runtime_authorization）；不产出生产明文凭据。

## 2. PPPoE Server 数据源契约（physical_pool SW 节点）

```yaml
# physical_pool.yaml 内 SW 节点扩展
- node_type: SW
  node_name: sw1
  management:
    host: "192.168.0.10"
    vendor: h3c
  role: pppoe-server
  pppoe_server:
    ip_pool_name: pool_pppoe
    ip_pool_subnet: "10.10.10.0/24"
    ip_pool_gateway: "10.10.10.1"
    virtual_template: 1
    authentication: chap
    local_users:
      - username: testuser
        password_cipher: "${ENV_PPPOE_USER_PASSWORD}"   # 密文占位，禁止明文
```

- `password_cipher` 必须 `${ENV_*}` 占位或设备 cipher 密文，禁止明文（ADR-02）。
- 缺 `ip_pool_name`/`ip_pool_subnet` → 结构化错误 `PPPOE_SERVER_INCOMPLETE`。

## 3. 三厂商命令族（commands.py 新增 PPPoE 键）

| vendor | 命令键 |
|---|---|
| h3c | `pppoe.enable_server` / `pppoe.ip_pool` / `pppoe.local_user` / `pppoe.virtual_template` / `pppoe.bind_interface` |
| ruijie | 同键名，命令模板按厂商差异 |
| huawei | 同键名，命令模板按厂商差异 |

- H3C 为参考实现（真实 telnet），ruijie/huawei 仅 `--dry-run` 生成 diff 供人工核对（RA-038-007 降级）。

## 4. 下发流程（switch_configurator.configure_pppoe_server）

1. `from_pool` 读取 SW 节点 + `pppoe_server` 块。
2. 构建命令序列（ip pool → local-user → virtual-template → bind interface），复用 `_build_vlan_commands` 的模板渲染模式。
3. `--dry-run`：打印 diff（默认）；真机：走独立 runtime_authorization 后 H3C telnet。
4. 幂等：重复下发不重复建 pool（先查再建，或按 existing 判定）。
5. 失败路径：vendor 不支持 / 缺 pppoe_server 块 → 结构化 reason。

## 5. PPPoE Client op 映射（STORY-038-05）

- op_mapper 映射 `fw_config_pppoe_client`（逻辑 intent）。
- op 缺失三选一（DQ-038-03）：**默认 fallback=atomic-skip 并写 reason**，可切 `abort`；若 ptm-atomic 提供对等 op 则走原子化。

## 6. 接口契约

| 接口 | 变更 | 消费方 |
|---|---|---|
| `COMMANDS['h3c']['pppoe.*']` | 新增 | switch_configurator |
| `SwitchConfigurator.configure_pppoe_server` | 新增 | STORY-038-04 / ptm-te 编排 |
| `op_mapper.map(intent=pppoe_client)` | 新增 | STORY-038-05 |

## 7. 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 0.1 | 2026-08-15 | meta-se | 初稿：pppoe_server schema + 三厂商命令族 + 下发流程 + op 映射 |
