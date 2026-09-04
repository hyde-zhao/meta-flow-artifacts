---
story_id: STORY-039-02
story_slug: g4-next-hop-a-placeholder
cr_id: CR-039
wave: 2
taker: meta-dev
implemented_at: 2026-08-19
status: dev-complete
evidence_type: implementation-execution
---

# STORY-039-02 实现执行证据：G4 next_hop_a 第 10 类占位符

## 实现前置检查

| 检查项 | 结果 |
|---|---|
| Story 状态与设计证据 | CP5 已 approve（process/checkpoints/CP5-CR039.md，STORY-039-01~04 全量批次，2026-08-19）；technical-note 已确认为实现基线 |
| 依赖门控 | STORY-039-01（file-ownership 串行，op_mapper.py 首个写入块）已完成 dev-complete；本 Story 为同文件第二个写入块，无写入冲突 |
| 文件所有权 | primary: tests/test_op_mapper_next_hop_a.py（新建）；shared: skills/policy-route-execution/scripts/op_mapper.py（G4 区块写入，merge_owner=STORY-039-03） |
| 基底 | canonical 当前工作区（含 W1 G1 区块：`_query_static_route_id` + rollback 分支），逐块合并，非整文件覆盖 |
| 验证环境 | Python 3.11 + pytest（uv run）；static-only，无网络/真机（CR-039 security_constraints） |

## 实现对象清单

1. `skills/policy-route-execution/scripts/op_mapper.py`（G4 区块，4 处改动）：
   - **A1 分支**（`_resolve_one` 内，紧随 `dut.next_hop` 分支后新增）：`path == "dut.next_hop_a"` -> `dut_node.get("next_hop_a")`，空值抛 `_missing(val, "nodes.dut1.next_hop_a")`；与 `next_hop` 分支逐行同构（工作区 L1566-1569 回源）。
   - **A2 占位符表**（`resolve_env_refs` docstring）：表头「9 类」->「10 类」，新增行 `${ENV.dut.next_hop_a} -> nodes.dut1.next_hop_a（策略路由下一跳1 GW-A）`（工作区 L1488 回源，置于 next_hop 行之后）。
   - **A3 报错文案**（未知占位符路径 raise）：「不在 9 类」->「不在 10 类」；⚠️ 工作区该处未同步（仍 9 类），canonical 按 DQ 口径一并对齐（Story 技术说明文件影响范围第 3 条）。
   - **A4 注释对齐**（2 处，文案级）：模块级注释 L1435「按 9 类占位符解析表替换」->「10 类」；分支段落注释「tg.url / dut.url / dut.next_hop」补「/ dut.next_hop_a」。
2. `tests/test_op_mapper_next_hop_a.py`（新建，primary 所有权）：6 个用例（成功解析 / 缺失字段报错 / 空值报错 / next_hop 回归 / docstring 表校验 / 未知路径报错文案校验）。

## 设计契约映射

| 技术说明契约 | 实现位置 | 验证入口 |
|---|---|---|
| `${ENV.dut.next_hop_a}` -> `env_topology["nodes"]["dut1"]["next_hop_a"]` | A1 分支 | test_resolve_success |
| 缺失（或空）抛 ValueError，消息含 `nodes.dut1.next_hop_a` 与占位符字面量 | A1 `_missing` | test_missing_field_raises / test_empty_field_raises |
| 占位符表第 10 类条目 + 表注释 10 类 | A2 docstring | test_docstring_contains_tenth_class |
| 未知路径报错文案含「10 类」 | A3 raise 文案 | test_unknown_path_error_mentions_ten_classes |
| 与 `next_hop` 分支同构（行为一致） | A1 与 L1537-1541 逐行同构 | test_next_hop_regression |
| `env_topology` 数据契约不变（`nodes.dut1.next_hop_a` 由 env-file 提供，ADR-09） | 未改 load_env_file / case_runner | 单测构造 env dict 与 env-file 字段结构一致 |
| 既有 9 类占位符不回归 | 未触碰 tg.port{N}/url/ports[] 分支 | 回归测试（见验证结果） |

## 单元测试与 Fixture 计划

- Fixture：`_env_with_next_hop_a()` 构造 env_topology（port_mapping/nodes/links），`nodes.dut1` 含 `next_hop=110.0.0.2` / `next_hop_a=80.0.0.2`（对齐工作区 env-file `node3_dut1(DUT2)..._link3(PPPoE).yml` L63-64 字段值）。
- 加载方式：importlib 按 OP_MAPPER_PATH 直载 op_mapper（与 test_op_mapper_rollback.py 同风格），不依赖 sys.path 注入。
- 用例 6 个：
  1. `test_resolve_success`：`${ENV.dut.next_hop_a}` -> `80.0.0.2`（SM-039-03 主场景，op_id=fw_config_policy_route）。
  2. `test_missing_field_raises`：删除 next_hop_a 字段 -> ValueError 含 `nodes.dut1.next_hop_a` 与 `${ENV.dut.next_hop_a}`。
  3. `test_empty_field_raises`：空字符串 -> ValueError（`if not v` 同构语义）。
  4. `test_next_hop_regression`：`${ENV.dut.next_hop}` -> `110.0.0.2`（第 9 类不回归）。
  5. `test_docstring_contains_tenth_class`：docstring 含「10 类」+ 第 10 类条目（文档-代码计数一致性，方案评审规则第 1 条）。
  6. `test_unknown_path_error_mentions_ten_classes`：未知路径报错含「不在 10 类」+「未知路径 dut.unknown_field」。
- mock：无需 mock（resolve_env_refs 纯解析、无 subprocess/网络调用，Gotcha #13）。

## 最小实现切片

- Slice-1（A1+A2+A3+A4，单 commit 粒度的同文件 4 处文案+分支改动）：实现后即跑新增单测（6 passed）。
- Slice-2（回归验证）：rollback 32 + tg_op_mapping 37 + env_unsupported 8 + pppoe_client 18 全量回归（见验证结果）。

## 本地对拍（工作区一致性）

- A1 分支与工作区 `/home/hyde/projects/ptm-te-manaul/.claude/skills/policy-route-execution/scripts/op_mapper.py` L1566-1569 逐行一致（含 `_missing` key_path）。
- A2 表行与工作区 L1488 逐字一致（含括注「策略路由下一跳1 GW-A」与对齐空格）。
- A3/A4 为**有意偏离**：工作区报错文案与 2 处计数注释仍为 9 类，canonical 按 DQ 口径对齐为 10 类（Story 技术说明已预告 A3；A4 为同口径的注释一致性收口，方案评审规则第 1 条要求表注释/报错文案/实际分支三者一致）。

## 验证结果

| 命令 | 结果 |
|---|---|
| `uv run --python 3.11 pytest tests/test_op_mapper_next_hop_a.py -x` | **6 passed**（SM-039-03 覆盖） |
| `uv run --python 3.11 pytest tests/test_op_mapper_rollback.py tests/test_tg_op_mapping.py` | **69 passed**（rollback 32 + tg_op_mapping 37；G1 回归 + validate_mapping_consistency；EXPECTED_OP_COUNT=44 不变，45 归 STORY-039-03） |
| `uv run --python 3.11 pytest tests/test_env_unsupported.py tests/test_op_mapper_pppoe_client.py` | 26 passed（既有 9 类 env 占位符用例 + pppoe_client 不回归） |
| `uv run --python 3.11 python -m py_compile skills/policy-route-execution/scripts/op_mapper.py` | 通过 |
| `git diff` 逐块审计 | G4 仅 4 处改动；G1 区块（W1 已落码）零触碰；G5/sw3 族与 EXPECTED_OP_COUNT 零 diff；case_runner.py / SKILL.md / op-coverage-matrix.md 零触碰 |

## 未覆盖项

- 真机策略路由 A 场景（`${ENV.dut.next_hop_a}` 在真实 env-file + ptm-atomic 执行链路中的端到端行为）：CR-039 static-only，不授权真机（DQ-039-04）；由 CP7 以 static-only 口径验证，运行时风险由 RA 台账承接。
- env-file canonical 侧样例（含 `next_hop_a` 字段的 example yaml）归 STORY-039-04（W1 已完成，`skills/case-execution/templates/env-file.pppoe-sw3.next_hop_a.example.yaml`）。

## 设计缺口反馈

无。technical-note 全部契约逐条落实；无新增设计分叉或需回开 CP5 的项。

## 平台差异处理

N/A。改动对象为 canonical 源 `skills/policy-route-execution/scripts/op_mapper.py`（Skill 私有脚本），不涉及平台安装目录、发现路径或 PLATFORM-CONTRACTS 平台分支；`${ENV.dut.next_hop_a}` 在重装 ptm-atomic 后随 Skill 脚本自然生效。

## 后续交接（-> meta-qa / CP6）

- Return Packet：`process/returns/STORY-039-02.return.json`；Evidence Index 摘要见本文件。
- 验证入口：`uv run --python 3.11 pytest tests/test_op_mapper_next_hop_a.py`（主）；`tests/test_tg_op_mapping.py` / `tests/test_env_unsupported.py` / `tests/test_op_mapper_rollback.py`（回归）。
- 风险提示：占位符表注释「10 类」、报错文案「10 类」、实际解析分支三者已对齐（单测 5/6 守护该一致性）；STORY-039-03（W3）落地 G5 时 EXPECTED_OP_COUNT 44->45 为预期后续变更，勿误判为本 Story 回归。
- 风险提示：工作区（ptm-te-manaul）侧报错文案与 2 处计数注释仍为 9 类；如后续以工作区为对拍基线，须以本 Story 的 A3/A4 对齐口径为准（canonical 为真相源）。
