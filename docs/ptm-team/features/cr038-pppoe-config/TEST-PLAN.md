---
doc_type: FEATURE-TEST-PLAN
id: cr038-pppoe-config-testplan
feature_id: F-CR038-E2
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
---

# Feature 测试计划 — PPPoE 配置（F-CR038-E2）

## 1. 测试对象

- `skills/topo-config/src/commands.py`（三厂商 PPPoE 命令族）
- `skills/topo-config/src/switch_configurator.py`（configure_pppoe_server）
- `skills/ptm-atomic/src/op_mapper.py`（PPPoE Client op 映射）

## 2. 测试场景（来源 SCENARIOS-CR-038.yaml）

| 场景 ID | 类别 | 覆盖点 |
|---|---|---|
| SCN-038-POS-003 | 正向 | H3C PPPoE Server 配置 dry-run 生成命令 |
| SCN-038-POS-004 | 正向 | 三厂商命令模板差异正确 |
| SCN-038-REC-001 | 幂等 | 重复下发不重复建 pool/local-user |
| SCN-038-POS-005 | 正向 | PPPoE Client op 映射 |
| SCN-038-PRE-001 | 前置 | op 缺失 fallback=atomic-skip 写 reason |

## 3. 安全校验

| 用例 | 断言 |
|---|---|
| 凭据占位 | pppoe_server.local_users[].password_cipher 仅接受 `${ENV_*}` 或 cipher 密文，拒绝明文 |
| 密文保留 | 设备 cipher 密文原样透传，不反解 |

## 4. 通过标准

- 全部正向/幂等/前置场景 PASS。
- 安全校验用例 PASS（ADR-02）。
- H3C dry-run 生成命令与 `_build_vlan_commands` 模板渲染一致性。
