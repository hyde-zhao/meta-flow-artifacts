---
story_id: STORY-038-05
story_slug: pppoe-client-op
cr_id: CR-038
feature_id: F-CR038-E2
lld_policy_required_level: full-lld
design_evidence_type: full-lld
tier: S
status: lld-ready-for-review
source_card: process/stories/STORY-038-05.md
source_feature_design: docs/features/cr038-pppoe-config/DESIGN.md
source_hld: docs/design/HLD-CR-038.md
source_adr: docs/design/ARCHITECTURE-DECISION-CR-038.md
source_requirements: docs/product/REQUIREMENTS-CR-038.md
---

# STORY-038-05 LLD — PPPoE Client 拨号 op（op_mapper 映射）

## 0. 工程依据

本 Story 是 F-CR038-E2（PPPoE 配置）的 Client 侧逻辑意图映射层，闭环 RA-038-001「ptm-atomic PPPoE op 是否已暴露」与 DQ-038-03「op 缺失三选一」。

- **HLD**：`op_mapper` 逻辑意图 → 逻辑 op_id 单点维护，本次新增 `pppoe_client`（等价 `fw_config_pppoe_client`）。
- **ADR**：ptm-atomic 未暴露 PPPoE op 时默认 `fallback=atomic-skip` + 写 `reason`，可切 `abort`；若 ptm-atomic 提供对等 op 则原子化。三条路径均留 `reason`，禁止静默跳过。
- **Feature DESIGN（cr038-pppoe-config）**：RA-038-001 外部依赖登记（UNAVAILABLE_OPS 固化 `external_dep_ref`）。
- **REQUIREMENTS**：R-F-005（PPPoE Client op）。

**文档结构导览（14 段语义要点对齐）**：§1 目标与需求，§2 文件影响范围（代码结构），§3 接口设计（API），§4 数据模型，§5 核心流程与技术细节，§6 异常处理，§7 测试设计，§8 实施步骤，§9 回滚策略，§10 安全约束，§11 需求契约映射，§12 依赖与前置，§13 风险与开放项，§14 验收与交接（DoD）。

---

## 1. 背景与目标

### 1.1 目标（Goal）

- 在 `skills/policy-route-execution/scripts/op_mapper.py` 新增逻辑意图映射入口，将 `pppoe_client`（逻辑 intent，等价逻辑 op_id `fw_config_pppoe_client`）映射到 ptm-atomic 原子操作或确定性降级路径。
- 闭环 RA-038-001 与 DQ-038-03「op 缺失三选一」：**默认 `fallback=atomic-skip` 并写 `reason`**，可切 `abort`；若 ptm-atomic 提供对等 op 则走原子化。三条路径均留 `reason` 可追溯，禁止静默跳过。
- 实现 R-F-005（PPPoE Client 拨号归属 ptm-atomic 执行层，op_mapper 增补映射）。

### 1.2 功能需求（Functional Requirements）

| 编号 | 需求 | 量化可验证条件 |
|---|---|---|
| FR-1 | `map_intent(intent="pppoe_client", ...)` 返回映射结果 | 输入 `intent="pppoe_client"` 必返回含 `intent`/`logical_op_id`/`available`/`reason` 的 dict |
| FR-2 | op 缺失走 fallback 并写 reason | `available=False` 且 `fallback="atomic-skip"` 时，返回 `reason` 非空字符串 + `manual_steps` 非空列表 |
| FR-3 | fallback 可切 abort | `fallback="abort"` 且 op 缺失时抛 `OpNotFoundError`（或返回 `error_type=OP_NOT_FOUND` 的 envelope） |
| FR-4 | op 存在走原子化 | `available=True` 时返回 `op_id`/`family`/`action` 三元组，供现有 `build_command`/`execute_op` 消费 |
| FR-5 | 禁止静默跳过 | 任一降级路径的返回 dict 必须含 `reason` 字段，值为非空字符串，说明探测结论与三选一选择 |

### 1.3 非功能需求（Non-Functional Requirements）

| 编号 | 需求 | 约束 |
|---|---|---|
| NFR-1 | 确定性判定 | op 可用性判定只依赖静态映射表 + 静态快照，不依赖运行时网络，不引入 `ptm-atomic` 运行时调用 |
| NFR-2 | 无副作用 | `map_intent` / 手工步骤生成 / 探测均为纯函数，不写设备、不写台账、不落盘 |
| NFR-3 | 向后兼容 | 现有 26 个 op 的映射、`EXPECTED_OP_COUNT=26`、`validate_mapping_consistency` 行为不变 |
| NFR-4 | 性能 | 意图映射为 O(1) 字典查找，无 subprocess，单次调用开销 < 1ms |
| NFR-5 | 凭据安全 | 拨号密码仅接受 `${ENV_*}` 占位或 cipher 密文，禁止明文（ADR-02） |

## 2. 文件影响范围

### 2.1 模块拆分与职责

| 模块/符号 | 归属文件 | 职责 | 变更类型 |
|---|---|---|---|
| `INTENT_TO_OP` | op_mapper.py | 逻辑意图 → 逻辑 op_id 映射表（单点维护） | 新增 |
| `UNAVAILABLE_OPS` | op_mapper.py | 已核实不可用的逻辑 op 快照（探测结论 + fallback 配置 + reason + 外部依赖 ref） | 新增 |
| `map_intent()` | op_mapper.py | 意图 → op 映射 + op 缺失三选一分发（核心入口） | 新增 |
| `_probe_op_availability()` | op_mapper.py | 确定性探测 op 可用性（静态表 + 快照，无 subprocess） | 新增 |
| `_build_manual_steps()` | op_mapper.py | 按意图模板产出手工步骤清单（凭据占位） | 新增 |
| `_assert_no_plaintext_secret()` | op_mapper.py | 拨号凭据明文校验（复用 ADR-02 语义） | 新增 |
| 现有 `OP_ID_TO_SUBCOMMAND` / `ARGS_TO_FLAGS` / `REQUIRED_FLAGS` / `ROLLBACK_STRATEGY` / `OP_METADATA` / `EXPECTED_OP_COUNT=26` | op_mapper.py | 现有映射真相源，保持不变 | 不变 |

### 2.2 输出文件（本 Story 唯一写入范围）

| 文件 | 类型 | 变更 |
|---|---|---|
| `skills/policy-route-execution/scripts/op_mapper.py` | primary | 新增意图映射层（§3 接口） |
| `tests/test_op_mapper_pppoe_client.py` | 测试 | 新增单测（§7） |

> 边界：不修改 `skills/ptm-atomic/**`（本仓库无此目录，仅 `skills/policy-route-execution/`）、不修改 ptm-atomic 本体仓库 `/home/hyde/projects/ptm-atomic/**`、不修改 `docs/ptm-te/op-coverage-matrix.md`（外部依赖台账落点见 §13 灰区 LCQ-STORY-038-05-01）。

### 2.3 代码结构（不改变现有函数签名）

新增符号全部为**新增函数/新增模块级常量**，不改动现有 `map_op_id_to_subcommand` / `map_args_to_flags` / `build_command` / `execute_op` / `validate_mapping_consistency` 的签名与行为，保证向后兼容（NFR-3）。

## 3. 接口设计（op_mapper 如何映射 PPPoE 拨号 op）

### 3.1 新增接口签名

```python
def map_intent(
    intent: str,
    args: dict,
    *,
    fallback: str = "atomic-skip",   # 可选 "atomic-skip" | "abort"
) -> dict:
```

### 3.2 返回结构（dict）

`available=True`（原子化路径，供 `build_command`/`execute_op` 消费）：

```json
{
  "intent": "pppoe_client",
  "logical_op_id": "fw_config_pppoe_client",
  "available": true,
  "fallback": null,
  "op_id": "fw_config_pppoe_client",
  "family": "<family>",
  "action": "<action>",
  "reason": "ptm-atomic 暴露 fw_config_pppoe_client op，走原子化",
  "manual_steps": []
}
```

`available=False` 且 `fallback="atomic-skip"`（降级路径）：

```json
{
  "intent": "pppoe_client",
  "logical_op_id": "fw_config_pppoe_client",
  "available": false,
  "fallback": "atomic-skip",
  "op_id": null,
  "family": null,
  "action": null,
  "reason": "ptm-atomic 未暴露 fw_config_pppoe_client op，且 fw_config_interface 的 --interface-kind 枚举 {bvi,sub,physical,bond,tunnel} 不含 pppoe-client；降级手工步骤（外部依赖 follow-up 已登记）",
  "manual_steps": ["...", "..."]
}
```

`available=False` 且 `fallback="abort"`：抛 `OpNotFoundError`（消息含 reason，复用现有 `OpNotFoundError` 类型，避免新增异常类型）。

### 3.3 意图映射表（常量）

```python
# 逻辑意图 -> 逻辑 op_id（单点维护，本次仅 pppoe_client）
INTENT_TO_OP: Dict[str, str] = {
    "pppoe_client": "fw_config_pppoe_client",
}
```

### 3.4 已核实不可用 op 快照（常量）

```python
UNAVAILABLE_OPS: Dict[str, Dict[str, Any]] = {
    "fw_config_pppoe_client": {
        "probe_conclusion": "missing",
        "fallback": "atomic-skip",      # 默认；可切 abort
        "reason": (
            "ptm-atomic 无独立 fw_config_pppoe_client op；"
            "fw_config_interface 的 --interface-kind choices={bvi,sub,physical,bond,tunnel} 不含 pppoe-client "
            "（run_interface.py L34 / runner/interface.py INTERFACE_KINDS L28）；"
            "fw_config_interface.yaml aliases 含 pppoe/pppoe-client 仅为 validate alias_suggestions 反向查找"
        ),
        "external_dep_ref": "CR-038-FOLLOW-UP: ptm-atomic 扩展 --interface-kind=pppoe-client（含拨号 username/password）或新增独立 fw_config_pppoe_client op",
        "manual_steps_template": "pppoe_client_manual_steps",
        "truth_source": (
            "ptm-atomic run_interface.py L34; runner/interface.py L28; "
            "atoms/fw/fw_config_interface.yaml aliases; docs/naming-convention.md L115"
        ),
        "probed_at": "2026-08-15",
    },
}
```

### 3.5 手工步骤模板（常量）

```python
PPPOE_CLIENT_MANUAL_STEPS: List[str] = [
    "1. 登录 NGFW Web 管理界面（https://<dut-host>，独立 runtime_authorization）",
    "2. 进入 网络 > 接口，选择 TE0_8",
    "3. 接口模式改为 PPPoE Client",
    "4. 填写拨号账号 username（非敏感值，来自 args['username']）",
    "5. 拨号密码从环境变量 ${ENV_PPPOE_USER_PASSWORD} 注入（禁止明文）",
    "6. 保存并拨号，验证接口获 163.0.0.x 地址",
]
```

> 模板内只含 `${ENV_*}` 占位符，不含任何明文密码（NFR-5 / §10）。

### 3.6 消费方集成契约

| 字段 | 内容 |
|---|---|
| 调用方向 | ptm-te 编排层 / case_runner → `op_mapper.map_intent` |
| 调用时机 | 用例 case_steps 声明 `intent=pppoe_client` 且需经 ptm-atomic 下发时 |
| 触发方式 | 显式函数调用（非隐式） |
| 输入契约 | `intent`（str，枚举 `{"pppoe_client"}`）+ `args`（dict，含 `username`/`password` 等）+ `fallback`（可选 `"atomic-skip"` 默认 / `"abort"`） |
| 输出契约 | 上述返回 dict（`available`/`fallback`/`reason`/`manual_steps`/`op_id`/`family`/`action`） |
| 后续衔接 | `available=True` → 调 `build_command(op_id, args, ...)` + `execute_op(...)`；`available=False` + atomic-skip → 输出 `manual_steps` 供人工/编排层消费 |
| 降级策略 | atomic-skip（默认产手工步骤）/ abort（抛 OpNotFoundError），见 §6 |
| 调用方需同步 | 编排层需在 `available=False` 且 atomic-skip 时处理 `manual_steps` 字段（新增处理分支，非本 Story 范围） |

## 4. 数据模型

### 4.1 无持久化设计

本 Story 为**纯映射/探测函数**，无数据库、无文件落盘、无新持久化对象。唯一「数据」是模块级常量（`INTENT_TO_OP` / `UNAVAILABLE_OPS` / `PPPOE_CLIENT_MANUAL_STEPS`），随 op_mapper.py 一起维护。

### 4.2 常量数据结构

| 常量 | key 类型 | value 类型 | 说明 |
|---|---|---|---|
| `INTENT_TO_OP` | `str`（intent） | `str`（逻辑 op_id） | 1:1 映射 |
| `UNAVAILABLE_OPS` | `str`（逻辑 op_id） | `dict`（`probe_conclusion`/`fallback`/`reason`/`external_dep_ref`/`manual_steps_template`/`truth_source`/`probed_at`） | op 缺失快照，单点维护 |
| `PPPOE_CLIENT_MANUAL_STEPS` | `int`（序号，隐式 list 索引） | `str`（步骤文本） | 手工步骤模板 |

### 4.3 与现有数据模型的关系

- 不改变 `OP_ID_TO_SUBCOMMAND` / `ARGS_TO_FLAGS` / `REQUIRED_FLAGS` / `ROLLBACK_STRATEGY` / `OP_METADATA` 现有 26 个 op 的条目与 `EXPECTED_OP_COUNT=26`。
- `fw_config_pppoe_client` **不进入** `OP_ID_TO_SUBCOMMAND` / `ARGS_TO_FLAGS` / `ROLLBACK_STRATEGY` / `OP_METADATA`（因为它不可用，进入会导致 `validate_mapping_consistency` 四表集合校验失败）。它只进入 `UNAVAILABLE_OPS` 快照。

## 5. 核心流程（含确定性判定逻辑）

### 5.1 `map_intent` 处理流程

```
输入 intent, args, fallback
  │
  ├─ [1] intent 不在 INTENT_TO_OP → 抛 OpNotFoundError("未识别 intent: <intent>")
  │
  ├─ [2] 取 logical_op_id = INTENT_TO_OP[intent]
  │
  ├─ [3] _assert_no_plaintext_secret(intent, args)   # 拨号凭据明文校验（ADR-02）
  │       明文密码 → 抛 ValidationError("password 禁止明文，需 ${ENV_*} 占位或 cipher 密文")
  │
  ├─ [4] _probe_op_availability(logical_op_id) → {available, reason, fallback_src}
  │
  ├─ [5] 若 available=True → 返回原子化 dict（op_id/family/action，reason 写"已暴露，走原子化"）
  │
  └─ [6] 若 available=False：
         ├─ fallback == "abort" → 抛 OpNotFoundError(reason)
         └─ fallback == "atomic-skip" → 返回降级 dict（manual_steps + reason）
```

### 5.2 确定性探测逻辑 `_probe_op_availability`

判定优先级（**确定性，无 subprocess，无运行时网络**）：

```
1. op_id in OP_ID_TO_SUBCOMMAND        → available=True,  path="mapped"
2. op_id in UNAVAILABLE_OPS            → available=False, path="snapshot"（用快照 reason）
3. 其他                                 → available=False, path="unknown"（保守降级，reason 写"未映射且无快照"）
```

- 设计决策：**运行时不调 `ptm-atomic list` / `ptm-atomic show`**。依据：`docs/ptm-te/op-coverage-matrix.md` 已明确「`ptm-atomic show` 不可信（与 yaml 不同步）」，且运行时调用会引入 `ptm-atomic` 二进制依赖、破坏测试环境可确定性（NFR-1）。
- 快照 `UNAVAILABLE_OPS` 是**设计期实测结论固化**（§13 列真相源与探测日期），LLD 阶段已核实，运行时直接消费。
- `path="unknown"` 分支保守降级为 `available=False`（宁降级不静默原子化），reason 明确「op_id 未映射且无不可用快照，需人工核实 ptm-atomic 是否暴露」。

### 5.3 三选一判定结论（确定性）

| 分支 | 判定条件 | 动作 | 落地 |
|---|---|---|---|
| (a) 映射到可用 op | `op_id in OP_ID_TO_SUBCOMMAND` | 原子化 | 当前 `fw_config_pppoe_client` 不命中（不在表内） |
| (b) 提外部依赖 | `op_id in UNAVAILABLE_OPS` 且快照 `external_dep_ref` 非空 | 登记 follow-up 台账（编排层汇总，本 Story 仅固化 `external_dep_ref` 字符串） | 命中，落 `UNAVAILABLE_OPS['fw_config_pppoe_client']` |
| (c) 降级手工步骤 | `available=False` 且 `fallback="atomic-skip"` | 返回 `manual_steps` + `reason` | 命中，默认路径 |

**最终结论：默认走 (c) atomic-skip（运行时降级手工步骤）+ 登记 (b) 外部依赖 follow-up；(a) 为 ptm-atomic 补 op 后的自动激活路径（移除快照条目 + 在 `OP_ID_TO_SUBCOMMAND` 补映射后即自动命中）。**

### 5.4 探测结论的事实依据（LLD 阶段已核实）

| 事实 | 来源 | 结论 |
|---|---|---|
| ptm-atomic 无 `fw_config_pppoe_client` op | `/home/hyde/projects/ptm-atomic/atoms/**` 全量 grep 无此 op_id | 独立 op 不存在 |
| `--interface-kind` choices = `("bvi","sub","physical","bond","tunnel")` | `ptm-atomic/src/atomic_ops/commands/run_interface.py` L34 | 不含 `pppoe-client` |
| runner `INTERFACE_KINDS = {"bvi","sub","physical","bond","tunnel"}` | `ptm-atomic/src/atomic_ops/runner/interface.py` L28 | `_validate_interface_kind` 拒绝非枚举 |
| `fw_config_interface.yaml` aliases 含 `pppoe`/`pppoe-client` | `ptm-atomic/atoms/fw/fw_config_interface.yaml` L93-94 | 仅 alias，非下发能力 |
| `fw_config_pppoe` → `fw_config_interface`（PPPoE 是 intf 的 IP 地址获取模式，非独立资源） | `ptm-atomic/docs/naming-convention.md` L115 | 印证无独立 op |
| alias 机制用途 = validate `OP_NOT_FOUND` 的 `alias_suggestions` | `ptm-atomic/CHANGELOG.md` L43-45 | alias 只做反向查找提示 |

## 6. 异常处理（op 缺失时的降级路径）

### 6.1 异常/降级路径表

| 场景 | 触发条件 | 行为 | 返回/异常 |
|---|---|---|---|
| intent 未识别 | `intent not in INTENT_TO_OP` | 终止 | `OpNotFoundError("未识别 intent: <intent>")` |
| 拨号凭据明文 | args 含明文 password | 终止（拒绝） | `ValidationError("password 禁止明文，需 ${ENV_*} 占位或 cipher 密文")` |
| op 已暴露 | `op_id in OP_ID_TO_SUBCOMMAND` | 原子化 | 返回 `available=True` dict |
| op 缺失（快照） | `op_id in UNAVAILABLE_OPS` 且 `fallback="atomic-skip"` | 降级 | 返回 `available=False` + `manual_steps` + `reason` |
| op 缺失（快照）+ abort | 同上 且 `fallback="abort"` | 终止 | `OpNotFoundError(reason)` |
| op 未知（无映射无快照） | 表 3 之外 | 保守降级 | 返回 `available=False` + `reason="op_id 未映射且无不可用快照"`，`manual_steps=[]` |

### 6.2 降级路径（op 缺失，默认 atomic-skip）

```
map_intent("pppoe_client", args, fallback="atomic-skip")
  → _probe_op_availability("fw_config_pppoe_client") = {available=False, path="snapshot"}
  → fallback="atomic-skip"
  → 返回 {available=False, fallback="atomic-skip", reason=<快照 reason>,
          manual_steps=_build_manual_steps("pppoe_client_manual_steps", args)}
```

### 6.3 abort 路径

```
map_intent("pppoe_client", args, fallback="abort")
  → available=False
  → 抛 OpNotFoundError(<快照 reason>)
```

- `OpNotFoundError` 复用现有类型（op_mapper.py L31-33），不新增异常类型。
- 由编排层决定 `fallback` 取值；未传时默认 `atomic-skip`（NFR-2 无副作用，DQ-038-03 默认）。

## 7. 测试设计

### 7.1 测试文件与框架

- 新增 `tests/test_op_mapper_pppoe_client.py`（仓库根 `tests/`，与 `test_op_mapper_rollback.py` / `test_tg_op_mapping.py` 同目录同约定）。
- 框架：`unittest` + `unittest.mock`，通过 `importlib.util.spec_from_file_location` 加载 `OP_MAPPER_PATH = REPO_ROOT / "skills/policy-route-execution/scripts/op_mapper.py"`（沿用现有测试 `load_module` 约定）。
- 验证入口：`uv run --python 3.11 pytest tests/test_op_mapper_pppoe_client.py`。

### 7.2 测试用例矩阵

| 用例 ID | 覆盖场景 | 断言 |
|---|---|---|
| TC-POS-005-01 | SCN-038-POS-005 映射层正向 | `map_intent("pppoe_client", args, fallback="atomic-skip")` 返回 dict；`available=False`；`reason` 非空且含 `"ptm-atomic"` 与 `"pppoe-client"`；`manual_steps` 长度 ≥ 5 |
| TC-POS-005-02 | (a) 原子化分支（mock 未来 ptm-atomic 补 op） | 将 `fw_config_pppoe_client` 临时插入 `OP_ID_TO_SUBCOMMAND`（`mock.patch.dict`）后，`map_intent` 返回 `available=True`，`op_id="fw_config_pppoe_client"`，`family`/`action` 正确 |
| TC-PRE-001-01 | SCN-038-PRE-001 可用性预检 | `_probe_op_availability("fw_config_pppoe_client")` 返回 `{available=False, path="snapshot"}`；reason 含 `external_dep_ref` 语义（跨仓库依赖）或手工降级语义 |
| TC-PRE-001-02 | 已映射 op 可用性探测 | `_probe_op_availability("fw_config_policy_route")` 返回 `{available=True, path="mapped"}` |
| TC-RA-001-01 | RA-038-001 闭环：缺失走 fallback 写 reason | `map_intent(..., fallback="atomic-skip")` 返回 `available=False` 且 `reason` 非空且 `manual_steps` 非空 |
| TC-RA-001-02 | RA-038-001 闭环：存在走原子化 | `map_intent` 在 `OP_ID_TO_SUBCOMMAND` 含 op 时返回 `available=True` 且 `reason` 非空 |
| TC-ABORT-01 | fallback=abort | `map_intent("pppoe_client", args, fallback="abort")` 抛 `OpNotFoundError`，消息含 `reason` |
| TC-SEC-01 | 凭据占位校验（§10） | `map_intent("pppoe_client", {"password": "明文123"}, ...)` 抛 `ValidationError`；`{"password": "${ENV_PPPOE_USER_PASSWORD}"}` 不抛 |
| TC-NFR-03-01 | 向后兼容 | `validate_mapping_consistency().passed == True` 且 `EXPECTED_OP_COUNT == 26`（新增符号不破坏现有 26 op 四表一致性） |
| TC-INTENT-01 | intent 未识别 | `map_intent("unknown_intent", {})` 抛 `OpNotFoundError` |

### 7.3 未覆盖项（本 Story 边界）

- SCN-038-POS-005 的端到端「NGFW 接口拨号获 163.0.0.x」属真机 `runtime_authorization`，本 Story 不覆盖（独立授权，由 CP7 meta-qa 在真机阶段验证）。
- 编排层对 `manual_steps` 字段的消费逻辑属调用方（ptm-te 编排 / case_runner），非本 Story 文件所有权。

## 8. 实施步骤（最小切片）

### 8.1 切片划分

| Slice | TASK-ID | 内容 | 产物 | 验证 |
|---|---|---|---|---|
| S1 | TASK-E2-05-01 | 新增 `INTENT_TO_OP` + `UNAVAILABLE_OPS` 快照 + `_probe_op_availability` + `map_intent` 的 available/unknown 分支 | op_mapper.py | TC-PRE-001-01/02、TC-POS-005-02 |
| S2 | TASK-E2-05-02 | 新增 `_build_manual_steps` + `PPPOE_CLIENT_MANUAL_STEPS` + `_assert_no_plaintext_secret` + `map_intent` 的 fallback=atomic-skip/abort 分支 | op_mapper.py | TC-POS-005-01、TC-RA-001-01/02、TC-ABORT-01、TC-SEC-01、TC-INTENT-01 |
| S3 | TASK-E2-05-03 | 新增 `tests/test_op_mapper_pppoe_client.py` 全部用例 + 回归 | tests/ | 全部用例 PASS + `validate_mapping_consistency` PASS |

### 8.2 实施顺序约束

- S1 → S2 → S3 串行（同一文件 op_mapper.py，单写）。
- S3 依赖 S1+S2 的接口稳定后才写断言（避免测试与实现同步返工）。

### 8.3 每切片前置校验

| Slice | 前置条件 |
|---|---|
| S1 | 本 LLD 已确认（CP5 approved）；`INTENT_TO_OP`/`UNAVAILABLE_OPS` 常量结构与 §4.2 一致 |
| S2 | S1 通过；`map_intent` 返回结构含 `available`/`fallback`/`reason`/`manual_steps` 字段 |
| S3 | S1+S2 通过；测试用 `mock.patch.dict` 注入 (a) 原子化分支，不触网 |

## 9. 回滚策略

- **代码回滚**：本 Story 为纯函数新增，无运行时副作用。回滚 = `git revert` 本 Story 提交，删除 `INTENT_TO_OP`/`UNAVAILABLE_OPS`/`map_intent`/`_build_manual_steps`/`_assert_no_plaintext_secret` 及测试文件，现有 26 op 行为不变。
- **运行时回滚**：不适用。`map_intent` 不写设备、不写台账、不落盘（NFR-2）。
- **降级可逆**：若 ptm-atomic 未来补 `fw_config_pppoe_client` op，从 `UNAVAILABLE_OPS` 删除对应条目并在 `OP_ID_TO_SUBCOMMAND`/`ARGS_TO_FLAGS` 补映射，即从 (c) 切回 (a)（§13 重访条件）。

## 10. 安全约束（PPPoE 拨号凭据占位、默认 dry-run）

### 10.1 拨号凭据占位（ADR-02）

- `map_intent` 在探测前调用 `_assert_no_plaintext_secret(intent, args)`，校验 args 中 `password`/`password_cipher`/`password_env` 字段：
  - 值以 `${ENV_` 开头且以 `}` 结尾 → 通过（占位）
  - 值为设备 cipher 密文（非 `${ENV_*}` 但非空）→ 通过（密文透传，不反解）
  - 值为明显明文（含可读用户名/常见口令形态）→ 抛 `ValidationError`（禁止明文）
- `PPPOE_CLIENT_MANUAL_STEPS` 模板只含 `${ENV_PPPOE_USER_PASSWORD}` 占位符，不含任何明文密码。
- 禁止明文写入 op 映射表（`INTENT_TO_OP` / `UNAVAILABLE_OPS` 只存 op_id/ref 字符串，不存凭据值）。

### 10.2 默认 dry-run

- `map_intent` 本身**不执行** ptm-atomic、不发真机请求（纯映射/探测/降级）。
- 真机下发仅经现有 `execute_op` 走 `dry_run` 默认门（`dry_run=True` 默认）与 `authorized` 门（`dry_run=False` 需 `authorized=True`），本 Story 不绕过、不修改该门。
- NGFW Web 拨号为独立 `runtime_authorization`（HLD §1.4），CP 批准不隐含。

### 10.3 性能设计

- 意图映射 O(1)；探测为字典查找 O(1)；无 subprocess、无网络。单次调用 < 1ms。

## 11. 契约映射（R-F-005 → 实现点）

| 契约 | 需求原文（REQUIREMENTS-CR-038.md） | 实现点 | 验收场景 |
|---|---|---|---|
| R-F-005 | PPPoE Client 拨号归属 ptm-atomic 原子操作（执行层）：新增/复用 `fw_config_pppoe_client` op，op_mapper 增补映射 | `INTENT_TO_OP["pppoe_client"]="fw_config_pppoe_client"` + `map_intent` 原子化分支（§3/§5） | SCN-038-POS-005（TC-POS-005-01/02） |
| RA-038-001 | ptm-atomic CLI 本体未暴露 PPPoE 拨号 op，需跨仓库新增 | `UNAVAILABLE_OPS` 快照 + `external_dep_ref`（登记外部依赖）+ `_probe_op_availability`（§3.4/§5） | SCN-038-PRE-001（TC-PRE-001-01）、TC-RA-001-01/02 |
| DQ-038-03 | 默认 fallback=atomic-skip 写 reason，可切 abort | `map_intent` 的 `fallback` 参数 + `_build_manual_steps` + abort 抛 `OpNotFoundError`（§3/§5/§6） | TC-ABORT-01、TC-POS-005-01 |
| ADR-02 | 凭据 `${ENV_VAR}` 占位、cipher 密文、禁止明文 | `_assert_no_plaintext_secret` + 手工步骤模板占位（§10） | TC-SEC-01 |
| R-C-006 | 拨号地址 163.0.0.x 不进 env-file 静态 IPAM（本 Story 边界外，由 STORY-038-06 exporter 保证） | 本 Story 不涉及 IPAM；手工步骤模板仅描述「验证获 163.0.0.x」，不写静态 IPAM | 不适用（跨 Story 边界注明） |

## 12. 依赖与前置

### 12.1 Story 依赖

- `depends_on: []`，`dependency_type: none`（独立外部接口映射，W2）。
- 与 STORY-038-03/04 无文件共享（不同文件、不同层：03=commands.py+physical_pool.py，04=switch_configurator.py，05=op_mapper.py），无 `file-conflict`。

### 12.2 前置条件

| 前置 | 状态 |
|---|---|
| 本 LLD 全量人工确认（CP5 approved） | 待（dev_gate=cp5-approved） |
| ptm-atomic 探测结论固化 | 已完成（§5.4 核实，probed_at=2026-08-15） |
| 现有 op_mapper 测试基线通过 | 前置（`uv run --python 3.11 pytest tests/test_op_mapper_rollback.py tests/test_step_refs.py tests/test_tg_op_mapping.py`） |

### 12.3 外部依赖（跨仓库）

- ptm-atomic 本体 `/home/hyde/projects/ptm-atomic` 需扩展 `--interface-kind=pppoe-client`（含拨号 username/password 参数）或新增独立 `fw_config_pppoe_client` op。本 Story **不修改** ptm-atomic（沿用 CR-033/037 边界），只固化 `external_dep_ref` 登记 follow-up。

## 13. 风险与开放项

### 13.1 风险

| 风险 | 影响 | 应对 | 状态 |
|---|---|---|---|
| RA-038-001 ptm-atomic op 缺失 | Client 拨号无法原子化 | 默认 (c) atomic-skip 手工步骤 + (b) 外部依赖 follow-up（§5.3） | 已闭环（本 LLD 判定） |
| 快照过期 | ptm-atomic 未来补 op 但 `UNAVAILABLE_OPS` 未删，误降级 | 重访条件：ptm-atomic 升级后重新探测并移除快照条目（§13.3） | non-blocking-open |
| alias 误读 | 误把 alias `pppoe` 当作可下发能力 | §5.4 真相源明确 alias 仅为 `alias_suggestions` 反向查找，非下发 | 已规避（文档固化） |

### 13.2 实现灰区与取舍记录

| 灰区 | 选项 | 决策 | 影响面 | 证据/重访条件 |
|---|---|---|---|---|
| (b) 外部依赖登记落点 | A：仅 op_mapper.py 内 `external_dep_ref` 字符串固化，台账由编排层 CP5 后统一汇总；B：本 Story 同时更新 op-coverage-matrix.md + 新建 follow-up 台账 | 推荐 A（不扩大 file_ownership） | op_mapper.py vs op-coverage-matrix.md/台账 | 见 LCQ-STORY-038-05-01（blocks_lld=false） |
| 运行时探测 vs 静态快照 | 运行时调 `ptm-atomic list` vs 静态快照 | 静态快照（`ptm-atomic show` 不可信 + 避免运行时依赖） | 确定性/测试可重复 | op-coverage-matrix.md「`ptm-atomic show` 不可信」 |
| 新函数命名 `map_intent` vs `map` | 与现有 `map_op_id_to_subcommand`/`map_args_to_flags` 区分 | `map_intent`（避免与 argparse `map` 子命令混淆） | 接口命名 | 已定 |

### 13.3 开放项（non-blocking）

| 编号 | 内容 | 状态 | 重访条件 |
|---|---|---|---|
| OPEN-038-05-01 | ptm-atomic 扩展 `--interface-kind=pppoe-client` 后，移除 `UNAVAILABLE_OPS['fw_config_pppoe_client']`，在 `OP_ID_TO_SUBCOMMAND`/`ARGS_TO_FLAGS` 补 `fw_config_pppoe_client`（或映射到 `fw_config_interface`+`interface-kind=pppoe-client`），从 (c) 切回 (a) | non-blocking-open | ptm-atomic 升级或 CP8 终验时复核 |
| LCQ-STORY-038-05-01 | 外部依赖 follow-up 登记落点（§13.2） | open（blocks_lld=false） | host-orchestrator CP5 后汇总 |

## 14. 验收与交接

### 14.1 验收标准

- SCN-038-POS-005（映射层）：`map_intent("pppoe_client", ...)` 正确返回映射/降级结果（TC-POS-005-01/02 PASS）。
- SCN-038-PRE-001（op 可用性预检）：`_probe_op_availability` 判定 `missing` 且 reason 明确「跨仓库依赖 / 手工降级」（TC-PRE-001-01 PASS）。
- RA-038-001 闭环：op 存在走原子化（TC-RA-001-02）、缺失走 fallback 写 reason（TC-RA-001-01），均留 reason 可追溯。
- 安全：凭据占位校验（TC-SEC-01）PASS，无明文密码。
- 向后兼容：`validate_mapping_consistency().passed == True`，`EXPECTED_OP_COUNT == 26`（TC-NFR-03-01）。

### 14.2 Definition of Done

- [ ] `op_mapper.py` 新增 `INTENT_TO_OP`/`UNAVAILABLE_OPS`/`PPPOE_CLIENT_MANUAL_STEPS`/`map_intent`/`_probe_op_availability`/`_build_manual_steps`/`_assert_no_plaintext_secret`，不改变现有 26 op 映射与签名。
- [ ] `tests/test_op_mapper_pppoe_client.py` 全部用例 PASS（`uv run --python 3.11 pytest tests/test_op_mapper_pppoe_client.py`）。
- [ ] 回归：`uv run --python 3.11 pytest tests/test_op_mapper_rollback.py tests/test_step_refs.py tests/test_tg_op_mapping.py` 无失败。
- [ ] `validate_mapping_consistency().passed == True`。
- [ ] 无明文密码落入映射表/手工步骤模板；密码仅 `${ENV_*}` 占位或 cipher 密文。
- [ ] 未修改 `REQUIREMENTS-CR-038.md` / `HLD-CR-038.md` / `ARCHITECTURE-DECISION-CR-038.md` / 蓝图三件套 / ptm-atomic 本体。
- [ ] LLD 三选一判定结论（(c)+(b)，(a) 预留）在实现摘要中可追溯。

### 14.3 交接摘要（给 meta-qa 验证入口）

- LLD 路径：`process/stories/STORY-038-05-pppoe-client-op-LLD.md`
- 实现文件：`skills/policy-route-execution/scripts/op_mapper.py`
- 测试入口：`uv run --python 3.11 pytest tests/test_op_mapper_pppoe_client.py`
- 三选一结论：**默认 (c) atomic-skip + 登记 (b) 外部依赖 follow-up；(a) 为 ptm-atomic 补 op 后自动激活路径**
- 风险提示：端到端真机拨号（获 163.0.0.x）不属本 Story（独立 runtime_authorization）；编排层需新增 `manual_steps` 消费分支（调用方范围）。
