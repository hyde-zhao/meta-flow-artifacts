---
doc_type: LLD
story_id: "CR037-S2"
story_slug: "limit-parser"
cr_id: "CR-037"
tier: "A"
status: "draft"
created_at: "2026-08-05"
owner: "meta-dev"
source_hld: "process/changes/CR-037-HLD.md §4.2"
source_story: "process/stories/STORY-CR037-S2.md"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
scope: "skills/topo-planning/src/limit_parser.py 新建 + skills/topo-planning/docs/limit-syntax.md 新建"
---

# STORY-CR037-S2-LLD：limit_parser.py 详细设计

> **Tier**: A（约束语义复杂，NLP 边界 + 平台别名 + 语法规范交付物）  
> **Shared Fragments**: `skills/device-management/reference/device-reference.md`（平台别名映射，只读消费）  
> **Open Items**: 无  
> **修订记录**: v0.4（2026-08-05，meta-dev）—— CP5 综合审核 NEEDS_REWORK 修订：S2-A speed_class 别名表（10GE/万兆→TE、QTE/40GE→QTE、TTE/25GE→TTE）纳入语法规范 + 单测（§5.2/§6.5/§8.5/§10/§15）；S2-B TG 平台约束独立分支（node_type=TG 时映射 sub_type/hardware_platform，不套 DUT 型号表，§3/§5.1/§6.4/§8.5）；S2-C 字段名统一为 HLD 的 `link`（弃用 link_id，§5.1/§6.1/§7/§8.5）；S2-D nxp1043 等价类拆分 `nxp1043-4g`(→160) vs `nxp1043-8g`(→160-PRO/290/490/690)（§5.2/§6.2/§8.1）；S2-E 型号冲突消歧改为「等价类 + devices.yaml 实际设备」联合决策，非首中，conflicts 逐一给消歧策略（§6.3/§8.3）；S2-F 保持多节点 all 语义/空 limit 默认/端口兼容矩阵。
> **修订记录**: v0.5（2026-08-05，meta-dev）—— CP5 第二轮复审 R4 + Minor 修订：R4 `_disambiguate_model` 接入 `parse_limit` 主流程（新增 `pool_devices` 入参，冲突型号展开后逐一消歧，输出契约含消歧后 device_type 集合，§1/§2/§3/§5.1/§6.1/§6.3/§7/§8.3/§10/§11/§15）；Minor "海光"简写 also_known_as 首中杰伦海光修正——完整平台名优先精确匹配（"海光3250"→乐研海光3250），简写"海光"多命中时抛 `PLATFORM_AMBIGUOUS` 或按 pool_devices 上下文消歧，非首中（§5.2/§6.2/§8.1/§10）。

---

## 0. 工程依据

本模块是 CR-037 topo-planning skill 的第二数据层组件。工程依据源自 HLD §4.2（limit_parser 模块设计），CR-037 评审意见 G5（NLP 语法交付物为显式交付物），DQ-037-02（平台别名消费 manaul device-reference.md 完整映射），以及 HLD §4.2 H3（多节点作用域 all 语义）与 H4（约束叠加优先级）。顶层目标是将用户自然语言拓扑约束转化为结构化设备/链路约束对象，供 S4 topo_mapper 消费做回溯剪枝。

---

## 1. 目标

新建 `skills/topo-planning/src/limit_parser.py`，定义 topo-limit 结构化模型，消费 `device-reference.md` 完整映射表展开平台别名（DUT 硬件系列）与 TG 平台约束（sub_type/hardware_platform），归一 speed_class 别名（NLP「10GE/万兆」→ TE 等），校验端口兼容矩阵，并交付显式 `limit-syntax.md` 语法规范文档（含示例/反例集）。模块接受 Agent 转译后的结构化 limit（dict/JSON），不直接处理自由文本 NLP。

核心价值：将用户自然语言约束（"海光3250上执行，需要 TE|10GE 链路"）转化为可验证的设备/链路约束对象，供 S4 topo_mapper 消费。其中「10GE」经 speed_class 别名归一为枚举 `TE`（S2-A），确保核心场景「海光3250 + TE|10GE 链路」可被解析。

**R4 主流程接入（MAJOR，第二轮复审）**：`parse_limit` 是消歧唯一入口。冲突型号（conflicts 中的 device_type，如 A1500-HU）在设备约束展开后**必须**经 `_disambiguate_model` 消歧（新增 `pool_devices` 入参，来自 S3 归并后 devices.yaml 实际设备），输出契约（DeviceConstraint）含消歧后 device_type 集合（`resolved_equiv_class` 更新 `equiv_class_map[device_type]`）。此前 `_disambiguate_model` 已有设计但未被 `parse_limit` 调用，导致冲突型号实际上仍按首中或默认等价类匹配——本轮修正为强制接入。

**Minor 别名优先级（第二轮复审）**：`also_known_as` 简写"海光"同时属于 `杰伦海光`（JL-HG）与 `乐研海光3250`（HG3250），旧实现按 aliases 字典序遍历**首中**杰伦海光。修正：完整平台名优先精确匹配（"海光3250"→乐研海光3250，因"海光3250"仅存在于乐研海光3250 的 also_known_as）；简写"海光"多命中时抛 `PLATFORM_AMBIGUOUS`（提示候选）或按 pool_devices 实际设备交集消歧（非首中，§8.1）。

## 2. Requirements

### Functional

| ID | 需求 | 优先级 |
|----|------|:------:|
| FR1 | `parse_limit(limit_input: dict | str | None) → TopoLimit` 解析入口 | P0 |
| FR2 | 平台别名展开（DUT 硬件系列）：输入 `"海光3250"` → 输出 `[DAS-TGFW-A1300-HU, DAS-TGFW-A1500-HU, DAS-TGFW-A1580-HU, DAS-TGFW-A1600-HU]` | P0 |
| FR3 | 设备约束：`device_constraints[]` 含 `node_ref` / `node_type` / `platform` / `device_type` | P0 |
| FR4 | 链路约束 3 模式：`include-at-least-one`（至少一条满足）、`all`（全部满足）、`specific`（指定 link 满足）；字段名统一用 `link`（与 collection links 命名一致，S2-C） | P0 |
| FR5 | 端口兼容矩阵：`speed_class × media_type` 合法性校验；非法组合 → `PARAM_INVALID` | P0 |
| FR6 | 多节点作用域：`node_ref` 空 → all 语义（作用于全部该 `node_type` 节点） | P0 |
| FR7 | 空 limit 默认行为：`None`/`""`/`{}` → 返回空 `TopoLimit`（不阻断后续匹配） | P0 |
| FR8 | 型号冲突消歧（**接入主流程**）：`parse_limit` 在设备约束展开后，对 conflicts 中的 device_type 逐一调 `_disambiguate_model`（入参 `pool_devices`），「用户平台约束等价类 + devices.yaml 实际设备」联合决策，非首中（R4，S2-E）；输出契约含消歧后 device_type 集合 | P1 |
| FR9 | 交付 `limit-syntax.md`（含 ≥5 个示例 + ≥5 个反例） | P0 |
| FR10 | speed_class 别名归一：NLP「10GE / 万兆 / TE」→ 枚举 `TE`；「QTE / 40GE」→ `QTE`；「TTE / 25GE」→ `TTE`；归一后做兼容矩阵校验（S2-A） | P0 |
| FR11 | TG 平台约束分支：`node_type: TG` 时 platform 约束映射到 `sub_type`/`hardware_platform`（ixia-c/trex × EP/C236/J1900，消费 device-reference.md TG 6 组合矩阵），不套 DUT 型号映射表（S2-B） | P0 |
| FR12 | 平台别名优先级：完整平台名优先精确匹配（"海光3250"→乐研海光3250）；简写"海光"多命中时抛 `PLATFORM_AMBIGUOUS` 或按 pool_devices 实际设备交集消歧，非首中（Minor） | P1 |

### Non-Functional

| ID | 需求 |
|----|------|
| NFR1 | 平台映射表不硬编码为 Python 常量，从 YAML 数据文件加载（可维护性） |
| NFR2 | 校验错误返回 `TopoLimitError` 异常（含 `error_code` + `details`），不使用裸 ValueError |
| NFR3 | 模块不依赖 topo-planning 其他模块（S1/S3）的代码实现；`pool_devices` 为可选入参（来自 S3 归并输出，作为普通 List[Dict] 传入，不 import S3）。**确定性依赖关系（R4 修正）**：给定相同 `limit_input` + 相同 `pool_devices` 输入 → 输出确定性一致；`pool_devices=None` 时冲突型号按用户显式 platform 等价类消歧，否则抛 `AMBIGUOUS_MODEL`（不静默首中） |
| NFR4 | limit-syntax.md 为独立交付文档，不嵌入 Python docstring 作为唯一载体 |

## 3. 模块拆分与职责

### 3.1 模块内部函数划分

| 函数 | 职责 |
|------|------|
| `parse_limit(limit_input, platform_map_path=None, pool_devices=None)` | 主入口：dict/str/None → `TopoLimit`；`pool_devices` 为 S3 归并后的 devices.yaml 实际设备列表（可选），冲突型号消歧与简写别名消歧的上下文（R4） |
| `_parse_device_constraints(devices_raw, pool_devices)` | 解析设备约束列表 → `DeviceConstraint[]`（含 DUT 平台展开与 TG sub_type 分支；冲突型号逐一调 `_disambiguate_model`，R4） |
| `_parse_link_constraints(links_raw)` | 解析链路约束列表 → `LinkConstraint[]`（含 speed_class 别名归一） |
| `_expand_platform(alias, platform_map, pool_devices=None)` | 平台别名 → device_type 列表（消费 device-reference DUT 映射；处理 equiv_class_group 拆分；**完整平台名优先 + 简写多命中消歧，非首中**，Minor） |
| `_expand_tg_platform(alias)` | TG 平台约束 → sub_type/hardware_platform（消费 TG 6 组合矩阵，S2-B） |
| `_normalize_speed_class(raw)` | speed_class 别名归一 → 枚举值（10GE/万兆→TE、QTE/40GE→QTE、TTE/25GE→TTE，S2-A） |
| `_disambiguate_model(device_type, pool_devices, user_equiv_class)` | 冲突型号消歧（等价类 + devices.yaml 实际设备联合决策，非首中，S2-E；**parse_limit 主流程调用**，R4） |
| `_validate_port_compat(speed_class, media_type)` | 端口兼容矩阵校验 → bool / raise |
| `_resolve_node_scope(constraint, topo_nodes)` | 解析 node_ref 空 → all 语义：返回受影响的 node_id 列表 |

### 3.2 平台别名数据层

平台映射表独立于模块代码，以 YAML 数据文件形式维护：

```
skills/topo-planning/config/platform_alias.yaml   # DUT 硬件系列别名 + TG 平台组合 + speed_class 别名 + conflicts
```

`limit_parser.py` 启动时加载该文件为 `PLATFORM_ALIAS_MAP: Dict[str, Any]`（含 `aliases` / `tg_platforms` / `speed_class_aliases` / `conflicts` 四个区段）。DUT 平台别名（aliases）派生自 device-reference.md §硬件系列->device_type 快速索引；TG 平台组合（tg_platforms）派生自 device-reference.md §TG 测试仪型号对照表（6 组合）；speed_class 别名（speed_class_aliases）为 NLP 输入归一映射（S2-A）。

## 4. 代码结构与文件影响范围

### 新增文件

```
skills/topo-planning/src/limit_parser.py              # 核心解析模块（~400 行）
skills/topo-planning/config/platform_alias.yaml        # 平台别名映射数据
skills/topo-planning/docs/limit-syntax.md              # 语法规范文档
```

### 只读消费

```
skills/device-management/reference/device-reference.md  # 平台映射只读参考
```

### 不修改

无。全部为新增文件。

## 5. 数据模型与持久化设计

### 5.1 TopoLimit 数据类

```python
@dataclass
class TopoLimit:
    """结构化 topo-limit"""
    device_constraints: List[DeviceConstraint] = field(default_factory=list)
    link_constraints: List[LinkConstraint] = field(default_factory=list)
    raw_input: Optional[str] = None        # 原始输入保留（调试审计）
    is_empty: bool = True                  # 空 limit 标记
    warnings: List[str] = field(default_factory=list)  # 非阻断警告

@dataclass
class DeviceConstraint:
    """设备类型约束"""
    node_ref: Optional[str] = None         # 目标逻辑节点 ID（None=all）
    node_type: Optional[str] = None        # DUT|TG|SW|PC|Mock
    platform: Optional[str] = None         # 硬件系列名（展开后归一化为规范系列名，如 "乐研海光3250"，P2-3）
    device_types: List[str] = field(default_factory=list)  # 展开后的 TGFW 型号列表
    equiv_class: Optional[str] = None      # hardware_platform 等价类（如 HG3250，P1-3）；family 有拆分时为默认子组等价类
    equiv_class_map: Dict[str, str] = field(default_factory=dict)  # device_type → equiv_class（S2-D，如 nxp1043 家族拆分）
    resolved_equiv_class: Optional[str] = None  # 冲突型号消歧后等价类（R4：parse_limit 经 _disambiguate_model 写入；无冲突=equiv_class，有冲突=消歧结果）
    disambiguation: Optional[str] = None   # 消歧依据描述（R4：如 "user_equiv_class" / "pool_devices:HG3250" / "no-conflict"），供 S4/审计追踪
    scope: str = "all"                     # "all" | "any"（多节点语义）
    # TG 分支（S2-B）：node_type=TG 时 platform 约束映射到 sub_type / hardware_platform，不套 DUT 型号表
    sub_type: Optional[str] = None         # TG: ixia-c | trex
    hardware_platform: Optional[str] = None  # TG: EP | C236 | J1900

@dataclass
class LinkConstraint:
    """链路约束"""
    link: Optional[str] = None             # 指定链路 ID（specific 模式）；字段名统一为 `link`，与 collection links 命名一致（S2-C）
    node_endpoints: List[str] = field(default_factory=list)  # 涉及的节点对
    speed_class: Optional[str] = None      # GE|TE|XTE|QTE|TTE（经 _normalize_speed_class 归一后的枚举值）
    media_type: Optional[str] = None       # copper|fiber
    mode: str = "include-at-least-one"     # include-at-least-one | all | specific
```

### 5.2 平台别名映射文件（platform_alias.yaml）

从 `device-reference.md` 提取，以 YAML 形式维护可维护的映射关系：

```yaml
# 平台别名 → device_type 映射（派生自 device-reference.md §硬件系列->device_type 快速索引）
# 主键 = 规范硬件系列名（device-reference.md 第1列）；also_known_as = 用户常用别名（P2-3）
aliases:
  nxp1043-4g:
    device_types: [DAS-TGFW-160]
    default: DAS-TGFW-160
    equiv_class: nxp1043-4g-无盘
    also_known_as: [nxp1043-4g-无盘]
  nxp1043-8g:
    device_types: [DAS-TGFW-160-PRO, DAS-TGFW-290, DAS-TGFW-490, DAS-TGFW-690]
    default: DAS-TGFW-160-PRO
    equiv_class: nxp1043-8g
    also_known_as: [nxp1043-8g]
  nxp1043:  # 家族入口（整族选择，S2-D）
    device_types: [DAS-TGFW-160, DAS-TGFW-160-PRO, DAS-TGFW-290, DAS-TGFW-490, DAS-TGFW-690]
    default: DAS-TGFW-160-PRO
    equiv_class: nxp1043-8g               # 默认匹配等价类（=160-PRO，有 msata）
    equiv_class_group:                     # S2-D：精确 device_type → equiv_class 反查
      nxp1043-4g-无盘: [DAS-TGFW-160]
      nxp1043-8g: [DAS-TGFW-160-PRO, DAS-TGFW-290, DAS-TGFW-490, DAS-TGFW-690]
  nxp1046:
    device_types: [DAS-TGFW-890, DAS-TGFW-1900, DAS-TGFW-1950, DAS-TGFW-2900]
    default: DAS-TGFW-1900
    equiv_class: nxp1046-16g
    also_known_as: [nxp1046-16g]
  C3758R:
    device_types: [DAS-TGFW-2900, DAS-TGFW-2950, DAS-TGFW-3900, DAS-TGFW-3950, DAS-TGFW-4900]
    default: DAS-TGFW-2900
    equiv_class: C3758
    also_known_as: [C3758]
  C236:
    device_types: [DAS-TGFW-5900, DAS-TGFW-6900]
    default: DAS-TGFW-5900
    equiv_class: C236
  EP:
    device_types: [DAS-TGFW-8900, DAS-TGFW-10900, DAS-TGFW-12900]
    default: DAS-TGFW-8900
    equiv_class: EP
  华电飞腾:
    device_types: [DAS-TGFW-A1200-FU, DAS-TGFW-A1280-FU]
    default: DAS-TGFW-A1200-FU
    equiv_class: HD-D2000
    also_known_as: [HD-D2000, 飞腾]
  杰伦海光:
    device_types: [DAS-TGFW-A1500-HU]
    default: DAS-TGFW-A1500-HU
    equiv_class: JL-HG
    also_known_as: [JL-HG, 海光]
  乐研E2000Q:
    device_types: [DAS-TGFW-A100-FU, DAS-TGFW-A200-FU, DAS-TGFW-A400-FU, DAS-TGFW-A600-FU, DAS-TGFW-A800-FU]
    default: DAS-TGFW-A200-FU
    equiv_class: E2000Q
    also_known_as: [E2000Q]
  乐研D2000:
    device_types: [DAS-TGFW-A1200-FU, DAS-TGFW-A1280-FU]
    default: DAS-TGFW-A1200-FU
    equiv_class: D2000
    also_known_as: [D2000]
  乐研海光3250:
    device_types: [DAS-TGFW-A1300-HU, DAS-TGFW-A1500-HU, DAS-TGFW-A1580-HU, DAS-TGFW-A1600-HU]
    default: DAS-TGFW-A1300-HU
    equiv_class: HG3250
    also_known_as: [HG3250, hg3250, 海光3250, 海光]
  乐研海光5380:
    device_types: [DAS-TGFW-A1800-HU, DAS-TGFW-A2200-HU]
    default: DAS-TGFW-A1800-HU
    equiv_class: HG5380
    also_known_as: [HG5380, hg5380, 海光5380]
  天池云:
    device_types: [DAS-TGFWV4000, DAS-TGFW-S100-GZ, DAS-TGFW-S200-GZ, DAS-TGFW-S400-GZ, DAS-TGFW-S600-GZ, DAS-TGFW-S800-GZ, DAS-TGFW-S1000-GZ]
    default: DAS-TGFW-S400-GZ
    equiv_class: 天池云
  华为鲲鹏:
    device_types: [DAS-TGFW-S100-KU-GZ, DAS-TGFW-S200-KU-GZ, DAS-TGFW-S400-KU-GZ, DAS-TGFW-S600-KU-GZ, DAS-TGFW-S800-KU-GZ, DAS-TGFW-S1000-KU-GZ]
    default: DAS-TGFW-S400-KU-GZ
    equiv_class: 华为鲲鹏
# 13 硬件系列完整映射（P1-2：nxp1043/nxp1046/C3758R/C236/EP/华电飞腾/杰伦海光/乐研E2000Q/乐研D2000/乐研海光3250/乐研海光5380/天池云/华为鲲鹏）
# equiv_class = device-reference.md 第2列「硬件等价类」，作为 hardware_platform 统一取值（P1-3）
# 别名歧义（Minor）：also_known_as 简写 "海光" 同时属于 杰伦海光(JL-HG) 与 乐研海光3250(HG3250)。
#   匹配规则 = 完整平台名优先精确匹配（"海光3250" 仅命中 乐研海光3250）；简写 "海光" 多命中时
#   抛 PLATFORM_AMBIGUOUS（提示候选）或按 pool_devices 实际设备 hardware_platform 交集消歧，非首中（§8.1）。
# nxp1043 按内存/盘位拆分为 nxp1043-4g-无盘 与 nxp1043-8g 两个等价类（S2-D：DAS-TGFW-160 实为 4g-无盘，
# 与 160-PRO/290/490/690 的 8g 分开，避免整族归 8g 错配）

# TG 平台组合（S2-B）：派生自 device-reference.md §TG 测试仪型号对照表（2 sub_type × 3 hardware_platform = 6 组合，SM-EX-01）
tg_platforms:
  sub_types: [ixia-c, trex]
  hardware_platforms: [EP, C236, J1900]
  combos:
    - {sub_type: ixia-c, hardware_platform: EP}
    - {sub_type: ixia-c, hardware_platform: C236}
    - {sub_type: ixia-c, hardware_platform: J1900}
    - {sub_type: trex, hardware_platform: EP}
    - {sub_type: trex, hardware_platform: C236}
    - {sub_type: trex, hardware_platform: J1900}

# speed_class 别名归一（S2-A）：NLP 输入（含中文）→ 枚举值；枚举值集合 = GE/TE/XTE/QTE/TTE/FE
speed_class_aliases:
  GE:  [ge, 千兆, 1ge, 1g]
  TE:  [te, 10ge, 万兆, 10g]
  XTE: [xte]
  QTE: [qte, 40ge, 40g]
  TTE: [tte, 25ge, 25g]
  FE:  [fe, 百兆, 100m]

# 冲突型号（同一 device_type 对应多个硬件系列）→ 映射到 hardware_platform 等价类（P1-3）
# 消歧策略 = 「用户平台约束等价类 + devices.yaml 实际设备 hardware_platform」联合决策，非首中（S2-E）
conflicts:
  DAS-TGFW-2900:
    candidates: [nxp1046-16g, C3758]
    strategy: "按 CPU 平台/OS 区分：nxp1046-16g=ARM/16G（老 NXP1046-2U），C3758=X86/das-os 用 HG 包；devices.yaml hardware_platform 实际值仲裁"
  DAS-TGFW-A1200-FU:
    candidates: [HD-D2000, D2000]
    strategy: "按硬件系列区分：华电飞腾 HD-D2000（老飞腾/千兆/无硬盘）vs 乐研D2000 D2000（飞腾 D2000）；devices.yaml 实际值仲裁"
  DAS-TGFW-A1280-FU:
    candidates: [HD-D2000, D2000]
    strategy: "同 A1200-FU：华电飞腾 HD-D2000 vs 乐研D2000 D2000；devices.yaml 实际值仲裁"
  DAS-TGFW-A1500-HU:
    candidates: [JL-HG, HG3250]
    strategy: "按海光平台区分：杰伦海光 JL-HG（老海光/万兆）vs 乐研海光3250 HG3250（海光 3250-40G）；devices.yaml 实际值仲裁；用户显式 platform 时平台约束等价类优先"
```

### 5.3 端口兼容矩阵

```python
# 内联常量 — 不频繁变化的物理约束
PORT_COMPAT_MATRIX = {
    # speed_class → 允许的 media_type 集合
    'GE':   {'copper', 'fiber'},
    'TE':   {'fiber'},
    'XTE':  {'fiber'},
    'QTE':  {'fiber'},
    'TTE':  {'fiber'},
    'FE':   {'copper', 'fiber'},
}
```

> **校验入口**（S2-A）：`_validate_port_compat` 收到的是**经 `_normalize_speed_class` 归一后的枚举值**（如用户写 `10GE` → 先归一为 `TE`，再查矩阵 `TE→{fiber}`）。若原始输入不在 speed_class 别名表，抛 `SPEED_CLASS_UNKNOWN`。

**无持久化**：所有数据模型均为内存对象。`platform_alias.yaml` 是只读数据文件（与代码同 tree），不运行时写入。

## 6. API / Interface 设计

### 6.1 `parse_limit(limit_input: Union[dict, str, None], platform_map_path: Optional[str] = None, pool_devices: Optional[List[Dict]] = None) -> TopoLimit`

**主入口**。接受 Agent 转译后的结构化 limit。`pool_devices` 为 S3 归并后的 devices.yaml 实际设备列表（`List[Dict]`，含 `device_type` / `hardware_platform` 字段），可选；**传入时**冲突型号（conflicts 中的 device_type）与简写平台别名（如 "海光"）可经实际设备消歧；**缺省时**冲突型号按用户显式 platform 等价类消歧，否则抛 `AMBIGUOUS_MODEL`（R4/Minor）。

**输入格式**（dict）：
```python
{
    "device_constraints": [
        {"node_type": "DUT", "platform": "海光3250"},
        {"node_ref": "tg1", "node_type": "TG", "sub_type": "trex"},            # TG 分支（S2-B）
        {"node_ref": "tg2", "node_type": "TG", "hardware_platform": "C236"}    # TG 分支（S2-B）
    ],
    "link_constraints": [
        {"mode": "include-at-least-one", "speed_class": "10GE", "media_type": "fiber"},  # speed_class 别名归一 → TE（S2-A）
        {"mode": "all", "speed_class": "GE", "media_type": "copper"},
        {"mode": "specific", "link": "link3", "speed_class": "TTE"}            # 字段名 link（S2-C）
    ]
}
```

**输入格式**（str — JSON 串）：
```python
parse_limit('{"device_constraints":[{"node_type":"DUT","platform":"海光3250"}]}')
```

**输入格式**（None/""/{} — 空 limit）：
```python
parse_limit(None)  # → TopoLimit(is_empty=True)
```

**返回**：`TopoLimit` 对象（展开后的 `device_types` + 校验通过的 `link_constraints`）

**异常**：
- `TopoLimitError(code="PARAM_INVALID", details=...)` — 端口兼容矩阵不合法
- `TopoLimitError(code="PLATFORM_UNKNOWN", details=...)` — DUT 平台别名不在映射表中
- `TopoLimitError(code="PLATFORM_AMBIGUOUS", details=...)` — 简写平台别名多命中（如 "海光" → 杰伦海光 JL-HG / 乐研海光3250 HG3250），需完整平台名或 pool_devices 上下文消歧（Minor）
- `TopoLimitError(code="SPEED_CLASS_UNKNOWN", details=...)` — speed_class 不在别名表/枚举集合（S2-A）
- `TopoLimitError(code="TG_PLATFORM_UNKNOWN", details=...)` — TG sub_type/hardware_platform 不在 6 组合矩阵（S2-B）
- `TopoLimitError(code="AMBIGUOUS_MODEL", details=...)` — 冲突型号无法自动消歧（无 pool_devices 且无用户显式 platform，或池中多平台并存）（R4/S2-E）
- `TopoLimitError(code="PARSE_FAILED", details=...)` — 输入格式无法解析

### 6.2 `_expand_platform(alias: str, platform_map: Dict, pool_devices: Optional[List[Dict]] = None) -> Tuple[List[str], str, Optional[str], Dict[str, str]]`

**参数**：`alias` 为 `"海光3250"` / `"HG3250"` / `"hg3250"` / `"乐研海光3250"` / `"海光"` 等（P2-3：主键为规范系列名，`also_known_as` 收别名；**简写"海光"为歧义别名**，Minor）；`pool_devices` 可选（S3 归并后的 devices.yaml 实际设备），用于简写别名消歧。

**查找优先级（Minor 修正：完整平台名优先 + 多命中非首中）**：
1. 精确匹配 `aliases[alias]`（主键 = 规范系列名，含 `nxp1043-4g` / `nxp1043-8g` 拆分键，S2-D）——**主键精确匹配始终最高优先**，如 `"乐研海光3250"` 直接命中。
2. **完整平台名优先精确匹配**：遍历全部 `also_known_as`，收集**精确相等**（`aka.lower() == alias_lower`）的命中条目集合：
   - **恰好 1 个命中** → 返回该条目。
   - **多个命中**（歧义简写，如 `"海光"` 同时命中 `杰伦海光` 与 `乐研海光3250`）→ 进入第 3 步歧义消歧（**非首中**）。
   - 0 个命中 → 进入第 4 步子串匹配。
   > 旧实现（v0.4 前）在 `also_known_as` 精确命中多个时按 aliases 字典序**首中**（"海光"→杰伦海光 JL-HG），本轮修正为收集全部命中再消歧，避免误判。
3. **歧义消歧（仅当第 2 步多命中）**：
   - 若有 `pool_devices` → 用候选条目的 `device_types` 与 pool_devices 实际 `device_type`/`hardware_platform` 交集仲裁：恰好 1 个候选有交集 → 返回该条目；0 或 ≥2 → 抛 `PLATFORM_AMBIGUOUS`（含候选列表）。
   - 无 `pool_devices` → 抛 `TopoLimitError(code="PLATFORM_AMBIGUOUS", details=f"platform alias '{alias}' 命中多个: {候选系列名}。请使用完整平台名（如 乐研海光3250 / 杰伦海光）或传入 pool_devices 消歧")`。
4. 子串匹配（`alias.lower()` 包含匹配，仅当候选唯一时可用，否则抛 `PLATFORM_AMBIGUOUS`）。

**返回**：`(device_types, default_device_type, equiv_class, equiv_class_map)` — 型号列表 + 默认型号 + 默认等价类 + `{device_type: equiv_class}` 反查映射。

- `equiv_class`：条目 `equiv_class` 字段；当条目含 `equiv_class_group`（如 `nxp1043` 家族，S2-D）时为**默认子组等价类**（`nxp1043-8g`，对应默认型号 160-PRO）。
- `equiv_class_map`：`{device_type: equiv_class}`。无 `equiv_class_group` 的条目，等价类对该家族所有 device_type 相同，`equiv_class_map` 为单值展开。S4 匹配按具体 device_type 反查对应等价类，避免 `nxp1043` 整族误归 8g 错配 DAS-TGFW-160（4g-无盘）。

### 6.3 `_disambiguate_model(device_type: str, pool_devices: List[Dict], user_equiv_class: Optional[str] = None) -> Optional[str]`

当 `device_type` 出现在 `conflicts` 中时调用。消歧为**「用户平台约束等价类 + devices.yaml 实际设备 hardware_platform」联合决策**（S2-E），**非首中**。**主流程接入（R4）**：`parse_limit` → `_parse_device_constraints` 对每个展开的 device_type 检查是否在 `conflicts`；在 → 必须调用本函数，结果写回 `DeviceConstraint.resolved_equiv_class` + `disambiguation`，并更新 `equiv_class_map[device_type]`。**不得跳过消歧直接按默认等价类匹配**。

**决策顺序**（任一分支命中即返回）：

1. **用户意图优先**：若调用方已通过 `user_equiv_class`（来自用户显式 `platform` 别名展开的等价类）传入，且该值在 `conflicts[device_type].candidates` 中 → 返回该等价类（用户平台约束优先；devices.yaml 仅作一致性 warning）。
2. **devices.yaml 实际设备仲裁**：收集 devices.yaml 中 `device_type` 相同设备的 `hardware_platform`（统一等价类取值）集合，与 `candidates` 求交：
   - 恰好 1 个不同等价类 → 返回该等价类。
   - 0 个 → `TopoLimitError(code="AMBIGUOUS_MODEL", details="device_type 'X' 无候选等价类命中。候选: [..]，请显式指定 platform")`。
   - 多个不同等价类 → `TopoLimitError(code="AMBIGUOUS_MODEL", details="device_type 'X' 在池中命中多个平台: {..}，请显式指定 platform")`。
3. 不做「遍历取首个命中」的首中逻辑（S2-E 关键修正）。

**语义（P1-3）**：`conflicts[device_type].candidates` 与 pool/devices 的 `hardware_platform` 字段统一使用 **hardware_platform 等价类**（device-reference.md 第2列：HG3250/JL-HG/HD-D2000/D2000/nxp1046-16g/C3758 等），不混用系列名（如 乐研海光3250/杰伦海光）。

**示例**：`device_type="DAS-TGFW-A1500-HU"`，用户未显式指定 platform；pool_devices 中 `hg3250-51` 的 `hardware_platform=HG3250`（等价类）→ 交集 {HG3250} 唯一 → 返回 `HG3250`（对应 乐研海光3250，非 杰伦海光）。若 `hg3250-51`(HG3250) 与 `某杰伦海光设备`(JL-HG) 同时存在 → `AMBIGUOUS_MODEL`，要求显式 platform。

**逐冲突消歧策略**（S2-E，完整表见 §8.3）：

| device_type | 候选等价类 | 消歧策略 |
|---|---|---|
| `DAS-TGFW-2900` | [nxp1046-16g, C3758] | 按 CPU 平台/OS：ARM/16G（老 NXP1046-2U）vs X86/das-os（HG 包） |
| `DAS-TGFW-A1200-FU` | [HD-D2000, D2000] | 按硬件系列：华电飞腾（老飞腾/千兆/无硬盘）vs 乐研D2000（飞腾 D2000） |
| `DAS-TGFW-A1280-FU` | [HD-D2000, D2000] | 同 A1200-FU |
| `DAS-TGFW-A1500-HU` | [JL-HG, HG3250] | 按海光平台：杰伦海光（老海光/万兆）vs 乐研海光3250（3250-40G） |

### 6.4 `_expand_tg_platform(alias: str, tg_map: Dict) -> Dict[str, Optional[str]]`

**TG 平台约束分支（S2-B）**。当 `node_type: TG` 时，`platform` 字段不再走 DUT 硬件系列映射表（`_expand_platform`），而是映射到 TG 的 `sub_type` / `hardware_platform`（消费 device-reference.md §TG 测试仪型号对照表，2 sub_type × 3 hardware_platform = 6 组合）。

**参数**：`alias` 为用户对 TG 平台的描述；`tg_map` 为 `platform_alias.yaml` 的 `tg_platforms` 区段。

**匹配规则**（按优先级）：
1. `alias` 命中 `sub_types`（`ixia-c` / `trex`）→ 返回 `{"sub_type": alias, "hardware_platform": None}`（该 sub_type 全平台适用）。
2. `alias` 命中 `hardware_platforms`（`EP` / `C236` / `J1900`）→ 返回 `{"sub_type": None, "hardware_platform": alias}`（该平台全 sub_type 适用）。
3. `alias` 为组合形式 `"sub_type:hardware_platform"`（如 `"trex:C236"`）→ 校验在 `combos` 中存在，返回拆分后的 `{"sub_type", "hardware_platform"}`。
4. 均未命中 → `TopoLimitError(code="TG_PLATFORM_UNKNOWN", details=..., 合法 sub_type=ixia-c|trex, hardware_platform=EP|C236|J1900, 6 组合=...)`。

**输出落点**：DeviceConstraint 的 `sub_type` / `hardware_platform` 字段（§5.1），`device_types` 置空、`platform` 保留原始别名。S3/S4 归并层按 TG 组合与池 TG 节点匹配。

### 6.5 `_normalize_speed_class(raw: str, platform_map: Dict) -> str`

**speed_class 别名归一（S2-A）**。将 NLP 输入归一为枚举值，供 `_validate_port_compat` 使用。

**参数**：`raw` 为 speed_class 原始输入（如 `"10GE"` / `"万兆"` / `"TE"` / `"40GE"` / `"25GE"`）。

**算法**：`platform_map["speed_class_aliases"]` 遍历，对每个枚举 key 的别名列表做不区分大小写精确匹配；命中返回枚举 key。

**别名表**（§5.2 `speed_class_aliases`，核心场景 S2-A）：

| 枚举值 | NLP 别名（含中文） |
|---|---|
| `GE` | ge / 千兆 / 1ge / 1g |
| `TE` | te / 10ge / 万兆 / 10g |
| `XTE` | xte |
| `QTE` | qte / 40ge / 40g |
| `TTE` | tte / 25ge / 25g |
| `FE` | fe / 百兆 / 100m |

**失败**：均未命中 → `TopoLimitError(code="SPEED_CLASS_UNKNOWN", details=..., 合法枚举=GE|TE|XTE|QTE|TTE|FE)`。

> 核心场景「需要包含 TE|10GE 链路」：`10GE` 归一为 `TE`，再经兼容矩阵 `TE→{fiber}` 校验（S2-A 关键修正，否则 `10GE` 不在枚举直接抛错）。

## 7. 核心处理流程

```
parse_limit(limit_input, platform_map_path=None, pool_devices=None)
 │
 ├─ 1. normalize_input ─── 统一为 dict（None → {}，str → json.loads → dict）
 │
 ├─ 2. 空 limit 检查 ─── 若 dict 为空 {} → 返回 TopoLimit(is_empty=True)
 │
 ├─ 3. _parse_device_constraints(devices_raw, pool_devices)
 │   │  for each constraint:
 │   │    ├─ 提取 node_ref（None → scope="all"）
 │   │    ├─ 提取 node_type（DUT|TG|SW|PC|Mock → 校验合法值）
 │   │    ├─ node_type == TG ?
 │   │    │   └─ YES → _expand_tg_platform(platform|sub_type|hardware_platform)
 │   │    │       ├─ 查 tg_platforms.sub_types / hardware_platforms / combos
 │   │    │       └─ 返回 {sub_type, hardware_platform} → DeviceConstraint(S2-B)
 │   │    │   └─ NO  → _expand_platform(platform, platform_map, pool_devices)
 │   │    │       ├─ 查 platform_alias.yaml aliases（含 nxp1043-4g/8g 拆分）
 │   │    │       ├─ 完整平台名优先精确匹配（"海光3250"→乐研海光3250，Minor）
 │   │    │       ├─ 简写多命中（如 "海光"）→ PLATFORM_AMBIGUOUS 或 pool_devices 交集消歧（Minor）
 │   │    │       ├─ 展开 also_known_as
 │   │    │       └─ 返回 device_types[] + 默认 device_type + equiv_class + equiv_class_map
 │   │    ├─ 冲突型号消歧（R4，主流程强制）：
 │   │    │   for dt in device_types:
 │   │    │     if dt in conflicts:
 │   │    │       resolved = _disambiguate_model(dt, pool_devices, user_equiv_class)
 │   │    │       # resolved=None（无冲突）→ 跳过
 │   │    │       # resolved=<等价类> → equiv_class_map[dt]=resolved; resolved_equiv_class=resolved
 │   │    │       # 抛 AMBIGUOUS_MODEL（无 pool_devices 且无用户 platform，或多平台并存）→ 终止
 │   │    └─ 收集 warnings（如 platform 为空但 node_type 已指定）
 │   └─ 返回 DeviceConstraint[] 列表
 │
 ├─ 4. _parse_link_constraints(links_raw)
 │   │  for each constraint:
 │   │    ├─ 校验 mode ∈ {include-at-least-one, all, specific}
 │   │    ├─ specific 模式 → 必须含 link 字段（S2-C，字段名统一为 link）
 │   │    ├─ speed_class 别名归一：_normalize_speed_class(raw) → 枚举值（S2-A）
 │   │    ├─ 校验端口兼容矩阵
 │   │    │   if speed_class and media_type:
 │   │    │       _validate_port_compat(normalized_speed_class, media_type)
 │   │    │       → GE=copper|fiber ✓ / TE=fiber ✓ / TE=copper ✗ → PARAM_INVALID
 │   │    └─ node_endpoints 可选（指定链路涉及的节点）
 │   └─ 返回 LinkConstraint[] 列表
 │
 └─ 5. 返回 TopoLimit(device_constraints=..., link_constraints=..., warnings=...)
```

### 约束叠加语义（H4）

在 TopoLimit 返回后，由 S4 topo_mapper 消费时按以下优先级叠加：

```
用户 topo-limit 约束 > 逻辑 topo（collection）自带链路约束 > 默认（无约束）
```

本模块只做解析/校验/展开，不做叠加。叠加逻辑在 S4 中实现。

### 多节点 all 语义

`device_constraints` 中 `node_ref` 为空且 `node_type` 指定时：
- `scope="all"` → 逻辑 topo 中所有 `node_type` 匹配的节点均受约束
- 示例：limit `"海光3250"`，逻辑 topo 有 2 个 DUT（dut1/dut2）→ 两个 DUT 均需为海光3250 型号

由消费方（S4 topo_mapper）在执行匹配时调用 `_resolve_node_scope(constraint, topo_nodes)` → 返回受影响的 `node_id` 列表。

## 8. 技术细节

### 8.1 平台别名展开算法

```python
def _expand_platform(alias: str, platform_map: Dict,
                     pool_devices: Optional[List[Dict]] = None) -> Tuple[List[str], str, Optional[str], Dict[str, str]]:
    """返回 (device_types, default_device_type, equiv_class, equiv_class_map)
    equiv_class_map = {device_type: equiv_class}（S2-D：nxp1043 家族拆分时逐型号反查等价类）
    Minor 修正：完整平台名优先精确匹配；简写多命中非首中（PLATFORM_AMBIGUOUS 或 pool_devices 交集消歧）"""
    aliases = platform_map.get('aliases', {})

    def _resolve(entry):
        device_types = entry['device_types']
        default = entry.get('default', device_types[0])
        equiv_class = entry.get('equiv_class')   # 默认等价类（family 拆分时 = 默认子组等价类，如 nxp1043-8g）
        equiv_class_map = {}
        group = entry.get('equiv_class_group')   # S2-D
        if group:
            # 家族拆分：逐 device_type 反查等价类（nxp1043 → 160=4g-无盘，160-PRO/290/490/690=8g）
            for ec, types in group.items():
                for dt in types:
                    equiv_class_map[dt] = ec
        else:
            # 单等价类家族：等价类对该家族所有 device_type 相同
            equiv_class_map = {dt: equiv_class for dt in device_types}
        return (device_types, default, equiv_class, equiv_class_map)

    # 1. 主键精确匹配（规范系列名，含 nxp1043-4g / nxp1043-8g 拆分键）—— 最高优先
    if alias in aliases:
        return _resolve(aliases[alias])

    # 2. also_known_as 完整精确匹配（Minor：收集全部命中，非首中）
    alias_lower = alias.lower()
    aka_matches = []  # [(platform_key, entry)]
    for platform_key, entry in aliases.items():
        for aka in entry.get('also_known_as', []):
            if aka.lower() == alias_lower:
                aka_matches.append((platform_key, entry))
                break

    if len(aka_matches) == 1:
        return _resolve(aka_matches[0][1])
    if len(aka_matches) > 1:
        # 歧义简写（如 "海光" → 杰伦海光 JL-HG / 乐研海光3250 HG3250）
        candidate_keys = [k for k, _ in aka_matches]
        if pool_devices:
            # 按 pool_devices 实际 device_type（型号）与 hardware_platform（等价类）交集消歧
            actual_dts = {d.get('device_type') for d in pool_devices if d.get('device_type')}
            actual_hps = {d.get('hardware_platform') for d in pool_devices if d.get('hardware_platform')}
            hit = []
            for key, entry in aka_matches:
                dt_set = set(entry['device_types'])
                ec = entry.get('equiv_class')
                if (dt_set & actual_dts) or (ec and ec in actual_hps):
                    hit.append(key)
            if len(hit) == 1:
                return _resolve(aliases[hit[0]])
            raise TopoLimitError(
                code="PLATFORM_AMBIGUOUS",
                details=f"platform alias '{alias}' 命中多个且 pool_devices 无法唯一消歧: {candidate_keys}。"
                        f"请使用完整平台名（如 乐研海光3250 / 杰伦海光）"
            )
        raise TopoLimitError(
            code="PLATFORM_AMBIGUOUS",
            details=f"platform alias '{alias}' 命中多个: {candidate_keys}。"
                    f"请使用完整平台名（如 乐研海光3250 / 杰伦海光）或传入 pool_devices 消歧"
        )

    # 3. 子串匹配（需唯一）
    candidates = []
    for platform_key, entry in aliases.items():
        if alias_lower in platform_key.lower():
            candidates.append(platform_key)
        else:
            for aka in entry.get('also_known_as', []):
                if alias_lower in aka.lower():
                    candidates.append(platform_key)
                    break

    if len(candidates) == 1:
        return _resolve(aliases[candidates[0]])
    elif len(candidates) > 1:
        raise TopoLimitError(
            code="PLATFORM_AMBIGUOUS",
            details=f"platform alias '{alias}' matches multiple: {candidates}"
        )

    raise TopoLimitError(
        code="PLATFORM_UNKNOWN",
        details=f"platform alias '{alias}' not found. Known: {list(aliases.keys())}"
    )
```

> **Minor 关键修正**：`"海光3250"` 仅在 `乐研海光3250` 的 `also_known_as` 中存在（`aka.lower() == "海光3250"` 唯一命中）→ 返回乐研海光3250（HG3250），正确；`"海光"` 同时命中 `杰伦海光`（[JL-HG, **海光**]）与 `乐研海光3250`（[..., **海光**]）→ 多命中，抛 `PLATFORM_AMBIGUOUS` 或按 pool_devices 交集消歧，**不再首中杰伦海光**。

### 8.1b 冲突型号消歧主流程调用（R4）

`_parse_device_constraints` 在 `_expand_platform` 展开后，对返回的 `device_types` 逐一遍历：若 `device_type in platform_map['conflicts']`，调用 `_disambiguate_model(device_type, pool_devices, user_equiv_class)`：

```python
def _parse_device_constraints(devices_raw, pool_devices=None, platform_map=None):
    constraints = []
    for raw in devices_raw:
        dc = DeviceConstraint(
            node_ref=raw.get('node_ref'),
            node_type=raw.get('node_type'),
            platform=raw.get('platform'),
        )
        if dc.node_type == 'TG':
            tg = _expand_tg_platform(raw.get('platform') or raw.get('sub_type') or raw.get('hardware_platform'), ...)
            dc.sub_type, dc.hardware_platform = tg['sub_type'], tg['hardware_platform']
            dc.device_types = []
        else:
            dts, default_dt, equiv_class, equiv_class_map = _expand_platform(
                raw.get('platform'), platform_map, pool_devices)
            dc.device_types = list(dts)
            dc.default_device_type = default_dt
            dc.equiv_class = equiv_class
            dc.equiv_class_map = dict(equiv_class_map)
            # R4：冲突型号逐一消歧（非首中）
            # 用户显式 platform 的等价类：直接复用 _expand_platform 返回的 equiv_class（含 family 拆分默认子组）
            user_equiv_class = equiv_class if raw.get('platform') else None
            for dt in list(dc.device_types):
                if dt in platform_map.get('conflicts', {}):
                    resolved = _disambiguate_model(dt, pool_devices or [], user_equiv_class, platform_map)
                    if resolved:  # 消歧成功 → 更新等价类映射 + resolved_equiv_class
                        dc.equiv_class_map[dt] = resolved
                        dc.resolved_equiv_class = resolved
                        dc.disambiguation = f"pool_devices:{resolved}" if pool_devices else f"user_equiv_class:{resolved}"
            # 所有冲突型号收敛到同一等价类时，dc.equiv_class 同步为消歧结果
            if dc.resolved_equiv_class:
                dc.equiv_class = dc.resolved_equiv_class
        constraints.append(dc)
    return constraints
```

`resolved_equiv_class` 写入 `DeviceConstraint`（§5.1），构成**消歧后 device_type 集合**的输出契约：S4 消费 `device_types` + `equiv_class_map[dt]=resolved` 做 hardware_platform 精确匹配，不再按默认等价类首中。

### 8.2 端口兼容矩阵校验

```python
def _validate_port_compat(speed_class: str, media_type: str) -> None:
    # speed_class 应为 _normalize_speed_class 归一后的枚举值（S2-A）
    allowed = PORT_COMPAT_MATRIX.get(speed_class.upper())
    if allowed is None:
        raise TopoLimitError(
            code="SPEED_CLASS_UNKNOWN",
            details=f"Unknown speed_class '{speed_class}'. Known: {list(PORT_COMPAT_MATRIX.keys())}"
        )
    if media_type.lower() not in allowed:
        raise TopoLimitError(
            code="PARAM_INVALID",
            details=f"speed_class '{speed_class}' does not support media_type '{media_type}'. "
                    f"Allowed: {allowed}"
        )
```

> **归一前置（S2-A）**：`_parse_link_constraints` 先调用 `_normalize_speed_class(raw)` 得到枚举值（`10GE`→`TE`），再传入本函数。因此本函数只接收枚举值，不接收 NLP 原始别名。

### 8.3 型号冲突消歧详细流程

```python
def _disambiguate_model(device_type: str, pool_devices: List[Dict], platform_map: Dict,
                        user_equiv_class: Optional[str] = None) -> Optional[str]:
    """
    当 device_type 对应多个硬件系列时，按「用户平台约束等价类 + devices.yaml 实际设备 hardware_platform」
    联合决策消歧（S2-E），返回 hardware_platform 等价类（P1-3）。非首中逻辑。

    关键：device-reference.md 中 A1500-HU 同时属于 杰伦海光 和 乐研海光3250，
    两者 hardware_platform 等价类不同（JL-HG vs HG3250）。conflicts 与 pool 的 hardware_platform
    统一使用等价类取值，不做系列名 ↔ 等价类混用。
    """
    conflicts = platform_map.get('conflicts', {})
    if device_type not in conflicts:
        return None  # 无冲突

    entry = conflicts[device_type]
    candidate_platforms = entry['candidates']   # 等价类列表（如 [JL-HG, HG3250]）

    # 分支 1：用户显式 platform 约束的等价类优先（S2-E）
    if user_equiv_class and user_equiv_class in candidate_platforms:
        return user_equiv_class

    # 分支 2：devices.yaml 实际设备仲裁（集合求交，非首中）
    actual_platforms = set()
    for dev in pool_devices:
        if dev.get('device_type') == device_type:
            hp = dev.get('hardware_platform')   # 池内取值 = 等价类（如 HG3250）
            if hp and hp in candidate_platforms:
                actual_platforms.add(hp)

    if len(actual_platforms) == 1:
        return next(iter(actual_platforms))
    if len(actual_platforms) == 0:
        raise TopoLimitError(
            code="AMBIGUOUS_MODEL",
            details=f"device_type '{device_type}' 无候选等价类命中。候选: {candidate_platforms}。"
                    f"请显式指定 platform（如 乐研海光3250 / 杰伦海光）。"
        )
    # len > 1：同一 device_type 在池中命中多个不同等价类 → 无法自动消歧
    raise TopoLimitError(
        code="AMBIGUOUS_MODEL",
        details=f"device_type '{device_type}' 在池中命中多个平台: {sorted(actual_platforms)}。"
                f"请显式指定 platform。"
    )
```

**逐冲突消歧策略**（S2-E）：

| device_type | 候选等价类 | 消歧策略 |
|---|---|---|
| `DAS-TGFW-2900` | [nxp1046-16g, C3758] | 按 CPU 平台/OS：nxp1046-16g=ARM/16G（老 NXP1046-2U），C3758=X86/das-os 用 HG 包；devices.yaml hardware_platform 实际值仲裁 |
| `DAS-TGFW-A1200-FU` | [HD-D2000, D2000] | 按硬件系列：华电飞腾 HD-D2000（老飞腾/千兆/无硬盘）vs 乐研D2000 D2000（飞腾 D2000）；devices.yaml 实际值仲裁 |
| `DAS-TGFW-A1280-FU` | [HD-D2000, D2000] | 同 A1200-FU：华电飞腾 HD-D2000 vs 乐研D2000 D2000；devices.yaml 实际值仲裁 |
| `DAS-TGFW-A1500-HU` | [JL-HG, HG3250] | 按海光平台：杰伦海光 JL-HG（老海光/万兆）vs 乐研海光3250 HG3250（海光 3250-40G）；devices.yaml 实际值仲裁；用户显式 platform 时平台约束等价类优先 |

### 8.4 空 limit 语义

空 limit 有以下含义：
- 用户未指定任何约束 → 等价于现有 topo-config 行为（仅按 node_type + min_ports 匹配）
- `TopoLimit(is_empty=True)` 在 S4 topo_mapper 中被特殊处理（跳过约束过滤，不做剪枝）

### 8.5 limit-syntax.md 交付结构

作为显式交付物（G5），`limit-syntax.md` 必须包含：

1. **概述**：语法用途与适用范围
2. **结构化格式定义**：`TopoLimit` JSON Schema（链路约束字段统一为 `link`，S2-C）
3. **设备约束语法**：
   - `platform` 别名表（13 硬件系列 × 可用别名，主键=规范系列名，also_known_as=别名，P1-2/P2-3；`nxp1043-4g` / `nxp1043-8g` 拆分说明，S2-D）
   - **TG 分支（S2-B）**：`node_type: TG` 时 `platform` 字段映射 `sub_type`（ixia-c/trex）与 `hardware_platform`（EP/C236/J1900），6 组合表，不套 DUT 型号表
   - `node_ref` 指定 vs 空（all 语义）示例
   - `node_type` 枚举：DUT/TG/SW/PC/Mock
4. **链路约束语法**：
   - 3 种模式：`include-at-least-one` / `all` / `specific`（specific 用 `link` 指定，S2-C）
   - **speed_class 别名表（S2-A）**：`10GE/万兆/TE`→`TE`，`QTE/40GE`→`QTE`，`TTE/25GE`→`TTE`，`GE/千兆`→`GE`
   - `media_type` 枚举：copper/fiber
5. **端口兼容矩阵表**
6. **完整示例集**（≥5）：
   - 例1：单 DUT 设备约束（海光3250）
   - 例2：多 DUT all 语义（海光3250，2 DUT）
   - 例3：链路约束 include-at-least-one（至少一条 **10GE** 链路 → 归一 TE，S2-A）
   - 例4：链路约束 all（所有链路 GE copper）
   - 例5：复合约束（设备+链路+**TG sub_type 分支**，S2-B）
7. **显式反例集**（≥5）：
   - 反例1：`TE + copper` → PARAM_INVALID
   - 反例2：`XTE + copper` → PARAM_INVALID
   - 反例3：`platform: "未知设备"` → PLATFORM_UNKNOWN
   - 反例4：`mode: "specific"` 但无 `link` 字段 → PARAM_INVALID（S2-C）
   - 反例5：`node_type: "INVALID"` → PARAM_INVALID
   - 反例6：`speed_class: "10ge"` 且 `media_type: "copper"` → 归一 TE 后 PARAM_INVALID（S2-A）
   - 反例7：`node_type: TG` + `platform: "ixia-c2"` → TG_PLATFORM_UNKNOWN（S2-B）
8. **空 limit 行为说明**
9. **型号冲突消歧说明**：conflicts（2900/A1200-FU/A1280-FU/A1500-HU）→ 候选等价类 + 「用户平台约束等价类 + devices.yaml 实际设备」联合决策，非首中（S2-E）
10. **nxp1043 等价类拆分说明**：`nxp1043-4g`(→DAS-TGFW-160，4g-无盘) vs `nxp1043-8g`(→160-PRO/290/490/690，8g)，防整族归 8g 错配（S2-D）

## 9. 安全与性能设计

### 安全

- **JSON 输入校验**：`json.loads` 仅解析合法 JSON；若输入是 dict 则跳过解析
- **无凭据传递**：limit 输入不含密码/IP，仅含设备型号与约束条件
- **YAML 安全加载**：`platform_alias.yaml` 使用 `yaml.safe_load`

### 性能

- 平台映射表一次加载，内存常驻（~50 条目 × ~10 型号 = <5KB）
- 单次 `parse_limit` 调用 < 10ms（纯内存操作，无 I/O）
- `platform_alias.yaml` 懒加载（首次调用 parse_limit 时加载，后续复用模块级缓存）

## 10. 测试设计

### 单元测试（`skills/topo-planning/tests/test_limit_parser.py`）

| 测试用例 | 覆盖要点 | 预期 |
|---------|---------|------|
| `test_parse_device_constraint_platform` | platform="海光3250" → 4 个 device_type 展开 | DeviceConstraint(device_types=[A1300-HU, A1500-HU, A1580-HU, A1600-HU]) |
| `test_parse_device_constraint_aka` | platform="HG3250" 通过 also_known_as 匹配 | 同上 |
| `test_parse_device_constraint_all_scope` | node_ref 空 + node_type=DUT | scope="all"（S2-F 保持） |
| `test_parse_device_constraint_specific_node` | node_ref="dut1" + platform="海光3250" | scope 默认 all 但仅作用于 dut1 |
| `test_disambiguate_in_main_flow` | `parse_limit({device_constraints:[{platform:"海光3250"}]}, pool_devices=[{device_type:"DAS-TGFW-A1500-HU", hardware_platform:"HG3250"}])` | DeviceConstraint.resolved_equiv_class=="HG3250"、`equiv_class_map["DAS-TGFW-A1500-HU"]=="HG3250"`、disambiguation 记录（R4 主流程接入） |
| `test_disambiguate_in_main_flow_no_pool` | 同上但 `pool_devices=None`，且用户未显式区分系列 | 冲突型号 A1500-HU 无法消歧 → `TopoLimitError(AMBIGUOUS_MODEL)`（R4：缺 pool_devices 且无显式 platform 时非首中） |
| `test_disambiguate_in_main_flow_user_priority` | `parse_limit({platform:"乐研海光3250"}, pool_devices=[{device_type:"DAS-TGFW-A1500-HU", hardware_platform:"JL-HG"}])` | 用户显式 platform 等价类 HG3250 优先 → resolved=="HG3250"（R4，用户约束优先） |
| `test_platform_alias_priority` | `parse_limit({platform:"海光3250"})` | 完整平台名精确命中乐研海光3250（HG3250），device_types=[A1300-HU,A1500-HU,A1580-HU,A1600-HU]，**不命中杰伦海光**（Minor） |
| `test_platform_short_alias_ambiguous` | `parse_limit({platform:"海光"})`（无 pool_devices） | `TopoLimitError(PLATFORM_AMBIGUOUS)`，候选含杰伦海光与乐研海光3250（Minor，非首中） |
| `test_platform_short_alias_disambiguated_by_pool` | `parse_limit({platform:"海光"}, pool_devices=[{device_type:"DAS-TGFW-A1500-HU", hardware_platform:"JL-HG"}])` | pool_devices 交集消歧 → 命中杰伦海光（JL-HG），非首中（Minor） |
| `test_tg_sub_type_branch` | node_type=TG + sub_type="trex" | DeviceConstraint(sub_type="trex", hardware_platform=None)，不套 DUT 型号表（S2-B） |
| `test_tg_hardware_platform_branch` | node_type=TG + hardware_platform="C236" | DeviceConstraint(sub_type=None, hardware_platform="C236")（S2-B） |
| `test_tg_combo_branch` | node_type=TG + platform="trex:C236" | DeviceConstraint(sub_type="trex", hardware_platform="C236")（S2-B） |
| `test_tg_platform_unknown` | node_type=TG + sub_type="ixia-c2" | TopoLimitError(TG_PLATFORM_UNKNOWN)（S2-B） |
| `test_parse_link_include_at_least_one` | mode="include-at-least-one" + TE fiber | 通过 |
| `test_parse_link_all_mode` | mode="all" + GE copper | 通过 |
| `test_parse_link_specific_mode` | mode="specific" + link="link3" | 通过（S2-C 字段名 link） |
| `test_parse_link_specific_no_link` | mode="specific" 无 link 字段 | TopoLimitError(PARAM_INVALID)（S2-C） |
| `test_speed_class_alias_10ge` | speed_class="10GE" → 归一 TE + fiber | 通过；LinkConstraint.speed_class == "TE"（S2-A） |
| `test_speed_class_alias_wangzhao` | speed_class="万兆" → 归一 TE + fiber | 通过（S2-A） |
| `test_speed_class_alias_qte_40ge` | speed_class="40GE" → 归一 QTE + fiber | 通过；speed_class == "QTE"（S2-A） |
| `test_speed_class_alias_tte_25ge` | speed_class="25GE" → 归一 TTE + fiber | 通过；speed_class == "TTE"（S2-A） |
| `test_speed_class_alias_te_copper_invalid` | speed_class="10GE" + media_type="copper" | 归一 TE 后 PARAM_INVALID（S2-A） |
| `test_speed_class_unknown` | speed_class="unknown-speed" | TopoLimitError(SPEED_CLASS_UNKNOWN)（S2-A） |
| `test_port_compat_te_copper_invalid` | TE + copper | TopoLimitError(PARAM_INVALID) |
| `test_port_compat_ge_fiber_valid` | GE + fiber | 通过 |
| `test_port_compat_xte_copper_invalid` | XTE + copper | TopoLimitError(PARAM_INVALID) |
| `test_empty_limit_none` | parse_limit(None) | TopoLimit(is_empty=True)（S2-F 保持） |
| `test_empty_limit_str` | parse_limit("") | TopoLimit(is_empty=True)（S2-F 保持） |
| `test_empty_limit_dict` | parse_limit({}) | TopoLimit(is_empty=True)（S2-F 保持） |
| `test_platform_unknown` | platform="不存在的平台" | TopoLimitError(PLATFORM_UNKNOWN) |
| `test_disambiguate_a1500_hu_hg3250` | A1500-HU + 池仅 HG3250 设备 | 消歧返回 HG3250（S2-E） |
| `test_disambiguate_a1500_hu_jl_hg` | A1500-HU + 池仅 JL-HG 设备 | 消歧返回 JL-HG（S2-E） |
| `test_disambiguate_a1500_hu_user_priority` | A1500-HU + 用户 platform=乐研海光3250（HG3250） | 返回 HG3250，用户约束优先（S2-E） |
| `test_disambiguate_multi_ambiguous` | A1500-HU + 池同时含 HG3250 与 JL-HG | TopoLimitError(AMBIGUOUS_MODEL)（S2-E 非首中） |
| `test_disambiguate_2900_strategy` | DAS-TGFW-2900 + 池 C3758 设备 | 消歧返回 C3758（按策略 + 实际设备，S2-E） |
| `test_nxp1043_split_equiv_class` | nxp1043 家族展开 | DAS-TGFW-160 → equiv_class nxp1043-4g-无盘；160-PRO/290/490/690 → nxp1043-8g（S2-D） |
| `test_compound_limit` | 设备约束+链路约束 复合 | 展开且校验通过 |
| `test_json_string_input` | 输入为合法 JSON 串 | 解析同 dict |
| `test_invalid_mode` | mode="invalid_mode" | TopoLimitError(PARAM_INVALID) |

### Fixture 需求

- `tests/fixtures/platform_alias_test.yaml`：精简版映射表（含 DUT aliases（含 `nxp1043-4g`/`nxp1043-8g` 拆分 + `equiv_class_group`）、`tg_platforms` 6 组合、`speed_class_aliases`、`conflicts`（candidates + strategy，S2-B/S2-D/S2-E）），供单元测试隔离
- `tests/fixtures/devices_pool_test.yaml`：含冲突型号设备的 devices.yaml 子集（如 A1500-HU 的 HG3250/JL-HG 设备、2900 的 C3758 设备），供消歧测试（S2-E）
- `tests/fixtures/devices_pool_ambiguous.yaml`：同时含 HG3250 与 JL-HG 设备的 pool（供 R4 多平台并存 AMBIGUOUS_MODEL 测试）
- S2 单元测试不依赖真实 `device-reference.md`，使用 fixture 映射表保证测试自包含

### 语法规范文档验证

- S10 将审核 `limit-syntax.md` 示例/反例集完整性（≥5+5）
- 每个示例可在 `limit_parser.parse_limit()` 中实际运行验证

## 11. 实施步骤

| TASK-ID | 描述 | 预估 | 产物 |
|---------|------|:---:|------|
| TASK-S2-01 | 定义 `TopoLimit` / `DeviceConstraint` / `LinkConstraint` / `TopoLimitError` 数据类（含 TG sub_type/hardware_platform 字段、equiv_class_map，S2-B/S2-D） | 1h | 数据模型 |
| TASK-S2-02 | 从 device-reference.md 提取 `platform_alias.yaml`（13 硬件系列 + nxp1043-4g/8g 拆分 + tg_platforms 6 组合 + speed_class_aliases + conflicts，P1-2/P2-3，S2-A/B/D/E） | 1.5h | 映射数据文件 |
| TASK-S2-03 | 实现 `_expand_platform`（精确/also_known_as/子串匹配 3 层 + equiv_class_group 反查，S2-D） | 1h | 别名展开 |
| TASK-S2-04 | 实现 `_parse_device_constraints`（含 all 语义 + scope 解析 + TG 分支 `_expand_tg_platform`，S2-B） | 1h | 设备约束解析 |
| TASK-S2-05 | 实现 `_parse_link_constraints`（3 模式 + link 字段名 + `_normalize_speed_class` + 端口兼容矩阵校验，S2-A/S2-C） | 1.5h | 链路约束解析 |
| TASK-S2-06 | 实现 `parse_limit` 主入口（dict/str/None 统一 + 空 limit） | 1h | 核心入口 |
| TASK-S2-07 | 实现 `_disambiguate_model`（「用户平台约束等价类 + devices.yaml 实际设备」联合决策，非首中，S2-E）+ **接入 `parse_limit` 主流程**（冲突型号逐一消歧 + `resolved_equiv_class`/`disambiguation` 输出，R4） | 1.5h | 冲突消歧 + 主流程接入 |
| TASK-S2-07b | `_expand_platform` 别名优先级修正（完整平台名优先 + 简写多命中 `PLATFORM_AMBIGUOUS`/pool_devices 消歧，Minor） | 45min | 别名消歧 |
| TASK-S2-08 | 编写单元测试（40 用例 + 3 fixture，含 speed 别名/TG 分支/nxp1043 拆分/联合消歧/R4 主流程/别名优先级） | 2.5h | test_limit_parser.py |
| TASK-S2-09 | 编写 `limit-syntax.md`（≥5 示例 + ≥7 反例 + speed_class 别名表 + TG 分支 + 联合消歧说明 + 别名歧义说明） | 2h | 语法规范文档 |
| TASK-S2-10 | 端到端验证：对所有示例/反例运行 `parse_limit` 确认识别 | 1h | 验证通过 |
| **合计** | | **~15h** | |

## 12. 风险、难点与预研建议

| 风险 | 级别 | 缓解 |
|------|:---:|------|
| device-reference.md 更新后 platform_alias.yaml 不同步 | 中 | 在 YAML 头部记录 `source: device-reference.md` + `source_version`，S10 测试校验映射覆盖率 |
| 中文平台别名模糊匹配误判 | 低 | 只支持精确 + also_known_as + 唯一子串匹配，不引入语义相似度 |
| Agent 生成的 limit JSON 格式不兼容 | 中 | `parse_limit` 做 schema 前置校验（JSON Schema 或字段白名单），非法字段 → PARAM_INVALID + 提示合法格式 |
| limit-syntax.md 与 parser 实现一致性问题 | 低 | S10 用语法文档示例做自动化回归（parse 每个示例并验证输出） |
| speed_class 别名表（10GE/万兆/40GE/25GE）覆盖不全，核心场景解析失败 | 高 | `_normalize_speed_class` + 单测覆盖别名映射（S2-A），语法规范同步；未知值抛 SPEED_CLASS_UNKNOWN 不静默 |
| TG 平台约束（ixia-c/trex）误走 DUT 型号表 | 中 | `_expand_tg_platform` 独立分支 + `tg_platforms` 6 组合校验（S2-B），单测覆盖 |
| nxp1043 整族归 8g 导致 DAS-TGFW-160（4g-无盘）错配 | 高 | `equiv_class_group` 拆分 nxp1043-4g/8g（S2-D），单测覆盖逐型号等价类反查 |
| **冲突型号消歧未接入主流程导致默认等价类首中**（R4） | 高 | `parse_limit` → `_parse_device_constraints` 对 conflicts 中 device_type 逐一调 `_disambiguate_model`（入参 `pool_devices`），`resolved_equiv_class` 写入输出契约；缺 pool_devices 且无显式 platform 时抛 `AMBIGUOUS_MODEL`（单测 `test_disambiguate_in_main_flow*`） |
| **简写平台别名首中误判**（如 "海光" 首中杰伦海光，Minor） | 中 | `_expand_platform` 完整平台名优先精确匹配；简写多命中抛 `PLATFORM_AMBIGUOUS` 或按 pool_devices 交集消歧，非首中（单测 `test_platform_alias_priority` / `test_platform_short_alias_ambiguous`） |

**预研建议**：
1. 过一遍 device-reference.md 13 个系列确认全部 also_known_as 覆盖（nxp1043/nxp1046/C3758R/C236/EP/华电飞腾/杰伦海光/乐研E2000Q/乐研D2000/乐研海光3250/乐研海光5380/天池云/华为鲲鹏 的常用别名，P1-2）
2. 与 S7（topo-planning skill）对齐 Agent 到 CLI 的 NLP 转译模板（确保 Agent 产出的 limit dict 格式与 parse_limit 预期一致）

## 13. 实现灰区与取舍记录

### 灰区 1：platform_alias.yaml 独立 vs 直接解析 device-reference.md

**问题**：平台映射数据从 device-reference.md 提取，可选择每次实时解析 Markdown 表，或提取为独立 YAML 数据文件。

**选项**：
- A. 每次实时解析 `device-reference.md` Markdown 表（单一真相源，但依赖 Markdown 格式稳定性）
- B. 提取为 `platform_alias.yaml`（稳定数据格式，但需同步维护）

**决策**：选 B。device-reference.md 的 Markdown 表格式为非结构化数据，直接解析依赖表格分列变化。独立 YAML 文件是结构化数据，解析稳定且可被 CI 校验覆盖率。

**影响面**：新增 `skills/topo-planning/config/platform_alias.yaml`；若 device-reference.md 新增硬件系列，需同步更新 YAML（S10 测试校验）。

### 灰区 2：多 DUT 约束中 all vs any 语义默认值

**问题**：逻辑 topo 有 2 DUT，用户 limit="海光3250"，是否要求两个 DUT 都是海光3250（all）或至少一个（any）？

**选项**：
- A. 默认 all（用户说"海光3250上执行"，隐含所有 DUT 都用海光）
- B. 默认 any（至少一个 DUT 是海光即可，其余可用其他型号）

**决策**：选 A。CR-037 HLD §4.2 H3 明确 `node_ref` 空且指定 `node_type` → "作用于**全部**该 `node_type` 节点（all 语义）"。与用户意图一致：要求拓扑在指定设备系列上执行。

### 灰区 3：Agent NLP 到结构化 limit 的转译边界

**问题**：S2 limit_parser 接受结构化 dict，但用户输入是自然语言（NLP）。谁负责 NLP→dict 转换？

**选项**：
- A. S2 内嵌 NLP 解析器（增加模块复杂度）
- B. Agent（ptm-te）负责 NLP→dict 转译，S2 只接受结构化输入
- C. S7 topo-planning CLI 接受 `--limit` 原生格式参数

**决策**：选 B + C。HLD §4.7 明确 "NLP→CLI 转换由 agent 实现"。S2 只处理结构化输入；S7 CLI 提供 `--limit`/`--limit-file` 命令行入口（接受 JSON 文件路径或 inline JSON 串），Agent 消费 CLI 而非直接 import S2。

**证据**：G5 评审意见确认 "NLP 语法交付物" 为显式文档交付物，不是代码实现。

### 灰区 4：speed_class 别名归一的实现位置（S2-A）

**问题**：用户 NLP 输入「10GE / 万兆」不在 speed_class 枚举（GE/TE/XTE/QTE/TTE/FE），核心场景「海光3250 + TE|10GE 链路」若不做归一将直接抛 SPEED_CLASS_UNKNOWN。

**选项**：
- A. `_parse_link_constraints` 内先 `_normalize_speed_class` 归一，`_validate_port_compat` 只收枚举值
- B. `_validate_port_compat` 内部同时做归一与矩阵校验（耦合归一与物理约束）

**决策**：选 A。归一与物理约束校验职责分离；`LinkConstraint.speed_class` 存归一后枚举值，raw 输入保留在 `TopoLimit.raw_input` 供审计。语法规范（§8.5）与单测（S2-A）同步覆盖别名映射。

**影响面**：新增 `speed_class_aliases` 区段 + `_normalize_speed_class`；limit-syntax.md 增加别名表；反例集增加「10ge+copper 归一后 PARAM_INVALID」。

### 灰区 5：TG 平台约束的映射路径（S2-B）

**问题**：`device-reference.md` 的 13 硬件系列→型号映射表只覆盖 DUT 防火墙型号；TG 节点（ixia-c/trex × EP/C236/J1900）无对应映射。若 TG 平台约束走 `_expand_platform` 会 PLATFORM_UNKNOWN。

**选项**：
- A. `_parse_device_constraints` 内 `node_type == TG` 时走 `_expand_tg_platform`（消费 tg_platforms 6 组合）
- B. 把 TG 组合硬塞进 DUT aliases（污染 DUT 型号表，且 sub_type 无型号展开语义）
- C. TG 不做平台约束（丢失 TG 选型能力）

**决策**：选 A。`tg_platforms` 独立区段承载 2×3=6 组合；DeviceConstraint 增加 `sub_type` / `hardware_platform` 字段；S3/S4 按组合与池 TG 节点匹配。TG 不套 DUT 型号映射表（S2-B 关键修正）。

**影响面**：DeviceConstraint 数据类、`_parse_device_constraints`、platform_alias.yaml `tg_platforms`、limit-syntax.md TG 分支说明。

### 灰区 6：nxp1043 等价类拆分方式（S2-D）

**问题**：`nxp1043` 整族归 8g 会错配 DAS-TGFW-160（实为 4g-无盘，device-reference.md 行1 `nxp1043-4g-无盘`）。等价类需拆分。

**选项**：
- A. 拆为 `nxp1043-4g` 与 `nxp1043-8g` 两个独立 alias 键，`nxp1043` 家族键带 `equiv_class_group` 反查映射
- B. 整族保持单一键，仅调整默认值（无法表达 4g 与 8g 并存）
- C. 全部 nxp1043 型号统一归 4g（错误：160-PRO 等实为 8g）

**决策**：选 A。`nxp1043-4g`(→[160]，4g-无盘) 与 `nxp1043-8g`(→[160-PRO/290/490/690]，8g)；`nxp1043` 家族键 `default=160-PRO`（8g），`equiv_class_group` 提供 `{device_type: equiv_class}` 反查（S2-D 关键修正）。`_expand_platform` 返回 4 元组（含 `equiv_class_map`）。

**影响面**：platform_alias.yaml aliases、`_expand_platform` 返回签名、DeviceConstraint.equiv_class_map、S4 匹配按 device_type 反查等价类。

### 灰区 7：冲突型号消歧的接入时机与输入来源（R4）

**问题**：`_disambiguate_model`（S2-E）设计已有，但 `parse_limit` 主流程未调用它——冲突型号（A1500-HU/2900/A1200-FU/A1280-FU）在无消歧时按默认等价类首中，与「非首中联合决策」矛盾。

**选项**：
- A. `parse_limit` 增加 `pool_devices` 入参，`_parse_device_constraints` 展开后对 conflicts 中 device_type 逐一调 `_disambiguate_model`，`resolved_equiv_class` 写入 DeviceConstraint（R4 主流程接入）
- B. 消歧留在 S4 topo_mapper 消费时做（S2 不感知 pool_devices，但 S2 输出 device_types 本身携带歧义）
- C. 冲突型号直接抛错要求用户显式 platform（无 pool_devices 时最保守）

**决策**：选 A，C 作为 A 的退化分支。S2 是消歧唯一入口（输出契约含消歧后 device_type 集合）；`pool_devices=None` 且用户未显式指定区分系列时抛 `AMBIGUOUS_MODEL`。理由：把消歧放 S4 会让 limit 解析产物携带歧义，S4 回溯剪枝时每个 device_type 无法确定唯一 hardware_platform，破坏匹配确定性。S3 归并后传 `pool_devices`（devices.yaml 实际设备）给 CLI/S7 skill，再由 CLI 传 `parse_limit`。NFR3 同步修正：确定性声明 = 给定相同 `limit_input` + `pool_devices` 输入则输出一致。

### 灰区 8：简写平台别名"海光"的匹配策略（Minor）

**问题**：`also_known_as` 中"海光"同时属于 `杰伦海光`（JL-HG）与 `乐研海光3250`（HG3250）。旧 `_expand_platform` 在 also_known_as 遍历时按字典序**首中**杰伦海光，导致用户写"海光"被误判为杰伦海光（老海光/万兆），而非其真实意图（通常为乐研海光3250 的海光平台）。

**选项**：
- A. **完整平台名优先精确匹配**：`"海光3250"`（唯一存在于乐研海光3250）→ 精确命中乐研海光3250；简写 `"海光"` 多命中 → 抛 `PLATFORM_AMBIGUOUS` 或按 pool_devices 实际设备交集消歧，非首中
- B. 保留首中（简单但误判核心场景）
- C. 移除"海光"简写别名（破坏用户自然语言输入习惯）

**决策**：选 A。完整平台名优先（`"海光3250"`→乐研海光3250，核心场景正确）；简写 `"海光"` 歧义时非首中——有 pool_devices 按交集消歧，无 pool_devices 抛 `PLATFORM_AMBIGUOUS` 提示候选。单测 `test_platform_alias_priority` / `test_platform_short_alias_ambiguous` / `test_platform_short_alias_disambiguated_by_pool` 覆盖。limit-syntax.md 增加「平台别名歧义处理」说明。

## 14. 回滚与发布策略

### 回滚

- `limit_parser.py` + `platform_alias.yaml` + `limit-syntax.md` 均为新增文件，回滚即删除
- 无对其他模块的修改，不影响现有功能

### 发布

- S2 模块随 `topo-planning` skill（S7）统一安装
- `limit-syntax.md` 随 skill 文档树安装（`~/.ptm-team/skills/topo-planning/docs/`）
- S2 模块独立于 S1/S3/S4（无内部依赖），可提前交付测试

## 15. Definition of Done（DoD）

- [ ] `parse_limit` 对 dict/str/None 三种输入格式正确处理
- [ ] `platform_alias.yaml` 覆盖 device-reference.md 全部 13 个硬件系列映射（主键=规范系列名，also_known_as 收别名，equiv_class 承载硬件等价类，P1-2/P2-3）
- [ ] **speed_class 别名归一**：`10GE/万兆/TE`→`TE`、`QTE/40GE`→`QTE`、`TTE/25GE`→`TTE`；归一后做兼容矩阵校验（S2-A）
- [ ] 端口兼容矩阵正确校验：GE→{copper,fiber}，TE/XTE/QTE/TTE→{fiber}，非法组合→PARAM_INVALID（S2-F 保持）
- [ ] **TG 平台约束分支**：`node_type: TG` 时 platform → sub_type/hardware_platform（ixia-c/trex × EP/C236/J1900），不套 DUT 型号表（S2-B）
- [ ] **链路约束字段名统一为 `link`**（弃用 link_id），与 collection links 命名一致（S2-C）
- [ ] **nxp1043 等价类拆分**：`nxp1043-4g`(→160) vs `nxp1043-8g`(→160-PRO/290/490/690)，`equiv_class_map` 逐型号反查（S2-D）
- [ ] 设备约束 all 语义：node_ref 空 + node_type 指定 → scope="all"（S2-F 保持）
- [ ] 链路约束 3 模式全部实现且校验通过
- [ ] **型号冲突消歧为联合决策且接入主流程**：`parse_limit` 对 conflicts 中 device_type（2900/A1200-FU/A1280-FU/A1500-HU）逐一按「用户平台约束等价类 + devices.yaml 实际设备」消歧，非首中；`resolved_equiv_class`/`disambiguation` 写入 DeviceConstraint（R4）；缺 pool_devices 且无显式 platform 或多平台并存 → AMBIGUOUS_MODEL（S2-E）
- [ ] **平台别名优先级**：完整平台名优先精确匹配（"海光3250"→乐研海光3250）；简写"海光"多命中抛 PLATFORM_AMBIGUOUS 或按 pool_devices 交集消歧，非首中（Minor，FR12）
- [ ] 空 limit 返回 is_empty=True，不阻断后续匹配（S2-F 保持）
- [ ] `limit-syntax.md` 包含 ≥5 个完整示例 + ≥7 个显式反例 + speed_class 别名表 + TG 分支 + 联合消歧说明 + 平台别名歧义说明
- [ ] 40 个单元测试全部通过（含 speed 别名/TG 分支/nxp1043 拆分/联合消歧/R4 主流程/别名优先级）
- [ ] TopoLimitError 统一错误格式（code + details）
- [ ] NFR3 确定性依赖关系成立：给定相同 `limit_input` + `pool_devices` → 输出确定性一致（R4）
- [ ] 语法规范文档示例可通过 `parse_limit` 实际运行验证
