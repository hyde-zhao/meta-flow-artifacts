---
cr_id: "CR-033"
artifact_type: "feature-design"
feature_id: "FE-EX-03"
feature_name: "规则固化与执行改进"
version: "1.0"
created_at: "2026-07-28T16:15:00+08:00"
author: "meta-se"
status: "draft"
source_hld: "docs/design/HLD-PTM-TE-EXEC.md"
source_blueprint: "docs/design/BLUEPRINT-PTM-TE-EXEC.md"
source_matrix: "docs/design/FEATURE-DESIGN-MATRIX-PTM-TE-EXEC.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-PTM-TE-EXEC.md"
source_dev_plan: "process/DEVELOPMENT-PLAN.yaml"
covered_stories:
  - "ST-EX-02"
  - "ST-EX-13"
cross_feature_dependencies:
  - "ST-EX-01 (FE-EX-01 waived): devices.yaml tg 块"
  - "ST-EX-03 (FE-EX-01 waived): op_mapper TREX_API_URL 注入 + resolve_env_refs"
  - "ST-EX-08 (FE-EX-02): ARP 预热引擎强制（规则侧双重保障 ADR-06）"
  - "ST-EX-15 (FE-EX-02): 用例结构化约定（24 用例整改对齐）"
  - "ST-EX-16 (FE-EX-02): 标签/关键字执行（24 用例 tags 列对齐）"
  - "ST-EX-17 (FE-EX-02): 环境解析层（24 用例 ${ENV.*} 改写对齐）"
validation_mode: "static-only + dry-run-only + review-only"
---

# FE-EX-03 规则固化与执行改进 Feature 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-28 | meta-se | CP3 approved / CP4 PASS 后生成：install.py 规则块文本设计（新增 ≥4 条 + 更新规则 4 执行入口）；24 用例整改映射表（目录迁移/重命名/frontmatter 16 列/tags/ARP 预热校验/${ENV.*} 改写）；exec_v4.py 废弃标记；跨 Feature 依赖（ST-EX-01/03/08/15/16/17）；ADR-03/06 追溯；覆盖 ST-EX-02/13 设计要点 |

## 1. Feature 能力边界

### 1.1 能力清单

| 能力 | 说明 | 承载 Story | HLD 章节 |
|---|---|---|---|
| install.py 规则块新增 ≥4 条 | TG路由/max_loss/ARP预热/session 生命周期规则固化防重装回退 | ST-EX-02 | §12.4, ADR-06 |
| 规则块执行入口更新 | 规则 4 从 cases/upload/ 更新为 cases/三级结构 | ST-EX-02, ST-EX-13 | §12.4, Gotcha #5 |
| 24 用例目录迁移 | cases/upload/ -> cases/特性/配置管理/策略配置/ 三级结构 | ST-EX-13 | §13.5, Gotcha #5 |
| 24 用例重命名 | <编号>-<名称>.md（ADR-07 正则） | ST-EX-13 | Gotcha #6 |
| 24 用例 frontmatter 16 列补全 | 8 必填 + 8 可选，缺失填 N/A | ST-EX-13 | Gotcha #7, RA-012 |
| 24 用例 tags/关键字标注 | tags 列 + 关键字列，支持 --tag/--keyword | ST-EX-13 | §12.1 [3] |
| 24 用例 ARP 预热校验 | warming_up:true + post_op 合规校验 | ST-EX-13 | §12.1 [5], ADR-06 |
| 24 用例 ${ENV.*} 引用改写 | 环境相关参数改写为 ${ENV.*} 占位符 | ST-EX-13 | §12.3, ADR-09 |
| exec_v4.py 废弃标记 | 迁移后加废弃标记不删除（RA-009） | ST-EX-13 | ADR-03 |
| 重装一致性验证 | install/uninstall/install 循环验证规则块+skill 不丢失 | ST-EX-02 | Gotcha #10 |

### 1.2 非目标（Out of Scope）

- 不改 traffic-skill / ngfw-install skill（R-C-005）
- 不采集 TG 系统快照（R-C-006）
- 不改 ptm-atomic CLI 本体（R-C-001）
- 不删除 exec_v4.py（ADR-03，保留废弃标记可回退）
- 不改 op_mapper 映射表（ST-EX-03/07 属 FE-EX-01/02）
- 不实现 case_runner.py 引擎逻辑（属 FE-EX-02）

### 1.3 相邻对象边界澄清

| 相邻对象 | 职责差异 | 边界 |
|---|---|---|
| FE-EX-02 (case_runner) | 引擎实现（warming_up 强制/四态分级/解析） | 本 Feature 只固化规则文本与整改用例 md，不实现引擎逻辑 |
| FE-EX-01 (op_mapper) | TREX_API_URL 注入/resolve_env_refs/fw_logout 映射 | 本 Feature 规则块描述 op_mapper 行为约束，不改 op_mapper 代码 |
| FE-EX-01 (device-management) | devices.yaml tg 块 schema | 本 Feature 规则块引用 devices.yaml tg.api_server，不定义 schema |
| install.py 既有 8 条规则 | dry-run/凭据/session 路径/执行入口/op_id/id 来源/清理/版本锁定 | 本 Feature 新增 ≥4 条 + 更新规则 4，不重写既有规则 |

## 2. 数据归属

| 数据对象 | 归属 | 存储位置 | 写入方 | 读取方 |
|---|---|---|---|---|
| install.py 规则块 | FE-EX-03 | script/ptm_team/install.py#render_ptm_te_rule_body | ST-EX-02 | install.py 安装时渲染到 CLAUDE.md |
| ptm-te-workflow 规则块（CLAUDE.md） | FE-EX-03 | workspace CLAUDE.md（managed block） | install.py 渲染 | ptm-te agent 执行时 |
| 24 用例 md | FE-EX-03 整改 / FE-EX-02 解析 | ptm-te/cases/特性/配置管理/策略配置/*.md | ST-EX-13 | case_runner.parse_case_file 只读 |
| cases/upload/（废弃） | FE-EX-03 | ptm-te/cases/upload/README.md（废弃说明） | ST-EX-13 | 不保留 .md 用例文件（Gotcha #5） |
| exec_v4.py 废弃标记 | FE-EX-03 | ptm-te/exec_v4.py | ST-EX-13 | 不执行（RA-009） |

## 3. 跨模块契约

### 3.1 install.py 规则块 <-> case_runner 引擎（ADR-06 双重保障）

| 字段 | 值 |
|---|---|
| 调用方向 | install.py 渲染规则块到 CLAUDE.md -> ptm-te agent 读规则约束 -> case_runner 引擎实现约束 |
| 调用时机 | install.py install ptm-te --component full 时渲染；ptm-te agent 执行用例时读规则 |
| 输入契约 | render_ptm_te_rule_body(platform) -> 规则块文本（≥12 条，既有 8 + 新增 ≥4） |
| 输出契约 | CLAUDE.md managed block `<!-- ptm-team:managed:ptm-te-workflow:begin -->...<!-- end -->` |
| 后续衔接 | case_runner 引擎实现 warming_up 强制（ST-EX-08）；规则块描述约束，引擎兜底（ADR-06） |
| 降级策略 | 规则块被绕过时引擎兜底（RA-008，ADR-06 双重保障） |
| 调用方需同步修改范围 | ST-EX-13 迁移后更新规则 4 执行入口（cases/upload/ -> cases/三级） |

### 3.2 24 用例 md <-> case_runner 解析（FE-EX-02 协作）

| 字段 | 值 |
|---|---|
| 调用方向 | ST-EX-13 整改用例 md -> case_runner.parse_case_file/parse_frontmatter 解析 |
| 调用时机 | case_runner 执行时只读 |
| 输入契约 | frontmatter 16 列 + case_steps YAML（含 ${ENV.*} 占位符） |
| 输出契约 | case_runner 解析为 CaseRecord + Step 列表 |
| 前置校验 | 24 用例 dry-run 校验全部解析成功（DA-002） |
| 降级策略 | 无 ${ENV.*} 字面值原样透传（Gotcha #12，向后兼容） |

### 3.3 规则块 <-> op_mapper 行为约束（FE-EX-01 协作）

| 字段 | 值 |
|---|---|
| 调用方向 | 规则块描述 op_mapper 行为约束（TREX_API_URL 注入/fw_logout/OP_NOT_FOUND） |
| 调用时机 | 规则块文本描述，不直接调用 op_mapper |
| 输入契约 | 规则块文本引用 op_mapper 函数名（_build_exec_env/execute_op） |
| 输出契约 | ptm-te agent 按规则约束执行 |
| 调用方需同步修改范围 | op_mapper 实现由 ST-EX-03（FE-EX-01）/ST-EX-07（FE-EX-02）承载 |

## 4. install.py 规则块文本设计（ST-EX-02）

### 4.1 既有 8 条规则（保留，不重写）

| 序 | 规则 | CR-033 变更 |
|---|---|---|
| 1 | dry-run 默认门与授权粒度 | 不变 |
| 2 | 凭据安全 | 不变 |
| 3 | session 路径 | 不变 |
| 4 | 执行入口 | **更新**：cases/upload/ -> cases/三级结构（ST-EX-13 迁移后） |
| 5 | op_id 未识别阻塞 | 不变 |
| 6 | id 来源 | 不变 |
| 7 | 清理回滚 | 不变 |
| 8 | 重装 ptm-atomic 前置检查与版本锁定 | 不变 |

### 4.2 新增 ≥4 条规则（ST-EX-02 实现）

规则块文本写入 `render_ptm_te_rule_body()`，f-string 格式（`${{ENV_VAR}}` 转义为 `${ENV_VAR}`）。

**规则 9: TG 路由与 TREX_API_URL 注入**

```
9. **TG 路由与 TREX_API_URL 注入**：TG 操作经 `ptm-atomic run tg trex <action>` 原子操作下发，框架禁止直接调 TG REST API。`case_runner` 从 `devices.yaml` 读 `tg.api_server`（或 `--env-file` 的 `${ENV.tg.url}` 优先）传入 `op_mapper._build_exec_env`，注入子进程环境变量 `TREX_API_URL`，由 ptm-atomic 子进程消费；不依赖调用方手动 export（ADR-05 方案 a）。
```

**规则 10: max_loss 参数化**

```
10. **max_loss 参数化**：`tg_verify_traffic_loss` 的 `--max-loss` 必须参数化传入，禁止硬编码；用例 case_steps 通过 args.max_loss 声明期望阈值，case_runner 透传 op_mapper，不得在脚本中写死默认值（#2 改进）。
```

**规则 11: ARP 预热双重保障**

```
11. **ARP 预热双重保障**：`warming_up: true` 的 step 执行主 op（tg_start_traffic_stream）后，case_runner 引擎强制补充 post_op（tg_stop_traffic_stream），参数从主 op 继承（ports/txport/rxport/name）；即使用例 md 未写 post_op，引擎自动补充（result 标记 auto_post_op=true）。本规则与引擎强制互为双重保障，重装后规则被绕过时引擎兜底（ADR-06，#3 改进）。
```

**规则 12: session 生命周期与 fw_logout 登出**

```
12. **session 生命周期与 fw_logout 登出**：本轮全部用例执行结束后，case_runner 必须调用 `op_mapper.execute_op(fw_logout)` 登出并清理 session；ptm-atomic 安装版未暴露 fw_logout 时降级为 `os.remove(session_file)`，result.json 记录 `logout=fallback_session_cleanup`。不得跨轮复用 session，新一轮启动需重新登录（#4 改进，ADR-04）。
```

### 4.3 规则 4 执行入口更新（ST-EX-13 协作）

ST-EX-13 迁移后，规则 4 从：

```
4. **执行入口**：用例从 `cases/upload/<特性名>特性测试用例.md` 读取，不直接读 ptm-tde 的 `ppdcs/delivery/`。
```

更新为：

```
4. **执行入口**：用例从 `cases/<特性>/<配置管理>/<策略配置>/*.md` 三级目录结构读取（case_runner --cases-dir glob），不直接读 ptm-tde 的 `ppdcs/delivery/`；旧 `cases/upload/` 已废弃，只保留 README.md 废弃说明，不含 .md 用例文件（Gotcha #5）。
```

**实施顺序**：ST-EX-02 先实现新增 ≥4 条规则（规则 4 暂保留旧文本）；ST-EX-13 迁移完成后更新规则 4。两者在同一规则块 ID `ptm-te-workflow` 下，由 `render_ptm_te_rule_body` 单点渲染。

### 4.4 重装一致性验证（Gotcha #10）

ST-EX-02 实现规则块后，执行 `install.py install ptm-te -> uninstall -> install` 循环验证：
- 规则块 12 条（既有 8 + 新增 4）不丢失
- managed block ID `ptm-te-workflow` 正则匹配成功
- op_mapper validate_mapping_consistency 通过（EXPECTED_OP_COUNT=22，ST-EX-07 协作）

## 5. 24 用例整改映射表（ST-EX-13）

### 5.1 目录迁移映射

| 迁移前 | 迁移后 | 说明 |
|---|---|---|
| ptm-te/cases/upload/IPv4策略路由特性测试用例.md | ptm-te/cases/IPv4策略路由/配置管理/策略配置/*.md | 拆分为三级目录，每用例一文件 |
| ptm-te/cases/upload/ | ptm-te/cases/upload/README.md（废弃说明） | 只保留废弃说明，不含 .md 用例文件（Gotcha #5） |

**目录结构**（与 ST-EX-15 用例结构化约定对齐，FE-EX-02）：

```
ptm-te/cases/
└── IPv4策略路由/
    └── 配置管理/
        └── 策略配置/
            ├── PC-COMB-M1-01-01-创建策略路由-有效参数.md
            ├── PC-COMB-M4-01-01-...md
            └── ... (24 个用例文件)
```

### 5.2 重命名规则（ADR-07，Gotcha #6）

文件名格式：`<编号>-<名称>.md`
- 编号正则：`^PC-[A-Z0-9]+-\d+-\d+-\d+`（如 PC-COMB-M4-01-01）
- 名称：用例标题，含连字符不冲突（按 frontmatter 用例编号列为唯一标识）
- 示例：`PC-COMB-M4-01-01-创建策略路由-有效参数.md`

### 5.3 frontmatter 16 列补全（DQ-05，RA-012）

| 序 | 列名 | 必填 | 整改要求 |
|---|---|---|---|
| 1 | 用例编号 | 是 | 已有，校验正则 |
| 2 | 用例标题 | 是 | 已有 |
| 3 | 用例目的 | 是 | 已有 |
| 4 | 特性 | 是 | 已有 |
| 5 | 优先级 | 是 | 已有 |
| 6 | 标签 | 是 | **补全**：按用例类型标注（正向/反向/边界/known_issue） |
| 7 | 关键字 | 是 | **补全**：策略路由/对象/接口/TG 流量等关键词 |
| 8 | 前置条件 | 是 | 已有 |
| 9 | 测试步骤 | 否 | 保留（与 case_steps 冗余，case_runner 忽略，AGA-03=C） |
| 10 | 预期结果 | 否 | 保留（同上） |
| 11 | 用例类型 | 否 | **补全**：正向/反向 |
| 12 | 自动化状态 | 否 | **补全**：auto |
| 13 | 创建人 | 否 | **补全** |
| 14 | 创建日期 | 否 | **补全** |
| 15 | known_issue | 否 | **补全**：DUT 行为差异用例标 true（DQ-CP3-03） |
| 16 | 备注 | 否 | 缺失填 N/A |

缺失可选列填 `N/A`（RA-012）。

### 5.4 ARP 预热校验（ADR-06，#3 改进）

24 用例中含 TG 流量的 step 必须校验：
- `warming_up: true` 标注在 tg_start_traffic_stream step
- post_op（tg_stop_traffic_stream）可由引擎自动补充（ST-EX-08），用例 md 可不显式写
- 整改时校验 warming_up 标注一致性，缺失的补全（SM-EX-07：24/24 合规）

### 5.5 ${ENV.*} 引用改写（ADR-09，与 ST-EX-17 对齐）

24 用例的环境相关参数改写为 ${ENV.*} 占位符（HLD §12.3 迁移示例）：

| 改写前（环境耦合） | 改写后（环境无关） | 说明 |
|---|---|---|
| `interfaces: '[{"port":"2_3",...}]'` | `interfaces: ${ENV.tg.ports[port1,port2]}` | TG 端口聚合 |
| `tx_port: "2_3"` | `tx_port: ${ENV.tg.port1}` | TG 物理端口 |
| `rx_port: "2_4"` | `rx_port: ${ENV.tg.port2}` | TG 物理端口 |
| `next_hop_ip: "192.168.102.1"` | `next_hop_ip: ${ENV.dut.next_hop}` | DUT 下一跳 |
| `in_interface: "eth0"` | `in_interface: ${ENV.dut.port1}` | DUT 物理端口 |

**测试意图参数保持字面值**（src_ip/dst_ip/object_name/l4_protocol 等），不改写。

改写后同一用例在 link3（port1=2_3/port2=2_4）和 link4（port1=2_1/port2=2_2）均可执行，只需换 `--env-file`（R-F-027）。

**向后兼容**：未改写的用例（无 ${ENV.*}）原样透传（Gotcha #12）。ST-EX-13 全量改写 24 用例。

### 5.6 exec_v4.py 废弃标记（ADR-03，RA-009）

ST-EX-13 迁移后在 `ptm-te/exec_v4.py` 头部加废弃标记：

```python
# [DEPRECATED] exec_v4.py 已被 skills/case-execution/scripts/case_runner.py 替代（CR-033）。
# 本文件保留仅作历史参考与回退，不得用于生产执行。
# 新执行入口：python skills/case-execution/scripts/case_runner.py run --cases-dir ...
# 废弃日期：2026-07-28（CR-033 ST-EX-13 迁移完成）
# 删除条件：若废弃标记后仍被误用，删除本文件（RA-009）。
```

不删除文件（ADR-03，可回退）。README 指向 case_runner 新入口。

## 6. 与 HLD 一致性 + ADR 追溯

| ADR | 决策 | 本 Feature 落点 | HLD 章节 |
|---|---|---|---|
| ADR-03 | exec_v4.py 迁移后废弃标记不删除 | §5.6 exec_v4.py 废弃标记 | §14 RA-009 |
| ADR-06 | ARP 预热双重保障（规则+引擎） | §4.2 规则 11（规则侧）+ ST-EX-08 引擎侧（FE-EX-02） | §12.1 [5] / §13.5 |
| ADR-07 | 用例编号正则匹配 | §5.2 重命名规则 | Gotcha #6 |
| ADR-05 | TREX_API_URL 注入（规则描述） | §4.2 规则 9 描述约束 | §12.2 / §12.3 |
| ADR-09 | 环境文件驱动（用例改写） | §5.5 ${ENV.*} 引用改写 | §12.3 |

**量化指标对齐**（HLD §成功标准）：SM-EX-05 规则固化 ≥4 条（§4.2 新增 4 条）/ SM-EX-06 改进覆盖 12/12（§4.2 + §5.4）/ SM-EX-07 ARP 预热 24/24（§5.4）/ SM-EX-09 重装一致性 0 丢失（§4.4）。

## 7. Story 设计要点

| Story | lld_policy | 设计要点 | 证据路径 |
|---|---|---|---|
| ST-EX-02 | technical-note | install.py render_ptm_te_rule_body 新增 ≥4 条规则（TG路由/max_loss/ARP预热/session）+ 重装一致性验证 | process/stories/STORY-EX-02.md |
| ST-EX-13 | full-lld | 24 用例目录迁移 + 重命名 + frontmatter 16 列 + tags + ARP 预热校验 + ${ENV.*} 引用改写 + exec_v4.py 废弃标记 | process/stories/STORY-EX-13-LLD.md |

**Story 数一致性**：本 Feature 覆盖 2 个 Story（ST-EX-02, 13），与 FEATURE-DESIGN-MATRIX FE-EX-03 关联 Story 列、BLUEPRINT §1.3 Epic 拆解（EP-EX-06/07）、DEVELOPMENT-PLAN file_ownership 一致。

## 8. 跨 Feature 依赖

本 Feature 规则固化与用例整改依赖多个跨 Feature Story 的产物，按矩阵归属这些 Story 不属 FE-EX-03，但契约关系必须显式声明：

| 依赖 Story | 归属 Feature | 依赖关系 | 契约 |
|---|---|---|---|
| ST-EX-01 | FE-EX-01 (waived) | 规则 9 引用 devices.yaml tg 块 | devices.yaml 含 tg.api_server（DA-003） |
| ST-EX-03 | FE-EX-01 (waived) | 规则 9 描述 op_mapper TREX_API_URL 注入 | _build_exec_env 签名扩展 + resolve_env_refs（ADR-05/09） |
| ST-EX-08 | FE-EX-02 | 规则 11 ARP 预热与引擎强制双重保障（ADR-06） | 引擎 apply_warming_up 兜底（规则被绕过时） |
| ST-EX-15 | FE-EX-02 | 24 用例整改对齐用例结构化约定 | 目录三级 + 命名正则 + frontmatter 16 列（§5.1-5.3） |
| ST-EX-16 | FE-EX-02 | 24 用例 tags/关键字列对齐 --tag/--keyword | frontmatter tags/关键字列（§5.3） |
| ST-EX-17 | FE-EX-02 | 24 用例 ${ENV.*} 改写对齐环境解析层 | ${ENV.*} 9 类占位符（§5.5，ADR-09） |

**依赖方向**：FE-EX-03 规则块文本描述 FE-EX-01/02 实现的行为约束；FE-EX-03 用例整改消费 FE-EX-02 用例结构化约定与环境解析层契约。FE-EX-03 不修改这些 Story 的代码，只对齐文本与数据格式。

## 9. 前置校验与失败路径

| 阶段 | 前置校验 | 失败行为 |
|---|---|---|
| ST-EX-02 规则块实现 | render_ptm_te_rule_body 既有函数存在 | fail fast |
| ST-EX-02 重装验证 | install/uninstall/install 循环 | 规则块丢失则修复 |
| ST-EX-13 目录迁移 | 24 用例 dry-run 解析成功（DA-002） | 解析失败修复 frontmatter/case_steps |
| ST-EX-13 重命名 | 编号正则匹配（ADR-07） | 不匹配报错让用户手动指定 |
| ST-EX-13 ${ENV.*} 改写 | env-file 存在 + ${ENV.*} 解析成功（DA-006/007） | resolve_env_refs 失败 -> VALIDATION_FAILED（RA-013） |
| ST-EX-13 ARP 预热校验 | warming_up:true 24/24 合规（SM-EX-07） | 缺失补全 |
| ST-EX-13 exec_v4.py 废弃 | 废弃标记头部存在 | 缺失补全 |

## 10. Gotchas

### Gotcha FE03-1: 规则块更新后需验证重装一致性（ST-EX-02）

install.py 新增 ≥4 条规则后，重装可能因规则块 ID 冲突或正则匹配失败丢失规则。ST-EX-02 实现后执行 install -> uninstall -> install 循环验证 12 条规则不丢失；managed block ID `ptm-te-workflow` 正则匹配成功。（HLD Gotcha #10）

### Gotcha FE03-2: 24 用例目录迁移后 case_runner --cases-dir 路径变更（ST-EX-13）

24 用例从 cases/upload/ 迁到 cases/三级/，case_runner --cases-dir 路径需同步更新。旧 cases/upload/ 只保留 README.md 废弃说明，不保留 .md 用例文件，避免误执行。迁移后 dry-run 校验新路径 24 用例全部解析成功。（HLD Gotcha #5）

### Gotcha FE03-3: 用例名称连字符与文件名分隔符冲突（ST-EX-13）

文件名 `<编号>-<名称>.md`，编号和名称都含连字符。直接 split('-') 解析错误。按 frontmatter 用例编号列为唯一标识；文件名按编号前缀正则 `^(PC-[A-Z0-9]+-\d+-\d+-\d+)` 匹配。（HLD Gotcha #6，ADR-07）

### Gotcha FE03-4: frontmatter 测试步骤/预期结果列冗余（ST-EX-13）

frontmatter 的"测试步骤"和"预期结果"列与 case_steps YAML 冗余。case_runner 忽略这两列（AGA-03=C）。整改时保留这两列（不删除），校验脚本只检查 16 列存在性，不校验内容一致性。（HLD Gotcha #7）

### Gotcha FE03-5: frontmatter 16 列补全工作量大（ST-EX-13）

24 用例 × 16 列补全工作量较大。按 DQ-05：8 必填 + 8 可选，缺失可选列填 N/A（RA-012）。known_issue 列需按 DUT 行为差异用例标注（DQ-CP3-03）。

### Gotcha FE03-6: ${ENV.*} 改写需与 ST-EX-17 环境解析层对齐（ST-EX-13）

24 用例 ${ENV.*} 改写必须与 ST-EX-17（FE-EX-02）实现的 ${ENV.*} 9 类占位符解析规则一致。改写后 dry-run 校验全部 ${ENV.*} 能解析成功；未改写的用例（无 ${ENV.*}）原样透传（向后兼容，Gotcha #12）。

### Gotcha FE03-7: 规则 4 执行入口更新需在 ST-EX-13 迁移后（ST-EX-02/13 协作）

ST-EX-02 先实现新增 ≥4 条规则（规则 4 暂保留旧文本 cases/upload/）；ST-EX-13 迁移完成后更新规则 4 为 cases/三级结构。两者在同一 render_ptm_te_rule_body 函数单点渲染，避免规则块分裂。

### Gotcha FE03-8: ARP 预热规则与引擎双重保障不可省略任一侧（ST-EX-02/13，ADR-06）

规则 11（规则侧）与 ST-EX-08 apply_warming_up（引擎侧）互为双重保障。只靠规则时重装后可被绕过（RA-008）；只靠引擎时规则不可见。两侧都必须实现，不得因"已有引擎"省略规则或反之。

## 11. 待确认问题

| 问题 ID | 问题 | 状态 | 决策引用 |
|---|---|---|---|
| O-FE03-01 | 24 用例 known_issue 标注完整性 | OPEN（整改时同步检查） | DQ-CP3-03 / RA-003 |
| O-FE03-02 | fw_logout 在 ptm-atomic 安装版是否暴露（规则 12 降级触发条件） | OPEN（安装前验证） | ADR-04 / DQ-CP3-02 |

## 12. 验证策略（与 TEST-PLAN.md 联动）

本 Feature `validation_mode=static-only + dry-run-only + review-only`（HLD §13.1/§13.3）。CP7 不真实设备写：
- static-only：install.py 规则块文本 review / render_ptm_te_rule_body 函数检查
- dry-run-only：24 用例迁移后 dry-run 批量校验解析 + ${ENV.*} 解析
- review-only：frontmatter 16 列 / 目录结构 / 重命名正则 / ARP 预热校验 / exec_v4.py 废弃标记 人工审查
- 重装一致性：install/uninstall/install 循环验证规则块 12 条不丢失

真实设备 --execute 验证需 runtime_authorization，超出 CR-033 CP7 范围。详见 `TEST-PLAN.md`。
